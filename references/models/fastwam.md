# FastWAM: real updates, RoPE and reload

## Scope and sources

Two distinct historical experiments used official FastWAM revision
`7faa71108368fbb3b6885649f112af607427a2d4`, real LIBERO data and the full
video/action experts with VAE frozen. Both used torch/torch_npu 2.6.0,
CANN 8.2.RC1, Accelerate 1.12.0 and DeepSpeed 0.18.5.

Sources: the toolkit's September FastWAM lessons and project
`FastWAM-NPU/NPU_PORTING.md` for 2026-10-04. Document hashes are in
[sources.json](sources.json). The later run does not retroactively validate the
earlier skill revision or establish an independent GLM replay.

## Reusable decisions

- Inventory existing assets before downloading; apply the user's hash scope.
  A missing project import calls for checking the actual interpreter and working
  directory, not replacing the platform's torch installation.
- Exercise real RoPE construction, concatenation, transfer, forward and backward.
  A multiply-only probe missed a complex128 concatenation failure. The October
  port retained CPU/CUDA complex FP64 behavior, constructed caches on CPU and
  used FP32 real rotations on NPU. Keep measured dtype-specific tolerances.
- Construct unsupported time embeddings with the original CPU FP64 formula and
  cast at the original caller boundary. Use eager VAE on NPU while retaining the
  CUDA Graph path; preserve SDPA mask semantics.
- Measure parameter changes at the framework's actual update boundary. Names
  from module aliases can disappear from deduplicated `named_parameters()`.
  Compare BF16 saved weights with initialization converted by the same loading
  procedure; a missing `.grad` after backward can reflect framework buffering.
- In multi-node runs, distinguish global process count, local rank and node-local
  optimizer shards. Model reload for inference is not optimizer-state resume.
- Check floating decoder outputs before uint8 conversion. Finite media alone is
  weaker evidence than finite model outputs.

## Observed validation

| Experiment | Evidence | Boundary |
|---|---|---|
| 2026-09-08, eight 910B1 cards, global batch 16 | One update, finite rank losses/gradient norms, sampled video weights changed, model/optimizer checkpoint written, launcher exit 0 | Sampled changes only; no inference, strict resume or convergence claim |
| 2026-10-04, two nodes / sixteen 910B1 cards, global batch 32 | One update over 32 unique samples; sampled video/action/proprio parameters changed; both nodes exit 0 | Optimizer shards saved on both nodes; no second update after optimizer restore |
| October fresh-process inference | Strict trained-weight reload; finite actions `[32,7]` and nine-frame joint output; exit 0 | Two denoising steps; no LIBERO success-rate or generation-quality claim |

The October real-shape RoPE and masked-attention component checks passed.
This does not erase the September local probe failures. First-execution timings
(15.17 seconds for actions and 55.90 for joint inference) are not steady-state
performance benchmarks. Case batch sizes and precision choices are not defaults.

For the mobile-manipulation derivative, see [fast_mobile_wam](fast-mobile-wam.md).
