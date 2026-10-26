# ActionWM: PAC ControlNet adaptation

## Historical source audit

The ActionWM example demonstrates why the complete library/launcher source graph
must be mapped. Treat the following revisions as evidence for that audit,
not as a promise that the external repository remains publicly accessible:

This is a historical audit, not a current checkout recipe. The source revisions
below establish only their own layout and behavior. For newer revisions,
derive the source graph and contracts again; do not require two branches or
silent audio merely because this example used them.

- `diffsynth-pac-controlnet` at audit revision
  `8998f3746c51637feaef2f490765773d17cd8fdc` is the modified DiffSynth library.
  Its README explicitly says data preparation, training launch, and evaluation
  are elsewhere.
- `minimax-h3-adaln` at audit revision
  `139aaf26acb0669edde43e2bfcd3a0535933fe96` contains
  `tools/robotwin_h3_controlnet.sh`, data preparation, the time-embedding-table
  builder, and evaluation tools.
- The library already contains NPU device helpers and optional TorchNPU
  dependencies, but that does not prove this PAC training path.

At those audit revisions, a capable GLM host should build and validate the
composed project, then triage these first-order blockers before broad edits:

1. The recommended H20 path loads a pre-quantized bitsandbytes NF4 host. Treat
   it as CUDA-specific until a representative NPU load and forward proves
   otherwise. Select an NPU-supported precision/memory plan from measurements;
   do not silently dequantize a 30B-class host and hope it fits.
2. The training launcher exports `CUDA_VISIBLE_DEVICES` and uses a GPU-oriented
   Accelerate command. Add target-aware visibility, device selection, and HCCL
   launch configuration while retaining the CUDA branch.
3. Trace optional FlashAttention/xFormers/custom-kernel imports and offload
   synchronization on the executed MiniMax-H3 path. Route by the real device
   type and exercise the NPU fallback at representative video shapes.
4. Preserve the PAC contracts: tail-aligned control rows, keyframe semantics,
   the `MINIMAX_H3_CONTROLNET_T_TABLE` artifact, frozen host DiT, trainable
   ControlNet, silent audio behavior, and checkpoint prefix/reload semantics.
5. Prove memory with the real 3.402B trainable ControlNet, optimizer state,
   gradients, activations, and checkpointing. A tiny import or a different
   DiffSynth NPU example cannot establish trainability.
6. Validate data-cache generation separately from training, then one real
   optimizer update, save/strict reload/resume, two-rank HCCL, and only then the
   intended topology.

If the port changes installed libraries, GLM must use
[dependency patch delivery](../dependency-patch-delivery.md) and leave small hash-guarded deltas rather than a
copy of `site-packages`. If it must create or operate a training job, it must use
[training-job lifecycle](../training-job-lifecycle.md) and distinguish preparing files from authorized
submission.

With repository access but no Ascend target or artifacts, GLM can produce a
reviewed **prepared** patch bundle. With a shell-capable host, the complete
source graph for the pinned revision, local weights/data, and target access,
GLM can attempt and evidence an end-to-end port. This describes prerequisites,
not a measured success rate. Training readiness and independent completion
remain unproven until their respective target and replay gates pass.

## Recorded development result and limits

The development run froze the full MiniMax-H3 video DiT, trained the released
ControlNet with real cached data and completed one update on a single node with
eight NPUs. It used BF16, CPU model/optimizer offload and explicit HCCL averaging.
An SSH interruption left a training exit-code evidence gap; do not report a
fully successful exit. This run did not establish full-parameter fine-tuning,
long-term convergence, fresh-process resume, arbitrary accumulation or another model.
The audit checklist above describes necessary checks, not checks all proven passed.

Sources: the toolkit's former ActionWM section in `references/glm-agent.md` and
Chinese `references/validation.md`; document snapshots are in
[sources.json](sources.json). Related operator evidence is in
[MiniMax-H3](minimax-h3.md).
