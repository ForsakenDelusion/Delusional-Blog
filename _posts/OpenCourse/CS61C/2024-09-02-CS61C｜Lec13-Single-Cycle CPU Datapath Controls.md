---
title: CS61C｜Lec13-Single-Cycle CPU Datapath Controls
date: 2024-09-02 00:34:03 +08:00
filename: 2024-09-02-CS61C｜Lec13-Single-Cycle CPU Datapath Controls
categories:
  - OpenCourse
  - CS61C
tags:
  - CS61C
  - ComputerArchitecture
dir: OpenCourse/CS61C
share: true
---
# Control and Status Registers

控制状态寄存器CSRs，不属于寄存器堆中的寄存器(x0-x31)，也就是不属于数据寄存器。被用于监视状态和性能，标准的RISCV最多可以有4096个CSRs。

他不属于基础指令集(因为模块化的原因被移除了)，但是几乎存在于每个处理器中。

>不知道为什么要提CSRs，去看了一下别的年份都没提到。

不过还是了解一下吧

## CSR Instructions

![CS61C｜Lec13-Single-Cycle CPU Datapath Controls-20250122.png](../../../assets/images/CS61C%EF%BD%9CLec13-Single-Cycle%20CPU%20Datapath%20Controls-20250122.png)

![CS61C｜Lec13-Single-Cycle CPU Datapath Controls-20250122-1.png](../../../assets/images/CS61C%EF%BD%9CLec13-Single-Cycle%20CPU%20Datapath%20Controls-20250122-1.png)

## CSRRW（Atomic Read/Write CSR）指令

#### 功能

CSRRW（Control and Status Register Read Word）指令是一种原子读写操作，它用于在不被其他操作中断的情况下读取和写入控制与状态寄存器（CSRs）。

#### 语法

- `CSRRW rd, csr, rs1`：读取CSR的当前值到寄存器`rd`，并将`rs1`中的值写入CSR。
- `CSRRWI rd, csr, uimm`：读取CSR的当前值到寄存器`rd`，并将立即数`uimm`写入CSR。

#### 伪指令

- `csrw csr, rs1`：等同于 `CSRRW x0, csr, rs1`，即不关心读取的结果，仅仅将`rs1`的值写入CSR。
- `csrwi csr, uimm`：等同于 `CSRRWI x0, csr, uimm`，即不关心读取的结果，仅仅将立即数`uimm`写入CSR。

#### 原子性

- 原子操作意味着在执行过程中不会被其他操作打断。这对于并发环境下的数据一致性非常重要。
- CSRRW确保读取CSR的值和随后写入新值作为一个整体完成，中间不会被其他操作插入。

#### 实现提示

- 使用写使能（Write Enable）信号来控制寄存器是否将输入数据写入。
- 时钟信号（CLK）用于同步写操作，确保数据在时钟上升沿被正确写入。

# Control Implementation

控制器负责设置多路选择器和数据通路中的其他配置选项，已执行不同的指令。

从指令内存和加法器的延迟是可比的。为什么？不是说从内存中获取东西就像从ucb到萨克拉门托吗？比在CPU内取东西慢多了。这里注意一下，指令内存实际上是一份内存的局部副本，涉及到cache的概念。

## Example

下面举两个例子，分析过程忘了可以自己去听一下视频里教授是怎么来的。

### SW

![CS61C｜Lec13-Single-Cycle CPU Datapath Controls-20250122-2.png](../../../assets/images/CS61C%EF%BD%9CLec13-Single-Cycle%20CPU%20Datapath%20Controls-20250122-2.png)

### beq

![CS61C｜Lec13-Single-Cycle CPU Datapath Controls-20250122-3.png](../../../assets/images/CS61C%EF%BD%9CLec13-Single-Cycle%20CPU%20Datapath%20Controls-20250122-3.png)

## Instruction Timing

### Add

![CS61C｜Lec13-Single-Cycle CPU Datapath Controls-20250122-4.png](../../../assets/images/CS61C%EF%BD%9CLec13-Single-Cycle%20CPU%20Datapath%20Controls-20250122-4.png)

注意，在PC在上升沿更新时，所有位时同时更新的，因为他们经过相同的触发器，但是PC+4是最低位比高位更先更新(应该是因为级联加法器的缘故，从低位加到高位)。在图表中，我们显示最后一位更新好了之后的时间。

