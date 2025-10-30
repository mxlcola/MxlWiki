# 🧩 总体流程口诀

> “建空闲，建时钟，设时基，备中断，切首任，永不返。”


### 具体要做什么事情

* 创建空闲任务(IdleTask)
* 创建 [[定时器任务]] (TimerTask)
* 启动调度器(xPortStartScheduler)
  * 设置PendSV和SysTick中断为最低优先级
  * 使能[[SysTick中断]]
  * 触发SVC异常：启动第1个任务


# 🏗️ 二、阶段详解

## ✅ 1️⃣ 准备阶段：建立必要任务

函数：`vTaskStartScheduler()`

主要做：

1. **创建 Idle 任务**（空闲任务）
   - 名称：`prvIdleTask()`
   - 优先级最低（0 级）
   - 保证系统总有任务可运行。
2. **若启用时间片或延时功能**，创建 **Timer Service 任务**（软件定时器守护任务）。
3. 检查是否成功创建所有必须任务。
   - 若失败（通常是内存不足），直接返回，不启动调度器。

------

## ✅ 2️⃣ 建立环境阶段：配置硬件支持

主要是 RTOS “底层端口”负责（`port.c` 中）：

- 初始化**任务栈指针**与 **TCB**：
   每个任务在创建时都已“伪造”一个上下文，栈中放入初始寄存器值、程序计数器（任务入口函数）、返回地址等。
   所以它们已经“准备好被切入”。
- 关闭中断、设置优先级分组（针对 Cortex-M：`portDISABLE_INTERRUPTS()`）。
- 启动 **系统时钟中断**（Tick 定时器）：
  - 例如 Cortex-M 下是 `SysTick`；
  - 用来周期触发调度（抢占或时间片）。

------

## ✅ 3️⃣ 启动阶段：进入第一个任务

调用：`xPortStartScheduler()`（架构相关）

### 内核层

`vTaskStartScheduler()` 会调用 `xPortStartScheduler()`。
 这个函数做两件事：

1. **设置中断优先级**和**启动系统 tick**。
2. 调用汇编函数 `vPortStartFirstTask()` —— 这是关键！

------

# ⚙️ 三、关键函数：`vPortStartFirstTask()`

### 🔍 在 Cortex-M 架构上的典型实现（汇编）

```asm
vPortStartFirstTask:
    ldr r0, =0xE000ED08     ; 向量表地址寄存器
    ldr r0, [r0]
    ldr r0, [r0]             ; 取栈顶地址
    msr msp, r0              ; 设置主栈指针 MSP
    cpsie i                  ; 开中断
    svc 0                    ; 触发 SVC 中断
```

### 🔍 对应的 SVC 处理函数：`vPortSVCHandler`

[[SVC中断]]

SVC 进入后执行：

1. **从第一个任务的 TCB 取出任务栈指针 SP。**
2. **加载任务上下文（寄存器、PSR、PC）。**
3. **异常返回**，CPU自动恢复寄存器并跳到任务入口函数执行。

于是——第一个任务开始运行！

------

# 🧠 四、运行后的状态

一旦第一个任务运行起来：

- 系统 Tick 中断定期发生；
- 每次 Tick 调用 `xTaskIncrementTick()`；
- 调度器根据优先级决定是否切换任务；
- 切换通过 PendSV 异常完成（同样是汇编实现）。

从此开始，**RTOS进入多任务循环**。

------

# 📜 五、FreeRTOS 启动调度器的核心调用链（简化）

```
vTaskStartScheduler()
 ├─ prvIdleTask() + TimerTask()
 ├─ xPortStartScheduler()
 │    ├─ vPortSetupTimerInterrupt()
 │    └─ vPortStartFirstTask()   ← 汇编触发 SVC
 └─ (never returns)
```

------



## 速记口诀版总结

| 阶段 | 口诀                 | 作用               |
| ---- | -------------------- | ------------------ |
| 准备 | 建空闲、建时钟       | 准备系统基础任务   |
| 建环 | 设时基、备中断       | 启动 Tick 定时器   |
| 起首 | 首任务、SVC 起       | 切入第一个任务     |
| 运行 | Tick 触发、PendSV 换 | 正式进入多任务调度 |

------