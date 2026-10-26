# MoPA: Qwen3-VL training on two NPU nodes

## Scope and source

Source: `mopa-mshab/NPU_PORTING.md`, 2026-10-18 snapshot recorded in
[sources.json](sources.json). The source combines XPolicyLab MoPA core at
`ddf5dcfff5e089a323da1d22891909b72864647e` and Qwen3-VL-4B-Instruct
revision `ebb281ec70b05090aa6165b016eac8ec08e71b17`.

The baseline trains all declared parameters, without LoRA or backbone freezing:
BF16 backbone, FP32 queries/action head, no global autocast. Two nodes provide
8×910B1 and 8×910B2, torch/torch_npu 2.6.0 and CANN 8.2.RC1.
Seven MS-HAB skills supply qpos12 and action13 with fixed head channels removed,
giving action11 split 8+3. This is not the paper's exact proprioception protocol.

## Reusable decisions

- Isolate transformers/tokenizers upgrades (4.57.0/0.22.2 here) while keeping the
  platform torch stack. Import and real pretrained-model checks are separate.
- Preserve official processor behavior: two RGB128 images were resized to 256
  and yielded the recorded native grid/token contract. Do not silently reduce
  the visual token budget to fit NPU memory.
- Check real backbone and head changes separately. Small learning rates can
  round away BF16 updates even with finite loss/gradients; parameter counts and
  requires-grad flags do not prove every tensor updated.
- Trace unused graph tails. Sixteen optimizer slots lacked state because the
  last query outputs no longer fed the loss; tiny-model backward corroborated
  the graph explanation. Do not manufacture gradients to fill optimizer slots.
- Node-local directories are not shared storage. Restore only after the complete
  checkpoint reaches both nodes; verify topology, normalization, learning-rate
  schedule, per-rank RNG and consumed sample positions.
- Compare CPU/NPU under the same quantized inputs and processor. A CPU BF16
  reference is not FP32 ground truth or a CUDA parity test.
- Keep cleanup fixes narrow: HDF5 finalization checks avoid interpreter-shutdown
  noise without suppressing normal data-read failures.

## Evidence and limits

The full model completed two real updates, then resumed from step2 to step4;
both nodes exited 0 for both runs. Four steps covered 256 unique samples, with
finite loss/gradients and consistent sampled cross-rank parameter hashes.
Backbone and query probes changed, but the probes deliberately selected small
nonzero-gradient elements and do not estimate the global update fraction.

The step4 checkpoint was replicated and file size/SHA-256 checked. Functional
NPU resume, learning-rate continuation and sample progress passed; a continuous
four-step versus resumed four-step bitwise NPU comparison was not performed.

One real-sample forward comparison gave query rRMSE 1.509% and action-velocity
rRMSE 0.522%/0.674% against CPU BF16. It does not establish backward parity or
task quality. The report mentions a later 100k job launch; this case does not
infer its completion from that launch. Simulation evaluation was deferred.
