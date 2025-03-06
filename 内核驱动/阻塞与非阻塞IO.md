阻塞操作是指在执行设备操作时，若不能获得资源，则挂起进程直到满足可操作的条件后再进行操作。被挂起的进程进入睡眠状态，被从调度器的运行队列移走，直到等待的条件被满足。而非阻塞操作的进程在不能进行设备操作时，并不挂起，它要么放弃，要么不停地查询，直至可以进行操作为止。
# 1.等待队列
## 1.1定义等待队列头
```c
wait_queue_head_t my_queue;
```
## 1.2初始化等待队列头
```c
init_waitqueue_head(&my_queue);

DECLARE_WAIT_QUEUE_HEAD(name);/*定义并初始化等待队列头*/
```
## 1.3定义等待队列元素
```c
DECLARE_WAITQUEUE(name,tsk);
```
## 1.4添加/移除等待队列
```c
void add_wait_queue(wait_queue_head_t *q,wait_queue_t *wait);
void remove_wait_queue(wait_queue_head_t *q, wait_queue_t *wait);
```
## 1.5等待事件
```c
wait_event(queue, condition);
wait_event_interruptible(queue,condition);
wait_event_timeout(queue, condition, timeout);
wait_event_interruptible_timeout(queue, condition, timeout);
```
## 1.6唤醒队列
```c
void wake_up(wait_queue_head_t *queue);
void wake_up_interruptible(wait_queue_head_t *queue);
```
## 1.7在等待队列上休眠
```c
sleep_on(wait_queue_head_t *q);
interruptible_sleep_on(wait_queue_head_t *q);
```
# 2.轮询操作
使用xxx_poll(),应用层调用select(),poll(),epoll()本质都是调用xxx_poll().
poll函数原型：
```c
unsigned int(*poll)(struct file * filp, struct poll_table* wait);
```
这个函数应该进行两项工作:
1. 对可能引起设备文件状态变化的等待队列调用poll_wait（）函数，将对应的等待队列头部添加到poll_table中。
2. 返回表示是否能对设备进行无阻塞读、写访问的掩码。
用于向poll_table注册等待队列的关键poll_wait（）函数的原型如下：
```c
void poll_wait(struct file *filp, wait_queue_heat_t *queue, poll_table * wait);
```
poll_wait（）函数所做的工作是把当前进程添加到wait参数指定的等待列表（poll_table）中，实际作用是让唤醒参数queue对应的等待队列可以唤醒因select（）而睡眠的进程。
