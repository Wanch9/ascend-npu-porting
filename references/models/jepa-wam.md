# JEPA-WAM: frozen-feature training

## Scope and source

Source: `JEPA-WAM-NPU-publish/NPU_PORTING.md`, observed on 2026-09-09;
snapshot hash in [sources.json](sources.json). Upstream JEPA-WAM branch
`vit-online-jepa-lora`, base `5aaa85777672dc1cea361062689a238cf552157a`.

The port trains full WAM ViT-B policy and optional future predictor Ψ from
existing V-JEPA/UMT5 feature shards. It does not instantiate or fine-tune the
encoders. The tested contract used two cameras, 256 tokens per camera,
1664 visual channels, 4096 text channels, state dimension 8 and action dimension 7.
Single-node 910B1, torch/torch_npu 2.6.0 and CANN 8.2.RC1 were used.

## Reusable decisions

- Trace the cached-feature trainer separately from online vision/text paths.
  Preserve encoder provenance and normalization; an old cache containing only
  mean/std does not satisfy a trainer expecting min/max. Derive new statistics
  from the intended training split without rewriting original shards.
- Materialize the NPU attention padding mask as `[B,1,Q,K]` for this runtime;
  validate masked and unmasked output and gradients. Preserve CPU/CUDA broadcast
  behavior. Build RoPE frequencies in CPU float64 and transfer the real cache.
- Keep parameters FP32, use BF16 forward autocast and FP32 loss/validation
  accumulators. Avoid unsupported NPU float64 statistics allocations.
- Apply NPU-specific non-foreach AdamW/clipping and per-tensor EMA, including after
  checkpoint reload. Preserve CPU/CUDA dispatch and original loss/gradient paths.
- Test policy + trunk Ψ, direct-GT Ψ + policy, and Ψ-only modes separately.
  GT-conditioned Ψ loss does not train the policy through GT actions; the
  predicted-action gradient path is a separate contract.
- Use HCCL with actual local-rank devices and verify collectives before real DDP.
  Cached data loaders should not inherit CUDA pin-memory assumptions.

## Validation and limits

Single-card policy + trunk Ψ completed two updates, validation, save and strict
reload. Two- and eight-card HCCL runs also resumed to step 3 and strictly reloaded
on every rank. Direct-GT and Ψ-only variants passed single-card update, validation,
save, strict reload and resume. Component attention maximum absolute error versus
CPU FP32 was below 0.00085 in the tested cases.

Resume restored optimizer/scheduler/step; upstream did not save RNG or sampler
position, so this is not uninterrupted-sequence or bitwise replay. The smoke
validation split was in-sample. No online encoder, L/H/G policy size, multi-node
run, robot rollout, long training or CUDA hardware regression was established.
