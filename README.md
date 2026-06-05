# 知识库索引

## AI / ML

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

阅读顺序：感知机 → FFN → Transformer & Attention → 残差连接 → 拓扑结构

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

## Linux / 系统

| 笔记 | 说明 |
|------|------|
| [cgroup 限制内存](linux/cgroup-memory-limit.md) | cgroup v2 memory.max，限制进程内存使用 |
| [NUMA & numactl](linux/numa-movepages-numactl.md) | NUMA 拓扑，move_pages，numactl 绑核绑内存 |

## Python / Web

| 笔记 | 说明 |
|------|------|
| [Uvicorn](python/uvicorn.md) | ASGI 服务器，ASGI vs WSGI，生产部署 |
