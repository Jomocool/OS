# MIT6.S081Lab

## 环境搭建

网上有一堆环境搭建教程，但是能成功的我觉得不多，终于还是在这个大佬的教程下我才搭建成功的

大佬讲得十分细心，虽然我第一遍卡住了，但是删掉ubuntu，重新安装ubuntu后，就没问题了，基本上每一步都和大佬一模一样

**参考：**

[帖子详情 - Diz (ourdiz.com)](https://ourdiz.com/post/2/6/39)



[首页 - diz (ourdiz.com)](https://ourdiz.com/2/6)

## Lab1 Unix utilities

This lab will familiarize you with xv6 and its system calls.

### Boot xv6(easy)

在前面环境搭配好的情况下，克隆仓库，编译编译器，qemu

```shell
$ git clone git://g.csail.mit.edu/xv6-labs-2020
$ cd xv6-labs-2020
$ git checkout util
$ make qemu
```

### sleep(easy)

Implement the UNIX program sleep for xv6; your sleep should pause for a user-specified number of ticks. A tick is a notion of time defined by the xv6 kernel, namely the time between two interrupts from the timer chip. Your solution should be in the file user/sleep.c.

- 首先通读 xv6 book 的chapter 1
- 查看其他的用户程序 例如user/echo.c user/grep.c 学习如何获取来自命令行的参数
- 如果用户忘记传入参数 ,sleep 函数应该打印错误信息
- 命令行会以string的类型进行传递, 你可以用atoi 函数将它转换成int类型
- 使用sleep系统调用
- 在kernel/sysproc.c 实现sleep 系统调用(寻找sys_sleep), user/user.h 有关于用户层是如何调用sleep, user/usys.S汇编代码 让用户指令跳到内核中
- 确保main函数调用了 exit() 退出你的程序
- 添加你的sleep用户程序 到Makefile的 UPROGS中; 一旦你添加完成, make qemu会自动编译你的程序, 接下来你就可以在xv6 的shell 中去执行它
- 通读K&R的“The C Programming Language” 学习更多关于C的知识

1. **在user目录下，新建sleep.c文件，代码如下**

   ```c
   #include "kernel/types.h"
   #include "user/user.h"
   
   int main(int argc,char* argv[]){
       //sleep用法：sleep (int)
       if(argc!=2){
           printf("Error Example: sleep 2");
           exit(-1);
       }
   
       //argv={"program","argv"}
       //argv=["sleep","3"];
       int num_of_ticks=atoi(argv[1]);
       if(sleep(num_of_ticks)<0){//系统调用，如果调用不成功返回值小于0
           printf("Can not sleep!");
           exit(-1);
       }
       
       exit(0);
   }
   ```

2. **记得在Makefile中加入sleep**

   ```shell
   UPROGS=\
   	$U/_cat\
   	$U/_echo\
   	$U/_forktest\
   	$U/_grep\
   	$U/_init\
   	$U/_kill\
   	$U/_ln\
   	$U/_ls\
   	$U/_mkdir\
   	$U/_rm\
   	$U/_sh\
   	$U/_stressfs\
   	$U/_usertests\
   	$U/_grind\
   	$U/_wc\
   	$U/_zombie\
   	$U/_sleep\    #here!
   ```

3. **在xv6-labs-2020目录下，确定git在util分支下，然后进入qemu环境下，进行sleep测试**

   ```shell
   git checkout util
   sudo make qemu
   
   #hart 2 starting
   #hart 1 starting
   #init: starting sh
   sleep 10
   #如果停顿一会后才出现下一行的符号'$‘，说明成功了，可以进行下一个实验
   ```



### pingpong(easy)

Write a program that uses UNIX system calls to “ping-pong” a byte between two processes over a pair of pipes, one for each direction. The parent should send a byte to the child; the child should print “: received ping", where is its process ID, write the byte on the pipe to the parent, and exit; the parent should read the byte from the child, print ": received pong", and exit. Your solution should be in the file user/pingpong.c.

- 使用pipe函数 创建一个pipe
- 使用fork函数 创建一个子进程
- 使用read函数 读取来自pipe的内容, 然后通过write函数往pipe里写入
- 使用getpid函数 找到当前进程的Process ID
- 添加pingpong用户程序 到Makefile的 UPROGS中
- 你可以在user/user.h 中找到用户程序中所有可用的库函数, 他们的实现代码在user/lib.c,user/printf.c, user/umalloc.c中
