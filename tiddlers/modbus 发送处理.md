##  modbus 发送处理

```c
BOOL
xMBTCPPortSendResponse( const UCHAR * pucMBTCPFrame, USHORT usTCPLength )
{
    memcpy( ucTCPResponseFrame , pucMBTCPFrame , usTCPLength);
    ucTCPResponseLen = usTCPLength;
   
    bFrameSent = TRUE; // 通过uip_poll发送数据
    return bFrameSent;
}
```
把传入的内容 pucMBTCPFrame复制给ucTCPResponseFrame，并设置bFrameSent为True，那么在下一次uip_poll时便会把响应发送会主机。

发送数据时候，参考数据监听部分，发送数据**bFrameSent**的状态
[[数据监听处理]]
