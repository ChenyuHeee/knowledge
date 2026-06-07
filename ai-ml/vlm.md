---
date: 2026-06-07
tags: [LLM, 多模态, VLM, Transformer]
---

# VLM（Vision-Language Model，视觉语言模型）

> 相关笔记：[Transformer & Attention](transformer/transformer-attention.md) | [Softmax](transformer/softmax.md) | [KV Cache 压缩趋势](transformer/kv-cache-trend.md) | [高速运动视频 Attention](diffusion/motion-video-attention.md) | [感知机](transformer/perceptron.md)

## 是什么

能看图的 LLM。GPT-4V、Claude Vision、LLaVA、Qwen-VL。

## 架构

```
图片 → Vision Encoder (ViT) → Patch Embeddings [N, d_vision]
         ↓
      Projection Layer（投影对齐）
         ↓
      Visual Tokens [N, d_model]
         ↓
      [Visual Tokens | Text Tokens] → 标准 Transformer → 输出
```

**Transformer 本身不改**，只加 Vision Encoder + 投影层。

## 训练

| 阶段 | 目标 | 数据 |
|------|------|------|
| 预训练 | 对齐视觉和语言空间 | 几亿图文对 |
| 微调 | 指令跟随（看图回答） | 几十万~百万图文对话 |

## 关键设计：分辨率

| Encoder | 分辨率 | 特点 |
|------|------|------|
| ViT-L (336px) | 低 | LLaVA-1.5 |
| SigLIP/ConvNeXt | 高 | InternVL2 |
| 动态分辨率 | 自适应 | LLaVA-NeXT, Qwen-VL |

一张 672×672 图 → ~2304 个 visual token。和之前知识直接相关：

## 与 Softmax 稀释的连接

```
527 visual tokens + 50 text tokens = 577 tokens
Softmax 预算: 91% 被 visual tokens 分走
→ 文本 token 之间互相 Attend 的能力被稀释
```

和高速运动视频中「大量无关 token 稀释关键 token」同一问题。

## 推理优化

| 优化 | 说明 |
|------|------|
| Visual Token 压缩 | Token Merging / Perceiver Resampler |
| KV Cache 共享 | Visual token 不变，KV 一次算完缓存 |
| Vision Encoder 加速 | ViT 量化/剪枝 |

## 知识链

```
Transformer → LLM → VLM
Attention/QKV/Softmax/KV Cache → 全部复用
```
