**定时器的时钟来源**：

- STM32的定时器时钟通常来源于APB1或APB2总线时钟（假设为`f_APB`）。
- 在默认情况下，定时器时钟频率是总线时钟的倍频（`f_tim = f_APB × 倍频因子`，通常是2倍或1倍，取决于芯片配置）。

**预分频器（Prescaler）的作用**：

- 预分频器的作用是将定时器输入时钟进一步分频，用于控制计数器的计数频率。
- 定时器的实际计数频率为：

![image-20241204192256304](https://tc8483.oss-cn-beijing.aliyuncs.com/image/image-20241204192256304.png)

**`Prescaler = 0` 表示什么**：

- 当`Prescaler = 0`时，分频因子为`Prescaler + 1 = 1`，即 **定时器的计数频率等于定时器时钟频率**。
- 此时，计数器每次递增的频率为`f_tim`。

<img src="https://tc8483.oss-cn-beijing.aliyuncs.com/image/image-20241204192337616.png" alt="image-20241204192337616" style="zoom:60%;" />