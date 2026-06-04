# FFN（Feed-Forward Network，前馈网络）

> 相关笔记：[大模型拓扑结构](llm-topology.md) | [感知机](perceptron.md)

## 直觉

- Attention：token 之间**横向交换信息**（「猫」看到「老鼠」）
- FFN：每个 token **纵向深化理解**（消化聚合后的上下文，重新编码）

## MLP 与 FFN 的关系

**MLP（Multi-Layer Perceptron）** 是最基础的神经网络结构：多层全连接 + 非线性激活堆叠。

万能近似定理：至少一层隐藏层 + 非线性激活 → 可以任意精度逼近任意连续函数。

**FFN 就是 MLP 在 Transformer 语境下的特例**：

```
MLP（泛称，任意层数）              FFN（Transformer内，通常2层）
  输入层                             输入 [d_model]
    ↓ W1                               ↓ W1 [d_model → d_ff]
  隐藏层 + ReLU                       隐藏层 + SwiGLU
    ↓ W2                               ↓ W2 [d_ff → d_model]
  输出层                             输出 [d_model]
```

日常交流中两者经常混用，说「Transformer 的 MLP 层」和「Transformer 的 FFN」是同一个东西。

## 结构

两层的 MLP，先升维再降维：

```
输入 [d_model]
  → Linear1: [d_model] → [d_ff]     (升维, 通常 d_ff = 4 × d_model)
  → 非线性激活 (ReLU / GeLU / SwiGLU)
  → Linear2: [d_ff] → [d_model]     (降维)
  → 输出 [d_model]
```

先升维再降维的原因：低维空间线性不可分的模式，升到高维后非线性激活能学到更复杂的特征，再压缩回低维保留精华。

## 激活函数演进

| 模型 | 激活 | 矩阵数 |
|------|------|--------|
| 原始 Transformer | ReLU | 2 |
| BERT / GPT-2 | GeLU | 2 |
| LLaMA / Mistral / Qwen | SwiGLU | **3**（gate / up / down） |

### SwiGLU 原理

```
Standard FFN (2矩阵):
  x → W1 → ReLU → W2 → out

SwiGLU FFN (3矩阵):
  x → gate(xW_gate) ⊙ (xW_up) → W_down → out
        ↑ 门控(Sigmoid-like)  ↑ 候选内容
```

gate 控制「哪些信息通过」，up 提供候选内容，逐元素相乘后经 down 压缩。比 ReLU 更精细，但参数量 +50%。

## 参数量占比

LLaMA-70B 为例（d_model=8192, d_ff=28672）：

- Attention：4 × [8192, 8192] = 256M / 层
- FFN：3 × [8192, 28672] = **672M / 层（~72%）**

FFN 是模型参数的大头。MoE 架构的核心优化就是把 FFN 替换成多个 Expert FFN，每次只激活部分专家来降低计算量。
