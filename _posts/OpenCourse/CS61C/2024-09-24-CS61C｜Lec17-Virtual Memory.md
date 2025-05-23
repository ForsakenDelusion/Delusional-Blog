---
title: CS61C｜Lec17-Virtual Memory
date: 2024-09-24 00:33:58 +08:00
filename: 2024-09-24-CS61C｜Lec17-Virtual Memory
categories:
  - OpenCourse
  - CS61C
tags:
  - CS61C
  - OS
dir: OpenCourse/CS61C
share: true
---
**虚拟内存的主要功能**：

- **大内存的幻觉**：虚拟内存使得程序看起来可以访问一个非常大的主存。程序的**工作集**（即正在活跃使用的内存页面）保存在物理内存中，而不常用的页面则保存在磁盘上。

- **请求分页（Demand Paging）**：虚拟内存通过请求分页技术，让程序运行的内存可以超过物理内存的大小。当程序需要访问某个不在主存中的页面时，会触发页面调度，将所需页面从磁盘加载到主存。

**虚拟内存的其他功能**：

- **配置差异隐藏**：虚拟内存可以隐藏不同机器配置的差异。无论物理内存的大小是多少，程序在执行时都以为自己有足够的内存。

- **内存共享与保护**：虚拟内存还允许操作系统在多个进程之间共享内存，同时保证进程之间的隔离，防止一个进程访问或破坏另一个进程的内存。这使得操作系统可以更有效地管理和分配内存。

- **重要的保护功能**：在现代系统中，虚拟内存的保护功能甚至比其作为内存层次结构的一部分更为重要。它能防止程序访问不属于它的内存空间，提供了额外的安全性。

- **虚拟内存的历史**：

- 虚拟内存的概念**早于缓存**，最初设计时主要是为了解决多线程问题。

![CS61C｜Lec17-Virtual Memory-20250122.png](../../../assets/images/CS61C%EF%BD%9CLec17-Virtual%20Memory-20250122.png)

## 虚拟地址与物理地址

![CS61C｜Lec17-Virtual Memory-20250122-1.png](../../../assets/images/CS61C%EF%BD%9CLec17-Virtual%20Memory-20250122-1.png)

1. **虚拟地址和物理地址**

- **虚拟地址（Virtual Addresses）**：这是程序在执行时看到的地址。每个进程会认为自己拥有完整的内存空间，例如从地址 0x00000000 到 0xffffffff。

- **物理地址（Physical Addresses）**：这是实际内存的地址，也可能是从 0x00000000 到某个上限。物理内存的大小有限，通常比虚拟地址空间小得多。

2. **地址转换**

- **地址映射**：虚拟内存系统通过**内存管理单元（Memory Management Unit, MMU）**将虚拟地址映射到物理地址。每个进程都有独立的虚拟地址空间，但这些虚拟地址可以映射到不同的物理地址。

- **虚拟地址转换到物理地址**的过程由操作系统和硬件共同管理，操作系统会负责分页和换页，MMU会实时进行地址转换。

3. **进程地址空间**

- 每个进程的虚拟地址空间通常包括：

- **堆栈（Stack）**：存放函数调用的局部变量、返回地址等。

- **堆（Heap）**：动态分配的内存区域，通常通过如 malloc() 的函数获取。

- **静态数据区（Static Data）**：包括全局变量、静态变量等。

- **代码段（Code）**：存放程序的指令。

图中的不同颜色块代表了程序的不同部分如何映射到虚拟地址空间。未使用的内存区域用来为进程提供足够的扩展空间。

4. **核心观点**

- **多进程并发**：多个进程可以同时运行，并且每个进程都使用相同的虚拟地址范围。例如，多个进程的虚拟地址空间都从 0x00000000 开始，但它们的虚拟地址会映射到不同的物理地址。因此，即使地址空间有冲突，虚拟内存系统依然能确保进程之间互不干扰。

- **保护功能**：虚拟内存不仅实现了内存管理，还提供了内存保护功能，使得每个进程都无法访问其他进程的内存。

## 地址空间

地址空间是所有可用内存的地址集合

**虚拟地址空间：**

- 用户程序能看见的地址集合，程序在执行的时候可以认为它可以访问整个虚拟地址空间的内存。

**物理地址空间：**

- 这是指实际的物理内存中的地址集合。与虚拟地址不同，物理地址指向实际的硬件内存位置。

- 物理地址是用户程序所看不到的，通常由操作系统和硬件来管理。

**内存管理器（Memory Manager）负责在虚拟地址空间**和**物理地址空间**之间进行地址映射或转换。这个过程通常由硬件中的**内存管理单元（MMU）来实现。

- 当程序要访问某个虚拟地址时，MMU 会根据操作系统维护的页表（Page Table）将该虚拟地址映射到实际的物理地址。

- 这种映射方式使得程序可以运行在一个“假想”的大内存空间中，而实际上访问的物理内存可能比虚拟内存小得多。

## 图书馆类比

1. **虚拟地址 vs. 书名**

- **虚拟地址**类似于**书名**，它是用户或程序所知的一个信息，就像你知道书的名字。

