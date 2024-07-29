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




