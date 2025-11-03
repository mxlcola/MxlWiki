## 🧠 三、列表的初始化与使用

### 1. 初始化列表

```c
void vListInitialise(List_t * const pxList);
```

主要操作：

- 将 `xListEnd` 的前后指针指向自身；
- `uxNumberOfItems = 0`；
- `pxIndex = &xListEnd`。

------

### 2. 初始化列表项

```c
void vListInitialiseItem(ListItem_t * const pxItem);
```

- 将 `pxContainer` 置为 `NULL`；
- 初始化时不属于任何列表。

------

### 3. 插入列表项

有两种插入方式：

#### （1）按排序值插入：

```c
void vListInsert(List_t * const pxList, ListItem_t * const pxNewListItem);
```

- 按 `xItemValue` 递增排序插入；
- 常用于“[[延时任务列表]]”。

#### （2）直接插入列表尾部：

```c
void vListInsertEnd(List_t * const pxList, ListItem_t * const pxNewListItem);
```

- 无需排序；
- 常用于“[[就绪链表]]”等 FIFO 场景。

------

### 4. 移除列表项

```c
UBaseType_t uxListRemove(ListItem_t * const pxItemToRemove);
```

- 将节点从当前列表中摘除；
- 更新列表项的容器指针；
- 返回剩余元素数量。

------