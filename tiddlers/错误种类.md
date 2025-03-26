对于 Cortex-M 内核，架构采用错误异常的机制来检测问题，当核心检测到一个错误时，异常中断会被触发，并且核心会跳转到相应的异常终端处理函数执行，错误异常的终端分为以下四种：

1. HardFault  
1. MemManage  
1. BusFault  
1. UsageFault