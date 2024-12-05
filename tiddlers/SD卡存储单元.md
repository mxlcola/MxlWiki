读写SD卡数据的基本单位是1 byte，所有数据都是以''[[block]]''的形式来传输的。SDHC标准卡的数据块长度为512 bytes。每个[[sector扇区]]的block大小是固定的，定义在**CSD寄存器**中。[[SD卡寄存器]]

>block长度可以通过CSD寄存器自定义

![img](https://tc8483.oss-cn-beijing.aliyuncs.com/image/v2-4509860c5e65b234e44dcce988827534_720w.jpg)