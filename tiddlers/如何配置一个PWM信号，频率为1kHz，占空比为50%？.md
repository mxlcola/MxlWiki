<img src="https://tc8483.oss-cn-beijing.aliyuncs.com/image/image-20241204191518532.png" alt="image-20241204191518532" style="zoom: 50%;" />

```c
TIM_HandleTypeDef htim1;

void TIM1_PWM_Init() {
    __HAL_RCC_TIM1_CLK_ENABLE(); // 启用TIM1时钟

    htim1.Instance = TIM1;
    htim1.Init.Prescaler = 71; // 1MHz计数频率，假设时钟为72MHz 预分频
    htim1.Init.CounterMode = TIM_COUNTERMODE_UP; // 向上计数
    htim1.Init.Period = 999; // PWM频率1kHz
    htim1.Init.ClockDivision = TIM_CLOCKDIVISION_DIV1;
    HAL_TIM_PWM_Init(&htim1); // 初始化TIM1
}

```