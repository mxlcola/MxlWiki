> **[[PendSV中断]]（`vPortPendSVHandler()`）** 完成的。

[[vPortPendSVHandler]]
* **SysTick** 或其它中断触发调度请求 → 设置 PendSV 挂起；
* CPU 退出中断后执行 PendSV；
* `vPortPendSVHandler()` 保存当前任务的上下文、切换栈指针、恢复下一个任务；
* 异常返回后 CPU 自动恢复下一个任务现场。