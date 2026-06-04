# 感知机（Perceptron）

> 相关笔记：[FFN / MLP](ffn.md) | [大模型拓扑结构](llm-topology.md)

## 定义

神经网络的最基本单元。Rosenblatt, 1958。

$$y = f\left(\sum_i w_i x_i + b\right)$$

- $x_i$：输入
- $w_i$：权重
- $b$：偏置
- $f$：激活函数（原始的用 step function，输出 0/1）

## 几何直觉

感知机 = 在特征空间画一条线（或超平面），把两类点分开。

$$w_1 x_1 + w_2 x_2 + b = 0$$

## XOR 问题

单层感知机无法解决 XOR——这不是算法问题，是**线性模型的根本局限**：

```
XOR: 一条直线永远分不开
  0 ──●───○
  1 ──○───●
```

解决方法：加隐藏层 → MLP。两层感知机 = 两条线组合 = XOR 可解。

## 演进路径

```
感知机 (1958)
  │  单层, 线性分类, XOR 搞不定
  ▼
MLP (多层感知机)
  │  多层 + 非线性激活 → 万能近似
  ▼
FFN (Transformer 里的 MLP)
  │  SwiGLU, 升维+降维
  ▼
Transformer
     Attention + FFN
```
