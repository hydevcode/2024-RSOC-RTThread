## 温湿度传感器

首先到Menuconfig开启下对应的软件包
![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407300009967.png)

保存后退出输入pkgs -update即可自动下载对应的代码

然后在代码目录下的packages即可看到刚刚下好的aht10的驱动了

虽然板子自带的是AHT21，但是这个驱动也是适配的了，而且温湿度传感器代码都差不多的，i2c地址正确就行

然后开始写下测试代码

```c

```

### 另外

如果需要输出小数，那么还需要添加rt_vsnprintf_full包

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407300027894.png)

## MQTT

首先来到阿里云物联网平台注册一个账号，并且选择开通试用一个实例

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407300034536.png)

然后新建一个产品和设备

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407300043147.png)


![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407300047004.png)
![](assets/image-20240730004735507.png)

接下来到menuconfig去开启下wifi模块的软件包

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407300049659.png)

还有阿里云的物联网平台的软件包
注意：这里需要设置下前四个选项
分别是产品key和密钥
以及设备名和密钥
这些都可以刚创建的产品和设备那里查看

下面还有个simple也开一下，这是实例

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407300050276.png)




