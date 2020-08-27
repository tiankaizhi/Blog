## 摘要

本文首先介绍了信号的本质及信号的来源，然后从不同角度阐述了信号的种类，接着对信号的生命周期做了详细的介绍。通过对这些内容的学习，可以加深对信号的理解。

## 信号本质

信号是在软件层次上对 OS 中断机制的一种模拟。在原理上，一个进程收到一个信号与处理器收到一个中断请求是类似的。信号是异步的，一个进程不必通过任何操作来等待信号的到达，事实上，进程也不知道信号到底什么时候到达。

## 信号来源

信号事件的发生有两个来源：硬件来源（如键盘或硬件故障）；软件来源，最常用发送信号的系统函数是 ```kill()```、```raise()```、```sigqueue()```、```alarm()```、```setitimer()``` 以及 ```abort()``` 等函数，软件来源还包括一些非法运算等操作。

## 信号种类

信号的分类大概可以从可靠性方面和实时性方面分为两类，即可靠，不可靠信号；实时，非实时信号。

### 可靠性方面

#### 不可靠信号

Linux 信号机制基本上是从 Unix 系统中继承过来的。早期 Unix 系统中的信号机制比较简单和原始，后来在实践中暴露出一些问题，因此，把那些建立在早期机制上的信号叫做"不可靠信号"，信号值小于 ```SIGRTMIN``` 的信号都是不可靠信号。这就是"不可靠信号"的来源。不可靠信号主要存在两个问题：

1. 进程每次处理信号后，就将对信号的响应设置为默认动作。在某些情况下，将导致对信号的错误处理；因此，用户如果不希望这样的操作，那么就要在信号处理函数结尾再一次调用 ```signal()```，重新安装该信号。

2. 信号可能丢失。

Linux 支持不可靠信号，但是对不可靠信号机制做了改进：在调用完信号处理函数后，不必重新调用该信号的安装函数（信号安装函数是在可靠机制上的实现）。因此，**Linux 下的不可靠信号问题主要指的是信号可能丢失**。

#### 可靠信号

随着时间的发展，后来出现的各种 Unix 版本力图实现"可靠信号"。由于原来定义的信号已有许多应用，不好再做改动，最终只好又新增加了一些信号，并在一开始就把它们定义为可靠信号，**可靠信号的信号值值位于 ```SIGRTMIN``` 和 ```SIGRTMAX``` 之间，这些信号支持排队，不会丢失**。

> 注意，可靠信号是信号值位于 ```SIGRTMIN``` 及 ```SIGRTMAX``` 之间的信号；不可靠信号是信号值小于 SIGRTMIN 的信号。信号的可靠与不可靠只与信号值有关，与其他无关。

### 实时信号与非实时信号

早期 Unix 系统只定义了 32 种信号，Red Hat 7.8 支持 64 种信号。```SIGRTMIN``` 之前（含 ```SIGRTMIN```）的信号已经有了预定义值，每个信号有了确定的用途及含义，并且每种信号都有各自的缺省动作。如按键盘的 CTRL C 时，会产生 ```SIGINT``` 信号，对该信号的默认反应就是进程终止。```SIGRTMIN```之后的信号表示实时信号，等同于前面阐述的可靠信号。

```
[root@centos7 /]# cat /etc/redhat-release
CentOS Linux release 7.8.2003 (Core)
[root@centos7 /]# kill -l
 1) SIGHUP       2) SIGINT       3) SIGQUIT      4) SIGILL       5) SIGTRAP
 6) SIGABRT      7) SIGBUS       8) SIGFPE       9) SIGKILL     10) SIGUSR1
11) SIGSEGV     12) SIGUSR2     13) SIGPIPE     14) SIGALRM     15) SIGTERM
16) SIGSTKFLT   17) SIGCHLD     18) SIGCONT     19) SIGSTOP     20) SIGTSTP
21) SIGTTIN     22) SIGTTOU     23) SIGURG      24) SIGXCPU     25) SIGXFSZ
26) SIGVTALRM   27) SIGPROF     28) SIGWINCH    29) SIGIO       30) SIGPWR
31) SIGSYS      34) SIGRTMIN    35) SIGRTMIN+1  36) SIGRTMIN+2  37) SIGRTMIN+3
38) SIGRTMIN+4  39) SIGRTMIN+5  40) SIGRTMIN+6  41) SIGRTMIN+7  42) SIGRTMIN+8
43) SIGRTMIN+9  44) SIGRTMIN+10 45) SIGRTMIN+11 46) SIGRTMIN+12 47) SIGRTMIN+13
48) SIGRTMIN+14 49) SIGRTMIN+15 50) SIGRTMAX-14 51) SIGRTMAX-13 52) SIGRTMAX-12
53) SIGRTMAX-11 54) SIGRTMAX-10 55) SIGRTMAX-9  56) SIGRTMAX-8  57) SIGRTMAX-7
58) SIGRTMAX-6  59) SIGRTMAX-5  60) SIGRTMAX-4  61) SIGRTMAX-3  62) SIGRTMAX-2
63) SIGRTMAX-1  64) SIGRTMAX
```

## 信号发送

