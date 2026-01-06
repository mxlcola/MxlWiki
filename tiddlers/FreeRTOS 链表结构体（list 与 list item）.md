### 1）链表头：`List_t`

常见成员（按你讲解逻辑归纳）：

- `uxNumberOfItems`：链表中 item 数量（计数器）
- `pxIndex`：遍历用指针（当前“正在用”的位置）
- `xListEnd`：哨兵结点（一个“迷你 item”）

> 记忆点：**List_t = [计数器] + [遍历指针 pxIndex] + [哨兵 xListEnd]**

### 2）链表项：`ListItem_t`

常见成员：

- `xItemValue`：用于排序的值（例如按优先级/超时 tick 排序）
- `pxNext` / `pxPrevious`：双向指针
- `pvOwner`：拥有者（通常指向包含该 item 的对象，如 TCB）
- `pxContainer`：反向指回“所在链表头 List_t”

> 记忆点：
>
> - `pvOwner` = “我属于谁（容器对象，如 TCB）”
> - `pxContainer` = “我在哪个链表里（List_t）”