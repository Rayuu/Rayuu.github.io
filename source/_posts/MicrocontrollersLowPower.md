---
title: 瑞萨单片机系统低功耗的设计
date: 2016-08-13 22:57:16
tags: 瑞萨 低功耗 单片机
categories: 单片机
---


经过一周多的单片机设计，功能基本完成。但是发现运行功耗非常大，电流在5mA左右，无线发射和接收时更是高达15mA。

这和要求的功耗设计都不在一个数量级，要求的功耗是在微安级别的。所以需要一个方法降低系统的功耗。

<!--more-->

单片机系统的功耗是由MCU和其他外围电路的功耗共同决定。所以首先要对单片机系统的工作特点非常熟悉。
查阅手册发现，现在用的瑞萨单片机，有STOP,HALT,SNOOZE三种模式可以降低功耗。

以下是用户手册的说明

> (1) HALT 模式


通过执行 HALT 指令进入 HALT 模式。 HALT 模式是停止 CPU 运行时钟的模式。在设定 HALT 模式前，如果高速系统时钟振荡电路、高速内部振荡器或者副系统时钟振荡电路正在振荡，各时钟就继续振荡。虽然此模式无法让工作电流降到 STOP 模式的程度，但是在想要通过中断请求立即重新开始处理或者想要频繁地进行间歇运行时是一种有效的模式。

> (2) STOP 模式


通过执行 STOP 指令进入 STOP 模式。 STOP 模式是停止高速系统时钟振荡电路和高速内部振荡器的振荡并且停止整个系统的模式。能大幅度地降低 CPU 的工作电流。
因为 STOP 模式能通过中断请求来解除，所以也能进行间歇运行。但是，在 X1 时钟的情况下，因为在解除STOP 模式时需要确保振荡稳定的等待时间，所以如果一定要通过中断请求立即开始处理，就必须选择 HALT 模式。

> (3) SNOOZE 模式


通过 CSIp 或者 UARTq 的数据接收以及由定时器触发信号 （中断请求信号 （ INTRTC/INTIT））产生的 A/D转换请求，解除 STOP 模式，不需要 CPU 运行而进行 CSIp 或者 UARTq 的数据接收，或者进行 A/D 转换。只有在选择高速内部振荡器作为 CPU/ 外围硬件时钟 （ fCLK）时才能设定 SNOOZE 模式。


在任何一种模式中，寄存器、标志和数据存储器全部保持设定为待机模式前的内容，并且还保持输入 / 输出端口的输出锁存器和输出缓冲器的状态。

最后决定使用STOP模式来进行功耗设计，使功耗尽可能降低到最低。

以单片机为核心构成的系统，其系统的总能耗是由单片机能耗及其外围电路能耗共同构成。

所以为了降低整个系统的功耗，除了要降低单片机自身的运行恭号外，还要降低外围电路的功耗。

如果要在STOP模式下工作，首先要把单片机外围的IO设备最大程度上禁用。

