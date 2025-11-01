# 🌀 内存布局示意（环形缓冲）

假设创建一个可容纳 3 个元素、每个元素大小 4 字节的队列：

```
pcHead ──► [msg0][msg1][msg2]
             ↑
             pcWriteTo（当前写入位置）

pcTail ──► 指向缓冲区末尾后1字节
```

- **写入时**：从 `pcWriteTo` 开始拷贝数据，之后 `pcWriteTo += uxItemSize`。若越过 `pcTail`，则回绕到 `pcHead`。
- **读取时**：先从 `pcReadFrom` 前进一个元素，再拷贝。若越过 `pcTail`，同样回绕。
- 通过 `uxMessagesWaiting` 判断队列空/满。