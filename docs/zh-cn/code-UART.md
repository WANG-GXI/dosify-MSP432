# 4   串口

>**串口可以说是单片机的灵魂之一，不仅仅是简单的收发数据，我们有时候使用MPU6050看似非常容易是因为原子哥都给我们把事情做好了
，但是当小鱼君在尝试移植到MSP432E401Y上时，总是失败。询问了好多小伙伴都不成功，当然是技术的问题喽。
但是比赛要用怎么办，我们就选择串口陀螺仪，只需要解析它的协议就可以了。  通过本章的学习， 你将掌握到串口接收和发送的方法。**

<font color=black size=3>每次开始我们都会把TI官方的网站放到最前面，大家可以自己提前了解一手：[API Guide](https://dev.ti.com/tirex/explore/node?node=AOD4LXqFA8XEr21FehvtQA__J4.hfJy__LATEST)

我们以练代学，直接以蓝牙模块为例。这里使用的是HC-08。
相关工程可以在群内获取到。


**话不多说直接开始。**

# 串口初始化
```
// 使用串口5 波特率 9600
void ConfigureUART5()
{
    MAP_SysCtlPeripheralEnable(SYSCTL_PERIPH_UART5);
    MAP_SysCtlPeripheralEnable(SYSCTL_PERIPH_GPIOC);
    MAP_IntMasterEnable();
    GPIOPinConfigure(GPIO_PC6_U5RX);
    GPIOPinConfigure(GPIO_PC7_U5TX);
    MAP_GPIOPinTypeUART(GPIO_PORTC_BASE, GPIO_PIN_6 | GPIO_PIN_7);
    MAP_UARTConfigSetExpClk(UART5_BASE, g_ui32SysClock, 9600,
                                        (UART_CONFIG_WLEN_8 | UART_CONFIG_STOP_ONE |
                                         UART_CONFIG_PAR_NONE));
    MAP_IntEnable(INT_UART5);
    MAP_UARTIntEnable(UART5_BASE, UART_INT_RX | UART_INT_RT);
}

```

# 接收中断函数

```
int32_t temp_UART5=0
void
UART5_IRQHandler(void)
{
    uint32_t ui32Status;

    ui32Status = MAP_UARTIntStatus(UART5_BASE, true);

    MAP_UARTIntClear(UART5_BASE, ui32Status);

   while(MAP_UARTCharsAvail(UART5_BASE))
   {
        temp_UART5=UARTCharGetNonBlocking(UART5_BASE);
        if(temp_UART5==97)           flag1=1;     //a 
        else if(temp_UART5==98)      flag2=1;     //b  
        else if(temp_UART5==99)      flag3=1;     //c
  }
}

```
到了这里就不得的提一嘴了，我们在这里可以看到使用方法和STM32是没什么区别的，但是为什么会用int32_t 定义一个变量来接收数据呢？在32中不应该是char嘛

**官方手册文件是这样说的，串口会将接收到的数据转换为int32_t。**

<img src=" https://wang-gxi.github.io/my-photo/11124.png" width="70%">

所以我们其实就会发现我接收到的数据它会强制给我转换为int32类型的，所以在初学的时候会有这样的情况，我给板子发送的是1，但是板子接收到的是49，
**别怀疑，就是它给你强制转化了**，到这里可能觉得没什么啊，我一般用一些标志位的话我一个个找ascii码去对应就可以了，但是如果你的标志位有20个，那你给板子发送20，它会接收到什么呢？我们这里留个悬念，结果是肯定错的。

之前小鱼君发现这个问题的时候是在用K210识别20种物体，然后把结果返回到432的，为了做出效果来就直接把0到9用光了，发现还不够又添加了
abcd等字母，后来才发现原来有一种东西叫用**数据协议**。

**是不是感觉坑特别多？**

<img src=" https://wang-gxi.github.io/my-photo/23889.jpg" width="30%">

>现在开始我们看一个最简单的数据发送

```
void usart5_send_char(u8 c)
{
    UARTCharPut(UART5_BASE, c);
}
void UART5Send(char* s)
{
    while(*s)
    {
        MAP_UARTCharPutNonBlocking(UART5_BASE, *s++);
    }
}
```
这个过程其实就是把32的东西换了一个底层而已。第一个是发送字符，第二个是发送字符串。

然后我们就可以开始连接蓝牙模块，然后手机和它配对就好了。建议大家手机端用“蓝牙调试器”这个软件。**我可没有打广告哈，**
它可以把数据转化为**波形**在手机上显示。（如果说对蓝牙模块不是很熟悉的，可以用USB连接到电脑上看效果）

>**完整工程**

```
int
main(void)
{
    uint32_t g_ui32SysClock;
    u8  uart_value=0xA5;
    g_ui32SysClock = MAP_SysCtlClockFreqSet((SYSCTL_XTAL_25MHZ | SYSCTL_OSC_MAIN |
            SYSCTL_USE_PLL | SYSCTL_CFG_VCO_480),120000000);
     
    ConfigureUART5();
    //在主循环中不断发送
    while(1)
    {
    usart5_send_char(uart_value);
    SysCtlDelay(g_ui32SysClock/30);//延时
    }

}
```
如果没什么问题的话我们是可以看到0XA5，不管是手机还是电脑都是一样的。在此基础上就是发送波形了，其实就是把上面的两个底层函数封装下，相关的教程我们后续会继续制作。


>**到这里大家就会配置完整的UART了，不知道屏幕前的小伙伴成功了吗，我在TI小分队等你QQ：866325871**


<img src=" https://wang-gxi.github.io/my-photo/2323.jpeg" width="50%">