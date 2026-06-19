---
date: 2026-06-18
tags: [计算机体系结构, GPU, CPU, SIMD, SIMT, CUDA]
---

# SIMD / SIMT

> 相关笔记：[GEMM](gemm.md) | [寄存器](register.md)

## SIMD（CPU 向量指令）

一条指令同时对多个数据做同一个操作。

```
VADD V1, V2  → 同时算 8 个 float32（AVX-256）
```

```
SSE (128bit) → AVX (256bit) → AVX-512 (512bit, 同时 16 个 float32)
```

编译器自动向量化或手写 intrinsics。

## SIMT（GPU 线程模型）

一条指令，32 个线程（Warp）同时执行，但各自读各自的数据。

```
Warp 32 线程:
  全部执行 ADD R1, R2
  但每个线程的 R1/R2 是各自独立寄存器, 指向不同数据
```

Tensor Core = SIMT 的延伸——一个 warp 协同做矩阵乘。

## 核心区别

| | SIMD | SIMT |
|------|------|------|
| 硬件 | CPU | GPU |
| 并行单元 | 向量寄存器 | 线程（各自寄存器） |
| 指令 | 指定向量宽度 | 写标量, 硬件打包 warp |
| 分支 | mask 处理 | **Warp divergence**（串行） |
| 代表 | AVX-512, NEON | CUDA Warp, AMD Wavefront |

## Warp Divergence（SIMT 的坑）

Warp 内线程走不同分支 → 两条路都执行 → 各自丢弃对方结果 → 效率直接砍半。CUDA 优化核心：避免 warp 内分支。

## 与已有知识连接

```
SIMD → 小矩阵 CPU 也能跑
SIMT → 为什么 GPU 算 GEMM 快: 一个 warp + Tensor Core = 一周期一个 4×4 矩阵乘加
```
