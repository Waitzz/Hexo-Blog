---
title: Linux perf 绘制火焰图
date: 2022-09-07 22:14:02
categories:
- Linux
tags:
- perf
---

## Linux perf 绘制火焰图

---

### 编译perf工具

perf 源码在内核源码目录下 tools/perf 文件夹，首先需要声明编译器及架构平台：

```shell
export ARCH=arm
export CROSS_COMPILE=arm-linux-gnueabihf-
```

在 tools/perf 下执行编译命令：

```shell
make clean
make
make LDFLAGS=-static   #编译静态文件
```

### 使用perf工具

将 perf 文件放到板子上运行：

```shell
./perf record -a -g $(cmd) &
```

* -a : 分析整个系统的性能

* -g : 记录函数间的调用关系

* -o : 指定输出文件，未指定则在当前目录下生成 perf.data 文件

在开发板上执行：

```shell
./perf script -i perf.data > perf_data
```

解析 perf.data 文件并将结果存入文件 perf_data。将文件 perf_data 文件传至 Linux 服务器。下载火焰图生成工具：

```shell
git clone "https://github.com/brendangregg/FlameGraph"
```

在 Linux 服务器上执行：

```shell
./FlameGraph/stackcollapse-perf.pl perf_data > perf.folder
./FlameGraph/flamegraph.pl perf.folder > perf.svg
```

使用浏览器打开 perf.svg 文件即可查看火焰图：

![](Linux%20perf%20绘制火焰图/2022-08-19-19-55-52-image.png)


