## 从 Bootloader 跳转到 APP 的完整流程

结合代码 [main.c:268-331](vscode-webview://1g6e2e4h5pq3rdk2egrvuj6ot7gjd70lj8mapg40tb2tevrve3j6/main.c#L268-L331)，整个过程分为 **三个阶段**：

---

### 一、初始化 & 升级检查

```
上电 → 硬件从 0x08000000 启动 Bootloader
  │
  ├── MPU_Config()            配置内存保护（你选中的这段代码）
  ├── HAL_Init()              HAL 库初始化
  ├── SystemClock_Config()    HSE + PLL → 480MHz
  ├── IWDG_Init()             看门狗
  ├── GPIO / UART 初始化       调试用
  ├── W25Q64_Init()           外部 Flash
  ├── Mcu_FloadLoaderInit()   读升级标志
  │
  └── Mcu_SystemUpdate()      有升级就执行，没有就跳过
```

---

### 二、跳转前的环境清理

这一步**最关键**，目的是让硬件恢复到接近复位的状态：

```c
// ① 释放外设
W25Q64_DeInit();          // QSPI 外设释放
HAL_RCC_DeInit();         // 时钟切回 HSI，关闭 PLL
MX_UART5_DeInit();        // 串口释放

// ② 关中断
__set_PRIMASK(1);         // 全局中断屏蔽

// ③ 停 SysTick
SysTick->CTRL = 0;       // 关闭定时器
SysTick->LOAD = 0;       // 清重装值
SysTick->VAL  = 0;       // 清计数值

// ④ 清除 NVIC（所有中断使能 + 挂起标志）
for (i = 0; i < 8; i++) {
    NVIC->ICER[i] = 0xFFFFFFFF;   // 关闭中断使能
    NVIC->ICPR[i] = 0xFFFFFFFF;   // 清除中断挂起
}

// ⑤ 开中断
__set_PRIMASK(0);
```

**为什么要这么做？**

```
如果不清理直接跳：

  SysTick 还在跑 → 触发中断 → 但此时 MSP 已切换 → HardFault
  某外设中断挂起 → APP 还没初始化 NVIC → 跳到野地址 → HardFault
  PLL 还在锁定  → APP 重新配 PLL 失败 → 卡死
```

---

### 三、跳转执行

[main.c:103-122](vscode-webview://1g6e2e4h5pq3rdk2egrvuj6ot7gjd70lj8mapg40tb2tevrve3j6/main.c#L103-L122)：

```
APP 在 0x08020000 的中断向量表：
┌──────────────┬──────────────────────┐
│ 0x08020000   │ 初始 MSP（栈顶）       │  ← 第 0 项
│ 0x08020004   │ Reset_Handler 地址    │  ← 第 1 项
│ 0x08020008   │ NMI_Handler          │
│ ...          │ ...                  │
└──────────────┴──────────────────────┘
```

```c
void Jump_ToApplication(0x08020000)
{
    // ① 校验：栈指针是否指向 AXI SRAM (0x24000000)
    if ((*(0x08020000) & 0x24000000) == 0x24000000)
    
    // ② 取 Reset_Handler 地址
    jumpAddress = *(0x08020004);
    
    // ③ 设 MSP = APP 的栈顶
    __set_MSP(*(0x08020000));
    
    // ④ 切特权级 + MSP 模式
    __set_CONTROL(0);
    
    // ⑤ 跳转，永不返回
    pApplicationFun();
}
```

本质上是**用软件模拟 CPU 上电复位的动作**：硬件复位时 CPU 自动从向量表取 MSP 和 Reset_Handler，这里用代码手动做了同样的事。

---

### 四、APP 启动后要做什么

APP 端的 `SystemInit` 或 `main` 入口必须：

```c
// 重定向中断向量表，否则中断还会跳到 Bootloader 的 handler
SCB->VTOR = 0x08020000;
```

---

### 完整时序图

```
Bootloader (0x08000000)                          APP (0x08020000)
     │                                                 │
     ├─ 初始化硬件                                       │
     ├─ 检查升级标志                                      │
     ├─ (有升级则执行)                                    │
     │                                                  │
     ├─ DeInit 外设                                      │
     ├─ 时钟切回 HSI                                     │
     ├─ 关中断                                           │
     ├─ 停 SysTick                                      │
     ├─ 清 NVIC                                         │
     ├─ 开中断                                           │
     │                                                  │
     ├─ 校验 APP 栈指针                                   │
     ├─ __set_MSP(APP栈顶)  ─────── 栈切换 ──────────→   │
     ├─ __set_CONTROL(0)                                │
     ├─ 跳转 Reset_Handler  ─────── 控制权移交 ──────→   │
     │  (永不返回)                                       ├─ SCB->VTOR = 0x08020000
                                                        ├─ SystemClock_Config()
                                                        ├─ 外设初始化
                                                        ├─ main 循环
                                                        └─ ...
```