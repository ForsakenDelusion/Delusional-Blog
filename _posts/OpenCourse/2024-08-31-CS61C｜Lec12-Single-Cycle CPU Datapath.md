---
title: CS61C｜Lec12-Single-Cycle CPU Datapath
date: 2024-08-31 00:34:03 +08:00
filename: 2024-08-31-CS61C｜Lec12-Single-Cycle CPU Datapath
categories:
  - OpenCourse
tags:
  - CS61C
  - Computer Architecture
  - Computer
  - Architecture
dir: OpenCourse
share: true
---
# What's a CPU

## Your CPU in two parts

**Datapath(数据通路)**:数据在功能部件之间传送的路径称为数据通路，路径上的部件称为数据通路部件，如ALU、通用寄存器等。

**Control(控制器)**:它需要根据输入指令做出决策，例如确定当前执行的操作类型、是否需要从内存中获取信息、是否需要将结果写入寄存器以及写入哪个寄存器。

![CS61C｜Lec12-Single-Cycle CPU Datapath-20250122.png](../../assets/images/CS61C%EF%BD%9CLec12-Single-Cycle%20CPU%20Datapath-20250122.png)

## One-Instruction-Per-Cycle RISC-V Machine

首先我们考虑单条指令是怎么样运作的，然后我们再将其推广到所有命令

注:IMEM instraction memory;DMEM data memory

![CS61C｜Lec12-Single-Cycle CPU Datapath-20250122-1.png](../../assets/images/CS61C%EF%BD%9CLec12-Single-Cycle%20CPU%20Datapath-20250122-1.png)

如果我们要进行一个加法操作，我们需要的内容都会在IMEM里面，我们并不是太在乎DMEM里面是什么，但是我们在乎REG和PC里面是什么

IMEM中PC和寄存器的内容将会用于执行这条指令

所以说，如果执行`add`，逻辑电路将会将寄存器中的内容相加，再将结果储存到寄存器中，然后再将PC更新

我们可以为每条指令都设计一个不同的组合逻辑电路泡(为什么叫泡，如上图所示，像个泡泡)，但是这样太繁琐，太笨重和低效了，实际上很多RISCV指令都比较类似，我们可以将执行指令的过程分解成多个阶段，然后将这些阶段连接起来形成整个数据路径。

## Five Stages of the Datapath

1. **Stage 1: Instruction Fetch (IF)**：
    
    - 在这个阶段，处理器从内存中读取下一条要执行的指令。这是指令执行过程的第一步，确保处理器知道接下来要做什么。

1. **Stage 2: Instruction Decode (ID)**：
    
    - 在这个阶段，处理器将读取的指令进行解码，以确定指令的类型和操作数。解码过程将指令转换成处理器可以理解的格式，为后续的执行做准备。

1. **Stage 3: Execute (EX) - ALU (Arithmetic-Logic Unit)**：
    
    - 在这个阶段，处理器使用算术逻辑单元（ALU）来执行指令的操作。ALU 负责执行算术运算（如加法、减法）和逻辑运算（如与、或、非）。

1. **Stage 4: Memory Access (MEM)**：
    
    - 在这个阶段，处理器根据指令的要求访问内存。这可能包括读取数据（如加载操作）或写入数据（如存储操作）。

1. **Stage 5: Write Back to Register (WB)**：
    
    - 在这个阶段，处理器将执行结果写回寄存器。

## Basic Phases of Instruction Execution

![CS61C｜Lec12-Single-Cycle CPU Datapath-20250122-2.png](../../assets/images/CS61C%EF%BD%9CLec12-Single-Cycle%20CPU%20Datapath-20250122-2.png)

我们实际上只有一块物理上的内存，但是我们在这里将内存看成两部分组成——IMEM和DMEM

PC下面的mux是用来在遇到branch指令的时候绕过+4的正常递增，转而在PC上加上需要跳转到的地方的相对距离

## Datapath Components: Combinational

![CS61C｜Lec12-Single-Cycle CPU Datapath-20250122-3.png](../../assets/images/CS61C%EF%BD%9CLec12-Single-Cycle%20CPU%20Datapath-20250122-3.png)

我们可以使用上面这些组合逻辑单元来组成单周期CPU，但是我们还要在其中添加状态元件/单元(state elements)，用于存储数据和时钟同步方法。

