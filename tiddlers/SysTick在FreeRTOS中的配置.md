
# 💡 结论总结一句话：

> **FreeRTOS 不自己算主频，而是依赖你提供的 `SystemCoreClock`。**
> HAL 层负责根据硬件配置计算出 `SystemCoreClock`，
> FreeRTOS 用它来配置 SysTick 定时器，生成周期性 tick 中断。
>
> 
# 🧩 一、背景：FreeRTOS 需要一个“心跳源”

FreeRTOS 依赖周期性中断（tick interrupt）来驱动时间与调度：

* 每个 tick 调用一次 `xTaskIncrementTick()`；
* 根据 `configTICK_RATE_HZ` 确定节拍频率（例如 1000Hz → 1ms/tick）；
* 通常由硬件定时器（SysTick 或 TIMx）提供中断源。

---

# ⚙️ 二、Cortex-M 平台常用 SysTick 作为 tick 定时器

在 Cortex-M 内核里，自带一个 24 位定时器 **SysTick**，
所以 FreeRTOS 默认用它作为时间基。

在端口层代码（`port.c`）中，调用：

```c
vPortSetupTimerInterrupt()
{
    portNVIC_SYSTICK_LOAD_REG = ( configSYSTICK_CLOCK_HZ / configTICK_RATE_HZ ) - 1UL;
    portNVIC_SYSTICK_CTRL_REG = ( CLK_BIT | INT_BIT | ENABLE_BIT );
}
```

这句中的 `configSYSTICK_CLOCK_HZ`，**必须告诉它 SysTick 的真实时钟频率**。
否则中断周期会算错。

---

# 🧮 三、`configSYSTICK_CLOCK_HZ` 通常怎么写？

在 **`FreeRTOSConfig.h`** 中：

```c
#define configSYSTICK_CLOCK_HZ    ( SystemCoreClock )
```

> 🔹 意思是让 FreeRTOS 使用全局变量 `SystemCoreClock` 的值，作为 SysTick 时钟频率。

---

# 🧠 四、`SystemCoreClock` 是什么？

`SystemCoreClock` 是由 **CMSIS（ARM 官方标准）** 和 **HAL 库**维护的一个全局变量。
它反映了**当前 CPU 核心时钟（HCLK）频率**，单位 Hz。

* 在启动时，`SystemInit()` 会根据 RCC 配置（PLL、分频器等）计算初始频率；
* 当你调用 HAL 的时钟配置函数（如 `HAL_RCC_ClockConfig()`）后，
  会执行 `SystemCoreClockUpdate()`，重新计算它；
* 它的值通常存放在：

  ```c
  extern uint32_t SystemCoreClock;
  ```

  定义在 `system_stm32f4xx.c` 中。

---

# 🧩 五、`SystemCoreClock` 与 `stm32f4xx_hal_conf.h` 的关系

`stm32f4xx_hal_conf.h` **不是直接设置频率的地方**，
而是通过启用/禁用 HAL 模块、定义时钟源选项等宏影响整个 HAL 行为。
真正影响时钟的是：

* **系统时钟配置函数**（例如 CubeMX 自动生成的）：

  ```c
  void SystemClock_Config(void)
  {
      RCC_OscInitTypeDef RCC_OscInitStruct = {0};
      RCC_ClkInitTypeDef RCC_ClkInitStruct = {0};

      RCC_OscInitStruct.OscillatorType = RCC_OSCILLATORTYPE_HSE;
      RCC_OscInitStruct.HSEState = RCC_HSE_ON;
      RCC_OscInitStruct.PLL.PLLState = RCC_PLL_ON;
      RCC_OscInitStruct.PLL.PLLSource = RCC_PLLSOURCE_HSE;
      RCC_OscInitStruct.PLL.PLLM = 8;
      RCC_OscInitStruct.PLL.PLLN = 336;
      RCC_OscInitStruct.PLL.PLLP = RCC_PLLP_DIV2;
      RCC_OscInitStruct.PLL.PLLQ = 7;
      HAL_RCC_OscConfig(&RCC_OscInitStruct);

      RCC_ClkInitStruct.ClockType = RCC_CLOCKTYPE_HCLK | RCC_CLOCKTYPE_SYSCLK |
                                    RCC_CLOCKTYPE_PCLK1 | RCC_CLOCKTYPE_PCLK2;
      RCC_ClkInitStruct.SYSCLKSource = RCC_SYSCLKSOURCE_PLLCLK;
      RCC_ClkInitStruct.AHBCLKDivider = RCC_SYSCLK_DIV1;
      RCC_ClkInitStruct.APB1CLKDivider = RCC_HCLK_DIV4;
      RCC_ClkInitStruct.APB2CLKDivider = RCC_HCLK_DIV2;
      HAL_RCC_ClockConfig(&RCC_ClkInitStruct, FLASH_LATENCY_5);

      SystemCoreClockUpdate();
  }
  ```

这里计算出的主频，比如 **168 MHz**，会被 `SystemCoreClock` 记录下来。
这样 FreeRTOS 里用 `(SystemCoreClock / configTICK_RATE_HZ)` 就能得到正确的 `SysTick` 装载值。

---

# 🧩 六、中断优先级配置

SysTick 作为系统节拍中断，优先级应设置为：

* **最低（数值最大）**；
* 保证它不会打断其它更重要的中断（如外设 DMA、通信等）。

通常写在 `FreeRTOSConfig.h`：

```c
#define configKERNEL_INTERRUPT_PRIORITY     ( 15 << (8 - __NVIC_PRIO_BITS) )
#define configMAX_SYSCALL_INTERRUPT_PRIORITY ( 5 << (8 - __NVIC_PRIO_BITS) )
```

---

# ✅ 七、总结逻辑链

| 层级              | 内容                                         | 谁来设定                        |
| --------------- | ------------------------------------------ | --------------------------- |
| **MCU 硬件时钟树**   | HSE、PLL、分频等                                | 你在 `SystemClock_Config()` 配 |
| **HAL 层变量**     | `SystemCoreClock` 保存主频                     | 自动更新（SystemCoreClockUpdate） |
| **FreeRTOS 配置** | `configSYSTICK_CLOCK_HZ = SystemCoreClock` | 你在 `FreeRTOSConfig.h` 配     |
| **RTOS 内核**     | 用这个值计算 SysTick 装载寄存器                       | FreeRTOS 内部完成               |
| **结果**          | SysTick 产生周期中断（Tick）                       | 驱动任务调度                      |

---

