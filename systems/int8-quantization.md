---
date: 2026-06-14
tags: [计算机体系结构, GPU, 量化, INT8, FP16, BF16, 推理优化]
---

# INT8 量化

> 相关笔记：[GEMM](gemm.md) | [寄存器](register.md) | [Prefill & Decode](../ai-ml/transformer/prefill-decode.md)

## 是什么

将 FP32 参数压缩到 8 位整数。线性映射：

$$x_{int8} = round(x_{fp32} / scale), \quad scale = max_{fp32} / 127$$

## 效果

```
FP32: 70B × 4 字节 = 280 GB
INT8: 70B × 1 字节 = 70 GB  ← 一张 H100 80GB 刚好放下
```

## 量化粒度

| 粒度 | 做法 | 精度 |
|------|------|------|
| Per-tensor | 全矩阵共用一个 scale | 粗 |
| Per-channel | 每行/列单独 scale | 好（权重标准） |
| Group-wise | 每 128 元素一组 scale | 最好（llama.cpp Q4 等） |

## 为什么加速

INT8 乘法器比 FP16 简单 → 同面积下数量翻倍 → INT8 GEMM 吞吐 2×。

数据量减半 → HBM 读取减半 → 直接影响 Decode 阶段的瓶颈。

## 什么能压什么不能压

| 组件 | 量化方式 | 说明 |
|------|------|------|
| 权重 | INT8（离线） | 训完不动，好压 |
| KV Cache | INT8（动态） | 每步统计范围 |
| 激活 | 通常不压 | 范围动态变化，精度敏感 |
| Attention | FP16/BF16 | Softmax 极敏感 |

## FP16 与 BF16

16 位浮点的两种格式，区别在位分配：

| | FP32 | FP16 | BF16 |
|------|------|------|------|
| 总位数 | 32 | 16 | 16 |
| 指数位 | 8 | 5 | **8（同 FP32）** |
| 尾数位 | 23 | 10 | 7 |
| 范围 | ±3.4×10³⁸ | ±65504 | **±3.4×10³⁸** |
| 训练 loss scaling | 否 | **是** | **否** |

FP16 指数小 → 梯度稍大就溢出 → 需要 loss scaling。BF16 指数同 FP32 → 绝不溢出 → 收敛更稳。精度低但 SGD 噪声吞掉了那几位尾数的误差。

**训练用 BF16/FP16 混合精度**（GEMM 用 BF16 算，参数主副本存 FP32），**推理用 INT8/INT4**。

## 与前文连接

- INT8 KV Cache → HBM 读取减半 → Decode 加速
- INT8 GEMM → 吞吐 2× → Compute-bound 阶段直接受益
- 精度链: FP32 → BF16/FP16 → INT8 → INT4
