```c
__asm void xPortPendSVHandler( void )
{
    extern uxCriticalNesting;
    extern pxCurrentTCB;
    extern vTaskSwitchContext;

/* *INDENT-OFF* */
    PRESERVE8

    mrs r0, psp
    isb

    ldr r3, =pxCurrentTCB /* Get the location of the current TCB. */
    ldr r2, [ r3 ]

    stmdb r0 !, { r4 - r11 } /* Save the remaining registers. */
    str r0, [ r2 ] /* Save the new top of stack into the first member of the TCB. */

    stmdb sp !, { r3, r14 }
    mov r0, #configMAX_SYSCALL_INTERRUPT_PRIORITY
    msr basepri, r0
    dsb
    isb
    bl vTaskSwitchContext
    mov r0, #0
    msr basepri, r0
    ldmia sp !, { r3, r14 }

    ldr r1, [ r3 ]
    ldr r0, [ r1 ] /* The first item in pxCurrentTCB is the task top of stack. */
    ldmia r0 !, { r4 - r11 } /* Pop the registers and the critical nesting count. */
    msr psp, r0
    isb
    bx r14
    nop
/* *INDENT-ON* */
}
```