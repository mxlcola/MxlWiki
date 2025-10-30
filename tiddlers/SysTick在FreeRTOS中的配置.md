# 🔩 SysTick 的配置（以 FreeRTOS 为例）

在端口层函数 `vPortSetupTimerInterrupt()` 中：

```c
portNVIC_SYSTICK_LOAD_REG = ( configSYSTICK_CLOCK_HZ / configTICK_RATE_HZ ) - 1UL;
portNVIC_SYSTICK_CTRL_REG = ( portNVIC_SYSTICK_CLK_BIT |
                              portNVIC_SYSTICK_INT_BIT |
                              portNVIC_SYSTICK_ENABLE_BIT );
```

例如：

* CPU 时钟 72 MHz，
* `configTICK_RATE_HZ = 1000`（即 1 ms/次），
  则：

```
LOAD = 72000 - 1
```

表示 SysTick 每过 72000 个 CPU 时钟周期触发一次中断。

---