- 比如，用户程序会使用虚拟地址来引用数据，但它实际上不知道这些数据存储在哪里。

2. **物理地址 vs. 书的编号（Call Number）**

- **物理地址**就像**书的编号**（Call Number）**，它对应的是书的实际存放位置。

- 书的编号是唯一的，能够精确地找到某一本书在某个架子上的位置。

- 同样，物理地址是唯一的，指向实际的物理内存位置。

3. **页表 vs. 图书馆的卡片目录**

- **页表（Page Table）类似于图书馆的卡片目录**，它记录了每本书的书名和对应的编号（Call Number）。

- 页表的作用是将**虚拟地址（书名）映射到物理地址（编号）**，帮助操作系统找到数据在物理内存中的实际存放位置。

- 比如你想在另一个图书馆找到这本书，只需要利用书名（虚拟地址）在卡片目录（页表）找到它在这个图书馆里面的编号（物理地址）即可。

4. **有效位（Valid Bit） vs. 书籍是否在本馆**

- **有效位（Valid Bit）可以类比为卡片上写的书是否在本馆**的信息。

- 如果卡片上显示书在本馆（即有效位为有效），你就可以在图书馆的架子上找到它。

- 如果卡片上显示书不在本馆，而是在另一个分馆或已经借出（即有效位无效），你可能需要去别处获取书。这对应于计算机内存中的数据在磁盘上而不在主存中。

5. **访问权限（Access Rights） vs. 借阅权限**

- **访问权限（Access Rights）类似于图书馆卡片上写的借阅权限**。

- 有些书只能在图书馆内借阅，比如只能借2小时，而不能带回家。

- 在虚拟内存系统中，访问权限指的是程序能否对某块内存进行读取、写入或执行等操作。如果没有相应的权限，程序就不能对该内存块进行某些操作。

# 虚拟内存

1. **裸机系统（Bare Metal System）**

- 在没有操作系统的情况下（裸机系统），**处理器**直接发出物理地址进行内存的加载或存储操作。这意味着，程序使用的地址就是**物理地址**。

- **物理地址**：在裸机系统中，程序可以自由访问任何内存地址，这包括不属于它的内存区域，如操作系统数据结构或其他程序的内存区域。

2. **潜在问题：无保护机制**

- 在这种模式下，**任何进程**都可以发出任意地址的访问请求，甚至可以访问它不应该访问的内存区域。这会导致以下问题：

- **安全问题**：进程可能会不小心或恶意访问系统中的敏感数据，例如操作系统的数据结构，进而导致系统崩溃或数据泄漏。

- **内存冲突**：多个进程可能会访问同一块物理内存，导致数据混乱或程序错误。

3. **虚拟内存的引入**

- 为了防止这种问题，我们需要**将所有地址通过操作系统控制的机制**，在它们被发送到物理内存（DRAM）之前进行检查和翻译。这就是**地址翻译机制**的作用。

4. **地址翻译机制（Translation Mechanism）**

- 虚拟内存系统通过**地址翻译机制**将**虚拟地址**转换为**物理地址**，并在转换过程中进行一系列检查。

- **权限检查**：系统会检查进程是否有权限访问某个特定的内存区域。如果没有权限，则会触发**异常（exception）或拒绝访问。

- **地址映射**：虚拟地址不会直接对应物理内存，而是通过**页表（Page Table）映射到物理地址。操作系统和**内存管理单元（MMU）**共同负责这个过程。

5. **操作系统的控制**

- 通过虚拟内存，操作系统可以控制内存的访问权限。每个进程只能访问它自己的内存区域，而无法访问其他进程或操作系统的数据。

- **保护机制**：这是虚拟内存系统的关键功能之一，它确保每个进程只能访问自己的虚拟地址空间，从而实现内存保护。

![CS61C｜Lec17-Virtual Memory-20250122-2.png](../../../assets/images/CS61C%EF%BD%9CLec17-Virtual%20Memory-20250122-2.png)

## 内存管理器

概念

![CS61C｜Lec17-Virtual Memory-20250122-3.png](../../../assets/images/CS61C%EF%BD%9CLec17-Virtual%20Memory-20250122-3.png)

### 内存管理器的职责

1. 将虚拟地址映射到物理地址  

2. 保护：  

   - 隔离进程之间的内存  

   - 每个进程获得专用的“私有”内存  

   - 一个程序中的错误不会损坏其他程序的内存  

   - 防止用户程序干扰操作系统的内存  

1. 将内存交换到磁盘  

   - 通过将部分内容存储在磁盘上，给人一种更大内存的错觉  

   - 磁盘通常比DRAM大得多但速度慢  

   - 使用“聪明”的缓存策略

# 分页内存

## 分页

1. **分页内存的概念**

- 在现代操作系统中，**物理内存（DRAM）被分成许多固定大小的块，称为页（Pages）**。

- **页面（Page）是虚拟内存和物理内存之间进行映射的基本单位。

- 这种分页的方式帮助系统更好地管理内存，提高效率，并通过分离虚拟地址和物理地址来保护系统。

