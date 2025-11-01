# 指针移动记录

FreeRTOS 用**两个移动指针**来“记录位置”：

- **写入指针** `pcWriteTo`：指向**下一次写入**的位置。`prvCopyDataToQueue()` 中拷完数据后做步进：
  1. `pcWriteTo += uxItemSize`
  2. 若 `pcWriteTo >= pcHead + uxLength * uxItemSize`，则 `pcWriteTo = pcHead`（**回绕**）
- **读取指针** `pcReadFrom`：指向**上一次被读出的**元素位置。`prvCopyDataFromQueue()` 里会：
  1. 先按步进得到“本次要读”的位置（`pcReadFrom += uxItemSize`，越界则回到 `pcHead`）
  2. 再从该位置拷出数据，并把 `pcReadFrom` 保持为“刚读过的”那个位置

同时配合**计数**：

- `uxMessagesWaiting`：写入成功 `++`，读出成功 `--`。
- 满的条件：`uxMessagesWaiting == uxLength`
   空的条件：`uxMessagesWaiting == 0`

> 小结：**不靠索引，靠“读/写指针 + 计数”**来判断空/满与拷贝位置。这避免了除法/取模，指针回绕只需比较一次。

# 一个极简“环形位置”小示例（伪代码）

```c
// 写入（队列未满前提）
memcpy(pcWriteTo, pvItem, uxItemSize);
pcWriteTo += uxItemSize;
if (pcWriteTo >= pcHead + uxLength * uxItemSize) {
    pcWriteTo = pcHead; // 回绕
}
uxMessagesWaiting++;

// 读取（队列非空前提）
uint8_t *pxRead = pcReadFrom + uxItemSize;
if (pxRead >= pcHead + uxLength * uxItemSize) {
    pxRead = pcHead; // 回绕
}
memcpy(pvItem, pxRead, uxItemSize);
pcReadFrom = pxRead;
uxMessagesWaiting--;
```