接下来，让我们来讨论一下这些状态单元

## Datapath Elements: State and Sequencing

### Register

首先来看寄存器，寄存器实际上就是由触发器(flip-flops)构成，之前也提过，32位寄存器实际上就是由32个触发器构成

而寄存器组(regsiter file)又是寄存器的集合。

多个触发器组成寄存器，而多个寄存器又组成寄存器组

下面的斜杠N，代表有N条线进入这里
![CS61C｜Lec12-Single-Cycle CPU Datapath-20250122-4.png](../../assets/images/CS61C%EF%BD%9CLec12-Single-Cycle%20CPU%20Datapath-20250122-4.png)

Write Enable(写使能):

- 写使能信号控制寄存器是否将输入数据写入寄存器中。

- 当写使能信号为低电平（0）时，寄存器不会将输入数据写入，输出数据保持不变。

- 当写使能信号为高电平（1）时，寄存器会在时钟的上升沿将输入数据写入，并更新输出数据。

### Register file(regfile,RF)

![CS61C｜Lec12-Single-Cycle CPU Datapath-20250122-5.png](../../assets/images/CS61C%EF%BD%9CLec12-Single-Cycle%20CPU%20Datapath-20250122-5.png)
![CS61C｜Lec12-Single-Cycle CPU Datapath-20250122-6.png](../../assets/images/CS61C%EF%BD%9CLec12-Single-Cycle%20CPU%20Datapath-20250122-6.png)
1. **寄存器文件（Register File, RF）**：

    - 寄存器文件由32个32位寄存器组成(RISCV32位)。在su20中，这一部分被描述成有31个寄存器，因为x0被硬编码成了0。

    - 它有两个32位输出总线（busA和busB）和一个32位输入总线（busW）。

1. **寄存器选择**：
    
    - 寄存器通过RA、RB和RW信号进行选择。

    - RA（读地址）选择要读取的寄存器，并将其数据放到busA上。

    - RB（读地址）选择要读取的寄存器，并将其数据放到busB上。

    - RW（写地址）选择要写入的寄存器，并将busW上的数据写入该寄存器。

1. **时钟输入（Clock input, Clk）**：
    
    - 时钟输入信号（Clk）在写操作期间起作用，当写使能信号（Write Enable）为1时，数据从busW写入选定的寄存器。

1. **读操作**：
    
    - 在读操作期间，寄存器文件的行为类似于组合逻辑块，RA或RB有效时，经过读写访问时间(access times,就是与内存读写相关的延迟)后busA或busB有效。

### "Magic" Memory

>Delusion:不知道这个magic的含义是什么

![CS61C｜Lec12-Single-Cycle CPU Datapath-20250122-7.png](../../assets/images/CS61C%EF%BD%9CLec12-Single-Cycle%20CPU%20Datapath-20250122-7.png)

1. **输入和输出**：
    
    - "魔法"内存有一个输入总线（Data In）和一个输出总线（Data Out）。

1. **寻址方式**：
    
    - 对于读操作，地址（Address）选择要读取的数据并将其放在输出总线上（Data Out）。

    - 对于写操作，设置写使能信号（Write Enable）为1，地址选择要写入的数据，通过输入总线（Data In）写入内存。

1. **时钟输入（CLK）**：
    
    - 时钟输入信号（CLK）仅在写操作期间起作用。

    - 在读操作期间，行为类似组合逻辑块：地址有效时，经过读写访问时间(access times,就是与内存读写相关的延迟)后，输出总线（Data Out）的有效数据可用。

## State Required by RV32I ISA

在执行每条指令的时候我们都要读取并且更新寄存器，PC和内存的状态

- **Registers(x0..x32)**

	- 寄存器文件（Register file, regfile）包含32个32位寄存器，编号为Reg\[0]到Reg\[31]。

	- 第一个被读取的寄存器由指令中的rs1字段指定。

	- 第二个被读取的寄存器由指令中的rs2字段指定。

	- 目标寄存器（write register）由rd字段指定。

	- 寄存器x0始终为0（写入Reg[0]的操作会被忽略，所以实际上不需要有触发器和x0相连，把x0和0连上就行）。

- **程序计数器（Program Counter, PC）**：
    
    - 程序计数器保存当前指令的地址。

