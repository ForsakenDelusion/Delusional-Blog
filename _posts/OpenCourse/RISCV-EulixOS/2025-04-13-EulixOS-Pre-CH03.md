---
title: RISCV操作系统-CH03-编译与链接
date: 2025-04-13 10:55:11 +08:00
filename: 2025-04-13-EulixOS-Pre-CH03
categories:
  - OpenCourse
  - EulixOS
tags:
  - EulixOS
  - RISCV
  - OS
dir: OpenCourse/RISCV-EulixOS
share: true
---
# 第3章 编译与链接

## 目录

- [GCC介绍](2025-04-13-EulixOS-Pre-CH03.md/#gcc介绍)
    - [GCC简介](2025-04-13-EulixOS-Pre-CH03.md/#gcc简介)
    - [GCC的命令格式](2025-04-13-EulixOS-Pre-CH03.md/#gcc的命令格式)
    - [GCC的主要执行步骤](2025-04-13-EulixOS-Pre-CH03.md/#gcc的主要执行步骤)
    - [GCC涉及的文件类型](2025-04-13-EulixOS-Pre-CH03.md/#gcc涉及的文件类型)
    - [针对多个源文件的处理](2025-04-13-EulixOS-Pre-CH03.md/#针对多个源文件的处理)
- [ELF介绍](2025-04-13-EulixOS-Pre-CH03.md/#elf介绍)
    - [ELF简介](2025-04-13-EulixOS-Pre-CH03.md/#elf简介)
    - [ELF文件格式](2025-04-13-EulixOS-Pre-CH03.md/#elf文件格式)
    - [ELF文件处理相关工具：Binutils](2025-04-13-EulixOS-Pre-CH03.md/#elf文件处理相关工具：binutils)
- [学习编译和链接的好处](2025-04-13-EulixOS-Pre-CH03.md/#学习编译和链接的好处)

## GCC介绍

### GCC简介

**GCC（GNU Compiler Collection）**是一个功能强大的编译器套件，具有以下特点：

- 官方网址：https://gcc.gnu.org/
- 由GNU开发的，遵循GPL许可证发行的编译器套件
- 支持多种编程语言，包括C、C++、Objective-C、Fortran、Ada和Go语言等
- 已被移植到多种计算机体系架构上，如x86、ARM、RISC-V等
- GCC的初衷是为GNU操作系统专门编写一款编译器，现已被大多数"Unix-like"操作系统（如Linux、BSD、MacOS等）采纳为标准的编译器

### GCC的命令格式

GCC的基本命令格式如下：

```bash
gcc [options] [filenames]
```

常用的命令行选项包括：

|常用选项|含义|
|---|---|
|-E|只做预处理|
|-c|只编译不链接，生成目标文件".o"|
|-S|生成汇编代码|
|-o file|把输出生成到由file指定文件名的文件中|
|-g|在输出的文件中加入支持调试的信息|
|-v|显示输出详细的命令执行过程信息|

### GCC的主要执行步骤

GCC编译程序的主要步骤包括：

1. **编译（cc1）**：
    
    - 针对C语言，不同的语言有自己的编译器
    - 编译器完成"预处理"和"编译"两个阶段
    - "预处理"指处理源文件中以"#"开头的预处理指令，如#include、#define等
    - "编译"则针对预处理的结果进行一系列的词法分析、语法分析、语义分析，优化后生成汇编指令
    - 示例命令：`gcc -E foo.c -o foo.i`（预处理）和`gcc -S foo.i -o foo.s`（编译）
2. **汇编（as）**：
    
    - 汇编器将汇编语言代码转换为机器（CPU）可以执行的指令
    - 生成目标文件（.o文件）
    - 示例命令：`gcc -c foo.s -o foo.o`
3. **链接（ld）**：
    
    - 链接器将汇编器生成的目标文件和一些标准库（如libc）文件组合
    - 形成最终可执行的应用程序
    - 示例命令：`gcc foo.o -o a.out`

### GCC涉及的文件类型

GCC编译过程中涉及多种文件类型：

- **.c**：C源文件
- **.cc/.cxx/.cpp**：C++源文件
- **.i**：经过预处理的C源文件
- **.s/.S**：汇编语言源文件
- **.h**：头（header）文件
- **.o**：目标（object）文件
- **.a/.so**：编译后的静态库（archive）文件和共享库（shared object）文件
- **a.out**：可执行文件

>`.o`是目标文件，`.out`是可执行文件，不要混淆了

### 针对多个源文件的处理

当一个项目包含多个源文件时，GCC处理流程如下：

1. 首先，分别编译各个源文件生成对应的目标文件：
    
    ```bash
    gcc -c 1.c -o 1.o
    gcc -c 2.c -o 2.o
    gcc -c 3.c -o 3.o
    ```
    
2. 然后，将所有目标文件链接成最终的可执行文件：
    
    ```bash
    gcc 1.o 2.o 3.o -o a.out
    ```
    

这个过程可以用下图表示：

```
1.c → 1.i → 1.s → 1.o ─┐
                         │
2.c → 2.i → 2.s → 2.o ─┼─→ a.out
                         │
3.c → 3.i → 3.s → 3.o ─┘
```

## ELF介绍

### ELF简介

**ELF（Executable Linkable Format）**是一种Unix-like系统上的二进制文件格式标准。

ELF标准中定义的采用ELF格式的文件分为4类：

|ELF文件类型|说明|实例|
|---|---|---|
|可重定位文件（Relocatable File）|内容包含了代码和数据，可以被链接成可执行文件或共享目标文件|Linux上的.o文件|
|可执行文件（Executable File）|可以直接执行的程序|Linux上的a.out|
|共享目标文件（Shared Object File）|内容包含了代码和数据，可以作为链接器的输入，在链接阶段和其他的Relocatable File或者Shared Object File一起链接成新的Object File；或者在运行阶段，作为动态链接器的输入，和Executable File结合，作为进程的一部分来运行|Linux上的.so|
|核心转储文件（Core Dump File）|进程意外终止时，系统可以将该进程的部分内容和终止时的其他状态信息保存到该文件中以供调试分析|Linux上的core文件|

### ELF文件格式

ELF文件格式支持两种视图：

1. **链接视图**：
    
    - 以节（Section）为单位组织
    - 主要用于链接过程
    - 包含Section Header Table，描述各个节的信息
2. **运行视图**：
    
    - 以段（Segment）为单位组织
    - 主要用于程序加载和运行
    - 包含Program Header Table，描述各个段的信息

>为了增加空间利用率，可以将多个section合并成一个segment，放在一个分页中。

ELF文件的整体结构包括：

- ELF Header：描述整个文件的基本信息
- Program Header Table：描述各段的信息（运行视图）
- 多个Section（.text、.init、.data、.bss等）
- Section Header Table：描述各节的信息（链接视图）

> 注意：Section是为链接器服务的，Segment是为加载器服务的。通常多个Section会被合并到一个Segment中，以优化程序加载和运行效率。

>segment fault：访问了不允许被访问的segment


### ELF文件处理相关工具：Binutils

**Binutils**是一套用于处理二进制文件的工具集，官网：https://www.gnu.org/software/binutils/

主要工具包括：

- **ar**：归档文件，将多个文件打包成一个大文件
- **as**：被gcc调用，输入汇编文件，输出目标文件供链接器ld连接
- **ld**：GNU链接器，被gcc调用，它把目标文件和各种库文件结合在一起，重定位数据，并链接符号引用
- **objcopy**：执行文件格式转换
- **objdump**：显示ELF文件的信息
- **readelf**：显示更多ELF格式文件的信息（包括DWARF调试信息）
- 以及其他工具

## 学习编译和链接的好处

学习和掌握编译与链接知识对程序员有很多好处：

1. **有利于优化程序性能**：
    
    - 理解控制结构的实现差异（如switch vs if-else）
    - 了解函数调用的开销和参数传递的优化（传变量还是指针）
    - 能够做出更明智的性能相关决策
2. **理解并解决编译链接时出现的错误**：
    
    - 常见错误如"error: XXXXXX redefined"
    - "error: cannot find XXXXXX"
    - 能够快速定位和修复编译链接问题
3. **写出更健壮的程序**：
    
    - 避免缓冲区溢出
    - 防止非法内存访问
    - 减少"Segmentation fault"等运行时错误
    - 提高程序的可靠性和安全性