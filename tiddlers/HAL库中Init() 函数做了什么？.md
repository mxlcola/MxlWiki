**A:**  
`Init()` 会完成 **初始化硬件外设** 所需要的全部步骤，包括但不限于：

1. **使能时钟**：开启对应外设和 GPIO 的时钟（如 `__HAL_RCC_FMC_CLK_ENABLE()`）  
2. **配置GPIO**：将相应引脚配置成复用功能（AF），比如地址线、数据线、控制线  
3. **初始化外设寄存器**：设置 FMC/SRAM/SDRAM 等控制器的各种控制参数（如时序、工作模式）  
4. **结构体赋值**：存储初始化状态用于后续函数调用（Handle 结构）
