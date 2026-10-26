# JEPA-WAM：冻结特征训练

## 范围与来源

来源：项目 `JEPA-WAM-NPU-publish/NPU_PORTING.md`，实测日期 2026-09-09；文档快照摘要见 [sources.json](sources.json)。上游分支 `vit-online-jepa-lora`，基线 `5aaa85777672dc1cea361062689a238cf552157a`。

本次训练完整 WAM ViT-B 策略及可选未来预测器 Ψ，输入为已有 V-JEPA/UMT5 特征，不实例化或微调编码器。实测契约为双相机、每相机 256 token、视觉通道 1664、文本通道 4096、state8/action7；单节点 910B1、torch/torch_npu 2.6.0、CANN 8.2.RC1。

## 可复用经验

- 将缓存特征训练入口与在线视觉/文本入口分开追踪。保留编码器来源与归一化协议；只有 mean/std 的旧缓存不满足 min/max 契约。基于指定训练集生成新的统计，不改原始 shard。
- 本环境的 NPU attention 将 padding mask 显式展开为 `[B,1,Q,K]`，验证有无 mask 的输出与梯度；CPU/CUDA 保留原广播行为。RoPE 频率在 CPU float64 构造，完成后传输实数缓存。
- 参数保留 FP32，前向 BF16 autocast，loss 和验证统计使用 FP32，避免 NPU float64 累加器。
- AdamW/裁剪在 NPU 使用 non-foreach，EMA 逐张量执行；checkpoint 重载后重新应用 NPU 优化器标志，保留 CPU/CUDA 路径和原损失、梯度语义。
- 策略 + trunk Ψ、direct-GT Ψ + 策略、Ψ-only 分别验收。GT 动作条件下的 Ψ loss 不会经 GT 动作训练策略；预测动作梯度是另一条路径。
- HCCL 使用实际 local rank 设备，先验证 collective 再跑真实 DDP；缓存数据加载不沿用 CUDA pin-memory 假设。

## 验证与边界

单卡策略 + trunk Ψ 完成两次更新、验证、保存和严格重载；双卡、八卡 HCCL 还完成续训至 step3 及各 rank 严格重载。direct-GT 和 Ψ-only 在单卡通过更新、验证、保存、重载及续训。测试形状下 attention 对 CPU FP32 的最大绝对误差低于 0.00085。

恢复涵盖 optimizer/scheduler/step，但上游未保存 RNG 与 sampler 位置，不能称为不中断数据序列或逐位复现。验证使用训练集内 smoke 划分。未证明在线编码器、L/H/G 策略、多节点、机器人闭环、长期训练或 CUDA 实机回归。