2. **典型页面大小**

- **典型页面大小**在现代操作系统中通常为**4 KiB**（4096字节）。

- 页面的大小决定了地址的分配方式。为了能够访问4 KiB内存，**需要12位**来表示页面内的偏移量（offset）。因为2^12 = 4096，也就是说，用12位可以精确地定位4 KiB的每一个字节。

3. **虚拟地址的结构**

- 以一个**32位的虚拟地址**为例：

- 这32位可以分成两部分：

- **页号（Page Number）**：高20位（为什么高20位用于表示页号，那是因为我们假设分页大小为4KB，那么就需要用低12位，也就是$2^{12}$用于所以4KB内的每一个字节，用32减去12就得到了20）用于表示虚拟内存中的页号。这20位能够表示2^20 = 1,048,576个页面，每个页面有4 KiB的大小。**页号是用于查找页表中的PTE，PTE则储存了将虚拟地址映射到物理地址的信息。**

- **页内偏移量（Offset）**：低12位用于表示该页面内的偏移量，这12位可以表示4 KiB页面内的每一个字节的位置。

![CS61C｜Lec17-Virtual Memory-20250122-4.png](../../../assets/images/CS61C%EF%BD%9CLec17-Virtual%20Memory-20250122-4.png)

4. **地址翻译过程**

- 当一个程序访问某个虚拟地址时，**内存管理单元（MMU）首先会从虚拟地址的页号**部分查找对应的物理页，然后通过**偏移量**定位页面中的具体位置。这个过程就完成了从虚拟地址到物理地址的转换。

5. **分页的优势**

- **内存保护**：分页机制可以隔离不同进程的内存，防止进程间的内存冲突。

- **高效的内存管理**：通过分页，系统可以更有效地利用内存，避免大块内存的浪费。

- **支持虚拟内存**：分页还支持操作系统将不常用的页面暂时存储到磁盘中，从而实现“虚拟内存”机制，允许程序使用比实际物理内存更大的地址空间。

![CS61C｜Lec17-Virtual Memory-20250122-5.png](../../../assets/images/CS61C%EF%BD%9CLec17-Virtual%20Memory-20250122-5.png)

## **页表项（Page Table Entry, PTE）**

- **定义**：页表项（PTE）是**页表**中的一项，它存储了虚拟页面和物理页面的映射信息，以及与页面状态有关的元数据。

- **主要内容**：

1. **物理页面地址**：PTE存储虚拟页面映射到的物理页面的地址。

2. **有效位（Valid Bit）**：表示该页面是否在内存中。如果无效，则可能意味着页面在磁盘中或未分配。

3. **读/写/执行权限**：PTE通常还包含页面的访问权限，控制该页面是否可读、可写或可执行。

4. **修改位（Dirty Bit）**：如果页面被修改过（写操作），这个位会被置为1，以便操作系统知道在页面换出时需要将其写回磁盘。

5. **引用位（Reference Bit）**：用于记录该页面是否最近被访问，用于页面替换算法（如LRU算法）。

- **功能**：

1. **地址映射**：PTE用于将虚拟页面号映射到物理页面号。

2. **状态控制**：通过有效位、读/写权限等，控制页面的状态和访问权限。

- **示例**：一个典型的PTE占用**4字节**（32位），其中部分位用于存储物理页面地址，其他位用于存储状态信息（如有效位、权限等）。

## 页表

**定义**：页表是用来存储**虚拟地址空间**中的每个**虚拟页面**到**物理页面**的映射关系的数据结构。操作系统为每个进程维护一个页表，确保虚拟地址能够正确转换为物理地址。

1. **页表（Page Table）与操作系统的管理**

- 操作系统（OS）负责管理多个进程的页表，并确保每个进程都使用正确的页表。

- 当某个进程被激活时，操作系统选择并加载该进程对应的页表。

- 页表记录了每个虚拟页面与物理页面之间的映射关系。

2. **地址转换过程**

- **内存管理单元（Memory Manager）或处理器的控制单元**，从虚拟地址中提取出**页号**。

- 例如，32位虚拟地址的**高20位**用于表示页号。

- 操作系统会根据页号在**页表**中查找对应的物理页面地址。这个查找过程通过页表中的**页表项（Page Table Entry, PTE）实现。

3. **计算物理地址**

- 得到物理页面地址后，系统还需要根据虚拟地址中的**偏移量**，将其加到物理页面地址上，得出完整的物理地址。

- **物理地址** = **物理页面地址** + **页内偏移量**

- 例如，如果虚拟地址的页号映射到一个物理页面，那么将偏移量加上这个物理页面的起始地址，就可以得到最终的物理地址。

4. **物理地址的位数**

- 物理地址的长度并不一定与虚拟地址相同。

- 物理地址可能比虚拟地址长（如支持更多物理内存），也可能短（如物理内存较少）。例如，虚拟地址为48位，而物理地址可能为39位等。

![CS61C｜Lec17-Virtual Memory-20250122-6.png](../../../assets/images/CS61C%EF%BD%9CLec17-Virtual%20Memory-20250122-6.png)

