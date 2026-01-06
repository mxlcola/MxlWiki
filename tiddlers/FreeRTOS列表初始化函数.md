## 初始化函数（两类）

### 1）初始化链表：`vListInitialise( List_t *pxList )`

做四件事：

1. `pxList->pxIndex = &pxList->xListEnd`
2. `pxList->xListEnd.xItemValue = 最大值`
3. `xListEnd.next = xListEnd.prev = &xListEnd`（自环）
4. `pxList->uxNumberOfItems = 0`

### 2）初始化链表项：`vListInitialiseItem( ListItem_t *pxItem )`

关键点：

- `pxItem->pxContainer = NULL`（表示未加入任何链表）
- 其他字段通常做默认/断言相关的初始化

------