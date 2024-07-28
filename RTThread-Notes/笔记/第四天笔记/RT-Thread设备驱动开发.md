
- RT-ThreadI/O设备框架概念 
- RT-Thread I/O API
- GPIO应用与驱动开发
- I2C从机应用与驱动开发 SPI从机应用与驱动开发

## RT-ThreadI/O设备框架概念

### 驱动开发问题思考

>不同厂家的MCU关于SPI接口API的设计都不一样,代码也不一样
>那么对于程序员来说，当需要更换某个模块的时候还要重新学一次代码这样就太麻烦了

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

RT-Thread尽量将代码进行合适的分离，达到代码复用的效果，于是就有了框架
### 框架演进

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407262059817.png)
![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407262100832.png)

可以看到经过几次的改进，框架中出现了一个I/O层，这个东西将硬件跟上层开发分开来，使得我们开发可以更专注于某个层级

### 驱动框架分析

看完了框架图，再看下代码方面，下图则是专注于I/O以下部分的代码

所有的设备都可以用rt_device表示,只需要继承下来在对设备具体的功能进行编写就好了

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407262102060.png)


![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407272343668.png)


RT-Thread驱动框架层里面就已经写好了一些常用的驱动设备类比如说：
SPI I2C GPIO RTC WDG特定API
所以我们接下来先学习如何使用这些类
### I/O框架

下图是针对I/O框架在应用层的使用，为了跟上面的图区分开特地说下,防止搞乱了
 ![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407270336071.png)

I/O提供一套接口open write read control close
我们在使用的时候就按照这个顺序调用我们需要设备驱动就可以了

然后我们还需要了解一些概念
下图是RT-Thread支持的设备驱动类型

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407272346146.png)

### 字符设备和块设备的特点与区别：

- 字符设备：提供连续的数据流，应用程序可以顺序读取，通常不支持随机存取。相反，
此类设备支持按字节/字符来读写数据。举例来说，键盘、串口、调制解调器都是典型的字符设备
- 块设备：应用程序可以随机访问设备数据，程序可自行确定读取数据的位置。硬盘、
软盘、CD-ROM驱动器和闪存都是典型的块设备，应用程序可以寻址磁盘上的任何位置，并由此读取数据。此外，数据的读写只能以块(通常是512B)的倍数进行。与字符设备不同，块设备并不支持基于字符的寻址。

总结一下，这两种类型的设备的根本区别在于它们是否可以被随机访问。字符设备只能顺序读取，块设备可以随机读取。

### 为什么要对设备分类

MSH可以重定向到任意的字符设备上，例如将lcd模拟成字符设备，就可以将打印输出到LCD上，或者是实现一套空字符设备，将msh重定向到这里。
Fatfs文件系统依赖块设备驱动，我们将SD卡读写实现成块设备，但是也可以用ram 来模拟块设备驱动，
不同的组件和应用会依赖不同的设备，对设备进行分类，可以做到对一类设备同样的控制

串口设备（RTDeviceClassChar） 
SDIO 网卡 （RT_Device_Class_SDIO)
CS43L22(音频codec）（RT_Device_Class_Sound) 
GPIO (RT Device Class Pin)
LCD屏幕（RT_Device_Class_Graphic）
录音驱动（RTDeviceClassSound）

## 代码实践

接下来简单实现一下创建自己的设备驱动

正如上面的框架图，创建设备驱动应该是要包括I/O层及往下的实现

作为应用层和硬件层的桥梁，我们要根据RT-Thread给我们提供的一套规范实现I/O 设备以及 I/O 设备管理接口

这样应用层才可以通过桥梁找到设备并使用他
#### 创建I/O 设备

注册一个名为dev的字符设备,相当于告诉RT-Thread我要建一座桥，桥在这里，大概的框架长什么样之类的

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407280121317.png)

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407280123591.png)

编写好创建的代码后就可以编译运行了，之后在终端输入rt_device_test就可以运行这段代码了
list device可以查看当前以及创建了的设备驱动

### 相关API

- rt_device_t rt_device_create(int type, int attach_size);
- void rt_device_destroy(rt_device_t device);
- rt_err_t rt_device_register(rt_device_t dev, const char* name,rt_uint8_t flags);
![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407280119833.png)
- rt_err_t rt_device_unregister(rt_device_t dev);

### 创建I/O 设备管理接口

在写代码前，我们先来看下刚才创建的rt_device里面有什么

首先是一个基类对象，这里跟上面的框架分析里的类继承图是对应上的
然后是另一个比较重点的ops，设备操作方法
这个RT-Thread提供的一套I/O 设备管理接口规范，主要是让建好的桥梁可以跟应用层对接上

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407280155026.png)

代表了设备的一些操作，可以看到他们都是函数指针，意味着我们要自己实现里面的函数并让这些指针指向实现的函数，这样应用层就可以使用了

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407280154544.png)

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407280203905.png)
比如说，当应用层调用rt_device_init，其实就是调用了另一个函数init

## I2C总线

常见的I2C总线以传输速率的不同分为不同的模式：
低速模式：10Kbit/s
标准模式：100Kbit/s
快速模式：400Kbit/s
高速模式：3.4Mbit/s

I2C主要是用来跟一些如传感器、EEPROM、RTC之类的设备进行通信

下图是可以看到三根横线代表了Vdd，SDA，SCL，然后主控作为Master主机发送，Slave外部设备接收数据，Vdd可以暂时不管，一般来说就是一根3.3V的线

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407280623234.png)

接下来看下SDA和SCL线，这是主要负责I2C的

>起始位和结束位：
> > 起始位（S）：在SCL为高电平时，SDA由高电平变为低电平
> > 结束位（P）：在SCL为高电平时，SDA由低电平变为高电平

通过改变SDA和SCL的高低电平即可实现数据传输

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407290252847.png)

然后就是实现数据传输还要遵循I2C的传输协议

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407280623897.png)

从上面的主从机图来看，可以发现I2C总线上可能不单单挂着一个设备
那么跟设备通讯就需要从设备地址

所以一开始需要主设备发送从设备地址，然后从设备收到后知道是来找他的那么就返回一个ACK响应，并准备接收数据，然后主设备就发数据，发完后，从设备发个响应确认接收完毕，那么这次的的数据传输就结束了

应答和非应答需要的高低电平:
- 应答（ACK）：拉低SDA线，并在SCL为高电平期间保持SDA线为低电平
- 非应答（NOACK）：不要拉低SDA线（此时SDA线为高电平），并在SCL为高电平期间保持SDA线为高电平

### 代码实践



### 一些可能的问题

i2c总线死锁：

