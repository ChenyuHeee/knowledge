---
date: 2026-06-07
tags: [扩散模型, 视频生成, Attention, 长序列]
---

# Time Chunk（时间分块）

> 相关笔记：[流式视频](streaming-video.md) | [高速运动视频 Attention](motion-video-attention.md) | [KV Cache 压缩趋势](../transformer/kv-cache-trend.md)

## 问题

128 帧 × 4096 token/帧 = 52 万 token → Full Attention $O(N^2)$ → 显存爆炸。

## 做法

沿时间轴把视频切成小块，每块内 Dense、块间 Sparse。

## 三种策略

### 1. Chunk 内 Full + Chunk 间稀疏

```
[帧 0-7] ←→ [帧 8-15] ←→ [帧 16-23]
  Full      Full         Full
  (3.2万 token, 可承受)

Chunk 间: 只和相邻 Chunk 最后几帧做 Attention
```

最常用方案。Chunk 大小由显存决定。

### 2. Chunk 内 Full + Chunk 间压缩

旧 Chunk 全部帧 → 压缩成 Memory Token → 新 Chunk Attend 到压缩代表。

和 KV Cache 压缩相同：压缩旧块信息，只保留精华。

### 3. 分层 Chunk

```
Level 1: Chunk 内 Full Attention
Level 2: Chunk 间额外 Encoder 编码长程依赖
Level 3: Super-Chunk 间稀疏 Attention
```

类似视频编码 GOP 的分层结构（I/P/B 帧）。

## Chunk 大小选择

| 大小 | 优势 | 劣势 |
|------|------|------|
| 大（64帧） | 内长程依赖好 | 显存大 |
| 小（4帧） | 快 | Chunk 间断裂 |
| 自适应 | 运动剧烈小 Chunk，静态大 Chunk | 实现复杂 |

## 与其他概念的联系

```
流式生成:  一段一段生成
Time Chunk: 一段内部也分块 Attention
Spec Dec:   Token → Draft Chunk → 验证

共同模式: 块内 Dense + 块间 Sparse，对抗 Attention 的 O(N²)
```
