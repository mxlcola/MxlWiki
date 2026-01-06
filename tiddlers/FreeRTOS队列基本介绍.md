## FreeRTOS队列基本介绍

- FreeRTOS 的“队列”是一个通用内核对象：**[[消息队列]]、计数[[信号量]]、二值信号量、[[互斥量]]（带[[优先级继承]]）、递归互斥量**都用同一套底层实现（主要在 `queue.c`），只是**长度/元素大小/特殊分支**不同。
- 队列内部用**环形缓冲区**（ring buffer）存放元素，配合**两个等待链表**管理阻塞任务：

队列(Queue)、队列集(Queue Set)、信号量(Semaphore)、互斥量(Mutex)、递归互斥量，这5种机制的核心都是"通用队列"：

![image-20220419093955498](https://mxloss112233.oss-cn-beijing.aliyuncs.com/img/44_queuegenericreate.png)