量化时间

![CS61C｜Lec13-Single-Cycle CPU Datapath Controls-20250122-5.png](../../../assets/images/CS61C%EF%BD%9CLec13-Single-Cycle%20CPU%20Datapath%20Controls-20250122-5.png)

下面关键路径里面的max大括号，实际上就是在找图上几条路径中谁的延迟最大
### LW

![CS61C｜Lec13-Single-Cycle CPU Datapath Controls-20250122-6.png](../../../assets/images/CS61C%EF%BD%9CLec13-Single-Cycle%20CPU%20Datapath%20Controls-20250122-6.png)

![CS61C｜Lec13-Single-Cycle CPU Datapath Controls-20250122-7.png](../../../assets/images/CS61C%EF%BD%9CLec13-Single-Cycle%20CPU%20Datapath%20Controls-20250122-7.png)

![CS61C｜Lec13-Single-Cycle CPU Datapath Controls-20250122-8.png](../../../assets/images/CS61C%EF%BD%9CLec13-Single-Cycle%20CPU%20Datapath%20Controls-20250122-8.png)

如果我们只执行数据通路的一部分，我们就可以按照每个执行单元中的最长延迟来设定对应频率，这样我们可以获得5GHz的频率，但是我们不能这样做，我们不能只执行一些命令而忽略其他命令。但是稍后，我们会谈到如何从这个想法中获取收益。


## Control Logic Truth Table

![CS61C｜Lec13-Single-Cycle CPU Datapath Controls-20250122-9.png](../../../assets/images/CS61C%EF%BD%9CLec13-Single-Cycle%20CPU%20Datapath%20Controls-20250122-9.png)

BrEq和BrLT是条件转移比较的结果，指向Control

## Control Realization Options

### ROM

- Read-Only Memory

- 结构规范

- 易于重新编程(易于修复错误或者增加指令)

在手动设计控制逻辑时很受欢迎。

### Combinatorial Logic

也可以用硬布线的方式来实现控制器，2024su proj3里面用的是ROM，不过大佬们都推荐用硬布线的方法锻炼自己

## RV32I,A Nine-Bit ISA!

我们可以用9bit来区分一个指令

![CS61C｜Lec13-Single-Cycle CPU Datapath Controls-20250122-10.png](../../../assets/images/CS61C%EF%BD%9CLec13-Single-Cycle%20CPU%20Datapath%20Controls-20250122-10.png)

## Combination Logic Control

![CS61C｜Lec13-Single-Cycle CPU Datapath Controls-20250122-11.png](../../../assets/images/CS61C%EF%BD%9CLec13-Single-Cycle%20CPU%20Datapath%20Controls-20250122-11.png)
## ROM-based Control

![CS61C｜Lec13-Single-Cycle CPU Datapath Controls-20250122-12.png](../../../assets/images/CS61C%EF%BD%9CLec13-Single-Cycle%20CPU%20Datapath%20Controls-20250122-12.png)

我们有9位来自指令的输入，还有2位来自数据通路本身的输入，(BrEq,BrLT决定着跳转与否)

当我们进入ROM内部

ROM是一个与或结构，左边的线在解码一条指令的时候只会有一条线是被点亮的。

关于AND->是指令的所有位and一个指令，得到1，所以才输出这个指令，即输入的每一位都要与某个指令完全匹配，才输出这个指令(个人理解)

关于OR->应该是指，输出只是这么多行中的一个，add或sub或or或...所以实际上始终只有一个输出。

![CS61C｜Lec13-Single-Cycle CPU Datapath Controls-20250122-13.png](../../../assets/images/CS61C%EF%BD%9CLec13-Single-Cycle%20CPU%20Datapath%20Controls-20250122-13.png)
下面是su20的图，都差不多其实
![CS61C｜Lec13-Single-Cycle CPU Datapath Controls-20250122-14.png](../../../assets/images/CS61C%EF%BD%9CLec13-Single-Cycle%20CPU%20Datapath%20Controls-20250122-14.png)

## Control Logic to Decode add

![CS61C｜Lec13-Single-Cycle CPU Datapath Controls-20250122-15.png](../../../assets/images/CS61C%EF%BD%9CLec13-Single-Cycle%20CPU%20Datapath%20Controls-20250122-15.png)

