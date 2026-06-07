---
date: 2026-06-07
tags: [扩散模型, 视频生成, Attention, 高速运动]
---

# 高速运动视频生成中的 Attention 失效与解决

## 背景：扩散模型生成视频

```
加噪：原始视频 → 逐步加噪 → 纯噪声
去噪：纯噪声 → 逐步去噪 → 生成视频（每步 = denoising step）
```

视频切成 3D patch（时空小块），每个 patch 是一个 token。Transformer + Attention 在这些 token 间传递信息。

## 为什么高速运动下 Attention 失效

### 静止 vs 高速

```
静止:   帧 t: 🐱────  帧 t+1: 🐱────  ← 同位置还是同一物体
高速:   帧 t: 🐱────  帧 t+1: ────🐱  ← 猫飞了，同位置是背景
```

### Softmax 稀释效应

$$Attention = softmax(Q·K^T / √d) · V$$

Softmax 总和恒为 1。高速运动时：

- 真正需要 attend 的远处 token → Q·K^T 虽然高，但就一个
- 附近大量无关 token → Q·K^T 低但数量多（背景、静态物）
- 大量低分 token 加起来稀释了关键 token 的注意力权重
- 结果：该 attend 的没 attend 到，信息通路断了

### 扩散模型的额外因素

静态背景 token 几步就收敛，高速运动 token 缺乏帧间对应关系，更难去噪。标准做法全员一样步数 → 背景浪费步数，运动区域步数不够。

## 解决方案

### 1. 对特定 token 增加 denoising steps

```
标准：所有 token 统一 50 步
改进：运动 token 80 步，静态 token 30 步
```

实现：用运动向量大小 / Attention 分数方差识别「困难 token」，在扩散 schedule 里做非均匀步长。

### 2. 对特定 token 加强 Attention

| 方法 | 思路 |
|------|------|
| 运动轨迹 Attention | 沿光流轨迹加 attention bias |
| 多尺度 Attention | 高速区域额外加全局 attention pass |
| Top-K 稀疏 + 运动补偿 | 给运动方向远处 token 更高优先级 |
| Adaptive compute | 注意力分散的 token 额外多做一轮 refinement |

## 一句话

高速运动 → 物体 token 在帧间空间距离远 → Softmax 预算被无关 token 稀释 → 信息通路断裂 → 运动模糊。解法：运动区域多步去噪 + 强化 Attention 通路。
