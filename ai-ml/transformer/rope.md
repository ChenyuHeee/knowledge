# RoPE（Rotary Position Embedding，旋转位置编码）

> 相关笔记：[Transformer & Attention](transformer-attention.md) | [KV Cache 压缩趋势](kv-cache-trend.md)

## 为什么需要

Attention 不感知位置。「A 打了 B」和「B 打了 A」的 Q·K^T 相同。RoPE 把位置信息编码进 Q 和 K。

## 直觉：旋转

根据 token 位置把 Q/K 向量做旋转。位置 i 旋转 θ_i，位置 j 旋转 θ_j。

Q·K^T 的结果只依赖于**旋转角度差** θ_j - θ_i = θ_{j-i}（相对位置）：

- 两个 token 近 → 角度差小 → 关联强
- 两个 token 远 → 角度差大 → 关联弱

## 数学

每组二维做旋转：

$$\begin{pmatrix} q_0' \\ q_1' \end{pmatrix} = \begin{pmatrix} \cos m\theta & -\sin m\theta \\ \sin m\theta & \cos m\theta \end{pmatrix} \begin{pmatrix} q_0 \\ q_1 \end{pmatrix}$$

旋转频率：

$$\theta_i = \frac{1}{10000^{2i/d}}$$

- 低维 → 高频 → 局部位置敏感
- 高维 → 低频 → 全局钝感

不同维度捕捉不同尺度的位置信息，类似傅里叶频域分解。

## 与 KV Cache 压缩的关系

RoPE 在完整维度上施加 2×2 旋转。低秩压缩（MLA）会把维度压扁，破坏旋转结构。

解决方案：
- DeepSeek-V2 MLA：KV 分两路，主路低秩压缩，小路保留完整维度专做 RoPE
- RAP (2026)：剪掉整对 RoPE 对齐的列，保持旋转结构不被破坏

## 与其他位置编码对比

| 方法 | 代表 | 思想 | 现状 |
|------|------|------|------|
| 绝对位置编码 | 原始 Transformer | 固定向量 + embedding | 淘汰 |
| 可学习 | BERT/GPT-2 | 学出位置向量 | 无法外推 |
| **RoPE** | LLaMA/Qwen/Mistral | 旋转 Q/K，隐含相对位置 | **主流** |
| ALiBi | BLOOM | Attention 加距离衰减偏差 | 简单但精度差 |
