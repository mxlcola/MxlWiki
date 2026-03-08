```c
void Jump_ToApplication(UINT32 appAddr)
{ 
    
    pFunction 	pApplicationFun = NULL;
    UINT32 		jumpAddress = 0;
    
    /* Check if valid stack address (RAM address) then jump to user application */
    if (((*(__IO UINT32*)appAddr) & 0x24000000 ) == 0x24000000)
    {
		/* Jump to user application */
		jumpAddress = *(__IO UINT32*) (appAddr + 4);
		pApplicationFun = (pFunction) jumpAddress;
		/* Initialize user application's Stack Pointer */
		__set_MSP(*(__IO UINT32*) appAddr);
		
		__set_CONTROL(0);
		
		pApplicationFun();
    }      
}

```



### 第一层：总体概括（先说结论）

> 这个函数的作用是从 Bootloader **跳转到用户 APP 执行**。**本质上是模拟了一次 CPU 复位后的启动过程——设置栈指针、取复位向量、跳转执行。**

------

### 第二层：逐步拆解

> ARM Cortex-M 的 Flash 起始位置存放的是**中断向量表**，前两个 word 分别是：



```
appAddr + 0  →  初始栈顶指针 (MSP)
appAddr + 4  →  Reset_Handler 入口地址
```

> 这个函数做了 4 件事：

**1. 校验 APP 是否有效**（第 110 行）

> 读取 `appAddr` 处的第一个 word（即栈顶指针），检查它是否指向 `0x24000000` 开头的 AXI SRAM 区域。如果 Flash 是空的或者数据损坏，栈指针不会落在 RAM 范围内，就不跳转，避免跑飞。

**2. 取出复位向量作为入口**（第 113~114 行）

> `appAddr + 4` 存的是 `Reset_Handler` 的地址，把它赋给函数指针，这就是 APP 的入口。

**3. 切换栈指针**（第 116 行）

> `__set_MSP()` 把 SP 设为 APP 定义的栈顶。从这一刻起，Bootloader 自己的栈就废弃了。

**4. 设置 CONTROL 寄存器 + 跳转**（第 118~120 行）

> `__set_CONTROL(0)` 确保 CPU 处于**特权级 + 使用 MSP** 的状态，和复位后一致。然后通过函数指针调用跳过去，APP 开始执行，不再返回。

------

### 第三层：主动补充（加分项）

> 不过这段代码有几个可以改进的地方：

1. **栈指针校验不严谨** —— `& 0x24000000 == 0x24000000` 只检查了两个 bit，`0xFFFFFFFF`（Flash 擦除态）也能通过。应该做范围判断 `sp >= 0x24000000 && sp < 0x24080000 && sp != 0xFFFFFFFF`
2. **跳转前应该加 ISB 指令** —— 修改 MSP 和 CONTROL 后，ARM 建议插入 `__ISB()` 确保流水线刷新，否则后续指令可能还在用旧的栈指针执行
3. **没有设置 VTOR** —— 跳转前最好把 `SCB->VTOR = appAddr` 设好，而不是依赖 APP 自己设置，更安全

![image-20260308073738856](https://mxloss112233.oss-cn-beijing.aliyuncs.com/img/image-20260308073738856.png)