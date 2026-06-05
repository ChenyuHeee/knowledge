---
date: 2026-06-05
tags: [强化学习, RL, 机器学习]
---

> 相关笔记：[MCTS + PRM](mcts-prm.md)

# RL（Reinforcement Learning，强化学习）

> 机器学习的第三范式：试错学习。

## 与监督学习的区别

| | 监督学习 | 强化学习 |
|------|----------|----------|
| 数据 | 带标签静态数据集 | 靠环境给奖励标量 |
| 反馈 | 即时 | **延迟** |
| 目标 | 拟合标签 | 最大化累积奖励 $G_t$ |

$$G_t = r_t + \gamma r_{t+1} + \gamma^2 r_{t+2} + ...$$

$\gamma \in [0,1]$ 为折扣因子，平衡短期与长期回报。

## 基本框架

```
        action (动作)
   ┌────────────────────→
   │                      │
  Agent               Environment
   │                      │
   ←────────────────────┘
   state (状态), reward (奖励)
```

## 核心概念

### Policy（策略）

给定状态下选什么动作：

$$\pi(a|s) = P(\text{做动作 } a \mid \text{看到状态 } s)$$

确定性 vs 随机性（用于探索）。

### Value Function（价值函数）

一个状态「有多好」：

$$V(s) = \mathbb{E}[\text{从 s 出发能拿到的累积奖励}]$$

Q-function：具体到每个动作的价值：

$$Q(s, a) = \mathbb{E}[\text{在 s 做 a 后拿到的累积奖励}]$$

## 两类算法

### Value-based：DQN

学 Q(s,a) → 选 Q 值最高的动作。

- Experience Replay：存经历随机抽样，打破数据相关
- Target Network：双网络防训练震荡

### Policy-based：PPO

直接优化策略网络 $\pi_\theta(a|s)$。

PPO 核心：限制每次更新幅度，防策略突变导致训崩。RLHF 中用的就是 PPO。

## RLHF 与 LLM 对齐

ChatGPT 背后的 RL：

```
1. SFT：人类优质回答训练
2. 训练 Reward Model：人类标注偏好 → 打分模型
3. PPO 优化：LLM 生成 → Reward Model 打分 → 更新参数
```

RL 不负责学知识（预训练已完成），负责**校准行为**——让模型更有用、更安全、更对齐。

## 核心矛盾：探索 vs 利用

- 利用（Exploitation）：吃已知好吃的店
- 探索（Exploration）：试试没去过的新店

只利用不探索 → 局部最优。只探索不利用 → 永远吃不到好的。RL 的核心 tradeoff。
