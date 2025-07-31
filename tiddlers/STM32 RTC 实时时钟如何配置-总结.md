

[STM32](https://zhida.zhihu.com/search?content_id=236442127&content_type=Article&match_order=1&q=STM32&zhida_source=entity) 里面 [RTC 模块](https://zhida.zhihu.com/search?content_id=236442127&content_type=Article&match_order=1&q=RTC%E6%A8%A1%E5%9D%97&zhida_source=entity)和时钟配置系统 (RCC_BDCR 寄存器) 处于后备区域，即在系统复位或从待机模式唤醒后，RTC 的设置和时间维持不变。因为系统对后备寄存器和 RTC 相关寄存器有写保护，所以如果想要对后备寄存器和 RTC 进行访问，则需要通过操作相应的寄存器来解除某些限制。

文章有点长，想要理解 RTC 原理的可以认真阅读。如果只想要源码的，可以直接拉到文章最下面

接下来进入正题：

一、解除寄存器操作限制
-----------

第一步首先需要设置 [RCC_APB1ENR](https://zhida.zhihu.com/search?content_id=236442127&content_type=Article&match_order=1&q=RCC_APB1ENR&zhida_source=entity) 的 PWREN 和 BKPEN 位。使能电源和后备接口时钟

![](https://mxloss112233.oss-cn-beijing.aliyuncs.com/imgv2-da7fabe84aa2151ec7285967f37f4f63_r.jpg)

第二步是设置寄存器 [PWR_CR](https://zhida.zhihu.com/search?content_id=236442127&content_type=Article&match_order=1&q=PWR_CR&zhida_source=entity) 的 DBP 位，使能对后备寄存器和 RTC 的访问

![](https://mxloss112233.oss-cn-beijing.aliyuncs.com/imgv2-e0c0abcf016e5b87e94fe40da153b9d8_r.jpg)

二、配置 RTC
--------

完成上面两步之后，我们就可以操作后备寄存器和 RTC 相关的寄存器了。但我们先来看看 RTC 的简单框图吧

![](https://mxloss112233.oss-cn-beijing.aliyuncs.com/imgv2-4756baa19e84d21ee872906864d81069_r.jpg)

从图 3 可以看出来，要想让 RTC 工作，得让它先有一个时钟，也就是图中 RTCCLK 时钟的选择。

### 1、RTC 时钟的选择

[RTCCLK 时钟源](https://zhida.zhihu.com/search?content_id=236442127&content_type=Article&match_order=1&q=RTCCLK%E6%97%B6%E9%92%9F%E6%BA%90&zhida_source=entity)可以由 HSE/128、LSE 或 LSI 时钟提供。除非备份域复位，此选择不能被改变。

[LSE 时钟](https://zhida.zhihu.com/search?content_id=236442127&content_type=Article&match_order=1&q=LSE%E6%97%B6%E9%92%9F&zhida_source=entity)在备份域里，但 HSE 和 LSI 时钟不是。因此：

● 如果 LSE 被选为 RTC 时钟：

─ 只要 V BAT 维持供电，_**尽管 V DD 供电被切断，RTC 仍继续工作**_。

● 如果 LSI 被选为自动唤醒单元 (AWU) 时钟：

─ 如果 V DD 供电被切断， AWU 状态不能被保证。

● 如果 HSE 时钟 128 分频后作为 RTC 时钟：

─ 如果 V DD 供电被切断或内部电压调压器被关闭 (1.8V 域的供电被切断)，则 RTC 状态不确定。

RTC 的时钟源有三个，但只有 LSE（外部低速振荡器，一般为 32.678kHz）在 VDD 供电被切断后，仍能继续工作，因此我们一般都选择它。

RTC 时钟源的选择需要操作备份域控制寄存器（RCC_BDCR）

![](https://pic1.zhimg.com/v2-b15c21bcf04e8fbdcf3ea9b5bfa7a27a_r.jpg)

配置时钟步骤

1）打开外部振荡器（LSEON 置 1）

2）然后等待 LSE 就绪，也就是等待 LSERDY 置 1

3）选择 RTC 时钟源，也就是配置 RTCSEL[1:0]

4) 使能 RTC 时钟（RTCEN 置 1）

### 2、配置 RTC 相关寄存器

从 RTC 框图可以知道，RTC 时钟选择后就应该配置 RTC 预分频器 ([RTC_PRL 寄存器](https://zhida.zhihu.com/search?content_id=236442127&content_type=Article&match_order=1&q=RTC_PRL%E5%AF%84%E5%AD%98%E5%99%A8&zhida_source=entity)）和 [RTC_CNT 计数器](https://zhida.zhihu.com/search?content_id=236442127&content_type=Article&match_order=1&q=RTC_CNT%E8%AE%A1%E6%95%B0%E5%99%A8&zhida_source=entity)和闹钟计数器 RTC_ALR。

一般我们通过预分频器将 RTCCLK 的时钟进行分频，让预分频器的输出时钟 TR_CLK 的频率变成 1Hz，也就是周期为 1s。然后 RTC_CNT 在 TR_CLK 频率下递增。如果 RTC_CNT 里面的值和 RTC_ALR 里面的数值相等，则会触发闹钟标志，即 ALRF 标志位置 1。在每个 TR_CLK 的周期都会触发一次秒标志，即 SECF 标志位会置 1.

一般如果用于时钟时钟的话，RTC_CNT 可以设置为当前的时间。

如果需要配置 RTC 的 RTC_PRL、RTC_CNT、RTC_ALR 寄存器。则必须判断 RTC 寄存器是否处于更新中，只有 RTC 寄存器不是处于跟新中才可以进行配置，可以通过 RTC_CR 寄存器里面的 RTOFF 位来判断。在配置前还必须将 RTC_CRL 寄存器里面的 CNF 位置 1，进入配置模式，等待配置后，还要退出配置模式。

配置过程

1）查询 RTOFF 位，直到 RTOFF 的值变为‘1’

2）置 CNF 为 1，进入配置模式

3）对一个或多个 RTC 寄存器进行写操作

4）清除 CNF 标志位，退出配置模式

5）查询 RTOFF, 直到 RTOFF 变为 1，才代表写操作完成

_注意：只有当 CNF 标志位被清除时，写操作才能进行，这个过程至少需要 3 个 RTCCLK 周期_

在正式进入配置之前我们先来看看 RTC 几个寄存器

**RTC 控制寄存器高位 RTC_CRH**

![](https://mxloss112233.oss-cn-beijing.aliyuncs.com/imgv2-bf1e37c2ce691c94ad42f2c7d4a87616_r.jpg)

这几位是用来使能中断的，可以配合前面的 RTC 框图 “食用”

**RTC 控制寄存器低位 RTC_CRL**

![](https://mxloss112233.oss-cn-beijing.aliyuncs.com/imgv2-240d5778a968a6e3b3934761267a7705_r.jpg)![](https://mxloss112233.oss-cn-beijing.aliyuncs.com/imgv2-b2fe2082785d0c40d8679fa0792f4c18_r.jpg)

_注意：标志位都需要由软件清零_

**RTC 预分频转载寄存器（RTC_PRLH/RTC_PRLL）**

该寄存器是用于配置预分频器的分频比的，只有前 20 位有效，即 PRL[19:0] 有效，总共 20 位。

时钟计算公式 fTR_CLK = fRTCCLK /(PRL[19:0]+1)。

当 LSE 位 32.678kHz 时，只需将 RTC_PRLL 配置成 32677 即可。

**RTC 计数器寄存器（RTC_CNTH/RTC_CNTL）**

该 32 寄存器可以通过配置来设定初值，并且在 TR_CLK 的基准下进行计数

**RTC 闹钟寄存器 (RTC_ALRH/RTC_ALRL)**

该 32 位寄存器用来配置闹钟的数值。

现在基本知识框架都已经介绍好了正式进入配置阶段

首先我先给出直接操作寄存器的版本，后面我也会给出操作固件库的版本。

我相信通过前面的讲解，直接操作寄存器反而会更简单！代码也很容易看懂

```
//寄存器版本
void RTC_Init(void)
{
//这里是第一步解除写保护
	RCC->APB1ENR |= RCC_APB1ENR_PWREN;//电源接口时钟使能
	RCC->APB1ENR |= RCC_APB1ENR_BKPEN;//备份接口时钟开启
	PWR->CR |= PWR_CR_DBP;//允许写入RTC和后备寄存器
//这里是第二步进入配置
	RCC->BDCR |= RCC_BDCR_LSEON;//打开外部32kHz振荡器
	while(!(RCC->BDCR & RCC_BDCR_LSERDY));//等待外部32kHz振荡器就绪
	RCC->BDCR |= RCC_BDCR_RTCSEL_LSE;//选择外部32kHz振荡器作为RTC时钟源
	RCC->BDCR |= RCC_BDCR_RTCEN;//RTC时钟使能
	RCC->BDCR |= RCC_BDCR_BDRST;//复位整个备份域
	while(!(RTC->CRL &RTC_CRL_RTOFF));//等待上一次写操作完成
	RTC->CRL |= RTC_CRL_CNF;//进入配置模式
	RTC->PRLL = 32767;//fTR_CLK = fRTCCLK /(PRL[19:0]+1),周期为1Hz
	RTC->CNTL = 0;//配置当前时间
	RTC->CNTH = 0;
	RTC->ALRH = 0;//配置闹钟时间
	RTC->ALRL = 2;
	RTC->CRH |= (RTC_CRH_ALRIE + RTC_CRH_SECIE);//使能秒中断和闹钟中断	
	RTC->CRL &= ~(RTC_CRL_CNF);//退出配置模式
}

```

下面给出的是操作固件库的版本

```
void RTC_Init(void)
{
//这里是第一步解除写保护
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_BKP | RCC_APB1Periph_PWR,ENABLE);//电源接口时钟使能和备份接口时钟开启
	PWR_BackupAccessCmd(ENABLE);//允许写入RTC和后备寄存器
//这里是第二步进入配置
	RCC_LSEConfig(RCC_LSE_ON);//打开外部32kHz振荡器
	while((RCC_GetFlagStatus(RCC_FLAG_LSERDY))== RESET);//等待外部32kHz振荡器就绪
	RCC_RTCCLKConfig(RCC_RTCCLKSource_LSE);//选择外部32kHz振荡器作为RTC时钟源
	RCC_RTCCLKCmd(ENABLE);//RTC时钟使能
	BKP_DeInit();//复位整个备份域
	RTC_WaitForLastTask();//等待上一次写操作完成
	RTC_SetPrescaler(32767);//fTR_CLK = fRTCCLK /(PRL[19:0]+1),周期为1Hz
	RTC_WaitForLastTask();//等待上一次写操作完成
	RTC_SetCounter(0);//配置当前时间
	RTC_WaitForLastTask();//等待上一次写操作完成
	RTC_SetAlarm(2);//配置闹钟时间
	RTC_WaitForLastTask();//等待上一次写操作完成
	RTC_ITConfig(RTC_IT_ALR | RTC_IT_SEC,ENABLE);//使能秒中断和闹钟中断
}

```

这里要注意一点，固件库里面设置预分频、计数值、闹钟数值的函数里面已经封装好有进入配置模式和退出配置模式，所以在操作固件库函数时，只需判断上一次操作是否完成即可，不需要再加进入配置模式和退出模式的命令。

### 三、配置中断

配置 [NVIC 通道](https://zhida.zhihu.com/search?content_id=236442127&content_type=Article&match_order=1&q=NVIC%E9%80%9A%E9%81%93&zhida_source=entity)

```c
void RTC_IRQ_Init(void)
{
	NVIC_PriorityGroupConfig (NVIC_PriorityGroup_3);//配置中断
	NVIC_InitTypeDef NVIC_InitStructure;
	NVIC_InitStructure.NVIC_IRQChannel = RTC_IRQn;
	NVIC_InitStructure.NVIC_IRQChannelCmd = ENABLE;
	NVIC_InitStructure.NVIC_IRQChannelPreemptionPriority = 0;
	NVIC_InitStructure.NVIC_IRQChannelSubPriority = 0;
	NVIC_Init(&NVIC_InitStructure);
}

```

接下来是配置 RTC 的中断函数

同样是先给出直接操作寄存器的版本，在中断函数里面我设置了两个标志位用来观察 RTC 是否如我们预想的在运行。因此我在 DEBUG 模式下，调用逻辑分析仪来观察这两个标志位。关于逻辑分析仪怎么使用，可以看我之前的一篇文章：[Keil 怎么用好 DEBUG 功能——逻辑分析仪的使用 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/667393282)

```
void RTC_IRQHandler(void)
{
	if((RTC->CRL & RTC_CRL_SECF))
	{
		RTC->CRL &= ~(RTC_CRL_SECF);//清除标志位
		if(SEC_Flag == 0)
		{
			SEC_Flag = 1;
		}
		else
		{
			SEC_Flag = 0;
		}
	}
	if((RTC->CRL & RTC_CRL_ALRF))
	{
		RTC->CRL &= ~(RTC_CRL_ALRF);//清除标志位
		if(ALM_Flag == 0)
		{
			ALM_Flag = 1;
		}
		else
		{
			ALM_Flag = 0;
		}
		/*重新设置闹钟时间*/
		while(!(RTC->CRL &RTC_CRL_RTOFF));//等待上一次写操作完成
		RTC->CRL |= RTC_CRL_CNF;//进入配置模式
		RTC->CNTL = 0;//配置当前时间
		RTC->CNTH = 0;
		RTC->ALRH = 0;//配置闹钟时间
		RTC->ALRL = 2;
		RTC->CRL &= ~(RTC_CRL_CNF);//退出配置模式
	}
}

```

接下来给出操作固件库的版本

```
void RTC_IRQHandler(void)
{
	if((RTC_GetITStatus(RTC_IT_SEC) == SET))
	{
		RTC_ClearITPendingBit(RTC_IT_SEC);//清除标志位
		if(SEC_Flag == 0)
		{
			SEC_Flag = 1;
		}
		else
		{
			SEC_Flag = 0;
		}
	}
	if((RTC_GetITStatus(RTC_IT_ALR)) == SET)
	{
		RTC_ClearITPendingBit(RTC_IT_ALR);//清除标志位
		if(ALM_Flag == 0)
		{
			ALM_Flag = 1;
		}
		else
		{
			ALM_Flag = 0;
		}
		/*重新设置闹钟时间*/
		RTC_WaitForLastTask();//等待上一次写操作完成
		RTC_SetCounter(0);//配置当前时间
		RTC_WaitForLastTask();//等待上一次写操作完成
		RTC_SetAlarm(2);//配置闹钟时间
		RTC_WaitForLastTask();//等待上一次写操作完成
	}
}

```

_这里有一点要特别注意，就是对中断标志位的清除只能由软件清除，且必须清除。否则程序会卡在中断函数里面。_

好了，现在我们可以来看看运行的结果了

![](https://mxloss112233.oss-cn-beijing.aliyuncs.com/imgv2-38bf77dddfffe7fdd459d95438122190_r.jpg)

从逻辑分析仪的结果来看，程序如我们预想那样。每隔一秒触发一次秒中断，SEC_Flag 标志位周期性变化。每隔两秒触发一次闹钟中断，ALM_Flag 标志位周期性变化。

**好啦，到这里 RTC 的配置和仿真就完成了，如果你觉得有用的话，就点个爱心和关注吧！**