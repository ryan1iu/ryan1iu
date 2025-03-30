---
title: Linux 系统编程笔记
description: null
date: 2024-12-27T04:12:32.000Z
image: null
math: null
license: null
hidden: false
comments: true
draft: false
categories:
  - 笔记
tags:
  - Linux
  - 系统编程
---

# C编程基础

## 基本注意事项

1. 头文件包含的重要性
2. 以函数为单位进行程序编写
3. 变量先定义再使用
4. return 0
5. 多用空格和空行
6. 大段函数注释可以通过#if 0 --- #endif 进行注释
7. 没有单位的数值在计算机中是没有意义的

`算法`： 解决问题的方法。（流程图、NS图、有限状态机FSM）  
`程序`： 用某种语言实现算法  
`进程`： 防止写越界，防止内存泄露

## 数据类型

1. 所占字节数
2. float 类型
3. char型是否有符号
4. 不同类型的0值： 0，‘0’， “0”， ‘\0’
5. 数据类型与后序代码中所使用的输入输出要相匹配，防止自相矛盾

### 常量与变量

`常量`： 在程序执行过程中值不会发生变化的量  
分类： 整型常量、实型常量、字符常量、字符串常量、标识常量

- 整型常量： 1，123
- 实型常量： 3.14
- 字符常量： 由单引号引起来的单个的字符或转义字符
- 字符串常量： 由双引号引起来的一个或多个字符组成的序列
  - 特殊情况： 空串，“”，字符串末尾以'\0'结束，因此空串也会占据一个字节的存储空间
- 标示常量： #define 定义的常量，预处理阶段，占用编译时间，GNU C 对宏进行了拓展，Linux中使用的就是GNU C

`变量`：用来保存一些特定内容，并在执行过程中值随时会发生变化的量

```
[存储类型] 数据类型 标识符 = 值
```

`标识符`：由字母、数字、下划线组成且不能以数字开头的一个标识序列  
`存储类型`： auto static register extern(说明型)

- auto: 默认、自动分配空间、自动回收空间
- register： 寄存器类型， 建议型关键字，由gcc做最终决定
  - 只能用来定义局部变量，不能定义全局变量；
  - 大小有限制，数据的大小不能超出当前寄存器的长度
  - 寄存器没有地址，所以一个寄存器类型的变量无法打印地址查看或使用
- static： 静态型，自动初始化为零值或空值，并且其变量的值具有继承性
- extern: 说明型，意味着不能改变被说明的变量的值或类型

### 变量的生命周期与作用范围

变量的生命周期本质上取决于其存储在进程地址空间中的哪个区域当中。对于`auto`和`register`类型的变量，在运行过程中分别存储在栈和寄存器中，其生存周期为函数调用开始到结束阶段。
对于`static`类型和全局变量，其在程序运行过程中存储在程序段中，因此其生命周期为程序整个运行期间。

对于局部变量（auto\register\局部static）而言，其作用域为定义该变量的函数或复合语句内部  
对于定义在函数外部的变量（外部static\全局变量）而言，其中`外部static`变量的作用域仅限于当前文件，无法被其他文件访问。`全局变量`的作用域为整个程序，如何需要在当前文件中访问其他文件定义的全局变量，需要使用`extern`关键字。

## 运算符

