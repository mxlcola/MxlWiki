# 时间线（一步步发生了什么）

1. **某处触发 PendSV**（通常在 SysTick 里决定该切换任务了）。
2. **硬件**异常入口：把 r0–r3、r12、lr、pc、xPSR 压到**当前 PSP**，然后切到 **MSP**，跳到 `xPortPendSVHandler`。
3. **软件**保存：`r4–r11` → 压到 PSP；更新 TCB 里的“top of stack”。
4. **软件**临界区：提高 BASEPRI，`vTaskSwitchContext()` 决定下一个任务，更新 `pxCurrentTCB`；再放开 BASEPRI。
5. **软件**恢复：从**新任务**的 PSP 弹出 `r4–r11`，把 PSP 指到“硬件帧”的位置。
6. `bx r14`（EXC_RETURN）→ **硬件**出栈“硬件帧”，恢复新任务的 r0–r3、r12、lr、pc、xPSR，回到线程态继续跑。

------