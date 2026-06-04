# Uvicorn：Python ASGI 服务器

## 是什么

Python 异步 Web 框架（FastAPI / Starlette / Django）的 HTTP 服务器，基于 `uvloop`（Cython 重写的 asyncio + libuv），单进程可处理数千并发连接。

## 在 Python 生态中的位置

```
HTTP 请求
  → Uvicorn (ASGI 服务器)     ← 你问的这个
    → FastAPI / Starlette (Web 框架)
```

类比：Uvicorn 是 Tomcat，FastAPI 是 Spring Boot。

## ASGI vs WSGI

| | WSGI | ASGI |
|------|------|------|
| 模型 | 同步 | 异步 |
| 服务器 | Gunicorn | Uvicorn |
| 框架 | Flask, Django | FastAPI, Starlette |
| 协议 | HTTP/1.1 | HTTP/1.1, HTTP/2, WebSocket, SSE |

ASGI 支持长连接协议，WSGI 一个请求一个函数调用，函数返回就结束。

## 常用命令

```bash
uvicorn main:app --host 0.0.0.0 --port 8000          # 单进程
uvicorn main:app --workers 4                           # 多进程
uvicorn main:app --reload                              # 开发热重载
```

## 生产部署

```
Nginx → Uvicorn (多 worker) → FastAPI
```

或配合 Gunicorn 管理进程：

```bash
gunicorn main:app -k uvicorn.workers.UvicornWorker -w 4
```

Gunicorn 管进程生命周期（重启、优雅退出），Uvicorn 管每个进程内的异步并发。
