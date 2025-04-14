# 0.准备
## 0.1创建线程
### 0.1.1 静态创建
```c
1.创建线程栈
static rt_uint8_t rt_led1_thread_stack[1024];
2.初始化线程
rt_err_t rt_thread_init(struct rt_thread *thread,
                        const char       *name,
                        void (*entry)(void *parameter),
                        void             *parameter,
                        void             *stack_start,
                        rt_uint32_t       stack_size,
                        rt_uint8_t        priority,
                        rt_uint32_t       tick)；
@brief 初始化线程栈
@param thread 定义的线程栈
@param name 线程名字
@param entry 线程入口函数
@param parameter 线程入口函数参数
@param stack_start 起始栈地址
@param stack_size 栈大小
@param priority 优先级
@param tick 时间片
@return 成功 RT_EOK 失败 -RT_ERROR
3.启动线程
rt_err_t rt_thread_startup(rt_thread_t thread);
@brif 启动线程
@param thread 线程栈
@return 成功 RT_EOK   失败 -RT_ERROR
4.编写线程入口函数
```
### 0.1.2 动态创建
```c
1.创建线程栈指针
static rt_thread_t led2_thread = RT_NULL;
2.初始化线程
rt_thread_t rt_thread_create(const char *name, 
                             void (*entry)(void *parameter),
                             void       *parameter,
                             rt_uint32_t stack_size,
                             rt_uint8_t  priority,
                             rt_uint32_t tick)；
@brif 创建线程栈
@param name 线程名字
@param entry 线程入口函数
@param parameter 线程入口函数参数
@param stack_size 线程栈大小
@param priority 线程优先级
@param tick 线程时间片
@return 成功 线程栈地址  失败  RT_NULL

3.启动线程
rt_err_t rt_thread_startup(rt_thread_t thread);
@brif 启动线程
@param thread 线程栈
@return 成功 RT_EOK   失败 -RT_ERROR
4.编写线程入口函数
```
## 0.2 重映射串口到rt_kprintf
1. 使能`rtconfig.h`下`RT_USING_CONSOLE`宏
2. 重写`board.c`下rt_hw_console_output()函数
```c
/**
 * @brief 重映射串口 DEBUG_USARTx 到 rt_kprintf() 函数
 * Note ： DEBUG_USARTx 是在 bsp_usart.h 中定义的宏，默认使用串口 1
 * @param str ：要输出到串口的字符串
 * @retval 无
 *
 * @attention
 *
 */
void rt_hw_console_output(const char *str)
{
    /* 进入临界段 */
    rt_enter_critical();
    /* 直到字符串结束 */
    while (*str != '\0') {
        /* 换行 */
        if (*str == '\n') {
            USART_SendData(DEBUG_USARTx, '\r');
            while (USART_GetFlagStatus(DEBUG_USARTx, USART_FLAG_TXE) == RESET);
        }
        USART_SendData(DEBUG_USARTx, *str++);
        while (USART_GetFlagStatus(DEBUG_USARTx, USART_FLAG_TXE) == RESET);
    }
    /* 退出临界段 */
    rt_exit_critical();

}
```
## 0.3 常用函数
### 0.3.1 rt_thread_yield
```c
rt_err_t rt_thread_yield(void);
@brief 放弃线程剩余时间片
@return RT_EOK
```
# 1.RT-Thread启动流程
![[rtt启动流程.png]]
## 1.1 start.S汇编文件
系统上电第一个执行汇编文件里的`Reset_Handler`复位函数,复位函数最终调用C库函数`__main`,rt-thread执行完`__main`后跳转到`component.c`文件中的`$Sub$$main`函数，执行完最终通过`$Super$$main`函数跳转到真正的main函数中。
```asm
Reset_Handler PROC
				EXPORT Reset_Handler [WEAK]
				IMPORT SystemInit
				IMPORT __main

				CPSID I ; 关中断
				LDR R0, =0xE000ED08
				LDR R1, =__Vectors
				STR R1, [R0]
				LDR R2, [R1]
				MSR MSP, R2
				LDR R0, =SystemInit
				BLX R0
				CPSIE i ; 开中断
				LDR R0, =__main
				BX R0
				ENDP
```
## 1.2 \$Sub\$\$main函数
```c
int $Sub$$main(void)
{
	/*关闭中断，除了硬 FAULT 和 NMI 可以响应外，其它统统关掉*/
	rt_hw_interrupt_disable();
	rtthread_startup();// 也在componet.c中实现
	return 0;
<<<<<<< HEAD
}[[Git]]
=======
}
>>>>>>> 9ea7dfaec01a3093442d16c908c3a7f470e7bd3e
```
## 1.3rtthread_startup函数
```c
int rtthread_startup(void)
{
	/* 关闭中断 */
	rt_hw_interrupt_disable();

/* 板级硬件初始化
* 注意 : 在板级硬件初始化函数中把要堆初始化好 ( 前提是使用动态内存 )
*/
	rt_hw_board_init();

/* 打印 RT-Thread 版本号 */
	rt_show_version(); // kservice.c中

	/* 定时器初始化 */
	rt_system_timer_init();
	/* 调度器初始化 */
	rt_system_scheduler_init();

#ifdef RT_USING_SIGNALS
	/* 信号量初始化 */
	rt_system_signal_init();
#endif

	/* 创建初始线程 */
	rt_application_init();

	/* 定时器线程初始化 */
	rt_system_timer_thread_init();

	/* 空闲线程初始化 */
	rt_thread_idle_init();

	/* 启动调度器 */
	rt_system_scheduler_start();

	/* 绝对不会回到这里 */
	return 0; (11)
}
```
## 1.4rt_application_init函数
```c
/* 使用动态内存时需要用到的宏：rt_config.h 中定义 *///
#define RT_USING_USER_MAIN
#define RT_MAIN_THREAD_STACK_SIZE 256
#define RT_THREAD_PRIORITY_MAX 32

/* 使用静态内存时需要用到的宏和变量：在 component.c 定义 */ //)
#ifdef RT_USING_USER_MAIN
#ifndef RT_MAIN_THREAD_STACK_SIZE
#define RT_MAIN_THREAD_STACK_SIZE 2048
#endif
#endif

#ifndef RT_USING_HEAP
	ALIGN(8)
	static rt_uint8_t main_stack[RT_MAIN_THREAD_STACK_SIZE];
	struct rt_thread main_thread;
#endif

void rt_application_init(void)
{
	rt_thread_t tid;

#ifdef RT_USING_HEAP
	/* 使用动态内存 */ 
	tid =
	rt_thread_create("main",
					main_thread_entry,
					RT_NULL,
					RT_MAIN_THREAD_STACK_SIZE,
					RT_THREAD_PRIORITY_MAX / 3, (初始线程优先级)
					20);
					RT_ASSERT(tid != RT_NULL);
#else
/* 使用静态内存 */
	rt_err_t result;
	
	tid = &main_thread;
	result =
	rt_thread_init(tid,
				"main",
				main_thread_entry,
				RT_NULL,
				main_stack,
				sizeof(main_stack),
				RT_THREAD_PRIORITY_MAX / 3, (初始线程优先级)
				20);
				RT_ASSERT(result == RT_EOK);
	(void)result;
#endif
	
	/* 启动线程 */
	rt_thread_startup(tid);
}

/* main 线程 */
void main_thread_entry(void *parameter)
{
	extern int main(void);
	extern int $Super$$main(void);
	
	/* RT-Thread 组件初始化 */
	rt_components_init();
	
	/* 调用$Super$$main() 函数，去到 main */
	$Super$$main();
}
```
## 1.5\$Super\$\$main()函数调用main函数
# 2.线程管理
![[线程状态切换.png]]
## 2.1 suspend函数
```c
rt_err_t rt_thread_suspend(rt_thread_t thread);
@brief 挂起线程，如果线程已经被挂起会失败
@param thread 线程栈
@return 成功 RT_EOK 失败 -RT_ERROR
@note 通常不应该使用这个函数来挂起线程本身，如果确实需要采用 rt_thread_suspend（） 函数挂起当前线程，需要在调用rt_thread_suspend()函数后立刻调用 rt_schedule()函数进行手动的线程上下文切换
```
## 2.2resume函数
```c
rt_err_t rt_thread_resume(rt_thread_t thread);
@brief 恢复挂起线程
@param thread 线程栈
@return 成功 RT_EOK 失败 -RT_ERROR
```
# 3.消息队列
* 消息支持先进先出方式排队与优先级排队方式，支持异步读写工作方式
* 读队列支持超时机制
* 支持发送紧急消息，这里的紧急消息是往队列头发送消息
* 可以允许不同长度（不超过队列节点最大值）的任意类型消息
*  一个线程能够从任意一个消息队列接收和发送消息
* 多个线程能够从同一个消息队列接收和发送消息
*  当队列使用结束后，需要通过删除队列操作释放内存函数回收
## 3.1常用函数
### 3.1.1 创建消息队列 rt_mq_create
```c
rt_mq_t rt_mq_create(const char *name,
                     rt_size_t   msg_size,
                     rt_size_t   max_msgs,
                     rt_uint8_t  flag);
@brief 创建消息队列
@param name 消息队列名
@param msg_size 消息最大长度
@param max_msgs 消息对列最大容量
@param flag 队列模式 
			RT_IPC_FLAG_FIFO 先到的线程先获取 
			RT_IPC_FLAG_PRIO 优先级高的线程先获取
@return 成功 消息队列句柄指针 失败 RT_NULL
```
### 3.1.2写队列操作函数 rt_mq_send
```c
rt_err_t rt_mq_send(rt_mq_t mq, const void *buffer, rt_size_t size);
@brief 向消息队列中发送消息 等待消息队列的线程将被唤醒
@param mq 消息队列句柄
@param buffer 消息
@param size 消息长度
@return 错误码
```
### 3.1.3 读队列操作函数 rt_mq_recv
```c
rt_err_t rt_mq_recv(rt_mq_t    mq,
                    void      *buffer,
                    rt_size_t  size,
                    rt_int32_t timeout);
@brief 从消息队列中接收消息，如果消息队列为空，将等待一段时间
@param mq 消息队列句柄
@param buffer 接收消息保存位置
@param size 接收消息的大小
@param timeout 超时时间
				RT_WAITING_FOREVER 一直等
				RT_WAITING_NO 不等
@return 错误码
```
### 3.1.4 删除队列 rt_mq_delete
```c
rt_err_t rt_mq_delete(rt_mq_t mq);
@brief 删除消息队列
@param mq 消息队列句柄指针
@return 错误码
```
# 4.信号量
>互斥或同步时使用，单值信号量0、1，计数信号量0~65535
>释放信号量将信号量加1
>获取信号量将信号量减1
## 4.1常用信号量函数
### 4.1.1 信号量创建函数 rt_sem_create
```c
rt_sem_t rt_sem_create(const char *name, rt_uint32_t value, rt_uint8_t flag);
@brief 创建信号量
@param name 信号量名字
@param value 信号量初始值
@param flag 信号量标志
			RT_IPC_FLAG_FIFO 先到的线程先获取 
			RT_IPC_FLAG_PRIO 优先级高的线程先获取
@return 成功 信号量操作句柄 失败 RT_NULL
@see rt_sem_init
```
### 4.1.2 信号量删除函数 rt_sem_delete
```c
rt_err_t rt_sem_delete(rt_sem_t sem)；
@brief 删除信号量并释放内存
@param sem 信号量操作句柄
@return 错误码
@see rt_sem_detach
```
### 4.1.3信号量释放函数 rt_sem_release
```c
rt_err_t rt_sem_release(rt_sem_t sem);
@brief 释放信号量，如果有等待信号量的线程休眠将被唤醒
@param sem 信号量操作句柄
@return 错误码
```
### 4.1.4 信号量获取函数 rt_sem_take
```c
rt_err_t rt_sem_take(rt_sem_t sem, rt_int32_t time);
@brief 获取信号量，如果不可获取，将等待一段时间
@param sem 信号量操作句柄
@param time 等待时间
		RT_WAITING_FOREVER 一直等
		RT_WAITING_NO 不等
@return 错误码
```
# 5.互斥量
 * 线程可能会多次获取互斥量的情况下。这样可以避免同一线程多次递归持有而造成死锁的问题；
