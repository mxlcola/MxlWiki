### **1. `sprintf`**
`sprintf` 直接将格式化字符串写入目标缓冲区 **而不进行[[边界检查]]**，如果写入的数据超出缓冲区大小，会导致**缓冲区溢出（Buffer Overflow）**，可能会破坏其他内存数据甚至导致程序崩溃。

#### **示例（容易溢出）**
```c
#include <stdio.h>

int main() {
    char buffer[10];  // 只分配了10字节
    sprintf(buffer, "Hello, World!"); // 超出 buffer 大小，导致溢出
    printf("%s\n", buffer);  
    return 0;
}
```
**问题：** `"Hello, World!"` 长度超过 `buffer`，会导致[[内存溢出]]，**可能覆盖临近的内存区域，造成不可预知的错误**。