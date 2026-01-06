关联：

[[哨兵结点 xListEnd]]

[[pxIndex 的作用（遍历 + “公平性”）]]

## ⚙️ 二、主要结构体定义

在 `list.h` 中定义了三个核心结构体：

### 1. `List_t` —— 列表结构

```c
typedef struct xLIST {
    volatile UBaseType_t uxNumberOfItems; // 列表中元素数量
    ListItem_t *pxIndex;                  // 当前索引指针（循环使用）
    MiniListItem_t xListEnd;              // 哨兵节点（列表结束标志）
} List_t;
```

- **`uxNumberOfItems`**：当前列表中的节点数；
- **`pxIndex`**：遍历用指针；
- **`xListEnd`**：特殊的“结束项”，起到循环链表哨兵作用。

------

### 2. `ListItem_t` —— 列表项（普通节点）

```c
struct xLIST_ITEM {
    TickType_t xItemValue;        // 排序或调度关键值（如延时到期时间）
    struct xLIST_ITEM *pxNext;    // 下一个节点
    struct xLIST_ITEM *pxPrevious;// 上一个节点
    void *pvOwner;                // 指向拥有此项的对象（通常是TCB任务控制块）
    struct xLIST *pxContainer;    // 指向所在的列表
};
```

🔹 这是列表中最常用的节点类型，用于挂载任务或事件。
 比如任务的 TCB 中会包含多个 `ListItem_t` 成员，用于插入不同列表。

------

### 3. `MiniListItem_t` —— 最小列表项

```c
struct xMINI_LIST_ITEM {
    TickType_t xItemValue;        
    struct xLIST_ITEM *pxNext;
    struct xLIST_ITEM *pxPrevious;
};
```

🔹 这是一个简化版本，仅在 `xListEnd` 作为哨兵使用。

------