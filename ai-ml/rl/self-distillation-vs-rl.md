---
date: 2026-06-07
tags: [强化学习, 自蒸馏, RLHF, 知识蒸馏]
---

> 相关笔记：[RL 概述](rl-intro.md) | [OPD](opd.md) | [MCTS + PRM](mcts-prm.md) | [RL 训练稳定性](rl-training-stability.md)

# 强化学习与自蒸馏的关系

## 光谱

```
SFT → 自蒸馏 → OPD → RLHF/PPO
 ↑      ↑        ↑        ↑
背答案  挑好的   老师改   试错学习
```

| | 自蒸馏 | OPD | RLHF / PPO |
|------|--------|-----|------------|
| Teacher | 自己（前 checkpoint） | 更强模型 | Reward Model |
| 反馈信号 | 二元：对/错 | 生成式：Teacher 重写 | 标量：RM 打分 |
| 优化 | SFT loss | SFT loss | PPO（Reward - KL） |
| 负梯度 | 无 | 无 | **有** |
| 稳定性 | 最稳定 | 稳定 | 可能训崩 |

## 自蒸馏 = Rejection Sampling

```
1. 当前模型每个 prompt 生成 N 条回答
2. 规则或 RM 打分
3. 保留最高分的一条
4. 用保留的回答做 SFT 更新
5. 模型变强 → 回到 1
```

Teacher = Student = 自己。DeepSeek-R1 第三步就是自蒸馏。

## 自蒸馏 vs RL 的本质区别

### 自蒸馏

```
生成: [✓ ✓ ✗ ✗ ✗] → 扔掉 ✗ → SFT 最大化 log P(✓)
```

只知道什么是好的，不知道 ✗ 错在哪、离 ✗ 多远。信号粗糙但训练稳定。

### RL (PPO)

```
生成: [0.9, 0.8, 0.3, 0.2, 0.1] → 推高 0.9/0.8，压低 0.2/0.1
```

有负梯度信号，区分程度上的好坏。更强大但可能训崩，需要 KL 惩罚约束。

## 为什么交替使用

```
自蒸馏 → RL → 自蒸馏 → RL → ...

自蒸馏:  模型弱时稳定提下限
RL:      二元信号不够用后，精细提上限
再自蒸馏: RL 后的强模型生成更好数据 → 正反馈
```

## 公式表述

```
自蒸馏 = RL with binary reward + no negative gradient + SFT loss
RL     = 自蒸馏 with continuous reward + negative gradient + KL penalty
```
