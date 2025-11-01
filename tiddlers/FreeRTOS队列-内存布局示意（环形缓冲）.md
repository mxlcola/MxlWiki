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

**注意：**

`xQueueGenericReset` 里面

xQueue.pcTail做的哨兵，在数据区域之外

```c
        pxQueue->u.xQueue.pcTail = pxQueue->pcHead + ( pxQueue->uxLength * pxQueue->uxItemSize ); /*lint !e9016 Pointer arithmetic allowed on char types, especially when it assists conveying intent. */
        pxQueue->uxMessagesWaiting = ( UBaseType_t ) 0U;
        pxQueue->pcWriteTo = pxQueue->pcHead;
        pxQueue->u.xQueue.pcReadFrom = pxQueue->pcHead + ( ( pxQueue->uxLength - 1U ) * pxQueue->uxItemSize ); /*lint !e9016 Pointer arithmetic allowed on char types, especially when it assists conveying intent. */
```