# MIT6.S081Lab

## 环境搭建

网上有一堆环境搭建教程，但是能成功的我觉得不多，终于还是在这个大佬的教程下我才搭建成功的

大佬讲得十分细心，虽然我第一遍卡住了，但是删掉ubuntu，重新安装ubuntu后，就没问题了，基本上每一步都和大佬一模一样

**参考：**

**[帖子详情 - Diz (ourdiz.com)](https://ourdiz.com/post/2/6/39)**



**[首页 - diz (ourdiz.com)](https://ourdiz.com/2/6)**



## **[实验指导书](https://pdos.csail.mit.edu/6.S081/2020/labs)**


## **[b站指导视频](https://b23.tv/ezFe1Qs)**
在此隆重感谢大佬的教学视频

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

### find ([moderate](https://pdos.csail.mit.edu/6.S081/2020/labs/guidance.html))

Write a simple version of the UNIX find program: find all the files in a directory tree with a specific name. Your solution should be in the file `user/find.c`.



**ls.c**

```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"
#include "kernel/fs.h"

char*
fmtname(char *path)
{
  static char buf[DIRSIZ+1];
  char *p;

  // Find first character after last slash.
  for(p=path+strlen(path); p >= path && *p != '/'; p--)
    ;
  p++;

  // Return blank-padded name.
  if(strlen(p) >= DIRSIZ)
    return p;
  memmove(buf, p, strlen(p));
  memset(buf+strlen(p), ' ', DIRSIZ-strlen(p));
  return buf;
}

void
ls(char *path)//path：当前文件路径
{
  char buf[512], *p;
  int fd;
  struct dirent de;
  struct stat st;

  if((fd = open(path, 0)) < 0){//打开文件异常
    fprintf(2, "ls: cannot open %s\n", path);
    return;
  }

  if(fstat(fd, &st) < 0){//把文件状态信息加载到st上，如果函数返回小于0，说明有异常
    fprintf(2, "ls: cannot stat %s\n", path);
    close(fd);
    return;
  }

  switch(st.type){//对于不同状态信息的文件，有不同的展示方式
  case T_FILE://文件
    printf("%s %d %d %l\n", fmtname(path), st.type, st.ino, st.size);
    break;

  case T_DIR://文件夹
    if(strlen(path) + 1 + DIRSIZ + 1 > sizeof buf){//超过buf大小
      printf("ls: path too long\n");
      break;
    }
    strcpy(buf, path);
    p = buf+strlen(buf);
    *p++ = '/';
    while(read(fd, &de, sizeof(de)) == sizeof(de)){
      if(de.inum == 0)
        continue;
      memmove(p, de.name, DIRSIZ);
      p[DIRSIZ] = 0;
      if(stat(buf, &st) < 0){
        printf("ls: cannot stat %s\n", buf);
        continue;
      }
      printf("%s %d %d %d\n", fmtname(buf), st.type, st.ino, st.size);
    }
    break;
  }
  close(fd);
}

int
main(int argc, char *argv[])
{
  int i;

  /*
  调用ls指令：ls 文件名
  argc用于接收参数个数，ls一个，文件名一个，总共两个
  如果argc小于2的话，说明没有文件名可接收，直接展示当前文件夹，所以就是ls(".")
  */
  if(argc < 2){
    ls(".");
    exit(0);
  }
    
  /*
  如果argc>=2，说明有文件名，遍历这些文件夹，依次展示
  */
  for(i=1; i<argc; i++)
    ls(argv[i]);
  exit(0);
}
```

在ls的基础上完成find

**find.c**

```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"
#include "kernel/fs.h"

char*
fmtname(char *path)
{
  static char buf[DIRSIZ+1];
  char *p;

  // Find first character after last slash.
  for(p=path+strlen(path); p >= path && *p != '/'; p--)
    ;
  p++;

  // Return blank-padded name.
  if(strlen(p) >= DIRSIZ)
    return p;
  memmove(buf, p, strlen(p));
  memset(buf+strlen(p), 0, DIRSIZ-strlen(p));
  return buf;
}

//判断当前路径能否递归，.和..无法递归
//1：可递归
//0：不可递归
int norecurse(char *path){
    char* buf=fmtname(path);
    if(buf[0]=='.'&&buf[1]==0){
        return 1;
    }
    if(buf[0]=='.'&&buf[1]=='.'&&buf[2]==0){
        return 1;
    }
    return 0;
}

void
find(char *path,char* target)//在当前文件夹下，找到target
{
  char buf[512], *p;
  int fd;
  struct dirent de;
  struct stat st;

  if((fd = open(path, 0)) < 0){
    fprintf(2, "ls: cannot open %s\n", path);
    return;
  }

  if(fstat(fd, &st) < 0){
    fprintf(2, "ls: cannot stat %s\n", path);
    close(fd);
    return;
  }

  if(strcmp(fmtname(path),target)==0){//标准化文件名和target相同，说明要找的就是当前文件，直接打印即可
    printf("%s\n",path);
  }

  switch(st.type){
  case T_FILE:
    break;

  case T_DIR:
    if(strlen(path) + 1 + DIRSIZ + 1 > sizeof buf){
      printf("ls: path too long\n");
      break;
    }
    strcpy(buf, path);
    p = buf+strlen(buf);
    *p++ = '/';
    while(read(fd, &de, sizeof(de)) == sizeof(de)){//while循环是为了遍历当前文件夹所有文件，同时递归查找target
      if(de.inum == 0)
        continue;
      memmove(p, de.name, DIRSIZ);
      p[DIRSIZ] = 0;
      if(stat(buf, &st) < 0){
        printf("ls: cannot stat %s\n", buf);
        continue;
      }
      if(norecurse(buf)==0){//递归查找
        find(buf,target);
      }
    }
    break;
  }
  close(fd);
}

int
main(int argc, char *argv[])//argv是参数列表，argc是参数列表中的参数个数
{
  if(argc==1){
    printf("usage: find [path] [target]\n");
    exit(0);
  }

  if(argc==2){//找到当前文件夹下的argv[1]文件
    find(".",argv[1]);
    exit(0);
  }

  if(argc==3){//找到argv[1]文件夹下的argv[2]文件
    find(argv[1],argv[2]);
    exit(0);
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
	$U/_primes\
	$U/_find\ //here!
```

**sudo make qemu**

```shell
xv6 kernel is booting
hart 2 starting
hart 1 starting
init: starting sh

$ echo > b
$ mkdir a
$ echo > a/b
$ find . b
```

**输出**

```shell
./b
./a/b
```

**评分**

```shell
./grade-lab-util find

== Test find, in current directory == find, in current directory: OK (3.9s) 
== Test find, recursive == find, recursive: OK (1.2s)
```



### xargs ([moderate](https://pdos.csail.mit.edu/6.S081/2020/labs/guidance.html))

Write a simple version of the UNIX xargs program: read lines from the standard input and run a command for each line, supplying the line as arguments to the command. Your solution should be in the file `user/xargs.c`.

The following example illustrates xarg's behavior:

```shell
    $ echo hello too | xargs echo bye
    bye hello too
    $
```

Note that the command here is "echo bye" and the additional arguments are "hello too", making the command "echo bye hello too", which outputs "bye hello too".

Please note that xargs on UNIX makes an optimization where it will feed more than argument to the command at a time. We don't expect you to make this optimization. To make xargs on UNIX behave the way we want it to for this lab, please run it with the -n option set to 1. For instance

```shell
    $ echo "1\n2" | xargs -n 1 echo line
    line 1
    line 2
    $
```

![](https://github.com/Jomocool/Operator-System/blob/main/MIT6.S081Lab-img/5.png)

**xargs.c**

```c
#include"kernel/types.h"
#include"kernel/param.h"
#include"kernel/stat.h"
#include"user/user.h"
#include"kernel/fs.h"

#define MSGSIZE 16

//echo hello too | xargs echo bye
int main(int argc, char* argv[]){
    sleep(10);//先让xargs sleep，防止find还没找到相应文件夹，xargs就结束了
    // Q1 怎么获取前一个命令的标准化输出（即此命令的标准化输入）
    char buf[MSGSIZE];
    read(0,buf,MSGSIZE);

    //Q2 如果获取到自己的命令行参数
    char *xargv[MAXARG];
    int xargc=0;
    for(int i=1;i<argc;++i){//从1开始，因为0是命令，而不是参数
        xargv[xargc]=argv[i];
        xargc++;
    }

    char *p=buf;
    for(int i=0;i<MSGSIZE;++i){
        if(buf[i]=='\n'){//遇到换行符，交给子进程去处理当前指令
            int pid=fork();
            if(pid>0){
                p=&(buf[i+1]);//父进程处理下一个指令，后移指针
                wait(0);
            }else{
                //Q3 如何使用exec取执行命令
                buf[i]=0;
                xargv[xargc]=p;
                xargc++;
                xargv[xargc]=0;
                xargc++;

                exec(xargv[0],xargv);
                exit(0);
            }
        }
    }
    wait(0);
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
	$U/_primes\
	$U/_find\
	$U/_xargs\
```

**sudo make qemu**

```shell
xv6 kernel is booting
hart 1 starting
hart 2 starting
init: starting sh

sh < xargstest.sh #测试脚本
```

**输出**

```shell
$ $ $ $ $ $ hello
hello
hello
```

**评分**

```shell
sudo ./grade-lab-util xargs

make: 'kernel/kernel' is up to date.
== Test xargs == xargs: OK (2.7s) 
```

## Lab2 System calls

### System call tracing ([moderate](https://pdos.csail.mit.edu/6.S081/2020/labs/guidance.html))

In this assignment you will add a system call tracing feature that may help you when debugging later labs. You'll create a new `trace` system call that will control tracing. It should take one argument, an integer "mask", whose bits specify which system calls to trace. For example, to trace the fork system call, a program calls `trace(1 << SYS_fork)`, where `SYS_fork` is a syscall number from `kernel/syscall.h`. You have to modify the xv6 kernel to print out a line when each system call is about to return, if the system call's number is set in the mask. The line should contain the process id, the name of the system call and the return value; you don't need to print the system call arguments. The `trace` system call should enable tracing for the process that calls it and any children that it subsequently forks, but should not affect other processes.

To start the lab, switch to the syscall branch:

```shell
  $ git fetch
  $ git checkout syscall
  $ make clean
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
	$U/_primes\
	$U/_find\
	$U/_xargs\
	$U/_trace\ #here!
```

**user.h**

```c
// system calls
int fork(void);
int exit(int) __attribute__((noreturn));
int wait(int*);
int pipe(int*);
int write(int, const void*, int);
int read(int, void*, int);
int close(int);
int kill(int);
int exec(char*, char**);
int open(const char*, int);
int mknod(const char*, short, short);
int unlink(const char*);
int fstat(int fd, struct stat*);
int link(const char*, const char*);
int mkdir(const char*);
int chdir(const char*);
int dup(int);
int getpid(void);
char* sbrk(int);
int sleep(int);
int uptime(void);
int trace(int); //here!
```

**usys.pl**

```perl
entry("fork");
entry("exit");
entry("wait");
entry("pipe");
entry("read");
entry("write");
entry("close");
entry("kill");
entry("exec");
entry("open");
entry("mknod");
entry("unlink");
entry("fstat");
entry("link");
entry("mkdir");
entry("chdir");
entry("dup");
entry("getpid");
entry("sbrk");
entry("sleep");
entry("uptime");
entry("trace");//here!
```

**kernel/syscall.h**

```c
// System call numbers
#define SYS_fork    1
#define SYS_exit    2
#define SYS_wait    3
#define SYS_pipe    4
#define SYS_read    5
#define SYS_kill    6
#define SYS_exec    7
#define SYS_fstat   8
#define SYS_chdir   9
#define SYS_dup    10
#define SYS_getpid 11
#define SYS_sbrk   12
#define SYS_sleep  13
#define SYS_uptime 14
#define SYS_open   15
#define SYS_write  16
#define SYS_mknod  17
#define SYS_unlink 18
#define SYS_link   19
#define SYS_mkdir  20
#define SYS_close  21
#define SYS_trace  22//here!
```

**sysproc.c**

```c
//Add a sys_trace() function in kernel/sysproc.c
uint64
sys_trace(void)
{
  printf("sys_trace: Hi!]\n");
  return 0;
}
```

**syscall.c**

```c
extern uint64 sys_chdir(void);
extern uint64 sys_close(void);
extern uint64 sys_dup(void);
extern uint64 sys_exec(void);
extern uint64 sys_exit(void);
extern uint64 sys_fork(void);
extern uint64 sys_fstat(void);
extern uint64 sys_getpid(void);
extern uint64 sys_kill(void);
extern uint64 sys_link(void);
extern uint64 sys_mkdir(void);
extern uint64 sys_mknod(void);
extern uint64 sys_open(void);
extern uint64 sys_pipe(void);
extern uint64 sys_read(void);
extern uint64 sys_sbrk(void);
extern uint64 sys_sleep(void);
extern uint64 sys_unlink(void);
extern uint64 sys_wait(void);
extern uint64 sys_write(void);
extern uint64 sys_uptime(void);
extern uint64 sys_trace(void);//here!

static uint64 (*syscalls[])(void) = {
[SYS_fork]    sys_fork,
[SYS_exit]    sys_exit,
[SYS_wait]    sys_wait,
[SYS_pipe]    sys_pipe,
[SYS_read]    sys_read,
[SYS_kill]    sys_kill,
[SYS_exec]    sys_exec,
[SYS_fstat]   sys_fstat,
[SYS_chdir]   sys_chdir,
[SYS_dup]     sys_dup,
[SYS_getpid]  sys_getpid,
[SYS_sbrk]    sys_sbrk,
[SYS_sleep]   sys_sleep,
[SYS_uptime]  sys_uptime,
[SYS_open]    sys_open,
[SYS_write]   sys_write,
[SYS_mknod]   sys_mknod,
[SYS_unlink]  sys_unlink,
[SYS_link]    sys_link,
[SYS_mkdir]   sys_mkdir,
[SYS_close]   sys_close,
[SYS_trace]   sys_trace, //here!
};

//通过表来驱动这些系统调用函数
```

**proc.h**

```c
// Per-process state
struct proc {
  struct spinlock lock;

  // p->lock must be held when using these:
  enum procstate state;        // Process state
  struct proc *parent;         // Parent process
  void *chan;                  // If non-zero, sleeping on chan
  int killed;                  // If non-zero, have been killed
  int xstate;                  // Exit status to be returned to parent's wait
  int pid;                     // Process ID

  // these are private to the process, so p->lock need not be held.
  uint64 kstack;               // Virtual address of kernel stack
  uint64 sz;                   // Size of process memory (bytes)
  pagetable_t pagetable;       // User page table
  struct trapframe *trapframe; // data page for trampoline.S
  struct context context;      // swtch() here to run process
  struct file *ofile[NOFILE];  // Open files
  struct inode *cwd;           // Current directory
  char name[16];               // Process name (debugging)

  //for trace
  int trace_mask //here!
};
```

**syscall.c**

```c
static char *syscall_names[]={
    "fork","exit","wait","pipe","read","kill","exec","fstat","chdir","dup","getpid","sbrk","sleep",
    "uptime","open","write","mknod","unlink","link","mkdir","close","trace"
};

void
syscall(void)//syscall是调用任何系统函数的入口
{
  int num;
  struct proc *p = myproc();

  num = p->trapframe->a7;
  if(num > 0 && num < NELEM(syscalls) && syscalls[num]) {//如果调用的函数是系统调用的话
    p->trapframe->a0 = syscalls[num]();//a0是寄存器，找到num对应的系统调用，
    int trace_mask=p->trace_mask;
    if((trace_mask>>num)&1){
      printf("%d: syscacll %s -> %d\n",p->pid,syscall_names[num-1],p->trapframe->a0);//知道当前那些系统调用被执行了
    }
  } else {
    printf("%d %s: unknown sys call %d\n",
            p->pid, p->name, num);
    p->trapframe->a0 = -1;
  }
}
```

**proc.c**

```c
// copy trace_mask in the child
  np->trace_mask=p->trace_mask;//here!

  // increment reference counts on open file descriptors.
  for(i = 0; i < NOFILE; i++)
    if(p->ofile[i])
      np->ofile[i] = filedup(p->ofile[i]);
  np->cwd = idup(p->cwd);

  safestrcpy(np->name, p->name, sizeof(p->name));

---------------------------------------------------------------------------------------------------------------

static void
freeproc(struct proc *p)
{
  if(p->trapframe)
    kfree((void*)p->trapframe);
  p->trapframe = 0;
  if(p->pagetable)
    proc_freepagetable(p->pagetable, p->sz);
  p->pagetable = 0;
  p->sz = 0;
  p->pid = 0;
  p->parent = 0;
  p->name[0] = 0;
  p->chan = 0;
  p->killed = 0;
  p->xstate = 0;
  p->state = UNUSED;
  p->trace_mask = 0;//here! reset trace_mask
}
```

**sysproc.c**

```c
//Add a sys_trace() function in kernel/sysproc.c
uint64
sys_trace(void)
{
  int mask;//接收的参数
  if(argint(0, &mask) < 0)
    return -1;

  struct proc *p=myproc();//获取当前进程
  p->trace_mask=mask;
  return 0;
}
```

### Sysinfo ([moderate](https://pdos.csail.mit.edu/6.S081/2020/labs/guidance.html))

In this assignment you will add a system call, `sysinfo`, that collects information about the running system. The system call takes one argument: a pointer to a `struct sysinfo` (see `kernel/sysinfo.h`). The kernel should fill out the fields of this struct: the `freemem` field should be set to the number of bytes of free memory, and the `nproc` field should be set to the number of processes whose `state` is not `UNUSED`. We provide a test program `sysinfotest`; you pass this assignment if it prints "sysinfotest: OK".



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
	$U/_trace\
	$U/_sysinfotest\ //here!
```

**user.h**

```c
struct stat;
struct rtcdate;
struct sysinfo;//here!

// system calls
int fork(void);
int exit(int) __attribute__((noreturn));
int wait(int*);
int pipe(int*);
int write(int, const void*, int);
int read(int, void*, int);
int close(int);
int kill(int);
int exec(char*, char**);
int open(const char*, int);
int mknod(const char*, short, short);
int unlink(const char*);
int fstat(int fd, struct stat*);
int link(const char*, const char*);
int mkdir(const char*);
int chdir(const char*);
int dup(int);
int getpid(void);
char* sbrk(int);
int sleep(int);
int uptime(void);
int trace(int);
int sysinfo(struct sysinfo*);//here!
```

**usys.pl**

```perl
entry("fork");
entry("exit");
entry("wait");
entry("pipe");
entry("read");
entry("write");
entry("close");
entry("kill");
entry("exec");
entry("open");
entry("mknod");
entry("unlink");
entry("fstat");
entry("link");
entry("mkdir");
entry("chdir");
entry("dup");
entry("getpid");
entry("sbrk");
entry("sleep");
entry("uptime");
entry("trace");
entry("sysinfo"); //here!
```

**syscall.h**

```c
// System call numbers
#define SYS_fork    1
#define SYS_exit    2
#define SYS_wait    3
#define SYS_pipe    4
#define SYS_read    5
#define SYS_kill    6
#define SYS_exec    7
#define SYS_fstat   8
#define SYS_chdir   9
#define SYS_dup    10
#define SYS_getpid 11
#define SYS_sbrk   12
#define SYS_sleep  13
#define SYS_uptime 14
#define SYS_open   15
#define SYS_write  16
#define SYS_mknod  17
#define SYS_unlink 18
#define SYS_link   19
#define SYS_mkdir  20
#define SYS_close  21
#define SYS_trace  22
#define SYS_sysinfo 23 //here!
```

**sysall.c**

```c
extern uint64 sys_chdir(void);
extern uint64 sys_close(void);
extern uint64 sys_dup(void);
extern uint64 sys_exec(void);
extern uint64 sys_exit(void);
extern uint64 sys_fork(void);
extern uint64 sys_fstat(void);
extern uint64 sys_getpid(void);
extern uint64 sys_kill(void);
extern uint64 sys_link(void);
extern uint64 sys_mkdir(void);
extern uint64 sys_mknod(void);
extern uint64 sys_open(void);
extern uint64 sys_pipe(void);
extern uint64 sys_read(void);
extern uint64 sys_sbrk(void);
extern uint64 sys_sleep(void);
extern uint64 sys_unlink(void);
extern uint64 sys_wait(void);
extern uint64 sys_write(void);
extern uint64 sys_uptime(void);
extern uint64 sys_trace(void);
extern uint64 sys_sysinfo(void);//here!

static uint64 (*syscalls[])(void) = {
[SYS_fork]    sys_fork,
[SYS_exit]    sys_exit,
[SYS_wait]    sys_wait,
[SYS_pipe]    sys_pipe,
[SYS_read]    sys_read,
[SYS_kill]    sys_kill,
[SYS_exec]    sys_exec,
[SYS_fstat]   sys_fstat,
[SYS_chdir]   sys_chdir,
[SYS_dup]     sys_dup,
[SYS_getpid]  sys_getpid,
[SYS_sbrk]    sys_sbrk,
[SYS_sleep]   sys_sleep,
[SYS_uptime]  sys_uptime,
[SYS_open]    sys_open,
[SYS_write]   sys_write,
[SYS_mknod]   sys_mknod,
[SYS_unlink]  sys_unlink,
[SYS_link]    sys_link,
[SYS_mkdir]   sys_mkdir,
[SYS_close]   sys_close,
[SYS_trace]   sys_trace,
[SYS_sysinfo] sys_sysinfo,//here!
};
```

**syscall.c**

```c
//记得在头文件加#include"sysinfo.h"

uint64
sys_sysinfo(void){
  struct sysinfo info;
  uint64 addr;
  struct proc*p=myproc();
  info.nproc=-1;
  info.freemem=-2;
  if(argaddr(0, &addr) < 0)
    return -1;

  if(copyout(p->pagetable, addr, (char *)&info, sizeof(info)) < 0)
      return -1;

  return 0;
}
```

**kalloc.c**

```c
//Add funtion
uint64 acquire_freemem(){
  struct run *r;
  uint64 cnt=0;//页表数

  acquire(&kmem.lock);
  r = kmem.freelist;
  while(r){
    r=r->next;
    cnt++;
  }
  release(&kmem.lock);

  return cnt*PGSIZE;
}
```

**sysproc.c**

```c
//引用 acquire_freemem()、acquire_nproc()
uint64 acquire_freemem();
uint64 accquire_freemem();

uint64
sys_sysinfo(void){
  struct sysinfo info;
  uint64 addr;
  struct proc*p=myproc();
  info.nproc=acquire_nproc();
  info.freemem=acquire_freemem();
  if(argaddr(0, &addr) < 0)
    return -1;

  if(copyout(p->pagetable, addr, (char *)&info, sizeof(info)) < 0)
      return -1;

  return 0;
}
```

