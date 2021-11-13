# 1   一灯大师

>**任何一个单片机，最简单的外设莫过于 IO 口的高低电平控制了，本章将通过一个经典的
点灯程序，带大家开启 MSP432之旅， 通过本章的学习， 你将了解到 IO 口作
为输出使用的方法。**

<font color=black size=3>每次开始我们都会把TI官方的网站放到最前面，大家可以自己提前了解一手：[API Guide](https://dev.ti.com/tirex/explore/node?node=AOD4LXqFA8XEr21FehvtQA__J4.hfJy__LATEST)

如果大家之前的步骤完成了，自己的电脑上就会有TI官方给的例程包。名字是【simplelink_msp432e4_sdk_4_20_00_12】。同样在之前给的TI云服务器中也可以找到代码。

我们话不多说直接开始。
```
// blinky.c - Simple example to blink the on-board LED.
#include <stdint.h>
#include <stdbool.h>
#include "ti/devices/msp432e4/driverlib/driverlib.h"


//*****************************************************************************
//
// The error routine that is called if the driver library encounters an error.
//
//*****************************************************************************
#ifdef DEBUG
void
__error__(char *pcFilename, uint32_t ui32Line)
{
    while(1);
}
#endif

//*****************************************************************************
//
// Blink the on-board LED.
//
//*****************************************************************************
int
main(void)
{
    volatile uint32_t ui32Loop;

    //
    // Enable the GPIO port that is used for the on-board LED.
    //
    SysCtlPeripheralEnable(SYSCTL_PERIPH_GPION);

    //
    // Check if the peripheral access is enabled.
    //
    while(!SysCtlPeripheralReady(SYSCTL_PERIPH_GPION))
    {
    }

    //
    // Enable the GPIO pin for the LED (PN0).  Set the direction as output, and
    // enable the GPIO pin for digital function.
    //
    GPIOPinTypeGPIOOutput(GPIO_PORTN_BASE, GPIO_PIN_0);

    //
    // Loop forever.
    //
    while(1)
    {
        //
        // Turn on the LED.
        //
        GPIOPinWrite(GPIO_PORTN_BASE, GPIO_PIN_0, GPIO_PIN_0);

        //
        // Delay for a bit.
        //
        for(ui32Loop = 0; ui32Loop < 200000; ui32Loop++)
        {
        }

        //
        // Turn off the LED.
        //
        GPIOPinWrite(GPIO_PORTN_BASE, GPIO_PIN_0, 0x0);

        //
        // Delay for a bit.
        //
        for(ui32Loop = 0; ui32Loop < 200000; ui32Loop++)
        {
        }
    }
}

```


我们可以发现它432的代码和32也是很类似的，学习起来也比较容易。

由于我们是以快速上手为目的，所以我们对底层也不做太多的研究，直接说该怎么用。

# 如何配置IO口

从上面的代码我们可以发现它的核心函数如下：
```
   SysCtlPeripheralEnable(SYSCTL_PERIPH_GPION);

    //
    // Check if the peripheral access is enabled.
    //
    while(!SysCtlPeripheralReady(SYSCTL_PERIPH_GPION))
    {
    }

    //
    // Enable the GPIO pin for the LED (PN0).  Set the direction as output, and
    // enable the GPIO pin for digital function.
    //
    GPIOPinTypeGPIOOutput(GPIO_PORTN_BASE, GPIO_PIN_0);
```
细心的小伙伴翻译它的英文注释后就会发现其实就是两个步骤：初始化GPION，之后再选择N0作为输出。在板子上它的引脚对应的就是LED灯，然后在主函数中通过给N0不同的值来控制LED的亮灭。

# 如何反转电平

那么问题来了，我们在32的开发中使用的不是PBout(5)=0或者PBout(5)=1嘛，在这里为什么给的值不是0和1呢？

在刚开始使用时也是折磨了小鱼君好久，后来才发现TI官方就是这样子规定的，我们其实可以直接按这样的方式输出：

```
//以PB4为例子
MAP_GPIOPinWrite(GPIO_PORTB_BASE,GPIO_PIN_4,GPIO_PIN_4);            //置高电平
MAP_GPIOPinWrite(GPIO_PORTB_BASE,GPIO_PIN_4,~(GPIO_PIN_4));         //置低电平
```
<img src=" https://wang-gxi.github.io/my-photo/2322.jfif" width="30%">


**我们与32对比：**
```
GPIO_SetBits(GPIOB, GPIO_Pin_5); //设置 GPIOB.5 输出 1,等同 LED0=1;
GPIO_ResetBits (GPIOB, GPIO_Pin_5); //设置 GPIOB.5 输出 0,等同 LED0=0;
```



好像发现了财富密码！
**通过上面这种方法就可以对IO口进行操作了！**

其实还有一种操作方法，小鱼君这里先卖个关子，大家可以在公众号后台回复**【432密码1】**即可呦


>**到这里相信大家也可以做432的一灯大师了，是不是很开心呢？**

<img src=" https://wang-gxi.github.io/my-photo/2323.jpeg" width="50%">