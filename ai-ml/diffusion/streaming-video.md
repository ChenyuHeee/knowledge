---
date: 2026-06-07
tags: [扩散模型, 视频生成, 流式生成, 长序列]
---

> 相关笔记：[高速运动视频 Attention](motion-video-attention.md) | [KV Cache 压缩趋势](../transformer/kv-cache-trend.md) | [Time Chunk](time-chunk.md)

# 流式视频（Streaming Video）

## 传统流式传输

边下边播（HLS/DASH），视频分块传输，多码率自适应。与视频生成关系不大。

## 流式视频生成

### 问题

非流式：一次生成全部帧，Attention over all tokens → 60 秒视频 = 52 万 token → Attention 矩阵 2700 亿元素 → 显存爆炸。

### 做法

```
生成段 1 (0~2s) → 生成段 2 (2~4s) 以段 1 为条件 → 生成段 3 以段 2 为条件 → ...
```

### 核心挑战：长程 Attention 衰减

段 10 和段 1 之间没有直接 Attention 通路。信息随段落拉长而衰减。

### 解决方案

| 方案 | 思路 |
|------|------|
| 滑动窗口 | 每帧只看前 N 帧 |
| 记忆压缩 | 旧帧 KV 压缩成 compact memory token |
| 锚点帧 | 保留关键帧的完整 KV，其余丢弃 |
| 分层 Attention | 局部窗口 dense + 全局稀疏 anchor frame |

## 与 LLM 推理的联系

KV Cache 压缩（LLM） 和 流式视频生成（Video）本质是同一个问题：

长序列的 Attention 状态管理。GQA/MLA/Token 剪枝/KV 量化在视频生成里全部能找到对应——token 变成视频 patch，数学一样。
