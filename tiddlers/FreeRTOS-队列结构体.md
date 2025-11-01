`xQueueGenericCreate`函数被用来创建一个通用队列结构体，如下图所示。

这个结构体可以用来实现队列、队列集、信号量、互斥量：

* pcWriteTo、xQueue里的pcReadFrom用来维护唤醒缓冲区：队列/队列集用来读写数据
* xTasksWaitingToSend：用来记录"想发送数据，但是没有空间，因此阻塞"的任务，信号量、互斥量不会用到它
* xTasksWaitingToReceive：用来记录"想读取数据，但是没有数据，因此阻塞"的任务
* uxMessagesWaiting：队列/队列集用来记录有多少个有效数据，信号量/互斥量用来记录数值

![image-20220419091704902](https://mxloss112233.oss-cn-beijing.aliyuncs.com/img/45_generic_queue_type.png)



### 2. 调用关系图

![image-20220419101123044](https://mxloss112233.oss-cn-beijing.aliyuncs.com/img/46_queue_params.png)