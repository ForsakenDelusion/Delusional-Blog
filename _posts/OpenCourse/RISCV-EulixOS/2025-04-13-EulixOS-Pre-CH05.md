---
title: RISCV操作系统-CH05-RISC-V-汇编
date: 2025-04-13 11:05:15 +08:00
filename: 2025-04-13-EulixOS-Pre-CH05
categories:
  - OpenCourse
  - EulixOS
tags:
  - RISCV
  - EulixOS
  - OS
dir: OpenCourse/RISCV-EulixOS
share: true
---
# 第5章 RISC-V 汇编语言编程

## 目录

- [RISC-V 汇编语言入门](#risc-v-汇编语言入门)
    - [汇编语言概念简介](#汇编语言概念简介)
    - [汇编语言语法介绍](#汇编语言语法介绍)
- [RISC-V 汇编指令总览](#risc-v-汇编指令总览)
    - [RISC-V 汇编指令操作对象](#risc-v-汇编指令操作对象)
    - [RISC-V 汇编指令编码格式](#risc-v-汇编指令编码格式)
    - [RISC-V 汇编指令分类](#risc-v-汇编指令分类)
    - [RISC-V 汇编伪指令一览](#risc-v-汇编伪指令一览)
- [RISC-V 汇编指令详解](#risc-v-汇编指令详解)
    - [算术运算指令](#算术运算指令)
    - [逻辑运算指令](#逻辑运算指令)
    - [移位运算指令](#移位运算指令)
    - [内存读写指令](#内存读写指令)
    - [条件分支指令](#条件分支指令)
    - [无条件跳转指令](#无条件跳转指令)
    - [RISC-V 指令寻址模式总结](#risc-v-指令寻址模式总结)
- [RISC-V 汇编函数调用约定](#risc-v-汇编函数调用约定)
    - [函数调用过程概述](#函数调用过程概述)
    - [汇编编程时为何需要制定函数调用约定](#汇编编程时为何需要制定函数调用约定)
    - [函数调用过程中有关寄存器的编程约定](#函数调用过程中有关寄存器的编程约定)
    - [函数调用过程中函数跳转和返回指令的编程约定](#函数调用过程中函数跳转和返回指令的编程约定)
    - [函数调用过程中实现被调用函数的编程约定](#函数调用过程中实现被调用函数的编程约定)
- [RISC-V 汇编与 C 混合编程](#risc-v-汇编与-c-混合编程)
    - [RISC-V 汇编调用 C 函数](#risc-v-汇编调用-c-函数)
    - [C 函数中嵌入 RISC-V 汇编](#c-函数中嵌入-risc-v-汇编)

## 参考文献

1. The RISC-V Instruction Set Manual, Volume I: Unprivileged ISA, Document Version 20191213
2. Using as: [https://sourceware.org/binutils/docs/as/](https://sourceware.org/binutils/docs/as/)
3. How to Use Inline Assembly Language in C Code: [https://gcc.gnu.org/onlinedocs/gcc/Using-Assembly-Language-with-C.html](https://gcc.gnu.org/onlinedocs/gcc/Using-Assembly-Language-with-C.html)

## RISC-V 汇编语言入门

### 汇编语言概念简介

**汇编语言**（Assembly Language）是一种"低级"语言，具有以下特点：

- **缺点**：
    - 难读
    - 难写
    - 难移植
- **优点**：
    - 灵活
    - 强大
- **应用场景**：
    - 需要直接访问底层硬件的地方
    - 需要对性能执行极致优化的地方

### 汇编语言语法介绍（GNU 版本）

>`.s`,` .S`都是汇编文件的后缀。区别是S包含宏的汇编文件，需要预处理。

一个完整的 RISC-V 汇编程序由多条**语句**（statement）组成。

一条典型的 RISC-V 汇编语句由 3 部分组成：

```
[label:] [operation] [comment]
```

- **label（标号）**: GNU汇编中，任何以冒号结尾的标识符都被认为是一个标号。
- **operation** 可以有以下多种类型：
    - **instruction（指令）**: 直接对应二进制机器指令的字符串
    - **pseudo-instruction（伪指令）**: 为了提高编写代码的效率，可以用一条伪指令指示汇编器产生多条实际的指令(instructions)
    - **directive（指示/伪操作）**: 通过类似指令的形式(以"."开头)，通知汇编器如何控制代码的产生等，不对应具体的指令
    - **macro**: 采用 .macro/.endm 自定义的宏
- **comment（注释）**: 常用方式，"#" 开始到当前行结束。

## RISC-V 汇编指令总览

### RISC-V 汇编指令操作对象

#### 寄存器

- 32个通用寄存器，x0 ~ x31（注意：本章节课程仅涉及 RV32I 的通用寄存器组）
- 在 RISC-V 中，Hart 在执行算术逻辑运算时所操作的数据必须直接来自寄存器

#### 内存

- Hart 可以执行在寄存器和内存之间的数据读写操作
- 读写操作使用字节（Byte）为基本单位进行寻址
- RV32 可以访问最多 2^32 个字节的内存空间

### RISC-V 汇编指令编码格式

- **指令长度**：ILEN1 = 32 bits (RV32I)
- **指令对齐**：IALIGN = 32 bits (RV32I)
- 32 个 bit 划分成不同的 "域（field）"
- funct3/funct7 和 opcode 一起决定最终的指令类型
- 指令在内存中按照**小端序**排列

#### 小端序的概念

- **主机字节序**（HBO - Host Byte Order）指一个多字节整数在计算机内存中存储的字节顺序
- 不同类型 CPU 的 HBO 不同，这与 CPU 的设计有关，分为**大端序**（Big-Endian）和**小端序**（Little-Endian）
- 大端序：数据的高位字节存放在内存的低地址
- 小端序：数据的低位字节存放在内存的低地址

### RISC-V 汇编指令编码格式

RISC-V 指令格式有 6 种：

|指令格式|说明|
|---|---|
|R-type|Register 格式，每条指令中有三个 fields，用于指定 3 个寄存器参数|
|I-type|Immediate 格式，每条指令除了带有两个寄存器参数外，还带有一个立即数参数（宽度为 12 bits）|
|S-type|Store 格式，每条指令除了带有两个寄存器参数外，还带有一个立即数参数（宽度为 12 bits，但 fields 的组织方式不同于 I-type）|
|B-type|Branch 格式，每条指令除了带有两个寄存器参数外，还带有一个立即数参数（宽度为 12 bits，但取值为 2 的倍数）|
|U-type|Upper 格式，每条指令含有一个寄存器参数再加上一个立即数参数（宽度为 20 bits，用于表示一个立即数的高 20 位）|
|J-type|Jump 格式，每条指令含有一个寄存器参数再加上一个立即数参数（宽度为 20 bits）|

### RISC-V 汇编指令分类

RISC-V 指令可以分为以下几类：

1. **算术运算指令**（如 ADD, SUB, ADDI 等）
2. **逻辑运算指令**（如 XOR, OR, AND 等）
3. **移位运算指令**（如 SLL, SRL, SRA 等）
4. **内存读写指令**（如 LB, LW, SB, SW 等）
5. **分支与跳转指令**（如 BEQ, BNE, JAL 等）

> 注：部分指令因篇幅原因未列出，如 Compare/Synch/Change Level 指令等。

### RISC-V 汇编伪指令一览

伪指令是为了提高编程效率，由汇编器转换成一条或多条实际指令的指令。RISC-V 常用伪指令包括：

|伪指令|等价指令|含义|
|---|---|---|
|LI RD, IMM|LUI 和 ADDI 的组合|将立即数 IMM 加载到 RD 中|
|LA RD, LABEL|AUIPC 和 ADDI 的组合|为 RD 加载一个地址值|
|NEG RD, RS|SUB RD, x0, RS|对 RS 中的值取反并将结果存放在 RD 中|
|MV RD, RS|ADDI RD, RS, 0|将 RS 中的值拷贝到 RD 中|
|NOP|ADDI x0, x0, 0|什么也不做|

## RISC-V 汇编指令详解

### 算术运算指令

#### ADD 指令

```
ADD RD, RS1, RS2
```

- **格式**：R-type
- **功能**：RD = RS1 + RS2
- **示例**：`add x5, x6, x7` 执行后 x5 = x6 + x7

#### SUB 指令

```
SUB RD, RS1, RS2
```

- **格式**：R-type
- **功能**：RD = RS1 - RS2
- **示例**：`sub x5, x6, x7` 执行后 x5 = x6 - x7

#### ADDI 指令

```
ADDI RD, RS1, IMM
```

- **格式**：I-type
- **功能**：RD = RS1 + IMM
- **示例**：`addi x5, x6, 1` 执行后 x5 = x6 + 1
- **注意**：IMM 是 12 位有符号数，范围 [-2048, 2047]，在运算前会进行符号扩展

#### LUI 指令（Load Upper Immediate）

```
LUI RD, IMM
```

- **格式**：U-type
- **功能**：将 20 位立即数左移 12 位（低 12 位置零）后写入 RD
- **示例**：`lui x5, 0x12345` 执行后 x5 = 0x12345000

#### AUIPC 指令（Add Upper Immediate to PC）

```
AUIPC RD, IMM
```

- **格式**：U-type
- **功能**：将 20 位立即数左移 12 位后与 PC 值相加，结果存入 RD
- **示例**：`auipc x5, 0x12345` 执行后 x5 = 0x12345000 + PC

#### 基于算术运算指令实现的伪指令

|伪指令|等价指令|功能|示例|
|---|---|---|---|
|LI RD, IMM|LUI+ADDI 组合|将任意 32 位立即数加载到寄存器|li x5, 0x12345678|
|MV RD, RS|ADDI RD, RS, 0|寄存器间的值拷贝|mv x5, x6|
|NEG RD, RS|SUB RD, x0, RS|对值取反|neg x5, x6|
|NOP|ADDI x0, x0, 0|空操作|nop|

### 逻辑运算指令

RISC-V 提供了以下逻辑运算指令：

|指令|格式|功能|示例|
|---|---|---|---|
|AND|R-type|按位与|and x5, x6, x7|
|OR|R-type|按位或|or x5, x6, x7|
|XOR|R-type|按位异或|xor x5, x6, x7|
|ANDI|I-type|立即数按位与|andi x5, x6, 20|
|ORI|I-type|立即数按位或|ori x5, x6, 20|
|XORI|I-type|立即数按位异或|xori x5, x6, 20|

基于逻辑运算的伪指令：

|伪指令|等价指令|功能|示例|
|---|---|---|---|
|NOT|XORI RD, RS, -1|按位取反|not x5, x6|

### 移位运算指令

RISC-V 提供以下移位运算指令：

#### 逻辑移位

|指令|格式|功能|示例|
|---|---|---|---|
|SLL|R-type|逻辑左移|sll x5, x6, x7|
|SRL|R-type|逻辑右移|srl x5, x6, x7|
|SLLI|I-type|立即数逻辑左移|slli x5, x6, 3|
|SRLI|I-type|立即数逻辑右移|srli x5, x6, 3|

#### 算术移位

|指令|格式|功能|示例|
|---|---|---|---|
|SRA|R-type|算术右移|sra x5, x6, x7|
|SRAI|I-type|立即数算术右移|srai x5, x6, 3|

> 注意：
> 
> 1. 逻辑移位时补充的位为 0
> 2. 算术右移时按照符号位值补足
> 3. 没有算术左移指令，因为逻辑左移就可以实现等效功能

### 内存读写指令

#### 内存读（Load）指令

|指令|格式|功能|示例|
|---|---|---|---|
|LB|I-type|从内存加载有符号字节到寄存器|lb x5, 40(x6)|
|LBU|I-type|从内存加载无符号字节到寄存器|lbu x5, 40(x6)|
|LH|I-type|从内存加载有符号半字（16位）到寄存器|lh x5, 40(x6)|
|LHU|I-type|从内存加载无符号半字（16位）到寄存器|lhu x5, 40(x6)|
|LW|I-type|从内存加载字（32位）到寄存器|lw x5, 40(x6)|

#### 内存写（Store）指令

|指令|格式|功能|示例|
|---|---|---|---|
|SB|S-type|将寄存器中的字节存储到内存|sb x5, 40(x6)|
|SH|S-type|将寄存器中的半字存储到内存|sh x5, 40(x6)|
|SW|S-type|将寄存器中的字存储到内存|sw x5, 40(x6)|

> 注意：
> 
> 1. Load 指令分为有符号和无符号两种，而 Store 指令不区分
> 2. 内存地址 = 基址寄存器 + 偏移量，偏移量范围是 [-2048, 2047]

### 条件分支指令

|指令|格式|功能|示例|
|---|---|---|---|
|BEQ|B-type|相等则分支|beq x5, x6, label|
|BNE|B-type|不相等则分支|bne x5, x6, label|
|BLT|B-type|小于则分支（有符号）|blt x5, x6, label|
|BLTU|B-type|小于则分支（无符号）|bltu x5, x6, label|
|BGE|B-type|大于等于则分支（有符号）|bge x5, x6, label|
|BGEU|B-type|大于等于则分支（无符号）|bgeu x5, x6, label|

条件分支指令的伪指令：

|伪指令|等价指令|功能|示例|
|---|---|---|---|
|BEQZ|BEQ RS, x0, offset|等于零则分支|beqz x5, label|
|BNEZ|BNE RS, x0, offset|不等于零则分支|bnez x5, label|
|BLTZ|BLT RS, x0, offset|小于零则分支|bltz x5, label|
|BLEZ|BGE x0, RS, offset|小于等于零则分支|blez x5, label|
|BGTZ|BLT x0, RS, offset|大于零则分支|bgtz x5, label|
|BGEZ|BGE RS, x0, offset|大于等于零则分支|bgez x5, label|

> 注意：分支指令的目标地址计算方法是将偏移量乘以 2，然后与 PC 相加，跳转范围是 [-4096, 4094] 字节

### 无条件跳转指令

#### JAL（Jump And Link）

```
JAL RD, label
```

- **格式**：J-type
- **功能**：将当前 PC+4 存入 RD（通常是 x1/ra），然后跳转到目标地址
- **示例**：`jal x1, function` 调用函数
- **跳转范围**：以 PC 为基准，±1MB

#### JALR（Jump And Link Register）

```
JALR RD, offset(RS1)
```

- **格式**：I-type
- **功能**：将当前 PC+4 存入 RD，然后跳转到 RS1+offset 的地址
- **示例**：`jalr x1, 0(x5)` 通过寄存器间接跳转
- **跳转范围**：以 RS1 为基准，±2KB

无条件跳转指令的伪指令：

|伪指令|等价指令|功能|示例|
|---|---|---|---|
|J|JAL x0, offset|无返回跳转|j label|
|JR|JALR x0, 0(rs)|无返回寄存器跳转|jr x5|
|RET|JALR x0, 0(x1)|函数返回|ret|
|CALL|AUIPC+JALR组合|长距离调用|call function|
|TAIL|AUIPC+JALR组合|长距离尾调用|tail function|

> 注意：
> 
> 1. 长距离跳转可以使用 AUIPC+JALR 组合实现
> 2. 尾调用是一种优化技术，用于减少函数调用的栈开销

### RISC-V 指令寻址模式总结

|寻址模式|解释|示例|
|---|---|---|
|立即数寻址|操作数是指令本身的一部分|addi x5, x6, 20|
|寄存器寻址|操作数存放在寄存器中|add x5, x6, x7|
|基址寻址|地址 = 基址寄存器 + 偏移量|sw x5, 40(x6)|
|PC 相对寻址|地址 = PC + 偏移量|beq x5, x6, label|

## RISC-V 汇编函数调用约定

### 函数调用过程概述

函数调用涉及栈操作，包括：

1. **栈的操作**：
    - 压栈（Push）：将数据保存到栈上，栈指针减小
    - 出栈（Pop）：从栈上取回数据，栈指针增大
2. **栈帧**：
    - 每个函数调用都有自己的栈帧
    - 栈帧包含返回地址、保存的寄存器、局部变量等

### 汇编编程时为何需要制定函数调用约定

函数调用约定（Calling Conventions）规定了函数间如何传递以下信息：

- **调用参数**：如何传递函数参数
- **返回地址**：如何保存和恢复返回地址
- **返回值**：如何传递函数返回值
- **寄存器使用**：哪些寄存器由调用者保存，哪些由被调用者保存

### 函数调用过程中有关寄存器的编程约定

|寄存器名|ABI 名|用途约定|保存责任|
|---|---|---|---|
|x0|zero|读取时总为 0，写入时不起任何效果|N/A|
|x1|ra|存放函数返回地址|调用者|
|x2|sp|栈指针|被调用者|
|x5-x7, x28-x31|t0-t6|临时寄存器，可能被被调用函数修改|调用者|
|x8-x9, x18-x27|s0-s11|保存寄存器，被调用函数必须保证不变|被调用者|
|x10-x11|a0-a1|参数/返回值寄存器|调用者|
|x12-x17|a2-a7|参数寄存器|调用者|

> 注意：
> 
> - "调用者保存"意味着如果调用者需要这些寄存器的值在调用后保持不变，必须自己负责保存
> - "被调用者保存"意味着被调用函数必须保证这些寄存器的值在返回时与调用前相同

### 函数调用过程中函数跳转和返回指令的编程约定

|伪指令|等价指令|描述|例子|
|---|---|---|---|
|jal|jal x1, offset|跳转到 offset，返回地址保存在 x1|jal function|
|jalr|jalr x1, 0(rs)|跳转到 rs 寄存器值，返回地址保存在 x1|jalr x5|
|call|auipc + jalr 组合|长距离调用函数|call function|
|ret|jalr x0, 0(x1)|从函数返回|ret|

### 函数调用过程中实现被调用函数的编程约定

函数实现通常包含以下部分：

1. **函数起始部分（Prologue）**：
    - 减少 sp 的值，为函数的栈帧分配空间
    - 将需要保存的寄存器（如 s0-s11）存入栈中
    - 如果函数会调用其他函数，需要保存 ra
2. **函数执行体**：
    - 函数的主要计算逻辑
3. **函数退出部分（Epilogue）**：
    - 从栈中恢复保存的寄存器
    - 如果保存了 ra，需要恢复
    - 增加 sp 的值，释放栈帧
    - 使用 ret 指令返回

## RISC-V 汇编与 C 混合编程

### RISC-V 汇编调用 C 函数

要从汇编代码调用 C 函数，需要遵循 ABI（Application Binary Interface）规定：

- **参数传递**：使用 a0-a7 寄存器传递前 8 个参数，更多参数使用栈
- **返回值**：使用 a0-a1 寄存器传递返回值
- **其他**：遵循调用者/被调用者保存的寄存器约定

### C 函数中嵌入 RISC-V 汇编

GCC 允许在 C 代码中内嵌汇编代码，使用 `asm` 关键字：

```c
asm [volatile] (
    "汇编指令"
    : 输出操作数列表（可选）
    : 输入操作数列表（可选）
    : 可能影响的寄存器或者存储器（可选）
);
```

例如：

```c
int foo(int a, int b)
{
    int c;
    asm volatile (
        "add %0, %1, %2" 
        : "=r" (c)       // 输出操作数
        : "r" (a), "r" (b) // 输入操作数
    );
    return c;
}
```

或者使用命名操作数：

```c
int foo(int a, int b)
{
    int c;
    asm volatile (
        "add %[sum], %[add1], %[add2]" 
        : [sum] "=r" (c)
        : [add1] "r" (a), [add2] "r" (b)
    );
    return c;
}
```

> 注意：
> 
> - 汇编指令用双引号括起来，多条指令用分号或换行符分隔
> - "=r" 表示输出寄存器，"r" 表示输入寄存器
> - 汇编代码中的 %0, %1, %2... 或者 %[name] 代表操作数
> - volatile 关键字告诉编译器不要优化此汇编语句