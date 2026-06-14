---
date: 2026-06-14
tags: [计算机体系结构, GPU, NVLink, PCIe, 分布式训练]
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

## PCIe：CPU 和外设的总线

PCIe 是连接 CPU 和 GPU/SSD/网卡的标准总线。

```
CPU → 内存控制器 → DDR5
CPU → PCIe Root Complex → PCIe x16 → GPU
                         → PCIe x4  → NVMe SSD
                         → PCIe x1  → 网卡
```

带宽由 **Lane × Generation** 决定：

| 代 | 单 Lane | x16 总带宽 |
|------|------|------|
| PCIe 3.0 | ~1 GB/s | ~16 GB/s |
| PCIe 4.0 | ~2 GB/s | ~32 GB/s |
| PCIe 5.0 | ~4 GB/s | ~64 GB/s |
| PCIe 6.0 | ~8 GB/s | ~128 GB/s |

### PCIe vs NVLink

| | PCIe 5.0 x16 | NVLink 5.0 |
|------|------|------|
| 带宽 | 64 GB/s | 1.8 TB/s (~30×) |
| 连接对象 | CPU ↔ GPU | **GPU ↔ GPU**（不经过 CPU） |
| 延迟 | 高（经过 Root Complex） | 极低（直连） |
| 定位 | 通用总线 | GPU 互连专有 |

PCIe 慢不在电气速率，而在 GPU 间通信必须绕 CPU → 每次多经过一跳。NVLink 绕过这个中间人。CXL 正在基于 PCIe 物理层建开放标准的直连方案。