1. **内存（Memory）**：
    
    - 内存同时存放指令和数据，在一个32位字节寻址空间内。

    - 使用独立的指令内存（IMEM）和数据内存（DMEM）来分别存放指令和数据。

    - 指令是从IMEM中读取出来的(假设IMEM是只读的)。

    - 加载/存储指令访问数据内存。

## Implementing R-Types

![CS61C｜Lec12-Single-Cycle CPU Datapath-20250122-8.png](../../assets/images/CS61C%EF%BD%9CLec12-Single-Cycle%20CPU%20Datapath-20250122-8.png)

## R-Type

### Add Datapath

![CS61C｜Lec12-Single-Cycle CPU Datapath-20250122-9.png](../../assets/images/CS61C%EF%BD%9CLec12-Single-Cycle%20CPU%20Datapath-20250122-9.png)

![CS61C｜Lec12-Single-Cycle CPU Datapath-20250122-10.png](../../assets/images/CS61C%EF%BD%9CLec12-Single-Cycle%20CPU%20Datapath-20250122-10.png)

![CS61C｜Lec12-Single-Cycle CPU Datapath-20250122-11.png](../../assets/images/CS61C%EF%BD%9CLec12-Single-Cycle%20CPU%20Datapath-20250122-11.png)

这里的PC和PC+4省略了高位的4个0

### Sub Datapath

![CS61C｜Lec12-Single-Cycle CPU Datapath-20250122-12.png](../../assets/images/CS61C%EF%BD%9CLec12-Single-Cycle%20CPU%20Datapath-20250122-12.png)

add和sub指令非常的相似，只有指令的第30位决定了是加还是减

![CS61C｜Lec12-Single-Cycle CPU Datapath-20250122-13.png](../../assets/images/CS61C%EF%BD%9CLec12-Single-Cycle%20CPU%20Datapath-20250122-13.png)

在alu上加一条来自控制器的ALU select就好了！ALUSel决定了ALU将会执行什么操作

## Arithmetic I-Type

示例:

`addi x15,x1,-50`

转换成指令就是

![CS61C｜Lec12-Single-Cycle CPU Datapath-20250122-14.png](../../assets/images/CS61C%EF%BD%9CLec12-Single-Cycle%20CPU%20Datapath-20250122-14.png)

我们采用一个多路选择器来实现R型指令和I型指令都支持的效果。用Bsel(Bsel,即operand B select)来决定使用寄存器还是立即数。

![CS61C｜Lec12-Single-Cycle CPU Datapath-20250122-15.png](../../assets/images/CS61C%EF%BD%9CLec12-Single-Cycle%20CPU%20Datapath-20250122-15.png)

因为I-Type指令中的立即数是12位的，所以我们还需要一个立即数生成器将其扩展到32位

所有其他 I 格式的算术指令（slti、sltiu、andi、ori、xori、slli、srli、srai）只需通过更改 ALUSel 即可工作。

![CS61C｜Lec12-Single-Cycle CPU Datapath-20250122-16.png](../../assets/images/CS61C%EF%BD%9CLec12-Single-Cycle%20CPU%20Datapath-20250122-16.png)

## Supporting Loads(I-Type)

示例:

`lw x14, 8(x2)`

![CS61C｜Lec12-Single-Cycle CPU Datapath-20250122-17.png](../../assets/images/CS61C%EF%BD%9CLec12-Single-Cycle%20CPU%20Datapath-20250122-17.png)

12位有符号立即数会加到寄存器rs1中的基地址上，形成内存地址。

这和addi的操作非常相似

从内存中拿到的值会被放在rd寄存器

![CS61C｜Lec12-Single-Cycle CPU Datapath-20250122-18.png](../../assets/images/CS61C%EF%BD%9CLec12-Single-Cycle%20CPU%20Datapath-20250122-18.png)

RV32的加载指令有很多个，想要支持这些指令只需要一个多路选择器和一些逻辑门即可。

![CS61C｜Lec12-Single-Cycle CPU Datapath-20250122-19.png](../../assets/images/CS61C%EF%BD%9CLec12-Single-Cycle%20CPU%20Datapath-20250122-19.png)

## Adding sw instruction(S-Type)

sw：读取两个寄存器，rs1，rs2，还有一个立即数作为偏移地址。

示例:

`sw x14, 8(x2)`

