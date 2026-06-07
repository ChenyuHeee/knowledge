---
date: 2026-06-07
tags: [LLM推理, 推测解码, 推理加速]
---

# Speculative Decoding（推测解码）

> 相关笔记：[KV Head](kv-head.md) | [MLA](mla.md) | [KV Cache 压缩趋势](kv-cache-trend.md)

## 动机

LLM 自回归生成一次一个 token，100 次串行前向。GPU 利用率低（显存带宽瓶颈，非算力瓶颈）。瓶颈是顺序依赖，不是总算力。

## 核心思路

```
Draft:   小模型快速自回归猜 K 个 token
Verify:  大模型一次性并行验证 K 个候选
Accept:  前面连续匹配的接受，第一个不匹配的用大模型自己的替换
```

一次大模型前向产生多个有效 token，而非 1 个。

## 质量保证

Rejection Sampling 保证输出分布和正常自回归一致：

$$P_{spec} = P_{auto}$$

**质量无损，纯加速**。

## Draft Model 方案

| 方案 | 代表 | 说明 |
|------|------|------|
| 独立小模型 | SpecInfer | 训同词表小 LLM |
| 自身层 + 多头 | **Medusa** | 大模型部分层 + 多个预测头同时猜 |
| 无 Draft | **EAGLE** | 从大模型最后一层特征直接预测下一 token |
| 树形推测 | SpecTr | 多分支树，全部并行验证 |

## 加速比

```
加速比 ≈ 1 / (1 - 接受率 × 猜测长度)
```

90% 接受率 × 猜 5 个 → 4~5× 加速。

## 关键：KV Cache 并行

验证阶段 K 个候选 token 一起前向 → KV Cache 一次算 K 个位置 → 并行化 → GPU 利用率高。这正是「快」的本质。

## 在推理优化中的位置

```
单 token 更快:  GQA/MLA, Flash Attention, 量化
一次多 token:  Speculative Decoding, Jacobi Decoding
              ↑ 两类方法正交，可组合使用
```
