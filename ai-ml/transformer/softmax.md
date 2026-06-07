---
date: 2026-06-07
tags: [深度学习, Softmax, Attention, 数学基础]
---

# Softmax

> 相关笔记：[Transformer & Attention](transformer-attention.md) | [高速运动视频 Attention](../diffusion/motion-video-attention.md)

## 定义

$$\text{softmax}(z_i) = \frac{e^{z_i}}{\sum_j e^{z_j}}$$

两条性质：每个输出 ∈ (0,1)，总和 = 1。输出 = 概率分布。

## 直觉：指数放大差异

```
z = [2.0, 1.0, 0.5]
e^z = [7.39, 2.72, 1.65]
softmax = [0.63, 0.23, 0.14]
```

2.0 → 0.63，1.0 → 0.23。原来的 2 倍差变成了 2.7 倍。$e^x$ 放大差距。

极端：$z = [5.0, 0.1, 0.1]$ → softmax ≈ $[0.985, 0.007, 0.007]$。「赢家通吃」。

## 温度系数

$$\text{softmax}(z_i / \tau)$$

| τ | 效果 |
|------|------|
| → 0 | 极端尖锐，全概率给 max |
| 1 | 正常 |
| → ∞ | 趋近均匀分布 |

Attention 中除以 $\sqrt{d_k}$ 本质是降温，防 Softmax 太尖导致梯度消失。

## 在 Attention 中的角色

$$\text{Attention} = \text{softmax}(Q·K^T / \sqrt{d_k}) · V$$

Softmax 决定每个 token 能从其他 token 获取多少信息（注意力预算）。

### 稀释问题的数学根源

10000 个背景 token 的 $e^{0.1} ≈ 1.1$，累加起来 = 11000。真正需要关注的 token $e^{2.0} ≈ 7.39$。分母 ≈ 11000 + 7.39 → 关键 token 只分到 0.06% 的预算。

**大量弱相关 token 的指数和碾碎了少数强相关 token 的指数**——这就是 Attention 失效的根源。

## 为什么必须用 e^x

- 保证输出非负（普通归一化 $\frac{z_i}{\sum z_j}$ 要求 z ≥ 0）
- 梯度简洁：$\text{softmax}(x)·(1 - \text{softmax}(x))$
