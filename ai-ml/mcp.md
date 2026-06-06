---
date: 2026-06-06
tags: [LLM, MCP, 工具调用, 协议]
---

# MCP（Model Context Protocol，模型上下文协议）

Anthropic 提出的开放协议，标准化 LLM 与外部工具/数据源的连接。类比：AI 应用的 USB-C 接口。

## 问题

没有 MCP 时，每接一个外部工具都要写定制适配代码，N 个工具 × M 个 LLM 提供商 = N×M 的工程量。

## 架构

```
LLM ↔ MCP Client ↔ MCP Protocol (JSON-RPC) ↔ MCP Server ↔ 工具/数据
       (Host内)                                 (独立进程)
```

## 三个核心原语

| 原语 | 含义 | 例子 |
|------|------|------|
| **Tools** | 可调用函数 | `search_docs(query)`, `send_email(to, body)` |
| **Resources** | 可读取数据 | 文档内容、数据库 schema |
| **Prompts** | 预定义模板 | 「审查这段代码」「总结这个 PR」 |

Server 启动时声明能力，Client 告知 LLM，LLM 需要时通过 Client 调 Server。

## 调用流程（以 GitHub 为例）

```
1. Client 启动，连接 GitHub MCP Server
2. Server 声明 Tools: list_prs(), get_pr()
3. 用户:「看看最近的 PR」
4. LLM 决定调 list_prs() → Client 转发 → Server 执行
5. Server 从 GitHub 取数据 → 返回 LLM → 总结给用户
```

LLM 不需要知道 GitHub API 细节，所有封装在 MCP Server 里。

## vs Function Calling

| | Function Calling | MCP |
|------|----------|------|
| 定义 | 手写 JSON schema | Server 自动声明 |
| 执行 | 自己实现函数体 | Server 封装好 |
| 复用 | 每个应用重写 | 一个 Server 到处用 |
| 协议 | 各 LLM 格式不同 | 统一开放标准 |
| 生态 | 各自实现 | 社区共享 |

MCP 解决生态和复用：写一次 Server，所有 MCP 兼容的 Client 都能用。

## 生态

- Claude Desktop / Cursor / Windsurf 原生支持
- 社区几百个 MCP Server（GitHub、Slack、Postgres、Filesystem、Brave Search...）
- 官方 Python SDK + TypeScript SDK