* 可能会引起优先级翻转的情况；
>持有互斥量 值加1
>释放互斥量 值减1
## 5.1信号量函数接口
### 5.1.1互斥量创建函数 rt_mutex_create
```c
rt_mutex_t rt_mutex_create(const char *name, rt_uint8_t flag);
@brief 创建互斥量
@param name 互斥量名字
@param flag 互斥量标志
			RT_IPC_FLAG_FIFO 先到的线程先获取 
			RT_IPC_FLAG_PRIO 优先级高的线程先获取
@return 成功 互斥量句柄 失败 RT_NULL
@see rt_mutex_init
```
### 5.1.2 互斥量删除函数 rt_mutex_delete
```c
rt_err_t rt_mutex_delete(rt_mutex_t mutex);
@brief 删除互斥量并释放内存
@param mutex 互斥量句柄
@return 错误码
@see rt_mutex_detach
```
### 5.1.3互斥量释放函数 rt_mutex_release
```c
rt_err_t rt_mutex_release(rt_mutex_t mutex);
@brief 释放互斥量，等待互斥量被挂起的线程将被唤醒
@param mutex 互斥量句柄
@return 错误码
```
### 5.1.4互斥量获取函数 rt_mutex_take
```c
rt_err_t rt_mutex_take(rt_mutex_t mutex, rt_int32_t time);
@brief 获取互斥量，如果不可获取，将等待一段时间
@param mutex 互斥量句柄
@param time 超时时间
				RT_WAITING_FOREVER 一直等
				RT_WAITING_NO 不等
@return 错误码
```
## 5.2 使用注意
 1. 两个线程不能对同时持有同一个互斥量。如果某线程对已被持有的互斥量进行获取，则该线程会被挂起，直到持有该互斥量的线程将互斥量释放成功，其他线程才能申请这个互斥量。
 2. 互斥量不能在中断服务程序中使用。
 3. RT-Thread 作为实时操作系统需要保证线程调度的实时性，尽量避免线程的长时间阻塞，因此在获得互斥量之后，应该尽快释放互斥量。
