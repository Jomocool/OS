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
### 6.2 Trap代码执行流程

1. 从Shell的角度看，Shell调用write系统调用实际上就是个Shell代码的C函数调用，然后write通过执行ECALL指令来执行系统调用。ECALL指令会切换到具有supervisor mode的内核中。在此过程中，内核中执行的第一个指令是一个由汇编语言写的函数，叫uservec。之后，在这个汇编函数中，代码执行跳转到了由C语言实现的函数usertrap中。接着在usertrap这个C函数中，执行了syscall函数，此函数在一个表单中，根据传入的代表系统调用的数字进行查找，并在内核中执行具体实现了系统调用功能的函数。在这里就是sys_write，它会将要显示的数据输出到console上，完成后返回给syscall。
2. 因为相当于在ECALL之后中断了用户代码的执行，为了恢复用户代码的执行，在syscall函数中，会调用一个函数叫做usertrapret，此函数完成了部分方便在C代码中实现返回用户空间的工作。此外，还有一些工作只能在汇编语言中完成，存在于trampoline.s文件中的userret函数中。最终，在这个汇编函数中会调用机器指令返回到用户空间，并且恢复ECALL之后的用户程序的执行。
3. vm.c中的所有函数都是内核中的一部分，所以运行在supervisor mode下
4. vm.c里的函数直接访问物理内存。能这么做的原因是，内核在page table中设置好了各个PTE。当内核接收到一个读写虚拟内存地址的请求时，会通过kernel page table将这个虚拟内存地址翻译成与之等价的物理内存地址，再完成读写。但是知道trap机制切换到内核之前，这些映射关系都不可用，因为在此之前，我们使用的仍然是user page table
5. 实际上很多操作系统都提供这种叫做内存映射文件（Memory-mapped file access）的机制，在这个机制里面通过page table，可以将用户空间的虚拟地址空间，对应到文件内容，这样你就可以通过内存地址直接读写文件。实际上很多操作系统都提供这种叫做内存映射文件（Memory-mapped file access）的机制，在这个机制里面通过page table，**可以将用户空间的虚拟地址空间，对应到文件内容**，这样你就可以通过内存地址直接读写文件。对于许多程序来说，这个机制的确会比直接调用read/write系统调用要快的多。对于许多程序来说，这个机制的确会比直接调用read/write系统调用要快的多。

### 6.3 ECALL指令之前的状态

