# cgroup 限制内存使用

> 相关笔记：[NUMA、move_pages 与 numactl](numa-movepages-numactl.md)

## 概念

cgroup（Control Group）是 Linux 内核的资源隔离机制。通过 cgroup v2 的 memory 控制器，可以用 `memory.max` 文件设置进程的硬内存上限，超出触发 OOM kill。

## 操作步骤

### 1. 确认 cgroup v2

```bash
stat -fc %T /sys/fs/cgroup
# 输出 cgroup2fs 即为 v2
```

### 2. 创建 cgroup 并设置限制

```bash
sudo mkdir /sys/fs/cgroup/myexp
echo "4G" | sudo tee /sys/fs/cgroup/myexp/memory.max
echo "0" | sudo tee /sys/fs/cgroup/myexp/memory.swap.max   # 禁用 swap
```

### 3. 放入进程运行

```bash
# 方式一：手动加入
echo $$ | sudo tee /sys/fs/cgroup/myexp/cgroup.procs
./your_experiment

# 方式二：systemd-run 一键启动（推荐）
systemd-run --user --scope -p MemoryMax=4G ./your_experiment
```

### 4. 验证

```bash
cat /sys/fs/cgroup/myexp/memory.current   # 当前用量
cat /sys/fs/cgroup/myexp/memory.max       # 设定的上限
```

### 5. 清理

```bash
sudo rmdir /sys/fs/cgroup/myexp
```

## 关键文件

| 文件 | 含义 |
|------|------|
| `memory.max` | 硬上限，超出 OOM kill |
| `memory.high` | 软上限，超出 throttle 但不杀进程 |
| `memory.low` | 保障下限，内存紧张时不低于此 |
| `memory.current` | 当前实际用量（只读） |
| `memory.swap.max` | swap 上限 |
| `cgroup.procs` | 写入 PID 加入控制 |

## 典型场景

- 跑实验限制内存，避免单个实验吃满整机
- 模拟低内存环境测试程序行为
- 多任务并行时各自有上限，互不干扰