1. **通过分配不同的物理页面实现隔离**

- **内存隔离**：在分页内存管理中，操作系统（OS）通过为每个进程分配不同的物理页面，确保它们无法访问彼此的内存空间。

- 这种机制确保了每个进程的内存访问是独立的，避免了恶意或错误的进程访问其他进程的数据。

- **隔离（Isolation）的作用在于提高系统的安全性和稳定性。

2. **操作系统处理页表**

- **页表由操作系统管理**：每个进程都有自己的**页表**，操作系统通过这些页表跟踪虚拟页面与物理页面的映射关系。

- 操作系统运行在**超级模式（Supervisory Mode）下，只有操作系统内核有权限修改页表，用户进程无法直接操作页表。

- 通过控制页表，操作系统可以决定哪个虚拟地址映射到哪个物理地址。

3. **内存共享**

- **共享内存**：尽管操作系统通过分页实现了内存隔离，但在某些情况下，操作系统也可以允许多个进程共享同一个物理页面。

- 例如，多个进程可以共享某个只读的内存区域（如共享库、共享代码段）。

- 在这种情况下，操作系统会将多个进程的虚拟地址映射到同一个物理页面，达成**内存共享**的目的。

**写保护**

如果尝试写入一个受保护的页面会引发异常

![CS61C｜Lec17-Virtual Memory-20250122-7.png](../../../assets/images/CS61C%EF%BD%9CLec17-Virtual%20Memory-20250122-7.png)

## 页表在哪儿？

1. **页表的大小**

- 以**32位虚拟地址**和**4 KiB页面**为例：

- 一个虚拟地址空间是2^32字节（4 GiB），每个页面大小为4 KiB，因此总共有2^20个页面（因为 4 GiB ÷ 4 KiB = 2^20）。

- 每个页表项需要**4字节**来存储虚拟页号和对应的物理页面信息。

	- **页表项**是页表中的一项，用于记录虚拟页面到物理页面的映射。每个PTE通常包含以下信息：
	
	- **物理页面的地址**：将虚拟页面映射到物理内存。
	
	- **状态位**：例如有效位、权限位等。
	
	- 每个页表项的大小在许多系统中为**4字节（32位）**，这意味着每个PTE占用**4字节**的空间。

- 因此，**单个页表的大小**为 4字节 × 2^20页 = 4 MiB。

- 尽管**4 MiB**只占4 GiB虚拟内存的**0.1%**，但这对于**缓存（Cache）来说依然过大。

2. **页表存放在内存中（DRAM）**

- 页表存储在**主存（DRAM）中，而不是缓存中，因为页表体积较大，放入缓存会占用过多空间。

- 当处理器访问某个虚拟地址时，它必须通过页表找到对应的物理地址。如果数据不在缓存中，需要：

1. 访问DRAM中的页表，找到页表项（慢）。

2. 再次访问DRAM，获取数据（慢）。

- 因此，每次缺页时，处理器需要进行**两次内存访问**，这会导致性能下降。

3. **如何减少性能损失？**

为了减少由于多次内存访问导致的性能损失，常用的方法有以下几种：

1. **传输块而不是字（利用空间局部性）**

- **空间局部性**：程序访问的内存地址通常在同一块内存区域中。因此，操作系统可以将**内存块**（而不仅仅是一个字）从DRAM传输到处理器缓存中。

- 这样，在后续访问中，整个内存块都可能已经在缓存中，从而减少了未来的缓存未命中。

2. **为页表项设置缓存（TLB，Translation Lookaside Buffer）**

- 处理器通常会使用一种特殊的缓存，称为**页表缓存（TLB，Translation Lookaside Buffer）**，用来存储最近访问的页表项。

- 当访问虚拟地址时，处理器首先查询TLB，如果找到对应的页表项（**TLB命中**），就无需再访问DRAM，减少了两次内存访问的开销。

- **TLB**缓存了常用的**页表项**，有效降低了分页带来的性能损失。

所以，页表存在内存中，进程通过页表找到分页。

![CS61C｜Lec17-Virtual Memory-20250122-8.png](../../../assets/images/CS61C%EF%BD%9CLec17-Virtual%20Memory-20250122-8.png)

## 字节，字，块，页

![CS61C｜Lec17-Virtual Memory-20250122-9.png](../../../assets/images/CS61C%EF%BD%9CLec17-Virtual%20Memory-20250122-9.png)

## 内存访问

![CS61C｜Lec17-Virtual Memory-20250122-10.png](../../../assets/images/CS61C%EF%BD%9CLec17-Virtual%20Memory-20250122-10.png)

1. **检查页表项的有效性**

当处理器访问内存时，它首先需要通过页表找到对应的物理页面。在这个过程中，它会检查页表项的**有效位（Valid bit）**，以确定该虚拟页是否有对应的物理页。

1. **如果页表项有效**（Valid bit = 1）：

	- 进一步检查该页是否已经在**DRAM**中。
	
	- **在DRAM中**：可以直接访问物理地址并进行数据的**读/写**操作。
	
	- **不在DRAM中**（即该页在**磁盘**上）：需要操作系统介入，通过**页面置换**机制将该页从磁盘调入内存。**页面错误  操作系统干预**

