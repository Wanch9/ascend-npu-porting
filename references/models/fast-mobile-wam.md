# fast_mobile_wam: derived architecture and resume diagnosis

## Scope and sources

This is a FastWAM derivative, not another independent upstream base model.
Sources in project `fast_mobile_wam`: its README and the October 5 dual-action,
October 11 Human300 resume-cause, and October 12 two-node validation records.
Exact document hashes and relative paths are in [sources.json](sources.json).

MS-HAB uses state12/action13. The two independent 30-layer, width-1024 ActionDiTs
split upper body `[0:11]` (including torso lift) and base `[11:13]`, sharing
video/text context and a once-computed first-frame video cache. Human300 uses a
separate LeRobot data/robot interface. Do not mix either protocol with legacy HDF5.

## Reusable decisions

- Distinguish full independent branches from merely splitting the final linear
  layer. The dual architecture adds about 1.02B parameters. Old single-expert or
  split-linear complete checkpoints must not silently restore into the new graph.
- Verify branch parameter/storage independence, channel order, attention
  visibility, cache parity and updates in both trunks and all I/O channels.
- Diagnose resume divergence in order: input/RNG, local gradients, collective
  outputs, clipping, then parameter updates. In Human300, first resumed loss and
  local gradients matched while global norms differed; fixed-input and real
  gradient-block tests demonstrated nondeterministic HCCL reductions.
- For that contract, set deterministic collectives before HCCL initialization and
  record the setting in checkpoint identity. Give the DataLoader its own CPU
  generator so iterator creation does not consume model RNG. That RNG fix was
  separate from the demonstrated loss-divergence cause.
- For pure data-parallel ZeRO-1 with DeepSpeed 0.18.5 node-local storage, both local
  rank-zero processes wrote a common model-state file, but only global rank zero
  had the complete shared-parameter alias table. Save common model state on global
  rank zero and retain optimizer/RNG shards for every rank. Do not require the
  original metadata-different files to be byte-identical or choose an arbitrary copy.
- Record full checkpoint commit separately from training updates. A run that
  completed updates but failed checkpoint assembly is still a failed pipeline.

## Validation and limits

The October 5 dual model passed 74 tests and an eight-card real update:
both trunks and all 13 input/output channels changed. A fresh process strictly
reloaded the checkpoint and produced finite `[32,13]` actions on eight tasks
and a nine-frame joint video; process exits were 0. No rollout or convergence
was established by that smoke run.

The October 11 Human300 single-node resume experiment passed its numerical
continuation checks after the communication fix, but retained a Python RNG
final-state discrepancy; do not claim complete bitwise state replay.

The October 12 two-node checkpoint fix produced committed step16/32 states with
16 optimizer shards and 16 NPU RNG files, both nodes exit 0. Before/after
from-scratch short runs matched sampled parameter hashes and loss/gradient norms.
That is regression evidence for the save fix, not a two-node restore test.
A later batch128 short benchmark saved inference weights only. Single-node resume
success does not prove multi-node resume or a different batch's full-state save.