![CS61C｜Lec12-Single-Cycle CPU Datapath-20250122-20.png](../../assets/images/CS61C%EF%BD%9CLec12-Single-Cycle%20CPU%20Datapath-20250122-20.png)

![CS61C｜Lec12-Single-Cycle CPU Datapath-20250122-21.png](../../assets/images/CS61C%EF%BD%9CLec12-Single-Cycle%20CPU%20Datapath-20250122-21.png)

**上图的关于立即数生成器的输入端应该是有误，正确的应该是\[31:7]进入立即数生成器，因为S-Type的命令的立即数分为两段，正确的应该如下图su20的slide所示。后面fa20的slide都有这个问题，注意一下**

![CS61C｜Lec12-Single-Cycle CPU Datapath-20250122-22.png](../../assets/images/CS61C%EF%BD%9CLec12-Single-Cycle%20CPU%20Datapath-20250122-22.png)


关于支持I型和S型指令不同的立即数生成器

![CS61C｜Lec12-Single-Cycle CPU Datapath-20250122-23.png](../../assets/images/CS61C%EF%BD%9CLec12-Single-Cycle%20CPU%20Datapath-20250122-23.png)

## Implementing Branches(SB-Type)

如果rs1和rs2寄存器中的值满足条件，PC就会更新为一个新的地址，一个由当前地址加上一个立即数偏移量得到的地址。不满足的话就继续执行下一条指令。

### B-Format for Branch

![CS61C｜Lec12-Single-Cycle CPU Datapath-20250122-24.png](../../assets/images/CS61C%EF%BD%9CLec12-Single-Cycle%20CPU%20Datapath-20250122-24.png)

- B-Format与S-Format类似，具有两个寄存器源（rs1和rs2）和一个12位立即数（imm[12:1]）。

- 立即数表示-4096到+4094之间的值，以2字节为增量。

- 12位立即数编码13位有符号字节偏移量，最低位始终为0，因此不需要存储。

### To Add Branches

![CS61C｜Lec12-Single-Cycle CPU Datapath-20250122-25.png](../../assets/images/CS61C%EF%BD%9CLec12-Single-Cycle%20CPU%20Datapath-20250122-25.png)

有六种条件转移指令:`beq`,`bne`,`blt`,`bge`,`bltu`,`bgeu`

我们需要计算PC + 立即数的值，我们还需要比较rs1和rs2的值，但是我们只有一个ALU

所以，我们需要添加新的硬件。这个硬件只需要做比较就可以了。

![CS61C｜Lec12-Single-Cycle CPU Datapath-20250122-26.png](../../assets/images/CS61C%EF%BD%9CLec12-Single-Cycle%20CPU%20Datapath-20250122-26.png)

### Branch Comparator

![CS61C｜Lec12-Single-Cycle CPU Datapath-20250122-27.png](../../assets/images/CS61C%EF%BD%9CLec12-Single-Cycle%20CPU%20Datapath-20250122-27.png)

BrEq = 1, if A = B

BrLT = 1, if A < B

BrUn = 1，在BrLT中选择无符号数

### Branch Immediates(In Other ISAs)

	在其他指令集中，将S立即数转换为B立即数的方法是，直接把12位的数往左移一位(因为地址是2的倍数，所以最后一位始终为0，就省略了)。但这样会造成一个后果，每一位都需要一个2路选择器去决定这个立即数到底是S型还是B型。

![CS61C｜Lec12-Single-Cycle CPU Datapath-20250122-28.png](../../assets/images/CS61C%EF%BD%9CLec12-Single-Cycle%20CPU%20Datapath-20250122-28.png)

但是在RISCV中，12位中的有11位都是一样的，所以只需要一个2路选择器就好了。

![CS61C｜Lec12-Single-Cycle CPU Datapath-20250122-29.png](../../assets/images/CS61C%EF%BD%9CLec12-Single-Cycle%20CPU%20Datapath-20250122-29.png)

![CS61C｜Lec12-Single-Cycle CPU Datapath-20250122-30.png](../../assets/images/CS61C%EF%BD%9CLec12-Single-Cycle%20CPU%20Datapath-20250122-30.png)

### Lighting Up Branch Path

![CS61C｜Lec12-Single-Cycle CPU Datapath-20250122-31.png](../../assets/images/CS61C%EF%BD%9CLec12-Single-Cycle%20CPU%20Datapath-20250122-31.png)

