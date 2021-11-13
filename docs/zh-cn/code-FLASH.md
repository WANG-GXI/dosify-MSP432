# 8 FLASH

>**它更多的是保存重要数据，比如你在调pid，然后得到了三个不错的参数，但是身边又没有纸和笔，
这时可以写到flash中以防断电丢失，通过本章的学习， 你将掌握到eeprom的使用。**

<font color=black size=3>每次开始我们都会把TI官方的网站放到最前面，大家可以自己提前了解一手：[API Guide](https://dev.ti.com/tirex/explore/node?node=AOD4LXqFA8XEr21FehvtQA__J4.hfJy__LATEST)


# 初始化
```

float setDataforEEPROM[EEPROM_WORDLIMIT];
float getDatafromEEPROM[EEPROM_WORDLIMIT];
int eeprom_Init()
{

    MAP_SysCtlPeripheralEnable(SYSCTL_PERIPH_EEPROM0);
     while(!(SysCtlPeripheralReady(SYSCTL_PERIPH_EEPROM0)))   ;

     retInitStatus = MAP_EEPROMInit();
     /* If EEPROM did not initialize then exit the program code */
     if(retInitStatus != EEPROM_INIT_OK)      return 0;
         return 1;
}
```
使用时等初始化结束就可以直接使用了。

# 写数据

```
void write_eeprom(float data)
{
    int ii=0;
    //给数组赋值
    setDataforEEPROM[0] =data;// (setCurrTime << 8 | ii);
    MAP_EEPROMProgram(&setDataforEEPROM[0], 0, EEPROM_WORDLIMIT*4);
/*
 * getDatafromEEPROM[i]中的数据有32个，可以定义变量直接读取
 */

}
```
这里就是一个最简单的使用，只是写一个数字，当然我们可以自行修改，这就是C语言的事情了。

# 读数据

```
void read_eeprom()
{
    MAP_EEPROMRead(&getDatafromEEPROM[0], 0, EEPROM_WORDLIMIT*4);
/*
 * getDatafromEEPROM[i]中的数据有32个，可以定义变量直接读取
 */
}
```

由于我只用了一个数据，所以可以看到我读取的是数组的第0个数据。

>**到这里大家就会自行读写数据了，不知道屏幕前的小伙伴成功了吗，我在公众号“小鱼君code”等你**

<img src=" https://wang-gxi.github.io/my-photo/2323.jpeg" width="50%">