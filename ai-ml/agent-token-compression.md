---
date: 2026-06-15
tags: [LLM, Agent, Token压缩, 上下文管理]
---

# Agent 的 Token 压缩

> 相关笔记：[KV Cache 压缩趋势](transformer/kv-cache-trend.md) | [Draft/Retrieve](transformer/draft-retrieve.md) | [定位关键 Token](../diffusion/key-token-detection.md)

## 问题

Agent 调一次工具回几十 KB，多轮叠加后上下文窗口爆满。与之前聊的 KV Cache 压缩不同——压缩的不是缓存，是**上下文 token 本身**。

## 四种策略

### 1. 结构化摘要

工具输出 → LLM 提炼为 1~3 句摘要：

```
原始: "main.py L12 unused import os, L45 function too long..." (3KB)
摘要: "main.py: unused import(L12), long function(L45)" (50B)
```

压缩 95%+，适合结构化输出、日志、搜索列表。

### 2. 滑动窗口 + 锚点

```
[System Prompt]          ← 永久保留（锚点）
[旧轮次摘要]
[最近 N 轮完整]           ← 窗口内无损
[当前轮]                 ← 全保留
```

MemGPT/Letta 把上下文当 OS 内存管理，核心 vs 归档，按需换入换出。

### 3. Token 级重要性剪枝

打分方式：Attention 权重高 / 去掉后模型输出变化大 / 小模型判断「是否还在用」→ 保留 Top-K%。LLMLingua 系列（Microsoft）。

### 4. 长短期记忆分离

```
Working Memory (在 context): 当前目标 + 最近 3 轮 + 工具输出摘要
Long-term Memory (外挂): 向量数据库存历史经验，embedding 检索
```

## 对比

| 方法 | 压缩率 | 信息损失 | 开销 | 适合 |
|------|------|------|------|------|
| 摘要 | 95%+ | 中 | 额外 LLM | 工具输出 |
| 滑动窗口 | 中 | 低 | 低 | 多轮对话 |
| Token 剪枝 | 50~80% | 中低 | 低 | 长文本 |
| 长短期记忆 | 极高 | 中 | 高 | 复杂 agent |

工程上四者组合：工具输出 → 摘要，对话 → 滑动窗口，全局 → 长期记忆检索。
