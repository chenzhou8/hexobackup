---
title: 文件描述符与系统IO
date: 2018-10-18 09:40:16
toc: true
categories: Linux系统编程
---

## 一、fopen函数
```c
#include <stdio.h>
FILE *fopen(const char *path, const char *mode);
```
参数说明：
path：要打开的文件路径+文件名
mode：打开模式，下面是第二个参数的说明
`来自CentOS 7：man 3 fopen`

| 选项 | 说明                                                         | 译文                                                         |
| :--: | :----------------------------------------------------------- | :----------------------------------------------------------- |
|  r   | Open text file for reading,The stream is positioned at the beginning of the file. | 打开只读文本文件，流位于文件的开头。                         |
|  r+  | Open for reading and writing. The stream is positioned at the beginning of the file. | 打开可读可写的文本文件，流位于文件的开头。                   |
|  w   | Truncate file to zero length or create text file for writing.The stream is positioned at the beginning of  the file. | 将文件清空或创建用于写入的文本文件。流位于文件的开头。       |
|  w+  | Open for reading and writing.The file is created if it does not exist, otherwise it is truncated.The stream  is  positioned at the beginning of the file. | 打开可读可写的文本文件，如果文件不存在，则创建该文件，否则将被截断，流位于文件的开头。 |
|  a   | Open for appending (writing at end of file).The file is created if it does not exist.The stream is positioned at the end of the file. | 打开以追加（在文件末尾写入）。如果文件不存在，则创建该文件。流位于文件的末尾。 |
|  a+  | Open for reading and appending（writing at end of file).The file is created if it does not exist.The initial file  position for reading is at the beginning of the file，but output is always appended to the end of the file. | 打开读取和追加（在文件结尾写入）。如果文件不存在，则创建该文件。用于读取的初始文件位置在文件的开头，但是输出总是附加在文件的结尾。 |
## 关于fseek、ftell、rewind函数
对于文件的读写方式，C 语言不仅支持简单地顺序读写方式，还支持随机读写（即只要求读写文件中某一指定的部分）。对顺序读写方式来说，随机读写方式需要将文件内部的位置指针移动到需要读写的位置再进行读写，这通常也被称为文件的定位。
### rewind
rewind 函数用于将文件内部的位置指针重新指向一个流（数据流或者文件）的起始位置。这里需要注意的是，这里的"指针"表示的不是文件指针，而是文件内部的位置指针。即随着对文件的读写，文件的位置指针（指向当前读写字节）向后移动。而文件指针指向整个文件，如果不重新赋值，文件指针不会发生改变。
```c
#include <stdio.h>
void rewind(FILE *fp);
```
从上面的函数原型可以看出，rewind 并没有返回值，因此也无法做安全性检查。因此，应该尽量使用 fseek 来替换 rewind 函数，从而以验证流已经成功地回绕。
### fseek
相对于 rewind 函数而言，fseek 函数的功能更加强大，它用来设定文件的当前读写位置，从而可以实现以任意顺序访问文件的不同位置，以实现文件的随机访问。
```c
#include <stdio.h>
int fseek(FILE *fp,long offset,int from);
```
#### fseek的返回值
如果该函数执行成功，fp 将指向以 from 为基准，偏移 offset 个字节的位置，函数的返回值为 0；如果该函数执行失败（比如 offset 超过文件自身大小），则不改变 fp 指向的位置，函数的返回值为 -1，并设置 errno 的值，可以用 perror 函数来输出错误信息。
#### fseek的参数
对于 fseek 函数中的参数：第一个参数 fp 为文件指针；第二个参数 offset 为偏移量，它表示要移动的字节数，整数表示正向偏移，负数表示负向偏移；第三个参数 from 表示设定从文件的哪里开始偏移，取值范围：
SEEK_SET 表示从文件起始位置增加 offset 个偏移量为新的读写位置；
SEEK_CUR 表示从目前的读写位置增加 offset 个偏移量为新的读写位置；
SEEK_END 表示将读写位置指向文件尾后，再增加 offset 个偏移量为新的读写位置。

