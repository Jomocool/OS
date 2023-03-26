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
