异步通知的意思是：一旦设备就绪，则主动通知应用程序，这样应用程序根本就不需要查询设备状态，这一点非常类似于硬件上“中断”的概念，比较准确的称谓是“信号驱动的异步I/O” 。
# 1.阻塞、结合轮询的非阻塞I/O和异步通知的区别
阻塞I/O意味着一直等待设备可访问后再访问，非阻塞I/O中使用poll（）意味着查询设备是否可访问，而异步通知则意味着设备通知用户自身可访问，之后用户再进行I/O处理。由此可见，这几种I/O方式可以相互补充。
![[阻塞、结合轮询的非阻塞IO和异步通知的区别.png]]
# 2.异步通知
## 2.1Linux信号
![[Linux信号及定义.png]]
除了SIGSTOP和SIGKILL两个信号外，进程能够忽略或捕获其他的全部信号。一个信号被捕获的意思是当一个信号到达时有相应的代码处理它。如果一个信号没有被这个进程所捕获，内核将采用默认行为处理。
## 2.2信号的接收
使用signal()函数设置对应信号处理函数
```c
void (*signal(int signum,void (*handler))(int))(int);
```
该函数可分解为：
```c
typedef void (*sighandler_t)(int);
sighandler_t signal(int signum, sighandler_t handler);
```
第一个参数指定信号的值，第二个参数指定针对前面信号值的处理函数，若为SIG_IGN，表示忽略该信号；若为SIG_DFL，表示采用系统默认方式处理信号；若为用户自定义的函数，则信号被捕获到后，该函数将被执行。
除了signal（）函数外，sigaction（）函数可用于改变进程接收到特定信号后的行为，它的原型为：
```c
/**
* @brief 如果把第二、第三个参数都设为NULL，那么该函数可用于检查信号的有效性。
* @param signum 信号的值，可以是除SIGKILL及SIGSTOP外的任何一个特定有效的信号
* @param act sigaction的一个实例的指针，在结构体sigaction的实例中，指定了对特定信号的处理函数，若为空，则进程会以缺省方式对信号处理的处理函数，可指定oldact为NULL
*/
int sigaction(int signum, const struct sigaction *act, struct sigaction *oldact);
```
为了能在用户空间中处理一个设备释放的信号，它必须完成3项工作：
1. 通过F_SETOWN IO控制命令设置设备文件的拥有者为本进程，这样从设备驱动发出的信号才能被本进程接收到。
2. 通过F_SETFL IO控制命令设置设备文件以支持FASYNC，即异步通知模式。
3. 通过signal（）函数连接信号和信号处理函数。
## 2.3信号的释放
为了使设备支持异步通知机制，驱动程序中涉及3项工作。
1. 支持F_SETOWN命令，能在这个控制命令处理中设置filp->f_owner为对应进程ID。不过此项工作已由内核完成，设备驱动无须处理。
2. 支持F_SETFL命令的处理，每当FASYNC标志改变时，驱动程序中的fasync（）函数将得以执行。因此，驱动中应该实现fasync（）函数。
3. 在设备资源可获得时，调用kill_fasync（）函数激发相应的信号。
![[异步通知中设备驱动和异步通知的交互.png]]
处理FASYNC标志变更的函数
```c
int fsdync_helper(int fd, struct file *filp, int mode, struct fasync_struct **fa);
```
释放信号用的函数
```c
void kill_fasync(struct fasync_struct **fa, int sig, int band);
```