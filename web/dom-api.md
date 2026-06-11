---
date: 2026-06-11
tags: [Web, 前端, DOM, API, 浏览器]
---

# DOM 与 API

## DOM（Document Object Model）

浏览器把 HTML 解析成的**树形数据结构**。每个 HTML 标签是一个节点，JS 通过遍历和修改这棵树来改变页面：

```
HTML: <html><body><h1>标题</h1><p>文字</p></body></html>

DOM:
  document → html → body → h1 ("标题")
                         → p  ("文字")
```

常见操作（通过 DOM API）：

```javascript
document.querySelector('h1').textContent = '新标题'   // 改
document.querySelector('p').remove()                  // 删
document.createElement('div')                         // 增
```

## API（Application Programming Interface）

一个**通用概念**——任意两个软件之间的约定接口。

| 层次 | 例子 |
|------|------|
| 语言内置 | `array.push()`, `list.sort()` |
| 浏览器 | `fetch()`, `setTimeout()`, `document.*` |
| 后端服务 | `GET /api/users`, `POST /api/login` |

## 关系

DOM 是数据结构（浏览器内部用 C++ 维护一棵树），DOM API 是操作这棵树的接口（`document.querySelector()` 等）。

```
JS 代码 → DOM API (桥梁) → C++ DOM 树 → 重排/重绘 → 页面更新
```

「调 API」通常指 HTTP 后端 API，但 `document.*` 同样是 API，只是运行在浏览器里。