2. **如果页表项无效**（Valid bit = 0）**页面错误  操作系统干预**：

	- 操作系统会分配一个新的物理页面到内存中，并更新页表。
	
	- 如果内存已满，则需要进行页面置换。

2. **页面置换过程**

- **分配新页面时如果内存已满**：

	1. 操作系统会选择一个**页面**从DRAM中**驱逐（Evict）**。
	
	2. 被驱逐的页面如果有修改，会被写回到磁盘保存。
	
	3. 操作系统将所需的页面从**磁盘**中读取到**DRAM**中。
	
	4. 更新页表以反映虚拟地址到物理地址的新映射。

## 分页错误

1. **页面错误是作为异常处理的**

- 当处理器访问某个虚拟地址对应的页面时，如果该页面**不在内存中**（例如，页面在磁盘上或页表项无效），会触发**页面错误（Page Fault）**。

- **页面错误**是一种特殊的异常，由处理器产生，操作系统通过异常处理程序来响应。

2. **页面错误处理程序（Page Fault Handler）**

- 处理页面错误的工作是由**页面错误处理程序（Page Fault Handler）完成的。这是操作系统中用于处理缺页的模块，通常作为中断或异常处理程序**的一部分。

- 页面错误处理程序主要负责以下工作：

1. **更新页表**：查找或分配新的页面，并更新页表以反映虚拟地址到物理地址的映射。

2. **启动数据传输**：如果所需的页面在磁盘上，则开始从磁盘加载页面到内存的过程。

3. **更新状态位**：处理器或操作系统会更新与页面状态相关的位（例如，**有效位（Valid bit）**、**已修改位（Modified bit）等），以跟踪页面在内存中的状态。

3. **如果页面需要从磁盘调入**

- **上下文切换（Context Switch）**：如果要从磁盘加载页面，可能会导致较长的延迟。在这种情况下，操作系统可以通过**上下文切换**将当前正在执行的进程**挂起**，并切换到另一个可运行的进程继续执行。

- 上下文切换是一种机制，操作系统保存当前进程的状态，并切换到另一个进程，从而在等待I/O操作（如从磁盘加载页面）完成时，不浪费CPU的处理时间。

4. **重新执行导致页面错误的指令**

- 当页面错误处理完成后，操作系统会让处理器重新执行导致页面错误的指令。这样，程序可以继续执行，不会丢失指令或数据。

- 在重新执行时，所需的页面已经在内存中，因此不再会触发页面错误。

## 写回还是写穿

1. **写穿式（Write-Through）**：每次写入都同步到磁盘，数据一致性高，但性能较差。

2. **写返回式（Write-Back）**：只有页面驱逐时才写回磁盘，性能较高，但存在数据丢失风险。

3. **建议**：大多数情况下，回写式更合适，特别是性能敏感的系统。写穿式适用于对数据一致性要求极高的系统。

# 分级页表

## 页表大小

每一个页表：

$4 \times 2^{20}Bytes = 4 MiB$

但是每一个进程都需要一个页表，现代操作系统一般有256个进程

所以我们需要

$256 \times 4 = 1 GiB$

占到4G运存的四分之一！太多了

而且现在有很多64位操作系统，会导致页表大小激增

所以我们需要一个更好的存储页表的方法。

## 优化思路

1. **分层页表（Hierarchical page tables）**

- 通过减少页面的大小，操作系统可以采用多级（分层）的页表结构。这样可以有效地减少内存占用，因为大多数程序并不需要使用整个地址空间。

- 多级页表的结构也使得查找虚拟地址的映射更加高效，因为它允许系统动态加载部分页表，而不是一次性加载整个表。

2. **大多数程序只使用部分内存（Most programs use only a fraction of memory）**

![CS61C｜Lec17-Virtual Memory-20250122-11.png](../../../assets/images/CS61C%EF%BD%9CLec17-Virtual%20Memory-20250122-11.png)

- 通常情况下，程序不会使用整个地址空间中的所有内存。因此，操作系统可以将页表拆分为多部分，只为实际使用的部分内存分配页表项。

- 例如，RISC-V体系结构中就采用了这种分段的页表设计，允许在更大范围的虚拟地址空间内只映射实际用到的部分。

稍后我们会更深入的讨论这一优化机制

## 分级页表

虚拟地址空间稀疏性（Sparsity）的利用，以及分层页表的工作机制。

1. **虚拟地址空间的稀疏性**

大多数程序不会使用整个虚拟地址空间中的所有内存。程序通常只会用到堆（Heap）、栈（Stack）和代码段（Code segment）中的一些部分，而其他虚拟地址空间可能是未使用的。这种未使用的虚拟地址空间通常很大，比如操作系统为每个进程分配的虚拟内存可能远大于物理内存。

2. **为什么需要分段/多级页表？**

- 如果我们使用**单级页表**，每个虚拟地址都需要对应一个页表项（PTE），无论该虚拟地址是否被程序实际使用。这样的话，页表会变得非常庞大，浪费内存。