* 调用 fseek 函数的文件指针 fp 应该指向已经打开的文件，否则将会出现错误。
* fseek 函数一般用于二进制文件，当然也可以用于文本文件。需要特别注意的是，当 fseek 函数用于文本文件操作时，一定要注意回车换行的情况。因为在一般浏览工具（如 UltraEdit）中，回车换行被视为两个字符 0x0D 和 0x0A，但真实的文件读写和定位却按照一个字符 0x0A 进行处理。因此，在碰到此类问题时，可以考虑将文件整个读入内存，然后在内存中手工插入 0x0D的方法，这样可以达到较好的处理效果。
* fseek 函数只返回执行的结果是否成功，并不返回文件的读写位置。因此，你可以使用 ftell 函数来取得当前文件的读写位置。
### ftell
```c
# include<stdio.h>
long ftell(FILE *fp);
```
该函数用于得到文件位置指针当前位置相对于文件首的偏移字节数。在随机方式存取文件时，由于文件位置频繁前后移动，程序不容易确定文件的当前位置。在使用 fseek 函数后，再调用函数 ftell 就能非常容易地确定文件的当前位置。示例代码:
```c
long getfilelength(FILE *fp){
    long curpos = 0L;
    long length = 0L;
    curpos = ftell(fp);
    fseek(fp, 0L, SEEK_END);
    length = ftell(fp);
    fseek(fp, curpos, SEEK_SET);
    return length;
}
```

## 二、Linux系统接口：open
```c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
       
int open(const char *pathname, int flags);
int open(const char *pathname, int flags, mode_t mode);
```
### 参数说明：
pathname：要打开或者要创建的文件

