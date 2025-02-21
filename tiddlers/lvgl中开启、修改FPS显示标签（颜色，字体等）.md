lvgl中的FPS显示开关在lv_conf.c文件中。

启用LV_USE_PERF_MONITOR，即可打开FPS显示。

启用LV_USE_MEM_MONITOR，即可打开内存占用显示

![img](https://tc8483.oss-cn-beijing.aliyuncs.com/image/f186bbe3897bfb78689d74e91dedda62.png)

具体显示内容的程序在lv_refr.c文件（lvgl\src\core）中417行。找到了显示标签的地方，修改标签的具体显示就很简单了。

![img](https://tc8483.oss-cn-beijing.aliyuncs.com/image/0c864ea3bed94a9598ba3d54c936cfde.png)