![img](https://imagebed-1300955178.cos.ap-beijing.myqcloud.com/20241230211112.png?imageSlim)

## 标准I/O

区分标准I/O与系统调用I/O

- 格式化输入输出scanf printf
- 字符输入输出 getchat putchar
- 字符串输入输出 gets puts

### 格式化输入输出

```c
int printf(const char *restrict format, ...);
int scanf(const char *restrict format, ...);
```

format格式：

```
% [修饰符] 格式字符
```

修饰符：  
![modify](https://imagebed-1300955178.cos.ap-beijing.myqcloud.com/20250101105411.png?imageSlim)

格式字符：  
![format](https://imagebed-1300955178.cos.ap-beijing.myqcloud.com/20250101084954.png?imageSlim)

scanf使用注意事项：

1. 在scanf中使用%s接受字符串是一个非常危险的操作，可能导致内存越界，最好使用专门的字符串接受函数
2. 如果在循环中使用scanf函数，一定要对其返回值进行校验。例如

```c
void func(void) {
    int i;
    while(1) {
        scanf("%d", &i);
        printf("%d\n", i);
    }
}
```

如果输出的内容与%d不匹配，那么程序就会进入死循环3. 抑制符的使用

```c
/*
* 如果连续调用两次scanf函数从终端读取内容，那么后一个scanf函数不能接收到正确的结果
*/
```

### 缓冲区刷新机制

C语言中，当使用标准I/O时，会有一个缓冲区来暂时存储输入输出的内容来提高效率。
行缓冲：在遇到\n时或者缓冲区已满时进行输出，终端使用行缓冲模式
全缓冲：缓冲区满时进行输出，文件使用全缓冲模式
标准错误流：标准错误流不缓冲

## 流程控制

## 数组

## 指针

## 构造类型

### struct内存对齐

1. 成员的起始地址必须是对齐值的整数倍，对齐值通常是数据类型的大小
2. 结构体的大小必须是最大对齐值的整数倍，否则需要在结构体末尾进行填充
3. 成员的排列顺序影响结构体的大小

### union的应用场景

1. 节省内存，当多个数据类型`不会`同时使用时，可以使用Union节省内存
2. 类型转换，通过Union可以实现相同位模式下的不同数据类型的解释

```c
union convert {
	int i;
	unsigned int j;
};

int main()
{
	union convert a;
	a.i = 0xffffffff;
	/**
     * a.i = -1
     * a.j = 4294967295
     */
	printf("a.i = %d\na.j = %u\n", a.i, a.j);

	return 0;
}
```

3. 硬件编程，在嵌入式开发中Union可以用来访问寄存器的不同部分

```c
union Register {
	unsigned int i;
	struct {
		unsigned char byte1;
		unsigned char byte2;
		unsigned char byte3;
		unsigned char byte4;
	} bytes;
};

int main()
{
	union Register r;
	r.i = 0x12345678;
	/*
     * byte1 = 78
     * byte2 = 56
     * byte3 = 34
     * byte4 = 12
     * 说明当前机器采用小端模式
     * */
	printf("byte1 = %x\nbyte2 = %x\nbyte3 = %x\nbyte4 = %x\n",
	       r.bytes.byte1, r.bytes.byte2, r.bytes.byte3, r.bytes.byte4);

	return 0;
}
```

## 函数

# Unix系统编程

## 系统调用IO

文件IO=系统调用IO
文件描述符是在文件IO中贯穿始终的类型

### 文件操作符的概念

整型数，数组下标，文件描述符优先使用当前可用范围内最小的

### 文件IO操作，read，write，open，close，lseek

### open系统调用的flags

可以在 flags 中进行按位或的零个或多个文件创建标志和文件状态标志。文件创建标志包括 O_CLOEXEC、O_CREAT、O_DIRECTORY、O_EXCL、O_NOCTTY、O_NOFOLLOW、O_TMPFILE 和 O_TRUNC。文件状态标志是下面列出的所有其他标志。这两组标志之间的区别在于，文件创建标志会影响打开操作本身的语义，而文件状态标志会影响后续 I/O 操作的语义。

### 文件IO与标准IO的区别

关键区别是标准IO提供了一个缓冲区，可以减少系统调用次数，减小模式转换的开销
标准IO的吞吐量大，文件IO的响应快

标准IO与文件IO不可混用

### IO的效率问题

### 文件共享

### 原子操作

### 程序中的重定向 dup dup2

`dup` 和 `dup2` 是 Unix/Linux 系统中用于 **复制文件描述符** 的函数。它们允许程序创建新的文件描述符，指向与原始文件描述符相同的文件表项，从而共享同一个文件偏移量、访问模式等。

| 特性             | `dup`                      | `dup2`                    |
| ---------------- | -------------------------- | ------------------------- |
| 参数数量         | 一个（`oldfd`）            | 两个（`oldfd`, `newfd`）  |
| 可指定新描述符   | 否，返回当前最小可用描述符 | 是，由 `newfd` 指定       |
| 自动关闭新描述符 | 不适用                     | 是（如果 `newfd` 已打开） |
| 功能             | 复制文件描述符             | 重定向文件描述符          |

这两个函数是文件描述符操作的基础，在系统编程中非常常用，例如实现 I/O 重定向或构建更复杂的流处理机制。

### 同步 sync fsync fdatasync

### fcntl() ioctl()

### /dev/fd/ 目录

## 目录和文件

### 获取文件属性

stat

### 文件访问权限

st_mode 是一个16位的位图

### umask: 防止产生权限过松的文件

### 文件权限的修改

chmod
fchmod

### 粘住位

t位，现在已经不再常用，目的是让某一个二进制文件在执行完成后留在内存中方便下一次加载

### 文件系统：FAT和UFS

### 硬链接、符号链接

link
unlink

rename
remove

remove() deletes a name from the filesystem. It calls unlink(2) for files, and rmdir(2) for directories.

硬链接与目录项是同义词，硬链接建立有限制：不能够跨分区建立，不能够给目录建立；符号链接可以

### utime: 更改文件最后读的时间和最后修改的时间

### 目录的创建和销毁

mkdir
rmdir

### 更改当前工作路径

chdir
fchdir
getcwd

### 分析目录/读取目录内容

glob
opendir
readdir
rewinddir
seekdir
closedir

### 系统数据文件和信息

和进程环境相关，和要做的ls练习相关

### /etc/passwd

    getpwuid() 根据uid或name来查询
    getpwnam()

### /etc/group

    getgrgid()
    getgrgrnam()

### /etc/shadow

    getspnam() 根据name返回shadow文件中的一行
    crypt() 加密
    getpass() 关闭终端回显，从终端接收密码

### 时间戳

    各种时间的转换函数，可以参考unix系统编程里面的图示
    time_t char * struct tm
    time()
    gmtime()
    localtime()
    mktime()
    strftime() 格式化的时间和日期

## 进程环境

### main函数

### 进程的终止 背诵

    正常终止
        从main函数返回
        调用exit 返回值从-128到127
            在进程正常终止时会调用atexit注册的函数
        调用_exit或_Exit
            exit是库函数，_exit是系统调用
        最后一个线程从其启动例程返回
        最后一个线程调用pthread_exit
    异常终止
        调用abort函数，发送一个杀死进程的信号
        接到一个信号并终止
        最后一个线程对其取消请求做出响应

### 命令行参数的分析

    getopt()
    getopt_long()

### 环境变量

    程序员和用户的约定
    KEY = VALUE
    getenv()
    setenv() 改变或添加一个环境变量
    unsetenv() 删除一个环境变量
    putenv() 不好用

### C程序的存储空间布局

    ps axf
    pmap

### 库

    动态库
    静态库
    共享库/手工装载库
        dlopen()
        dlclose()
        dlerror()
        dlsym()

### 函数跳转

    setjmp() 安全的跨函数跳转，区别于goto
    longjmp()
    有一种情况不能跳，以后学到信号再来补充

### 资源的获取以及控制

    getrlimit()
    setrlimit()

### 实现myls

支持ls -a -l -n -i /dirorfile
inode节点号 文件类型文件权限 用户 组 大小 时间 文件名

## 进程基本知识

### 进程的终止 背诵

    正常终止
        从main函数返回
        调用exit 返回值从-128到127
            在进程正常终止时会调用atexit注册的函数
        调用_exit或_Exit
            exit是库函数，_exit是系统调用
        最后一个线程从其启动例程返回
        最后一个线程调用pthread_exit
    异常终止
        调用abort函数，发送一个杀死进程的信号
        接到一个信号并终止
        最后一个线程对其取消请求做出响应

### 进程标识符pid

    pid_t
    命令ps
    进程号是顺次向下增长
    getpid()
    getppid()

### 进程的产生

    fork()
        fork后父子进程的区别
            frok的返回值不同
            pid不同，ppid不同
            未决信号和文件锁不继承，资源利用量清0
        init进程：是所有进程的祖先进程
        调度器的策略决定哪个进程先运行
        在fork之前要调用fflush刷新缓冲区
    vfork() 基本废弃

### 进程的消亡与释放资源

    wait()
    waitpid()
    waitid()

### exec()函数族的使用

    execl()
    execlp()
    execle()
    execv()
    execvp()

### 用户权限和组权限

    u+s
    g+s
        real
            用户的实际身份标识，代表运行进程的实际用户
            与用户登录系统时的身份一致，通常由登录会话分配
        effective
            进程实际使用的权限标识，用于决定进程运行时的权限级别
        save
            不一定存在
        S_ISUID     04000   set-user-ID bit (see execve(2))
            如果设置了这个位，进程在执行该文件时将获得文件所有者的权限而不是进程所有者的权限
        S_ISGID     02000   set-group-ID bit (see below)
    getuid()
    geteuid()
    getgid()
    getegid()
    setuid() set effictive uid sudo的底层原理
    setgid()
    setreuid()
    setregid()
    seteuid()
    setegid()

### 观摩课：解释器文件

    #!/bin/解释器

### system()

    相当于 execl("/bin/sh", "sh", "-c", command, (char *) NULL);

### 进程会计

    acct() 方言
    当进程结束时会在指定的文件中写入进程信息

### 进程时间

    times()

### 守护进程

    会话session，sid
    终端
        一个会话绑定一个终端
    前台进程组和后台进程组
        前台进程组最多只能有一个
        前台进程组可以接收来自终端的输入
    setsid()
        使进程脱离控制终端
    getpgrp()
    getpgid()
    单实例守护进程: 锁文件/var/run/name.pid

### 系统日志

    syslogd系统日志
    openlog()
    syslog()
    closelog()

## 并发-信号

同步：
异步：事件什么时候到来不知道，事件会产生什么样的结果不知道
异步事件的处理：查询、通知

### 信号的概念

信号是软件层面的中断
信号的响应依赖于中断 \*信号会打断阻塞的系统调用 open sleep write read accept poll...

### signal()

### 信号的不可靠

信号的行为不可靠

### sig_atomic_t的局限性

虽然`sig_atomic_t`保证了单次读写操作的安全性，但它不能保证多步操作的原子性

1. count++不是原子操作
   - 读取count
   - 递增count
   - 写入count

### 可重入函数

信号处理函数需要使用[可重入函数](./reentrant_function.md)
所有的系统调用都是可重入的
一部分库函数是可重入的

### 信号屏蔽字和Pending集的处理

未决集和阻塞集 pending mask, 内核为每个进程维护，位图，32位
mask是掩码，默认全为1，pending默认全为0
mask & pending

### 信号的响应过程

信号是在程序从内核态到用户态转化之前进行一次检查，mask & pending，来判断是否有待处理信号
如果存在待处理信号，内核将该信号的mask位和pending位置零（目的是保证handler在执行过程中不会因为同样的信号被打断），
然后将原本程序上下文中保存的返回地址，替换为信号Handler的地址，信号处理函数执行完成之后返回，
将之前置零的mask位置1，重新进行mask & pending
因此
信号处理会有延迟
思考
如何忽略掉一个信号
标准信号为什么会丢失
标准信号的响应没有严格的顺序
不能从信号处理函数中随意的往外跳（setjump\longjmp),使用sigsetjmp和siglongjmp替代

### 信号常用函数

- kill()
  向进程或进程组发信号，也可以用来检测某个进程是否存在，具体看手册
- arise()
  给当前进程或线程发送一个信号
- alarm()
- pause()
  等待一个信号
- abort()
  `abort()` 函数首先会解除对 `SIGABRT` 信号的阻塞，然后向调用进程发送该信号（类似于调用 `raise(3)`）。这将导致进程异常终止，除非 `SIGABRT` 信号被捕获并且信号处理程序没有返回（参见 `longjmp(3)`）。

如果 `SIGABRT` 信号被忽略，或者被一个会返回的信号处理程序捕获，`abort()` 函数仍然会终止进程。它会通过恢复 `SIGABRT` 信号的默认处理方式，然后再次发送该信号来实现这一点。

与其他异常终止的情况一样，使用 `atexit(3)` 和 `on_exit(3)` 注册的函数不会被调用。

- system()

### sleep()函数的局限性

可以把sleep当作alarm和pause的封装（但Linux不是这样实现的），pause会被任意信号打断，因此在sleep过程中如果有其他信号到来，会打断当前
阻塞的系统调用

可以使用`nanosleep`或`usleep`替代。sleep不具有移植性

### 高精度计时器 setitimer()

[高精度计时器](./高精度计时器.md)，提供毫秒级精度响应

### 信号集操作函数

- sigemptyset()
- sigfillset()
- sigaddset()
- sigdelset()
- sigismember()
- sigprocmask()
  将某些信号添加到屏蔽字Mask中，阻止或解除阻止
- sigpending()
  似乎不好用

### sigsuspend()与信号驱动程序

sigsuspend与pause类似都是等待一个信号的到来，但是

```c
for(;;) {
    // 阻塞某些信号
    sigprocmask(SIG_BLOCK, &set, &oset);
    dosomething();
    // 恢复之前的mask
    sigprocmask(SIG_SETMASK, &oset, NULL);
    // 等待一个信号到来之后再继续循环，所谓信号驱动
    pause();
}
```

观察以上程序，会发现在dosomething过程中发送一个信号，信号并没有打在pause上，而是在sigprocmask 和 pause之间溜走到了信号处理程序上，究其根本是因为sigprocmask和pause不原子
**sigsuspend**就是为了解决这个问题而存在的

```c
for(;;) {
    // 阻塞某些信号
    sigprocmask(SIG_BLOCK, &set, &oset);
    dosomething();
    // 恢复之前的mask并阻塞并等待一个信号到来
    sigsuspend(&oset);
}
```

### sigaction()与signal()所面对的问题

- sigaction可以避免信号处理函数嵌套导致的重入问题[例子](./mydaemon_normalexit.c)
- sigaction可以获取到有关信号的更加详细的信息，从而可以做进一步的判断。可以看token_bucket_sigaction

### 标准信号和实时信号的区别

- 实时信号不会丢失，有一个实时信号待处理队列
  队列长度有限制，可以通过`ulimit -a`查看，`pending signals`

## 并发-线程

### 线程的概念

一个正在运行的函数

线程间通信比进程间通信要简单

线程是先标准化再实现。一个新的库发布出来默认要求必须支持线程并发

线程有多个标准，POSIX标准，是标准而非实现

pthread_t: POSIX标准下的线程标识，但是这个类型到底是什么取决于具体实现

进程就是容器，用来承载线程

- `pthread_equal()`
  比较两个线程id是否相同

- `pthread_self()`
  返回当前线程的线程标识

### 线程的创建

- `pthread_create`
  创建一个新线程

  ```c
  int pthread_create(pthread_t *thread, const pthread_attr_t *attr,
                    void *(*start_routine) (void *), void *arg);
  /*
    attr: 线程属性，NULL为默认属性
    start_routine: 线程函数的入口点
    arg: 传递给start_routine的参数
  */
  ```

### 线程的终止

- 线程的三种终止方式

  - 线程从启动例程返回，返回值就是线程的退出码
  - 线程可以被同一进程中的其他线程取消
  - 线程调用`pthread_exit()`函数，该函数会进行线程栈的清理

- 收尸
  `pthread_join()`，等待一个指定的线程结束

- 栈清理
  `pthread_cleanup_push()` `pthread_cleanup_pop()`。 两个函数必须成对出现，原因很独特

- 线程的取消选项
  - 线程取消：`pthread_cancel()`
  - 取消有两种状态，允许和不允许。可以通过`pthread_setcancelstate()`指定
  - 允许取消分为`异步cancel`和`推迟cancel`（默认）
    - 异步取消简单直接
    - `pthread_setcanceltype()` 设置取消方式
    - 推迟cancel推迟到取消点，`取消点`由POSIX定义，是可能引发阻塞的系统调用
    - `pthread_testcancel()`设置一个取消点

### 线程同步

- 互斥量

  - `pthread_mutex_t;`
  - `pthread_mutex_init()`
  - `pthread_mutex_destory()`
  - `pthread_mutex_lock()`
  - `pthread_mutex_trylock()`
  - `pthread_mutex_trylock()`
  - `pthread_mutex_unlock()`
  - `pthread_once()` 用于加载一个模块仅一次

- [条件变量](./条件变量.md)

  - `pthread_cond_t`
  - `pthread_cond_init()`
  - `pthread_cond_destory()`
  - `pthread_cond_signal()`
  - `pthread_cond_broadcast()`
  - `pthread_cond_wait()`
  - `pthread_cond_timewait()`

- 信号量
- 读写锁

### 线程属性

pthread_attr函数族

### 线程同步的属性

- 互斥量的属性
  pthread_mutexattr函数族

  - pthread_mutexattr_init()
  - pthread_mutex_attr_destory()
  - pthread_mutexattr_setpshared()
  - pthread_mutexattr_getpshared()
  - clone()
  - pthread_mutexattr_gettype()
  - pthread_mutexattr_settype()

- 条件变量
  - pthread_condattr_init()
  - ...

### 可重入

多线程中的IO处理库函数默认已经支持并发，如果不支持并发会在名字中体现出来：在函数后加一个unlocked后缀

### 线程和信号的关系

实际上，每个线程分别拥有一个mask和pending，进程用有一个pending，在从内核返回用户态反而过程中，扎回哪个线程，哪个线程负责处理所属进程的pending和本线程的penging，也就是要按位与两次

- pthread_sigmask()
- sigwait()
- pthread_kill()

### 线程与fork

POSIX原语规定新的进程只包含调用它的那个线程# 线程的概念

一个正在运行的函数

线程间通信比进程间通信要简单

线程是先标准化再实现。一个新的库发布出来默认要求必须支持线程并发

线程有多个标准，POSIX标准，是标准而非实现

pthread_t: POSIX标准下的线程标识，但是这个类型到底是什么取决于具体实现

进程就是容器，用来承载线程

- `pthread_equal()`
  比较两个线程id是否相同

- `pthread_self()`
  返回当前线程的线程标识

### 线程的创建

- `pthread_create`
  创建一个新线程

  ```c
  int pthread_create(pthread_t *thread, const pthread_attr_t *attr,
                    void *(*start_routine) (void *), void *arg);
  /*
    attr: 线程属性，NULL为默认属性
    start_routine: 线程函数的入口点
    arg: 传递给start_routine的参数
  */
  ```

### 线程的终止

- 线程的三种终止方式

  - 线程从启动例程返回，返回值就是线程的退出码
  - 线程可以被同一进程中的其他线程取消
  - 线程调用`pthread_exit()`函数，该函数会进行线程栈的清理

- 收尸
  `pthread_join()`，等待一个指定的线程结束

- 栈清理
  `pthread_cleanup_push()` `pthread_cleanup_pop()`。 两个函数必须成对出现，原因很独特

- 线程的取消选项
  - 线程取消：`pthread_cancel()`
  - 取消有两种状态，允许和不允许。可以通过`pthread_setcancelstate()`指定
  - 允许取消分为`异步cancel`和`推迟cancel`（默认）
    - 异步取消简单直接
    - `pthread_setcanceltype()` 设置取消方式
    - 推迟cancel推迟到取消点，`取消点`由POSIX定义，是可能引发阻塞的系统调用
    - `pthread_testcancel()`设置一个取消点

### 线程同步

- 互斥量

  - `pthread_mutex_t;`
  - `pthread_mutex_init()`
  - `pthread_mutex_destory()`
  - `pthread_mutex_lock()`
  - `pthread_mutex_trylock()`
  - `pthread_mutex_trylock()`
  - `pthread_mutex_unlock()`
  - `pthread_once()` 用于加载一个模块仅一次

- [条件变量](./条件变量.md)

  - `pthread_cond_t`
  - `pthread_cond_init()`
  - `pthread_cond_destory()`
  - `pthread_cond_signal()`
  - `pthread_cond_broadcast()`
  - `pthread_cond_wait()`
  - `pthread_cond_timewait()`

- 信号量
- 读写锁

### 线程属性

pthread_attr函数族

### 线程同步的属性

- 互斥量的属性
  pthread_mutexattr函数族

  - pthread_mutexattr_init()
  - pthread_mutex_attr_destory()
  - pthread_mutexattr_setpshared()
  - pthread_mutexattr_getpshared()
  - clone()
  - pthread_mutexattr_gettype()
  - pthread_mutexattr_settype()

- 条件变量
  - pthread_condattr_init()
  - ...

### 可重入

多线程中的IO处理库函数默认已经支持并发，如果不支持并发会在名字中体现出来：在函数后加一个unlocked后缀

### 线程和信号的关系

实际上，每个线程分别拥有一个mask和pending，进程用有一个pending，在从内核返回用户态反而过程中，扎回哪个线程，哪个线程负责处理所属进程的pending和本线程的penging，也就是要按位与两次

- pthread_sigmask()
- sigwait()
- pthread_kill()

### 线程与fork

POSIX原语规定新的进程只包含调用它的那个线程

## 进程间通信

### 管道

内核提供、单工、自同步机制
非常灵活简洁，使用广泛

### 匿名管道

具有亲缘关系的进程间通信

- `pipe()`

### 命名管道

- `mkfifo()`

### XSI

> `ipcs`命令可以查看一些信息,关系system V资源

可以用于有亲缘关系和没有亲缘关系的进程间通信

#### 消息队列

#### 共享内存段

- `shmget()`
- `shmat()`
- `shmctl()`

#### 信号量数组

进程的同步与互斥机制，结合POSIX线程提供的互斥量和条件变量来理解

- `semget()`
- `semop()`
- `semctl()`

## 高级IO-非阻塞IO

NONBLOCK选项，如果读写失败会返回EAGAIN，不会阻塞等待

### 有限状态机编程思想FSM

参考mycopy_fsm下的实现，简单来说就是使用状态来驱动程序运行，依赖实现定义好的流程图，适合处理非结构化复杂程序，符合人类直觉

### IO多路转接

实现文件描述符的监视

- select()
  移植性好，古老
- poll()
  和select的思路完全不一样，select以事件为单位组织文件描述符，poll以文件描述符为单位组织事件
  可移植
- epoll()
  Linux方言，和poll思路一致，企图简化用户维护内容

### 其他读写函数

- `readv()`
- `writev()`
  向多个地址读写数据

### 存储映射IO

把某个文件当中的内容映射到进程空间当中

- `mmap()`
- `munmap()`

匿名映射的情况下，mmap和munmap可以替代malloc和free

### 文件锁

- fcntl()
- lockf()
- fclock()

给文件加锁是反映在inode层面而不是结构体层面
如果不同的文件描述符指向同一个文件，向一个文件描述符加锁，另一个close会导致**意外解锁**

## 网络编程

跨主机的传输要注意的问题

- 字节序问题
- 结构体对齐问题，指定不对齐
- 类型长度问题，显示指定类型长度。例如uint32_t
