## pvOwner 和 pxContainerr容易记混

FreeRTOS 里：

- `pvOwner`：更像我们常说的 **container_of**（包含该 item 的结构体，如 [[TCB]]）
- `pxContainer`：反而指向 **List_t（所在链表）**

**避免混淆口诀：**

- **Owner = 谁拥有我（TCB/对象）**
- **Container = 我在哪个 List 里**

------