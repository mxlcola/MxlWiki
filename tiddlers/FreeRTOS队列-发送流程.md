# 发送流程（xQueueGenericSend）

核心步骤（省略锁与临界细节）：

1. 若队列**未满**：
   - `prvCopyDataToQueue()` 把数据拷到 `pcWriteTo`，步进回绕；`uxMessagesWaiting++`。
   - 若有任务**在等接收**（`xTasksWaitingToReceive` 非空），调用 `xTaskRemoveFromEventList()` 解除**一个最高优先级**等待者的阻塞；必要时触发一次 **yield**。
2. 若队列**已满**且 `xTicksToWait > 0`（允许阻塞）：
   - 把当前任务放入 `xTasksWaitingToSend`，用 `vTaskPlaceOnEventList()` 阻塞相应的滴答数；切出（yield）。
   - 被唤醒后重试发送。
3. 若队列**已满**且不允许阻塞：直接返回 `errQUEUE_FULL`。

从 ISR 发送：`xQueueSendFromISR()` / `xQueueGenericSendFromISR()`

- 不可阻塞；若成功且唤醒了更高优先级任务，通过 `*pxHigherPriorityTaskWoken` 通知外层在退出中断时请求切换（`portYIELD_FROM_ISR()`）。