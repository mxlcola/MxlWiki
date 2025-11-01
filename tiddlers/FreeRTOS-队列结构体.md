`xQueueGenericCreate`函数被用来创建一个通用队列结构体，如下图所示。

这个结构体可以用来实现队列、队列集、信号量、互斥量：

```c
typedef struct QueueDefinition
{
    int8_t *pcHead;                   /* 队列缓冲区起始地址 */
    int8_t *pcTail;                   /* 队列缓冲区结束地址（最后一个字节后的位置） */
    int8_t *pcWriteTo;                /* 下一个写入位置 */
    union
    {
        QueuePointers_t xQueue;       /* 普通队列使用的指针 */
        SemaphoreData_t xSemaphore;   /* 互斥量或信号量使用的数据 */
    } u;

    List_t xTasksWaitingToSend;       /* 等待发送的任务列表 */
    List_t xTasksWaitingToReceive;    /* 等待接收的任务列表 */

    volatile UBaseType_t uxMessagesWaiting; /* 队列中已有消息数 */
    UBaseType_t uxLength;             /* 队列可容纳的最大消息数 */
    UBaseType_t uxItemSize;           /* 每个消息大小（字节） */

    volatile int8_t cRxLock;          /* 接收锁计数（防止嵌套唤醒） */
    volatile int8_t cTxLock;          /* 发送锁计数（防止嵌套唤醒） */

#if ( configUSE_TRACE_FACILITY == 1 )
    UBaseType_t uxQueueNumber;        /* 追踪用（非关键） */
    uint8_t ucQueueType;
#endif

} xQUEUE;
typedef xQUEUE Queue_t;

```

![image-20220419091704902](https://mxloss112233.oss-cn-beijing.aliyuncs.com/img/45_generic_queue_type.png)



### 2. 调用关系图

![image-20220419101123044](https://mxloss112233.oss-cn-beijing.aliyuncs.com/img/46_queue_params.png)