# MoPA：Qwen3-VL 双节点训练

## 范围与来源

来源：`mopa-mshab/NPU_PORTING.md`，2026-10-18 文档快照摘要见 [sources.json](sources.json)。组合 XPolicyLab MoPA 核心 `ddf5dcfff5e089a323da1d22891909b72864647e` 与 Qwen3-VL-4B-Instruct revision `ebb281ec70b05090aa6165b016eac8ec08e71b17`。

全部声明参数设为可训练，无 LoRA 或骨干冻结；BF16 骨干、FP32 query/动作头，不使用全局 autocast。两节点分别为 8×910B1 和 8×910B2，torch/torch_npu 2.6.0、CANN 8.2.RC1。MS-HAB 七技能使用 qpos12，action13 去除固定头部通道后为 action11（8+3 分支），不等同论文原始 proprioception。

## 可复用经验

- 额外依赖在独立环境升级（本例 transformers/tokenizers 4.57.0/0.22.2），保留平台 torch 组合。导入通过与完整预训练模型通过分别记录。
- 保留官方 processor：两张 RGB128 实际被放大到 256，并生成原生 grid/token 契约，不能为了显存静默减少视觉 token。
- 分别验证骨干和动作头的真实更新。低学习率下 BF16 可能舍入掉更新；loss/梯度有限、参数数量或 requires-grad 不证明每个张量都更新。
- 追踪计算图尾部：最后 query 输出不再参与 loss，造成 16 个 optimizer slot 无状态；小模型反向支持这一解释。不能为了填满状态而制造梯度。
- 两节点同名本地目录不是共享存储。完整 checkpoint 到达两节点后再恢复，并核对拓扑、归一化、学习率计划、各 rank RNG 和已消费样本位置。
- CPU/NPU 对照使用相同量化输入与 processor；CPU BF16 参考不是 FP32 真值，也不是 CUDA 等价对照。
- HDF5 析构修复只处理解释器退出阶段，不吞掉正常读取错误。

## 证据与边界

完整模型真实更新两步，再从 step2 恢复至 step4；初训及恢复均为两节点退出 0。四步合计 256 个不同样本，loss/梯度有限，参数抽样哈希跨 rank 一致。骨干与 query 探针均有变化，但探针刻意选择非零梯度且绝对值较小的元素，不能推算全模型更新比例。

step4 checkpoint 已跨节点复制并校验大小/SHA-256。功能恢复、学习率和样本进度连续性通过；未做 NPU 不间断四步与恢复四步的逐位相等对照。

单真实样本前向对 CPU BF16 的 query rRMSE 为 1.509%，动作 velocity 两支为 0.522%/0.674%；不代表反向等价或任务效果。报告提到后续十万步作业启动，本页不从“已启动”推断完成。仿真评测暂缓。
