---
date: 2026-06-18
tags: [计算机体系结构, GPU, Blackwell, NVIDIA, 训练]
---

# Blackwell：NVIDIA 最新 GPU 架构

> 相关笔记：[NVLink](nvlink.md) | [INT8/FP8/FP16/BF16/MXFP4](int8-quantization.md) | [RDMA](rdma.md)

## 定位

```
A100 (Ampere, 2020) → H100 (Hopper, 2022) → B200/GB200 (Blackwell, 2024)
```

## 关键规格

| | H100 SXM | **B200** |
|------|------|------|
| 显存 | 80 GB HBM3 | **192 GB HBM3e** |
| 显存带宽 | 3.35 TB/s | **8 TB/s** |
| FP16 | 990 TFLOPS | 2.25 PFLOPS (~2.3×) |
| FP8 | 1.98 PFLOPS | 4.5 PFLOPS (~2.3×) |
| FP4 | 不支持 | **9 PFLOPS** |
| NVLink | 900 GB/s (v4) | **1.8 TB/s (v5)** |
| 功率 | 700W | 1000W |

192 GB 显存 → 一张卡塞下 400B 模型 INT4 版。FP4 新增支持。

## GB200 Superchip

```
GB200 = 2×B200 GPU + 1×Grace CPU
NVL72  = 72×B200 装一个机柜，NVSwitch 全互联
        → 30 TB 总显存 + GPU-GPU 1.8 TB/s
```

## 互连体系

```
机内 GPU 间: NVLink 5 (1.8 TB/s, 翻倍)
机内交换:    NVSwitch (全互联)
跨机:        InfiniBand / Spectrum-X (RDMA)
```

万卡 Blackwell 集群 = 这个全栈的矩阵化。

## NVFP4

NVIDIA 私有的 4 位浮点，Blackwell 独占。

### 精度链

```
FP8 (E4M3/E5M2, H100) → NVFP4 (Blackwell) → MXFP4 (AMD 联盟)
```

### NVFP4 vs MXFP4

| | NVFP4 | MXFP4 |
|------|------|------|
| 推手 | NVIDIA | 微软/AMD/Intel/ARM |
| 硬件 | **Blackwell 专有** | 多厂商开放 |
| 位宽 | 4 bit | 4 bit |
| 指数 | 微缩放（一组共享） | 微缩放（一组共享） |
| 算力 | **9 PFLOPS** | 取决于实现 |
| 生态 | CUDA 闭源 | 各厂商自写 |

思路相同——4 位浮点 + 共享指数。INT4 的全局 scale 在 4bit 下更严重，NVFP4 靠微缩放在极限上保精度。9 PFLOPS = H100 FP8 的 2× = H100 FP16 的近 5×。
