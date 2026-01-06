## 链表插入的位置？

`pxIndex` 是 FreeRTOS 为了“遍历/轮转使用”设计的指针，它会在遍历时不断移动。

你讲的“公平性”可以总结为：

- **新插入的结点，不应该插到 pxIndex 之后抢占刚排队的结点**
- 所以“尾部插入”使用的是：**插到 pxIndex 之前**（更准确：插到 `pxIndex->pxPrevious` 之后）

> 记忆句：**Append = 插在 pxIndex 前面（保证轮转公平）**

![image-20260105202003304](https://mxloss112233.oss-cn-beijing.aliyuncs.com/img/image-20260105202003304.png)

------