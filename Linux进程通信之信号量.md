---
title: Linux进程通信之信号量
date: 2018-11-27 17:21:17
toc: true
categories: Linux系统编程
---

虽然本文是记录使用信号量保证进程的同步与互斥的，但是其实也可以看做是进程之间的通信问题，为了与前面的保持一致，所以还是叫做 Linux进程间通信了 (强迫症...)
## 信号量
### 基本概念
进程间通信的方式有管道、消息队列、共享内存这些都是进程间的信息通信，而信号量可以理解为进程使用的临界资源的状态说明，信号量主要用于保证同步与互斥

* 临界资源：两个进程看到的一份公共资源称之为临界资源
* 临界区：各个进程中访问临界资源的代码叫做临界区
* 互斥：每个进程访问临界资源的时候必须是独占式的(排他式的)，只能自己一个人访问
* 同步：防止不间断的占有资源和释放资源，这样的话其他进程就会长时间得不到资源，这样会造成进程的饥饿问题

由此可见我们之前用于进程间通信的管道，消息队列，共享内存都是临界资源，管道是内核已经提供了同步与互斥，但是消息队列和共享内存都是不保证同步与互斥的

### 信号量PV原语
信号量：本质上是一把计数器
如果一个信号只有0或者1，那么这个就是二元信号量，所以二元信号量可以实现**互斥锁**

P操作：计数器 `--`
V操作：计数器 `++`
信号量本身也是临界资源，所以P、V操作必须是原子的

### 信号量集结构
```c
struct ipc_perm { 
       key_t __key;    // 提供给 semget（）的键 
       uid_t uid;      // 所有者有效 UID  
       gid_t gid;      // 所有者有效 GID 
       uid_t cuid;     // 创建者有效 UID 
       gid_t cgid;     // 创建者有效 GID
       unsigned short mode;     // 权限 
       unsigned short __seq;    // 序列号
}; 
//信号量集的结构
struct semid_ds {
	struct ipc_perm sem_perm;   // 所有者和权限
	time_t sem_otime;           // 上次执行semop的时间  
	time_t sem_ctime;           // 上次更新时间 
	unsigned short sem_nsems;   // 在信号量集合里的索引
}
```
## 信号量API
### semget
作用：用于创建信号量集

```c
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/sem.h>

int semget(key_t key, int nsems, int semflg);
```

`key` 信号集的名字，这个与再创建管道、消息队列、共享内存等用的key是一致的

`nsems` 信号集中信号量的个数，一般为1（信号集底层就是数组）

`semflg` 同创建消息队列等一样的权限，sem_flags取两个值，`IPC_CREATE` 和 `IPC_EXCL`，需要配权限使用

* IPC_CREATE 表示若信号量已存在，返回该信号量标识符
* IPC_EXCL 表示若信号量已存在，返回错误

`return` 成功返回一个非负整数，即该信号集的标识码；失败返回-1

num_sems:信号量的数目，

### shmctl

作用：用于控制信号量集

```c
#include <sys/ipc.h>
#include <sys/shm.h>

int shmctl(int shmid, int semnum, int cmd, ...);
```

`shmid` 这个就是要控制的信号量集

`semnum` 这个是具体要控制的信号量，因为shmid只能指明是哪一个信号量集(数组)，而semnum就是数组下标

`cmd` 将要采取的动作(有三个可取值)

* SETVAL (常用) 用来把信号量初始化为一个已知的值。p这个值通过union semun中的val成员设置，其作用是在信号量第一次使用的时候
* GETVAL 获取信号量集中的信号量计数值
* IPC_STAT 把semid_ds结构中的数据设置为信号量集的当前关联值
* IPC_SET 在进程有足够权限的情况下，把信号量集的当前关联值设置为semid_ds数据结构中给出的值
* IPC_RMID (常用) 删除信号量集

### semop

作用：修改信号量集中的值 

```c
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/sem.h>

int semop(int semid, struct sembuf *sops, unsigned nsops);
```

`semid` 这个就是要修改的信号量集

`sops` 如下结构体的指针，这个结构体是这样的：

```c
struct sembuf{  
    short sem_num;//除非使用一组信号量，否则它为0  
    short sem_op;//信号量在一次操作中需要改变的数据，通常是两个数，一个是-1，即P（等待）操作，一个是+1，即V（发送信号）操作。  
    short sem_flg;//通常为SEM_UNDO,使操作系统跟踪信号,并在进程没有释放该信号量而终止时，操作系统释放信号量
}; 
```
`nsops` 信号量的个数

`return` 成功返回0，失败返回1

### 信号量使用示例

`makefile`

```makefile
test:comm.c main.c
	gcc -o $@ $^

.PHONY:clean
clean:
	rm -rf $@
```

`comm.c` && `comm.h` && `main.c`

