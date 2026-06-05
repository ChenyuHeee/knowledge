---
date: 2026-06-04
tags: [Linux, NUMA, numactl]
---
# NUMA、move_pages 与 numactl

## NUMA 概念

Non-Uniform Memory Access。多路服务器上每个 CPU 有本地内存，访问本地快、访问远端慢。

```
CPU 0 + 本地 DDR5 (Node 0) ←→ 慢 ←→ CPU 1 + 本地 DDR5 (Node 1)
```

`numactl --hardware` 可查看拓扑和 node distances。

## move_pages() 系统调用

将指定进程的内存页从一个 NUMA node 迁移到另一个。

```c
long move_pages(int pid, unsigned long count,
                void **pages, const int *nodes,
                int *status, int flags);
```

| 参数 | 含义 |
|------|------|
| pid | 目标进程 PID |
| pages | 虚拟地址数组 |
| nodes | 每页目标 node |
| flags | `MPOL_MF_MOVE` / `MPOL_MF_MOVE_ALL` |

一般不直接调用，而是通过 numactl 或 mbind 高层控制。

## numactl 命令行工具

控制进程的 NUMA 亲和性。

### 常用命令

```bash
numactl --hardware                           # 查看 NUMA 拓扑
numactl --membind=0 ./prog                   # 内存只从 node 0 分配
numactl --cpunodebind=0 --membind=0 ./prog   # CPU+内存都绑 node 0
numactl --preferred=0 ./prog                 # 优先 node 0，不够再 fallback
numactl --interleave=all ./prog              # 轮询分配（大数组顺序访问）
```

### 参数说明

| 参数 | 效果 |
|------|------|
| `--membind=nodes` | 内存只能从指定 node 分配 |
| `--cpunodebind=nodes` | 进程只能跑在指定 node 的 CPU 上 |
| `--preferred=node` | 优先指定 node，可 fallback |
| `--interleave=nodes` | 轮询均匀分配 |
| `--show` | 当前进程 NUMA 策略 |

## 实验场景

限制 DDR5 的两层含义：

1. **cgroup `memory.max`**：限制总量，防 OOM
2. **numactl `--membind`**：绑内存到本地 node，避免跨 node 访问导致性能波动

性能敏感实验建议两者配合使用。
