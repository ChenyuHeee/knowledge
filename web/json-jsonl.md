---
date: 2026-06-13
tags: [Web, 数据格式, JSON, 数据处理]
---

# JSON 与 JSONL

## JSON（JavaScript Object Notation）

用文本表示结构化数据，六种类型：`string`、`number`、`boolean`、`null`、`object`、`array`。

```json
{"name": "何宸禹", "age": 19, "courses": ["数据结构", "计算机组成"]}
```

直接映射到各语言原生结构（Python：dict+list，JS：object+array，Go：map+slice）。零配置。

## JSON vs XML

XML 标签成对冗余。JSON 简洁，和代码数据结构一一对应。JSON 胜出。

## JSONL（JSON Lines）

每行一个独立 JSON 对象：

```jsonl
{"id": 1, "text": "第一条数据"}
{"id": 2, "text": "第二条数据"}
```

| | JSON | JSONL |
|------|------|-------|
| 结构 | 完整对象/数组 | 每行独立 |
| 读取 | 一次性全部解析 | **逐行流式处理** |
| 追加 | 需重新序列化 | **尾部 append** |
| 内存 | 全部加载 | 可远超内存 |
| 用途 | API、配置文件 | **训练数据、日志** |

## 为什么 LLM 训练用 JSONL

- 逐行读，不需要全文件进内存
- 可 shard：10TB = 10000 个 `.jsonl`，worker 各自读
- 断点续传：崩在第 12345 行 → 重开从这行继续