发送信号的函数主要有：```kill()```、```raise()```、```sigqueue()```、```alarm()```、```setitimer()``` 以及 ```abort()```。

### kill()

该系统调用可以用来向任何进程或进程组发送任何信号。参数 ```pid``` 为信号的接收进程 id；```signo``` 为 0 时（即空信号），实际不发送任何信号，但照常进行错误检查，因此，可用于检查目标进程是否存在，以及当前进程是否具有向目标发送信号的权限（root 权限的进程可以向任何进程发送信号，非 root 权限的进程只能向属于同一个 session 或者同一个用户的进程发送信号）。

```
#include <sys/types.h>

#include <signal.h>

int kill(pid_t pid,int signo)
```

### sigqueue()

1 ```sigqueue()``` 是比较新的发送信号系统调用，主要是针对实时信号提出的（当然也支持非实时的信号），支持信号带有参数，与函数 ```sigaction()``` 配合使用。第一个参数 ```pid``` 是指定接收信号的进程 I D，第二个参数确定即将发送的信号，第三个参数是一个联合数据结构 ```union sigval```，指定了信号传递的参数，即通常所说的 4 字节值。

```
#include <sys/types.h>

#include <signal.h>

int sigqueue(pid_t pid, int sig, const union sigval val)
```

```
typedef union sigval {
    int  sival_int;
    void *sival_ptr;
}sigval_t;
```

1 ```sigqueue()``` 比 ```kill()``` 传递了更多的附加信息，但 ```sigqueue()``` 只能向一个进程发送信号，而不能发送信号给一个进程组。如果 ```signo=0```，将会执行错误检查，但实际上不发送任何信号，0 值信号可用于检查 pid 的有效性以及当前进程是否有权限向目标进程发送信号。

在调用 ```sigqueue()``` 时，```sigval_t``` 指定的信息会拷贝到对应 ```sig``` 注册的 3 参数信号处理函数的 ```siginfo_t``` 结构中，这样信号处理函数就可以处理这些信息了。由于 ```sigqueue()``` 系统调用支持发送带参数信号，所以比 ```kill()``` 系统调用的功能要灵活和强大得多。

> 注：```sigqueue()``` 发送非实时信号时，第三个参数包含的信息仍然能够传递给信号处理函数；```sigqueue()``` 发送非实时信号时，仍然不支持排队。

### raise()

向进程本身发送信号，```signo``` 参数为即将发送的信号值。调用成功返回 0；否则，返回 -1。

```
#include <signal.h>

int raise(int signo)
```

### alarm()

```
#include <unistd.h>
unsigned int alarm(unsigned int seconds)
```

专门为 ```SIGALRM``` 信号而设，在指定的时间 seconds 秒后，将向进程本身发送 ```SIGALRM``` 信号，又称为闹钟信号。进程调用 ```alarm()```后，任何以前的 ```alarm()``` 调用都将无效。如果参数 seconds 为零，那么进程内将不再包含任何闹钟时间。如果调用 ```alarm()```，进程中已经设置了闹钟时间，则返回上一个闹钟时间的剩余时间，否则返回 0。

### setitimer()

```
#include <sys/time.h>
int setitimer(int which, const struct itimerval *value, struct itimerval *ovalue);
```

```setitimer()``` 比 ```alarm()``` 功能强大，支持 3 种类型的定时器：

1. ```ITIMER_REAL```：设定绝对时间，经过指定的时间后，内核将发送 ```SIGALRM``` 信号给本进程；

2. ```ITIMER_VIRTUAL```：设定程序执行时间，经过指定的时间后，内核将发送 ```SIGVTALRM``` 信号给本进程；

3. ```ITIMER_PROF```：设定进程执行以及内核因本进程而消耗的时间和，经过指定的时间后，内核将发送 ```ITIMER_VIRTUAL``` 信号给本进程；

```setitimer()``` 第一个参数 ```which``` 指定定时器类型（上面三种之一）；第二个参数是结构 ```itimerval``` 的一个实例，结构 ```itimerval``` 形式见附录1。第三个参数可不做处理。

```setitimer()``` 调用成功返回 0，否则返回 -1。

### bort()

```
#include <stdlib.h>
void abort(void);
```

向进程发送 ```SIGABORT``` 信号，默认情况下进程会异常退出，当然可定义自己的信号处理函数。即使 ```SIGABORT``` 被进程设置为阻塞信号，调用 ```abort()``` 后，```SIGABORT``` 仍然能被进程接收。该函数无返回值。


## 进程对信号的响应

进程可以通过三种方式来响应一个信号：

1. 忽略信号，即对信号不做任何处理，其中，有两个信号不能忽略：```SIGKILL``` 和 ```SIGSTOP```；

2. 捕捉信号。定义信号处理函数，当信号发生时，执行相应的处理函数；

3. 执行缺省操作，Linux 对每种信号都规定了默认操作，参考下图。**注意，进程对实时信号的缺省反应是进程终止**。

![](assets/markdown-img-paste-2020082710452078.png)

Linux 究竟采用上述三种方式的哪一个来响应信号，取决于传递给相应 API 函数的参数。
