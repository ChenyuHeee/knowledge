---
date: 2026-06-07
tags: [强化学习, RLHF, 训练稳定性, PPO]
---

# RL 训练崩掉的检测与预防

> 相关笔记：[RL 概述](rl-intro.md) | [自蒸馏 vs RL](self-distillation-vs-rl.md) | [OPD](opd.md)

## 三大核心信号

### 1. KL 散度失控

PPO: `max Reward - β × KL(old, new)`

- 正常：KL 缓慢 ↑，Reward ↑，同步健康
- 要崩：KL 突然跳升（> 上次 5 倍），Reward 不动或 ↓

KL 跳升一个数量级 = 即将崩的明确前兆。

### 2. Reward Hacking

模型找到 RM 盲区，Reward 分数狂涨但输出实际是垃圾。

```
正常 (Reward 2.3): "秦始皇统一了六国..."
Hacking (Reward 4.8): "秦始皇秦始皇统一统一统一..." ← 重复关键词骗分
```

监控项：

| 指标 | 正常 | 崩了 |
|------|------|------|
| Reward | 缓慢 ↑ | 快速 ↑ 或震荡 |
| 输出熵 | 维持 | 急剧 ↓ |
| 平均长度 | 稳定 | 突然突变 |
| PPL | 平稳 | 飙升 |

### 3. PPL 爆炸（Catastrophic Forgetting）

RL 更新破坏预训练的语言能力。PPL 连续上升且不回头 → 模型开始「忘本」。

## 检查清单

每次 RL epoch 后：

```
□ KL 是否异常（突然 > 上次 5 倍）
□ Reward 分布是否合理
□ 输出长度是否稳定
□ n-gram 重复率是否飙升
□ PPL 是否连续 3+ 步上升
□ 人工抽查 10 条
```

## 崩了怎么办

| 症状 | 对策 |
|------|------|
| KL 跳升 | 加大 β，减小 lr |
| Reward Hacking | 重训 RM，加对抗样本 |
| PPL 飙升 | 加 LM loss 联合训练（PPO-ptx） |
| 输出重复 | 加 diversity penalty，降 temperature |
| 全崩 | 回 checkpoint，减小更新幅度重来 |

## 最稳妥的防崩手段：PPO-ptx

PPO update 中同时加 SFT loss：

```
L = L_PPO + γ × L_LM(on high-quality data)
```

给模型一个「锚」，防止飘太远。OpenAI 在 InstructGPT 论文里就是这么做的。
