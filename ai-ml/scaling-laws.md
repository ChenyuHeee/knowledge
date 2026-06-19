---
date: 2026-06-18
tags: [LLM, 训练, Scaling Law, Chinchilla]
---

# Scaling Laws（缩放定律）

LLM 时代的「牛顿定律」——给定算力预算，预测最优模型大小和数据量。

## 核心问题

100 万 GPU 小时怎么花？大模型+少数据 or 中等模型+海量数据？Scaling Laws 回答的就是这个。

## OpenAI Kaplan (2020)

Loss 随参数量 N 和数据量 D 独立地服从幂律：

$$L(N) = (N_c/N)^{\alpha_N}, \quad L(D) = (D_c/D)^{\alpha_D}$$

$\alpha_N \approx 0.076, \alpha_D \approx 0.095$。结论：参数比数据重要，把模型做大。

## DeepMind Chinchilla (2022)：数据喂少了

**每多 1 个参数，多喂 ~20 个 token**。参数和数据应等比增长。

$$N_{opt} \propto C^{0.5}, \quad D_{opt} \propto C^{0.5}$$

$$L(N, D) = \frac{A}{N^\alpha} + \frac{B}{D^\beta} + E$$

| 项 | 含义 |
|------|------|
| $A/N^\alpha$ | 模型不够大 |
| $B/D^\beta$ | 数据不够多 |
| $E$ | 不可约误差 |

Chinchilla 70B（1.4T token）干掉了 Gopher 280B。**数据喂饱比堆参数有效**。

## 最优配比实例

| 预算 | 参数 | 数据 | 例子 |
|------|------|------|------|
| 小 | 几十M | 几B | 小实验 |
| 中 | 7B | 200B | LLaMA-2 7B |
| 大 | 70B | 1.4T | Chinchilla |
| 超大 | 400B+ | 15T | LLaMA-3 405B（已超 Chinchilla 最优，数据继续喂仍有效） |

## Emergent Abilities（涌现）

Loss 平滑下降 → 某个任务的表现突然从随机跳到可用。不是某个参数让它会的，而是多项能力的平滑提升在特定任务上表现为突变。
