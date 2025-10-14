## 释放流程 `rt_mutex_release(mtx)`

1. 进入临界区。若 `mtx->owner != curr`，返回 `-RT_ERROR`（非法释放）。
2. `mtx->hold--`；如果 `hold > 0`，说明只是递归深度减少，退出临界区并返回 `RT_EOK`。
3. `hold == 0`：真正“放下”互斥量。
   - **若等待队列为空**：
     - `mtx->owner = NULL`；
     - 如果之前发生过继承，把 `curr` 的优先级恢复为 `original_priority`；
     - 退出临界区，返回 `RT_EOK`。
   - **若等待队列非空**：
     - 选取**最高优先级**的等待线程 `next`（队首）；
     - 将 `next` 从等待队列移除并**直接设为新 owner**：`mtx->owner = next; mtx->hold = 1;`
     - 恢复旧 owner（`curr`）的优先级到 `original_priority`（如果被提升过）；
     - 将 `next` 置为就绪（resume），随后调度器会很快让 `next` 运行；
     - 退出临界区，返回 `RT_EOK`。

> 注意：**转移所有权**时，不是先清空再让等待者“再 take 一次”，而是释放方在内核中把 `owner` 直接切到等待者并唤醒它，这样被唤醒的线程醒来就已经拥有互斥量（拿锁成功）。

![image-20251014212543805](https://mxloss112233.oss-cn-beijing.aliyuncs.com/img/image-20251014212543805.png)



![image-20251014212613993](https://mxloss112233.oss-cn-beijing.aliyuncs.com/img/image-20251014212613993.png)

![image-20251014212827831](https://mxloss112233.oss-cn-beijing.aliyuncs.com/img/image-20251014212827831.png)