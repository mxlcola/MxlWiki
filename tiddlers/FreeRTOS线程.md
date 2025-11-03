关联：

[[FreeRTOS任务]]

## 创建线程

## xTaskCreate源码分析

它主要做4件事：

* 分配TCB结构体
* 分配栈
* [[伪造现场|FreeRTOS-伪造现场]]：构造栈的内容
* 把TCB放入就绪链表



## xTaskCreateStatic源码分析

它主要做2件事：

* 伪造现场：构造栈的内容
* 把TCB放入就绪链表

