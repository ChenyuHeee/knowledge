---
date: 2026-06-05
tags: [索引]
---

# 知识库索引

## AI / ML

### LLM 工具与协议

| 笔记 | 说明 |
|------|------|
| [MCP](ai-ml/mcp.md) | Model Context Protocol，LLM 连接外部工具的标准协议 |
| [全双工](ai-ml/full-duplex.md) | 实时 AI 交互，流式 ASR/LLM/TTS，Barge-in、Turn-taking |
| [VLM](ai-ml/vlm.md) | 视觉语言模型，ViT + LLM，Visual Token 与 Softmax 稀释 |

### Transformer 与深度学习

| 笔记 | 说明 |
|------|------|
| [感知机](ai-ml/transformer/perceptron.md) | 神经网络最小单元，XOR 局限，到 MLP 的演进 |
| [FFN / MLP](ai-ml/transformer/ffn.md) | 前馈网络，SwiGLU，MLP 与 FFN 的关系，参数量占比 |
| [Transformer & Attention](ai-ml/transformer/transformer-attention.md) | Attention 机制，Q/K/V，Multi-Head，Transformer 完整结构 |
| [残差连接](ai-ml/transformer/residual-connection.md) | 梯度消失，残差直觉，Pre-Norm vs Post-Norm |
| [大模型拓扑结构](ai-ml/transformer/llm-topology.md) | Layer / Tensor / Tile / Block / Checkpoint 层级解析 |
| [KV Head](ai-ml/transformer/kv-head.md) | MHA / MQA / GQA，KV Cache 与推理显存优化 |
| [MLA](ai-ml/transformer/mla.md) | 多头潜在注意力，低秩 KV 压缩，DeepSeek-V2 |
| [RoPE](ai-ml/transformer/rope.md) | 旋转位置编码，相对位置隐含在 Q/K 旋转中 |
| [Softmax](ai-ml/transformer/softmax.md) | 指数归一化，温度系数，注意力预算与稀释的数学根源 |
| [KV Cache 压缩趋势](ai-ml/transformer/kv-cache-trend.md) | MLA 之后：非对称量化、Token 剪枝、RoPE 适配 |
| [Speculative Decoding](ai-ml/transformer/speculative-decoding.md) | 推测解码，小模型草稿 + 大模型验证，推理加速 |
| [Draft/Retrieve](ai-ml/transformer/draft-retrieve.md) | 缓存检索 + 草稿生成混合策略，前缀复用 |
| [Prefill & Decode](ai-ml/transformer/prefill-decode.md) | Prefill/Decode 阶段详解，KV Cache 读写模式，Compute vs Memory bound |

阅读顺序：感知机 → FFN → Transformer & Attention → 残差连接 → 拓扑结构

### 强化学习

| 笔记 | 说明 |
|------|------|
| [RL 概述](ai-ml/rl/rl-intro.md) | 强化学习基本框架，Agent/Environment/Policy/Value，DQN/PPO，RLHF |
| [MCTS + PRM](ai-ml/rl/mcts-prm.md) | 蒙特卡洛树搜索 + 过程奖励模型，o1/R1 推理架构 |
| [OPD](ai-ml/rl/opd.md) | 同策略蒸馏，与传统指示蒸馏的对比，DeepSeek-R1 训练 |
| [自蒸馏 vs RL](ai-ml/rl/self-distillation-vs-rl.md) | 强化学习与自蒸馏的关系、区别、交替使用 |
| [RL 训练稳定性](ai-ml/rl/rl-training-stability.md) | KL 监控、Reward Hacking、防崩检查清单与对策 |

### 扩散模型与生成

| 笔记 | 说明 |
|------|------|
| [高速运动视频 Attention](ai-ml/diffusion/motion-video-attention.md) | 高速运动下 Attention 失效分析，Softmax 稀释，多步去噪 + 强化 Attention |
| [定位关键 Token](ai-ml/diffusion/key-token-detection.md) | 四种定位方法：光流、Attention 熵、Token Loss、跨帧匹配 |
| [流式视频](ai-ml/diffusion/streaming-video.md) | 流式视频生成，长程 Attention 衰减，与 KV Cache 的同一性 |
| [Time Chunk](ai-ml/diffusion/time-chunk.md) | 时间分块 Attention，块内 Dense + 块间 Sparse |

### 向量检索

