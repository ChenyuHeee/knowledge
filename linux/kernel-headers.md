---
date: 2026-06-15
tags: [Linux, 内核, GPU驱动, 运维]
---

# 内核头文件（Kernel Headers）

> 装 GPU 驱动时必须编译内核模块，需要内核头文件。

## 是什么

Linux 内核 API 的 C 头文件集合。定义内核函数、数据结构、宏。编译内核模块时 `#include` 的入口。

## 为什么装 GPU 驱动需要

NVIDIA 驱动包含一个 `.ko` 内核模块，需要编译：

```
NVIDIA 安装程序 → 编译 nvidia.ko → 需要内核头文件 → 没装就报错退出
```

内核模块负责：管理 PCIe 设备、分配 DMA 缓冲区、处理中断。

## 操作

```bash
uname -r                      # 看内核版本
sudo apt install linux-headers-$(uname -r)  # 装匹配的头文件
ls /usr/src/linux-headers-$(uname -r)      # 确认
sudo sh NVIDIA-Linux-*.run                 # 再跑安装
```

## 为什么必须版本精确匹配

内核内部 API 无 ABI 稳定保证。小版本变化可能多了结构体字段。错版本 → 编译出来的 `.ko` 加载时 kernel panic。
