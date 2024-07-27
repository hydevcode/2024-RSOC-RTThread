
- RT-ThreadI/O设备框架概念 
- RT-Thread I/O API
- GPIO应用与驱动开发
- I2C从机应用与驱动开发 SPI从机应用与驱动开发

## RT-ThreadI/O设备框架概念

### 驱动开发问题思考

>不同厂家的MCU关于SPI接口API的设计都不一样,代码也不一样
>那么对于程序员来说，当需要更换某个模块的时候还要重新学一次代码这样就太麻烦了
>RT-Thread尽量将代码进行合适的分离，达到代码复用的效果，于是就有了框架

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407262055744.png)

使用了SPI统一API将设备驱动跟SPi驱动分开
使得：
```
- 更换MCU只需要改变对应的对接驱动
- 重新驱动设备，只需要重新编写设备驱动相关的代码
- 同一API接口，学习成本低
- 分离后设备驱动可以入库，供公司其他项目使用，减少碎片化开发，防止反复造轮子
- 代码框架会变复杂，但是从上面的优点来看是值得的
```

### 框架演进

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407262059817.png)
![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407262100832.png)

### 驱动框架分析

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407262102060.png)

 ![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407270336071.png)

I/O提供一套接口open write read control close SPI I2C GPIO RTC WDG特定API

### I/O派生设备种类

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407270404800.png)


字符设备和块设备的特点与区别：
·字符设备：提供连续的数据流，应用程序可以顺序读取，通常不支持随机存取。相反，
此类设备支持按字节/字符来读写数据。举例来说，键盘、串口、调制解调器都是典型的字符设备
·块设备：应用程序可以随机访问设备数据，程序可自行确定读取数据的位置。硬盘、
软盘、CD-ROM驱动器和闪存都是典型的块设备，应用程序可以寻址磁盘上的任何位置，并由此读取数据。此外，数据的读写只能以块(通常是512B)的倍数进行。与字符设备不同，块设备并不支持基于字符的寻址。
总结一下，这两种类型的设备的根本区别在于它们是否可以被随机访问。字符设备只能顺序读取，块设备可以随机读取。


为什么要对设备分类

MSH可以重定向到任意的字符设备上，例如将lcd模拟成字符设备，就可以将打印输出到LCD上，或者是实现一套空字符设备，将msh重定向到这里。
Fatfs文件系统依赖块设备驱动，我们将SD卡读写实现成块设备，但是也可以用ram 来模拟块设备驱动，
不同的组件和应用会依赖不同的设备，对设备进行分类，可以做到对一类设备同样的控制


串口设备（RTDeviceClassChar） SDIO 网卡 （RT_Device_Class_SDIO)
CS43L22(音频codec）（RT_Device_Class_Sound) GPIO (RT Device Class Pin)
LCD屏幕（RT_Device_Class_Graphic）录音驱动（RTDeviceClassSound）