!["STOP模式中的运行状态"](http://7xlqnm.com1.z0.glb.clouddn.com/2016/08/2578120507.png)

> 在本系统中，除了单片机，还有无线模块，蜂鸣器和一个LED灯耗电。


下面进行分步描述

### 1、首先要做的是对无线模块休眠，因为射频在工作的时候，电流高达15mA左右。

Si24R1芯片内部有状态机，控制着芯片在不同工作模式的切换。
Si24R1可配置为Shutdown,Standby,Idle-TX,TX,Rx五种工作模式。

![无线工作模式](http://7xlqnm.com1.z0.glb.clouddn.com/2016/08/4010025032.png)

为了降低无线模块的功耗，也要让无线模块在Shutdown模式下工作。

在Shutdown工作模式下，Si24R1所有收发功能模块关闭，芯片停止工作，消耗电流最小，
但所有内部寄存器值和FIFO值保持不变，仍可通过SPI实现对寄存器的读写。

设置CONFIG寄存器的PWR_UP位的值为0，芯片立即返回到Shutdown工作模式。

查看CONFIG寄存器的配置如下：

![CONFIG寄存器](http://7xlqnm.com1.z0.glb.clouddn.com/2016/08/3292061551.png)

PWR_UP在CONFIG寄存器的第一位，所以把该位拉低即可。

编写子函数如下：

```

	/******************************************************************
	
	SPI Shutdown 关断模式
	
	把config 配置寄存器的第一位PWR_up拉低
	
	********************************************************************/
	
	void SI24R1_Shutdown(void )
	
	{
	               
	 				SI24R1_CEL;
	
	                SPI_Write_Reg(NRF_WRITE_REG + CONFIG, 0x00);
	
	                SI24R1_Delay_ms(2);
	
	                SI24R1_CEH;
	
	}

```

然后在必要的时候需要对无线模块进行唤醒。所以编写无线开启子函数。

	/******************************************************************
	SPI 上电模式        
	********************************************************************/
	void SI24R1_PowerOn(void )
	{
	                SI24R1_CEL;
	                SPI_Write_Reg(NRF_WRITE_REG + CONFIG, 0x71);//发射模式
	                SI24R1_Delay_us(120);
	                SPI_Write_Reg(NRF_WRITE_REG + CONFIG, 0x3F);  // Set PWR_UP bit, enable CRC(2 bytes)
	                SI24R1_Delay_ms(2);
	                SI24R1_CEH;
	                
	                RX_Mode(WireComm.Addr,WireComm.CommChannel);
	                
	}

至此无线低功耗设计完成。

### 2、单片机I/O的处理


该系统对单片机I/O的利用有A/D转换，蜂鸣器，LED灯，EEPROM等。所以在进入STOP模式时，要把这些引脚给拉高。

蜂鸣器的话要根据硬件情况拉低或拉高。

### 3、编写STOP模式函数

进入STOP模式很简单，一条STOP();就可以进入，所以在进入STOP模式之前要把单片机系统外围的I/O进行处理。没有用到的I/O最好也要设为输出拉高。

但是没有用到I/O对电流影响不大。

对于唤醒stop模式，打开单片机的内部定时器，周期为200ms。根据用户手册进行时序设计。它可以用来唤醒STOP模式。

内部定时器是一个12位间隔定时器。我们按事先设定的任意时间间隔产生中断 （ INTIT），能用于从 STOP 模式的唤醒以及 A/D 转换器的 SNOOZE模式的触发。

在定时器里面设定无线为休眠状态，主函数里面调用STOP模式。

无线的状态为每5s唤醒一次，工作50ms，然后重新进入休眠。

定时器中断和STOP模式部分程序如下：

	/***********************************************************************************************************************
	* Function Name: r_it_interrupt
	* Description  : This function is INTIT interrupt service routine.
	* Arguments    : None
	* Return Value : None
	***********************************************************************************************************************/
	__interrupt static void r_it_interrupt( void)
	{
	    /* Start user code. Do not edit comment generated here */
	                 if(Envent.Si24R1Sleep< 0xF0) Envent.Si24R1Sleep++;
	                 if(Envent.DoorSleep< 0xF0) Envent.DoorSleep++;
	                 //按键扫描
	                KeyScan();
	               ....................................
	    /* End user code. Do not edit comment generated here */
	}



```

	/**********************************************************************************
	Function Name :   Stop_mode(void)
	Description             : 睡眠模式
	Parameters               : none
	Return value  : none
	***********************************************************************************/
	void Stop_mode(void )
	{
	                 if((Envent.Si24R1Work> 0) || (WIRREC !=WireComm.WorkSta) ) return ;
	                
	                SI24R1_Shutdown();
	                R_ADC_Set_OperationOff();
	                                
	                P6.0 =1;
	                P6.1 =1;
	                BELLOFF;
	                P2.2 =1;
	                P7.0 =1;
	                Envent.Si24R1Sleep =1;
	                 while(Envent.Si24R1Sleep)
	                {
	                                 STOP();
	                                 NOP();
	                                 NOP();
	                                 NOP();
	                                 NOP();
	                    ....................
	                                   ...............
	                                 if(Envent.DoorSleep>= 5)
	                                {
	                                                Envent.DoorSleep =0;
	                                                 //状态改变上报一次
	                                                 if(ReedSwitch!= Envent.DoorSta)
	                                                {
	                                                                Envent.DoorSta =ReedSwitch;
	                                                                 if(WirPro.ParaMode== 1 && ReedSwitchOn ==ReedSwitch)
	                                                                {
	                                                                                 BELLON;
	                                                                                 delay_ms(200);
	                                                                                 delay_ms(100);
	                                                                                 BELLOFF;
	                                                                                 TimeDelayUp=0;
	                                                                }
	                                                                Envent.DeviceUp =0;
	                                                                Envent.CtrlUpSend =VALID_FLAG;
	                                                                
	                                                                Envent.Si24R1Sleep =25;
	                                                }
	                                                
	                                }
	                                 else if (Envent.Si24R1Sleep>=25) //5s
	                                {
	                                                Envent.Si24R1Sleep =0;
	                                                Envent.Si24R1Work =50;//50ms
	                                }
	                                R_WDT_Restart();
	               } 
	               SI24R1_PowerOn();
	                ...............
	}

```

最后对系统的功能进行整体调试，得到的待机模式下工作电流为6微安左右。无线发射和接收时有15mA，蜂鸣器工作时，最高有37mA。

基本满足要求设计。不过还需要继续完善。

![效果图](http://7xlqnm.com1.z0.glb.clouddn.com/2016/08/3328869344.png)