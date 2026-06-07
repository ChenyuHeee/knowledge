---
date: 2026-06-05
tags: [深度学习, KV Cache, 趋势]
---
# KV Cache 压缩趋势：MLA 之后

> 相关笔记：[KV Head](kv-head.md) | [MLA](mla.md) | [Transformer & Attention](transformer-attention.md) | [RoPE](rope.md) | [Speculative Decoding](speculative-decoding.md) | [流式视频](../diffusion/streaming-video.md) | [Time Chunk](../diffusion/time-chunk.md) | [Draft/Retrieve](draft-retrieve.md)

## 演进回顾

```
MHA → MQA → GQA → MLA → ?
共享结构 ─────→ 低秩压缩 ─────→ 多维度联合压缩
```

## 当前核心发现

### 1. 非对称处理 K 和 V

V 极其耐压（2~4 bit 量化几乎无损失），K 非常敏感。
TurboQuant (2026)：K 用 8-bit + V 用 4-bit 的非对称方案最优，vLLM/llama.cpp 已集成。

### 2. Token 级剪枝

不是所有 token 的 KV 都值得缓存。

- **FastKV**：中间层后只保留重要 token → 1.82× prefill 加速 + 2.87× decode 加速
- **KVzap (NVIDIA, 2026)**：输入自适应，prefill/decode 双阶段剪枝，2~4× 压缩几乎无损失
- **DynSplit-KV**：按语义边界切分，2.2× vs FlashAttention
- 核心发现：token 价值随层数变化，浅层全重要，深层只留少数关键 token

### 3. RoPE 是低秩压缩的隐形杀手

RAP (2026)：RoPE 的旋转结构迫使 K/V 恢复完整维度。解法：剪掉整对 RoPE 对齐的列，保持 2×2 旋转结构不被破坏。

### 4. MLA 普惠化

MHA2MLA (2025)：用 0.3%~0.6% 数据微调即可将已有 MHA/GQA 模型转为 MLA，KV Cache -92%，LongBench -0.5%。

## 多维度联合压缩

```
维度压缩:    MLA, MatryoshkaKV（低秩投影）
Token 剪枝:  FastKV, KVzap, DynSplit-KV
精度压缩:    非对称量化 (K 8-bit + V 4-bit)
层级复用:    跨层共享 KV
RoPE 适配:   RAP（旋转感知剪枝）
```

未来推理引擎将同时叠加这些维度，每个维度独立工作，组合效果可达数量级提升。

## 趋势总结

从「一刀切压缩每个 token 的维度」→「选择性保留重要 token + 差异化压缩 K/V + 精度量化」的语义+精度联合优化。
