**概念**：可读可写数据段存储具有初始非零值的全局变量或静态变量。 

**特点**：

- 需要从ROM复制到RAM中
- 可以在程序运行期间被修改
- 占用ROM和RAM空间

**示例**：

```c
char g_Char = 'A';           // 存储在可读可写数据段中
static int count = 10;       // 存储在可读可写数据段中
```