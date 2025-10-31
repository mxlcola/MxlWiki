在 FreeRTOS 的 Cortex-M 端口中，[[SVC中断]] 只负责“启动第一个任务”。
它不会参与后续任务切换，后续调度都是由 [[PendSV中断]]完成的


## 🧩 一、`vPortSVCHandler()` 的职责 —— 启动第一个任务

在调用 `vTaskStartScheduler()` 时，最终会执行：

```
xPortStartScheduler()
 └── vPortStartFirstTask()
      └── SVC 0
```

`SVC 0` 指令触发 **SVC 异常**，进入 `vPortSVCHandler()`。
这个 SVC 异常只执行一次，用来：

1. 从全局变量 `pxCurrentTCB` 取出当前任务（第一个任务）的栈指针；
2. 恢复该任务保存的上下文（R4–R11 + PSP）；
3. 异常返回，让硬件自动弹出剩余寄存器（R0–R3、R12、LR、PC、xPSR）；
4. CPU “落地”到第一个任务的入口函数，开始运行。

之后调度器开始工作，**SVC Handler 不再被调用**（除非用户自己用 `SVC` 指令触发别的服务）。

---

## ⚙️ 二、后续的任务切换由谁负责？

从第二次切换开始（即从一个任务换到另一个任务），是由：

> **[[PendSV中断]]（`vPortPendSVHandler()`）** 完成的。

* **SysTick** 或其它中断触发调度请求 → 设置 PendSV 挂起；
* CPU 退出中断后执行 PendSV；
* `vPortPendSVHandler()` 保存当前任务的上下文、切换栈指针、恢复下一个任务；
* 异常返回后 CPU 自动恢复下一个任务现场。

> 简言之：
>
> * **SVC**：只在启动阶段，用来“进入第一个任务”；
> * **PendSV**：在运行阶段，负责“任务之间的切换”；
> * **SysTick**：负责周期性 tick 中断，驱动调度时机。

---


## 🧠 三、Cortex-M FreeRTOS 三兄弟分工速记

| 异常          | 名称                    | 作用         | 调用时机                           |
| ----------- | --------------------- | ---------- | ------------------------------ |
| **SVC**     | Supervisor Call       | 启动第一个任务    | 仅在 `vPortStartFirstTask()` 时触发 |
| **PendSV**  | Pendable Service Call | 执行任务上下文切换  | 每次调度发生时                        |
| **SysTick** | System Tick           | 系统心跳中断（时基） | 周期触发，可能引发任务切换                  |

---



✅ **总结一句话**

> `vPortSVCHandler()` = **“一键启动第一个任务”**
> 之后切换都由 `vPortPendSVHandler()` 负责。



