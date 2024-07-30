## 软件包
### 温湿度传感器

首先到Menuconfig开启下对应的软件包
![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407300009967.png)

保存后退出输入pkgs -update即可自动下载对应的代码

然后在代码目录下的packages即可看到刚刚下好的aht10的驱动了

虽然板子自带的是AHT21，但是这个驱动也是适配的了，而且温湿度传感器代码都差不多的，i2c地址正确就行

#### 另外

如果需要输出小数，那么还需要添加rt_vsnprintf_full包

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407300027894.png)

### MQTT

首先来到阿里云物联网平台注册一个账号，并且选择开通试用一个实例

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407300034536.png)

然后新建一个产品和设备

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407300043147.png)


![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407300047004.png)

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407300856900.png)

顺便添加几个模块，方便后面代码实践

接下来到menuconfig去开启下wifi模块的软件包,需要按图配置下针脚

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407300049659.png)

还有阿里云的物联网平台的软件包
注意：这里需要设置下前四个选项
分别是产品key和密钥
以及设备名和密钥
这些都可以刚创建的产品和设备那里查看

下面还有个simple也开一下，这是实例

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407300050276.png)

#### 代码实践

实现下上传温湿度到阿里云以及通过mqtt控制板子的led
为了方便理解，所以直接在mqtt提供的示例代码里面改

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407302002766.png)

初始化温湿度设备

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407302002359.png)

按下图实现温湿度上传

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407302003705.png)


按下图实现控制亮灯

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407302004786.png)

## 组件

定义：指的是一个可以独立开发、测试、部署和维护的软件单元。

软件包跟组件大概就是类似于房子跟建房子用的砖块把
就好像温湿度软件包里面使用了I2C组件

### 文件系统

定义:
>**DFS** 是 RT-Thread 提供的虚拟文件系统组件，全称为 Device File System。

>在 RT-Thread DFS 中，文件系统有统一的根目录，使用 `/` 来表示。
>有点类似linux

![DFS 层次架构图](https://www.rt-thread.org/document/site/rt-thread-version/rt-thread-standard/programming-manual/filesystem/figures/fs-layer.png)


| 类型    | 特点                                                                                                                                             |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| FatFS | FatFS 是专为小型嵌入式设备开发的一个兼容微软 FAT 格式的文件系统，采用ANSI C编写，具有良好的硬件无关性以及可移植性，是 RT-Thread 中最常用的文件系统类型。我们今天使用到的elm_fat就是这个类型。                               |
| RomFS | 传统型的 RomFS 文件系统是一种简单的、紧凑的、只读的文件系统，不支持动态擦写保存，按顺序存放数据，因而支持应用程序以 XIP(execute In Place，片内运行) 方式运行，在系统运行时, 节省 RAM 空间。我们一般拿其作为挂载根目录的文件系统             |
| DevFS | 即设备文件系统，在 RT-Thread 操作系统中开启该功能后，可以将系统中的设备在 `/dev` 文件夹下虚拟成文件，使得设备可以按照文件的操作方式使用 read、write 等接口进行操作。                                              |
| UFFS  | UFFS 是 Ultra-low-cost Flash File System（超低功耗的闪存文件系统）的简称。它是国人开发的、专为嵌入式设备等小内存环境中使用 Nand Flash 的开源文件系统。与嵌入式中常使用的 Yaffs 文件系统相比具有资源占用少、启动速度快、免费等优势。 |
| NFS   | NFS 网络文件系统（Network File System）是一项在不同机器、不同操作系统之间通过网络共享文件的技术。在操作系统的开发调试阶段，可以利用该技术在主机上建立基于 NFS 的根文件系统，挂载到嵌入式设备上，可以很方便地修改根文件系统的内容。                |

### POSIX接口层

POSIX 表示可移植操作系统接口（Portable Operating System Interface of UNIX，缩写 POSIX），POSIX 标准定义了操作系统应该为应用程序提供的接口标准，是 IEEE 为要在各种 UNIX 操作系统上运行的软件而定义的一系列 API 标准的总称。

学过linux的应该都知道当需要对文件进行读写操作，都需要用到read open write那些函数，这就是POSIX接口层了

以上都是介绍，下面了解下原理就开始实践了

### 文件系统启动流程

内容比较复杂，先浅浅的来个概念的理解先

这里分为两部分理解

第一部分就是初始化一个DFS组件，把文件系统初始化好，但是还没办法存储东西，因为缺少了类似硬盘一样的东西，于是有了第二部分

第二部分就是针对W25Q64的，首先用SFUD进行驱动，然后通过FAL抽象层注册为BLK设备，然后绑定到第一部分里，这样就可以访问存储了

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407302048989.png)


### 代码实践

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407302042248.png)

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407302043387.png)

接下来开一下组件

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407302044978.png)

按路径打开完后在进去fatfs里面设置一下

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407302045897.png)

顺便开下这个
![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407302155337.png)


然后保存编译运行即可使用

**注意**：由于Rw007模块和W25Q64用的同一个spi2所以还需要在main函数最底下空行加上这个代码
作用是初始化w25q64前先把Rw007关了
```
#define WIFI_CS GET_PIN(F, 10)

void WIFI_CS_PULL_DOWN(void)

{

    rt_pin_mode(WIFI_CS,PIN_MODE_OUTPUT);

    rt_pin_write(WIFI_CS, PIN_LOW);

}

INIT_BOARD_EXPORT(WIFI_CS_PULL_DOWN);
```

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407302116129.png)

每次运行都可以看到这个分区表，这个分区表差不多类似于电脑的硬盘的分区

这是RT-Thread默认分的分区表，虽然有分区了但是还没有格式化
(默认格式化了filesystem分区,也就是/下的fas文件夹)

首先格式化一下font分区，在此之前需要分配个块设备，相当于硬盘分配盘符d盘，e盘之类
![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407302130053.png)

然后格式化一下,运行后来到终端输入mkfs -t elm font

**注意**：第一次的新板子可能需要mkfs -t elm filesystem
mount filesystem /fal elm 分配下filesystem，然后重启,不然可能会报错
```
[684] E/app.filesystem: Failed to initialize filesystem!
```

继续，挂载font上去就能用了 
```
mkdir /fal/font
mouint font /fal/font elm
```

接下来可以写代码读写文件了
### 代码实践

实现以下每次mqtt发送温湿度的时候，另外存一份文件

>文件名为：Data.txt；
	文件内容： 
	Num：0 (代表总数)
	Temp：XX ; Humi：XX ; Count： 1（自上电起所采集的数据次数）
    Temp：XX ; Humi：XX ; Count： 2（自上电起所采集的数据次数）
    Temp：XX ; Humi：XX ; Count： 3（自上电起所采集的数据次数）






