---
title: 2024年暑期（6到9月）学习心得
date: 2024-09-15 00:33:58 +08:00
filename: 2024-09-15-2024-summer-study-summary
categories:
  - Record
  - StudySum
tags:
  - Summary
dir: Record/StudySum
share: true
---
为什么挑6-9月作为一个时间阶段？

因为对于在3月还几乎是0基础（算了其实就是0基础）的我来说，前三个月的时间基本上都花在了学习基础语法上了，并没有什么做出来的项目。所以就挑6-9月（~~致敬大耳朵~~），也刚好是暑假期间的所学所感，正巧也记录下我人生中第一个没有虚度的暑假吧。
## 6月份主要把时间花在gitlet上

### **gitlet**：

一个轻量版的git，我只拿到了基础的学分，也就是在本地进行管理的部分，额外的学分是关于远程仓库的。

作为今年开年以来一直向往的目标，在经过长达3个月的学习之后（写61b的时间可能没这么多，纯粹是我是javaweb和61b一起学的），终于在6月19号完成，大约用时一周半，不过做gitlet的期间是从早肝到晚的。

- **那么经历了痛苦的阅读文档，设计思路和不断重构之后我学到了什么？**

1. 首先我得到最宝贵的东西一定是作为计算机系学生的信心（但是这个信心又在我学s081操作系统的时候被打碎了），作为刚开始正儿八经学习计算机三个月的我来说，我完全不敢相信我能完成gitlet，老实说我在开始写项目之前心里一直很畏惧，首先我需要阅读约1.4w字的说明文档，还是英文！！这对于之前的我来说根本不敢想，但是现在遇到了全英文的文档我也没有那么害怕了（虽然看不懂还是得用翻译）。

2. 对于设计思路以及代码风格的理解。在开始gitlet之前，文档上有一个非常重要的要求就是，必须！！先给出你的设计思路。但是作为蹭ucb课的学生，我们并没有助教的压力，何况写完了设计文档也不知道给谁看。于是我并没有重视这一点，所以想到什么写什么。这导致了我在完成后续功能的时候不得不重构之前写的东西。于是感觉整个程序是在缝缝补补的基础上构建起来的，**极其不优雅**。而且ucb的自动评分机加入了代码风格判断，一开始我并不知道，导致后面改代码风格改的很幸苦，而且这个自动评分机还会限制你的代码行数，导致你不得不把一些很大的功能拆分成几个小方法，也就是解耦。以前我完全不在乎代码风格，因为受过这次洗礼开始慢慢在意起来了。不过现在我的代码仍然不够优雅，还是需要慢慢积累吧～

### **JavaWeb**

学习javaweb主要是因为兴趣+刚好可以完成数据库课程设计。

在经历了枯燥且抽象的javase学习之后，我们终于可以看到java在实际生活中的运用了！这对我来说充满着诱惑，于是我在5月份用了一周半爆肝看完了30小时的javaweb入门教程（史无前例的效率，再也见不到了），又用了一周半完成了我的第一个web项目，并部署到了我的服务器上。不过学习的都是老框架老技术，没有涉及到现在主流的Spring全家桶，也只是个小玩具罢了。

![2024年暑期（6到9月）学习心得-20250122.jpg](../../../assets/images/2024%E5%B9%B4%E6%9A%91%E6%9C%9F%EF%BC%886%E5%88%B09%E6%9C%88%EF%BC%89%E5%AD%A6%E4%B9%A0%E5%BF%83%E5%BE%97-20250122.jpg)

![2024年暑期（6到9月）学习心得-20250122-1.jpg](../../../assets/images/2024%E5%B9%B4%E6%9A%91%E6%9C%9F%EF%BC%886%E5%88%B09%E6%9C%88%EF%BC%89%E5%AD%A6%E4%B9%A0%E5%BF%83%E5%BE%97-20250122-1.jpg)

## 7月 学校实训加旅游，没有学太多，刷了leetcode数据结构的五十题

### **感觉数据结构的学习还是要刷题**：

虽然前面学习的61B在名义上是教授数据结构与算法的课，但是我感觉更偏向于软件工程的课，以至于完成gitlet之后我都不知道我用了哪些数据结构和算法。在七月份我突然想刷一点leetcode，于是便选择了从基础的数据结构与算法开始刷起。

关于链表，哈希表字符串之类的我感触不是很深，收获最大的还是要在二叉树这一块。由于涉及到很多递归，所以一直很困惑的递归在这一块终于得到了更深的理解。明白了栈帧和回溯思想，另外经常还需要使用栈来模拟递归，所以对于栈的认识也更上一层楼。不过总的来说学的都是基础的数据结构与算法。

## 8月至今 完成CS61C大部分内容开始进行操作系统的学习

### 61C心得

从八月开始，我便正式开始了CS61C，体系结构的学习。这门课是自顶（C语言）向下（到汇编再到机器码再到硬件体系设计再到逻辑门）再向上（再从数字电路讲到 CPU 的硬件实现，解释了机器语言是如何被执行的，后面还提到了Cache，操作系统和并发）。学习完之后有一种知识被打通的感觉。

在C语言部分，Project1是实现一个贪吃蛇，这一块主要是熟悉C语言，以及学会debug工具gdb和内存检测工具valgrind的基础用法。因为个人对读写不太熟悉，所以感觉时间都花在研究读写的buffer size上去了（因为有时候需要动态扩容）。

