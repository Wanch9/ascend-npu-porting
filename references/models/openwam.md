# OpenWAM-α: MS-HAB training and scaling

## Scope and sources

Sources: project `OpenWAM-NPU/NPU_PORTING.md` and the saved
`artifacts/npu_formal/latest-status.json` snapshot; hashes in
[sources.json](sources.json). The report covers October 14–25, 2026.
This summary uses that saved evidence, not a live query of a training service.

Source base `4d7f642b841913ea3c176010b692b1de3d5ef180`; official α checkpoint
revision `52df4e66c82c5c8b480adcc8d01f4db7415dfb56`, step 154000.
The MS-HAB five-task port uses state12/action13, 32 actions and nine video frames.
Only action-input, action-output and state-input interfaces are reinitialized.
VAE/T5 remain frozen; baseline video/action DiTs train with BF16 and ZeRO-2.

## Reusable decisions

- Keep robot action semantics explicit; joint/base controls are not end-effector
  poses. Strictly load all non-replaced keys/shapes and initialize new interfaces
  consistently across ranks. Disable interface reset for continued training.
- Use the α checkpoint's native 512-token T5 contract rather than old FastWAM
  128-token caches. Check video/text precision separately from task success.
- Construct RoPE frequencies on CPU, use validated FP32 real rotation on NPU,
  and keep runtime numerical flags aligned between probes and actual training.
- Save Python/NumPy/CPU/NPU RNG and deterministic sampler progress. Test a
  mid-epoch resume across later epochs, not just a restart at an epoch boundary.
- Audit ZeRO padding before declaring a checkpoint corrupt: a validator initially
  missed three padding values. Separate restored-state equality from equality
  after the next backward/update.
- Profile framework overhead. DeepSpeed 0.18.5 ZeRO-2 hooks repeatedly traversed
  1,651 parameters and autograd nodes. A project-local cache keyed by backward
  graph task, with parameter identity/requires-grad checks, removed repeated work.
  It matched the original path over 26,416 calls in the bounded comparison.
- Measure real data-loader throughput and topology. Parameter freezing with
  joint attention may still require a gradient path through video computation;
  a cross-attention/no-grad variant changes architecture and is a separate
  experiment, not a numerically equivalent speedup.

## Evidence and limits

Eight-card real updates, save and fresh-process restore/update exited 0.
Restored FP32 master weights, Adam moments and step samples matched saved values.
However, bitwise continuation failed: 96/95 of 206,848 sampled parameter values
differed in two resumes, despite matching inputs/losses; maximum branch relative
RMS differences were about 0.0103%/0.0080%. The responsible operator was not
identified; deterministic-HCCL findings from other models do not resolve this case.

The saved formal-run snapshot records 30,000 completed updates and training and
backup exit codes 0. Its inference-only checkpoint policy does not establish
full-state resume for that long run or policy success rates.

On October 25, two nodes (8×910B1 + 8×910B2), global batch 256 measured
3.587 seconds/update. Frozen-Wan joint-attention and cross-attention/no-grad
experiments measured 3.248 and 2.863 seconds/update respectively. The latter adds
about 476 million action-side parameters. Treat these as distinct short-run
contracts, not task-quality-equivalent improvements or default configurations.