- **多级页表**（或分段页表）的设计可以针对这种稀疏的虚拟地址空间进行优化：只为实际使用的部分内存创建页表项，从而节省了存储页表的开销。

![CS61C｜Lec17-Virtual Memory-20250122-12.png](../../../assets/images/CS61C%EF%BD%9CLec17-Virtual%20Memory-20250122-12.png)

1. **虚拟地址的拆分**

- 虚拟地址由三部分组成：

- 页号 p1（前10位）：用于一级页表（L1 页表）的索引。

- 页号 p2（中间10位）：用于二级页表（L2 页表）的索引。

- 页内偏移量 offset（最后12位）：用于标识当前页内的字节位置。

2. **分层页表结构**

- **一级页表（L1 Page Table）**：

- 虚拟地址的前10位 p1 指向一级页表中的一个条目，该条目包含了指向二级页表的指针。

- 操作系统在“监督模式”下管理页表，并将一级页表的根存储在“监督页表基址寄存器”（SPTBR）中，如图中RISC-V的实现。

- **二级页表（L2 Page Table）**：

- 一级页表指向二级页表，虚拟地址的中间10位 p2 用作二级页表的索引，从而找到物理页面的起始地址。

3. **页内偏移量（Offset）**

- 虚拟地址的最后12位用于标识页面内的具体字节位置。这12位偏移量直接加到找到的物理页地址上，计算出物理内存中具体的字节位置。

4. **物理内存（Physical Memory）**

- 图的右边展示了物理内存的页面。每个页表项（Page Table Entry，PTE）可以指向物理内存中的一个页面。如果该页表项不存在或指向的页面不在内存中，可能需要从磁盘中读取数据。

5. **Supervisor Page Table Base Register (SPTBR)**

- 该寄存器存储了一级页表的根地址。当处理器发出内存访问请求时，它使用此寄存器中的地址来查找页表。

## 利用缓存加速页表查询

VPN Virtual Page No.
PPN Physical Page No.

![CS61C｜Lec17-Virtual Memory-20250122-13.png](../../../assets/images/CS61C%EF%BD%9CLec17-Virtual%20Memory-20250122-13.png)

这是虚拟页号到物理页号的转换过程，每条指令和数据访问都需要进行地址转换和保护检查。

一个好的虚拟内存设计应该要求快（一周期内）且空间利用率高

地址转换是非常耗时间的，如果每次都要去内存中拿东西，那么时间开销太大了。一级页表每次引用需要访问两次内存。二级页表需要访问三次内存。

所以我们引入**Translation-lookaside buffer(TLB) 块表（也称旁视缓冲器）**

![CS61C｜Lec17-Virtual Memory-20250122-14.png](../../../assets/images/CS61C%EF%BD%9CLec17-Virtual%20Memory-20250122-14.png)

1. **TLB 的条目数量与关联性**

- **TLB**（Translation Lookaside Buffer） 是处理器中的一部分，用来加速虚拟地址到物理地址的转换。它是一个存储最近使用的**页表项**的缓存，位于处理器内部。

- **通常包含 32-128 个条目**，即它可以存储 32 到 128 个虚拟页面的地址映射。

- **完全关联（fully associative）**：这意味着 TLB 的每个条目都可以与任何虚拟页面号进行匹配，而不需要固定在某个特定的槽位上。完全关联的设计可以最大化匹配的灵活性，但硬件复杂性较高。

2. **较少的空间局部性（spatial locality）**

- **空间局部性**指的是程序访问的内存位置往往在空间上是接近的。比如，如果你访问了某个地址，那么很有可能你下次访问的地址就在这个地址附近。

- TLB 每个条目映射一个较大的页面，因此当跨越多个页面访问数据时，空间局部性可能较弱，导致两个页面条目可能发生冲突。

- 如果多个虚拟页面映射到相邻的物理地址，但这些页面在 TLB 中有不同的条目时，可能会导致**冲突**，即 TLB 需要不断替换。

3. **集合关联（set-associative）TLB**

- 有些系统中的 TLB 会更大，比如**256 到 512 个条目**，并且采用**4 路到 8 路集合关联（set-associative）**结构。

- **集合关联**的意思是，TLB 中的条目被划分为多个“组（sets）”，每组包含若干个条目。在每次查找时，系统只会在某一个特定组中查找对应的页表条目，而不是在整个 TLB 中查找。这种设计在权衡硬件复杂性和速度方面表现较好。

4. **多级 TLB（L1 和 L2 TLB）**

- 大型系统（如高性能服务器）可能会有多级 TLB。最常见的是**L1 和 L2 两级 TLB**。

- **L1 TLB**：快速、小型，通常在处理器核心内，能够快速处理大部分的地址翻译请求。

- **L2 TLB**：稍大、稍慢，作为 L1 TLB 的后备，当 L1 没有找到地址时，L2 TLB 可以提供帮助。

- 通过多级设计，可以在速度和容量之间进行良好的平衡。

5. **TLB 替换策略**

