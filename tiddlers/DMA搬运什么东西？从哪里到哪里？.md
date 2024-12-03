** DMA搬运，[[外设]] 和存储器的数据**

##  DMA搬运的方向？
**存储器→存储器（例如：复制某特别大的数据buf）**

​    ![img](https://tc8483.oss-cn-beijing.aliyuncs.com/img/b753e1ecbe8a4b6ca49ee925e535a001.png)

​    **存储器→外设 （例如：将某数据buf写入串口TDR寄存器）**

​    ![img](https://tc8483.oss-cn-beijing.aliyuncs.com/img/8860ee6ee60c47099f9942860fa6ff4d.png)

​    **外设→存储器 （例如：将串口RDR寄存器写入某数据buf）**

​    ![img](https://tc8483.oss-cn-beijing.aliyuncs.com/img/7a6e1b3f4de0456b929e28bd6a7101fb.png)