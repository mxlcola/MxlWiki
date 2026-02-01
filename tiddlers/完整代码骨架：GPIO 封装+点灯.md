## 示例：GPIO 结构体封装 + PB5 点灯（STM32F1）

1) 用 struct 描述 GPIO 寄存器布局  
2) 用 GPIOB_BASE 强转成 GPIOB 指针  
3) 通过 RCC 使能 GPIOB 时钟  
4) 配 PB5 输出模式  
5) 用 BSRR/BRR 控制电平点灯

## 关键代码（最小骨架）
```c
#include <stdint.h>

typedef struct {
    volatile uint32_t CRL;   // 0x00
    volatile uint32_t CRH;   // 0x04
    volatile uint32_t IDR;   // 0x08
    volatile uint32_t ODR;   // 0x0C
    volatile uint32_t BSRR;  // 0x10
    volatile uint32_t BRR;   // 0x14
    volatile uint32_t LCKR;  // 0x18
} GPIO_TypeDef;

typedef struct {
    volatile uint32_t CR;       // 0x00
    volatile uint32_t CFGR;     // 0x04
    volatile uint32_t CIR;      // 0x08
    volatile uint32_t APB2RSTR; // 0x0C
    volatile uint32_t APB1RSTR; // 0x10
    volatile uint32_t AHBENR;   // 0x14
    volatile uint32_t APB2ENR;  // 0x18
    volatile uint32_t APB1ENR;  // 0x1C
    volatile uint32_t BDCR;     // 0x20
    volatile uint32_t CSR;      // 0x24
} RCC_TypeDef;

#define PERIPH_BASE     0x40000000UL
#define APB2PERIPH_BASE (PERIPH_BASE + 0x00010000UL)
#define GPIOB_BASE      (APB2PERIPH_BASE + 0x0C00UL)  // 0x40010C00
#define RCC_BASE        (APB2PERIPH_BASE + 0x1000UL)  // 0x40021000

#define GPIOB ((GPIO_TypeDef*)GPIOB_BASE)
#define RCC   ((RCC_TypeDef*)RCC_BASE)

static inline void led_init(void){
    RCC->APB2ENR |= (1U<<3);          // IOPBEN
    GPIOB->CRL &= ~(0xFU << 20);      // 清 PB5 配置位
    GPIOB->CRL |=  (0x2U << 20);      // 0b0010 输出2MHz推挽
}

