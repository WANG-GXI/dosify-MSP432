# 7   定时器

>**对于定时器我们有很多要说的，可以计数，可以作为编码器，可以输出50HzPWM波，通过本章的学习， 你将掌握到timer的配置。**

<font color=black size=3>每次开始我们都会把TI官方的网站放到最前面，大家可以自己提前了解一手：[API Guide](https://dev.ti.com/tirex/explore/node?node=AOD4LXqFA8XEr21FehvtQA__J4.hfJy__LATEST)

我们分为三个部分，首先我们的定时器用来进行严格的计时，比如说有的传感器数据我需要100ms读取一次。

# 定时器--严格计时
```
void Timerw_Init()
{
    MAP_SysCtlPeripheralEnable(SYSCTL_PERIPH_TIMER2);

       // Enable processor interrupts.

       MAP_IntMasterEnable();

       //
       // Configure the two 32-bit periodic timers.
       //
       MAP_TimerConfigure(TIMER2_BASE, TIMER_CFG_PERIODIC);

       MAP_TimerLoadSet(TIMER2_BASE, TIMER_A, g_ui32SysClock /10);//定时100ms
       //
       // Setup the interrupts for the timer timeouts.
       //
       MAP_IntEnable(INT_TIMER2A);
       MAP_TimerIntEnable(TIMER2_BASE, TIMER_TIMA_TIMEOUT);

       //
       // Enable the timers.
       //
       MAP_TimerEnable(TIMER2_BASE, TIMER_A);

}
```
以上代码我们可以得知，用到了定时器2，周期为100ms，然后就可以开心的在中断函数中做事了。

>中断函数

```
void
TIMER2A_IRQHandler(void)
{

    //
    // Clear the timer interrupt.
    //
    MAP_TimerIntClear(TIMER2_BASE, TIMER_TIMA_TIMEOUT);

/*
在这里放代码

*/
    // Update the interrupt status.
    MAP_IntMasterDisable();

    MAP_IntMasterEnable();
}
```
这里就是一个最简单的使用，像平衡小车之家他们代码风格就是把要处理的东西都扔到定时器里，然后while中都是一些显示数字的函数。

>测试现象

小鱼君这里提供一种思路，比如说我想看定时器这个时间到底准不准，可以定义一个变量让它累加，然后在debug中实时查看这个
变量的值，然后过几秒做个差，看在这段时间内加了多少次。


到这里大家就可以用定时器计数了，不知道屏幕前的小伙伴成功了吗，由于定时器非常重要，所以我们还要用一下它来驱动舵机。所以这里不能太早结束。

# 定时器驱动舵机

这里其实很多人会说，我PWM波不可以嘛，为什么要用定时器浪费我资源，没错，这里就是需要这样。其实这是由于舵机
独特的工作频率要50Hz，实在是太低了，又有同学要说我可以分频呀。

我们这里计算一下，现在MCU工作频率在120MHz，那么分频系数为2400000，也就是0x249F00，我们可以把0x249F00设定为装载值。但是我PWM
波只有16bit的计数器，这24bit的数我放不进去。所以只能用定时器来输出。

那么问题来了，我定时器不也是16bit么？不要慌，这里我们加入一个8bit的预分频来共同构成一个24bit的计数器。以STM32的目光来看
是没问题的，它是将时钟源的输入频率降低到原来的1/2^n，但在MSP432E4里则完全不是这个样子，预分频器的值
代表了一个周期内16bit的计数器满溢（即到达最大值0xFFFF）的次数。

举个例子，假设预分频器设置为1，装载值设定为0x2000，那么实际的计数数将会是1*0x10000+0x2000=0x12000。
因此，需要的24bit装载值的前8bit放入预分频中，后16bit放入装载值中。当然对于懒人来说我们实际使用的值直接用系统时钟来
计算就可以了，也就是一直出现的g_ui32SysClock。
>舵机初始化

```
void  DUOJI1_Init()
{
       total_period1 = g_ui32SysClock/50;
       high_period1 = pwmhigh_period_cal(0, total_period1);
       /* Enable the clock to the GPIO Port A and wait for it to be ready */
       SysCtlPeripheralEnable(SYSCTL_PERIPH_GPIOB);
       while(!(SysCtlPeripheralReady(SYSCTL_PERIPH_GPIOB)));

       /* Configure the GPIO PA7 as Timer-3 CCP1 output */
       GPIOPinConfigure(GPIO_PB3_T5CCP1);
       GPIOPinTypeTimer(GPIO_PORTB_BASE, GPIO_PIN_3);

       /* Enable the Timer-3 in 16-bit PWM mode */
       SysCtlPeripheralEnable(SYSCTL_PERIPH_TIMER5);
       while(!(SysCtlPeripheralReady(SYSCTL_PERIPH_TIMER5)))
             ;
       MAP_TimerConfigure(TIMER5_BASE, TIMER_CFG_SPLIT_PAIR | TIMER_CFG_B_PWM);

       /* The timer shall count from load value to match value and the
        * output will be high. From the match value to the 0 the timer will be low.*/
       TimerPrescaleSet(TIMER5_BASE, TIMER_B, total_period1>>16);
       TimerLoadSet(TIMER5_BASE, TIMER_B, total_period1&0xFFFF);

       /* Initialize the servo to 0° */
       uint32_t high_period1 = pwmhigh_period_cal(90, total_period1);
       TimerPrescaleMatchSet(TIMER5_BASE, TIMER_B, high_period1>>16);
       TimerMatchSet(TIMER5_BASE, TIMER_B, high_period1&0xFFFF);

       TimerEnable(TIMER5_BASE, TIMER_B);

}

```
然后我们直接在主函数中调用这个函数就可以改变角度了，以角度制表示的匹配值的计算函数如下所示

```
/*
 * 舵机角度控制
 */
uint32_t pwmhigh_period_cal(int angle, uint32_t total_period)
{
    return (0.04*total_period*angle/90 + 0.925*total_period);
}
void duoji1()
{
     high_period1 = pwmhigh_period_cal(duoji_Angle1, total_period1);
    TimerPrescaleMatchSet(TIMER5_BASE, TIMER_B, high_period1>>16);
    TimerMatchSet(TIMER5_BASE, TIMER_B, high_period1&0xFFFF);
    TimerEnable(TIMER5_BASE, TIMER_B);
}
```

怎么样，整个通俗的名字，直接调用duoji1()，然后我们只需要修改(duoji_Angle1的值就可以了。

>**到这里大家就会配置完整的定时器了，不知道屏幕前的小伙伴成功了吗，我在公众号“小鱼君code”等你**

<img src=" https://wang-gxi.github.io/my-photo/2323.jpeg" width="50%">