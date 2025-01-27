---
title: CS61C｜Lec8-RISC-V Instruction Formats
date: 2024-08-19 00:34:03 +08:00
filename: 2024-08-19-CS61C｜Lec8-RISC-V Instruction Formats
categories:
  - OpenClass
tags:
  - CS61C
  - RISCV
dir: OpenClass
share: true
---
![CS61C｜Lec8-RISC-V Instruction Formats-20250122.png](../../assets/images/CS61C%EF%BD%9CLec8-RISC-V%20Instruction%20Formats-20250122.png)

![CS61C｜Lec8-RISC-V Instruction Formats-20250122-1.png](../../assets/images/CS61C%EF%BD%9CLec8-RISC-V%20Instruction%20Formats-20250122-1.png)
# Stored-Program Concept

以前的计算机是非常难reprogram的，因为要用编程线和开关进行编程。通常需要2到3天去编一个新程序

## Big Idea:Stored-Program Concept

指令可以用bit来表示

- 整个程序像是数据一样被保存在内存中

- 重新编程就是重写数据(而不是“重写计算机”)

## Everything has a memory address

地址用来指向保存在内存中的数据和指令

- C指针只是用来指向数据的地址

- PC(The Program Counter)存着指向代码的内存地址

## How do you distinguish code and data?

**Depends on Interpretation**

0x007302B3既可以被当成数字`7,537,331`，也可以当作指令`add x5,x6,x7`

- 程序在内存中以数字的形式被存储

- 代码或数字全靠你如何interpret它

- 我们需要一个方法将数组解释成指令

## Binary compatibility

程序将会被分发成二进制形式(与指令集绑定)

新的机器为了兼容旧的程序("binaries")，同时运行使用新指令的程序，导致了向后兼容的指令集越来越多

![CS61C｜Lec8-RISC-V Instruction Formats-20250122-2.png](../../assets/images/CS61C%EF%BD%9CLec8-RISC-V%20Instruction%20Formats-20250122-2.png)

# Instructions as Numbers

按照惯例，RISCV指令每个占 1 word = 4 bytres = 32 bits

1. **指令字段划分**：
    
    - 为了简化硬件设计，RISC-V 指令中的 32 位被划分为不同的字段（field），每个字段代表指令的不同部分。

    - 通过使用标准的字段大小，可以简化处理器的设计和实现。
	
2. **指令格式类型**：

    RISC-V 定义了 6 种基本的指令格式类型，分别是：
	
- **R-Format (Register-Type Instructions)**

	- 适用于大多数算术和逻辑运算指令，如加法、减法、逻辑与等。

	- 包含源寄存器和目标寄存器的信息。

- **I-Format (Immediate-Type Instructions)**

	- 用于立即数操作指令，如加载指令（load）、带立即数的算术操作等。

	- 包含一个立即数字段。

- **S-Format (Store-Type Instructions)**

	- 专门用于存储指令（store），如存储字指令（SW）等。

	- 包含基址寄存器、偏移量和目的寄存器等字段。

- **U-Format (Upper Immediate-Type Instructions)**

	- 用于加载大立即数到寄存器的指令。

	- 包含一个较大的立即数字段。

- **SB-Format (Branch-Type Instructions)**

	- 用于分支指令，如条件分支（BEQ、BNE 等）。

	- 包含分支的目标地址偏移量。

- **UJ-Format (Upper Immediate Jump-Type Instructions)**

	- 用于跳转指令（JAL），特别是带有立即数的跳转指令。

	- 包含一个较大的立即数字段，用于指定跳转目标地址。

## R-type

![CS61C｜Lec8-RISC-V Instruction Formats-20250122-3.png](../../assets/images/CS61C%EF%BD%9CLec8-RISC-V%20Instruction%20Formats-20250122-3.png)

- 为使用三个寄存器且没有立即数的指令设计

	- 诸如`add`或是`sub`这样的算数操作
	
- 每个寄存器都通过它的编号进行标识。32个寄存器 → 需要5个位来唯一标识一个寄存器。

 - **funct7 (7 位)**：这是一个额外的标识符，用于区分具有相同 `opcode` 和 `funct3` 的极其相似的指令。例如，`sra`（算术右移）和 `srl`（逻辑右移）指令具有相同的 `opcode` 和 `funct3`，但它们的功能略有不同，这时就需要 `funct7` 来区分它们。
    
 - **rs2 (5 位)**：表示第二个源寄存器的编号。
    
 - **rs1 (5 位)**：表示第一个源寄存器的编号。
    
- **funct3 (3 位)**：这是一个 3 位的标识符，用于区分具有相同 `opcode` 的不同指令。例如，不同的算术运算指令（如加法、减法等）具有相同的 `opcode`，0x33，但通过 `funct3` 可以区分它们。
    
- **rd (7 位)**：表示目标寄存器的编号。
    
- **opcode (7 位)**：这是指令的标识符，位于指令的最后 7 位。所有指令格式的最后 7 位都是 `opcode`。它用于识别指令的类型。

### 实践一下

`add s4 t2 a1`

Step 1: Determine opcode and instruction type from reference card

- Type:R

- Opcode:011 0011

- funct3:000

- funct7:000 0000\

Step 2:Registers

- rd:s4 x20->0b10111

- rs1:t2 x7->0b00111

- rs2:a1 ->0b01011

Step 3:Write out format

- 0b0000000 01011 00111 000 10111 0110011

- 0b0000 0000 1011 0011 1000 1011 1011 0011

- 0x00B38A33

`0x01F3 E533` to RISCV

答案是 `or a0 t2 t6`，自己尝试一下

## I-type

这里就解释了前面的为什么imm只有12位了

