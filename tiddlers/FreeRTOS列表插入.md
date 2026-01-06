## 尾部插入（公平轮转）

对应函数常见为：`vListInsertEnd( List_t *pxList, ListItem_t *pxNewListItem )`

**你讲解的定位规则：**

- “尾部”不是 `xListEnd` 之后
- 而是：**插入到 pxIndex 之前**
- 等价说法：插入到 `pxIndex->pxPrevious` 之后

**核心指针修改（四步连线）**：
设：

- `I = pxList->pxIndex`
- `P = I->pxPrevious`
- `N = pxNewListItem`

则插入：

- `N->pxNext = I`
- `N->pxPrevious = P`
- `P->pxNext = N`
- `I->pxPrevious = N`

再做：

- `N->pxContainer = pxList`
- `uxNumberOfItems++`

------

## 空链表插入为什么也成立

空链表时：

- `pxIndex == &xListEnd`
- `pxIndex->pxPrevious == &xListEnd`

所以“插到 pxIndex 前面”就变成：

- **插到 xListEnd 前面**
- 结果链表变为：`xListEnd <-> item1` 并且保持循环

> 记忆点：**哨兵自环让空表逻辑自动合并到通用插入逻辑，无需 if/else 特判。**

![image-20260105202253833](https://mxloss112233.oss-cn-beijing.aliyuncs.com/img/image-20260105202253833.png)

------

## 按序插入（按 xItemValue 排序）

![image-20260105202447537](https://mxloss112233.oss-cn-beijing.aliyuncs.com/img/image-20260105202447537.png)

对应函数常见为：`vListInsert( List_t *pxList, ListItem_t *pxNewListItem )`

**目标：**按 `xItemValue` 从小到大插入
（哨兵 `xListEnd` 的 value 是最大值，所以它总能兜底）

**查找位置逻辑（你举的 1,2,6 插入 5）：**

![image-20260105202536036](https://mxloss112233.oss-cn-beijing.aliyuncs.com/img/image-20260105202536036.png)

- 从某个起点（常为 `&xListEnd` 或 `xListEnd.pxNext`）开始
- 不断检查“下一项的 value 是否仍 <= newValue”
- 找到第一个 `next->value > newValue` 的位置停下
- 在当前位置之后插入

插入动作仍是“双向链表标准四连线”，外加：

- `pxContainer = pxList`
- `uxNumberOfItems++`

> 记忆点：**排序插入 = 先找洞，再用标准插入。**



------