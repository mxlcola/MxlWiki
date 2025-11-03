### 🧩 一、任务的概念

- 任务是一个无限循环的函数，用于执行某种特定功能（例如：读取传感器、通信、控制设备等）。
- 在 FreeRTOS 中，每个任务由内核调度器管理，调度器会根据优先级决定哪个任务占用 CPU。

------

### ⚙️ 二、任务的创建

任务由 `xTaskCreate()` 或 `xTaskCreateStatic()` 创建：

```c
xTaskCreate(
    vTaskCode,        // 任务函数
    "TaskName",       // 任务名称
    configMINIMAL_STACK_SIZE, // 栈大小
    NULL,             // 传入任务的参数
    tskIDLE_PRIORITY, // 优先级
    &xHandle          // 任务句柄
);
```

任务函数的典型结构：

```c
void vTaskCode(void *pvParameters) {
    for(;;) {
        // 任务要反复执行的内容
        vTaskDelay(1000 / portTICK_PERIOD_MS); // 延时1秒
    }
}
```

------

### 🧠 三、任务的运行机制

FreeRTOS 使用**优先级抢占调度**：

- 优先级高的任务会抢占低优先级任务的执行。
- 同优先级的任务通过**时间片轮转**调度（如果启用了时间片功能）。

调度器通过 `vTaskStartScheduler()` 启动。一旦运行，系统就进入多任务模式。

------

### 🕹 四、任务的状态

FreeRTOS 任务在运行时可处于以下几种状态：

| 状态      | 含义                       |
| --------- | -------------------------- |
| Running   | 正在使用 CPU               |
| Ready     | 可运行但暂未获得 CPU       |
| Blocked   | 等待事件（如延时、信号量） |
| Suspended | 被挂起，不参与调度         |
| Deleted   | 已被删除，等待资源释放     |

------

### 🧾 五、任务间通信

任务之间可以通过以下机制通信：

- **队列（Queue）**：用于任务间消息传递。
- **信号量（Semaphore）**：用于同步或资源保护。
- **互斥量（Mutex）**：用于防止共享资源竞争。
- **事件组（Event Group）**：用于多事件同步。
- **消息缓冲区（Message Buffer）\**和\**流缓冲区（Stream Buffer）**：用于流式数据传输。

------

### 🔍 六、任务的管理

- **vTaskSuspend() / vTaskResume()**：挂起和恢复任务。
- **vTaskDelete()**：删除任务。
- **uxTaskGetStackHighWaterMark()**：查看任务栈使用情况。
- **vTaskPrioritySet() / uxTaskPriorityGet()**：动态调整优先级。

------

### ✅ 总结

在 FreeRTOS 中：

- **任务 = 独立的执行单元**；
- **内核调度任务运行**；
- **优先级和调度策略决定执行顺序**；
- **通过信号量、队列等机制实现任务间通信与同步**。

------

