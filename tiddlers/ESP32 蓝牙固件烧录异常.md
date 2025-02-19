ESP32 蓝牙固件烧录异常复盘

## 异常现象

ESP32开发板烧录固件时，点击“ERASE”后，显示“**等待上电同步**”

原解决办法：

按开发板上的“BOOT”按钮1秒左右即可；

![image-20250219113912084](https://tc8483.oss-cn-beijing.aliyuncs.com/image/image-20250219113912084.png)

## 引脚配置

和蓝牙烧录相关的引脚:

- WIFI_POWER
- WIFI_EN
- POWER_DOWN_MCU

重点是 **POWER_DOWN_MCU引脚**，作用：

**mcu给外设进行供电**，所以需要初始化

<img src="https://tc8483.oss-cn-beijing.aliyuncs.com/image/image-20250219114006641.png" alt="image-20250219114006641" style="zoom:80%;" />

<img src="https://tc8483.oss-cn-beijing.aliyuncs.com/image/image-20250219114209922.png" alt="image-20250219114209922" style="zoom:80%;" />

相关代码

<img src="https://tc8483.oss-cn-beijing.aliyuncs.com/image/image-20250219114346817.png" alt="image-20250219114346817" style="zoom: 80%;" />