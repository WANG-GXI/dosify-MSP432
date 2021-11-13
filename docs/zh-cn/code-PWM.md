# 5   PWM

>**如果想做小车，那么让它跑起来必不可少的就是PWM波了。 通过本章的学习， 你将掌握到PWM的配置以及如何改变占空比。**

<font color=black size=3>每次开始我们都会把TI官方的网站放到最前面，大家可以自己提前了解一手：[API Guide](https://dev.ti.com/tirex/explore/node?node=AOD4LXqFA8XEr21FehvtQA__J4.hfJy__LATEST)

这里多提一嘴，在设计时怎么去寻找自己要用的PWM口和432的对应关系。这些都可以在上方的手册中找到，但是为了便利我直接贴出来。

<img src=" https://wang-gxi.github.io/my-photo/52179.png" width="80%">

可以看到红框中说到，PWM0、PWM1都是Gen0，所以在初始化的时候就是用的`PWM_GEN_0`

**话不多说直接开始。**

# PWM初始化
```
void PWM1_Init()
{
    /* PWM外设必须启用才能使用. */
       MAP_SysCtlPeripheralEnable(SYSCTL_PERIPH_PWM0);
       while(!(MAP_SysCtlPeripheralReady(SYSCTL_PERIPH_PWM0)));

       /* 设置PWM时钟为系统时钟。 */
       MAP_PWMClockSet(PWM0_BASE,PWM_SYSCLK_DIV_1);

       /* 为PWM引脚使能GPIO端口F的时钟 */
       MAP_SysCtlPeripheralEnable(SYSCTL_PERIPH_GPIOF);
       while(!MAP_SysCtlPeripheralReady(SYSCTL_PERIPH_GPIOF));

       MAP_GPIOPinConfigure(GPIO_PF0_M0PWM0);
       MAP_GPIOPinConfigure(GPIO_PF1_M0PWM1);
       MAP_GPIOPinTypePWM(GPIO_PORTF_BASE, (GPIO_PIN_0 | GPIO_PIN_1));

       /* 配置PWM0为向上/向下计数，不进行同步。 */
       MAP_PWMGenConfigure(PWM0_BASE, PWM_GEN_0, PWM_GEN_MODE_UP_DOWN |
                           PWM_GEN_MODE_NO_SYNC);

       /* 设置PWM周期为250Hz。计算合适的参数N=(1/ f) * SysClk. 
           N = (1 / f)其中N是函数参数，
           f为所需频率，SysClk为系统时钟频率。在这种情况下，
           你得到:(1 / 250Hz) * 16MHz = 64000周期。请注意,
           您可以设置的 最大周期是2^16 - 1。*/
       MAP_PWMGenPeriodSet(PWM0_BASE, PWM_GEN_0, 7058);


       /* 设置PWM0 PF0的占空比为25%。你把占空比设为
	期间的功能。由于上面设置了句点，您可以使用
	PWMGenPeriodGet()函数。对于这个例子，PWM将是高的
	 25%的时间或16000个时钟周期(64000 / 4)。 */
       PWM0_NUMBER =MAP_PWMGenPeriodGet(PWM0_BASE, PWM_GEN_0)/5;

     /*  MAP_PWMPulseWidthSet(PWM0_BASE, PWM_OUT_0,
                            PWM0_NUMBER);*/

       /* PWM0 PF1占空比设置为75%。你把占空比设为
	期间的功能。由于上面设置了句点，您可以使用
	PWMGenPeriodGet()函数。对于这个例子，PWM将是高的
	7%的时间或16000时钟周期3*(64000 / 4)。 */
       MAP_PWMPulseWidthSet(PWM0_BASE, PWM_OUT_1,
                            0);


       MAP_IntMasterEnable();

       /* 该定时器处于up-down模式。中断将发生时
	计数器为这个PWM计数到负载值(64000)，当
	计数器计数到64000/4 (PWM A up)，计数到64000/4
	 (PWM A Down)，计数到0。 */
       MAP_PWMGenIntTrigEnable(PWM0_BASE, PWM_GEN_0,
                               PWM_INT_CNT_ZERO | PWM_INT_CNT_LOAD |
                               PWM_INT_CNT_AU | PWM_INT_CNT_AD);



       MAP_IntEnable(INT_PWM0_0);
       MAP_PWMIntEnable(PWM0_BASE, PWM_INT_GEN_0);
       /* 使能PWM0 Bit 0 (PF0)和Bit 1 (PF1)输出信号。*/
       MAP_PWMOutputState(PWM0_BASE, PWM_OUT_0_BIT | PWM_OUT_1_BIT, true);

       /* 使PWM发生器块计数器. */
       MAP_PWMGenEnable(PWM0_BASE, PWM_GEN_0);


}
```

看到这里是不是感觉很麻烦的，而且这么多配置万一出错了要怎么排查问题？非常抱歉的对屏幕前的小伙伴们说，这里没有捷径。
最好的办法其实就是自己把官方例程下一遍然后自己试试看，因为小鱼君也是按需来改代码，有的PWM口我是不需要的，就懒得再配置一遍了，但是官方是给的十分全面的。

# 如何改变占空比

```
    PWM0_NUMBER= MAP_PWMGenPeriodGet(PWM0_BASE, PWM_GEN_0)/4;//25%的占空比
    MAP_PWMPulseWidthSet(PWM0_BASE, PWM_OUT_1,PWM0_NUMBER);

```

可以看到这个过程是非常容易的。

这里的`PWM0_NUMBER`是什么呢？我们可以理解为PWM的占空比值，它是作为函数的参数去直接改变占空比的。也就是下面这个函数：

<img src=" https://wang-gxi.github.io/my-photo/1011.png" width="80%">

# 查错小手册
其实也没什么好说的了，最后就是几点注意事项帮助大家第一时间遇到问题该如何解决。

* 首先查引脚配置，会不会把引脚PF0改为了PE0之类的要命错误。
* 在调试中可以用按键来改变占空比然后接示波器看波形变化。
* 最后再怀疑是引脚被烧了，一般可能性很小。



>**到这里大家就可以使能PWM了，不知道屏幕前的小伙伴成功了吗，我在TI小分队等你QQ：866325871**


<img src=" https://wang-gxi.github.io/my-photo/2323.jpeg" width="50%">