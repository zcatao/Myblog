title: STM32_I2C硬件BUG引发的血案(转)
date: 2015-12-20 12:43:41
tags: [STM32,I2C,嵌入式,ARM]
categories: STM32
---
>江湖传言，STM32的硬件I2C一直有bug，最近面临做四旋翼的芯片选型问题,[原文](http://blog.gkong.com/zjcsharp_112878.ashx)

下面的函数中有一个BUG, 也就是SR2不能用WHILE来轮询,而应直接读出.如下面代码段, 因此,在这里说的这是STM32的BUG其实是我的代码的错误:
``` cpp
    I2C2->DR = inerAddress[1];
    while( (I2C2->SR1&Q_I2C_SR1_BIT_BTF)==0 );
    I2C2->SR2;   // 正解
```
一直都不相信STM32的I2C接口会存在问题，因为工作经验无数次告诉我，嵌入式系统设计中的99.999%的问题不会是由于MCU本身的设计问题所引起,绝大部分都是硬件工程师或软件工程师的某个设计缺陷所造成的. 这次的设计经历也不例外.

由于终于可以抽多点时间用于设计STM32的I2C的软件接口, 因此, 决定重构之前用于主从STM32通讯所用的I2C模块（基于I2C中断+状态机收发）。I2C的中断发送模块很快就重构完毕并且强化了错误处理和I2C总线hang的自恢复功能代码模块（I2C BUS Hang：也就是SDA 和SCL都被某Slave DEVICE拉低，大部分发生在主机接收从机传过来的数据包的最后一个字节的数据时，没有发送P（停止位）所造成的，这时Slave device （EEPROM）因为接收不到P，从而会DEAD LOOP地发送数据给主机，在主机方向看，I2C总线就相当于一直处于BUSY的状态，也就是网上问得最多的关于I2C的问题-I2C总线TMD的怎么会突然间死掉了、BUSY了、HANG的原因。）。

本以为I2C中断读模块也会很顺利地完成，但是却被卡住了一整天，只要加一个断点，在大部份代码处加无所谓，读FUN每次都正确执行，但只要全速运行，就只能运行一次，然后就过不去了，而且过不去的地方并不固定。由于DUBEG得有点集中不了精力，于是就做好问题的Brainstrom的笔记，放了下，晚上出差到深圳，到客户处做系统需求的讨论会，主要是上位机部分的。然后第二天晚上又赶回来，在车上的半梦半觉中让潜意思去思考。回到家后，选择在我的生物钟的最佳的时侯，晚上10点。开始重新在问题的Brainstorm处接着DUBGE。WORK PLAN如下：
1. 把程序恢复到测试I2C写的测试用例状态，PASS。
2.   一小段代码就DEBUG一次的步步为营方法，按之前的思路把代码加入到读的模块（忽略防守代码，只加入必需的功能代码）。都OK后，全速运行，XXX，又卡住了。细过了一片读模块。PASS。于是进入计划中的（3）
3. 全速运行，然后在可能卡住的地方加上断点，而不是之前的先加断点，然后运行的方式（这种方式，已确认PASS）：
加了几处后，发现程序是在下面代码处卡住的：
![](http://7xpc7h.com1.z0.glb.clouddn.com/9KGSA@[JTLD6I3V9B~`4P{U.png)

![](http://7xpc7h.com1.z0.glb.clouddn.com/stm32image_thumb[9]%25207AD77FA8.png)
但是奇怪了，每次我只要在箭头指向的语句后加上断点，每次都能PASS。于是知道要到 I2C_CheckEvent 函数中就能找到问题的原因了，于是做下面的几步，目的是把包含I2C_CheckEvent Fun的链接库中的stm32f10x_i2c.o排除掉,而把stm32f10x_i2c.c加入到项目中,使得DEBUG时能进入到 I2C_CheckEvent Fun 中去.
![](http://7xpc7h.com1.z0.glb.clouddn.com/stm32image_thumb[3]%2520604637F6.png)

于是全速运行，然后在 stm32f10x_i2c.c中加上断点，终于捕捉到问题点了，分析如下图所示：

![](http://7xpc7h.com1.z0.glb.clouddn.com/stm32image_thumb[7]%25200E51DD10.png)
``` cpp
#define vu32 volitile unsigned long long 
/* Read the I2Cx status register */
//  flag1 = I2Cx->SR1;             // 原代码
//  flag2 = I2Cx->SR2;            //原代码
//  flag2 = flag2 << 16;           //原代码

 /* Get the last event value from I2C status register */
  //lastevent = (flag1 | flag2) & FLAG_Mask; //原代码
 lastevent = (vu32)( (vu32)(I2Cx->SR1) | ((vu32)(I2Cx->SR2) << (vu32)16) ); //qzm为了确认而加入的，实际效果和用库的是一样的结果。
 lastevent &= (vu32)FLAG_Mask; //qzm为了确认而加入的，实际效果和用库的是一样的结果。


```

为了确认，我也把 Fun中的所有变量改为以v开头的，以确保不被编译器所优化掉，编译代码也不作任何的优化。但是全速时I2C获得的事件会多出个BTF位，而在一开始时如果先进入DEBUG，加上断点，然后运行，lastevent == 0x30001(之也说明了库代码是不存在BUG的）,如下图所示:

  ![](http://7xpc7h.com1.z0.glb.clouddn.com/stm32image_thumb[5]%2520078E6204.png)

这应该是STM32 I2C硬件接口的BUG，解决方法如下：
![](http://7xpc7h.com1.z0.glb.clouddn.com/stm32image_thumb[11]%20471D6A2F.png)

把库中的I2C中断事件判定结合SR1和SR2的思路相反,我把读SR1和SR2明确地分了出现,并进行确认, PASS, 给模块加上防守代码,做好文档, 至此,模块的生命周期进入到白盒测试和黑盒测试阶段.
