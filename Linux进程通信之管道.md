---
title: Linux进程通信之管道
date: 2018-11-21 23:11:16
toc: true
categories: Linux系统编程
---

**<font color='red'>Linux进程间通信的基本思想是：让两个进程看到一份公共的资源！</font>**
## Linux进程间通信的目的
* 数据传输：⼀个进程需要将它的数据发送给另⼀个进程
* 资源共享：多个进程之间共享同样的资源。
* 通知事件：⼀个进程需要向另⼀个或⼀组进程发送消息，通知它们发生了某种事件（如进程终止时要通知父进程）。
* 进程控制：有些进程希望完全控制另⼀个进程的执行（如Debug进程），此时控制进程希望能够拦截另一个进程的所有陷入和异常，并能够及时知道它的状态改变

## 通信方式之管道
管道是Unix中最古老的进程间通信的形式。
我们把从⼀个进程连接到另⼀个进程的⼀个数据流称为⼀个"管道"
* 管道是面向字节流的
* 管道的生命周期：与进程一致
* 管道只能用于单向通信
* 内核会对管道操作进行同步与互斥

接下来看看管道的使用：

![](https://s2.ax1x.com/2019/05/07/Est2sH.png)

## 匿名管道

### pipe函数
功能：创建匿名管道

```c  
#include <unistd.h>
int pipe(int pipefd[2]);
```
返回值：
On success, zero is returned.  On error, -1 is returned, and errno is set appropriately.
成功返回0，失败返回错误代码！
参数说明：
pipe()  creates  a pipe, a unidirectional data channel that can be used for interprocess communication.  The array pipefd is used to return two file descriptors referring to the ends of the pipe.  pipefd[0] refers  to the  read end of the pipe.  pipefd[1] refers to the write end of the pipe.  Data written to the write end of the pipe is buffered by the kernel until it is read from the read end of the pipe. 
pipe() 创建一个管道，一个可用于进程间通信的单向数据通道。该文件描述符数组用于返回引用管道末端的两个文件描述符。 pipefd [0]指的是管道的读端。 pipefd [1]指的是管道的写端。写入结束的数据管道由内核缓冲，直到从管道的读取端读取。

管道简单使用示例：从键盘读取数据，写⼊管道，读取管道，写到屏幕
```c
#include <string.h>
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>

int main(int argc, char *argv[]) {
	int fds[2];
	char buf[100] = { 0 };
	size_t len;
	ssize_t r_len;
	

	if (pipe(fds) == -1) {
		perror("make pipe");
		exit(1);
	}

	//read from stdin
	while (fgets(buf, 100, stdin)) {
		len = strlen(buf);
		//write to pipe
		if (write(fds[1], buf, len) != len) {
			perror("write to pipe");
			break;
		}
		memset(buf, 0, 100);

		//read from pipe
		if ((r_len = read(fds[0], buf, (size_t)100)) == -1) {
			perror("read form pipe");
			break;
		}

		//write to stdout
		if (write(1, buf, len) != len) {
			perror("write to stdout");
			break;
		}
	}
		
	return 0;
}
```
### 亲缘关系进程直接的通信
上面的示例演示了管道的基本使用方式，但是不包含进程之间的通信！接下来看看父子进程之间的通信：
![](https://s2.ax1x.com/2019/05/07/EstbQg.png)

从文件描述符理解管道：
![](https://s2.ax1x.com/2019/05/07/Estjwn.png)
从内核角度理解管道：
![](https://s2.ax1x.com/2019/05/07/EsNSYV.png)
所以可以看到，管道其实也是文件，在Linux下一切皆文件！

```cpp
#include <stdio.h>
#include <errno.h>
#include <string.h>
#include <unistd.h>
#include <stdlib.h>

int main(){
    int pipefd[2];
    
    //创建一个匿名管道，失败直接退出
    if(pipe(pipefd) == -1){
        perror("pipe");
        exit(EXIT_FAILURE);
    }

    pid_t pid;
    pid = fork();
    if(pid == -1){
        perror("pipe");
        exit(EXIT_FAILURE);
    }

	 //父进程
    if(pid == 0){
        close(pipefd[0]);
        write(pipefd[1], "hello", 5);
        close(pipefd[1]);
        exit(EXIT_SUCCESS);
    }

    close(pipefd[1]);

    char buf[10] = {0};
    read(pipefd[0], buf, 10);
    printf("buf = %s\n", buf);

    return 0;
}
```
### 管道的读写规则
* 当没有数据可读时
	* O_NONBLOCK disable：read调用阻塞，即进程暂停执行，一直等到有数据来到为止
	* O_NONBLOCK enable：read调用返回-1，errno值为EAGAIN
* 当管道满的时候
	* O_NONBLOCK disable： write调用阻塞，直到有进程读走数据
	* O_NONBLOCK enable：调用返回-1，errno值为EAGAIN
* 如果所有管道写端对应的文件描述符被关闭，则read返回0
如果所有管道读端对应的文件描述符被关闭，则write操作会产生信号SIGPIPE,进而可能导致write进程退出，这个不难理解，因为没有人从管道读信息时向管道写入东西是没有意义的，这是一种资源的浪费，所以管道读端对应的文件描述符被关闭时其实应该结束掉write进程；
* 当要写入的数据量不大于`PIPE_BUF`时，linux将保证写入的原子性。
* 当要写入的数据量大于`PIPE_BUF`时，linux将不再保证写入的原子性。


### 管道的特点
* 只能用于具有共同祖先的进程（具有亲缘关系的进程）之间进行通信；通常一个管道由一个进程创建，然后该进程调用fork，此后父、子进程之间就可应用该管道。
* 管道提供流式服务
* 一般而言，进程退出，管道释放，所以<font color='red'>管道的生命周期随进程</font>
* 一般而言，内核会对管道操作进行同步与互斥
* 管道是半双工的，数据只能向一个方向流动；需要双方通信时，需要建立起两个管道

## 命名管道
管道应用的一个限制就是只能在具有共同祖先（具有亲缘关系）的进程间通信。
如果我们想在不相关的进程之间交换数据，可以使用FIFO文件来做这项工作，它经常被称为命名管道。命名管道是一种特殊类型的文件!
接下来使用一个脚本来演示用管道进行通信：
![](https://s2.ax1x.com/2019/05/07/EsNpWT.gif)
如果此时我们关闭读端，那么写端就会退出，道理很简单，和匿名管道一样，如果读端都关闭了那么此时如果写端还在写的话其实是一种资源浪费，于是操作系统直接向写端发信号终止写端进程！

### 匿名管道和命名管道之间的区别
* 匿名管道由pipe函数创建并打开，命名管道由mkfifo函数创建，打开用open函数
* FIFO(命名管道)与pipe(匿名管道)之间唯一的区别在它们创建与打开的方式不同

### 命名管道的打开规则
如果当前打开操作是为读而打开FIFO时：
* O_NONBLOCK disable：阻塞直到有相应进程为写而打开该FIFO
* O_NONBLOCK enable：立刻返回失败，错误码为ENXIO
如果当前打开操作是为写而打开FIFO时：
* O_NONBLOCK disable：阻塞直到有相应进程为读而打开该FIFO
* O_NONBLOCK enable：立刻返回失败，错误码为ENXIO

接下来是一个模拟客户端和服务器端通信的示例：
![](https://s2.ax1x.com/2019/05/07/EsNPlF.gif)
`client.c`

```c
#include <stdio.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <fcntl.h>
#include <unistd.h>
#include <memory.h>

#define FIFONAME "mypipe"

int main() {
    //创建一个命名管道
    mkfifo(FIFONAME, 0664);

    //打开这个命名管道
    int fd = open(FIFONAME, O_WRONLY);

    if(fd < 0){
        return 1;
    }

    char buf[1024];
    while(1){
        printf("Please Enter Your Message To Server# ");
        fflush(stdout);
        //从标准输入读取信息
        ssize_t s = read(0,buf,sizeof(buf));
        buf[s-1] = 0;//覆盖掉之前的回车换行
        
        //向管道写
        write(fd,buf,strlen(buf));
    }
    close(fd);
    return 0;
}
```
`server.c`
```c
#include <stdio.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <fcntl.h>
#include <unistd.h>

#define FIFONAME "mypipe"

int main() {
    //创建一个命名管道
    mkfifo(FIFONAME, 0664);

    //打开这个命名管道
    int fd = open(FIFONAME, O_RDONLY);

    if(fd < 0){
        return 1;
    }

    char buf[1024];
    while(1){
        ssize_t s = read(fd, buf, sizeof(buf)-1);
        if(s > 0){
            //读取成功
            buf[s] = 0;
            printf("client# %s\n", buf);
        }else if(s == 0){
            //写端把文件描述符关闭
            printf("client quit！server quit too!\n");
            break;
        }else{
            break;
        }
    }
    return 0;
}
```
Bash中的管道是采用匿名管道的方式进行通信，由于匿名管道只能用于在具有亲缘关系之间通信，但是由Bash开启的进程之间是属于兄弟关系，自然就可以通过匿名管道进行通信，多个进程(多个管道)的情况下只要每个进程关闭或打开相应的读写端，形成链式数据结构便可以进行通信了(再次佩写服Bash的大佬)