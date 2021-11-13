# 3   按键

>**之前两节使用的只有输出IO，而没有输入，相信大家肯定也有些腻烦了，所以我们将使用按键来引入IO的输入功能。 通过本章的学习， 你将掌握到 IO 口作
为输入使用的方法。**

<font color=black size=3>每次开始我们都会把TI官方的网站放到最前面，大家可以自己提前了解一手：[API Guide](https://dev.ti.com/tirex/explore/node?node=AOD4LXqFA8XEr21FehvtQA__J4.hfJy__LATEST)

我们使用的是之前介绍过的小系统板：对应的按键引脚大家直接看代码就可以找到。


<img src=" https://wang-gxi.github.io/my-photo/56.png" width="70%">

# 按键初始化

话不多说直接开始。
```

#include <stdint.h>
#include <stdbool.h>
#include "ti/devices/msp432e4/driverlib/driverlib.h"

#define k1  MAP_GPIOPinRead(GPIO_PORTM_BASE,GPIO_PIN_0)
#define k2  MAP_GPIOPinRead(GPIO_PORTM_BASE,GPIO_PIN_1)

#define KEY0_PRES     1   //KEY0按下
#define KEY1_PRES     2   //KEY1按下
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
void KEY_Init(void)
{
    MAP_SysCtlPeripheralEnable(SYSCTL_PERIPH_GPIOM);
    while(!(SysCtlPeripheralReady(SYSCTL_PERIPH_GPIOM)))
    ;
    MAP_GPIOPinTypeGPIOInput(GPIO_PORTM_BASE, (GPIO_PIN_0 | GPIO_PIN_1| GPIO_PIN_2| GPIO_PIN_3| GPIO_PIN_6| GPIO_PIN_7));
    GPIOM->PUR |= GPIO_PIN_0 | GPIO_PIN_1| GPIO_PIN_2| GPIO_PIN_3| GPIO_PIN_6| GPIO_PIN_7;
}
u8 KEY_Scan(u8 mode)
{
        static u8 sta=1;
        if(mode)sta=1;
        if(sta&&(k1==0||k2==0))
        {
        delay_ms(10);
        sta=0;
        if(k1==0)                return KEY0_PRES;
        else if(k2==0)           return KEY1_PRES;
        }
        else if(k1!=0 && k2!=0)     sta=1;
        return 0;
}
int
main(void)
{
    uint32_t g_ui32SysClock;
    int key_value=0;
    g_ui32SysClock = MAP_SysCtlClockFreqSet((SYSCTL_XTAL_25MHZ | SYSCTL_OSC_MAIN |
            SYSCTL_USE_PLL | SYSCTL_CFG_VCO_480),120000000);
     
    KEY_Init();//初始化PN4
    //在主循环中不断取反PN4
    while(1)
    {
      key_value=KEY_Scan(0);
    
     if(key_value==KEY0_PRES) 
{
//点灯或者蜂鸣器
}
     if(key_value==KEY1_PRES) 
{
//点不同灯
}
      SysCtlDelay(g_ui32SysClock/30);//延时
    }
}
```

# 配置时的细节

我们这里需要用到配置为上拉：
```
    GPIOM->PUR |= GPIO_PIN_0 | GPIO_PIN_1| GPIO_PIN_2| GPIO_PIN_3| GPIO_PIN_6| GPIO_PIN_7; 
```
这里的操作是小鱼君看TI官方例程中学到的，大家直接CtrlCV就可以了。

# 按键的应用

可以看到其实和STM32的开发是一致的，我们把底层做好之后应用层的东西都是相通的。
在这里我们只是用到了其中的两个，完整的应该是六个按键同时使用，这里只是演示。
```
    key_value=KEY_Scan(0);
    
     if(key_value==KEY0_PRES) 
{
//点灯或者蜂鸣器
}
     if(key_value==KEY1_PRES) 
{
//点不同灯
}
```

>**到这里大家就会配置完整的GPIO了，不知道屏幕前的小伙伴成功了吗，我在TI小分队等你QQ：866325871**

<img src=" https://wang-gxi.github.io/my-photo/2323.jpeg" width="50%">