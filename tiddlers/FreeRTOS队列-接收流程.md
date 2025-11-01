# 接收流程（xQueueGenericReceive）

类似镜像逻辑：

1. 若队列**非空**：
   - `prvCopyDataFromQueue()` 从“步进后的位置”拷出数据，`uxMessagesWaiting--`。
   - 若有任务**在等发送**（`xTasksWaitingToSend` 非空），解除其一并可能触发 **yield**。
2. 若队列**为空**且 `xTicksToWait > 0`：
   - 当前任务进入 `xTasksWaitingToReceive` 阻塞，超时或被唤醒后重试。
3. 若队列**为空**且不允许阻塞：返回 `errQUEUE_EMPTY`。

从 ISR 接收：`xQueueReceiveFromISR()`（同样不可阻塞，可能唤醒更高优先级任务）。