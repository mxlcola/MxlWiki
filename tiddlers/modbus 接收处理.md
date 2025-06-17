```c
BOOL
xMBTCPPortGetRequest( UCHAR ** ppucMBTCPFrame, USHORT * usTCPLength )
{
    *ppucMBTCPFrame = &ucTCPRequestFrame[0];
    *usTCPLength = ucTCPRequestLen;
   
    /* Reset the buffer. */
    ucTCPRequestLen = 0;
    return TRUE;
}
```

代码说明】

​    【1】 ppucMBTCPFrame为一个指向数据的指针，而*ppucMBTCPFrame可以指向一个数组，在这里可把ucTCPRequestFrame复制给该变量，配合usTCPLength，那么从uIP接收到的内容就”转移“到freemodbus