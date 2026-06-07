---
date: 2026-06-07
tags: [LLM推理, KV Cache, Prefill, Decode, Flash Attention]
---

# Prefill 与 Decode：KV Cache 处理全解析

> 相关笔记：[KV Head](kv-head.md) | [MLA](mla.md) | [KV Cache 压缩趋势](kv-cache-trend.md) | [Speculative Decoding](speculative-decoding.md) | [Flash Attention / Tile](llm-topology.md)

## 前置：KV Cache 为什么可以不重算

对于任意 token i，它进入每层时的隐藏状态 $x_i$ 在前向传播后就确定了，后续生成新 token 不会改变旧 token 的隐藏状态（Transformer 无循环连接）。

$$K_i = x_i \cdot W_K, \quad V_i = x_i \cdot W_V$$

$x_i$ 不变 → $K_i$, $V_i$ 静态 → 算一次终身有效。这就是 KV Cache 的存在前提。

## Prefill：一次性灌入全部 Prompt

### 输入

所有 Prompt token 一次性进入（例如 50 个 token → `X_in: [50, d_model]`）。

### 计算流程（每层）

```
X_in: [50, d_model]

1. 并行计算 Q/K/V:
   Q = X_in · W_Q → [50, d_model]
   K = X_in · W_K → [50, d_model]  ← 存入 K_cache[l]
   V = X_in · W_V → [50, d_model]  ← 存入 V_cache[l]

2. Attention:
   S = Q · K^T / √d        → [50, 50]  ← 完整 Attention 矩阵
   A = softmax(S + mask)   → [50, 50]
   O = A · V               → [50, d_model]

3. O → FFN → X_out: [50, d_model]
```

### 关键特点

- **Compute-bound**：50×50 Attention 矩阵 + [50, d_model] 矩阵乘 → Tensor Core 满载
- **KV Cache 行为**：纯写入，不读旧缓存（Prompt 前无历史）
- **Flash Attention 价值巨大**：$O(L^2)$ Attention 矩阵不完整存放，tile 化后逐块计算 + softmax + 乘 V 累加

### K/V 计算的深层原因

Prefill 时的 X_in 是 Prompt token 进入该层时的初始编码——不依赖后续层的处理。无循环依赖 → 所有 token 的 Q/K/V 可一次性并行算出。

## Decode：逐 Token 生成

### 输入

每步只有 1 个 token（上一步生成的）。`X_in: [1, d_model]`。

### 计算流程（每层）

```
X_in: [1, d_model]  ← 只有 1 个 token

1. 计算新 token 的 Q/K/V:
   Q_new = X_in · W_Q  → [1, d_model]
   K_new = X_in · W_K  → [1, d_model]  ← 追加到 K_cache
   V_new = X_in · W_V  → [1, d_model]  ← 追加到 V_cache

2. Attention（瓶颈）:
   S = Q_new · K_cache^T / √d   → [1, prev_len+1]
        ↑               ↑
     1 个 query      读取整个 K_cache（全部历史 token）
   
   O = A · V_cache             → [1, d_model]
              ↑
         同样读取整个 V_cache

3. O → FFN → X_out: [1, d_model]
```

### 关键特点

- **Memory-bound**：Q 只有 [1, d_model]，矩阵乘极小。瓶颈是读取整个 KV Cache 的 HBM 带宽。
- **KV Cache 行为**：99.9% 读，0.1% 写。每步读全部历史 KV，写 1 个新 token 的 K/V。

### 带宽开销实例（LLaMA-70B, FP16, 4096 token 上下文）

```
每层 KV Cache = 2 × 8 KV heads × 4096 tokens × 128 d_head × 2 bytes
             = 16 MB / 层
80 层 = 1.28 GB KV Cache 总量
每生成 1 token → 读 1.28 GB
HBM 带宽 ~2 TB/s → 读耗时 0.64ms
GPU 算力利用率 < 5%
```

**GQA/MLA/KV 量化的全部价值都在这里**：每减少 1 MB 的 KV Cache 就是节省 1 MB 的 HBM 读取量。

## 两个阶段对比

| | Prefill | Decode |
|------|---------|--------|
| 输入量 | 全部 Prompt token | 1 token/步 |
| KV Cache | **只写** | **读为主**（读全部历史，写 1 个新） |
| Attention | [L, L] 大矩阵 | [1, L] 一行 |
| 瓶颈 | Compute-bound | Memory-bound |
| GPU 利用率 | 高 | < 5% |
| Flash Attention | 省显存关键 | 无明显加速 |
| KV 压缩收益 | 较小 | **极大** |

## 工程延伸

### Chunked Prefill

超长 Prompt（128k token）一次灌入扛不住 → 切 Chunk，每个 Chunk 内 Compute-bound、Chunk 间有 KV Cache 读写。Prefill 从纯 Compute 向半 Memory 过渡。

### Prefill-Decode 分离

```
Prefill GPU (高算力):  接到 Prompt → 完成 → 传 KV Cache
Decode GPU (高带宽):   接收 KV Cache → 逐 token 生成 → 高吞吐
```

两个阶段物理分离，按需分别扩容。
