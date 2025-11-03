> **[[PendSV中断]]（`vPortPendSVHandler()`）** 完成的。

## PendSV中断作用

[img width = 600 [https://mxloss112233.oss-cn-beijing.aliyuncs.com/img/20251103200000.png]]

## 实现代码

[[vPortPendSVHandler函数]]


* **SysTick** 或其它中断触发调度请求 → 设置 PendSV 挂起；
* CPU 退出中断后执行 PendSV；
* `vPortPendSVHandler()` 保存当前任务的上下文、切换栈指针、恢复下一个任务；
* 异常返回后 CPU 自动恢复下一个任务现场。