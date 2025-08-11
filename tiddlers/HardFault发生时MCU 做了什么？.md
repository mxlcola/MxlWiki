## HardFault发生时 MCU 做了什么？

当异常产生且被处理器接受时，压栈流程会将寄存器压入栈中，压栈操作中的栈可以为主栈（使用主栈指针，[[MSP]]）或者进程栈（使用进程栈，MSP）。若处理器运行在线程模式且使用 MSP （CONTROL 寄存器的第 1 位为 0，默认配置），则压栈操作在执行时使用主栈 MSP，如下图所示。

![](https://edit.wpgdadawant.com/uploads/news_file/blog/2021/4484/tinymce/1.png)  
图 1. 使用主栈的异常栈帧

若处理器运行在线程模式且使用 进程栈（CONTROL 寄存器的第 1 位为 1），则压栈操作在执行时使用进程栈 [[PSP]]，如下图所示。

![](https://edit.wpgdadawant.com/uploads/news_file/blog/2021/4484/tinymce/2.png)  
图 2. 使用进程栈的异常栈帧

对于未使用 OS 的多数应用，只会用到主栈指针 MSP，若应用使用了 PSP，则需检查连接寄存器 LR 的第二位来确定实际使用的 SP，如下图。

![](https://edit.wpgdadawant.com/uploads/news_file/blog/2021/4484/tinymce/3.png)  
图 3. 栈跟踪流程定位栈帧和压栈寄存器