---
title: Linux进程通信之消息队列
date: 2018-12-05 16:50:51
toc: true
categories: Linux系统编程 
---

## 消息队列
消息队列提供了一个从一个进程向另外一个进程发送一块数据的方法
每个数据块都被认为是有一个类型，接收者进程接收的数据块可以有不同的类型值
消息队列也有管道一样的不足，就是**每个消息的最大长度是有上限的**（MSGMAX），每个消息队列的总的字节数是有上限的（MSGMNB），系统上消息队列的总数也有一个上限（MSGMNI）
**<font color='red'>消息队列不提供同步与互斥</font>**
**<font color='red'>消息队列不提供同步与互斥</font>**
**<font color='red'>消息队列不提供同步与互斥</font>**
![](https://s2.ax1x.com/2019/05/07/EsUmHs.png)

## IPC对象数据结构
IPC全称是：Inter Process Communication (进程间通信)
IPC的源码：/include/linux/ipc.h
**PS:这个链接可以下载到1.0版本的源码
https://mirrors.edge.kernel.org/pub/linux/kernel/v1.0/**

```c
struct ipc_perm
{
  key_t  key;
  ushort uid;   /* owner euid and egid */
  ushort gid;
  ushort cuid;  /* creator euid and egid */
  ushort cgid;
  ushort mode;  /* access modes see mode flags below */
  ushort seq;   /* sequence number */
};

```
消息队列结构源码：/include/linux/msg.h
```c
/* one msqid structure for each queue on the system */
struct msqid_ds {
    struct ipc_perm msg_perm;
    struct msg *msg_first;  /* first message on queue */
    struct msg *msg_last;   /* last message in queue */
    time_t msg_stime;       /* last msgsnd time */
    time_t msg_rtime;       /* last msgrcv time */
    time_t msg_ctime;       /* last change time */ 
    struct wait_queue *wwait;
    struct wait_queue *rwait;
    ushort msg_cbytes;      /* current number of bytes on queue */
    ushort msg_qnum;        /* number of messages in queue */
    ushort msg_qbytes;      /* max number of bytes on queue */
    ushort msg_lspid;       /* pid of last msgsnd */
    ushort msg_lrpid;       /* last receive pid */
};
```
![](https://s2.ax1x.com/2019/05/07/EsUuEn.png)
在管道的通信方式中，两个进程需要看到同一份资源那就是管道文件，具有亲缘关系的进程之间其实很容易使用匿名管道进行通信，没有亲缘关系的进程也可以使用文件名来识别同一块资源(管道)，**因为Linux下，同一个目录下不可能出现同名文件，根据这个道理，每个文件的路径+自身的文件名就会形成唯一的标识**，那么对于两个不相干的进程如何看到同一份消息队列呢？其实`ipc_perm`结构体的`key_t  key`值解决了这个问题，可以把这个消息队列的唯一标识存储在`key_t  key`中，这样只要是`key_t  key`一致，那么看到的消息对列就是一致的！

## ftok
根据指定的路径和一个8位的整数来生成一个唯一的识别码
```c
#include <sys/types.h>
#include <sys/ipc.h>
key_t ftok(const char *pathname, int proj_id)
```
`pathname`:指定的文件路径
`proj_id`:自己设定的序列号
`return`:唯一标识ID
The ftok() function uses the identity of the file named  by  the  given pathname  (which  must  refer  to an existing, accessible file) and the least significant 8 bits of proj_id (which must be nonzero) to generate a  key_t  type  System  V  IPC  key,  suitable  for use with msgget(2),semget(2), or shmget(2).
The resulting value is the same for all pathnames that  name  the  same file,  when  the  same  value  of  proj_id is used.  The value returned should be different when the (simultaneously  existing)  files  or  the project IDs differ.
On  success,  the  generated key_t value is returned.  On failure -1 is returned, with errno indicating the error as  for  the  stat(2)  system call.
ftok()函数使用由给定路径名命名的文件的标识(必须引用现有的可访问文件)和proj_id的最低8位（必须非零）来生成key_t类型System V IPC密钥 ，适用于msgget，semget或shmget。
当使用相同的proj_id值时，对于命名同一文件的所有路径名，结果值是相同的。 当（同时存在的）文件或项目ID不同时，返回的值应该不同。
成功时，返回生成的key_t值。 失败时返回-1，errno指示stat系统调用的错误。

共享内存，信号量，消息队列都是通过共享文件的方式进行通信，但是这些共享文件的区分方式就是需要一个识别码，你可以理解为每个人都有自己的身份证号码一样，都是唯一的，简言之就是：ftok函数可以根据指定的路径和一个8位的整数来生成一个唯一的识别码，一般在UNIX中，通常是将文件的索引节点取出，然后在前面加上子序号就得到key_t的值。
**注意：ftok根据文件路径生成ID和文件的权限无关**
## msgget
用来创建和访问一个消息队列
```c
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>
int msgget(key_t key, int msgflg)
```
`key`: 某个消息队列的名字
`msgflg`:由九个权限标志构成，用法和创建文件时使用的mode模式标志一致
`return`：成功返回一个非负整数，即该消息队列的标识码；失败返回-1
![](https://s2.ax1x.com/2019/05/07/EsUB8K.png)

关键点：
IPC_CREAT：如果内核中不存在键值与key相等的消息队列，则新建一个消息队列；如果存在这样的消息队列，返回此消息队列的标识符
IPC_CREAT|IPC_EXCL：如果内核中不存在键值与key相等的消息队列，则新建一个消息队列；如果存在这样的消息队列则报错

错误代码：
EACCES：指定的消息队列已存在，但调用进程没有权限访问它
EEXIST：key指定的消息队列已存在，而msgflg中同时指定IPC_CREAT和IPC_EXCL标志
ENOENT：key指定的消息队列不存在同时msgflg中没有指定IPC_CREAT标志
ENOMEM：需要建立消息队列，但内存不足
ENOSPC：需要建立消息队列，但已达到系统的限制

## msgctl
控制消息队列
```c
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>
int msgctl(int msqid, int cmd, struct msqid_ds *buf);
```
`msqid`: 由msgget函数返回的消息队列标识码
`cmd`:是将要采取的动作,（有三个可取值）
`return`：成功返回0，失败返回-1
>**cmd:将要采取的动作，分别如下**

| 命令     | 说明                                                         |
| :------- | :----------------------------------------------------------- |
| IPC_STAT | 把msqid_ds结构中的数据设置为消息队列的当前关联值，获得msgid的消息队列头数据到buf中 |
| IPC_SET  | 在进程有足够权限的前提下，把消息队列的当前关联值设置为msqid_ds数据结构中给出的值，设置消息队列的属性，要设置的属性需先存储在buf中，可设置的属性包括：msg_perm.uid、msg_perm.gid、msg_perm.mode以及msg_qbytes |
IPC_RMID|删除消息队列|
错误代码：
EACCESS：参数cmd为IPC_STAT，确无权限读取该消息队列
EFAULT：参数buf指向无效的内存地址
EIDRM：标识符为msqid的消息队列已被删除
EINVAL：无效的参数cmd或msqid
EPERM：参数cmd为IPC_SET或IPC_RMID，却无足够的权限执行

## msgsnd函数
把一条消息添加到消息队列中
```c
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>
int msgsnd(int msqid, const void *msgp, size_t msgsz, int msgflg);
```
`msgid`: 由msgget函数返回的消息队列标识码
`msgp`:是一个指向结构体的指针，指针指向准备发送的消息
* msgp可以是任何类型的结构体，但第一个字段必须为long类型，即表明此发送消息的类型，msgrcv根据此接收消息

`msgsz`:是msgp指向的消息长度，这个长度不含保存消息类型的那个long int长整型
`msgflg`:控制着当前消息队列满或到达系统上限时将要发生的事情，
* 0：当消息队列满时，msgsnd将会阻塞，直到消息能写进消息队列
* IPC_NOWAIT：当消息队列已满的时候，msgsnd函数不等待立即返回
* IPC_NOERROR：若发送的消息大于size字节，则把该消息截断，截断部分将被丢弃，且不通知发送进程

`return`：成功返回0；失败返回-1
注意：消息结构必须小于系统规定的上限值；其次，必须以一个long int长整数开始，接收者函数将利用这个长整数确定消息的类型，消息结构参考形式如下：
```c
struct msgbuf {
	long mtype;
	char mtext[1];
}
```

msgsnd()解除阻塞的三个条件：
①    不满足消息队列满或个数满两个条件，即消息队列中有容纳该消息的空间。
②    msqid代表的消息队列被删除。
③    调用msgsnd函数的进程被信号中断。
## msgrcv
从标识符为msqid的消息队列读取消息并存于msgp中，读取后把此消息从消息队列中删除
```c
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>
int msgsnd(int msqid, const void *msgp, size_t msgsz, int msgflg);
```
`msgid`: 由msgget函数返回的消息队列标识码
`msgp`:是一个指针，指针指向准备接收的消息，
`msgsz`:是msgp指向的消息长度，这个长度不含保存消息类型的那个long int长整型
`msgtype`:它可以实现接收优先级的简单形式
`msgflg`:控制着队列中没有相应类型的消息可供接收时将要发生的事
* 0: 阻塞式接收消息，没有该类型的消息msgrcv函数一直阻塞等待
* IPC_NOWAIT：如果没有返回条件的消息调用立即返回，此时错误码为ENOMSG
* IPC_EXCEPT：与msgtype配合使用返回队列中第一个类型不为msgtype的消息
* IPC_NOERROR：如果队列中满足条件的消息内容大于所请求的size字节，则把该消息截断，截断部分将被丢弃

`return`:成功返回实际放到接收缓冲区里去的字符个数，失败返回-1

msgtype=0返回队列第一条信息
msgtype>0返回队列第一条类型等于msgtype的消息　
msgtype<0返回队列第一条类型小于等于msgtype绝对值的消息，并且是满足条件的消息类型最小的消息


## ipcs、ipcrm
这是两个命令，ipcs:可以显示IPC资源，ipcrm用于手动删除IPC资源
管道虽然也可以用于进程间通信，但是管道的声明周期是随进程的，但是消息队列、信号量、共享内存的声明周期是随内核，需要我们自己调用函数或者命令手动清除，所以当每次使用消息队列的时候不要忘记删除IPC资源：

### ipcs
显示 IPC 设施的信息。
`ipcs [资源选项...] [输出选项]`
`ipcs -m|-q|-s -i <id>`
选项：
-i, --id <id>  打印由<id>标识的资源的详细信息
-h, --help     display this help
-V, --version  display version

资源选项：
-m, --shmems      共享内存段
-q, --queues      消息队列
-s, --semaphores  信号量
-a, --all         全部(默认)
输出选项：
-t, --time        显示附加、脱离和更改时间
-p, --pid         显示创建者和最后操作者的 PID
-c, --creator     显示创建者和拥有者
-l, --limits      显示资源限制
-u, --summary     显示状态摘要
-h, --human       以易读格式显示大小
-b, --bytes       以字节数显示大小


### ipcrm
移除某个 IPC 资源。
ipcrm [选项]
`ipcrm shm|msg|sem <id>...`
选项：
 -m, --shmem-id <id>        按 id 号移除共享内存段
 -M, --shmem-key <键>       按键值移除共享内存段
 -q, --queue-id <id>        按 id 号移除消息队列
 -Q, --queue-key <键>       按键值移除消息队列
 -s, --semaphore-id <id>    按 id 号移除信号量
 -S, --semaphore-key <键>  按键值移除信号量
 -a, --all[=<shm|msg|sem>]  (将指定类别中的)全部移除
 -v, --verbose              解释正在进行的操作

方式一：使用ipcs -q命令查询IPC资源中消息队列的msqid，再使用ipcrm
![](https://s2.ax1x.com/2019/05/07/EsU6DH.png)
方式二：在代码中使用消息队列控制函数msgctl设置参数为IPC_RMID
下面是两个进程之间使用消息队列通信的代码：
comm.c

```c
#include "comm.h"

static int CommGetMsgQueue(int flag){
    key_t k = ftok(PATHNAME, PROJID);//失败返回-1
    if(k < 0){
        perror("ftok error");
        return -1;
    }

    int msgid = msgget(k, flag);

    if(msgid < 0){
        perror("msgget error");
    }
    return msgid;
}

//创建消息队列
int CreateMsgQueue(){
    return CommGetMsgQueue(IPC_CREAT|IPC_EXCL|0666);//附加权限
}

//打开消息队列
int OpenMsgQueue(){
    return CommGetMsgQueue(IPC_CREAT);
}

//发送消息
void SendMsg(int msgid, char msg[], int type){
    //准备要发出的数据
    struct msgbuf s_msg;
    s_msg.mtype = type;
    strcpy(s_msg.mtext, msg);

    if(msgsnd(msgid, (void*)&s_msg, sizeof(s_msg.mtext), 0) < 0){
        printf("msgsend error");
    }
}

//接受消息
void RecvMsg(int msgid, char msg[], int type){
    struct msgbuf _msg;
    //读取消息到_msg中
    if(msgrcv(msgid, (void*)&_msg, sizeof(_msg.mtext),type, 0) > 0){
        //把数据传出去
        strcpy(msg, _msg.mtext);
    }
}


//删除消息队列
void DestroyMsgQueue(int msgid){
    msgctl(msgid, IPC_RMID, NULL);
}
```
comm.h
```c
#ifndef INC_32_CODE_COMM_H
#define INC_32_CODE_COMM_H

#include <stdio.h>
#include <errno.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>
#include <memory.h>
#include <unistd.h>

#define PATHNAME "/tmp"
#define PROJID 0x6666

#define SERVER_TYPE 1
#define CLIENT_TYPE 2

struct msgbuf{
    long mtype;
    char mtext[256];
};

//创建消息队列
int CreateMsgQueue();

//打开消息队列
int OpenMsgQueue();

//删除消息队列
void DestroyMsgQueue(int msgid);

//发送消息
void SendMsg(int msgid, char msg[], int type);

//接受消息
void RecvMsg(int msgid, char msg[], int type);

#endif //INC_32_CODE_COMM_H
```
client.c
```c
#include "comm.h"

int main(){

    int msgid = OpenMsgQueue();

    //接收其他进程（server）发的消息
    char msg[256];
    while(1){
        RecvMsg(msgid,msg,SERVER_TYPE);
        printf("RECV = %s\n", msg);
        sleep(1);
    }
    return 0;
}
```
server.c
```c
#include "comm.h"

int main(){
    int msgid = CreateMsgQueue();
    printf("msgid = %d\n",msgid);

    while(1){
        //发送消息到消息队列
        SendMsg(msgid, "hello,xpu", SERVER_TYPE);
        sleep(1);
    }

    SendMsg(msgid, "hello,xpu", SERVER_TYPE);
    sleep(1);
    SendMsg(msgid, "hello,xpu", SERVER_TYPE);
    sleep(1);
    SendMsg(msgid, "hello,xpu", SERVER_TYPE);
    sleep(1);
    SendMsg(msgid, "hello,xpu", SERVER_TYPE);
    sleep(1);
#if 0

    //自己接收自己发的消息
    char msg[256];
    RecvMsg(msgid,msg,SERVER_TYPE);
    printf("RECV = %s\n", msg);
    sleep(1);
    RecvMsg(msgid,msg,SERVER_TYPE);
    printf("RECV = %s\n", msg);
    sleep(1);
    RecvMsg(msgid,msg,SERVER_TYPE);
    printf("RECV = %s\n", msg);
    sleep(1);
    RecvMsg(msgid,msg,SERVER_TYPE);
    printf("RECV = %s\n", msg);
    sleep(1);
    RecvMsg(msgid,msg,SERVER_TYPE);
    printf("RECV = %s\n", msg);
    sleep(1);
#endif
    DestroyMsgQueue(msgid);
    return 0;
}
```