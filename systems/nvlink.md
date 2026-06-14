---
date: 2026-06-14
tags: [计算机体系结构, GPU, NVLink, 分布式训练]
---

# NVLink

> 相关笔记：[RDMA](rdma.md) | [GEMM](gemm.md)

## 是什么

NVIDIA 的 GPU 间高速互连。GPU 直接读写对方显存，不经过 PCIe/CPU。

## 为什么需要

PCIe 5.0 x16: ~64 GB/s。NVLink 5.0: ~1.8 TB/s（单链路 100 GB/s × 18 路）。差约 30 倍。

多 GPU 并行训练需要每层 AllReduce 聚合 → PCIe 会成为瓶颈。

## 拓扑：NVLink + NVSwitch

```
GPU1 ─┐
GPU2 ─┤
 ... ─┼─ NVSwitch（无阻塞交叉开关）─ 任意 GPU 对之间全带宽通信
GPU71─┤
GPU72─┘
```

NVSwitch 把 72 张 GPU 全接入交换，两两直连全带宽。不需要 CPU，不经过 PCIe。GB200 NVL72 是典型。

## 多级互连

```
芯片内:  SM ↔ HBM
单机多卡: GPU ↔ GPU: NVLink + NVSwitch (1 TB/s+)
跨机:    GPU ↔ GPU: InfiniBand / RDMA (400 GB/s)
跨集群:  以太网 / InfiniBand
```

NVIDIA 全栈：CUDA + NCCL + NVLink + NVSwitch + InfiniBand/Mellanox。
