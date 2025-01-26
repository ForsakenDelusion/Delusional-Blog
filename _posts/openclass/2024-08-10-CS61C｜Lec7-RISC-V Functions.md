---
title: CS61C｜Lec7-RISC-V Functions
date: 2024-08-10 00:34:03 +08:00
filename: 2024-08-10-CS61C｜Lec7-RISC-V Functions
categories:
  - openclass
tags:
  - CS61C
  - RISCV
dir: openclass
share: true
---
# Pseudo-Instructions

## Assemby Instructions

低级程序语言的指令会与特定架构的操作相匹配

- 代码可以被编译成不同的汇编语言，但是一种汇编语言只能在支持它的硬件上运行。

## Pseudo-Instructions

为了程序员的利益，可以有一些并不真正由硬件实现，而是由转换成真实指令的指令，称之为伪指令。

比如`mv dst, reg1`转换之后`addi dst, reg1, 0`

还有一些伪指令比如

### li(Load Immediate)

```
li dst, imm
```

加载32bit的立即数存入`dst`内

利用了`addi`，`lui`

### la(Load Address)

```
la dst, lable
```

加载特定标签的地址存入`dst`内

转换：`auipc dst, <offset to lable>`

### nop(No Operation)

```
nop
```

啥也不干

转换：`addi x0, x0, 0`


## C->RISCV Practice

![CS61C｜Lec7-RISC-V Functions-20250122.png](../../assets/images/CS61C%EF%BD%9CLec7-RISC-V%20Functions-20250122.png)

如果`lb`进行符号位扩展怎么办？

>没关系，因为我们在下一步也只读取一个byte。所以这是个byte to byte的操作

当然这个循环也能用bne来实现，在汇编里面循环的实现非常灵活。

# Function in Assmebly

## Six Step of Calling a Function

1. Put arguments in a place where the function can access them

2. Transfer control to the function

3. The function will acquire any (local) storage resources it needs

4. The function performs its desired task

5. The function puts return value in an accessible place and “cleans up”

6. Control is returned to you

## 1 & 5 Where should we put the arguments and return values?

a0-a7:用来传参

a0-a1:用来返回值

- 参数的顺序很重要

- 如果我们需要更多的空间，就用内存（stack

![CS61C｜Lec7-RISC-V Functions-20250122-1.png](../../assets/images/CS61C%EF%BD%9CLec7-RISC-V%20Functions-20250122-1.png)

sp(stack pointer):这里存放着栈底的地址

## 2 and 6: How do we Transfer Control?

我们可以用一下命令来“转移控制”

### J(Jump)

`j label`

### jal(Jump and Link)

`jal dst label`

Used to invoke a function

Lable：想要跳转到的标签。

>实际上汇编器会将标签转化成现在为止到指定标签偏移量，所以相当于PC = PC + 偏移量

### jalr(Jump and Link Register)

`jalr dst src imm`


>"and link":Save the location of instruction in a register before jumping

### jr(Jump Register)

`jr src`

Used to return form a function(src = ra)

>ra:返回地址寄存器，用来储存用来调用函数的返回地址（你让我过去，但你得告诉我回来的路）如果我们从一个函数跳转到另一个函数时，我们需要一个返回地址，因为当一个`callee`运行完成之后，我们需要返回到`caller`继续执行代码

![CS61C｜Lec7-RISC-V Functions-20250122-2.png](../../assets/images/CS61C%EF%BD%9CLec7-RISC-V%20Functions-20250122-2.png)

硬编码我们就不知道在函数结束之后到底返回到哪里

所以我们用`jal`来解决这个问题

![CS61C｜Lec7-RISC-V Functions-20250122-3.png](../../assets/images/CS61C%EF%BD%9CLec7-RISC-V%20Functions-20250122-3.png)

j其实是个伪指令，通过`jal x0 label`来实现

## 3: Local storage for variables

刚刚我们了解到`sp`是栈底指针，一直指向栈底，所以如果我们想要存储数据，就得让栈底指针往下移动（相当于在栈顶开辟了一块空间），然后使用`sw`来存储数据。如果需要“清理”栈，就只需要让`sp`回到原来的位置就可以了（数据实际上没有被真正的清理）

```
# store t0 to the stack
addi sp, sp, -4
sw t0, 0(sp)
```

# 4:The function performs its desired task

## Which registers can we use?

![CS61C｜Lec7-RISC-V Functions-20250122-4.png](../../assets/images/CS61C%EF%BD%9CLec7-RISC-V%20Functions-20250122-4.png)

比如

```C
int sumSquare(int x, int y) {
	return mul(x, x) + y;
}
```

- 我们需要保留什么？
	- 调用mul将会覆盖掉ra，所以留住ra
	- a1存着y，但是mul需要两个参数，所以我们要留着a1

## Calling Conventions

Caller:the calling function

Callee:the function being called

调用一个函数的函数叫做`caller`，而被调用的那个函数叫做`callee`。

- Register Conventions:规定着哪些寄存器在被调用之后不会改变，哪些可以改变（saved & volatile you'll see later

## Saved Register(Callee Saved)

这些寄存器在调用之后应该保持着与调用前相同的值，所以

- Callee如果要使用这些寄存器，需要在返回之前恢复其值(可以利用栈实现,我们将会在本文后面进行详细解释)

- 这意味着保存旧的值->使用`Saved Register`->恢复旧的值到`Saved Register`

以下寄存器就是**Saved Register**

- s0-s11

-  sp(stack pointer):如果callee更改了`sp`的值，caller就会找不到它原来该在什么地方了

## Volatile Registers (Caller Saved)

这些寄存器可以被Callee自由的改变

- 如果Caller需要用这些寄存器，需要在调用其他函数之前保存这些值(同样可以通过栈实现)，因为不保存的话这些寄存器很有可能被Callee使用

以下寄存器就是**Volatile Register**

- t0-t6

- a0-a7

- ra

## How do we save registers?

**Stack!!**

![CS61C｜Lec7-RISC-V Functions-20250122-5.png](../../assets/images/CS61C%EF%BD%9CLec7-RISC-V%20Functions-20250122-5.png)

![CS61C｜Lec7-RISC-V Functions-20250122-6.png](../../assets/images/CS61C%EF%BD%9CLec7-RISC-V%20Functions-20250122-6.png)

```C
int sumSquare(int x, int y) {  
	return mult(x, x) + y;  
}
```

```
sumSquare:  
	addi  sp, sp, -8   # space on stack (push)  
	sw    ra, 4(sp)    # save ret addr  (push)  
	sw    a1, 0(sp)    # save y         (push)  
	mv    a1, a0       # mult(x, x)  
	jal   mult         # call mult  
	lw    a1, 0(sp)    # restore y      (pop)  
	add   a0, a0, a1   # mult() + y  
	lw    ra, 4(sp)    # get ret addr   (pop)  
	addi  sp, sp, 8    # restore stack  (pop)  
	jr    ra
```

# Basic Structure of a Function

```
Prologue
	func_label:
	addi sp,sp, -framesize
	sw ra, <framesize-4>(sp)
	#store other callee saved registers
	#save other regs if needbe
Body(call other functions...)
	...
Epilogue
	#restore other regs if need be
	#restore other callee saved registers
	lw ra, <framesize-4>(sp)
	addi sp,sp, framesize
	jr ra
```

## Stack during function execution

![CS61C｜Lec7-RISC-V Functions-20250122-7.png](../../assets/images/CS61C%EF%BD%9CLec7-RISC-V%20Functions-20250122-7.png)




