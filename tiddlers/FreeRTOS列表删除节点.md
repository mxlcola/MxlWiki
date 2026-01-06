## 删除结点（以及 pxIndex 的特殊处理）

对应函数常见为：`uxListRemove( ListItem_t *pxItemToRemove )`

**标准删除两步：**

![image-20260105202900575](https://mxloss112233.oss-cn-beijing.aliyuncs.com/img/image-20260105202900575.png)

- 左边跨过去：`pxItem->pxPrevious->pxNext = pxItem->pxNext`
- 右边跨过去：`pxItem->pxNext->pxPrevious = pxItem->pxPrevious`

**FreeRTOS 的特殊点：如果删除的正好是 pxIndex 指向的结点：**

- 需要调整遍历指针（例如挪到前一个结点）
- 防止 pxIndex 悬空

**删除后的收尾：**

- `pxItem->pxContainer = NULL`（表示不属于任何链表）
- `uxNumberOfItems--`
- 返回更新后的 `uxNumberOfItems`（常见设计）

------