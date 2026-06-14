---
date: 2026-06-14
tags: [计算机体系结构, 网络, RDMA, 分布式训练]
---

# RDMA（Remote Direct Memory Access，远程直接内存访问）

> 相关笔记：[GEMM](gemm.md) | [CPI](cpi.md)

## 是什么

一台机器直接读写另一台机器的内存，不唤醒对方 CPU。

```
传统 TCP/IP: 应用 → CPU copy → 内核 → CPU copy → 网卡 → 网络 → 对端 CPU copy → 内核 → CPU copy → 应用
RDMA:       应用 → 网卡直接通过 PCIe 读写内存 → 网络 → 对端网卡直接写内存 → 应用（CPU 全程无关）
```

## 关键差异

| | TCP/IP | RDMA |
|------|--------|------|
| 延迟 | 几十~几百 μs | **1~5 μs** |
| 带宽 | CPU 开销大 | 接近线速 |
| CPU 参与 | 每包中断 | **零 CPU 参与传输** |
| 拷贝 | 多次内核↔用户态 | **零拷贝** |
| 协议栈 | OS 内核 | **硬件卸载到网卡** |

## 为什么 LLM 训练需要

分布式训练：`GPU 计算 → AllReduce 梯度同步 → GPU 计算 → ...`

TCP 的 200μs 延迟累积成瓶颈。RDMA 把同步压到 2μs，几乎不耽误计算。

千卡集群标配 InfiniBand（原生 RDMA）或 RoCE。NVIDIA GB200：机柜内 NVLink + 跨机柜 InfiniBand = 统一远程内存网。

## 三种实现

| 技术 | 说明 |
|------|------|
| InfiniBand | 原生 RDMA，性能最强，NVIDIA/Mellanox |
| RoCE v2 | RDMA over Converged Ethernet |
| iWARP | RDMA over TCP/IP，最兼容 |
