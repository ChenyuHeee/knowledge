# 大模型拓扑结构：Layer / Tensor / Tile / Block / Checkpoint

> 相关笔记：[FFN 详解](ffn.md) | [Transformer & Attention](transformer-attention.md)

## Layer（层）

模型的基本结构单元。Decoder-only Transformer 的每层包含：

```
Transformer Layer
├── Input RMSNorm
├── Multi-Head Attention
│   ├── Q/K/V 投影: [d_model, d_model]
│   ├── RoPE 位置编码
│   └── Output 投影: [d_model, d_model]
├── Post-Attention RMSNorm
└── FFN (SwiGLU)
    ├── gate_proj: [d_model, d_ff]
    ├── up_proj:   [d_model, d_ff]
    └── down_proj: [d_ff, d_model]
```

LLaMA-70B = 80 层。层数决定模型深度。

## Block（块）

两层含义：

### 架构层面

Transformer Block = 一个 Layer（Attention + FFN），与 layer 混用。

### 分布式层面（Pipeline Parallelism）

N 层切成若干 stage，每个 stage 是一个 pipeline block：

```
GPU 0: Layer 0-19   (PP Block 0)
GPU 1: Layer 20-39  (PP Block 1)
GPU 2: Layer 40-59  (PP Block 2)
GPU 3: Layer 60-79  (PP Block 3)
```

## Tensor（张量）

模型中的一切数据：

| 类型 | 形状示例 | 说明 |
|------|---------|------|
| 权重 | `[d_model, d_model]` | 模型参数，固定 |
| 激活 | `[batch, seq_len, d_model]` | 前向中间结果，动态 |
| 梯度 | 同 weight | 反向传播，用于更新权重 |
| KV Cache | `[batch, n_heads, seq_len, d_head]` | 推理缓存 |
| 优化器状态 | 同 weight × 2 | Adam 的 m 和 v，训练时内存大头 |

### 参数量计算

LLaMA-70B：d_model=8192, d_ff=28672, 80 层：

$$80 \times (4 \times 8192^2 + 3 \times 8192 \times 28672) \approx 70\text{B}$$

## Tile（分块）

大 tensor 切小块，分批算。两种典型：

### Flash Attention 的 tiling

沿 seq_len 切 Q，每次只算一小块 attention matrix S，当场 softmax 后乘 V 累加，不存完整 S。显存 O(n²) → O(n)。

### Tensor Parallelism 的 tiling

沿列切权重，多 GPU 各算各的再 gather。

## Checkpoint（检查点）

### 模型存储

训练快照：权重 + 优化器状态 + 训练步数。常用格式 `.pt` / `.safetensors`。

### Gradient / Activation Checkpointing

用时间换空间。前向不保存所有激活，反向时遇到缺失的从最近 checkpoint 重算。
显存 O(N) → O(√N)，计算量 +33%。

## 整体拓扑图

```
模型 (checkpoint)
  └── Layer 0 (Transformer Block)
  │     ├── Attention (Q/K/V/O, 每个都是 Tensor)
  │     │     └── Flash Attention: 沿 seq_len 切 tile 省显存
  │     └── FFN (gate/up/down, 每个都是 Tensor)
  │           └── TP: 切 tile 多 GPU 并行
  ├── Layer 1
  ├── ...
  └── Layer N-1
        ↕ PP: Block = 一组 Layer, 分 GPU 串行
```
