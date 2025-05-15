STM32 内部专门为 FSMC 功能预留了地址线、数据线和控制线的引脚，这些引脚一旦配置好 FSMC 模式，就能自动实现存储器的读写控制。

## FSMC读写数据的流程

**定义 FSMC 数据线对应的地址**

```c
//FSMC_Bank1_NORSRAM  
#define      FSMC_Addr_DATA        ( ( uint32_t ) 0x60020000 )

```

如果要写数据，直接给这个地址赋值就可以：

```c
/**
  * @brief  SRAM写入数据
  * @param  usData :要写入的数据
  * @retval 无
  */	
 void SRAM_Write_Data ( uint16_t usData )
{
	* ( __IO uint16_t * ) ( FSMC_Addr_DATA ) = usData;
	
}

```

**如果要从存储器中读取数据，直接读取这个地址就可以：**

```c
/**
  * @brief  从SRAM读取数据
  * @param  无
  * @retval 读取到的数据
  */	
 uint16_t SRAM_Read_Data ( void )
{
	return ( * ( __IO uint16_t * ) ( FSMC_Addr_DATA ) );	
}

uint16_t usR=0;

usR = SRAM_Read_Data (); 	/* READ DATA*/

```