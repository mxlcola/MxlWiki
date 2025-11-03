# vPortPendSVHandler代码逐行讲解（带关键点）

```asm
__asm void xPortPendSVHandler( void )
{
    extern uxCriticalNesting;
    extern pxCurrentTCB;
    extern vTaskSwitchContext;

    PRESERVE8
```

- `PRESERVE8`：要求栈 8 字节对齐（ARM AAPCS 约定）。异常入栈和浮点扩展都需要对齐，避免硬件对齐故障。

```asm
    mrs r0, psp
    isb
```

- `mrs r0, psp`：把 **PSP（任务栈）**取到 r0。注意现在在 **Handler mode**，代码本身用的是 **MSP**，但我们要操作任务自己的栈，所以单独把 PSP 读出来。
- `isb`：指令同步屏障，确保后续指令看到最新的系统状态（这里属于保守做法，常配合特权/栈切换等）。

```asm
    ldr r3, =pxCurrentTCB     /* 取 pxCurrentTCB 的地址 */
    ldr r2, [ r3 ]            /* r2 = *pxCurrentTCB（当前任务 TCB 指针）*/
```

- `pxCurrentTCB` 指向**当前任务**的 TCB。**TCB 的第一个成员就是“任务栈顶指针（top of stack）”。**

```asm
    stmdb r0!, { r4 - r11 }   /* 把 r4–r11 压到 PSP（任务栈），栈向下生长 */
    str r0, [ r2 ]            /* 把更新后的 PSP 写回 TCB 的第一个成员 */
```

- **为什么只存 r4–r11？** 因为 **Cortex-M 硬件**在异常入口时，已经自动把 **r0–r3、r12、lr、pc、xPSR** 压到 PSP 了（这就是“硬件帧”）。
- **r4–r11** 是 AAPCS 里的**被调用者保存寄存器（callee-saved）**，硬件不管，**上下文切换时由内核保存**。
- `stmdb r0!` 使用了 **Full Descending** 模式并**写回**，所以 r0 最后变成“新的栈顶”。

```asm
    stmdb sp!, { r3, r14 }    /* 现在用的是 MSP，把 r3（pxCurrentTCB 地址）和 r14 保存一下 */
```

- **注意这里用的是 `sp`（MSP）**，因为我们在异常里。
- 为何要存 r3、r14？
  - r3 里放着 `&pxCurrentTCB`，等会儿调用函数会破坏它；
  - r14 是异常里的 **LR（=EXC_RETURN 值）**，返回路径的关键信息，等会儿也要用。

```asm
    mov r0, #configMAX_SYSCALL_INTERRUPT_PRIORITY
    msr basepri, r0
    dsb
    isb
```

- **BASEPRI**：设置“屏蔽门限”。数值 **≥ BASEPRI** 的中断都被屏蔽，仅**更高优先级（更小数值）**的中断还能打断。
- 这让 `vTaskSwitchContext` 在“内核可打断但受控”的环境下运行，避免被那些可能会调用内核 API 的中断打乱状态。
- `dsb/isb`：数据/指令屏障，确保优先级屏蔽立刻生效，不被乱序或流水线影响。

```asm
    bl vTaskSwitchContext     /* 选下一个要运行的任务，更新 pxCurrentTCB */
```

- 这个 C 函数只做**调度决策**：把 `pxCurrentTCB` 指向**下一个任务的 TCB**。上下文的保存/恢复都在汇编里做。

```asm
    mov r0, #0
    msr basepri, r0           /* 取消 BASEPRI 屏蔽 */
    ldmia sp!, { r3, r14 }    /* 从 MSP 把 r3、r14 恢复 */
```

- 恢复我们刚刚压到 MSP 的临时寄存器。

```asm
    ldr r1, [ r3 ]            /* r3 仍是 &pxCurrentTCB */
    ldr r0, [ r1 ]            /* r0 = (*pxCurrentTCB)->topOfStack（下一个任务的 PSP）*/
```

- 现在 `pxCurrentTCB` 已经被切到**下一个任务**的 TCB 了，所以这两步就把**下一个任务的 PSP**（准确说是它保存的“栈顶”）取出来放在 r0。

```asm
    ldmia r0!, { r4 - r11 }   /* 从下一个任务的栈里弹出 r4–r11（软件保存的半包） */
    msr psp, r0               /* 把 PSP 切到下一个任务最新的栈顶 */
    isb
```

- 这一步是在**恢复**下一个任务的软件部分上下文（r4–r11），并把 PSP 指向“硬件帧”的栈顶。
- 再次 `isb`，确保接下来 `bx r14` 返回时使用的 PSP 是最新的。

```asm
    bx r14                    /* 用异常返回序列退回 Thread mode（依据 EXC_RETURN）*/
    nop
```

- **关键点**：`r14`（LR）在异常里不是普通返回地址，而是 **EXC_RETURN**，它编码了“返回到线程态/处理态、用 PSP/MSP、是否有浮点帧”等信息。
- 这条 `bx r14` 会触发 **硬件**自动把 **r0–r3、r12、lr、pc、xPSR**（也就是“硬件帧”）从 **PSP** 弹出来，于是控制权就跳到下一个任务的 **PC**，状态恢复到任务的 **xPSR**。上下文切换完成。

------