- TLB 中的条目是有限的，因此需要替换策略来决定哪些条目在缓存满时被替换掉。

- 常见的替换策略有：

- **随机替换（Random Replacement）**：随机选择一个条目进行替换，简化硬件实现。

- **FIFO 替换（First-In-First-Out）**：这是比较常见的策略，最早进入 TLB 的条目最先被替换掉，因为我们希望保留最新的内容。 

6. **TLB Reach（TLB 可达范围）**

- **TLB Reach** 是指 TLB 能够同时映射的最大虚拟地址空间的大小。它是**TLB 中的条目数量**和**每个条目映射的页面大小**的乘积。

- **TLB Reach** 越大，意味着 TLB 能够缓存的虚拟地址空间范围越大，这样在地址翻译过程中，TLB 缺失（TLB Miss）的概率就越低。

- 举例来说，如果 TLB 中有 128 个条目，每个条目映射 4 KiB 的页面，那么 TLB Reach 就是 128 * 4 KiB = 512 KiB。这意味着程序的虚拟地址空间中有 512 KiB 的区域可以直接通过 TLB 进行快速映射。

## 关于TLB

![CS61C｜Lec17-Virtual Memory-20250122-15.png](../../../assets/images/CS61C%EF%BD%9CLec17-Virtual%20Memory-20250122-15.png)

1. **先检查 Cache 还是 TLB？**

- 问题是：我们应该先检查 Cache 还是先检查 TLB？

- 答案是：**应该先检查 TLB**。因为**Cache** 使用的是物理地址（PA），而**TLB** 的任务是将**虚拟地址（VA）** 转换为**物理地址（PA）**。如果没有物理地址，Cache 是无法工作的。

2. **缓存（Cache）和 TLB 交互过程**

- **如果直接检查 Cache**

- Cache 中保存的是物理内存中的数据。由于程序访问内存时使用的是虚拟地址（VA），所以在没有转换成物理地址（PA）之前，我们无法确定 Cache 是否包含所需的数据。

- **问题**：如果页面不在物理内存中，Cache 是无法持有相关数据的。因此，直接检查 Cache 是没有意义的，必须先进行地址转换。

- **TLB 先进行地址转换**

- 当 CPU 请求一个虚拟地址（VA）时，**TLB** 首先检查是否有这个虚拟地址的映射记录。

- **TLB 命中**：如果 TLB 中有该虚拟地址的映射（TLB hit），则直接将其转换为物理地址（PA），然后再去查询 Cache。

- **TLB 缺失**：如果 TLB 中没有该映射（TLB miss），则需要查找页表（Page Table）来进行地址转换，并将映射加载到 TLB 中。

- 最后，使用物理地址（PA）去访问 Cache 和内存。

3. **TLB 替代了页表的作用**

- 由于**TLB** 可以缓存常用的页表项，因此在大多数情况下，**TLB** 直接执行地址转换的任务，而不需要频繁访问存储在内存中的**页表**。

- 页表的作用仍然存在，但它只在**TLB miss** 时才被访问，显著提高了系统的效率。

TLB的命中率通常非常高，达到99%，甚至99.99%

## 地址转换过程

![CS61C｜Lec17-Virtual Memory-20250122-16.png](../../../assets/images/CS61C%EF%BD%9CLec17-Virtual%20Memory-20250122-16.png)

虚拟地址和物理地址的TIO没有关系，虚拟地址用于寻址页表，物理地址则用于直接查找内存或者缓存中的数据。

## TLB数据通路

![CS61C｜Lec17-Virtual Memory-20250122-17.png](../../../assets/images/CS61C%EF%BD%9CLec17-Virtual%20Memory-20250122-17.png)

**页错误（Page Fault）**

- 页错误发生在**虚拟地址所对应的页面不在内存中**时，例如页面在磁盘上，需要从磁盘加载到内存。

- **处理页错误**：

- 当页面不在内存中时，操作系统会触发一个 **精确的陷阱（Precise Trap）**，即产生一个 **中断**，使得操作系统可以暂停当前程序，调度一个页错误处理程序。

- **处理过程**：

1. 操作系统会找到所需的页面，并从磁盘中读取该页面。

2. 将页面加载到内存，并更新页表。

3. TLB 也可能会更新相应的条目。

4. 完成后，系统恢复执行之前被暂停的指令。

- **精确的陷阱** 允许操作系统准确地捕捉到错误发生的位置，并在页面加载后无缝地恢复执行。

**保护违规（Protection Violation）**

- **保护违规**是指某个进程尝试访问它无权访问的内存区域，例如试图写入只读页面，或访问被操作系统保护的内存区域。

- **处理保护违规**：

- 当发生保护违规时，操作系统会立即中止进程，以确保系统的安全和稳定。这可能会导致程序崩溃或关闭。

![CS61C｜Lec17-Virtual Memory-20250122-18.png](../../../assets/images/CS61C%EF%BD%9CLec17-Virtual%20Memory-20250122-18.png)

![CS61C｜Lec17-Virtual Memory-20250122-19.png](../../../assets/images/CS61C%EF%BD%9CLec17-Virtual%20Memory-20250122-19.png)

