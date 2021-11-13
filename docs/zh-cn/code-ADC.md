# 6   ADC

>**其实这个官方已经给了很多了，21年电赛A题就是这样，第一天我们还在挣扎做A，因为正好用过432的ADC进行FFT
，后来发现还是学艺不精就放弃了，但是频率测量是很准确的。 通过本章的学习， 你将掌握到ADC的配置以及如何改变占空比。**

<font color=black size=3>每次开始我们都会把TI官方的网站放到最前面，大家可以自己提前了解一手：[API Guide](https://dev.ti.com/tirex/explore/node?node=AOD4LXqFA8XEr21FehvtQA__J4.hfJy__LATEST)

由于官方给的例程实在是太多了，这里就挑一个比较舒服的来说，是用定时器来采集的。


**话不多说直接开始。**

# ADC初始化
```
void adc_averge_Init()
{ /* Enable the clock to GPIO Port E and wait for it to be ready */
    MAP_SysCtlPeripheralEnable(SYSCTL_PERIPH_GPIOE);
    while(!(MAP_SysCtlPeripheralReady(SYSCTL_PERIPH_GPIOE)))
    {
    }

    /* Configure PE3 as ADC input channel */
    MAP_GPIOPinTypeADC(GPIO_PORTE_BASE, GPIO_PIN_3);

    /* Enable the clock to ADC-0 and wait for it to be ready */
    MAP_SysCtlPeripheralEnable(SYSCTL_PERIPH_ADC0);
    while(!(MAP_SysCtlPeripheralReady(SYSCTL_PERIPH_ADC0)))
    {
    }
    MAP_ADCSequenceStepConfigure(ADC0_BASE, 2, 3, ADC_CTL_CH3 | ADC_CTL_IE |
                                     ADC_CTL_END);
    MAP_ADCSequenceConfigure(ADC0_BASE, 2, ADC_TRIGGER_TIMER, 2);

       /* Since sample sequence 2 is now configured, it must be enabled. */
       MAP_ADCSequenceEnable(ADC0_BASE, 2);

       /* Clear the interrupt status flag before enabling. This is done to make
        * sure the interrupt flag is cleared before we sample. */
       MAP_ADCIntClear(ADC0_BASE, 2);
       MAP_ADCIntEnable(ADC0_BASE, 2);

       /* Enable the Interrupt generation from the ADC-0 Sequencer */
       MAP_IntEnable(INT_ADC0SS2);

       /* Enable Timer-0 clock and configure the timer in periodic mode with
        * a frequency of 10 KHz. Enable the ADC trigger generation from the
        * timer-0. */
       MAP_SysCtlPeripheralEnable(SYSCTL_PERIPH_TIMER0);
       while(!(MAP_SysCtlPeripheralReady(SYSCTL_PERIPH_TIMER0)))
       {
       }

       MAP_TimerConfigure(TIMER0_BASE, TIMER_CFG_A_PERIODIC);
       MAP_TimerLoadSet(TIMER0_BASE, TIMER_A, (g_ui32SysClock/10000));
       MAP_TimerADCEventSet(TIMER0_BASE, TIMER_ADC_TIMEOUT_A);
       MAP_TimerControlTrigger(TIMER0_BASE, TIMER_A, true);
       MAP_TimerEnable(TIMER0_BASE, TIMER_A);

}
```
我们长话短说，这里其实就是三个步骤，**先配置PE3作为ADC采样的引脚；然后配置ADC采样通道；最后将定时器0作为采样定时器。**

# ADC中断

```
volatile bool bgetConvStatus = false;;
void ADC0SS2_IRQHandler(void)
{
    uint32_t getIntStatus;

     Get the interrupt status from the ADC
    getIntStatus = MAP_ADCIntStatus(ADC0_BASE, 2, true);

     If the interrupt status for Sequencer-2 is set the
     * clear the status and read the data
    if(getIntStatus == 0x4)
    {
         Clear the ADC interrupt flag.
        MAP_ADCIntClear(ADC0_BASE, 2);

         Read ADC Value.
        MAP_ADCSequenceDataGet(ADC0_BASE, 2, getADCValue);

        bgetConvStatus = true;
    }
}
```
可以看到以上都是照搬TI官方大大的源码，大家看了如果想深究的话可以去手册那边查每一个函数的作用，想快速通关的话直接往下看就可以了。

# 主函数调用
```
while(1)
{
    while(!bgetConvStatus);
    bgetConvStatus = false;
    adc_i=(float)getADCValue[3];
}
```
这里其实就是对标志位一直取反，然后把数组中的数据读出来，这里用的是PE3所以我只看这个数组【3】中的数据
如果想看别的也可以打印出来，但是其他的引脚为了偷懒我是没有初始化的，想看4个adc的可以去把例程下一遍。


# 测试现象

在代码下载进去时候可以把引脚分别接到GND和3V3上，然后看自己定义的变量（我这里是adc_i）分别为0和4096，
整体来说可以看作是线性的关系。


<font color=red size=3>切记不要接到5V上，这样会烧坏ADC的引脚的。




>**到这里大家就可以用adc采样了，不知道屏幕前的小伙伴成功了吗，我在TI小分队等你QQ：866325871**


<img src=" https://wang-gxi.github.io/my-photo/2323.jpeg" width="50%">