1. 运行sh.c

   ![](https://github.com/Jomocool/Operator-System/blob/main/MIT6.S081-img/9.png)

   选中的行是一个write系统调用，将"$"写入到文件描述符2。

2. Shell调用write时，实际上调用的是关联到Shell的一个库函数

   ![](https://github.com/Jomocool/Operator-System/blob/main/MIT6.S081-img/10.png)

   上面几行代码是实际调用的write函数的实现，首先将SYS_write加载到a7寄存器，SYS_write是常量16，也就是告诉内核我要运行第16个系统调用，这个系统调用正好是write，之后调用ecall指令，从这里开始代码执行跳转到了内核，内核完成工作后代码执行会返回用户空间，继续执行ecall之后的指令，也就是ret，最终返回到Shell用户代码中。

3. 用户空间所有地址都比较小，一旦进入内核后，内核会使用大得多的内存地址。

4. 系统调用的时间点会有大量状态的变更，其中一个最重要的变更状态，且在它变更之前我们仍对它有依赖的，就是当前的page table。

5. attr列式PTE的标志位，rwx表示这个page可读、可写、可执行。u表示PET_U标志位是否被设置，用户代码只能访问U标志位被设置了的PTE。再下一位标志位是Global。再下一个标志位是a(accessed)，表明这条PTE是否被使用过。d(dirty)表明这条PTE是否被写过。

   ![](https://github.com/Jomocool/Operator-System/blob/main/MIT6.S081-img/11.png)

### 6.4 ECALL指令之后的状态

1. 查看程序计数器，已经在supervisor mode

2. page table还没有改变

3. 寄存器的值也还没有改变，仍是用户程序的数据，因此要十分小心，在寄存器的数据保存到某处之前，不能使用任何寄存器，否则无法恢复寄存器数据，后续返回用户程序执行时就会出错

4. 即使trampoline page是在用户地址空间的user page table完成的映射，用户代码不能写它，因为这些page对应的PTE并没有设置PTE_u标志位。这也是为什么trap机制是安全的。即使trampoline page是在用户地址空间的user page table完成的映射，用户代码不能写它，因为这些page对应的PTE并没有设置PTE_u标志位。这也是为什么trap机制是安全的。

5. ecall只会改变三件事：

   第一，ecall将代码从user mode改到supervisor mode。

   第二，ecall将程序计数器的值保存在了SEPC寄存器。

   第三，ecall会跳转到STVEC寄存器指向的指令。第三，ecall会跳转到STVEC寄存器指向的指令。

6. 接下来：

   第一，保存32个用户寄存器的内容

   第二，切换到kernel page table

   第三，创建或找到kernel stack，并将Stack Pointer寄存器的内容指向那个kernel stack，这样才能给C代码提供栈。

   第四，还需要跳转到内核中C代码的某些合理的位置

   ecall不会为我们做上面的任何一件事。但是可以通过修改硬件让ecall完成这些工作，而不是交给软件来做。实际上，有的机器在执行系统调用时，会在硬件中完成所有这些工作。但是RISC-V并不会，RISC-V秉持了这样一个观点：ecall只完成尽量少必须要完成的工作，其他的工作都交给软件完成。这里的原因是，RISC-V设计者想要为软件和操作系统的程序员提供最大的灵活性，这样他们就能按照他们想要的方式开发操作系统。所以你可以这样想，尽管XV6并没有使用这里提供的灵活性，但是一些其他的操作系统用到了。实际上，有的机器在执行系统调用时，会在硬件中完成所有这些工作。但是RISC-V并不会，RISC-V秉持了这样一个观点：ecall只完成尽量少必须要完成的工作，其他的工作都交给软件完成。这里的原因是，RISC-V设计者想要为软件和操作系统的程序员提供最大的灵活性，这样他们就能按照他们想要的方式开发操作系统。所以你可以这样想，尽管XV6并没有使用这里提供的灵活性，但是一些其他的操作系统用到了。

   一些原因：

   - 在这里，对于某些不需要切换页表的系统调用就不会每次都切换页表了，提高了效率，因为切换页表的代价比较高。
   - 某些操作系统同时将user和kernel的虚拟地址映射到一个page table中，这样在user和kernel之间切换时根本就不用切换page table。对于这样的操作系统来说，如果ecall切换了page table那将会是一种浪费，并且也减慢了程序的运行。某些操作系统同时将user和kernel的虚拟地址映射到一个page table中，这样在user和kernel之间切换时根本就不用切换page table。对于这样的操作系统来说，如果ecall切换了page table那将会是一种浪费，并且也减慢了程序的运行。
   - 或许在一些系统调用过程中，一些寄存器不用保存，而哪些寄存器需要保存，哪些不需要，取决于于软件，编程语言，和编译器。通过不保存所有的32个寄存器或许可以节省大量的程序运行时间，所以你不会想要ecall迫使你保存所有的寄存器。或许在一些系统调用过程中，一些寄存器不用保存，而哪些寄存器需要保存，哪些不需要，取决于于软件，编程语言，和编译器。通过不保存所有的32个寄存器或许可以节省大量的程序运行时间，所以你不会想要ecall迫使你保存所有的寄存器。
   - 最后，对于某些简单的系统调用或许根本就不需要任何stack，所以对于一些非常关注性能的操作系统，ecall不会自动为你完成stack切换是极好的。最后，对于某些简单的系统调用或许根本就不需要任何stack，所以对于一些非常关注性能的操作系统，ecall不会自动为你完成stack切换是极好的。

   所以，ecall尽量的简单可以提升软件设计的灵活性。所以，ecall尽量的简单可以提升软件设计的灵活性。

7. ecall是CPU指令，自然无法在gdb看到具体内容。ecall只会更新CPU中的mode标志位为supervisor，并且设置程序计数器成STVEC寄存器内的值。在进入到用户空间之前，内核会将trampoline page的地址存在STVEC寄存器中。所以ecall的下一条指令的位置是STVEC指向的地址，也就是trampoline page的起始地址。ecall只会更新CPU中的mode标志位为supervisor，并且设置程序计数器成STVEC寄存器内的值。在进入到用户空间之前，内核会将trampoline page的地址存在STVEC寄存器中。所以ecall的下一条指令的位置是STVEC指向的地址，也就是trampoline page的起始地址。

### 6.5 uservec函数

1. 在一些其他的机器中，我们或许直接就将32个寄存器中的内容写到物理内存中某些合适的位置。但是我们不能在RISC-V中这样做，因为在RISC-V中，supervisor mode下的代码不允许直接访问物理内存。所以我们只能使用page table中的内容，但是从前面的输出来看，page table中也没有多少内容。在一些其他的机器中，我们或许直接就将32个寄存器中的内容写到物理内存中某些合适的位置。但是我们不能在RISC-V中这样做，因为在RISC-V中，supervisor mode下的代码不允许直接访问物理内存。所以我们只能使用page table中的内容，但是从前面的输出来看，page table中也没有多少内容。

2. 虽然XV6并没有使用，但是另一种可能的操作是，直接将SATP寄存器指向kernel page table，之后我们就可以直接使用所有的kernel mapping来帮助我们存储用户寄存器。这是合法的，因为supervisor mode可以更改SATP寄存器。但是在trap代码当前的位置，也就是trap机制的最开始，我们并不知道kernel page table的地址。并且更改SATP寄存器的指令，要求写入SATP寄存器的内容来自于另一个寄存器。所以，为了能执行更新page table的指令，我们需要一些空闲的寄存器，这样我们才能先将page table的地址存在这些寄存器中，然后再执行修改SATP寄存器的指令。虽然XV6并没有使用，但是另一种可能的操作是，直接将SATP寄存器指向kernel page table，之后我们就可以直接使用所有的kernel mapping来帮助我们存储用户寄存器。这是合法的，因为supervisor mode可以更改SATP寄存器。但是在trap代码当前的位置，也就是trap机制的最开始，我们并不知道kernel page table的地址。并且更改SATP寄存器的指令，要求写入SATP寄存器的内容来自于另一个寄存器。所以，为了能执行更新page table的指令，我们需要一些空闲的寄存器，这样我们才能先将page table的地址存在这些寄存器中，然后再执行修改SATP寄存器的指令。

3. 对于保存用户寄存器，XV6在RISC-V上的实现包括了两个部分。第一个部分是，XV6在每个user page table映射了trapframe page，这样每个进程都有自己的trapframe page。这个page包含了很多有趣的数据，但是现在最重要的数据是用来保存用户寄存器的32个空槽位。所以，在trap处理代码中，现在的好消息是，我们在user page table有一个之前由kernel设置好的映射关系，这个映射关系指向了一个可以用来存放这个进程的用户寄存器的内存位置。这个位置的虚拟地址总是0x3ffffffe000。对于保存用户寄存器，XV6在RISC-V上的实现包括了两个部分。第一个部分是，XV6在每个user page table映射了trapframe page，这样每个进程都有自己的trapframe page。这个page包含了很多有趣的数据，但是现在最重要的数据是用来保存用户寄存器的32个空槽位。所以，在trap处理代码中，现在的好消息是，我们在user page table有一个之前由kernel设置好的映射关系，这个映射关系指向了一个可以用来存放这个进程的用户寄存器的内存位置。这个位置的虚拟地址总是0x3ffffffe000。

4. 所以，如何保存用户寄存器的一半答案是，内核非常方便的将trapframe page映射到了每个user page table。

   另一半的答案在于我们之前提过的SSCRATCH寄存器。这个由RISC-V提供的SSCRATCH寄存器，就是为接下来的目的而创建的。在进入到user space之前，内核会将trapframe page的地址保存在这个寄存器中，也就是0x3fffffe000这个地址。更重要的是，RISC-V有一个指令允许交换任意两个寄存器的值。而SSCRATCH寄存器的作用就是保存另一个寄存器的值，并将自己的值加载给另一个寄存器。

5. 一台机器总是从内核开始运行的，当机器启动的时候，它就是在内核中。 任何时候，不管是进程第一次启动还是从一个系统调用返回，进入到用户空间的唯一方法是就是执行sret指令。sret指令是由RISC-V定义的用来从supervisor mode转换到user mode。所以，在任何用户代码执行之前，内核会执行fn函数，并设置好所有的东西，例如SSCRATCH，STVEC寄存器。一台机器总是从内核开始运行的，当机器启动的时候，它就是在内核中。 任何时候，不管是进程第一次启动还是从一个系统调用返回，进入到用户空间的唯一方法是就是执行sret指令。sret指令是由RISC-V定义的用来从supervisor mode转换到user mode。所以，在任何用户代码执行之前，内核会执行fn函数，并设置好所有的东西，例如SSCRATCH，STVEC寄存器。

6. trampoline page在user page table中的映射与kernel page table中的映射是完全一样的。这两个page table中其他所有的映射都是不同的，只有trampoline page的映射是一样的，因此我们在切换page table时，寻址的结果不会改变，我们实际上就可以继续在同一个代码序列中执行程序而不崩溃。这是trampoline page的特殊之处，它同时在user page table和kernel page table都有相同的映射关系。trampoline page在user page table中的映射与kernel page table中的映射是完全一样的。这两个page table中其他所有的映射都是不同的，只有trampoline page的映射是一样的，因此我们在切换page table时，寻址的结果不会改变，我们实际上就可以继续在同一个代码序列中执行程序而不崩溃。这是trampoline page的特殊之处，它同时在user page table和kernel page table都有相同的映射关系。之所以叫trampoline page，是因为你某种程度在它上面“弹跳”了一下，然后从用户空间走到了内核空间。之所以叫trampoline page，是因为你某种程度在它上面“弹跳”了一下，然后从用户空间走到了内核空间。

### 6.6 usertrap函数

1. 当程序还在内核中执行时，我们可能切换到另一个进程，并进入到那个程序的用户空间，然后那个进程可能再调用一个系统调用进而导致SEPC寄存器的内容被覆盖。所以，我们需要保存当前进程的SEPC寄存器到一个与该进程关联的内存中，这样这个数据才不会被覆盖。当程序还在内核中执行时，我们可能切换到另一个进程，并进入到那个程序的用户空间，然后那个进程可能再调用一个系统调用进而导致SEPC寄存器的内容被覆盖。所以，我们需要保存当前进程的SEPC寄存器到一个与该进程关联的内存中，这样这个数据才不会被覆盖。

### 6.7 usertrapret函数

1. usertrap函数的最后调用了usertrapret函数。它首先关闭了中断。我们之前在系统调用的过程中是打开了中断的，这里关闭中断是因为我们将要更新STVEC寄存器来指向用户空间的trap处理代码，而之前在内核中的时候，我们指向的是内核空间的trap处理代码它首先关闭了中断。我们之前在系统调用的过程中是打开了中断的，这里关闭中断是因为我们将要更新STVEC寄存器来指向用户空间的trap处理代码，而之前在内核中的时候，我们指向的是内核空间的trap处理代码。我们关闭中断因为当我们将STVEC更新到指向用户空间的trap处理代码时，我们仍然在内核中执行代码。如果这时发生了一个中断，那么程序执行会走向用户空间的trap处理代码，即便我们现在仍然在内核中，出于各种各样具体细节的原因，这会导致内核出错。所以我们这里关闭中断。我们关闭中断因为当我们将STVEC更新到指向用户空间的trap处理代码时，我们仍然在内核中执行代码。如果这时发生了一个中断，那么程序执行会走向用户空间的trap处理代码，即便我们现在仍然在内核中，出于各种各样具体细节的原因，这会导致内核出错。所以我们这里关闭中断。

2. 现在我们在usertrapret函数中现在我们在usertrapret函数中，填入trapframe的内容：

   - 存储了kernel page table的指针

   - 存储了当前用户进程的kernel stack

   - 存储了usertrap函数的指针，这样trampoline代码才能跳转到这个函数
   - 从tp寄存器中读取当前的CPU核编号，并存储在trapframe中，这样trampoline代码才能恢复这个数字，因为用户代码可能会修改这个数字从tp寄存器中读取当前的CPU核编号，并存储在trapframe中，这样trampoline代码才能恢复这个数字，因为用户代码可能会修改这个数字

3. 接下来我们要设置SSTATUS寄存器，这是一个控制寄存器。这个寄存器的SPP bit位控制了sret指令的行为，该bit为0表示下次执行sret的时候，我们想要返回user mode而不是supervisor mode。这个寄存器的SPIE bit位控制了，在执行完sret之后，是否打开中断。因为我们在返回到用户空间之后，我们的确希望打开中断，所以这里将SPIE bit位设置为1。修改完这些bit位之后，我们会把新的值写回到SSTATUS寄存器。接下来我们要设置SSTATUS寄存器，这是一个控制寄存器。这个寄存器的SPP bit位控制了sret指令的行为，该bit为0表示下次执行sret的时候，我们想要返回user mode而不是supervisor mode。这个寄存器的SPIE bit位控制了，在执行完sret之后，是否打开中断。因为我们在返回到用户空间之后，我们的确希望打开中断，所以这里将SPIE bit位设置为1。修改完这些bit位之后，我们会把新的值写回到SSTATUS寄存器。

4. 接下来，我们根据user page table地址生成相应的SATP值，这样我们在返回到用户空间的时候才能完成page table的切换。实际上，我们会在汇编代码trampoline中完成page table的切换，并且也只能在trampoline中完成切换，因为只有trampoline中代码是同时在用户和内核空间中映射。但是我们现在还没有在trampoline代码中，我们现在还在一个普通的C函数中，所以这里我们将page table指针准备好，并将这个指针作为第二个参数传递给汇编代码，这个参数会出现在a1寄存器。接下来，我们根据user page table地址生成相应的SATP值，这样我们在返回到用户空间的时候才能完成page table的切换。实际上，我们会在汇编代码trampoline中完成page table的切换，并且也只能在trampoline中完成切换，因为只有trampoline中代码是同时在用户和内核空间中映射。但是我们现在还没有在trampoline代码中，我们现在还在一个普通的C函数中，所以这里我们将page table指针准备好，并将这个指针作为第二个参数传递给汇编代码，这个参数会出现在a1寄存器。

### 6.8 userret函数

1. 第一步是切换page table。在执行*csrw satp, a1*之前，page table应该还是巨大的kernel page table。这条指令会将user page table（在usertrapret中作为第二个参数传递给了这里的userret函数，所以存在a1寄存器中）存储在SATP寄存器中。执行完这条指令之后，page table就变成了小得多的user page table。但是幸运的是，user page table也映射了trampoline page，所以程序还能继续执行而不是崩溃。
2. sret是我们在kernel中的最后一条指令，当我执行完这条指令：sret是我们在kernel中的最后一条指令，当我执行完这条指令：
   - 程序会切换会user mode
   - SEPC寄存器的数值会被拷贝到PC寄存器(程序计数器)
   - 重新打开中断
3. 系统调用被刻意设计的看起来像是函数调用，但是背后的user/kernel转换比函数调用要复杂的多。之所以这么复杂，很大一部分原因是要保持user/kernel之间的隔离性，内核不能信任来自用户空间的任何内容。系统调用被刻意设计的看起来像是函数调用，但是背后的user/kernel转换比函数调用要复杂的多。之所以这么复杂，很大一部分原因是要保持user/kernel之间的隔离性，内核不能信任来自用户空间的任何内容。
