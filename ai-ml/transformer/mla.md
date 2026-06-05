---
date: 2026-06-05
tags: [深度学习, MLA, KV Cache, DeepSeek]
---
# MLA（Multi-head Latent Attention，多头潜在注意力）

> 相关笔记：[KV Head](kv-head.md) | [Transformer & Attention](transformer-attention.md) | [KV Cache 压缩趋势](kv-cache-trend.md)

DeepSeek-V2 (2024) 提出，通过低秩压缩进一步缩减 KV Cache。

## GQA 的局限

GQA 共享 KV 头，但每个头的维度仍是 d_head（如 128 维）。MLA 进一步压缩维度本身。

## 核心思路：低秩 KV 压缩

```
传统:  x → W_K → K（完整维度）  → 缓存 K
       x → W_V → V（完整维度）  → 缓存 V

MLA:   x → W_down → c（低维瓶颈, latent）→ 缓存只存 c！
                     ↓
                W_up_K → K
                W_up_V → V
```

K 和 V 共享同一个压缩向量 c，推理时只缓存 c，用时各自解压。

## 数学

$$c = x \cdot W_{down}$$
$$K = c \cdot W_{up\_K}$$
$$V = c \cdot W_{up\_V}$$

DeepSeek-V2：c ≈ 512 维，原始 K+V ≈ 10240 维 → KV Cache 减少约 20 倍。

## 为什么叫 Latent

K 和 V 不直接存储，而是从潜在表示 c 中「解码」出来，类似 VAE 的 latent → decoder 结构。

## 代价

- 每次 Attention 需做 up-projection 解压 c（开销可控，c 本身很小）
- 训练需额外训练 down/up 投影矩阵
- 极端压缩下的精度边界仍需探索

## 演进路线

```
MHA → MQA → GQA → MLA → ?
每个 Q 专属 K/V → 全部共享 → 分组共享 → 低秩压缩
```
