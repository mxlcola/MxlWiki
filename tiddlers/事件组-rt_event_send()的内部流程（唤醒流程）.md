## ！重点：这里是遍历挂起队列所有的线程

1. **关中断/锁调度**，`evt->set |= posted_set;` 把要发送的位**并入**事件集合。
2. **遍历挂起队列** `evt->parent.suspend_thread`：

   * 对每个等待线程 `th`，拿到其 `th->event_set`（关心的位）与 `th->event_option`（AND/OR、CLEAR）。
   * 判断是否满足：

     * AND：`((evt->set & th->event_set) == th->event_set)`
     * OR：`((evt->set & th->event_set) != 0)`
   * **若满足**：

     * 计算本次“命中的位”：`hit = evt->set & th->event_set`（AND 模式等于 `th->event_set`）。
     * 若线程设置了 `RT_EVENT_FLAG_CLEAR`：

       * AND：`evt->set &= ~th->event_set;`
       * OR：`evt->set &= ~hit;`
     * 把 `hit` 写回到 `th->wakeup_set`（或通过参数返回给接收者）。
     * **将该线程从挂起队列移除并就绪**（`rt_thread_resume(th)`）。
3. **是否唤醒“一个”还是“多个”线程？**

   * 事件是**位级别的共享资源**，一组位可能同时满足多个线程（特别是**未清位**或**OR 且 posted\_set 含多位**的场景）。
   * RT-Thread 的实现会**遍历队列，逐个判断并唤醒所有满足条件的线程**（若 CLEAR 清掉了对应位，后续线程就可能不再满足）。
   * 因此，**实际被唤醒的线程数量**取决于：posted 位、等待掩码、AND/OR 模式以及是否 CLEAR。
4. **遍历结束**后解锁。如果有线程被就绪，**调度器可能立即切换**到最高优先级就绪线程。

> 小结：**发送→置位→遍历等待队列→匹配就绪（可清位）→可能唤醒一个或多个线程**；是否“只唤醒一个”并不是事件模型的固定规则，而是匹配/清位策略共同决定的结果。

---