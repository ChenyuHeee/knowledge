---
date: 2026-06-07
tags: [扩散模型, 视频生成, Attention, Token重要性]
---

# 定位关键 Token 的方法

> 相关笔记：[高速运动视频 Attention](motion-video-attention.md) | [Softmax](../transformer/softmax.md)

## 问题

在视频扩散中，如何识别哪些 token 是「关键 token」（高速运动、去噪困难、需要更多资源）？

## 四种方法

### 1. 光流运动估计

```
去噪中间帧 → 光流估计 → 运动幅度 → 按 patch 聚合 → 标记高速 token
```

直观但光流有额外开销，且运动大不一定等于需要多步数。

### 2. Attention 熵（推荐，零开销）

```python
entropy_i = -Σ attention[i, j] × log(attention[i, j])
```

| 熵 | 含义 |
|------|------|
| 低 | 注意力集中，已找到对应关系（静态背景） |
| **高** | 注意力分散，在努力找但被稀释了（**关键 token**） |

直接复用 Attention 矩阵，**零额外计算**。

### 3. Token Loss 差异化

计算每个 patch 的 denoising loss（而非全域平均）：

```
Token loss 高 → 去噪不充分 → 关键 token → 多分配资源
Token loss 低 → 已收敛 → 可跳过
```

信号来自训练目标本身，不需要外部模型。

### 4. 跨帧 Attention 匹配度

```
帧 t token[i] 对帧 t+1 所有 token 做 Attention:
  静态：最高权重在同一空间位置附近
  高速：最高权重远离同位置，且峰值低
```

不需要完整光流，复用跨帧 Attention 中间结果。

## 对比

| 方法 | 额外开销 | 准确度 | 可微分 |
|------|---------|--------|--------|
| 光流运动 | 中等 | 高 | 否 |
| **Attention 熵** | **零** | 中高 | **是** |
| Token Loss | 低 | 高 | 是 |
| 跨帧 Attention | 低 | 高 | 是 |

## 工程方案

```
1. Attention 熵实时打分（零开销）
2. 物理场景叠加光流验证
3. Token loss 在验证集评估整体方案
4. 关键 token → 1.5× attention + 1.5× steps
```
