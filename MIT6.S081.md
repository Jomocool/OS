# MIT6.S081

## Lecture 1 - Introduction and Examples

**fork()：**

fork（）函数通过系统调用创建一个与原来进程几乎完全相同的进程，也就是两个进程可以做完全相同的事，但如果初始参数或者传入的变量不同，两个进程也可以做不同的事。

一个进程调用fork（）函数后，系统先给新的进程分配资源，例如存储数据和代码的空间。然后把原来的进程的所有值都复制到新的新进程中，只有少数值与原来的进程的值不同。相当于克隆了一个自己。

1. 在父进程中，fork返回新创建子进程的进程ID
2. 在子进程中，fork返回0
3. 如果出现错误，fork返回一个负值



![](https://github.com/Jomocool/Operator-System/blob/main/MIT6.S081-img/1.png)

**执行过程：**

1. 从第12行开始，就有两个一模一样的进程了，其中一个为父进程，另一个为子进程
2. 父进程pid>0，子进程pid==0
3. 父进程执行完19行之后开始wait，子进程进行系统调用(15行)，调用成功，直接退出了，把返回值0写道&status上，之后父进程继续执行第21行

输出：
parent waiting

THIS IS ECHO

the child exited with status 0



![](https://github.com/Jomocool/Operator-System/blob/main/MIT6.S081-img/2.png)

xklsdksdjkecho不存在

输出：

parent waiting

exec failed

the child exited with status 1



以上的输出并不能保证每次都相同，因为每条命令的执行顺序并不能保证每次都是一样的。因为在执行exec时，操作系统需要先到磁盘中获取exec，接着释放旧进程的内存，时间开销相对大



- fork会复制整个父进程的内存，可能会造成内存浪费，但exec会扔掉所有复制的内存，并将其替换为你运行的文件的内容，因为进入到exec后，子进程将不会用到复制的内存了

- 如果一个父进程有多个子进程，就需要多个wait函数，每退出一个子进程，就返回一个wait，wait返回后，会显示子进程ID

## Lecture3 - OS Organization and System calls

**Isolation：**

**操作系统的意义：**实现multiplexing和物理内存隔离

应用程序不能直接与CPU交互，但是可以和进程交互。CPU向应用程序提供“进程”，进程抽象了CPU，这样操作系统才能在多个应用程序之间复用一个或者多个CPU

调用exec函数时，传入一个文件名，该文件名对应了一个应用程序的内存镜像(包括程序对应的指令，全局的数据)，但是应用程序没有直接访问物理内存的权限，因为操作系统会提供内存隔离并控制内存，操作系统会在应用程序和硬件资源之间提供一个中间层

files抽象了磁盘，操作系统确保一个磁盘块只出现在一个文件中，并且确保用户A不能操作用户B的文件，实现了不同用户或同一用户的不同进程的文件强隔离

**Defensive：**

如果说应用程序无意或者恶意向系统调用传入一些错误的参数，会导致操作系统崩溃，操作系统就会拒绝为所有应用程序服务

攻击者打破对应用程序的隔离，进而控制内核，也就控制了计算机所有资源

如果需要操作系统具有强防御性，那就需要让操作系统和应用程序之间具有强隔离性

如果处理器要能够处理多进程，就要支持user/kernel mode和virtual memory

**user/kernel mode：**

- kernel mode：cpu可以运行特定权限的指令
- user mode：cpu只能运行普通权限的指令

当一个应用程序尝试执行一条特殊权限指令，处理器会拒绝执行这条指令，因为不允许在user mode里执行特殊权限指令，因为这时会将控制权限从user mode转到kernel mode，当操作系统拿到控制权后，或许会杀掉进程，因为应用程序执行了不该执行的指令

BIOS先于操作系统启动

**virtual memory：**

每个进程都有有自己的page table(页表)，这样每个进程只会访问出现在自己page table中的物理内存，操作系统会设置页表，使每个进程都有不重合的物理内存。

页表定义了对内存的视图，使每个用户进程都有自己对内存的独立视图，这提供了强内存隔离性

![](https://github.com/Jomocool/Operator-System/blob/main/MIT6.S081-img/3.png)

**Entring Kernel：**

excall(n)：当一个用户程序想要程序执行的控制权转移到内核，需要执行ecall指令，并传入一个数字，数字代表应用程序想要调用的System Call

当调用fork函数时，并不是直接调用操作系统，而是调用ecall函数，并将fork对应的数字作为参数传给ecall，而后用户态转为内核态，在内核态中调用syscall函数，syscall函数会检查ecall函数的参数，通过这个参数知道需要调用的是fork。并且内核态的fork函数会进行一切想要的安全检查。**用户空间和内核空间的界限是一个硬性的界限，所以用户程序不能直接调用内核态的fork，用户程序执行系统调用的唯一方法，就是通过ecall指令**

write函数实际上也是调用封装好的系统中的ecall指令

如果一个用户应用程序恶意执行死循环，操作系统会通过提前设定好的计时器来夺回执行权

**为什么用c语言？**c提供了很多对于硬件的控制能力，与硬件的交互能力强

kernel=trusted computing base(TCB)



**宏内核：**在kernel中运行尽可能多的代码

**微内核：**在kernel中运行尽可能少的代码

微内核在用户态和内核态之间的跳转次数更多，性能损耗更大



**Kernel下的makefile：**

proc.c (gcc)-> proc.s(RISC-V汇编语言文件) (assembler)-> proc.o(汇编语言的二进制格式)  

pipec -> pipe.s -> pipe.o

之后，系统加载器(Loader)会收集所有.o文件，并生成kernel(内核)文件

makefile还会生成kernel.asm文件，包含了内核的完整汇编语言，方便阅读

想象内核是运行在一个主板上的

## Lecture4 - Page Tables

**Address Spaces：**包括内核在内的所有程序专属的地址空间

所有程序都必须存在于物理内存之中，否则处理器不能处理程序的指令。每个程序运行在自己的地址空间，且这些地址空间相互独立，所以也就不具备引用其他程序所用内存地址的能力。

![](https://github.com/Jomocool/Operator-System/blob/main/MIT6.S081-img/4.png)



**问题：如何在一个物理内存上，创建不同的地址空间？**

需要硬件支持，所以由处理器在硬件上实现，或者通过内存管理单元(MMU)实现

CPU在处理任何带地址的指令时，其中的地址都应该是虚拟地址，而不是物理地址

从CPU的角度看，一旦MMU打开了，它执行的每条指令中的地址都是虚拟地址



**CPU处理虚拟地址流程：**

1. CPU处理指令，发现有虚拟地址，将该虚拟地址转到内存管理单元(内存地址映射关系也存在内存中，CPU中需要寄存器存放表单在物理地址中的地址)
2. 内存管理单元会将虚拟地址翻译成物理地址(MMU查看存放在物理内存的表单，一边是虚拟内存地址，一边是物理内存地址，它们之间的映射十分灵活)
3. 该物理地址会被用来索引物理内存，并从该物理内存加载或存储数据

基本思想是让每个物理内存都有自己的表单，所以每个程序都有自己的表单

当CPU从一个应用程序切换到另一个应用程序时，同时也需要切换寄存器中的内容，从而指向新的进程保存在物理内存中的地址对应表单(page table)，内核会读写寄存器，这是一条特殊权限指令，所以用户应用程序不能通过更新这个寄存器来更换一个地址对应表单，防止破坏隔离性

页表是分页机制的关键部分，负责记录虚拟页到物理页的映射关系;操作系统负责对页表进行配置。请思考这个问题: 如果操作系统按照线性数组来记录映射关系，那么对于64位的虚拟地址空间，这个页表有多大?假设页的大小为4KB，页表中的每一项为8字节(主要用于存储物理地址)，那么一张页表的大小就是2^64/4KB*8字节，即33 554 432GB!一个虚拟地址可以映射到一页物理地址中的其中一个，所以要计算需要多少个虚拟地址，就记录有几个物理页(2^64/4KB)，这也代表了有几个页表项，总容量=物理页数\*4KB

在分页机制中，一个虚拟地址并不一定会映射到一个物理地址上。实际上，一个虚拟地址会被分成两个部分：**页号和页内偏移量**。页号用于查找页表中对应的物理页框号，而页内偏移量则用于定位物理页中的具体地址。因此，一个虚拟地址和物理地址之间的映射关系可以被分成两个部分：虚拟地址到物理页框号的映射关系，以及页内偏移量在虚拟地址和物理地址中的保持一致。因此，一个虚拟地址可以映射到一个物理页框号上，但不一定映射到一个物理地址上，因为它可能只是在物理页中的某个位置。

**创建表单：**

1. 不要为每个地址创建一条表单项目，而是为每个page创建一条表单条目，所以每次翻译针对的是一个page
2. 一个page有4096byte，所以offset需要12bit
3. 如果虚拟地址过少，可能会出现虚拟地址用完了，但是物理地址还有剩余的现象

**RISC-V Page Table**

![](https://github.com/Jomocool/Operator-System/blob/main/MIT6.S081-img/5.png)

最高一级的page directory的PPN存放的是物理地址，因为需要在物理内存中查找下一个page directory的地址。不能让地址翻译依赖于另一个翻译，否则可能会陷入递归的无限循环中。SATP寄存器存放的是物理地址，因为最高级的page directory存在物理内存中



**Translation Look-aside Buffer(TLB)：**页表缓存。Page Table Entry的缓存，即PTE缓存。在CPU内核中

当处理器第一次查找一个虚拟地址时，硬件通过3级page table得到最终的PPN，TLB会保存虚拟地址到物理地址的映射关系，然后TLB会存储VA PA PN PA映射，下一次访问同一个虚拟地址时，处理器可以查看TLB，TLB会直接返回物理地址，而不需要通过page table得到结果。如果切换了page table，TLB不再有效，因为它存放的是上一个page table的映射关系，所以需要清空，如果不清空，会出现翻译错误的情况。

3级page table由硬件实现，因为3级page table查找都发生在硬件中，MMU是硬件的一部分而不是操作系统的一部分

TLB有一个walk函数，设置了最初的page table，需要对3级page table进行编程，所以需要具备模拟3级page table的能力。内核有自己的page table，用户进程也有自己的page table，当用户进程指向sys_info结构体时，它的指针存在于用户空间的page table，但是内核需要将这个指针翻译成一个自己可以读写的物理地址，内核会通过用户进程的page table将虚拟地址翻译得到物理地址，这样内核可以读写相应的物理内存地址

page table提供了一层抽象，即虚拟地址到物理地址的映射，这些由操作系统控制，意味着操作系统对地址翻译有完全的控制权，可以实现各种各样的功能



**为什么用三级page directory也不用一个巨大的page directory：**因为当上级有的PTE为空时，不需要为其创建下级，节省空间

|       线性数组记录映射关系       |    3级树记录映射关系     |
| :------------------------------: | :----------------------: |
| 未被使用的也需要提前开辟内存空间 | 用多少就开辟多少内存空间 |

![](https://github.com/Jomocool/Operator-System/blob/main/MIT6.S081-img/6.png)

## Lecture5 - RISC-V Calling Convention and Stack Frames

**RISC-V处理器 => RISC-V指令集，每条指令都有一个相关联的二进制编码或操作码**

汇编语言有多种，RISC-V的汇编语言不能够在Linux上运行，大多数现代计算机都运行在x86和x86-64处理器上。其实就是指令集的不同

寄存器之所以重要是因为汇编代码并不是在内存上执行，而是在寄存器上执行。寄存器是用来进行任何运算和数据读取的最快的方式

如果需要存放数据的数量大于寄存器个数的时候，就需要使用内存了

**Caller saved：**Not preserved across function call

**Callee saved：**Preserved across function call

![](https://github.com/Jomocool/Operator-System/blob/main/MIT6.S081-img/7.png)

堆栈之所以重要，是因为它使得我们的函数变得有组织。且能够正常返回

每执行一次函数调用就会产生一个Stack Frame，每次调用一个函数，函数都会为自己创建一个Stack Frame，函数通过移动Stack Pointer来完成Stack Frame的空间分配。对于Stack来说，是从高地址开始向低地址使用，所以栈总是向下扩展。
函数栈帧包含寄存器、局部变量，如果参数寄存器用完了，额外的参数就会出现在栈上。所以Stack Frame大小并不总是一样。不同的函数有不同数量的本地变量，不同的寄存器

两件确定的事：
1.Return address总是会出现在Stack Frame的第一位

2.指向前一个Stack Frame的指针也会出现在栈中固定位置

Stack Frame有两个重要的寄存器

1.SP(Stack Pointer)：它指向了当前Stack Frame的底部并代表了当前Stack Frame的位置

2.FP(Frame Pointer)：它指向了当前Stack Frame的顶部



leaf function：指不调用别的函数的函数



![](https://github.com/Jomocool/Operator-System/blob/main/MIT6.S081-img/8.png)

1.开始减16，开辟16字节大小的栈，因为不想覆盖原来在Stack Pointer位置的数据

2.函数结束后，将sp指向原先的栈



Struct里的成员变量一般都是存储于连续地址的



## Lecture 6 - Isolation & System Call Entry_Exit

### 6.1Trap机制

1. 程序运行何时发生用户空间和内核空间的切换：

   - 程序执行系统调用
   - 程序出现了类似page fault、运算时除以0的错误
   - 一个设备出发了中断使得当前程序运行需要相应内核设备驱动

2. 用户空间和内核空间的切换通常被称为trap，trap的实现细节对于实现安全隔离和性能十分重要。同时因为很多应用程序，要么因为系统调用，要么因为page fault，都会频繁地切换到内核中。所以，trap机制要尽可能的简单。

3. 程序从只拥有user权限并且位于用户空间的Shell，切换到拥有supervisor权限的内核，这个过程中，硬件的状态十分重要，因为许多工作都是将硬件从适合运行用户应用程序的状态，改变到适合运行内核代码的状态

4. 32个寄存器：

   - stack pointer：堆栈寄存器（stack register）
   - 程序计数器（Program Counter Register）：表明了当前是supervisor mode还是user mode
   - SATP寄存器（Supervisor Address Translation and Protection）：控制CPU工作方式，包含了指向page table的物理内存地址
   - STVEC寄存器（Supervisor Trap Vector Base Address Register）：指向了内核中处理trap的指令的起始地址
   - SEPC寄存器：在trap的过程中保存程序计数器的值
   - SSRATCH寄存器（Supervisor Scratch Register）

   这些寄存器表明了执行系统调用时计算机的状态

5. 在trap的最开始，CPU所有状态都设置成运行用户代码而不是内核代码。在trap处理的过程中，实际需要改变一些状态，才可以运行系统内核中普通的C程序。过程如下：

   - 需要保存32个用户寄存器。因为后续需要恢复用户应用程序的执行，尤其是当用户程序被随机的中断所打断时。内核能够响应中断，同时用户程序能够在无感知的情况下恢复并继续运行。
   - 程序计数器需要保存，地位和用户寄存器一样。因为我们需要在用户程序运行中断的位置继续执行用户程序
   - 需要将mode转变为supervisor mode，因为需要使用内核中各种各样的特权指令
   - 需要将SATP指向kernel page table。因为SATP寄存器目前正指向user page table，其只包含了用户程序所需要的内存映射和一两个其他映射，并无包含完整的内核数据的内存映射。
   - 需要将堆栈寄存器指向位于内核的一个地址，因为需要一个堆栈来调用内核的C函数
   - 当设置好之后，所有的硬件状态都适合在内核中使用，就跳入内核的C代码

6. trap中涉及到的硬件和内核机制不能依赖任何来自用户空间的东西，否则有可能会破坏安全性。因此xv6的trap机制不会查看这些寄存器，而只是保存起来。同时，trap应对用户代码透明，这样也更容易写用户代码。即使这样，也要时刻小心用户代码可能会尝试欺骗它。

7. 在supervisor mode下：

   - SATP寄存器：page table指针
   - STVEC寄存器：处理trap的内核指令地址
   - SEPC寄存器：保存当发生trap时的程序计数器
   - SSCRATCH
   - 可以使用PTE_U标志位是0的PTE，而user mode只有在PTE_U是1的时候才可以使用PTE
   - 不可以读写任意物理地址，像普通的用户代码，也需要通过page table来访问内存。如果一个虚拟地址并不在当前由SATP指向的page table中，又或者SATP指向的page table中PTE_U=1，那么supervisor mode不能使用那个地址。如果一个虚拟地址并不在当前由SATP指向的page table中，又或者SATP指向的page table中PTE_U=1，那么supervisor mode不能使用那个地址。
