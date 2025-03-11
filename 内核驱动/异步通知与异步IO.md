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
# 3.异步IO
Linux中最常用的输入/输出（I/O）模型是同步I/O。在这个模型中，当请求发出之后，应用程序就会阻塞，直到请求满足为止。
但I/O请求可能需要与CPU消耗产生交叠，以充分利用CPU和I/O提高吞吐率。
**异步I/O**，应用程序发起I/O动作后，直接开始执行，并不等待I/O结束，它要么过一段时间来查询之前的I/O请求完成情况，要么I/O请求完成了会自动被调用与I/O完成绑定的回调函数。
![[异步IO时序.png]]
## 3.1glibc中的异步IO
Linux的AIO有多种实现，其中一种实现是在用户空间的glibc库中实现的，它本质上是借用了多线程模型，用开启新的线程以同步的方法来做I/O，新的AIO辅助线程与发起AIO的线程以pthread_cond_signal（）的形式进行线程间的同步。
### 3.1.1aio_read()
```c
int aio_read(struct aiocb *aiocbp);
@brief 请求对一个有效的文件描述符进行异步读操作，这个文件描述符可以使一个文件、套接字、管道
@param aiocb 包含传输所有信息，以及为AIO操作准备的用户空间缓冲区。
@return 在请求进行排队后会立即返回(即使读操作未完成)，成功 0 失败 -1重置错误码
```
### 3.1.2aio_write()
```c
int aio_write(struct aiocb *aiocbp);
@brief 请求一个异步写操作
@return 立即返回，并且它的请求已经被排队 成功0 失败-1 重置错误码
```
### 3.1.3aio_error()
```c
int aio_error(struct aiocb *aiocbp);
@brief 确定请求状态
@return
	EINPROGRESS 请求尚未完成
	ECANCELED   请求被应用程序取消了
	-1          发生错误重置错误码
```
### 3.1.4aio_return()
```c
ssize_t aio_return(struct aiocb *aiocbp);
@brief 只有在aio_error调用确定请求已经完成之后，才会调用这个函数，
@return aio_return()等价于同步情况中read()或write()系统调用的返回值
		如果发生错误，返回值为负数
```
### 3.1.5aio_suspend()
```c
int aio_suspend(const struct aiocb *const cblist[],
			   int n,const struct timespec *timeout);
@brief 阻塞调用进程，直到异步请求完成为止
```
### 3.1.6aio_cancel()
```c
int aio_cancel(int fd, struct aiocb *aiocbp);
@brief 取消对某个文件描述符执行的一个或所有IO请求
@details 要取消一个请求，用户提供文件描述符和aiocb指针
		要取消对某个文件描述符的所有请求，用户需要提供这个文件描述符，并将aiocbp参数设置为NULL
@return 取消一个请求：
			成功取消 AIO_CANCELED
			请求完成 AIO_NOTCANCELED
		取消所有请求：
			所有都取消了 AIO_CANCELED
			至少有一个没有被取消 AIO_NOT_CANCELED
			一个都没取消掉 AIO_ALLDONE
```
### 3.1.7lio_listio()
```c
int lio_listio(int mode, struct aiocb *list[],int nent, struct sigevent *sig);
@brief 同时发起多个传输，用户可以在一个系统调用中启动大量的IO操作
@param mode LIO_WAIT 阻塞调用，直到所有IO都完成
			LIO_NOWAIT 在IO操作进行排队后立即返回
@param list aiocb引用的列表，最大元素nent个，如果为NULL会忽略
@param nent 列表最大元素个数
@param sig 指向 `struct sigevent` 结构体的指针，用于指定当所有 I/O 操作完成后如何通知调用进程（仅在 `mode` 为 `LIO_NOWAIT` 时有效）
```
## 3.2内核AIO与设备驱动
用户空间调用io_submit（）后，对应于用户传递的每一个iocb结构，内核会生成一个与之对应的kiocb结构。file_operations包含3个与AIO相关的成员函数：
```c
ssize_t (*aio_read) (struct kiocb *iocb, const struct iovec *iov, unsigned long nr_segs, loff_t pos);

ssize_t (*aio_write) (struct kiocb *iocb, const struct iovec *iov, unsigned long nr_segs, loff_t pos);

int (*aio_fsync) (struct kiocb *iocb, int datasync);
```
io_submit（）系统调用间接引起了file_operations中的aio_read（）和aio_write（）的调用。
AIO一般由内核空间的通用代码处理，对于块设备和网络设备而言，一般在Linux核心层的代码已经解决。字符设备驱动一般不需要实现AIO支持。Linux内核中对字符设备驱动实现AIO的特例包括drivers/char/mem.c里实现的null、zero等，由于zero这样的虚拟设备其实也不存在在要去读的时候读不到东西的情况，所以aio_read_zero（）本质上也不包含异步操作