flags：打开文件时，可以传入多个参数选项，用下面的一个或者多个常量进行 `或` 运算，这样就可以根据二进制位来判断打开文件的模式
* O_EDONLY：只读打开
* O_WRONLY：只写打开
* O_RDWR：读写打开
<font color=RED>**这三个常量，必须指定一个且只能指定一个**</font>
* O_CREAT：若文件不存在，则创建它，需要使用mode选项来指定文件的访问权限
* O_APPEND：追加写入
### 返回值：
成功：新打开的文件描述符
失败：-1
### mode参数
mode参数表示设置文件访问权限的初始值，和用户掩码umask有关，比如0644表示-rw-r–r–，也可以用S_IRUSR、S_IWUSR等宏定义按位或起来表示。要注意的是，有以下几点
#### umask
umask与chmod是配套使用的，umask默认情况下的umask值是002，可以直接用umask命令查看，文件权限由open的mode参数和当前进程的umask掩码共同决定。
![](https://s2.ax1x.com/2019/05/05/EwxHOO.png)
![](https://s2.ax1x.com/2019/05/05/EwxL0e.png)
从图中可以看出，只要umask设置的为1的二进制位，在新建文件的时候就不会加上这些权限！所以第三个参数是在第二个参数中有O_CREAT时才作用，如果没有创建新文件，则第三个参数可以忽略！

### 接口使用示例
写入数据：
```c
#include <stdio.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

int main(){
  umask(0);
  int fd = open("myfile", O_WRONLY|O_CREAT, 0664);
  if(fd < 0){
    perror("open");
    return 1;
  }

  int count = 5;
  const char *msg = "hello bit!\n";
  int len = strlen(msg);

  while(count--){
   int ret =  write(fd, msg, len);
   printf("实际写入数据%d字节\n", ret);
  }

  close(fd);
  return 0;
}
```
![](https://s2.ax1x.com/2019/05/05/EwxvtA.png)
读取数据：

```c
#include <stdio.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

int main(){
  int fd = open("myfile", O_RDONLY);
  if(fd < 0){
    perror("open");
    return 1;
  }

  
  const char *msg = "hello bit!\n";
  char buf[1024];

  while(1){
   ssize_t s =  read(fd, buf, strlen(msg));//返回实际读到的字节数

   if(s > 0){
     printf("%s", buf);
   }else{
     break;
   }
  }

  close(fd);
  return 0;
}
```
## 三、文件描述符与FILE结构体
文件描述符就是open函数成功之后的值，本质就是一个数字！
Linux进程默认有3个缺省打开的文件描述符，分别是标准输入0，标准输出1，标准错误2，对应的物理设备分别是：键盘、显示器、显示器

### 进程怎么知道打开了那些文件呢？下面是PCB(task_struct)的一部分
查看task_struct结构体请看：[《深入理解进程》](https://zouchanglin.cn/2018/09/27/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3%E8%BF%9B%E7%A8%8B/)中的task_struct，很显然，Linux的进程是通过files_struct 类型的一个属性来管理打开得文件列表：
![](https://s2.ax1x.com/2019/05/05/Ewzp1P.png)
而现在知道，文件描述符就是从0开始的小整数。当我们打开文件时，操作系统在内存中要创建相应的数据结构来描述目标文件。于是就有了file结构体。表示一个已经打开的文件对象。而进程执行open系统调用，所以必须让进程和文件关联起来。每个进程都有一个指针*files，指向一张表files_ struct，该表最重要的部分就是包涵一个指针数组，每个元素都是一个指向打开文件的指针!所以，本质上，文件描述符就是该数组的下标。所以，只要拿着文件描述符，就可以找到对应的文件！

### 文件描述符的分配规则：
**从0开始分配最小的，可用的！** 看代码理解一下：
```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

int fun(){
  int fd = open("myfile", O_RDONLY);
  if(fd < 0){
    perror("open");
    return 1;
  }

  printf("fd = %d\n", fd);
  close(fd);
  return 0;
}

int fun2(){
  close(0);
  int fd = open("myfile", O_RDONLY);
  if(fd < 0){
    perror("open");
    return 1;
  }

  printf("fd = %d\n", fd);
  close(fd);
  return 0;
}

int main(){
  //fun(); 
  fun2();
  return 0;
}
```
调用fun()的时候打印3，调用fun2()的时候打印0，所以**从0开始分配最小的，可用的！** 
### 文件描述符与重定向
```c
int main(){
  close(1);
  int fd = open("myfile", O_WRONLY|O_CREAT, 00644);
  if(fd < 0){
    perror("open");
    return 1;
  }

  printf("fd = %d\n", fd);
  fflush(stdout);

  close(fd);
  return 0;
}
```
![](https://s2.ax1x.com/2019/05/05/Ewz96f.md.png)
从上面的代码不难看出：本来应该打印到屏幕的内容却输出到了文件中，这就实现了重定向！从下图中可以看出：
![在这里插入图片描述](https://s2.ax1x.com/2019/05/05/EwzF0g.png)
不难得出，要完成输出重定向需要先关闭1号文件描述符所定义的标准输出，然后用其他的文件来占用这个1号文件描述符，所以输出结果便转移到了普通文件中，键盘、屏幕也是文件，所以这也再次体现了**在Linux下：一切皆文件**
所以很久上面的理论：要完成输入重定向，只要关闭0号文件描述符，然后用其他的文件来占用0号文件描述符，这样的话输入重定向就很容易完成了：
先准备一个作为输入的文件(\n即是Linux下的回车换行)：
![](https://s2.ax1x.com/2019/05/05/EwzQnU.png)

```c
int main(){
  close(0);
  int fd = open("myfile2", O_RDONLY);
  if(fd < 0){
    perror("open");
    return 1;
  }
  char buf[1024] = {0};
  scanf("%s", buf);
  printf("buf = %s\n", buf);
  return 0;
}
```
运行结果：
![](https://s2.ax1x.com/2019/05/05/Ewz8AJ.png)
从结果不难看出，其实只要关闭了0号文件描述符，然后打开的文件就会分配到0号文件描述符，所以这样就完成了输入重定向，这样的话追加重定向输入其实也不难，只要把文件指针在写入文件的时候使用`O_APPEND`参数即可完成！

### FILE结构体
FILE结构体是C库中封装的描述文件的结构体：
```c
struct _IO_FILE {
  int _flags;		/* High-order word is _IO_MAGIC; rest is flags. */
#define _IO_file_flags _flags

  //缓冲区相关
  /* The following pointers correspond to the C++ streambuf protocol. */
  /* Note:  Tk uses the _IO_read_ptr and _IO_read_end fields directly. */
  char* _IO_read_ptr;	/* Current read pointer */
  char* _IO_read_end;	/* End of get area. */
  char* _IO_read_base;	/* Start of putback+get area. */
  char* _IO_write_base;	/* Start of put area. */
  char* _IO_write_ptr;	/* Current put pointer. */
  char* _IO_write_end;	/* End of put area. */
  char* _IO_buf_base;	/* Start of reserve area. */
  char* _IO_buf_end;	/* End of reserve area. */
  /* The following fields are used to support backing up and undo. */
  char *_IO_save_base; /* Pointer to start of non-current get area. */
  char *_IO_backup_base;  /* Pointer to first valid character of backup area */
  char *_IO_save_end; /* Pointer to end of non-current get area. */

  struct _IO_marker *_markers;

  struct _IO_FILE *_chain;

  int _fileno; //文件描述符
#if 0
  int _blksize;
#else
  int _flags2;
#endif
  _IO_off_t _old_offset; /* This used to be _offset but it's too small.  */

#define __HAVE_COLUMN /* temporary */
  /* 1+column number of pbase(); 0 is unknown. */
  unsigned short _cur_column;
  signed char _vtable_offset;
  char _shortbuf[1];

  /*  char* _save_gptr;  char* _save_egptr; */

  _IO_lock_t *_lock;
#ifdef _IO_USE_OLD_IO_FILE
};
```
可见，FILE结构体中封装了文件描述符
#### 示例代码
```c
#include <stdio.h>
#include <string.h>
#include <unistd.h>

int main(){
  const char *msg0 = "hello printf\n"; 
  const char *msg1 = "hello fwrite\n";
  const char *msg2 = "hello write\n";
  
  printf("%s", msg0);
  fwrite(msg1, strlen(msg0), 1, stdout);
  write(1, msg2, strlen(msg2));

  fork();
  return 0;
}
```
![](https://s2.ax1x.com/2019/05/05/Ewzt91.md.png)
如果只是打印出来，很显然只是打印了三次，但是如果是重定向到一个文件中，printf与fwrite都输出了两次，只有write始终只输出一次，这是什么原因呢？
**肯定和缓冲区有关系！！！**
先说几种缓冲的区别：
行缓冲：意思就会说只是缓冲一行，当行结束的时候缓冲区就满了，就会刷新流数据，printf()就是典型的行缓冲！
全缓冲：意思就是只有缓冲区写满了才会刷新流，除非我们自己主动刷新
无缓冲：不存在缓冲区，也就是没事每刻都在刷新流！

#### 原因分析
* 一般写入文件的时候就是全缓冲，但是打印到显示器的时候就是行缓冲，当重定向到普通文件的时候行缓冲变成了全缓冲
* 放在缓冲区的数据不会立即刷新，甚至fork之后也不会刷新
* 进程退出后才刷新，写入文件中
* fork（）之后父进程和子进程会发生数据拷贝，所以当父进程准备刷新的时候子进程也有了相同的一份数据
* write函数没有变化，说明wirte函数没有缓冲区
通过分析可知，系统IO接口是没有缓冲区的，而C库的IO函数却有缓冲区，这样的缓冲区是用户级缓冲区，为了提升性能，操作系统也会提供内核缓冲区！