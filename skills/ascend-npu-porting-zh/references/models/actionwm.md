# ActionWM：PAC ControlNet 与组合源码

## 范围与来源

来源：工具包原 `references/glm-agent.md` 的组合源码审计，以及中文包 `references/validation.md` 的开发轮结果；快照摘要见 [sources.json](sources.json)。这是历史资料，不承诺外部仓库仍公开，也不是新任务默认 checkout 配方。

审计时 `diffsynth-pac-controlnet` revision `8998f3746c51637feaef2f490765773d17cd8fdc` 是修改后的 DiffSynth 库；`minimax-h3-adaln` revision `139aaf26acb0669edde43e2bfcd3a0535933fe96` 才提供 launcher、数据准备、时间嵌入表生成和评测。后续版本需重新追踪实际源码图。

## 可复用经验

- 库含 NPU helper 不等于实际训练路径已经可用。冻结完整库/launcher/配置/生成资产的组合 revision，并验证组合入口。
- H20 推荐的 bitsandbytes NF4 主模型路径按 CUDA 专属候选处理，先实测 NPU 加载与前向。不能静默将约 30B 主干反量化，再假定显存足够。
- 按真实设备路由可见卡、Accelerate/HCCL、FlashAttention/xFormers 和 offload 同步；保留原 CUDA 路径。
- 保留 PAC 尾部对齐控制、关键帧、时间嵌入表、冻结 DiT、可训练 ControlNet、静音音频及 checkpoint 前缀契约；旧版静音音频约定不能推广到所有新模型。
- 用真实约 3.402B 可训练 ControlNet 测量参数、梯度、optimizer、激活和重计算内存，不能用 tiny import 替代。
- 数据缓存生成、更新、严格重载、恢复及分布式分别验收。库补丁交付和作业提交也有各自范围，准备完成不等于训练完成。

## 开发轮结果与未证实项

开发轮采用完整 MiniMax-H3 视频 DiT 冻结、发布版 ControlNet 训练、真实缓存，在单节点八 NPU 完成一次更新；使用 BF16、CPU 模型/optimizer offload 和显式 HCCL 平均。

SSH 中断留下训练退出码缺口，不能写成全流程退出 0。该轮没有证明全参微调、长期收敛、新进程恢复、任意梯度累积数或其他模型；源码审计中的建议检查也不能当作全部已通过。

相关历史算子与推理经验见 [MiniMax-H3](minimax-h3.md)。
