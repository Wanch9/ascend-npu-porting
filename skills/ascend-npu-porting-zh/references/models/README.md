# 模型适配案例

汇总七个模型项目和一个 FastWAM 衍生项目的历史经验。先按当前模型或问题选择一行，只读取相关案例；硬件、版本、形状、batch 和阈值属于各实验契约，不是新模型默认配置或兼容性承诺。

| 模型 / 项目 | 适合查阅的问题 | 已有结果及主要边界 |
|---|---|---|
| [MiniMax-H3](minimax-h3.md) | 设备路由、packed attention、VAE/offload | 八卡推理与代表形状反向；不等同完整训练恢复 |
| [ActionWM](actionwm.md) | 库/launcher 分离、PAC 契约、冻结 DiT + ControlNet | 八卡开发轮更新；退出码证据缺口仍保留 |
| [FastWAM](fastwam.md) | 真实 RoPE、框架更新时点、节点本地分片 | 九月八卡更新；十月十六卡更新与严格重载推理；对应实验未测 optimizer 恢复 |
| [π0.5 / OpenPI](pi05.md) | 嵌套重计算、HCCL 梯度、融合 AdamW、通信桶 | 四卡基线更新/恢复；推荐短测加速通过梯度检查，更快候选未通过检查 |
| [JEPA-WAM](jepa-wam.md) | 冻结特征、padding mask、EMA、Ψ 模式 | ViT-B 单/双/八卡验证与功能续训；无在线编码或完整 RNG/sampler 重放 |
| [OpenWAM-α](openwam.md) | 接口迁移、ZeRO-2 钩子、恢复诊断、多节点扩展 | 八卡更新/恢复、已保存的三万步完成记录、十六卡测速；逐位续训失败 |
| [MoPA](mopa.md) | Qwen3-VL 预处理、BF16 更新、多节点 checkpoint | 十六卡真实更新/恢复；未做 NPU 逐位续训对照或仿真效果声明 |
| [fast_mobile_wam](fast-mobile-wam.md) | 双 ActionDiT、确定性 HCCL、ZeRO-1 分片汇集 | FastWAM 衍生项目，含八卡更新/重载及 Human300 恢复/保存诊断；非通用多节点恢复保证 |

## 来源与维护

2026-10-26 根据已有项目报告与验证记录整理。[sources.json](sources.json) 记录每份来源的项目名、项目内相对路径和读取内容的 SHA-256。原路径指本次整理前的来源布局；摘要只标识文档，不替代实验复验，也不表示该报告已公开。来源清单不依赖开发电脑的绝对路径。

各案例保留可复用决策及明确的失败/未测项。原始代码、日志、checkpoint 和数据集留在各自项目。短程更新、推理权重重载、完整状态功能恢复、数值连续性和任务效果分别记录；后续新增证据注明日期与配置，不把旧实验失败改写成通过。
