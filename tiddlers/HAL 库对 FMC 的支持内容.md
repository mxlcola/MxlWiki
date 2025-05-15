HAL 库提供了针对每种存储器类型的初始化结构体和函数，例如：
- `HAL_SRAM_Init()`：初始化外部 SRAM
- `HAL_NOR_Init()`：初始化外部 NOR Flash
- `HAL_SDRAM_Init()`：初始化 SDRAM
- `HAL_NAND_Init()`：初始化 NAND（部分系列支持）

[[FMC能连接什么设备？]]