4. 持有互斥量的过程中，不得再调用 rt_thread_control() 等函数接口更改持有互斥量线程的优先级
# 6.事件
 * 事件只与线程相关联，事件相互独立，一个 32 位的事件集合（set 变量），用于标识该线程发生的事件类型，其中每一位表示一种事件类型（0 表示该事件类型未发生、1 表示该事件类型已经发生），一共 32 种事件类型。
* 事件仅用于同步，不提供数据传输功能。
* 事件无排队性，即多次向线程发送同一事件 (如果线程还未来得及读走)，等效于只发送一次。
*  允许多个线程对同一事件进行读写操作。
*  支持事件等待超时机制。
## 6.1事件函数接口
### 6.1.1事件创建函数 rt_event_create
```c
rt_event_t rt_event_create(const char *name, rt_uint8_t flag);
@brief 创建事件
@param name 事件名字
@param flag 事件标志
@return 成功 事件句柄指针 失败 RT_NULL
```
### 6.1.2事件删除函数 rt_event_delete
```c
rt_err_t rt_event_delete(rt_event_t event);
@brief 删除事件并释放资源
@param event 事件句柄
@return 错误码
```
### 6.1.3事件发送函数 rt_event_send
```c
rt_err_t rt_event_send(rt_event_t event, rt_uint32_t set);
@brief 发送事件，如果线程等待该事件处于挂起，该线程会唤醒
@param event 事件句柄
@param set 设置的事件
@return 错误码
```
### 6.1.4事件接受函数 rt_event_recv
```c
rt_err_t rt_event_recv(rt_event_t   event,
                       rt_uint32_t  set,
                       rt_uint8_t   option,
                       rt_int32_t   timeout,
                       rt_uint32_t *recved);
@brief 从事件目标中接收事件，如果没有接收到，则会等待一会
@param event 事件句柄
@param set 感兴趣的事件
@param option 接收选项 可用|连接
			RT_EVENT_FLAG_AND     感兴趣的事件全部发生
	        RT_EVENT_FLAG_OR      感兴趣的事件有一个发生
	        RT_EVENT_FLAG_CLEAR   清除标志位
@param recved 接收的事件，如果不关心设置 RT_NULL 将不被设置
@return 错误码
```
# 7.软件定时器
*  `rt_tick`，它是一个 32 位无符号的变量，用于记录当前系统经过的 tick 时间，当硬件定时器中断来临时，它将自动增加 1。
* 软件定时器列表 `rt_soft_timer_list`。系统新创建并激活的定时器都会以超时时间升序的方式插入到 rt_soft_timer_list 列表中。系统在定时器线程中扫描 rt_soft_timer_list 中的第一个定时器，看是否已超时，若已经超时了则调用软件定时器超时函数。否则出软件定时器线程，因为定时时间是升序插入软件定时器列表的，列表中第一个定时器的定时时间都还没到的话，那后面的定时器定时时间自然没到。
## 7.1软件定时器注意
* 软件定时器的超时函数中应快进快出，绝对不允许使用任何可能引软件定时器起线程挂起或者阻塞的 API 接口，在超时函数中也绝对不允许出现死循环。
* 软件定时器使用了系统的一个队列和一个线程资源，软件定时器线程的优先级默认为RT_TIMER_THREAD_PRIO。
* 创建单次软件定时器，该定时器超时执行完超时函数后，系统会自动删除该软件定时器，并回收资源。
* 定时器线程的堆栈大小默认为 RT_TIMER_THREAD_STACK_SIZE，512 个字节。
## 7.2软件定时器函数 rt_timer_create
```c
rt_timer_t rt_timer_create(const char *name,
                           void (*timeout)(void *parameter),
                           void       *parameter,
                           rt_tick_t   time,
                           rt_uint8_t  flag);
@brief 创建定时器
@param name 定时器名字
@param timeout 定时器超时函数
@param parameter 定时器超时函数参数
@param time 定时器的tick
@param flag 定时器标志
				RT_TIMER_FLAG_DEACTIVATED  计时器是无效的
				RT_TIMER_FLAG_ACTIVATED  定时器是可用的
				RT_TIMER_FLAG_ONE_SHOT  单次定时器
				RT_TIMER_FLAG_PERIODIC  周期定时器

				RT_TIMER_FLAG_HARD_TIMER  硬定时器，定时器的超时函数将在tick isr 中调用
				RT_TIMER_FLAG_SOFT_TIMER 软定时器，定时器的超时函数将在定时器线程中调用
@return 创建定时器控制块句柄
```
# 8.邮箱
- 邮件支持先进先出方式排队与优先级排队方式，支持异步读写工作方式。
- 发送与接收邮件均支持超时机制。
- 一个线程能够从任意一个消息队列接收和发送邮件。
- 多个线程能够向同一个邮箱发送邮件和从中接收邮件。
- 邮箱中的每一封邮件只能容纳固定的 4 字节内容（可以存放地址）。
- 当队列使用结束后，需要通过删除邮箱以释放内存。
## 8.1邮箱的函数接口
### 8.1.1 邮箱创建函数 rt_mb_create
```c
rt_mailbox_t rt_mb_create(const char *name, rt_size_t size, rt_uint8_t flag);
@brief 创建邮箱
@param name 邮箱名字
@param size 邮箱大小
@param flag 邮箱标志
			RT_IPC_FLAG_FIFO 先到的线程先获取 
			RT_IPC_FLAG_PRIO 优先级高的线程先获取
@return 成功 邮箱句柄 失败 RT_NULL
```
### 8.1.2 邮箱删除函数 rt_mb_delete
```c
rt_err_t rt_mb_delete(rt_mailbox_t mb);
@brief 删除邮箱并释放资源
@param mb 邮箱操作句柄
@return 错误码
```
### 8.1.3 邮箱邮件发送函数 rt_mb_send_wait (阻塞）
```c
rt_err_t rt_mb_send_wait(rt_mailbox_t mb, rt_ubase_t value, rt_int32_t timeout);
@brief 发送邮件给邮箱 如果邮箱满了将挂起 直到超时
@param mb 邮箱操作句柄
@param value 邮件
@param timeout 等待时间
@return 错误码
```
### 8.1.4邮箱邮件发送函数 rt_mb_send （非阻塞）
```c
rt_err_t rt_mb_send(rt_mailbox_t mb, rt_ubase_t value)；
@brief 发送邮件给邮箱，如果等待邮件线程休眠将被唤醒，立即返回，如果想发送给指定线程使用 rt_mb_send_wait instead
@param mb 邮箱操作句柄
@param value 邮件
@return 错误码
```
### 8.1.5 邮箱邮件接收函数 rt_mb_recv
```c
rt_err_t rt_mb_recv(rt_mailbox_t mb, rt_ubase_t *value, rt_int32_t timeout)；
@brief 从邮箱中接收邮件，如果邮箱为空，则等待一段时间
@param mb 邮箱操作句柄
@param value 接受的邮件保存位置
@param timeout 等待时间
@return 错误码
```
# 9.内存管理
- 静态内存一旦创建就指定了内存块的大小，分配只能以内存块大小粒度进行分配
- 动态内存分配则根据运行时环境确定需要的内存块大小，按照需要分配内存
## 9.1 静态内存管理的函数接口
### 9.1.1静态内存创建函数 rt_mp_create
```c
rt_mp_t rt_mp_create(const char *name,
                     rt_size_t   block_count,
                     rt_size_t   block_size);
@brief 创建内存池，并从堆区申请空间
@param name 内存池名字
@param block_count 内存池中块的数量
@param block_size 每一块大小
@return 内存池控制块
```
### 9.1.2 静态内存删除函数 rt_mp_delete
```c
rt_err_t rt_mp_delete(rt_mp_t mp);
@brief 删除内存池，释放空间
@param mp 内存池控制块
@return RT_EOK
```
### 9.1.3静态内存申请函数 rt_mp_alloc
```c
void *rt_mp_alloc(rt_mp_t mp, rt_int32_t time)；
@brief 从内存池中申请一块
@param mp 内存池控制块句柄
@param time 等待时间
@return 成功 申请的内存块 失败 RT_NULL
```
### 9.1.4静态内存释放函数 rt_mp_free
```c
void rt_mp_free(void *block);
@brief 释放内存块
@param block 释放内存块的地址
```
## 9.2 动态内存管理的函数接口
### 9.2.1系统堆内存申请函数 rt_malloc
```c
void *rt_malloc(rt_size_t size);
@brief 动态申请size大小的内存
@param size 内存块大小
@return 成功 内存块地址 失败 NULL
```
### 9.2.2系统堆内存释放函数 rt_free
```c
void rt_free(void *rmem);
@brief 释放动态申请的内存回到堆区
@param rmem 被释放内存块的地址
```
<<<<<<< HEAD
# 10.链表
## 10.1 双向链表的函数接口
### 10.1.1 链表初始化函数 rt_list_init
```c
rt_inline void rt_list_init(rt_list_t *l);
@brief 初始化链表
@param l 要初始化的链表
```
### 10.1.2 向链表指定节点后面插入节点 rt_list_insert_after
```c
rt_inline void rt_list_insert_after(rt_list_t *l, rt_list_t *n);
@brief 向链表中指定节点后边插入节点
@param l 指定节点
@param n 插入的节点
```
### 10.1.3 向链表指定节点前面插入节点 rt_list_insert_before
```c
rt_inline void rt_list_insert_before(rt_list_t *l, rt_list_t *n);
@brief 向链表指定节点前边插入节点
@param l 指定节点
@param n 插入的节点
```
### 10.1.4 从链表删除节点函数 rt_list_remove
```c
rt_inline void rt_list_remove(rt_list_t *n);
@brief 从链表中删除节点
@param n 要删除的节点
```

=======
>>>>>>> 9ea7dfaec01a3093442d16c908c3a7f470e7bd3e
