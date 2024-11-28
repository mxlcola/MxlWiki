关联：

[[SD卡引脚定义]]

标准SD卡有九个对外的触点，SD模式和SPI模式下的引脚定义不同。

- DAT线和CMD线需要接[[上拉电阻]]。
- CD线在power up后能够作为SD模式的“卡检测”端口或[[SPI]]模式的片选端。
- RESERVED引脚也需外接上拉电阻，避免浮空输入导致功耗的增加。




**DAT3引脚**被用作检测SD卡的插入状态的输入引脚，那么DAT3引脚需要通过上拉电阻连接到电源电压，以确保在SD卡未插入时DAT3引脚的电平为高电平。

当SD卡插入时，SD卡会将DAT3引脚拉低，从而检测到SD卡的插入状态。

![img](https://tc8483.oss-cn-beijing.aliyuncs.com/image/v2-6fd5914d0a1df1b06f2c750d8142764d_r.jpg)