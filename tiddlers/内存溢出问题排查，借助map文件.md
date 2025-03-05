## 内存溢出问题排查，借助map文件

现象：

进入校准界面之后，算法突然运行异常，示数一直保持不变

问题排查：

算法异常主要是由于算法参数被修改，导致不能正常工作



为什么算法参数被修改？

主要由于内存溢出导致,内存地址附近的区域影响

借助map文件找到 相邻区域

![image-20250305203916987](https://tc8483.oss-cn-beijing.aliyuncs.com/image/image-20250305203916987.png)



![image-20250305204603816](https://tc8483.oss-cn-beijing.aliyuncs.com/image/image-20250305204603816.png)

主要问题：

[[数组或指针越界访问]]

## 如何避免这个情况？优化方案？

使用snprintf处理

[[snprintf和sprintf 的区别]]