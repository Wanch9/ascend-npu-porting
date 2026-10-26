# Model adaptation cases

Historical experience for seven model projects and one FastWAM derivative.
Start with the row matching the current model or failure; read only that case.
Hardware, versions, shapes, batch sizes and thresholds are experiment contracts,
not defaults or promises of support for another model.

| Model / project | Read for | Recorded result and main boundary |
|---|---|---|
| [MiniMax-H3](minimax-h3.md) | Device routing, packed attention, VAE/offload | Eight-card inference and representative attention backward; not full-training resume |
| [ActionWM](actionwm.md) | Split library/launcher, PAC contracts, frozen DiT + ControlNet | Eight-card development update; exit-code evidence gap remains |
| [FastWAM](fastwam.md) | Real-shape RoPE, framework update timing, node-local shards | September eight-card update; October sixteen-card update and strict inference reload; optimizer resume untested in those runs |
| [π0.5 / OpenPI](pi05.md) | Nested checkpointing, HCCL gradients, fused AdamW, buckets | Four-card baseline update/resume; recommended short-run speedup passed gradient checks; faster candidates failed checks |
| [JEPA-WAM](jepa-wam.md) | Frozen features, padding masks, EMA, Ψ modes | ViT-B one/two/eight-card checks and functional resume; no online encoder or full RNG/sampler replay |
| [OpenWAM-α](openwam.md) | Interface migration, ZeRO-2 hooks, resume diagnostics, scaling | Eight-card updates/restore, saved 30k-run completion, sixteen-card speed tests; bitwise continuation failed |
| [MoPA](mopa.md) | Qwen3-VL processing, BF16 updates, multi-node checkpoint | Sixteen-card real updates/resume; no bitwise NPU continuation comparison or simulation claim |
| [fast_mobile_wam](fast-mobile-wam.md) | Dual ActionDiT, deterministic HCCL, ZeRO-1 checkpoint assembly | Derivative with eight-card update/reload and Human300 resume/save diagnostics; not blanket multi-node resume support |

## Sources and maintenance

Summaries were compiled from saved project reports and validation records on
2026-10-26. [sources.json](sources.json) identifies each source by project,
project-relative path and SHA-256 of the bytes read. Original paths refer to
the source layout before this reorganization. A digest identifies a document;
it does not independently verify an experiment or make the report public.
The source inventory has no dependency on a developer's absolute workspace path.

Keep reusable decisions and explicit failed/untested gates in each case.
Original code, raw logs, checkpoints and datasets remain in their projects.
A short update, inference-weight reload, functional full-state resume, numerical
continuation and task-quality evaluation are separate results. When later work
adds evidence, record its date and configuration rather than rewriting an older
failure as a success.
