---
date: 2026-06-13
tags: [计算机体系结构, GPU, 矩阵乘法, 深度学习]
---

# GEMM（GEneral Matrix Multiply，通用矩阵乘法）

> 相关笔记：[CNN](../ai-ml/cnn.md) | [寄存器](register.md) | [CPI](cpi.md)

$$C = \alpha \cdot A \cdot B + \beta \cdot C$$

深度学习 90% 以上的 FLOP 最终都落在 GEMM 上。GPU 为它而生。

## 为什么重要

所有网络层的计算都变成 GEMM：

| 层 | 操作 | GEMM 形式 |
|------|------|------|
| 全连接 | Y = X·W | 直接 GEMM |
| FFN | gate/up/down | 全部 GEMM |
| Attention | S = Q·K^T, O = A·V | 全部 GEMM |
| CNN | im2col 后 | GEMM |

## 参数和计算量

```
A: [M, K]
B: [K, N]
C: [M, N]
计算量: ≈ 2 × M × N × K FLOP
```

| 层 | M | K | N | FLOP |
|------|---|---|----|------|
| Q·K^T | seq_len | d_head | seq_len | 2L²d |
| FFN | B×L | d_model | d_ff | 2BLd×4d |
| LM Head | B×L | d_model | vocab | **2BLdV** (最大) |

## 优化：Tiling 分块

直接算大矩阵 → 数据反复从内存读 → 带宽瓶颈。

Tiling：切成小块（如 128×128）→ 每块留在 cache 里反复用直到算完。三级 tiling：

```
Global Memory (HBM) → Shared Memory → 寄存器 (Tensor Core)
```

利用率对比：cuBLAS > 90%，手写 for 循环 < 5%。

## im2col：卷积变 GEMM

CNN 的 3×3 滑动窗口 → GEMM：

```
1. im2col: 每 3×3 窗口拉成一列
   7×7 图 = 49 个位置, 每个取 3×3×3=27 值
   → 输入展开为 [49, 27]

2. 卷积核展开: [27, 64]

3. GEMM: [49, 27] × [27, 64] = [49, 64]
```

一次矩阵乘完成全部 49 个位置的 64 通道卷积。
