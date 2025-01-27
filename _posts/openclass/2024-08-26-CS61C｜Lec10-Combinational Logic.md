---
title: CS61C｜Lec10-Combinational Logic
date: 2024-08-26 00:34:03 +08:00
filename: 2024-08-26-CS61C｜Lec10-Combinational Logic
categories:
  - openclass
tags:
  - CS61C
  - Computer Architecture
  - Computer
  - Architecture
dir: openclass
share: true
---
# Hardware Design Overview

![CS61C｜Lec10-Combinational Logic-20250122.png](../../assets/images/CS61C%EF%BD%9CLec10-Combinational%20Logic-20250122.png)

## Synchronous Digital Systems (SDS)

**Synchronous**:

- 所有操作由中央时钟协调。
	- 系统的“心跳(Heartbeat)”(处理器频率(processor frequency))

**Digital**:

- 将所有的值都表示成两个离散的值

- 电信号被看成1和0(两个相反的值)

- 高电位是1/True，低电位是0/false

# Switches and Transistors

## Switches

![CS61C｜Lec10-Combinational Logic-20250122-1.png](../../assets/images/CS61C%EF%BD%9CLec10-Combinational%20Logic-20250122-1.png)

![CS61C｜Lec10-Combinational Logic-20250122-2.png](../../assets/images/CS61C%EF%BD%9CLec10-Combinational%20Logic-20250122-2.png)

## Transistor(晶体管)

![CS61C｜Lec10-Combinational Logic-20250122-3.png](../../assets/images/CS61C%EF%BD%9CLec10-Combinational%20Logic-20250122-3.png)

我们可以用mos管来构建逻辑门

![CS61C｜Lec10-Combinational Logic-20250122-4.png](../../assets/images/CS61C%EF%BD%9CLec10-Combinational%20Logic-20250122-4.png)

![CS61C｜Lec10-Combinational Logic-20250122-5.png](../../assets/images/CS61C%EF%BD%9CLec10-Combinational%20Logic-20250122-5.png)

实际上，芯片只是由晶体管和导线组成。一些晶体管就可以构建出一些有用的模块

一个与非门的例子

![CS61C｜Lec10-Combinational Logic-20250122-6.png](../../assets/images/CS61C%EF%BD%9CLec10-Combinational%20Logic-20250122-6.png)

# Combinational Logic

## Combinational Logic Gates

数字系统由两种基本电路组成

- Combinational Logic(CL)｜组合逻辑电路

	- 输出结果仅仅是由输入变量决定的，而不与执行历史有关。

- Sequential Logic(SL)｜时序逻辑电路

	- 电路会记住或是储存信息

	- "State Elements",e.g. memory and registers(Registers)

## Logic Gates

![CS61C｜Lec10-Combinational Logic-20250122-7.png](../../assets/images/CS61C%EF%BD%9CLec10-Combinational%20Logic-20250122-7.png)


![CS61C｜Lec10-Combinational Logic-20250122-8.png](../../assets/images/CS61C%EF%BD%9CLec10-Combinational%20Logic-20250122-8.png)

![CS61C｜Lec10-Combinational Logic-20250122-9.png](../../assets/images/CS61C%EF%BD%9CLec10-Combinational%20Logic-20250122-9.png)

## Truth Tables

N个输入中的每一个都是0或1,所以一共有$2^N$行

![CS61C｜Lec10-Combinational Logic-20250122-10.png](../../assets/images/CS61C%EF%BD%9CLec10-Combinational%20Logic-20250122-10.png)

![CS61C｜Lec10-Combinational Logic-20250122-11.png](../../assets/images/CS61C%EF%BD%9CLec10-Combinational%20Logic-20250122-11.png)

多个输出就相当于多个独立的函数

假设 F 是一个具有以下功能的电路：

- X = A AND B
- Y = NOT(C)
- Z = D OR B

你可以使用真值表来表示这些函数，如下所示：

|A|B|C|D|X|Y|Z|
|---|---|---|---|---|---|---|
|0|0|0|0|0|1|0|
|0|0|0|1|0|1|1|
|0|0|1|0|0|0|0|
|0|0|1|1|0|0|1|
|0|1|0|0|0|1|0|
|0|1|0|1|0|1|1|
|0|1|1|0|0|0|0|
|0|1|1|1|0|0|1|
|1|0|0|0|0|1|0|
|1|0|0|1|0|1|1|
|1|0|1|0|0|0|0|
|1|0|1|1|0|0|1|
|1|1|0|0|1|1|0|
|1|1|0|1|1|1|1|
|1|1|1|0|1|0|1|
|1|1|1|1|1|0|1|

在这个真值表中，每一列代表一个输出函数，而不需要增加额外的行。每个输出都可以独立计算，与其它输出无关。

![CS61C｜Lec10-Combinational Logic-20250122-12.png](../../assets/images/CS61C%EF%BD%9CLec10-Combinational%20Logic-20250122-12.png)

关于真值表的另一种看待方法

2 输入门通常都是可以直接扩展到n输入的，但是异或(XOR)需要注意，XOR 是一个当且仅当其输入的二进制位中 1 的个数为奇数时成立的逻辑运算。

## Boolean Algebra

- **逻辑补（NOT）**：用头上的横行或 `¬` 表示，表示变量的逻辑反。
    - 如果 `A` 是 0，则 `¬A` 是 1；如果 `A` 是 1，则 `¬A` 是 0。
- **逻辑或（OR）**：用 `+` 表示，表示两个变量的逻辑或。
- **逻辑与（AND）**：用 `·` 或省略（如 `AB`）表示，表示两个变量的逻辑与。

![CS61C｜Lec10-Combinational Logic-20250122-13.png](../../assets/images/CS61C%EF%BD%9CLec10-Combinational%20Logic-20250122-13.png)

**Truth Table->Boolean Expression**

变量用1表示，变量的补用0表示，我们有两种方式来表示真值表，一种是积的和(Sum of Products)，另一种是和的积(Product of Sums)

### Sum of Products(SoP)

将所有输出位1的行的输入使用积的和的形式表示

### Product of Sums(PoS)

将所有输出位0的行的输入使用积的和的形式表示,比如

![CS61C｜Lec10-Combinational Logic-20250122-14.png](../../assets/images/CS61C%EF%BD%9CLec10-Combinational%20Logic-20250122-14.png)

## Laws of Boolean Algebra

![CS61C｜Lec10-Combinational Logic-20250122-15.png](../../assets/images/CS61C%EF%BD%9CLec10-Combinational%20Logic-20250122-15.png)

## Boolean Expression->Gate Diagram

![CS61C｜Lec10-Combinational Logic-20250122-16.png](../../assets/images/CS61C%EF%BD%9CLec10-Combinational%20Logic-20250122-16.png)

# Converting Combinational Logic

![CS61C｜Lec10-Combinational Logic-20250122-17.png](../../assets/images/CS61C%EF%BD%9CLec10-Combinational%20Logic-20250122-17.png)