```c
#ifndef __COMM_H__
#define __COMM_H__

#include <stdio.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/sem.h>
#include <unistd.h>
#include <wait.h>

#define PATHNAME "."
#define PROJ_ID 0x6666

union semun {
  int val; /* Value for SETVAL */
  struct semid_ds *buf; /* Buffer for IPC_STAT, IPC_SET */
  unsigned short *array; /* Array for GETALL, SETALL */
  struct seminfo *__buf; /* Buffer for IPC_INFO */
};

int createSemSet(int nums);

int initSem(int semid, int nums, int initVal);

int getSemSet(int nums);

int P(int semid, int who);

int V(int semid, int who);

int destorySemSet(int semid);

#endif //!__COMM_H__


//---------------------comm.c----------------------------

#include "comm.h"

static int commSemSet(int nums, int flags){
  key_t _key = ftok(PATHNAME, PROJ_ID);
  if(_key < 0){
    perror("ftok");
    return -1;
  }

  int semid = semget(_key, nums, flags);
  if(semid < 0){
    perror("semget");
    return -2;
  }
  return semid;
}

int createSemSet(int nums){
  return commSemSet(nums, IPC_CREAT|IPC_EXCL|0666);  
}

int getSemSet(int nums){
  return commSemSet(nums, IPC_CREAT);
}

int initSem(int semid, int nums, int initVal){
  union semun _un;
  _un.val = initVal;
  if(semctl(semid, nums, SETVAL, _un) < 0){
    perror("semctl");
    return -1;
  }
  return 0;
}

static int commPV(int semid, int who, int op){
  struct sembuf _sf;
  _sf.sem_num = who;
  _sf.sem_op = op;
  _sf.sem_flg = 0;

  if(semop(semid, &_sf, 1) < 0){
    perror("semop");
    return -1;
  }
  return 0;
}

int P(int semid, int who){
  return commPV(semid, who, -1);
}

int V(int semid, int who){
  return commPV(semid, who, 1);
}

int destorySemSet(int semid){
  int ret = semctl(semid, 0, IPC_RMID);
  if(ret < 0){
    perror("semctl");
    return -1;
  }
  return ret;
}
//----------------------------main.c----------------------
#include "comm.h"

int main(){
  int semid = createSemSet(1);
  initSem(semid, 0, 1);

  pid_t id = fork();

  if(id == 0){
    int _semid = getSemSet(0);
    while(1){
      //P(_semid, 0);
      printf("A");
      fflush(stdout);
      usleep(100000);
      printf("A");
      fflush(stdout);
      usleep(100000);
      //V(_semid, 0);
    }
  }
  else{
    while(1){
      //P(semid, 0);
      printf("B");
      fflush(stdout);
      usleep(100000);
      printf("B");
      fflush(stdout);
      usleep(100000);
      //V(semid, 0);
    }
    wait(NULL);
  }

  destorySemSet(semid);
  return 0;
}
```
打开PV操作时与未打开时的对比：
![](https://s2.ax1x.com/2019/05/07/EsIxKg.png)
同样的使用`ipcs -s`命令即可查看信号量，使用`ipcrm -s `即可释放信号量资源

## 进程间通信总结

### 管道

* 数据只能向一个方向流动；需要双方通信时，需要建立起两个管道 
* 匿名管道只能用于具有亲缘关系的进程，否则使用命名管道
* 管道内部保证同步机制，从而保证访问数据的一致性。
* 管道是面向字节流的
* 管道生命周期随进程，进程在管道在，进程消失管道对应的端口也关闭，两个进程都消失管道也消
* 管道读端关闭，操作系统向写端发信号终止写端进程
* 每写一块数据的最大长度是有上限的

### System V

#### 消息队列

* 消息队列提供了一个从一个进程向另外一个进程发送一块数据的方法
* 每个数据块都被认为是有一个类型，接收者进程接收的数据块可以有不同的类型值
* 每写一块数据的最大长度是有上限的，这点与管道一致
* 每个消息队列的总的字节数是有上限的（MSGMNB），系统上消息队列的总数也有上限
* 消息队列生命周期随内核
* 不保证同步与互斥

#### 共享内存

* 共享内存区是最快的IPC形式，无需内核干预，直接映射同一块物理内存
* 作为IPC资源存在，共享内存生命周期同样随内核
* 不保证同步与互斥

#### 信号量

* 主要提供对进程间共享资源访问控制机制。相当于内存中的标志，进程可以根据它判定是否能够访问某些共享资源，同时，进程也可以修改该标志。除了用于访问控制外，还可用于进程同步。
* 二元信号量：最简单的信号量形式，信号灯的值只能取0或1，类似于互斥锁



### Socket

* 两台计算机相互通信本质上也是两个不在同一个计算机上的进程之间的通信(事实上本机之间的进程通过Socket通信也属于这个范畴)，以后再说！