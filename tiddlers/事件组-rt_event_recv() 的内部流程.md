# rt_event_recv() 

```c
rt_err_t rt_event_recv(rt_event_t   event,
                       rt_uint32_t  set,
                       rt_uint8_t   option,
                       rt_int32_t   timeout,
                       rt_uint32_t *recved)
```
## rt_event_recv参数
1. 所属事件组
1. 哪些事件
1. 事件组合，是全部还是满足其中一个，是否需要清除
1. 超时

## 等待事件处理逻辑

> 以“等待事件”为主线，梳理关键步骤：

1. **关中断/锁调度**进入临界区，读取当前事件集合 `evt->set`。
2. **匹配判断**：

   * AND：`((evt->set & req_mask) == req_mask)`
   * OR：`((evt->set & req_mask) != 0)`
3. **若已满足**：

   * 根据 `RT_EVENT_FLAG_CLEAR` 决定是否清位：

     * AND：清 `req_mask`
     * OR：清 `evt->set & req_mask`（即匹配到的那一部分）
   * 把**实际满足的位**写入 `*recved`（OR 模式可能只是子集）。
   * **立即返回 `RT_EOK`**。
4. **若未满足且 `timeout == 0`**：立刻返回 `-RT_ETIMEOUT`。
5. **若未满足且允许阻塞**：

   * 填写当前线程的等待信息（`thread->event_set = req_mask`、`thread->event_option = flags` 等）。
   * 当前线程加入 `evt->parent.suspend_thread` **挂起队列**（通常按优先级入队）。
   * 设置超时（若非永久等待），进入阻塞态 `RT_THREAD_SUSPEND` 并**触发调度**。
6. **被唤醒后**（见第 5 节“唤醒流程”）：

   * 若因满足事件而唤醒：返回 `RT_EOK`，`*recved` 为满足位；
   * 若因超时被内核从挂起队列移除：返回 `-RT_ETIMEOUT`。

---
