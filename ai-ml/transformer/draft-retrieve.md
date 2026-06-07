---
date: 2026-06-07
tags: [LLM推理, 推测解码, KV Cache, 缓存策略]
---

# Draft/Retrieve 结合策略

> 相关笔记：[Speculative Decoding](speculative-decoding.md) | [KV Cache 压缩趋势](kv-cache-trend.md) | [流式视频](../diffusion/streaming-video.md)

## 两种 Draft 来源

```
传统:  Draft Model 生成候选 → Target Model 验证
混合:  Retrieve (缓存检索) + Draft Model (生成) → 混合候选 → 验证
```

## 为什么加 Retrieve

LLM 推理存在大量重复计算：

- **System Prompt**：每次对话相同的 2000 token
- **多轮对话**：前 9 轮历史 KV Cache 可复用
- **批量请求**：100 个用户问同一问题 → 前半段计算相同

## 工作流程

```
1. 查缓存：当前前缀是否在 KV Cache 库中？
   → 命中：取对应 KV + 后续 token 序列作为 Draft
   → 未命中：走 Draft Model 生成

2. Draft Model 兜底缓存未命中部分

3. 混合 Draft：前半段 Retrieve + 后半段 Draft Model → 验证

4. 验证后更新缓存
```

## 关键设计

### 缓存粒度

| 粒度 | 特点 |
|------|------|
| 粗粒度（整段 prompt） | 命中率低但一次收益大 |
| 细粒度（N-gram） | 命中率高但拼接开销大 |
| 多级缓存 | 全量粗匹配 + 局部细拼接 |

### 淘汰策略

- LRU：最近访问优先保留
- LFU：访问频率优先保留
- 重要性：System Prompt 永不过期

### 一致性

Retrieve 的 KV Cache 必须是确定性的——相同 token 序列产生相同 K/V，不能用随机 dropout 或非确定性 Attention。

## 实际系统

| 系统 | 策略 |
|------|------|
| TriForce | 分层推测：Retrieve 长程（稀疏）+ Draft 补短程（稠密） |
| CacheGen | KV Cache 有损压缩 → Retrieve 解压拼接 |
| Prompt Cache | System Prompt KV 预计算持久化，多请求共享 |
| SGLang RadixAttention | 前缀树组织 KV Cache，自动复用公共前缀 |

## 与其他概念的联系

```
Speculative Decoding: Draft Model → 验证
Draft/Retrieve:      缓存 Draft ← Retrieve + Draft Model → 验证

流式视频对应: 锚点帧 KV（Retrieve）+ 新帧生成（Draft）= 增量计算
```
