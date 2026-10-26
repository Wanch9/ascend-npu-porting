# 昇腾 NPU 适配

[English](README.md) · **简体中文**

## 项目概览

这是一套不依赖特定智能体、以代码和实际执行为核心的工作流，用于将 PyTorch 项目适配到华为昇腾 NPU 并验证结果。无需 Codex：GLM 所在宿主、其他编码智能体和人工操作者都可以使用其中的 Markdown 指南及 Python 标准库工具。

工作流涵盖源码兼容性、真实模型及训练和服务验收、HCCL/DDP、checkpoint 保存与恢复证据、离线交接，以及可选的适配后性能分析与优化。同时包含通过 hash 校验保护的跨依赖库补丁交付，以及不依赖特定调度器的训练作业生命周期契约。工作流包含本地资产可用性约定，不包含 OBS 和数据传输操作。

## 快速开始

```bash
git clone https://github.com/Wanch9/ascend-npu-porting.git
cd ascend-npu-porting
```

从下面选择一套技能包，并按其入口指南执行。英文包和中文包是两个使用入口，不要把两套说明当成需要重复执行的检查清单。

## 选择技能包

### 英文技能包

- 在新电脑上从 [PORTABLE_AGENT_GUIDE.md](PORTABLE_AGENT_GUIDE.md) 开始。
- 将 [SKILL.md](SKILL.md) 交给智能体，作为执行决策指南。
- 使用 GLM 时，结合 [references/glm-agent.md](references/glm-agent.md) 核实宿主的实际能力和任务持久化行为。
- 使用复制的技能包前，在仓库根目录执行 `python3 scripts/self_check.py`。它只检查离线工具，不证明 NPU 或模型可用。

### 中文技能包

先读 **[中文技能包的 README](skills/ascend-npu-porting-zh/README.md)**，再按引导读取 [SKILL.md](skills/ascend-npu-porting-zh/SKILL.md)。指南包含可直接交给 GLM 的启动提示、输入清单、快速训练模式和断线续接方法；整目录可复制到任何具备文件、Shell 和目标机访问能力的编码宿主。

选择中文包时只需复制 `skills/ascend-npu-porting-zh/`，它不依赖根目录的英文包。中文包优先复用已有资产，按用户要求选择资产 hash 校验范围，并在训练框架下测量真实参数更新；已有经验集中在[模型案例索引](skills/ascend-npu-porting-zh/references/models/README.md)。

给 GLM 的最短提示：

```text
请读取当前仓库 skills/ascend-npu-porting-zh/README.md，并依照其中的
SKILL.md 和启动模板，为我指定的模型执行昇腾 NPU 适配。
```

## 工具、参考与交付范围

- 远程 Bash 执行使用仓库附带的 SSH 工具，详见 [references/ssh-execution.md](references/ssh-execution.md)。
- GLM 独立执行验证参考 [references/agent-validation.md](references/agent-validation.md)，包含记录开发运行、全新重放，以及迁移到第二个模型的测试。
- 中文技能包的工具和参考资料入口见其 [README](skills/ascend-npu-porting-zh/README.md)。

本仓库只有文档和通用工具，没有模型权重、数据集、环境或具体模型源码。公开交付内容包括指南、小型源码补丁、项目维护的覆盖文件、启动与配置文件、测试和 hash 清单；不包含完整依赖树、加速器运行时、虚拟环境、权重、数据集或缓存。跨已安装依赖库的修改如何重建，详见 [references/dependency-patch-delivery.md](references/dependency-patch-delivery.md)。

## 已有适配实践

我们已开展以下模型及衍生项目的昇腾 NPU 适配：

具体经验、来源和验证边界集中在[中文模型案例目录](skills/ascend-npu-porting-zh/references/models/README.md)，每个模型／项目一份文档；英文版位于 [references/models/](references/models/README.md)。

- [MiniMax-H3](skills/ascend-npu-porting-zh/references/models/minimax-h3.md)：八卡 TP/SP 推理，以及 attention 前向与反向验证。
- [ActionWM](skills/ascend-npu-porting-zh/references/models/actionwm.md)：冻结 MiniMax-H3 视频 DiT，完成八卡 ControlNet 单次训练更新。
- [FastWAM](skills/ascend-npu-porting-zh/references/models/fastwam.md)：八卡及后续十六卡 LIBERO 真实更新；后续实验还通过训练权重严格重载与有界推理。
- [π0.5](skills/ascend-npu-porting-zh/references/models/pi05.md)：四卡完整模型训练、基线路径保存与恢复、有限范围的动作推理，以及训练加速实验。
- [JEPA-WAM](skills/ascend-npu-porting-zh/references/models/jepa-wam.md)：基于冻结特征的 ViT-B 策略与未来预测器训练、验证、checkpoint 重载和续训，覆盖单卡、双卡及八卡检查；本次适配不包含在线 V-JEPA 编码。
- [OpenWAM-α](skills/ascend-npu-porting-zh/references/models/openwam.md)：八卡 MS-HAB 训练、checkpoint 保存与恢复，以及双节点十六卡吞吐实验；逐位续训一致性检查未通过。
- [MoPA](skills/ascend-npu-porting-zh/references/models/mopa.md)：基于 Qwen3-VL-4B 骨干与 MoPA 动作头的 MS-HAB 全参训练，完成双节点十六卡真实更新与 checkpoint 续训；仿真评测暂缓。
- [fast_mobile_wam](skills/ascend-npu-porting-zh/references/models/fast-mobile-wam.md)（FastWAM 衍生项目）：双完整 ActionDiT 更新与重载、Human300 HCCL 续训差异诊断及多节点 checkpoint 汇集经验。

各项结论限于对应项目的实测配置与验证范围。经验摘要和来源文档哈希已纳入本仓库；原始代码、日志及运行产物仍保存在各自项目中。
