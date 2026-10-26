# Ascend NPU Porting

**English** · [简体中文](README.zh-CN.md)

## Overview

An agent-independent, code-first workflow for adapting and validating PyTorch
projects on Huawei Ascend NPU. Codex is not required: GLM-hosted coding agents,
other coding agents, and human operators can use the Markdown instructions and
Python standard-library helpers.

The workflow covers source compatibility, real model/training/serving gates,
HCCL/DDP, checkpoint and resume evidence, offline handoff, and optional
post-port profiling and performance tuning. It also covers hash-guarded
multi-library patch delivery and scheduler-independent training-job lifecycle
contracts. It includes local artifact availability contracts but deliberately
excludes OBS and data-transfer operations.

## Quick start

```bash
git clone https://github.com/Wanch9/ascend-npu-porting.git
cd ascend-npu-porting
```

Choose one toolkit below and follow its entry guide. The English and Chinese
toolkits are separate entry points, not checklists to execute twice.

## Choose a toolkit

### English toolkit

- Start on a new computer with [PORTABLE_AGENT_GUIDE.md](PORTABLE_AGENT_GUIDE.md).
- Give an agent [SKILL.md](SKILL.md) as the authoritative decision guide.
- For GLM, also use [references/glm-agent.md](references/glm-agent.md) to verify
  the host's actual capabilities and persistence behavior.
- Verify a copied toolkit with `python3 scripts/self_check.py` from the repository
  root before use. This checks the offline tools, not NPU or model readiness.

### Chinese toolkit

Start with the [Chinese toolkit README](skills/ascend-npu-porting-zh/README.md),
then follow its instructions to read [SKILL.md](skills/ascend-npu-porting-zh/SKILL.md).
The guide includes a GLM startup prompt, required inputs, fast training mode,
and instructions for resuming interrupted work. The toolkit can be copied to
any coding-agent host with file, shell, and target-machine access.

Copy only `skills/ascend-npu-porting-zh/` when choosing the Chinese toolkit;
it does not depend on the English toolkit at the repository root. Its workflow
prioritizes existing assets, applies asset hashing according to user requirements,
and measures real parameter updates within the training framework. It includes
the model cases listed in its [case index](skills/ascend-npu-porting-zh/references/models/README.md).

A minimal prompt for GLM (the Chinese version is in the
[Chinese homepage](README.zh-CN.md)):

```text
Read skills/ascend-npu-porting-zh/README.md in this repository, then follow
its SKILL.md and startup template to port my specified model to Ascend NPU.
```

## π0.5 case study and validation limits

- Ported project: [Wanch9/pi05-npu](https://github.com/Wanch9/pi05-npu).
- [English case study](references/models/pi05.md) / [Chinese case study](skills/ascend-npu-porting-zh/references/models/pi05.md):
  real full-model updates and resume, HCCL gradient discrepancy diagnosis,
  recomputation optimization, fused AdamW, and communication bucket experiments.
- In a short benchmark, the recommended configuration reduced step time from
  **1.3038 to 1.1715 seconds (−10.15%, throughput +11.29%)**, passing comparison
  across all valid gradients. Faster fused AdamW and larger buckets remain
  experimental options because they did not pass the established numerical thresholds.
- The case used 4 × 910B1 and 8 LIBERO cache windows. Long-term convergence and
  task success rates were not validated; checkpoint resume on the accelerated path
  was not separately validated either. Case parameters are not defaults for new models.
- The π0.5 project has integrated the recommended strategy into its regular
  `scripts/train_pytorch.py` entry point, explicitly enabled with
  `--pytorch-npu-training-optimization`. The flag is off by default; fused
  optimizers and larger buckets are still not defaults.

## Tools, references, and delivery scope

- For remote Bash, use the bundled SSH helper described in
  [references/ssh-execution.md](references/ssh-execution.md).
- To test independent execution by GLM, use
  [references/agent-validation.md](references/agent-validation.md) for a recorded
  development run, a fresh replay, and a second-model transfer test.
- For Chinese toolkit tools and references, follow its
  [README](skills/ascend-npu-porting-zh/README.md).

This repository contains documentation and general-purpose tools, without model
weights, datasets, environments, or specific model source code. Public delivery
contains instructions, small source patches, project-owned overlays, launch/config
files, tests, and hash manifests—not complete dependency trees, accelerator runtimes,
virtual environments, weights, datasets, or caches. See
[references/dependency-patch-delivery.md](references/dependency-patch-delivery.md)
for reconstructing changes that span installed libraries.

## Models adapted in practice

Our Ascend NPU adaptation work includes the following models and derived projects:

Detailed lessons, sources and validation boundaries are collected in
[references/models/](references/models/README.md), with one file per model/project.

- [MiniMax-H3](references/models/minimax-h3.md): eight-card TP/SP inference and attention forward/backward validation.
- [ActionWM](references/models/actionwm.md): one eight-card ControlNet training update with a frozen MiniMax-H3 video DiT; the recorded run has an exit-code evidence gap.
- [FastWAM](references/models/fastwam.md): eight-card and later sixteen-card real LIBERO updates; the later run also passed strict trained-weight reload and bounded inference.
- [π0.5](references/models/pi05.md): four-card full-model training, baseline save/resume, bounded action inference, and training acceleration experiments.
- [JEPA-WAM](references/models/jepa-wam.md): frozen-feature ViT-B policy and future-predictor training, validation, checkpoint reload and resume, including one-, two- and eight-card checks; online V-JEPA encoding was outside this port.
- [OpenWAM-α](references/models/openwam.md): eight-card MS-HAB training, checkpoint save/restore and two-node, sixteen-card throughput experiments; bitwise continuation equivalence did not pass.
- [MoPA](references/models/mopa.md): full-parameter training of the Qwen3-VL-4B backbone and MoPA action head on MS-HAB, with two-node, sixteen-card real updates and checkpoint resume; simulation evaluation was deferred.
- [fast_mobile_wam](references/models/fast-mobile-wam.md) (derived from FastWAM): dual ActionDiT update/reload, plus Human300 HCCL resume diagnosis and multi-node checkpoint assembly lessons.

Validation scope is specific to each project and configuration. Case summaries
and source-document hashes are included here; original code, raw logs and runtime
artifacts remain in their respective projects.

Originally based on [Wancha0/ascend-npu-porting](https://github.com/Wancha0/ascend-npu-porting), with further adaptation lessons maintained here.
