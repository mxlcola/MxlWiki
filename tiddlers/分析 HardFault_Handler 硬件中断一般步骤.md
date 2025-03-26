Coretex-M3 和 Coretex-M4 架构中，R14(LR) 数值及其代表含义：

![](https://tc8483.oss-cn-beijing.aliyuncs.com/img/2fc112061fbf1668b6b89ac6f71717c8.jpeg)

*   LR = 0xFFFFFFF9，MSP 指向的地址保存有异常信息；(使用 FPU 时，LR=0xFFFFFFE9)
*   LR = 0xFFFFFFFD，PSP 指向的地址保存有异常信息；(使用 FPU 时，LR=0xFFFFFFED)
*   LR = 0xFFFFFFF1，表示中断返回时从 MSP 堆栈恢复寄存器值，中断返回后进入 Handler 模式，使用 MSP 堆栈；(使用 FPU 时，LR=0xFFFFFFE1)

以下使用 Coretex-M3 内核架构，进行异常分析,

3.1 保护现场：
---------

  出现异常时，不要复位设备，保持当前设备运行的异常环境，能更大概率找到问题。若开启了看门狗，超时就会硬件复位，要复现并找到该问题，需要将看门狗进行关闭并重新烧写程序，等待问题复现。

3.2 使用 KEIL 进入 Debug 调试
-----------------------

![](https://tc8483.oss-cn-beijing.aliyuncs.com/img/80d3c8e58b2782ef0ff402fb5bbb5a84.jpeg)  

![](https://tc8483.oss-cn-beijing.aliyuncs.com/img/39c852a8971786872c4e59a55bcbc9bf.jpeg)

  若进入调试界面后，没有 `Registers` 窗口，就点击`KEIL-->View-->Registers Windown` 就会弹出该窗口。

3.3 `Registers` 窗口查看 R14（LR），确认保存异常数据的堆栈指针
------------------------------------------

![](https://tc8483.oss-cn-beijing.aliyuncs.com/img/29c0befa6c942d861a912ba012ec794c.jpeg)

3.4 在 Memory1 窗口 查看栈帧内容
-----------------------

  若 `Memory1` 窗口未显示，就点击`KEIL-->View-->Memory Windowns-->Memory1` 就会弹出该窗口。

![](https://tc8483.oss-cn-beijing.aliyuncs.com/img/8ee5002b41388aebe67a69a7527330e9.jpeg)

3.5 使用 Disassembly Window 反汇编窗口 定位异常区域
--------------------------------------

  若 `Disassembly` 反汇编窗口未显示，就点击`KEIL-->View-->Disassembly Windown` 就会弹出该窗口。

![](https://tc8483.oss-cn-beijing.aliyuncs.com/img/38326e1cdf679fc38ef8b65cbbfdd37f.jpeg)  
![](https://tc8483.oss-cn-beijing.aliyuncs.com/img/482f175c892fb6681d2c73816bea26ec.jpeg)  
![](https://tc8483.oss-cn-beijing.aliyuncs.com/img/00b593c0c80e438d9e3aeb713ac516c3.jpeg)

  通过 MSP 堆栈指针指向的地址 0x20002B10 记录的上下文信息，查找 Returnaddress 地址定位到异常程序代码处，可分析出产生 HardFault_Handler 中断的原因 是由于函数内部数组 buf[50] 只有 50 个字节大小，但使用 buf 时操作了 100 个字节大小，导致了数组越界错误。