| 笔记 | 说明 |
|------|------|
| [大规模向量索引概述](ai-ml/vector-search/vector-indexing.md) | ANN 问题，四类方法（图/量化/哈希/树），流式索引 |
| [IVF-PQ 概述](ai-ml/vector-search/ivf-pq.md) | IVF 粗筛 + PQ 压缩，基本流程 |
| [IVF-PQ 深入](ai-ml/vector-search/ivf-pq-deep-dive.md) | K-means 细节，ADC 距离计算，内存测算，调参 |

阅读顺序：向量索引概述 → IVF-PQ 概述 → IVF-PQ 深入

## 算法与数据结构

| 笔记 | 说明 |
|------|------|
| [BVH](algorithms/bvh.md) | 层次包围盒，光线追踪空间加速结构，O(log N) 剪枝 |
| [霍夫曼编码](algorithms/huffman.md) | 贪心最优前缀码，高频短码低频长码，与 BPE 的关系 |

## 系统 / 体系结构

| 笔记 | 说明 |
|------|------|
| [cgroup 限制内存](linux/cgroup-memory-limit.md) | cgroup v2 memory.max，限制进程内存使用 |
| [NUMA & numactl](linux/numa-movepages-numactl.md) | NUMA 拓扑，move_pages，numactl 绑核绑内存 |
| [CPI](systems/cpi.md) | 每条指令时钟周期数，IPC，Hazard，与 LLM 推断的类比 |
| [寄存器](systems/register.md) | 存储金字塔顶层，RISC Load-Store，Register File |

## Web / 前端

| 笔记 | 说明 |
|------|------|
| [DOM 与 API](web/dom-api.md) | 文档对象模型，API 概念，DOM API 的关系 |

## Python

| 笔记 | 说明 |
|------|------|
| [Uvicorn](python/uvicorn.md) | ASGI 服务器，ASGI vs WSGI，生产部署 |

---

## 知识网络：跨簇的深层线索

### 线索 1：Softmax 稀释

同一问题出现在三个领域：

```
Attention 基础 → 高速运动视频 → VLM
   Softmax        视觉 token 间       visual token 挤占
   全局归一化      的注意力被稀释      文本的注意力预算
```

**关键笔记**：[Softmax](ai-ml/transformer/softmax.md) → [高速运动视频 Attention](ai-ml/diffusion/motion-video-attention.md) → [VLM](ai-ml/vlm.md) → [定位关键 Token](ai-ml/diffusion/key-token-detection.md)

### 线索 2：长序列的 Attention 状态管理

同一问题换三个名字：

```
LLM 推理:              视频生成:
  KV Cache 压缩         流式视频
  MLA / GQA             Time Chunk
  Draft/Retrieve        锚点帧 + 记忆压缩
  Speculative Decoding  分层 Attention
```

**关键笔记**：[KV Cache 压缩趋势](ai-ml/transformer/kv-cache-trend.md) → [流式视频](ai-ml/diffusion/streaming-video.md) → [Time Chunk](ai-ml/diffusion/time-chunk.md) → [Draft/Retrieve](ai-ml/transformer/draft-retrieve.md)

### 线索 3：Chunk 级并行流水线

三个领域的同一个模式——块内 Dense + 块间 Sparse：

```
Spec Decoding:     Token → Draft Chunk → 验证
Time Chunk:        帧 → Temporal Chunk → 稀疏 Attention
Full Duplex:       音频 → 200ms Chunk → 并行流水线
```

**关键笔记**：[Speculative Decoding](ai-ml/transformer/speculative-decoding.md) → [Time Chunk](ai-ml/diffusion/time-chunk.md) → [全双工](ai-ml/full-duplex.md)

### 线索 4：从试错到训练

```
RL 概述 → MCTS + PRM → OPD → 自蒸馏 → RL 训练稳定性
   ↓          ↓           ↓      ↓          ↓
 Agent    推理搜索    知识蒸馏  SFT替代   KL/PPL监控
```

### 线索 5：从神经元到 VLM

```
感知机 → MLP → FFN → Attention → Transformer → LLM → VLM
  ↑                                               ↑
 基础单元                                      全在复用
```

**关键笔记**：[感知机](ai-ml/transformer/perceptron.md) → [FFN](ai-ml/transformer/ffn.md) → [Transformer & Attention](ai-ml/transformer/transformer-attention.md) → [大模型拓扑](ai-ml/transformer/llm-topology.md) → [VLM](ai-ml/vlm.md)
