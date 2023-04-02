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

**pipe()：**

- 管道只是单向通信，即我们可以使用管道，使得一个进程写入管道，另一个进程从管道读取。它打开一个管道，这是被视为***“虚拟文件”\***的主内存区域。
- 创建进程及其所有子进程可以使用管道进行读取和写入。一个进程可以写入此“虚拟文件”或管道，另一个相关进程可以从中读取。
- 如果进程在将某些内容写入管道之前尝试读取，则该进程将挂起，直到写入某些内容。
- 管道系统调用查找进程的打开文件表中的前两个可用位置，并将它们分配给管道的读取端和写入端。

![](https://github.com/Jomocool/Operator-System/blob/main/MIT6.S081Lab-img/1.png)

```c
函数调用：
int pipe(int fds[2]);

Parameters :
fd[0] will be the fd(file descriptor) for the 
read end of pipe.
fd[1] will be the fd for the write end of pipe.
Returns : 0 on Success.
-1 on error.
```

管道的行为类似于**FIFO（**先进先出），管道的行为类似于**队列**数据结构。读取和写入的大小不必在此处匹配。我们一次可以写入 **512** 个字节，但我们一次只能在管道中读取 1 个字节。



**eg.**

```c
// C program to illustrate
// pipe system call in C
#include <stdio.h>
#include <unistd.h>
#define MSGSIZE 16
char* msg1 = "hello, world #1";
char* msg2 = "hello, world #2";
char* msg3 = "hello, world #3";

int main()
{
	char inbuf[MSGSIZE];
	int p[2], i;

	if (pipe(p) < 0)
		exit(1);

	/* continued */
	/* write pipe */

	write(p[1], msg1, MSGSIZE);
	write(p[1], msg2, MSGSIZE);
	write(p[1], msg3, MSGSIZE);

	for (i = 0; i < 3; i++) {
		/* read pipe */
		read(p[0], inbuf, MSGSIZE);
		printf("% s\n", inbuf);
	}
	return 0;
}
```

**输出：**

```c
hello, world #1
hello, world #2
hello, world #3
```



**Parent and child share a pipe**

当我们在任何进程中使用 [fork](https://www.geeksforgeeks.org/fork-system-call/) 时，文件描述符在子进程和父进程中保持打开状态。如果我们在创建管道后调用 fork，那么父子可以通过管道进行通信。

![](https://github.com/Jomocool/Operator-System/blob/main/MIT6.S081Lab-img/2.png)

**eg.**

```c
// C program to illustrate
// pipe system call in C
// shared by Parent and Child
#include <stdio.h>
#include <unistd.h>
#define MSGSIZE 16
char* msg1 = "hello, world #1";
char* msg2 = "hello, world #2";
char* msg3 = "hello, world #3";

int main()
{
	char inbuf[MSGSIZE];
	int p[2], pid, nbytes;

	if (pipe(p) < 0)
		exit(1);

	/* continued */
	if ((pid = fork()) > 0) {
		write(p[1], msg1, MSGSIZE);
		write(p[1], msg2, MSGSIZE);
		write(p[1], msg3, MSGSIZE);

		// Adding this line will
		// not hang the program
		// close(p[1]);
		wait(NULL);
	}

	else {
		// Adding this line will
		// not hang the program
		// close(p[1]);
		while ((nbytes = read(p[0], inbuf, MSGSIZE)) > 0)
			printf("% s\n", inbuf);
		if (nbytes != 0)
			exit(2);
		printf("Finished reading\n");
	}
	return 0;
}
```

**输出：**

```c
hello world, #1
hello world, #2
hello world, #3
//(hangs) program does not terminate but hangs -- 非输出内容
```



**pingpong.c**

```c
#include"kernel/types.h"
#include"user/user.h"

#define BUF_SIZE 16 //定义读入写出块大小

int main(int argc,char* argv[]){
	int fd[2];//声明数组用于write，read
	char buf[BUF_SIZE];//读入写出块
	//1.Use pipe to create a pipe.
	if(pipe(fd)<0){//异常
		printf("error\n");
		exit(1);
	}

	//* continue *
	//2.Use fork to create a child.
	if(fork()>0){
		//parent process
		write(fd[1],"ping",BUF_SIZE);//3.父进程向管道写入ping给子进程
		wait((int*)0);//等待子进程接收
		read(fd[0],buf,BUF_SIZE);//6.父进程接收管道中来自子进程的pong
		printf("<%d> received %s.\n",getpid(),buf);//父进程输出自己的进程号，并说明received pong
	}else{
		//child process
		read(fd[0],buf,BUF_SIZE);//4.子进程接收管道中来自父进程的ping
		printf("<%d> received %s.\n",getpid(),buf);//子进程输出自己的进程号，并说明received ping
		write(fd[1],"pong",BUF_SIZE);//5.子进程向管道中写入pong给父进程
	}

	exit(0);
}
```

**Add the program to `UPROGS` in Makefile.**

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
	$U/_sleep\
	$U/_pingpong\  //here!
```

**进入qemu环境**

```shell
xv6 kernel is booting
hart 1 starting
hart 2 starting
init: starting sh

$ pingpong
```

**输出：**

```shell
<4> received ping.
<3> received pong.
```

### primes([moderate](https://pdos.csail.mit.edu/6.S081/2020/labs/guidance.html))/([hard](https://pdos.csail.mit.edu/6.S081/2020/labs/guidance.html))

Write a concurrent version of prime sieve using pipes. This idea is due to Doug McIlroy, inventor of Unix pipes. The picture halfway down [this page](http://swtch.com/~rsc/thread/) and the surrounding text explain how to do it. Your solution should be in the file `user/primes.c`.

Your goal is to use `pipe` and `fork` to set up the pipeline. The first process feeds the numbers 2 through 35 into the pipeline. For each prime number, you will arrange to create one process that reads from its left neighbor over a pipe and writes to its right neighbor over another pipe. Since xv6 has limited number of file descriptors and processes, the first process can stop at 35.

![](https://github.com/Jomocool/Operator-System/blob/main/MIT6.S081Lab-img/3.png)

[素数筛法](https://zhuanlan.zhihu.com/p/100051075)

![](https://github.com/Jomocool/Operator-System/blob/main/MIT6.S081Lab-img/4.png)

**primes.c**

```c
#include"kernel/types.h"
#include"user/user.h"

#define MSGSIZE 36
#define ZERO '0'
#define ONE '1'

void prime(int pipe_read,int pipe_write){
    char buf[MSGSIZE];//用于接收管道中父进程传来的数据
    read(pipe_read,buf,MSGSIZE);//读取父进程从管道传来的数据
    int val=0;//接收第一个素数
    for(int i=2;i<MSGSIZE;i++){
        if(buf[i]==ONE){//由于2是第一个素数，所以先设置为ONE
            val=i;
            break;
        }
    }
    if(val==0){
        exit(0);//说明全部都处理完了，退出进程
    }

    printf("prime %d\n",val);

    for(int i=1;val*i<MSGSIZE;i++){
        //把素数的倍数淘汰，因为肯定不是素数。同时把val本身置为ZERO，防止后续重复打印
        buf[val*i]=ZERO;
    }

    if(fork()>0){
        //子节点(当前)
        write(pipe_write,buf,MSGSIZE);
        wait(0);
    }else{
        //孙节点
        prime(pipe_read,pipe_write);
    }
}

int main(int argc,char*argv[]){
    int fd[2];
    pipe(fd);//开辟管道
    
    char buf[MSGSIZE];//记录是否为素数
    for(int i=2;i<MSGSIZE;i++){
        buf[i]=ONE;//都先标为素数，之后再淘汰(标为ZERO)
    }

    if(fork()>0){
        //parent process
        buf[0]=ZERO;//ZERO(非素数)
        buf[1]=ONE;//ONE(素数)
        write(fd[1],buf,MSGSIZE);//向管道中写入buf传给子进程
        wait(0);//等待子进程
    }else{
        //child process;
        prime(fd[0],fd[1]);
        wait(0);//子进程也是别人的父进程，等待
    }

    exit(0);
}
```

**Makefile**

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
	$U/_sleep\
	$U/_pingpong\
	$U/_primes\ //here!
```

**sudo make qemu**

```shell
xv6 kernel is booting
hart 2 starting
hart 1 starting
init: starting sh

$ primes
```

**输出**

```shell
prime 2
prime 3
prime 5
prime 7
prime 11
prime 13
prime 17
prime 19
prime 23
prime 29
prime 31
```

**总结：**

1. pipe实际上就是用来帮助进程之间相互传输资源的，但是由于pipe类似于队列，所以要注意读写顺序。
2. pipe实际上很占用资源，所以要注意使用。



### find ([moderate](https://pdos.csail.mit.edu/6.S081/2020/labs/guidance.html))

Write a simple version of the UNIX find program: find all the files in a directory tree with a specific name. Your solution should be in the file `user/find.c`.

