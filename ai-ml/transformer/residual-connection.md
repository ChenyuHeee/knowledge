---
date: 2026-06-05
tags: [深度学习, 残差连接, Transformer]
---
# 残差连接（Residual Connection）

> 相关笔记：[Transformer & Attention](transformer-attention.md) | [大模型拓扑结构](llm-topology.md)

ResNet (He et al., 2015) 提出，所有深层网络的标准配置。

## 解决的问题：梯度消失

80 层网络反向传播，梯度连乘 80 次：

$$\partial L/\partial x_1 = \partial L/\partial x_{80} \times (W_{80} \times ... \times W_1)$$

每层缩放因子 0.9 → $0.9^{80} \approx 0.0002$，浅层权重几乎不更新。

## 做法

在每层外面加一条短路：

```
原来:  output = Layer(input)
残差:  output = Layer(input) + input
```

## 数学原理

梯度有两条路径：

$$\partial(Layer(x) + x)/\partial x = \partial Layer(x)/\partial x + 1$$

那个 +1 保证即使 $\partial Layer(x)/\partial x \approx 0$，也至少有一路梯度无损回传。

L 层残差网络的输出展开：

$$x_L = x_0 + \sum_{i=1}^{L} F_i(x_{i-1})$$

$$\frac{\partial L}{\partial x_0} = \frac{\partial L}{\partial x_L} \cdot \left(1 + \sum_{i=1}^{L} \frac{\partial F_i}{\partial x_0}\right)$$

## 直觉：学「增量」

| 方式 | 模型学的 | 难度 |
|------|---------|------|
| 普通网络 | 从零构造整个输出 | 难 |
| 残差网络 | 只学「还要补什么」 | 易（输入已八九不离十） |

类比：改文章——普通方式 = 从头重写，残差方式 = 在原文上批注修改。

## Transformer 中的位置

```          
       ┌────── + ──────┐
       │               │
  RMSNorm         残差连接
       │               │
   Attention           │
       │               │
       └──→ 相加 ←─────┘

       ┌────── + ──────┐
       │               │
  RMSNorm         残差连接
       │               │
     FFN               │
       │               │
       └──→ 相加 ←─────┘
```

每个子层（Attention / FFN）各一个残差连接。

## Pre-Norm vs Post-Norm

| 方式 | 公式 | 特点 |
|------|------|------|
| Pre-Norm | `x = x + Layer(Norm(x))` | 梯度在残差路径无 Norm 阻挡，训练稳定（LLaMA/GPT-3） |
| Post-Norm | `x = Norm(x + Layer(x))` | 需要 warmup，容易训崩（原始 Transformer） |

Pre-Norm 是现代默认做法。
