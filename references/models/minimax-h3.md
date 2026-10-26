# MiniMax-H3: historical inference and attention lessons

## Scope and evidence

This case summarizes historical Ascend 910B1, CANN 8.2.RC1 and
PyTorch/torch_npu 2.6.0 records. The main inference path used a pinned SGLang,
full FL2VA weights and eight-card TP/SP. These observations are not proof that
ActionWM or another model passes its own training gates.

Source: the toolkit's former `references/minimax-h3-lessons.md`; its compilation
snapshot is identified in [sources.json](sources.json). Model patches and raw
server reports remain in the original project.

## Decisions worth reusing

- Select CUDA JIT/Triton/C++ paths by the real tensor device. A compatibility
  layer can make an NPU tensor report `.is_cuda == True`; a CUDA_HOME error is
  not a reason to install CUDA or fake that variable.
- Test attention by API, layout, head count, dtype and segment boundaries.
  Historical fused inference rejected TND with 7/14 heads and D=128; per-segment
  BNSD fallback passed nine BF16 comparisons including noncontiguous QKV,
  isolation and padding. A different TND combination (8 heads, D=192) worked.
  This is not a blanket statement that NPU cannot execute TND.
- Training used `torch_npu.npu_fusion_attention` for BF16 TND
  `[2368,7,128]`, comparing output and dq/dk/dv to CPU FP32. Packed bounds
  `[0,2327,2368]` became cumulative ends `[2327,2368]`; changing padding V
  left real-segment output unchanged. New shapes need their own tests.
- Freezing backbone parameters does not permit `no_grad()` when gradients must
  pass through the backbone to train an upstream module. Inference collectives
  may lack the autograd behavior needed by a training path.
- Check first-frame VAE encoding and full video decoding separately from DiT.
  Offloading weights does not eliminate decoder or backward working-set peaks.
- Register torch_npu before framework device discovery; validate cold-process
  imports and avoid cycles from low-level helpers importing platform packages.
- Validate actual model registration, shard inventory, strict loading and device
  residency. A local directory name once selected the wrong generic pipeline.
- Preserve cache provenance and audio/video layout. Missing, silent and absent
  audio tokens are different contracts. Check generated floating frames before
  encoding; a codec failure can be repaired without repeating denoising.

## Limits

Eight-card inference and representative attention backward do not establish
single-card full-model training, optimizer updates, strict reload or resume.
A newer DiffSynth/ControlNet path needs its own memory and graph analysis.
Record authorized reuse of these lessons; do not describe a replay that consumed
them as independent of prior model-family knowledge.

See [ActionWM](actionwm.md) for a related, separately scoped adaptation.
