1. **CubeMX 配置**  
   - 启用 FMC 接口  
   - 选择存储类型（SRAM/NOR/SDRAM）  
   - 配置地址线、数据线、控制线引脚  
   - 设置时序参数（地址建立时间、数据保持时间等）

2. **生成代码后调用初始化函数**  
   - 使用 `HAL_SRAM_Init()`、`HAL_SDRAM_Init()` 等初始化接口

3. **通过指针访问外部存储器地址空间**  
   - 比如：
     ```c
     *(uint16_t *)(0x60000000) = 0x1234; // 写入SRAM
     ```

---