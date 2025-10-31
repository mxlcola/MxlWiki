## 核心: 伪造现场

调用过程：

```shell
xTaskCreate / xTaskCreateStatic
	prvInitialiseNewTask
		pxPortInitialiseStack			
```

![image-20220407093010592](https://mxloss112233.oss-cn-beijing.aliyuncs.com/img/36_pxPortInitialiseStack.png)

## 伪造之前需要设置栈顶，注意字节对齐

一部分寄存器是硬件处理，一部分是软件处理

```c
 pxTopOfStack = &( pxNewTCB->pxStack[ ulStackDepth - ( uint32_t ) 1 ] );

pxTopOfStack = ( StackType_t * ) ( ( ( portPOINTER_SIZE_TYPE ) pxTopOfStack ) & ( ~( ( portPOINTER_SIZE_TYPE ) 
```


```c

StackType_t * pxPortInitialiseStack( StackType_t * pxTopOfStack,
                                     TaskFunction_t pxCode,
                                     void * pvParameters )
{
    /* Simulate the stack frame as it would be created by a context switch
     * interrupt. */
    pxTopOfStack--;                                                      /* Offset added to account for the way the MCU uses the stack on entry/exit of interrupts. */
    *pxTopOfStack = portINITIAL_XPSR;                                    /* xPSR */
    pxTopOfStack--;
    *pxTopOfStack = ( ( StackType_t ) pxCode ) & portSTART_ADDRESS_MASK; /* PC */
    pxTopOfStack--;
    *pxTopOfStack = ( StackType_t ) prvTaskExitError;                    /* LR */

    pxTopOfStack -= 5;                                                   /* R12, R3, R2 and R1. */
    *pxTopOfStack = ( StackType_t ) pvParameters;                        /* R0 */
    pxTopOfStack -= 8;                                                   /* R11, R10, R9, R8, R7, R6, R5 and R4. */

    return pxTopOfStack;
}

```

[img width = 600 [https://mxloss112233.oss-cn-beijing.aliyuncs.com/img/20251030195022.png]]