Project2是使用RISCV汇编来实现一个手写文字识别的神经网络。听起来挺高大上的，实际上需要我们完成的部分大概也就是个矩阵乘法。主要是让大家体会一下“Calling Convention”，让我更清楚的了解编程语言在进行函数调用的时候发生了什么——调用的时候要先在return address寄存器里面填好下一条指令的地址，然后保存当前函数里面的易失寄存器到栈（如果有用到的话，主要是如果是在一个函数内（caller）调用另一个函数（callee），那么第一个函数的参数可能会丢失，没错参数寄存器也是易失的）。然后进入到调用的函数（callee）里面，完成之后返回到刚刚保存的地址。除了函数的调用过程之外，我还有一点感受很深，就是写汇编的时候寄存器是真的不够用啊(汗)。

Project3是实现一个二级流水线并且处理了因分支带来阻塞的RISCV CPU，这也是我在学习61C的时候最想做的项目（因为感觉很高大上）。

这个项目很好的总结了前面学习到的知识（关于硬件设计的部分），有一些听课没听懂的内容，在这里也得到了更好的理解。虽然说是最期待的项目，但是真正写起总结来却感觉没什么好写的，主要是让我明白了指令是如何由机器码变成执行的命令吧。现将机器码输入，再利用控制器点亮对应的控制信号，以此来控制CPU对不同的指令进行不同的处理。真正完成之后感觉挺toy的，没有想象的那么可怕（因为根本没处理到难的那一部分）。

下面是一些看起来高大上（自认为）的图片

![2024年暑期（6到9月）学习心得-20250122.png](../../../assets/images/2024%E5%B9%B4%E6%9A%91%E6%9C%9F%EF%BC%886%E5%88%B09%E6%9C%88%EF%BC%89%E5%AD%A6%E4%B9%A0%E5%BF%83%E5%BE%97-20250122.png)

![2024年暑期（6到9月）学习心得-20250122-1.png](../../../assets/images/2024%E5%B9%B4%E6%9A%91%E6%9C%9F%EF%BC%886%E5%88%B09%E6%9C%88%EF%BC%89%E5%AD%A6%E4%B9%A0%E5%BF%83%E5%BE%97-20250122-1.png)

![2024年暑期（6到9月）学习心得-20250122-2.png](../../../assets/images/2024%E5%B9%B4%E6%9A%91%E6%9C%9F%EF%BC%886%E5%88%B09%E6%9C%88%EF%BC%89%E5%AD%A6%E4%B9%A0%E5%BF%83%E5%BE%97-20250122-2.png)

另外还学习到了一个重要思想——**指令是需要硬件实现支持的**，以前我对这一块没有概念，完成这个项目之后我才真正理解了为什么精简指令集的CPU比复杂指令集更省电，原来是从硬件上就更简单啊（恍然大悟），当然还包括RISC本身就是面向流水线设计这一重要特点，使得精简指令集的CPU通常拥有更深的流水线设计，提高了指令执行效率。

再往后就是Cache这一块了，了解了计算机运行背后的原理之后，我可以更好的利用时间局部性和空间局部性来写代码了。

### 6.S081

大名鼎鼎的操作系统课，围绕着基于 RISC-V 开发了一个新的教学用操作系统 xv6展开。不过因为我刚开始，所以感悟暂时写不了。现阶段的感受就是好难啊（汗），lab手册都没有一步一步带你走，就是叫你看代码，然后自己想。入手之后感觉cs61系列是真的友好。。。

## 另一大收获，折腾服务器

因为小时候玩我的世界，看到主播们经常自己开服，所以小时候的我便很向往有一台自己的服务器，开游戏服或者搭建网站。

今年，我终于拥有了自己的服务器了！折腾完之后现在是2台云服务器，一台本地做了内网穿透的旧笔记本做的服务器。目前没想到太多应用，可能也有懒得弄了的缘故，只有一个博客网站，一个由内网穿透而来的codeserver（网页版的vscode），一个自建的rustdesk服务器（用于远程控制电脑的软件）。

![2024年暑期（6到9月）学习心得-20250122.jpeg](../../../assets/images/2024%E5%B9%B4%E6%9A%91%E6%9C%9F%EF%BC%886%E5%88%B09%E6%9C%88%EF%BC%89%E5%AD%A6%E4%B9%A0%E5%BF%83%E5%BE%97-20250122.jpeg)

作为兴趣，我学习到了ssh，docker（一些服务有现成的容器，很方便的就能在服务器上部署起来），内网穿透，域名解析，端口映射。也尝试了不同的网站框架（总之现在就是很想学习一下前端知识，感觉又可以写网站又可以做跨平台开发）。

我的博客，，平时会写一些笔记发上去。

![2024年暑期（6到9月）学习心得-20250122-3.png](../../../assets/images/2024%E5%B9%B4%E6%9A%91%E6%9C%9F%EF%BC%886%E5%88%B09%E6%9C%88%EF%BC%89%E5%AD%A6%E4%B9%A0%E5%BF%83%E5%BE%97-20250122-3.png)

codeserver，通过网页访问的vscode，现在只需要用平板和键盘便可以随时随地写代码了（只是幻想罢了，，实际上用的很少）

![2024年暑期（6到9月）学习心得-20250122-4.png](../../../assets/images/2024%E5%B9%B4%E6%9A%91%E6%9C%9F%EF%BC%886%E5%88%B09%E6%9C%88%EF%BC%89%E5%AD%A6%E4%B9%A0%E5%BF%83%E5%BE%97-20250122-4.png)

最后附上一张我的本地服务器，作为笔记本功耗低还有电池不怕断电，真不错。

![2024年暑期（6到9月）学习心得-20250122-1.jpeg](../../../assets/images/2024%E5%B9%B4%E6%9A%91%E6%9C%9F%EF%BC%886%E5%88%B09%E6%9C%88%EF%BC%89%E5%AD%A6%E4%B9%A0%E5%BF%83%E5%BE%97-20250122-1.jpeg)