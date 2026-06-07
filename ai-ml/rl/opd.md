---
date: 2026-06-07
tags: [强化学习, 知识蒸馏, OPD, DeepSeek-R1]
---

# OPD（On-Policy Distillation，同策略蒸馏）

> 相关笔记：[RL 概述](rl-intro.md) | [MCTS + PRM](mcts-prm.md)

DeepSeek-R1 训练流程的关键步骤。Student 用自己的策略生成回答，Teacher 纠正，Student 从自己的错误中学习。

## 传统指示蒸馏 vs 同策略蒸馏

### 传统指示蒸馏（Off-Policy）

Teacher 提前对一批 prompt 批量生成回答，存成静态数据集。Student 从头到尾学这份固定数据。

```
Teacher 批量生成 → 存成数据集 → Student 学
```

**致命问题**：Student 只见过 Teacher 的分布（off-policy），没见过自己的错误，学不会「走错了怎么回来」。

### 同策略蒸馏（OPD / On-Policy）

Student 自己先试，Teacher 纠错。学生用自己的策略（on-policy）生成 → 被修正 → 更新 → 重新生成。

```
Student 生成 → Teacher 纠错 → Student 更新 → 循环
```

### 对比

| | 传统指示蒸馏 | 同策略蒸馏（OPD） |
|------|------------|-----------|
| 数据生成者 | Teacher | **Student 自己** |
| 数据分布 | Teacher 分布（off-policy） | **Student 当前分布**（on-policy） |
| 数据集 | 静态，一次生成 | 动态，随训练迭代更新 |
| 学生见过 | 只有正确输出 | 自己的错误 + Teacher 怎么改 |
| 迭代性 | 一次完成 | 多轮循环 |
| 推理链 | 只保留最终答案 | 保留探索 → 纠错 → 回溯全过程 |
| 本质 | SFT | RL（用 Teacher 替代 Reward Model） |

### 为什么 OPD 对推理重要

传统: Prompt → Teacher 输出 "x=5, y=3" → Student 学 Prompt → 答案
OPD:   Prompt → Student "x=3...不对...x=5 ✓" → Teacher 认可整个探索过程 → Student 学会探索+纠错

推理模型的核心能力「探索+纠错」在传统蒸馏中被完全丢弃，OPD 保留了这条链。

## OPD 循环

```
1. Student 自己生成一批回答
2. Teacher 打分/纠正/重写
3. 用纠正后的高质量回答训练 Student (SFT)
4. Student 策略更新 → 回到 1
```

## 对推理模型的意义

推理模型（o1/R1）的核心能力是长链推理：尝试多条路径、自我纠错、回溯重来。

OPD 只学 Teacher 的最终答案 vs OPD 让学生自己探索、Teacher 评估整条探索链（包括「自我纠错」的过程也值得学）。

## DeepSeek-R1 完整训练流程

```
1. Cold-Start SFT → 基本推理格式
2. RL (Reasoning) → 强化推理能力 (RLVR)
3. Rejection Sampling + SFT → 筛选高质量数据
4. RL (All-Scenario) → 全场景 RLHF
5. OPD → Teacher 纠正 Student, 蒸馏推理能力
```

OPD 放在最后：Student 需要先有推理能力，否则生成的回答太烂，全是纠错学不到探索能力。

## OPD vs RLHF

| | RLHF | OPD |
|------|------|-----|
| 评判者 | Reward Model（神经网络） | Teacher Model（更强的大模型） |
| 优化目标 | 最大化 Reward | 最大化 Teacher 输出似然（SFT loss） |
| 稳定性 | 可能训崩（Reward hacking） | 更稳定（SFT 不会 RL 训崩） |

OPD 是 RLHF 的 SFT 化变体——用 Teacher 替代 Reward Model，更稳定，但需要更强的 Teacher。
