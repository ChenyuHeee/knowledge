# KV Head：Key-Value 注意力头

> 相关笔记：[Transformer & Attention](transformer-attention.md) | [大模型拓扑结构](llm-topology.md) | [MLA](mla.md)

## 背景

推理时每生成一个 token，要把历史所有 token 的 K/V 缓存起来避免重复计算，这就是 KV Cache。

```
每层 KV Cache = 2 × batch × num_KV_heads × seq_len × d_head × 2 bytes
```

序列越长，显存越大。KV Cache 是推理显存的第一大开销。

## 三种配置

### MHA（Multi-Head Attention）

Q 头 = K 头 = V 头。每个 Q 有专属 K/V。KV Cache 最大。原始 Transformer、LLaMA-1 使用。

### MQA（Multi-Query Attention）

所有 Q 共享同一套 K/V。KV Cache 缩至 1/h，但精度明显下降。

### GQA（Grouped Query Attention）

Q 头分组共享 K/V。例：8 个 Q、4 组 → 每 2 个 Q 共享一对 K/V。

| 方案 | Q 头 | K/V 头 | KV Cache | 精度 |
|------|------|--------|----------|------|
| MHA | 8 | 8 | 100% | 最高 |
| GQA | 8 | 4 | 50% | 几乎无损 |
| MQA | 8 | 1 | 12.5% | 明显下降 |

## 实际模型

| 模型 | Q 头 | KV 头 | 类型 |
|------|------|-------|------|
| LLaMA-1 | 64 | 64 | MHA |
| LLaMA-2/3 70B | 64 | 8 | GQA |
| Mistral 7B | 32 | 8 | GQA |

**KV 头越少 → KV Cache 越小 → 推理越省显存**。GQA 是当前主流——显著省显存，精度几乎不降。
