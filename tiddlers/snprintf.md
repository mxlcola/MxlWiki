### **2. `snprintf`**
`snprintf` 提供了**安全性**，可以指定最大写入的字符数，防止溢出。

#### **示例（安全写入）**
```c
#include <stdio.h>

int main() {
    char buffer[10];
    snprintf(buffer, sizeof(buffer), "Hello, World!"); // 限制写入大小
    printf("%s\n", buffer);  
    return 0;
}
```