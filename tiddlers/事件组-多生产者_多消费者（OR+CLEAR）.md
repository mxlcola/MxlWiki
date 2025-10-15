## 多生产者/多消费者（OR+CLEAR）

* 多个中断/任务向同一事件组的**不同位**发信号：

  * 发送：`rt_event_send(&evt, BIT_UART | BIT_SPI);`
  * 接收端按 OR 等待**任意一位**，并用 `CLEAR` 消耗命中位：

    ```c
    rt_event_recv(&evt, BIT_UART | BIT_SPI,
                  RT_EVENT_FLAG_OR | RT_EVENT_FLAG_CLEAR,
                  RT_WAITING_FOREVER, &got);
    if (got & BIT_UART) { /* 处理 UART 事件 */ }
    if (got & BIT_SPI ) { /* 处理 SPI 事件  */ }
    ```
* 这样不会把“别的接口的事件”误消费掉，因为 CLEAR 只清**命中的位**。