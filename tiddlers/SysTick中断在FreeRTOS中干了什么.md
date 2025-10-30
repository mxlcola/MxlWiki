# 📦 SysTick 中断服务函数干了什么

在 FreeRTOS 端口层里，中断服务函数一般命名为：

```c
void xPortSysTickHandler(void)
```

它的核心逻辑大致是：

```c
void xPortSysTickHandler(void)
{
    /* 通知内核系统时间前进一tick */
    if (xTaskIncrementTick() != pdFALSE)
    {
        /* 若有更高优先级任务就绪 → 请求一次上下文切换 */
        portYIELD_FROM_ISR(pdTRUE);
        // 实际上是设置 PendSV 挂起标志
    }
}
```

即：

1. 增加系统节拍计数；
2. 检查是否有任务时间到期（例如 vTaskDelay() 到时）；
3. 若有更高优先级任务准备就绪，则触发 [[PendSV中断]]进行任务切换。

---