## Adding JALR to Datapath(I-Type)

![CS61C｜Lec12-Single-Cycle CPU Datapath-20250122-32.png](../../assets/images/CS61C%EF%BD%9CLec12-Single-Cycle%20CPU%20Datapath-20250122-32.png)

1. **JALR指令格式**：
    
    - JALR指令的格式为：JALR rd, rs, immediate。

    - 其中，rd是目标寄存器，rs是源寄存器，immediate是立即数。

1. **指令执行**：
    
    - 将PC+4的值写入寄存器rd（返回地址）。

    - 将寄存器rs的值与立即数相加，设置为新的PC值。

1. **数据通路**：
    
    - 需要修改多路复用器（MUX），允许将PC+4的值作为写回值。

    - 可以将rs和immediate作为ALU的输入，因此不需要对ALU进行修改。

    - 使用与算术和加载指令相同的立即数，不需要乘以2字节（与分支指令不同）。所以实际上少了一位的范围。这是我们为了简化电路设计做出的折衷——少一位范围，但是可以复用I-Type的立即数生成器。(为什么不直接用B-Type的立即数生成器呢？个人感觉是因为rs与立即数相加可以直接用算数I型指令直接执行，要是变成B-Type的立即数了，就不能直接相加了？)

![CS61C｜Lec12-Single-Cycle CPU Datapath-20250122-33.png](../../assets/images/CS61C%EF%BD%9CLec12-Single-Cycle%20CPU%20Datapath-20250122-33.png)

## J-Type

![CS61C｜Lec12-Single-Cycle CPU Datapath-20250122-34.png](../../assets/images/CS61C%EF%BD%9CLec12-Single-Cycle%20CPU%20Datapath-20250122-34.png)

让我们看看我们需要支持那些东西

- JAL将PC+4存进`Reg[rd]`->已经支持

- 设定PC = PC + offset(PC - reletive jump)->已经支持

	- 偏移量的范围为$\pm{2^{19}}$个位置，以2字节为增量，所以可以访问$\pm{2^{19}}$个32位指令。
   
- 唯一需要支持的就是J型立即数生成器，实际上我们稍微加几个多路选择器就可以复用B型立即数生成器来生成J型立即数。

我们复用了R和Branch的机制来支持了JAL

![CS61C｜Lec12-Single-Cycle CPU Datapath-20250122-35.png](../../assets/images/CS61C%EF%BD%9CLec12-Single-Cycle%20CPU%20Datapath-20250122-35.png)

## U-Type

![CS61C｜Lec12-Single-Cycle CPU Datapath-20250122-36.png](../../assets/images/CS61C%EF%BD%9CLec12-Single-Cycle%20CPU%20Datapath-20250122-36.png)

U型指令在高二十位有20位的立即数。一个目标寄存器rd，`lui`和`auipc`都是U型指令

为了支持U型指令，只需要增加一个U型立即数生成器即可。

![CS61C｜Lec12-Single-Cycle CPU Datapath-20250122-37.png](../../assets/images/CS61C%EF%BD%9CLec12-Single-Cycle%20CPU%20Datapath-20250122-37.png)

# Our CPU

![CS61C｜Lec12-Single-Cycle CPU Datapath-20250122-38.png](../../assets/images/CS61C%EF%BD%9CLec12-Single-Cycle%20CPU%20Datapath-20250122-38.png)

![CS61C｜Lec12-Single-Cycle CPU Datapath-20250122-39.png](../../assets/images/CS61C%EF%BD%9CLec12-Single-Cycle%20CPU%20Datapath-20250122-39.png)

- **设计处理器的五个步骤**：
    
    1. 分析指令集，确定数据通路需求。

    2. 选择数据通路组件，建立时钟方法。

    3. 组装数据通路，满足需求。

    4. 分析每条指令的实现，确定控制点的设置，以实现寄存器传输。

    5. 组装控制逻辑。

- **设计原则**：
    
    - 确定控制信号：

        - 当数据通路元素的输入改变行为时，需要控制信号（例如ALU操作、读写）。

        - 当需要根据指令选择不同的输入时，添加一个带有控制信号作为选择器的多路复用器（例如下一个PC、ALU输入、要写入的寄存器）。

    - 控制信号会根据具体的数据通路而变化。

    - 数据通路会根据指令集架构（ISA）而变化。
