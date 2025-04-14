---
title: RISCV操作系统-CH04-嵌入式开发介绍
date: 2025-04-13 11:00:11 +08:00
filename: 2025-04-13-EulixOS-Pre-CH04
categories:
  - OpenCourse
  - EulixOS
tags:
  - EulixOS
  - OS
  - RISCV
dir: OpenCourse/RISCV-EulixOS
share: true
---
# 第4章 嵌入式开发介绍

## 目录

- [什么是嵌入式开发](#什么是嵌入式开发)
- [交叉编译](#交叉编译)
    - [交叉编译的定义](#交叉编译的定义)
    - [编译系统分类](#编译系统分类)
    - [GNU交叉编译工具链](#gnu交叉编译工具链)
- [调试器GDB](#调试器gdb)
    - [GDB简介](#gdb简介)
    - [GDB基本调试流程](#gdb基本调试流程)
- [模拟器QEMU](#模拟器qemu)
    - [QEMU简介](#qemu简介)
    - [QEMU的安装和使用](#qemu的安装和使用)
- [项目构造工具Make](#项目构造工具make)
    - [Make简介](#make简介)
    - [Makefile的构成](#makefile的构成)
    - [make的运行](#make的运行)

## 什么是嵌入式开发

**嵌入式开发**是一种比较综合性的技术，它不单指纯粹的软件开发技术，也不单是一种硬件配置技术；它是在特定的硬件环境下针对某款硬件进行开发，是一种系统级别的与硬件结合比较紧密的软件开发技术。

嵌入式开发通常涉及以下几个组成部分：

- 目标板（Target Board）：运行嵌入式系统的硬件板
- 主机（Host PC）：用于开发程序的计算机
- 连接方式：如串口（Serial）或以太网（Ethernet）
- 网络设备：如路由器（Router）
- 互联网（Internet）：可选组件，用于远程访问和更新

> 嵌入式系统的开发过程通常需要考虑资源约束、稳定性、实时性等特殊要求，与常规软件开发有很大不同。

## 交叉编译

### 交叉编译的定义

**交叉编译**是指在一个平台上生成另一个平台上的可执行代码的过程。在嵌入式开发中，交叉编译是必不可少的技术，因为目标硬件通常与开发环境使用不同的指令集架构。

### 编译系统分类

参与编译和运行的机器根据其角色可以分成以下三类：

- **构建（build）系统**：执行编译构建动作的计算机。
- **主机（host）系统**：运行build系统生成的可执行程序的计算机系统。
- **目标（target）系统**：特别地，当以上生成的可执行程序是GCC时，我们用target来描述用来运行GCC将生成的可执行程序的计算机系统。

根据build/host/target的不同组合我们可以得到如下的编译方式分类：

- **本地（native）编译**：build == host == target
- **交叉（cross）编译**：build == host != target

![RISCV操作系统-CH04-20250413-1.png](../../../assets/images/RISCV%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F-CH04-20250413-1.png)

### GNU交叉编译工具链

**GNU交叉编译工具链（Toolchain）**是进行交叉编译的工具集合。

- **命名格式**：arch-vendor-os1-\[os2-]XXX
- **例子**：
    - x86_64-linux-gnu-gcc
    - riscv64-unknown-elf-gcc
    - riscv64-unknown-elf-objdump

在典型的交叉编译场景中，开发者通常在x86_64架构的Linux系统上编译RISC-V架构的程序，编译出的程序将在RISC-V硬件上运行。

## 调试器GDB

### GDB简介

**GDB (GNU Project Debugger)** 是GNU项目调试器，用于查看另一个程序在执行过程中正在执行的操作，或该程序崩溃时正在执行的操作。

- 官方网址：https://www.gnu.org/software/gdb/
- 被调试的程序可能与GDB在同一台计算机上执行，也可能在另一台计算机（远程）上或者在模拟器上执行。
- GDB支持调试多种语言：如Assembly、C、Go、Rust等。

GDB支持两种主要的调试模式：

1. **本地调试**：调试器和被调试的程序在同一系统上运行
2. **远程调试**：被调试程序在远程系统或模拟器上运行，调试器通过网络连接控制程序

![RISCV操作系统-CH04-20250413.png](../../../assets/images/RISCV%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F-CH04-20250413.png)

>提到了ptrace的概念

### GDB基本调试流程

GDB的基本调试流程如下：

1. **重新编译程序并在编译选项中加入 "-g"**
    
    ```bash
    $ gcc -g test.c
    ```
    
2. **运行gdb和程序**
    
    ```bash
    $ gdb a.out
    ```
    
3. **设置断点**
    
    ```bash
    (gdb) b 6
    ```
    
4. **运行程序**
    
    ```bash
    (gdb) r
    ```
    
5. **程序暂停在断点处，执行查看**
    
    ```bash
    (gdb) p xxx
    ```
    
6. **继续、单步或者恢复程序运行**
    
    ```bash
    (gdb) s/n/c
    ```
    

> 注意：`s`代表step（单步进入）, `n`代表next（单步执行）, `c`代表continue（继续执行）。

## 模拟器QEMU

### QEMU简介

**QEMU**是一套由Fabrice Bellard编写的以GPL许可证分发源码的计算机系统模拟软件，在GNU/Linux平台上使用广泛。

- 官方网址：https://www.qemu.org/
- 支持多种体系架构：如IA-32 (x86)、AMD 64、MIPS 32/64、RISC-V 32/64等。
- QEMU有两种主要运作模式：
    - **User mode**：直接运行应用程序。
    - **System mode**：模拟整个计算机系统，包括中央处理器及其他周边设备。

### QEMU的安装和使用

**安装方法**：

- Ubuntu上使用apt安装：`apt install qemu`
- 源码编译安装

**基本用法**：

```bash
qemu-system-riscv32 ... -kernel ./test.elf
```

**调试模式启动**：

```bash
qemu-system-riscv32 ... -kernel ./test.elf -s -S
```

- `-s`："-gdb tcp::1234"的缩写，启动gdbserver并在1234端口号上监听客户端
- `-S`：在启动时停止CPU（只有到在客户端键入'c'才会开始执行）

## 项目构造工具Make

### Make简介

**Make**是一种自动化工程管理工具。

- 官方网址：https://www.gnu.org/software/make/
- **Makefile**是配合make使用的文件，用于描述构建工程过程中所管理的对象以及如何构造工程的过程。
- make如何找到Makefile：
    - **隐式查找**：当前目录下按顺序找寻文件名为"GNUmakefile"、"makefile"、"Makefile"的文件
    - **显式查找**：使用`-f`选项指定Makefile文件

### Makefile的构成

**Makefile由一条或者多条规则（rule）组成**。每条规则由三要素构成：

1. **target**：目标，可以是obj文件也可以是可执行文件
2. **prerequisites**：生成target所需要的依赖
3. **command**：为了生成target需要执行的命令，可以有多条

一个简单的Makefile规则如下:

```makefile
target...:prerequisites...
	command...
	...
```

> 注意：command前必须是一个TAB，而不是空格。

示例：

```makefile
hello: hello.c
	gcc hello.c -o hello
```

Makefile中的其他元素包括：

- **缺省规则**：
    
    ```makefile
    .DEFAULT_GOAL := all
    all :
    ```
    
- **伪规则**：
    
    ```makefile
    .PHONY : clean
    clean:
        rm -f *.o
    ```
    
- **注释**：行注释，以"#"开头
    

### make的运行

make执行时会按照**目标依赖关系**决定构建顺序，从最基本的依赖开始，逐步构建，最终生成终极目标。

以一个完整的C程序编译为例，依赖关系如下：

```
hello.c → hello.i → hello.s → hello.o → a.out
```

对应的Makefile规则执行顺序：

1. `hello.i : hello.c`：预处理（gcc -E hello.c -o hello.i）
2. `hello.s : hello.i`：编译（gcc -S hello.i -o hello.s）
3. `hello.o : hello.s`：汇编（gcc -c hello.s -o hello.o）
4. `a.out : hello.o`：链接（gcc hello.o -o a.out）

make工具会自动识别这些依赖关系，并按照正确的顺序执行相应的构建命令。

> 当一个大型项目包含多个源文件和依赖关系时，make工具的自动化构建能力将极大提高开发效率。