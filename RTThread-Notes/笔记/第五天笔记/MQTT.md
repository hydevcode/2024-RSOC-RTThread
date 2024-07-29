## 软件包
## 温湿度传感器

首先到Menuconfig开启下对应的软件包
![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407300009967.png)

保存后退出输入pkgs -update即可自动下载对应的代码

然后在代码目录下的packages即可看到刚刚下好的aht10的驱动了

虽然板子自带的是AHT21，但是这个驱动也是适配的了，而且温湿度传感器代码都差不多的，i2c地址正确就行

### 另外

如果需要输出小数，那么还需要添加rt_vsnprintf_full包

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407300027894.png)

## MQTT

首先来到阿里云物联网平台注册一个账号，并且选择开通试用一个实例

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407300034536.png)

然后新建一个产品和设备

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407300043147.png)


![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407300047004.png)

接下来到menuconfig去开启下wifi模块的软件包,需要按图配置下针脚

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407300049659.png)

还有阿里云的物联网平台的软件包
注意：这里需要设置下前四个选项
分别是产品key和密钥
以及设备名和密钥
这些都可以刚创建的产品和设备那里查看

下面还有个simple也开一下，这是实例

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407300050276.png)

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