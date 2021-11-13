# 2   蜂鸣器

>**上一节掌握了LED灯的使用，我们这里将示例蜂鸣器来进一步增加大家对432基础外设的操作。 通过本章的学习， 你将掌握到 IO 口作
为输出使用的方法。**

<font color=black size=3>每次开始我们都会把TI官方的网站放到最前面，大家可以自己提前了解一手：[API Guide](https://dev.ti.com/tirex/explore/node?node=AOD4LXqFA8XEr21FehvtQA__J4.hfJy__LATEST)

我们的蜂鸣器对应的引脚是**PN4**。
# 初始化

```

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
void Beep()
{
    SysCtlPeripheralEnable(SYSCTL_PERIPH_GPION);
    while(!SysCtlPeripheralReady(SYSCTL_PERIPH_GPION));
    GPIOPinTypeGPIOOutput(GPIO_PORTN_BASE, GPIO_PIN_4);
    MAP_GPIOPinWrite(GPIO_PORTN_BASE,GPIO_PIN_4,0);
}
int
main(void)
{
    uint32_t g_ui32SysClock;
    g_ui32SysClock = MAP_SysCtlClockFreqSet((SYSCTL_XTAL_25MHZ | SYSCTL_OSC_MAIN |
            SYSCTL_USE_PLL | SYSCTL_CFG_VCO_480),120000000);
     
    Beep();//初始化PN4
    //在主循环中不断取反PN4
    while(1)
    {
      MAP_GPIOPinWrite(GPIO_PORTN_BASE,GPIO_PIN_4,0xff);
      SysCtlDelay(g_ui32SysClock/30);//延时
      MAP_GPIOPinWrite(GPIO_PORTN_BASE,GPIO_PIN_4,0);
      SysCtlDelay(g_ui32SysClock/30);//延时
    }
}
```

# 配置自己需要的IO口

一般的输出引脚大家直接使用上面的函数就可以了：
```
 void Beep()
{
    SysCtlPeripheralEnable(SYSCTL_PERIPH_GPION);
    while(!SysCtlPeripheralReady(SYSCTL_PERIPH_GPION));
    GPIOPinTypeGPIOOutput(GPIO_PORTN_BASE, GPIO_PIN_4);
    MAP_GPIOPinWrite(GPIO_PORTN_BASE,GPIO_PIN_4,0);
}
```


>**万事开头难，解决了最简单的GPIO之后的开发就如鱼得水了**

<img src=" https://wang-gxi.github.io/my-photo/2323.jpeg" width="50%">