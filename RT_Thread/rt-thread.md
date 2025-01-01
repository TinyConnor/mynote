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