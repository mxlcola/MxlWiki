> 面试官，关于 H743 的 Cache 和 DMA 一致性问题，我从**问题本质、我的方案、具体实现、方案选型理由**四个方面来说。

------

### 一、问题本质

> H743 是 Cortex-M7 内核，有 L1 D-Cache。CPU 读写内存经过 Cache，但 DMA 直接走总线访问物理 RAM，绕过了 Cache。这就导致两个方向的不一致：
>
> - **DMA 接收方向**：DMA 把新数据写到了 RAM，但 CPU 读的时候 Cache 命中了旧值，读到脏数据
> - **DMA 发送方向**：CPU 写的新数据还在 Cache 里没刷回 RAM，DMA 从 RAM 读到的是旧值
>
> 这类 bug 非常隐蔽，表现为偶发的数据异常，不好复现，排查成本很高。

------

### 二、我的方案

> 我用的是 **MPU 分区 + Non-cacheable RAM** 的方案，核心思路是：让 DMA 缓冲区所在的内存区域不经过 Cache，CPU 和 DMA 直接读写同一份物理 RAM，从根源上消除一致性问题。

------

### 三、具体实现

> 分三步：
>
> **第一步，MPU 配置。** 在系统初始化阶段，把 H743 的 D3 域 SRAM4（地址 `0x38000000`，64KB）配置为 Non-cacheable + Shareable：
>
> 
>
> ```c
> MPU_InitStruct.BaseAddress = 0x38000000;
> MPU_InitStruct.Size        = ARM_MPU_REGION_SIZE_64KB;
> MPU_InitStruct.IsCacheable = MPU_ACCESS_NOT_CACHEABLE;
> MPU_InitStruct.IsShareable = MPU_ACCESS_SHAREABLE;
> ```
>
> **第二步，链接脚本。** 定义一个 `NoneCacheable` 段，映射到 `0x38000000` 这块物理地址。
>
> **第三步，代码中使用宏声明。** 定义一个统一的宏：
>
> 
>
> ```c
> #define ATTR_NONE_CACHEABLE_RAM  __attribute__((section("NoneCacheable"))) __attribute__((aligned(4)))
> ```
>
> 所有 DMA 缓冲区都加这个宏：
>
> 
>
> ```c
> // 串口 DMA 缓冲区
> UINT8 uartDmaRxBuff[2048] ATTR_NONE_CACHEABLE_RAM;
> UINT8 uartDmaTxBuff[2048] ATTR_NONE_CACHEABLE_RAM;
> 
> // I2S 音频 DMA 双缓冲
> INT32 adcRecvBuff1[480]   ATTR_NONE_CACHEABLE_RAM;
> INT32 adcRecvBuff2[480]   ATTR_NONE_CACHEABLE_RAM;
> 
> // DAC 播放缓冲和内存池
> INT32 dacSendBuff[480]    ATTR_NONE_CACHEABLE_RAM;
> UINT8 dacMemPool[23040]   ATTR_NONE_CACHEABLE_RAM;
> 
> // Modbus RTU DMA 缓冲区
> UCHAR modbusRxBuf[256]    ATTR_NONE_CACHEABLE_RAM;
> UCHAR modbusTxBuf[256]    ATTR_NONE_CACHEABLE_RAM;
> ```
>
> 同时，其他内存区域保持 Cache 开启。比如 D1 AXI SRAM（`0x24000000`，512KB）配置为 Write-back + Read/Write Allocate，给主程序代码、栈、堆、音频算法计算用，享受 Cache 带来的性能提升。

------

### 四、为什么选这个方案

> 我对比过三种方案：
>
> | 方案                  | 做法                             | 优点                    | 缺点                           |
> | --------------------- | -------------------------------- | ----------------------- | ------------------------------ |
> | **Non-cacheable RAM** | MPU 划区域                       | 可靠，不会漏            | Non-cacheable 空间有限         |
> | **手动 Cache 维护**   | 每次 DMA 前后调 Clean/Invalidate | 不占 Non-cacheable 空间 | 容易遗漏，缓冲区要 32 字节对齐 |
> | **关闭 D-Cache**      | SCB_DisableDCache                | 最简单                  | 整体性能大幅下降               |
>
> 我选方案一的理由：
>
> 1. **可靠性优先**——项目是声学测量仪器，数据准确性是核心要求。手动维护 Cache 每个 DMA 通道都要配对处理，串口、I2S、Modbus 加起来好几路，漏一个就是偶发数据错误，排查成本远大于预防成本
> 2. **空间够用**——我统计了所有 DMA 缓冲区总量约 45KB，SRAM4 有 64KB，还有余量
> 3. **性能影响可忽略**——DMA 缓冲区主要是 DMA 硬件在读写，CPU 只是在中断或回调中做一次 memcpy 搬到队列里，不是运算热点。真正吃性能的音频 DSP 算法运行在 Cacheable 的 AXI SRAM 上，不受影响
> 4. **维护成本低**——团队里新加一路 DMA，只要给缓冲区加个 `ATTR_NONE_CACHEABLE_RAM` 宏就行，不需要理解 Cache 维护的时序

------

### 五、补充（如果面试官追问）

> 如果后续 SRAM4 空间不够了，我会采用**混合方案**：高频 DMA（比如 48kHz 采样率的 I2S 音频）继续放 Non-cacheable 区域，低频 DMA（比如偶尔触发的 Modbus）改用手动 Cache 维护，把缓冲区搬回普通 RAM。两种方案混合使用在业界也是常见做法。
>
> 另外，如果用手动维护方案，有一个容易踩的坑：`SCB_InvalidateDCache_by_Addr` 操作的粒度是 32 字节（一个 Cache line）。如果 DMA 缓冲区没有 32 字节对齐，Invalidate 可能会误杀同一 Cache line 上其他变量的数据。所以手动方案的缓冲区必须用 `__attribute__((aligned(32)))` 保证对齐，而且大小也最好是 32 的倍数。