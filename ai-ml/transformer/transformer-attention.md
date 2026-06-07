---
date: 2026-06-05
tags: [深度学习, Transformer, Attention]
---
# Transformer & Attention 详解

> 相关笔记：[感知机](perceptron.md) | [FFN / MLP](ffn.md) | [大模型拓扑结构](llm-topology.md) | [残差连接](residual-connection.md) | [KV Head](kv-head.md) | [RoPE](rope.md) | [Softmax](softmax.md)

## Attention：让 token 互相看见

### 直觉

句子「我把香蕉吃了，它坏了」中，「它」需要知道指向「香蕉」。Attention 机制就是让每个词计算和其他所有词的相关度，按相关度聚合信息。

### Self-Attention 计算

```
输入: 每个 token 的 embedding (d_model 维)

Step 1 — 投影:
  Q = x · W_Q    Query:  「我要找什么」
  K = x · W_K    Key:    「我能提供什么」
  V = x · W_V    Value:  「我具体是什么」

Step 2 — 分数:
  score(i,j) = Q_i · K_j^T / √d_k    (√d_k 防梯度爆炸)

Step 3 — 归一化:
  attention_weights = softmax(scores)   (每行和为 1)

Step 4 — 聚合:
  output_i = Σ attention_weight(i,j) × V_j
```

Q 和 K 算出谁重要，V 提供真正的内容。

### Multi-Head Attention

同时做 h 次 Attention，每次用不同的投影矩阵，学习不同角度的关系：

```
Head 1: 代词 → 名词
Head 2: 形容词 → 修饰对象
Head 3: 动词 → 主语
...

MultiHead(x) = Concat(head_1, ..., head_h) · W_O
```

LLaMA-70B: 64 head（8 KV head × 8 组，GQA）。

## Transformer 整体结构

```
          ┌─ 残差 ─┐
          ↓         │
x → RMSNorm → Attention → + → RMSNorm → FFN → + → 输出
                                    ↑        │
                               └─ 残差 ───┘
```

### 各组件职责

| 组件 | 作用 |
|------|------|
| Attention | 横向：token 间交换信息 |
| FFN | 纵向：每个 token 深化理解 |
| 残差连接 | 防梯度消失，信息保底通路 |
| RMSNorm | 归一化，稳定训练 |

## 完整流程

```
文本 → Tokenizer → Token IDs
         ↓
      Embedding 查表 → [seq_len, d_model]
         ↓
      + RoPE 位置编码
         ↓
      Transformer × N 层
         ↓
      LM Head → softmax → 预测下一个 token
```

## 知识地图

```
感知机 → MLP → FFN → Attention → Transformer Layer → LLM
  ↑               ↑        ↑              ↑
perceptron     ffn.md    本文         llm-topology.md
```
