---
title: RISCV操作系统-CH02-RISC-V-ISA
date: 2025-04-13 10:47:54 +08:00
filename: 2025-04-13-EulixOS-Pre-CH02
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
# 第2章 RISC-V ISA 介绍

## 目录

- [ISA的基本介绍](2025-04-13-EulixOS-Pre-CH02.md#isa的基本介绍)
    - [ISA是什么](2025-04-13-EulixOS-Pre-CH02.md#isa是什么)
    - [为什么要ISA](2025-04-13-EulixOS-Pre-CH02.md#为什么要isa)
    - [CISC vs RISC](2025-04-13-EulixOS-Pre-CH02.md#cisc-vs-risc)
    - [ISA的宽度](2025-04-13-EulixOS-Pre-CH02.md#isa的宽度)
    - [知名ISA介绍](2025-04-13-EulixOS-Pre-CH02.md#知名isa介绍)
    - [自由与开放](2025-04-13-EulixOS-Pre-CH02.md#自由与开放)
- [RISC-V ISA基本介绍](2025-04-13-EulixOS-Pre-CH02.md#risc-v-isa基本介绍)
    - [RISC-V的历史简介](2025-04-13-EulixOS-Pre-CH02.md#risc-v的历史简介)
    - [RISC-V究竟是什么](2025-04-13-EulixOS-Pre-CH02.md#risc-v究竟是什么)
    - [RISC-V发展现状](2025-04-13-EulixOS-Pre-CH02.md#risc-v发展现状)
    - [RISC-V的特点](2025-04-13-EulixOS-Pre-CH02.md#risc-v的特点)
    - [RISC-V ISA规范一览](2025-04-13-EulixOS-Pre-CH02.md#risc-v-isa规范一览)
    - [RISC-V ISA命名规范](2025-04-13-EulixOS-Pre-CH02.md#risc-v-isa命名规范)
    - [模块化的ISA](2025-04-13-EulixOS-Pre-CH02.md#模块化的isa)
    - [通用寄存器](2025-04-13-EulixOS-Pre-CH02.md#通用寄存器)
    - [Hart](2025-04-13-EulixOS-Pre-CH02.md#hart)
    - [特权级别](2025-04-13-EulixOS-Pre-CH02.md#特权级别)
    - [控制和状态寄存器](2025-04-13-EulixOS-Pre-CH02.md#控制和状态寄存器)
    - [内存管理与保护](2025-04-13-EulixOS-Pre-CH02.md#内存管理与保护)
    - [异常和中断](2025-04-13-EulixOS-Pre-CH02.md#异常和中断)

## 参考资料

- **参考1**：The RISC-V Instruction Set Manual，Volume I: Unprivileged ISA，Document Version 20191213
- **参考2**: The RISC-V Instruction Set Manual，Volume II: Privileged Architecture，Document Version 20190608-Priv-MSU-Ratified
- **参考3**：RISC-V手册 (中文版)：http://riscvbook.com/chinese/

## ISA的基本介绍

### ISA是什么

**ISA (Instruction Set Architecture) 指令集架构**：是底层硬件电路面向上层软件程序提供的一层接口规范。

ISA定义了：

- **基本数据类型**：BYTE/HALFWORD/WORD/......
- **寄存器 (Register)**
- **指令**
- **寻址模式**
- **异常或者中断的处理方式**
- **等等......**

ISA处于计算机系统分层结构的重要位置，位于应用程序和操作系统之下，微架构之上，是软件和硬件之间的分界线。

### 为什么要ISA

ISA的主要作用是：

- **为上层软件提供一层抽象**，制定规则和约束，让编程者不用操心具体的电路结构。
- 允许硬件设计者在不影响软件兼容性的情况下优化底层实现。

**IBM 360** 是第一个将ISA与其实现分离的计算机，在计算机发展史上具有重要意义。

### CISC vs RISC

**CISC (Complex Instruction Set Computing) 复杂指令集**：

- 针对特定的功能实现特定的指令，导致指令数目比较多，但生成的程序长度相对较短。
- 典型代表：x86架构

**RISC (Reduced Instruction Set Computing) 精简指令集**：

- 只定义常用指令，对复杂的功能采用常用指令组合实现，这导致指令数目比较精简，但生成的程序长度相对较长。
- 典型代表：ARM, RISC-V

> 注意：现如今，RISC和CISC也逐渐有相互融合的趋势，界限不再那么明显。

### ISA的宽度

**ISA (处理器) 的宽度**指的是CPU中**通用寄存器**的宽度（二进制的位数），这决定了寻址范围的大小以及数据运算的能力。

|通用寄存器的宽度|寻址范围|应用场景|
|---|---|---|
|8位|2^8 = 256|早期的单片机8051|
|16位|2^16 = 65536|X86系列的鼻祖8086，MSP430系列单片机|
|32位|2^32 = 4294967296|早期的终端，个人计算机和服务器|
|64位|2^64 = 18446744073709551616|目前主流的移动智能终端，个人计算机和服务器|

> 注意一个问题：ISA的宽度和指令编码长度无关。这是一个常见的误解。

### 知名ISA介绍

目前业界知名的ISA包括：

- **X86**：Intel和AMD产品采用的ISA，用于个人电脑和服务器
- **SPARC**：Sun公司开发的精简指令集架构
- **Power**：IBM开发的ISA
- **ARM**：ARM公司开发的精简指令集架构，广泛用于移动设备
- **MIPS**：MIPS公司开发的精简指令集架构
- **RISC-V**：开源指令集架构
- 等等......

### 自由与开放

对比软件业界的自由与开放标准：

|领域|开放的标准|开源的实现|闭源的实现|
|---|---|---|---|
|操作系统|POSIX|Linux，FreeBSD，......|Windows, ......|
|编译器|C|Gcc, LLVM, ......|Intel icc, ARMcc,......|
|数据库|SQL|MySQL,......|Oracle, DB2, ......|
|ISA|???|......|X86, ARM, ......|

在ISA领域，开放标准和开源实现一直是个空白，直到RISC-V的出现。这引发了对"ISA的未来在哪里？"这一问题的思考。

## RISC-V ISA基本介绍

### RISC-V的历史简介

**RISC-V** 念作 "risk-five"，代表着Berkeley所研发的第五代精简指令集。

该项目2010年始于加州大学伯克利（Berkeley）分校，希望选择一款ISA用于科研和教学。经过前期多年的研究和选型，最终决定放弃使用现成的X86和ARM等ISA，而是自己从头研发一款：

- **X86**：太复杂，IP问题
- **ARM**：一样的复杂，而且在2010年之前还不支持64位，以及同样的IP问题。

主要研发人员：

- Andrew Waterman, Yunsup Lee, David Patterson, Krste Asanovic

### RISC-V究竟是什么

RISC-V是：

- **一款高质量，免许可证，开放的RISC ISA**
- **一套由非营利的RISC-V基金会维护的标准**: https://riscv.org/
- **适用于所有类型的计算系统**：从微控制器到超级计算机

RISC-V不是一家公司，也不是一款CPU实现。它是一个开放的指令集架构规范，任何人都可以基于这个规范设计和实现处理器。

### RISC-V发展现状

根据分析机构Semico Research的报告：

- 预计到2025年，市场将总共消费624亿个RISC-V CPU内核
- 工业领域将是最大的细分市场，拥有167亿个内核
- 在各个市场领域，RISC-V CPU内核的复合年增长率(CAGR)在2018年至2025年之间将高达146.2%

截至PPT制作时(2021年)，RISC-V成员已超过700个，分布在全球50个国家，覆盖了芯片设计、系统集成、研究机构等多个领域。

### RISC-V的特点

RISC-V的主要特点包括：

- **简单**：设计清晰，容易实现
- **清晰的分层设计**：基础指令集与扩展模块分离
- **模块化**：可以根据需要选择不同的指令集扩展
- **稳定**：基础指令集一旦确定，永远不会改变
- **社区化**：开源社区驱动，避免单一公司控制

### RISC-V ISA规范一览

官方ISA标准下载地址(截至2021/3)： https://riscv.org/technical/specifications/

主要包括两个核心文档：

- Volume 1, Unprivileged Spec v. 20191213
- Volume 2, Privileged Spec v. 20190608

### RISC-V ISA命名规范

**ISA命名格式**：RV[###][abc.....xyz]

其中：

- **RV**：用于标识RISC-V体系架构的前缀，即RISC-V的缩写。
- **[###]**：{32, 64, 128} 用于标识处理器的字宽，也就是处理器的寄存器的宽度（单位为bit）。
- **[abc...xyz]**：标识该处理器支持的指令集模块集合。

例子：

- **RV32IMA**：32位RISC-V处理器，支持基本整数指令集(I)、整数乘除法(M)和原子操作(A)
- **RV64GC**：64位RISC-V处理器，支持通用指令集(G = IMAFD)和压缩指令集(C)

### 模块化的ISA

RISC-V与传统**增量ISA**（如x86）不同，采用了**模块化ISA**的设计方法：

**增量ISA**：计算机体系结构的传统方法，同一个体系架构下的新一代处理器不仅实现了新的ISA扩展，还必须实现过去的所有扩展，目的是为了保持向后的二进制兼容性。典型的，以80x86为代表。

**模块化ISA**：由1个基本整数指令集 + 多个可选的扩展指令集组成。基础指令集是固定的，永远不会改变。

**基本整数(Integer)指令集**是唯一强制要求实现的基础指令集，其他指令集都是可选的扩展模块：

|基本指令集|描述|
|---|---|
|RV32I|32位整数指令集|
|RV32E|RV32I的子集，用于小型的嵌入式场景|
|RV64I|64位整数指令集，兼容RV32I|
|RV128I|128位整数指令集，兼容RV64I和RV32I|

**扩展模块指令集**：

|扩展指令集|描述|
|---|---|
|M|整数乘法(Multiplication)与除法指令集|
|A|存储器原子(Atomic)指令集|
|F|单精度(32bit)浮点(Float)指令集|
|D|双精度(64bit)浮点(Double)指令，兼容F|
|C|压缩(Compressed)指令集|
|......|其他标准化和非标准化的指令集|

特定组合"IMAFD"被称为"通用(General)"组合，用英文字母G表示。

例子：

- **RV32I**：最基本的RISC-V实现
- **RV32IMAC**：32位实现，支持Integer + Multiply + Atomic + Compressed
- **RV64GC**：64位实现，支持IMAFD(G) + Compressed(C)

### 通用寄存器

RISC-V的Unprivileged Specification定义了32个通用寄存器以及一个PC（程序计数器）：

- 对RV32I/RV64I/RV128I都一样
- 如果实现支持F/D扩展则需要额外支持32个浮点(Float Point)寄存器
- RV32E将32个通用寄存器缩减为16个

**寄存器的宽度由ISA指定**：

- RV32的寄存器宽度为32位
- RV64的寄存器宽度为64位
- 依次类推

每个寄存器具体编程时有特定的用途以及各自的别名，由RISC-V Application Binary Interface (ABI)定义。

### Hart

**HART = HARdware Thread**，指硬件线程。

在RISC-V架构中，hart是一个独立的指令执行流，包含自己的程序计数器和寄存器组。一个处理器可以包含多个hart，类似于多核或超线程技术。

根据规范定义：

> "From the perspective of software running in a given execution environment, a hart is a resource that autonomously fetches and executes RISC-V instructions within that execution environment."

### 特权级别

RISC-V的Privileged Specification定义了三个特权级别(privilege level)：

|Level|Encoding|Name|Abbreviation|
|---|---|---|---|
|0|00|User/Application|U|
|1|01|Supervisor|S|
|2|10|Reserved|-|
|3|11|Machine|M|

其中：

- **Machine级别**是最高的级别，所有的RISC-V实现都需要支持
- 此外还有可选的**Debug级别**

特权级别的组合有不同的使用场景：

|Number of levels|Supported Modes|Intended Usage|
|---|---|---|
|1|M|Simple embedded systems|
|2|M, U|Secure embedded systems|
|3|M, S, U|Systems running Unix-like operating systems|

### 控制和状态寄存器

**CSR (Control and Status Registers)** 用于控制处理器的工作状态：

- 不同的特权级别下分别对应各自的一套CSR
- 高级别的特权级别下可以访问低级别的CSR，譬如Machine Level下可以访问Supervisor/User Level的CSR；但反之不可以
- RISC-V定义了专门用于操作CSR的指令（"Zicsr"扩展）
- RISC-V定义了特定的指令可以用于在不同特权级别之间进行切换（ECALL/EBREAK）

### 内存管理与保护

RISC-V提供两种内存管理机制：

**物理内存保护 (Physical Memory Protection, PMP)**：

- 允许M模式指定U模式可以访问的内存地址
- 支持R/W/X权限控制，以及Lock机制
- 适用于简单的嵌入式系统

**虚拟内存 (Virtual Memory)**：

- 需要支持Supervisor Level
- 用于实现高级的操作系统特性(Unix/Linux)
- 提供多种映射方式：Sv32/Sv39/Sv48
- 适用于复杂的计算机系统

### 异常和中断

RISC-V区分了两种控制流转移机制：

**异常 (Exception)**：

> "an unusual condition occurring at run time associated with an instruction in the current RISC-V hart"

异常是由当前指令执行过程中的异常条件引起的，属于同步事件。

**中断 (Interrupt)**：

> "an external asynchronous event that may cause a RISC-V hart to experience an unexpected transfer of control"

中断是由外部异步事件引起的，与当前执行的指令无关，属于异步事件。

这两种机制的处理方式不同，但都会导致控制流的转移，通常会跳转到相应的处理程序。