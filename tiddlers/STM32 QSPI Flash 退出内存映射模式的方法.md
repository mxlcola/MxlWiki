# STM32 QSPI Flash 退出内存映射模式的方法

片外flash内存映射模式下

- 只能在读取状态下进行，开启了就写入不了了

## 映射模式下存在的问题？

只能读取数据，但是无法写入数据，原因：

内存映射模式下，spi状态会变成

<img src="https://tc8483.oss-cn-beijing.aliyuncs.com/image/image-20250217201800526.png" alt="image-20250217201800526" style="zoom:50%;" />、

写入操作需要执行指令 HAL_QSPI_Command 操作 指令操作无效

```c
HAL_QSPI_Command(&QSPIHandle,&sCommand,1000);
```

<img src="https://tc8483.oss-cn-beijing.aliyuncs.com/image/image-20250217202007961.png" alt="image-20250217202007961" style="zoom:50%;" />



## 如何退出映射模式？

**清除繁忙位并重新初始化Flash即可**

<img src="https://tc8483.oss-cn-beijing.aliyuncs.com/image/image-20250217202250716.png" alt="image-20250217202250716"  />

官网回答：

[STM32F7 QSPI Exit Memory Mapped Mode - STMicroelectronics Community](https://community.st.com/t5/stm32-mcus-products/stm32f7-qspi-exit-memory-mapped-mode/m-p/452589)

实现代码

```c
HAL_QSPI_Abort(&hqspi);
BSP_QSPI_Init();
```

[[退出内存映射执行参考代码]]

