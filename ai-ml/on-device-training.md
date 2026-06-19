---
date: 2026-06-18
tags: [LLM, 训练, 端侧部署, LoRA, 量化]
---

# On-device Training（端侧训练）

> 相关笔记：[INT8/FP8/FP16/BF16](../systems/int8-quantization.md) | [Prefill & Decode](transformer/prefill-decode.md)

## 是什么

在设备本地（手机/电脑/IoT）训练模型，不靠云端 GPU。大的预训练在云端完成，设备上只做微调。

## 与云训练的对比

| | 云端 | On-device |
|------|------|-----------|
| 算力 | 千卡 GPU | 手机 NPU（几 TOPS） |
| 内存 | TB 级 HBM | 几 GB 共享 |
| 电力 | 不限 | 电池 |
| 数据 | 上传 | **不出设备** |
| 用法 | 训大模型 | 微调/个性化 |

## 核心矛盾：内存和算力

7B FP32 全量训练 ~100GB 显存。手机 8GB RAM。解法：

| 技术 | 怎么省 | 效果 |
|------|--------|------|
| **LoRA** | 只训 0.1%~1% 参数 | 内存 10~100× |
| 量化训练 | INT8/FP8 | 精度链压低 |
| 梯度检查点 | 不存全部激活 | 时间换空间 |
| 模型剪枝 | 删不重要参数 | 模型变小 |

LoRA 是核心：冻结原权重，只训旁路小矩阵。

## 与已有知识连接

KV Cache 量化 / INT8 / FP8 / Prefill/Decode → 同样在 on-device 场景下用到：更少内存、更低精度、更少计算。
