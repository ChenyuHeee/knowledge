---
date: 2026-06-14
tags: [计算机体系结构, 网络, RDMA, InfiniBand, 分布式训练]
---

# RDMA（Remote Direct Memory Access，远程直接内存访问）

> 相关笔记：[GEMM](gemm.md) | [NVLink](nvlink.md)

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

## InfiniBand（IB）

RDMA 的原生载体。NVIDIA 通过收购 Mellanox 将其绑入全栈生态。

### 在互连栈的位置

```
机内 GPU 间:  NVLink + NVSwitch (1.8 TB/s)
跨机 GPU 间:  InfiniBand (400 GB/s, RDMA)
机架间:       InfiniBand / 以太网
```

### 与以太网的区别

| | 以太网 | InfiniBand |
|------|--------|----|
| RDMA | 需 RoCE 扩展 | **原生支持** |
| 延迟 | 几十 μs | **< 2 μs** |
| 带宽 | 100G/200G/400G | 400G/800G |
| 拥塞控制 | 丢包重传 | **信用机制**（无损） |
| 生态 | 开放多厂商 | NVIDIA/Mellanox 垄断 |

快的原因：**硬件信用流控**——发送前先获取接收方缓存额度，保证绝不丢包；**端到端 RDMA 卸载**——网卡硬件完成全部传输。

### 为什么 GPU 集群用它

以太网丢包 → 200μs 操作变 50ms → 训练吞吐波动剧烈。InfiniBand 通信延迟稳定在 2μs 以内，不分「好时候」「坏时候」。
