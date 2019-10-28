---
title: Linux进程通信之共享内存
date: 2018-12-05 21:01:05
toc: true
categories: Linux系统编程
---

## 共享内存
共享内存按照页为基本单位分配的，一页是4K
共享内存无同步与互斥，生命周期随内核
共享内存无同步与互斥，生命周期随内核
共享内存无同步与互斥，生命周期随内核
共享内存区是最快的IPC形式，一旦这样的内存映射到共享它的进程的地址空间，这些进程间数据传递不再涉及到内核，换句话说是进程不再通过执行进入内核的系统调用来传递彼此的数据：
![](https://s2.ax1x.com/2019/05/07/Es5zA1.png)

## 共享内存的数据结构
```c
struct shmid_ds {
	struct ipc_perm shm_perm; /* operation perms */
	int shm_segsz; /* size of segment (bytes) */
	__kernel_time_t shm_atime; /* last attach time */
	__kernel_time_t shm_dtime; /* last detach time */
	__kernel_time_t shm_ctime; /* last change time */
	__kernel_ipc_pid_t shm_cpid; /* pid of creator */
	__kernel_ipc_pid_t shm_lpid; /* pid of last operator */
	unsigned short shm_nattch; /* no. of current attaches */
	unsigned short shm_unused; /* compatibility */
	void *shm_unused2; /* ditto - used by DIPC */
	void *shm_unused3; /* unused */
};
```
## shmget
得到一个共享内存标识符或创建一个共享内存对象并返回共享内存标识符
```c
#include <sys/ipc.h>
#include <sys/shm.h>
int shmget(key_t key, size_t size, int shmflg)
```
`key`:此值来源于ftok返回的IPC键值
`size`大于0的整数：新建的共享内存大小，以字节为单位
`shmflg` 取共享内存标识符，若不存在则函数会报错
 * IPC_CREAT：如果内核中不存在键值与key相等的共享内存，则新建一个共享内存；如果存在这样的共享内存，返回此共享内存的标识符
* IPC_CREAT|IPC_EXCL：如果内核中不存在键值与key相等的共享内存，则新建一个消息队列；如果存在这样的共享内存则报错

`return `:成功返回共享内存的标识符,出错返回-1，错误原因存于error中
错误代码：
EINVAL：参数size小于SHMMIN或大于SHMMAX
EEXIST：预建立key所指的共享内存，但已经存在
EIDRM：参数key所指的共享内存已经删除
ENOSPC：超过了系统允许建立的共享内存的最大值(SHMALL)
ENOENT：参数key所指的共享内存不存在，而参数shmflg未设IPC_CREAT位
EACCES：没有权限
ENOMEM：核心内存不足

## shmat
把共享内存区对象映射到调用进程的地址空间，其实就是将共享内存绑定到当前进程
```c
#include <sys/types.h>
#include <sys/shm.h>
void *shmat(int shmid, const void *shmaddr, int shmflg)
```
`msqid`:共享内存标识符
`shmaddr`指定共享内存出现在进程内存地址的什么位置，直接指定为NULL让内核自己决定一个合适的地址位置
`shmflg`:SHM_RDONLY：为只读模式，其他为读写模式，默认写0即可
`return`:成功：附加好的共享内存地址,出错：-1，错误原因存于error中

错误代码
EACCES：无权限以指定方式连接共享内存
EINVAL：无效的参数shmid或shmaddr
ENOMEM：核心内存不足
注意：fork后子进程继承已连接的共享内存地址。exec后该子进程与已连接的共享内存地址自动脱离。进程结束后，已连接的共享内存地址会自动脱离

## shmdt
与shmat函数相反，解除当前进程与共享内存的绑定
```c
#include <sys/types.h>
#include <sys/shm.h>
int shmdt(const void *shmaddr)
```
`msqid`:共享内存标识符
`return`:成功：0 出错：-1，错误原因存于error中

错误代码
EINVAL：无效的参数shmaddr
函数调用并不删除所指定的共享内存区，而只是将之前绑定的共享内存解除绑定

## shmctl
控制共享内存
```c
#include <sys/types.h>
#include <sys/shm.h>
int shmctl(int shmid, int cmd, struct shmid_ds *buf);
```
`msqid`:共享内存标识符
`cmd`:对共享内存的控制选项
* IPC_STAT：得到共享内存的状态，把共享内存的shmid_ds结构复制到buf中
* IPC_SET：改变共享内存的状态，把buf所指的shmid_ds结构中的uid、gid、mode复制到共享内存的shmid_ds结构内
* IPC_RMID：删除这片共享内存

`buf`:共享内存管理结构体，不关心则设置为NULL
`return`:成功：0,出错：-1，错误原因存于error中

错误代码
EACCESS：参数cmd为IPC_STAT，确无权限读取该共享内存
EFAULT：参数buf指向无效的内存地址
EIDRM：标识符为msqid的共享内存已被删除
EINVAL：无效的参数cmd或shmid
EPERM：参数cmd为IPC_SET或IPC_RMID，却无足够的权限执行
## ipcs与ipcrm
与消息队列一致，由于共享内存的生命周期随内核，要么使用共享内存控制函数将其释放，那么通过命令手动释放，`ipcs -m`就可以查看IPC资源中的共享内存，`iprm -m shmid`就可以释放对应shmid号的共享内存
![](https://s2.ax1x.com/2019/05/07/EsIStx.png)

## 通信示例
writer.c
```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <unistd.h>
#include <memory.h>

int main() {
    key_t k = ftok(".", 0x7777);//.表示当前目录

    if(k < 0){
        printf("ftok error\n");
        return 1;
    }

    //申请共享内存
    int shmid = shmget(k, 4096,IPC_CREAT|IPC_EXCL|0666);

    if(shmid < 0){
        printf("shmget error\n");
    }

    //绑定共享内存
    char *buf = shmat(shmid, NULL, 0);
    if(buf == NULL){
        printf("shmar error\n");
    }

    int i =0;
    memset(buf, '\0', 4096);
    while(i < 26){
        sleep(1);
        buf[i] = 'A'+i;
        i++;
    }
    //取消绑定共享内存
    shmdt(shmid);

    //释放共享内存
    shmctl(shmid, IPC_RMID, NULL);
    return 0;
}
```
reder.c
```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <unistd.h>

int main() {
    key_t k = ftok(".", 0x7777);//.表示当前目录

    if(k < 0){
        printf("ftok error\n");
        return 1;
    }

    //申请共享内存
    int shmid = shmget(k, 4096,IPC_CREAT);

    if(shmid < 0){
        printf("shmget error\n");
    }

    //绑定共享内存
    char *buf = shmat(shmid, NULL, 0);
    if(buf == NULL){
        printf("shmar error\n");
    }

    while(1){
        sleep(1);
        printf("%s\n", buf);
    }


    //取消绑定共享内存
    shmdt(shmid);
    return 0;
}
```