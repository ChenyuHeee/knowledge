---
date: 2026-06-14
tags: [扩散模型, 表示学习, 自编码器, 训练优化]
---

# REPA 与 RAE

## REPA（Representation Alignment，表示对齐）

训练扩散模型时，将模型中间层特征与预训练编码器（DINO/CLIP）对齐，注入视觉先验，加速收敛。

```
传统:  噪声 → UNet/DiT → 预测噪声 → loss
REPA:  噪声 → UNet/DiT → 中间特征 h_j → 与 DINO(原图) 的表示 z 对齐
       L = L_noise + λ × MSE(h_j · W_proj, z)
```

**为什么有用**：扩散模型大量训练步耗在学基础视觉特征上（猫的耳朵、边缘等）。REPA 直接注入已训好的视觉知识，收敛快数倍，质量更高。本质是前面 CNN→ViT 知识链的应用——用已有视觉模型指导生成模型。

## RAE（Regularized Autoencoder）

让 AE 的潜在空间光滑连续，后续才能用于生成。普通 AE 的 latent space 不连续，相邻 latent 点解码出的内容可能截然不同。

RAE 加正则约束让 latent space 向高斯靠拢。是 VAE 和标准 AE 的中间方案——不做随机采样（避免 KL 塌缩），但通过正则让空间规整。SD/CogVideo 的 VAE 底层都用到类似思想，在规整性和重构精度之间取平衡。

## 两者关系

```
RAE:  正则化约束 → 让 AE latent space 规整可用
REPA: 对齐预训练模型 → 让扩散模型中间表示吸收视觉知识

共同: 用外部信号约束模型内部表示的质量
```
