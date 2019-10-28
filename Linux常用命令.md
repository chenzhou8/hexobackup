---
title: Linux常用命令
date: 2018-04-09 18:53:29
toc: true
categories: Linux应用
---


本次介绍如下命令：du、 df、 top、 free、 pstack、su、sudo、 adduser、 passwd
# du命令
Linux下命令，统计目录（或文件）所占磁盘空间的大小！
## 格式：`du [选项] [文件]`
完整格式：`du [-abcDhHklmsSx][-L <符号连接>][-X <文件>][--block-size][--exclude=<目录或文件>][--max-depth=<目录层数>][--help][--version][目录或文件]`
## 参数说明
>**-a或-all 显示目录中个别文件的大小。**
**-b或-bytes 显示目录或文件大小时，以byte为单位。**
**-c或--total 除了显示个别目录或文件的大小外，同时也显示所有目录或文件的总和。**
**-k或--kilobytes 以KB(1024bytes)为单位输出。**
**-m或--megabytes 以MB为单位输出。**
**-s或--summarize 仅显示总计，只列出最后加总的值。**
**-h或--human-readable 以K，M，G为单位，提高信息的可读性。**
**-x或--one-file-xystem 以一开始处理时的文件系统为准，若遇上其它不同的文件系统目录则略过。**
**-L<符号链接>或--dereference<符号链接> 显示选项中所指定符号链接的源文件大小。**
**-S或--separate-dirs 显示个别目录的大小时，并不含其子目录的大小。**
**-X<文件>或--exclude-from=<文件> 在<文件>指定目录或文件。**
**--exclude=<目录或文件> 略过指定的目录或文件。**
**-D或--dereference-args 显示指定符号链接的源文件大小。**
**-H或--si 与-h参数相同，但是K，M，G是以1000为换算单位。**
**-l或--count-links 重复计算硬件链接的文件。**
## 使用示例
接下来看看这样一个现象，大小不一样，对于.viminfo 这个文件 ls 查看信息查出是862字节，为什么到du命令下就变成4k了？
![](https://s2.ax1x.com/2019/05/02/EtOPIO.png)
原因解释：因为底层的内存管理是以页面为单位，Linux内核的默认页面大小就是4k，至于为什么这个设计，详见：[《为什么Linux内核页面最小单位是4K》](https://blog.csdn.net/justlinux2010/article/details/8910136)
注意最下面的是一个总结值，也可以使用-s直接显示总结值，对显示结果排序：
![](https://s2.ax1x.com/2019/05/02/EtOkJe.png)

# df命令

df命令作用是列出文件系统的整体磁盘空间使用情况。可以用来查看磁盘已被使用多少空间和还剩余多少空间。 
df命令显示系统中包含每个文件名参数的磁盘使用情况，如果没有文件名参数，则显示所有当前已挂载文件系统的磁盘空间使用情况 ，在默认情况下，磁盘空间是以1KB为单位进行显示的，但是，如果POSIXLY_CORRECT环境变量被设置为true，这种情况下默认使用512字节为单位显示！
## 格式： `df [选项] [文件名]`
## 参数说明
>**-a：--all，显示所有的文件系统，包括虚拟文件系统** 
**-B：--block-size，指定单位大小。比如1k，1m等** 
**-h：--human-readable，以人们易读的GB、MB、KB等格式显示**。 
**-H：--si，和-h参数一样，但是不是以1024，而是1000，即1k=1000，而不是1k=1024。** 
**-i：--inodes，不用硬盘容量，而是以inode的数量来显示**
**-k：以KB的容量显示各文件系统，相当于--block-size=1k。**
**-m：以KB的容量显示各文件系统，相当于--block-size=1m。** 
**-l：--local，只显示本地文件系统。 **
**--no-sync：在统计使用信息之前不调用sync命令(默认)。 **
**-sync：在统计使用信息之前调用sync命令。 **
**-P：--portability，使用POSIX格式显示** 
**-t：--type=TYPE，只显示指定类型的文件系统.** 
**-T：--print-type，显示文件系统类型**
**-x：--exclude-type=TYPE，不显示指定类型的文件系统** 
**--help：显示帮助信息。 **
**--version：显示版本信息。**

![](https://s2.ax1x.com/2019/05/02/EtOueP.png)
Filesystem：代表该文件系统时哪个分区，所以列出的是设备名称。
1K-blocks：说明下面的数字单位是1KB，可利用-h或-m来改变单位大小，也可以用-B来设置。
Used：已经使用的空间大小。
Available：剩余的空间大小。
Use%：磁盘使用率。如果使用率在90%以上时，就需要注意了，避免磁盘容量不足出现系统问题，尤其是对于文件内容增加较快的情况(如/home、/var/spool/mail等)。
Mounted on：磁盘挂载的目录，即该磁盘挂载到了哪个目录下面。
![](https://s2.ax1x.com/2019/05/02/EtO1Jg.png)

![](https://s2.ax1x.com/2019/05/02/EtO8zj.png)

# top命令
实时动态地查看系统的整体运行情况，是一个综合了多方信息监测系统性能和运行信息的实用工具。通过top命令所提供的互动式界面，用热键可以管理。
## 格式：`top [选项]`
## 参数说明
>**-b：以批处理模式操作**
**-c：显示完整的治命令**
**-d：屏幕刷新间隔时间**
**-I：忽略失效过程**
**-s：保密模式**
**-S：累积模式**
**-i<时间>：设置间隔时间**
**-u<用户名>：指定用户名**
**-p<进程号>：指定进程**
**-n<次数>：循环显示的次数**

top命令中一些字段的含义
`VIRT：virtual memory usage 虚拟内存`

* 进程“需要的”虚拟内存大小，包括进程使用的库、代码、数据等
* 假如进程申请100m的内存，但实际只使用了10m，那么它会增长100m，而不是实际的使用量

`RES：resident memory usage 常驻内存`

* 进程当前使用的内存大小，但不包括swap out
* 包含其他进程的共享
* 如果申请100m的内存，实际使用10m，它只增长10m，与VIRT相反
* 关于库占用内存的情况，它只统计加载的库文件所占内存大小

`SHR：shared memory 共享内存`

* 除了自身进程的共享内存，也包括其他进程的共享内存
* 虽然进程只使用了几个共享库的函数，但它包含了整个共享库的大小
* 计算某个进程所占的物理内存大小公式：RES – SHR
* swap out后，它将会降下来

`DATA：真实内存`

* 数据占用的内存。如果top没有显示，按f键可以显示出来。
* 真正的该程序要求的数据空间，是真正在运行中要使用的。

![](https://s2.ax1x.com/2019/05/02/EtOJQs.png)

显示完整命令示例
![](https://s2.ax1x.com/2019/05/02/EtOtLq.png)
>**显示完整命令** `top -c`
**以批处理模式显示程序信息** `top -b`
**以累积模式显示程序信息** `top -S`
**设置信息更新次数** `top -n 2`(表示更新两次后终止更新显示)
**设置信息更新时间** `top -d 3`(表示更新周期为3秒)
**显示指定的进程信息** `top -p 139`(显示进程号为139的进程信息，CPU、内存占用率等)
**显示更新十次后退出** `top -n 10`
**使用者将不能利用交谈式指令来对行程下命令** `top -s`
**将更新显示二次的结果输入到名称为 top.log 的档案里** `top -n 2 -b < top.log`

### 使用示例
![](https://s2.ax1x.com/2019/05/02/EtOdoT.png)<hr/>

![](https://s2.ax1x.com/2019/05/02/EtOBYF.png)<hr/>
![](https://s2.ax1x.com/2019/05/02/EtOsSJ.png)

# free命令
Linux free命令用于显示内存状态。
free指令会显示内存的使用情况，包括实体内存，虚拟的交换文件内存，共享内存区段，以及系统核心使用的缓冲区等。
### 格式：`free [-选项][-s <间隔秒数>]`
###参数说明
>**-b 　以Byte为单位显示内存使用情况。**
**-k 　以KB为单位显示内存使用情况。**
**-m 　以MB为单位显示内存使用情况。**
**-o 　不显示缓冲区调节列。**
**-s<间隔秒数> 　持续观察内存使用状况。**
**-t 　显示内存总和列。**
**-V 　显示版本信息**

![](https://s2.ax1x.com/2019/05/02/EtOcO1.png)

###详细解释
![](https://s2.ax1x.com/2019/05/02/EtO4YD.png)
我们可以发现，free命令会输出这4行，第二行是站在操作系统的角度去看待的内存使用情况，也就是总共有2054224KB（默认情况下单位是KB），换算过来大约是1.96G，这些内存中只有1917816KB被使用了，剩余为136804KB，0.13G 可用。
shared是共享的意思，表示几个进程的共享内存，buffer缓冲区之意，cached也是缓存的意思，接下来说说他们的区别，先看看外语解释：
>A buffer is something that has yet to be "written" to disk. 
A cache is something that has been "read" from the disk and stored for later use.

翻译过来就是：
**缓冲区是尚未被“写入”到磁盘的东西。**
**缓存是从磁盘上“读取”并存储以供以后使用的东西。**

buffer是用于存放要输出到disk（块设备）的数据的，而cache是存放从disk上读出的数据。这二者是为了提高IO性能的，并由OS管理。Linux和其他成熟的操作系统（例如windows），为了提高IO的性能，总是要多cache一些数据，这也就是为什么cached memory比较大，而free比较小的原因！

free输出的第三行是从一个应用程序的角度看系统内存的使用情况。
对于-buffers/cache，表示一个应用程序认为系统被用掉多少内存；
对于+buffers/cache，表示一个应用程序认为系统还有多少内存；
因为被系统cache和buffer占用的内存可以被快速回收，所以通常+buffers/cache比-buffers/cache会大很多！

# pstack命令
## 格式：`pstack <-PID>`
显示每个进程的栈跟踪。pstack 命令必须由相应进程的属主或 root 运行。可以使用 pstack 来确定进程挂起的位置。此命令允许使用的唯一选项是要检查的进程的 PID。
需要先安装此工具：
![](https://s2.ax1x.com/2019/05/02/EtOTld.png)
![](https://s2.ax1x.com/2019/05/02/EtOqmt.png)
这个命令在排查进程问题时非常有用，比如我们发现一个服务一直处于work状态（如假死状态，好似死循环），使用这个命令就能轻松定位问题所在；可以在一段时间内，多执行几次pstack，若发现代码栈总是停在同一个位置，那个位置就需要重点关注，很可能就是出问题的地方!
## 作用归纳
*  查看线程数(比pstree, 包含了详细的堆栈信息)
*  能简单验证是否按照预定的调用顺序/调用栈执行
*  采用高频率多次采样使用时, 能发现程序当前的阻塞在哪里, 以及性能消耗点在哪里
* 能反映出疑似的死锁现象(多个线程同时在wait lock, 具体需要进一步验证)

# su命令
Linux su命令用于变更为其他使用者的身份，除 root 外，需要键入该使用者的密码。使用权限：所有使用者
## 格式：`su [-fmp] [-c command] [-s shell] [--help] [--version] [-] [USER [ARG]]`
## 参数说明
>**-f 或 --fast 不必读启动档（如 csh.cshrc 等），仅用于 csh 或 tcsh**
**-m -p 或 --preserve-environment 执行 su 时不改变环境变数**
**-c command 或 --command=command 变更为帐号为 USER 的使用者并执行指令（command）后再变回原来使用者**
**-s shell 或 --shell=shell 指定要执行的 shell （bash csh tcsh 等），预设值为 /etc/passwd 内的该使用者（USER） shell**
**--help 显示说明文件**
**--version 显示版本资讯**
**--l 或 --login 这个参数加了之后，就好像是重新 login 为该使用者一样，大部份环境变数（HOME **SHELL USER等等）都是以该使用者（USER）为主，并且工作目录也会改变，如果没有指定USER ，内定是 root**
**USER 欲变更的使用者帐号**
**ARG 传入新的 shell 参数**

# sudo命令
Linux sudo命令以系统管理者的身份执行指令，也就是说，经由 sudo 所执行的指令就好像是 root 亲自执行。使用权限：在 /etc/sudoers 中有出现的使用者。
参数说明
>**-V 显示版本编号**
**-h 会显示版本编号及指令的使用方式说明**
**-l 显示出自己（执行 sudo 的使用者）的权限**
**-v 因为 sudo 在第一次执行时或是在 N 分钟内没有执行（N 预设为五）会问密码，这个参数是重新做一次确认，如果超过 N 分钟，也会问密码**
**-k 将会强迫使用者在下一次执行 sudo 时问密码（不论有没有超过 N 分钟）**
**-b 将要执行的指令放在背景执行**
**-p prompt 可以更改问密码的提示语，其中 %u 会代换为使用者的帐号名称， %h 会显示主机名称**
**-u username/#uid 不加此参数，代表要以 root 的身份执行指令，而加了此参数，可以以 username 的身份执行指令（#uid 为该 username 的使用者号码）**
**-s 执行环境变数中的 SHELL 所指定的 shell ，或是 /etc/passwd 里所指定的 shell**
**-H 将环境变数中的 HOME （家目录）指定为要变更身份的使用者家目录（如不加 -u 参数就是系统管理者 root ）**
**command 要以系统管理者身份（或以 -u 更改为其他人）执行的指令**

## sudo命令的特点

* sudo能够限制用户只在某台主机上运行某些命令。
* sudo提供了丰富的日志，详细地记录了每个用户干了什么。它能够将日志传到中心主机或者日志服务器。
* sudo使用时间戳文件来执行类似的“检票”系统。当用户调用sudo并且输入它的密码时，用户获得了一张存活期为5分钟的票（这个值可以在编译的时候改变）。
* sudo的配置文件是sudoers文件，它允许系统管理员集中的管理用户的使用权限和使用的主机。它所存放的位置默认是在/etc/sudoers，属性必须为0440。
# adduser命令
Linux adduser命令用于新增使用者帐号或更新预设的使用者资料。
adduser 与 useradd 指令为同一指令（经由符号连结 symbolic link）。
使用权限：系统管理员。
adduser是增加使用者。相对的，也有删除使用者的指令，userdel
## 格式：`adduser [login ID]`
完全格式：`adduser -D [-g default_group] [-b default_home] [-f default_inactive] [-e default_expire_date] [-s default_shell]`
## 参数说明
**-c comment 新使用者位于密码档（通常是 /etc/passwd）的注解资料**
**-d home_dir 设定使用者的家目录为 home_dir ，预设值为预设的 home 后面加上使用者帐号 loginid**
**-e expire_date 设定此帐号的使用期限（格式为 YYYY-MM-DD），默认值为永久有效**
**-f inactive_time 范例**
## 使用示例
添加一个一般用户 `useradd name`

为添加的用户指定相应的用户组 `useradd ? g root kk`	 添加用户kk，并指定用户所在的组为root用户组

创建一个系统用户 `useradd ?r kk ` 创建一个系统用户kk

为新添加的用户指定/home目录 `useradd-d /home/myf kk ` 新添加用户kk，其home目录为/home/myf，当用户名kk登录主机时，系统进入的默认目录为/home/myf
# passwd命令
Linux passwd命令用来更改使用者的密码！
## 格式：`passwd [-k] [-l] [-u [-f]] [-d] [-S] [username]`

## 参数说明
>`必要参数`
**-d 删除密码**
**-f 强制执行**
**-k 更新只能发送在过期之后**
**-l 停止账号使用**
**-S 显示密码信息**
**-u 启用已被停止的账户**
**-x 设置密码的有效期**
**-g 修改群组密码**
**-i 过期后停止用户账号**
`选择参数`
**--help 显示帮助信息**
**--version 显示版本信息**

## 使用示例
删除Tim的密码 `passwd -d Tim`

# Linux下的命令分类
![](https://s2.ax1x.com/2019/05/02/EtX96s.png)
