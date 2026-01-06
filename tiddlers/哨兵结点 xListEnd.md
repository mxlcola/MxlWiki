## 为什么是 mini item?

哨兵只需要参与链表连接与排序边界，因此通常不需要 `owner/container` 等字段（节省内存，嵌入式里很关键）。

哨兵初始化后的关键状态：

- `xListEnd.xItemValue = 最大值（portMAX_DELAY / 最大无符号值）`
- `xListEnd.pxNext = &xListEnd`（指向自己）
- `xListEnd.pxPrevious = &xListEnd`（指向自己）
- `pxIndex = &xListEnd`
- `uxNumberOfItems = 0`

> 记忆点：**空链表时，next/prev 都“绕回自己”，形成自环。**