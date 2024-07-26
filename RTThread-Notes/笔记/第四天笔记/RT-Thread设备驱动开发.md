
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

I/O提供一套接口open write read controlclose SPII2CGPIORTCWDG特定API



