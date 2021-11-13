## 一、硬件
  	我们独立设计了一款拓展版作为日常使用。在2021电赛也十分感谢它，我们没有烧任何东西。嘿嘿

<img src=" https://wang-gxi.github.io/my-photo/2244.png" width="70%">

<font color=black size=3>&ensp;&ensp;这里只是为大家提供一种类似于32开发板的方法。网上也有更牛的大佬直接画了特别小的板子。我们就不献丑了。<font color=black size=3>


<font color=red size=3>&ensp;&ensp;电路的设计思路就简要说下：<font color=red size=3>


<img src=" https://wang-gxi.github.io/my-photo/2245.png" width="70%">


<font color=black size=3>&ensp;&ensp;为了防止出现什么离谱的问题，我们在降压时用了两个降压芯片，这样可以把一路电源给单片机，另一路给外设。<font color=black size=5>

#  二、自我开发时如何选择引脚
<font color=black size=3>

&ensp;&ensp;还是查手册。那就有人问了，这么简单还需要说嘛？其实确实没什么可说的。但是我们用32的角度开发时，会遇到下面的问题：<font color=black size=3>

<img src=" https://wang-gxi.github.io/my-photo/2247.png" width="90%">

&ensp;&ensp;我们想要寻找到串口时在右上角搜索栏输入的是USART，但是在432的官方手册中却什么也没有找到。难道是没有吗？其实是在432中串口的名字并不是USART，而是``` U1RX、U1TX ``` ,类似的还有``` U2RX、U2TX``` 之类
<img src=" https://wang-gxi.github.io/my-photo/2246.png" width="90%">

&ensp;&ensp;其实官方的手册给的很详细，大家可以在群文件中直接下载。

#  三、CCS（Code Composer Studio）的安装

&ensp;&ensp;这可是一个大工程，大家可以自行CSDN，也可以在QQ群文件中下载文档安装。

&ensp;&ensp;其中包括新建工程，导入官方代码等基操。大家务必要学习后再开始之后的学习。
<img src=" https://wang-gxi.github.io/my-photo/2321.png" width="90%">