![CS61C｜Lec8-RISC-V Instruction Formats-20250122-4.png](../../assets/images/CS61C%EF%BD%9CLec8-RISC-V%20Instruction%20Formats-20250122-4.png)

- 为那些使用了两个寄存器（一个源一个目标）和一个立即数的指令设计

	- 带立即数的算数操作（addi，andi，slti，etc
	
	- 加载（lw，lh，lhu，lb，lbu
	
	- Jalr
	
	- ecall和ebreak从技术上说也是属于I-type

1. **Immediate 存储**：
    
    - 立即数（immediate）通常是一个较小的整数值，直接包含在指令中。

    - 在 I-Format 指令中，立即数被存储在一个名为 `imm` 的字段中。

    - 立即数的位被按照特定的方式放置在指令的 32 位中，例如，立即数的第 11 位会被放在指令的第 31 位上，以此类推直到第 0 位被放在指令的第 20 位上。

1. **I-Format 立即数的大小**：
    
    - I-Format 指令中的立即数是 12 位长。

    - 因此，可以存储的最大立即数值是 2^12 - 1 = 4095（对于无符号数），或者 -2048 到 +2047（对于有符号数）。

## I*-type

![CS61C｜Lec8-RISC-V Instruction Formats-20250122-5.png](../../assets/images/CS61C%EF%BD%9CLec8-RISC-V%20Instruction%20Formats-20250122-5.png)

为了位移指令(slli,srli,srai)，最大可以位移31位
- 这是因为寄存器的宽度是 32 位，而移位操作通常不会包括最左边的符号位（对于有符号数）。因此，任何超过 31 的移位量实际上都会将整个寄存器的内容移出，导致寄存器清零。

- 对于移位指令，它们采用了一种修改过的 I-Format，称为 I*（I-star）格式。这种格式包括了一个额外的 `funct7` 字段来指定移位量。

## S-type

![CS61C｜Lec8-RISC-V Instruction Formats-20250122-6.png](../../assets/images/CS61C%EF%BD%9CLec8-RISC-V%20Instruction%20Formats-20250122-6.png)

为那些需要两个源寄存器和一个立即数的指令设计（Store instructions

- 立即数的分割

	S-Format 指令的立即数被分割成两部分，一部分存储在指令的高位置，另一部分存储在低位置。这样做的原因是为了保持 `rs1` 和 `rs2` 字段的位置与 R-Format 指令相同，从而简化硬件设计并保持一致性。

**注意**：rs2在指令中在rs1前面，但是rs1才是与立即数相加的那一个

## SB-type

![CS61C｜Lec8-RISC-V Instruction Formats-20250122-7.png](../../assets/images/CS61C%EF%BD%9CLec8-RISC-V%20Instruction%20Formats-20250122-7.png)

格式和S差不多，所以B-Type又叫SB-Type

包含beq, bne,bge,blt这些指令。他比较两个寄存器，然后决定要不要跳转到给定的地址上去。注意，他不会进行写入寄存器的操作。

## U-type

![CS61C｜Lec8-RISC-V Instruction Formats-20250122-8.png](../../assets/images/CS61C%EF%BD%9CLec8-RISC-V%20Instruction%20Formats-20250122-8.png)

为那些需要20位立即数的指令设计的,比如`lui`和`auipc`

回忆一下：`lui`,`auipc`主要用于两个伪指令：

- li rd imm：把 rd 设为 imm

- la rd Label：把 rd 设为 Label 的地址

### lui(Load upper immediate)

`lui rd imm`

set td to imm << 12

当我们需要加载一个很大的立即数时，我们就会用到`lui`.

比如

`li t0 0x12345678`

无法用`addi t0 x0 0x12345678`来表示，因为这个立即数太大了，超出了12bit的范围

仔细想一下，我们可以用多个addi配合着slli来完成。但是这样太复杂了！我们需要5条指令！

- 解决办法：lui

```
lui t0 0x12345
addi t0 t0 0x678
```

This works, assuming lui (U-type) has 20 bits of immediate

#### 边界情况

`li t0 0xDEADBEEF

最初想法:
```
lui t0 0xDEADB
addi t0 0xEEF
```

因为进行了Sign Extend,所以实际上是0xDEADB000+0xFFFFFEEF=0xDEAD**A**EEF

  0xDEADB
+0xFFFFFF
\------------
  0xDEADA

实际上加上0xFFFFF...就等于减一

- 解决办法

如果一个被加的数是一个负数，那么就会在加这个12bit的值之前在高20位上加1

```
lui x10, 0xDEADC
addi x10, x10,0xEEF
```

`li`指令将会自动处理这个边界情况

### auipc(Add upper immediate to Program Counter)

`auipc rd imm`

sets rd to(imm << 12) + pc

## UJ-Format instructions(J-type)

![[CS61C｜Lec8-RISC-V Instruction Formats-20250122-9.png]]

对于branch（条件转移），我们假定它不会跳到太远的地方，所以我们可以指定PC的改变

对于general jump（jal），我们可以跳到code memory中的任何地方

- 理论上，我们可以指定32bit的内存地址跳转

- 不幸的是，我们不能把7位操作码和32位地址都塞进32位的字里

- 此外，在链接(指的是jump and link的link)时，我们必须写入一个寄存器rd

`jal`将PC + 4存入`rd`寄存器(the return address)

Set PC = PC + offset (PC-relative jump)

`jal` 指令支持跳转到当前指令指针前后最多 ±2^19 个位置，间隔为 2 字节。

这意味着它可以跳转到最多 ±2^18 个 32 位指令的距离内。

## jalr Instruction (I-Format)

`jalr rd, rs1, offset`

将PC + 4存入rd

Set PC = rs1 + offset

立即数的处理方式与算术指令和加载指令相同，即不乘以 2 字节。