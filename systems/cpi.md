---
date: 2026-06-10
tags: [计算机体系结构, CPU, 性能, CPI, IPC]
---

> 相关笔记：[寄存器](register.md)

# CPI（Cycles Per Instruction，每条指令时钟周期数）

## CPU 性能公式

$$\text{Execution Time} = \text{Instruction Count} \times CPI \times \text{Clock Cycle Time}$$

| 因素 | 负责方 | 含义 |
|------|--------|------|
| Instruction Count | 编译器 + ISA | 程序共多少条指令 |
| **CPI** | **微架构** | 每条指令平均几个周期 |
| Cycle Time | 工艺 + 物理设计 | 每个周期多长 |

三个因素互制约。RISC 指令简单 CPI 低但指令多；CISC 指令复杂但 CPI 高。

## CPI 构成

$$CPI = CPI_{base} + CPI_{stall}$$

**CPI_base**：流水线完美运转、无停顿。标量单发射 = 1，4 发射超标量可到 0.25。

**CPI_stall**：三大 hazard：

| Hazard | 原因 | 典型代价 |
|--------|------|----------|
| 数据冒险 | 指令依赖，等前一条写回 | 1~3 个 bubble |
| 控制冒险 | 分支预测错，错误路径指令作废 | 10~20 周期 |
| 结构冒险 | 两条指令抢同一硬件资源 | 1 个 bubble |

现代 CPU 的 CPI 大头是 cache miss（L3 miss = 200+ 周期）和分支预测错误。

## CPI 与 IPC

$$IPC = 1/CPI$$

| CPI | IPC | 含义 |
|-----|-----|------|
| 1.0 | 1.0 | 单发射标量 |
| 0.5 | 2.0 | 超标量 2 发射 |
| 0.25 | 4.0 | Apple M1 Firestorm 8 发射理想情况 |
| 5.0 | 0.2 | 数据库 workload, 大量 cache miss |

工业界习惯说 IPC，学术界教材习惯说 CPI。

## 为什么 CPI < 1 可以成立

超标量处理器一周期发射多条指令到不同执行单元。ALU + LSU + FPU 同时干活。

但要求指令间无依赖、执行单元不冲突、分支预测正确。实际程序 IPC ~1.5~3（桌面级）。

## 与 LLM 推断的联系

```
CPU CPI 被 stall 吃掉   ↔   LLM Decode 被 HBM 带宽卡住
理论峰值从来不是问题      停顿才是
```
