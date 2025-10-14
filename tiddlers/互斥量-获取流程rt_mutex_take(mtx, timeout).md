# 获取流程 `rt_mutex_take(mtx, timeout)`

核心逻辑（简化伪码）：

1. 进入[[临界区]]（关中断或调度锁）
2. **如果 `mtx->owner == curr`（同一线程再次获取）**
   - 互斥量是递归的：`mtx->hold++`，退出临界区，返回 `RT_EOK`。
3. **如果 `mtx->owner == NULL`（空闲）**
   - 把 `owner = curr`，`hold = 1`，记录 `original_priority = curr->current_priority`（或在首次继承时记录），退出临界区，返回 `RT_EOK`。
4. **否则（被其他线程占用）**
   - **非阻塞/零等待**：如果 `timeout == 0`，退出临界区，返回 `-RT_ETIMEOUT`（有实现也用 `-RT_EBUSY`，RT-Thread 常见为 `-RT_ETIMEOUT`）。
   - **需要等待**：
     1. 将 `curr` 以优先级顺序插入 `mtx->parent.suspend_thread` 等待队列；
     2. **优先级继承**：若 `curr` 的优先级高于 `owner`，临时把 `owner` 的动态优先级提升到更高（数值更小）以避免优先级反转；
     3. 挂起 `curr` 并触发调度；
     4. 被唤醒后（获取到锁或超时）：
        1. 若 `thread->error == RT_EOK`，说明已被设置为新 owner（见释放流程），返回 `RT_EOK`；
        2. 若是超时，则返回 `-RT_ETIMEOUT`。
5. 退出临界区。