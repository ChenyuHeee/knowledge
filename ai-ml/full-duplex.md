---
date: 2026-06-07
tags: [LLM推理, 实时交互, 流式处理, 多模态]
---

# 全双工（Full Duplex）

> 相关笔记：[Speculative Decoding](transformer/speculative-decoding.md) | [Time Chunk](diffusion/time-chunk.md) | [流式视频](diffusion/streaming-video.md)

## 通信层面：原始定义

| 模式 | 说明 | 例子 |
|------|------|------|
| 半双工 | 同时只能一方发 | 对讲机 |
| **全双工** | 双方同时收发 | 电话 |

物理层需两路独立信道（发送 + 接收）。

## AI 交互层面

### 半双工（当前 ChatGPT）

```
你说完 → 模型听完 → 模型生成 → 你等
不能打断，不能插话
```

### 全双工（GPT-4o）

```
你说的同时模型在听 → 停顿瞬间模型插话 → 模型说话时你能打断
```

## 技术难点

### 打破串行流水线

半双工：`ASR → 攒一句话 → LLM → TTS → 播放`（等）

全双工：`音频流 chunk(0.2s) → ASR → LLM → TTS → 音频 chunk`（并行流转）

### 五项能力

| 能力 | 为什么难 |
|------|---------|
| 流式 ASR | 不能等句尾，每个 chunk 都要出结果 |
| 流式 LLM | 边收 token 边生成，不等 prompt 完整 |
| Barge-in | 生成中检测新输入 → 中止 → 切换上下文 |
| VAD | 区分「在说」「说完等回复」「噪音」 |
| Turn-taking | 200ms 内决定插话还是等 |

### GPT-4o 的做法

端到端多模态模型：音频直接进、音频直接出，无中间文本转换。200ms turn-taking 决策，接近人类反应速度。

## 与之前知识的联系

```
Spec Dec: Token 推理并行化
Time Chunk: 视频 Attention 分块
Full Duplex: 交互流水线 Chunk 化
          ↑ 都是把串行长步骤打散成并行小 Chunk
```
