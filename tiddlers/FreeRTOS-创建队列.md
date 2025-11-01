[[FreeRTOS-队列结构体]]

## 初始化队列

- `xQueueGenericCreate()` / `xQueueCreate()`：动态创建，分配 `Queue_t` 和数据区（数据区大小 `uxLength * uxItemSize`）。
- `xQueueCreateStatic()`：静态创建，你提供 `StaticQueue_t` 和数据缓冲。
- 初始化时常见做法：
  - `pcWriteTo = pcHead;
  - ` pcReadFrom = pcHead + (uxLength - 1) * uxItemSize;`
     这样**首次接收**时，`pcReadFrom` 先向前步进一个元素，就正好落在第 0 个元素的位置。

![image-20251101192320042](https://mxloss112233.oss-cn-beijing.aliyuncs.com/img/image-20251101192320042.png)