## 上下文切换

1. **如何让单个处理器运行多个程序？**

- **时间片轮转（time-slicing）**：处理器将时间划分成多个**时间片**，每个进程在一个时间片内运行。当时间片结束时，操作系统会强制暂停当前进程，切换到下一个进程。这种切换被称为**上下文切换**（context switching）。

- 每个程序并不是**同时运行**，而是在处理器上快速交替执行，给用户一种它们同时运行的错觉。

2. **上下文切换的过程**

- **上下文切换**是指从一个进程切换到另一个进程时，保存当前进程的状态并加载新进程的状态。

- 操作系统必须**保存当前进程的寄存器值**，包括**程序计数器（PC）**，以及其他关键寄存器的内容（如通用寄存器、状态寄存器）。

- 还需要更新**Supervisor Page Table Base Register (SPTBR)**，这是指向当前进程的**页表**的寄存器。因为不同的进程有各自独立的虚拟内存空间，所以切换进程时也要更新页表的指针。

3. **TLB（Translation Lookaside Buffer）在上下文切换中的作用**

- **TLB** 是一个缓存虚拟地址到物理地址映射的硬件表，用来加快地址转换过程。

- 每个进程有自己独立的地址空间，因此当进程切换时，TLB 中的条目往往对应的是**旧进程**的虚拟地址映射。这样如果不处理 TLB，可能会发生**错误的地址转换**。

4. **在上下文切换时如何处理 TLB？**

- **无效化 TLB**：在上下文切换时，系统会将**TLB 中的所有条目**设为**无效**，因为这些条目属于上一个进程，不再有效。

- 当新进程运行时，TLB 将逐渐重新填充，随着新进程的内存访问，新的虚拟地址-物理地址映射会被加载到 TLB 中。

# 评估虚拟内存的性能

实际上可以沿用Cache的那一套

## 缓存和虚拟内存的对比

![CS61C｜Lec17-Virtual Memory-20250122-20.png](../../../assets/images/CS61C%EF%BD%9CLec17-Virtual%20Memory-20250122-20.png)

1. **虚拟内存的层次结构**

- **虚拟内存**是计算机内存层次结构中的一个重要组成部分。它位于**主存（DRAM）和磁盘**之间。虚拟内存允许系统通过在磁盘上存储数据，模拟出比实际物理内存大得多的地址空间。

- 在传统的层次结构中，我们认为**主存（RAM）是内存层次结构的最底层**。但引入虚拟内存后，**磁盘（硬盘/SSD）** 成为了更底层的存储设备，**主存就像是中间层的缓存**，用来加速磁盘和 CPU 之间的数据访问。

2. **TLB 与缓存的关系**

- **TLB（Translation Lookaside Buffer）位于缓存之前，负责加速虚拟地址到物理地址的转换**。每当程序访问内存时，CPU 会先通过 TLB 快速查找虚拟地址对应的物理地址，如果找到匹配的地址转换（TLB hit），就可以直接访问主存或缓存。

- **TLB 缺失（TLB miss）时，系统必须查找页表，甚至可能引发页面置换（page fault）**，即需要从磁盘中将数据加载到内存。这种情况不仅涉及到 TLB 和缓存，还涉及到虚拟内存和磁盘的交互。

相同的CPI，AMAT方程适用，但现在将主存视为中间缓存。

AMAT = TLB access time + (1 - TLB hit rate) × (memory access time + (1 - page hit rate) × disk access time)

## 常规性能指标

![CS61C｜Lec17-Virtual Memory-20250122-21.png](../../../assets/images/CS61C%EF%BD%9CLec17-Virtual%20Memory-20250122-21.png)

## 平均内存访问时间的例题

条件:

-  L1 cache hit = 1 clock cycles, hit 95% of accesses

-  L2 cache hit = 10 clock cycles, hit 60% of L1 misses

-  DRAM = 200 clock cycles (≈100 nanoseconds)

-  Disk = 20,000,000 clock cycles (≈10 milliseconds)

**没有分页的 AMAT 计算**

AMAT = L1命中时间 + L1未命中的惩罚 = 1 + 5% × 10 + 5% × 40% × 200 = 5.5 个时钟周期

**引入分页**

AMAT = 5.5（不考虑分页时的 AMAT）+ 5% × 40% × (1 - HRMem) × 20,000,000

所以我们希望HRMem尽可能接近1

**不同 HRMem（主存命中率）下的 AMAT 计算**

- **HRMem = 99%**

频繁swap pages（比如物理内存不足的时候）的时候会出现这种情况

称之为抖动（Thrashing）

AMAT = 5.5 + 0.02 × 0.01 × 20,000,000 = 5.5 + 4000 = 4005.5 个时钟周期

- **HRMem = 99.9%**

AMAT = 5.5 + 0.02 × 0.001 × 20,000,000 = 5.5 + 400 = 405.5 个时钟周期

- **HRMem = 99.9999%**

AMAT = 5.5 + 0.02 × 0.000001 × 20,000,000 = 5.5 + 4 = 5.9 个时钟周期