

## 相关概念
### 临界区

临界区的资源在同一时间内只能被一个线程使用，所以一旦临界资源被占用，其他的线程只能等待
举例子，全局变量被一个线程访问了就无法被其他线程使用

#### 线程阻塞

当资源被线程占用，其他线程无法执行就叫线程阻塞
a线程执行完后通知b线程，期间b线程也能继续运行的情况叫非阻塞式线程

### 挂起

![](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407251933550.png)

挂起线程就是把当前线程主动转让给其他线程
阻塞则是被动的因为某个资源没取到只能转让线程给其他线程

### 死锁

这个解释起来有点复杂，简单的解释例子就是两个线程互相等着对方给自己资源,于是造成了死锁
## 信号量

> 信号量(Semaphore)是一种**轻型**的用于解决线程间同步问题的内核对象，一个或多个运行线程可以获取或释放它,从而达到同步或互斥的目的。
> 用于实现任务与任务之间、任务与中断处理程序之间的同步与互斥。

信号量一般分为三种：
- 互斥信号量 
	 - 用于解决**互斥**问题。它比较特殊，可能会引起优先级反转问题
- 二值信号量
	- 用于解决**同步**问题
- 计数信号量 
	- 用于解决资源计数问题

>例子：红绿灯就类似信号量，公路上不同的汽车(类似线程)根据灯的颜色来决定停下来还是继续走
### 二值信号量

二值信号量主要用于 **线程与线程之间、线程与中断服务程序(ISR)** 之间的同步。
- 用于同步的二值信号量初始值为0，表示同步事件尚未产生；
- 线程获取信号量以等待该同步事件的发生；
- 另一个任务或 ISR 到达同步点时，释放信号量（将其值设置为1）表示同步事件已发生，唤醒等待的任务。

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407252022358.png)

> 例子：大人把蛋糕放桌子上，当桌子上的蛋糕从0变为1，说明有蛋糕了，那么小孩子发现有蛋糕了就取下来吃掉

## 互斥信号量

互斥量又叫相互排斥的信号量，是一种特殊的二值信号量。它和信号量不同的是，它支持：
- 互斥量所有权；
- 递归访问;
- 防止优先级翻转的特性。

### 优先级翻转

> 有优先级为A、B和C的三个线程，优先级A>B>C。线程A，B处于挂起状态，等待某一事件触发，线程C正在运行，此时线程C开始使用某一共享资源M。在使用过程中，线程A等待的事件到来，线程A转为就绪态，因为它比线程C优先级高，所以立即执行。但是当线程A要使用共享资源M时，由于其正在被线程C使用，因此线程A被挂起切换到线程C运行。如果此时线程B等待的事件到来，则线程B转为就绪态。由于线程B的优先级比线程C高，且线程B没有用到共享资源 M，因此线程B开始运行，直到其运行完毕，线程C才开始运行。只有当线程C释放共享资源M后，线程A才得以执行。在这种情况下，优先级发生了翻转：线程B先于线程A运行。这样便不能保证高优先级线程的响应时间。

说白了就是A优先级比C的高，当A线程执行的时候，C就让给A执行，A需要访问共享资源时发现被C用着，就切回C执行，这就是优先级翻转

之后轮到B执行，B优先级也高过C，B不需要访问共享资源，那就直接抢过来直到B运行完才轮到C

就这样C的执行时间被拖得很长，A也被卡住了，高优先级A线程响应时间不能保证

那么RT-Thread是怎么解决这个问题的

当A线程抢到执行权的时候，需要访问共享资源，于是让给C先跑，但是不能给B抢去了，于是把C的优先级暂时提高，直到C跑完，在接着让A，B线程抢

## 事件集

上面说的信号量都只能处理单个信号也就是有信号和没信号,接下来看下事件集
 
事件集是一个32bit的数，每个事件用一个bit位代表；触发方式有与触发、或触发
发送：可以从中断或者线程中进行发送
接收：线程接收，条件检查（逻辑与方式、逻辑或方式）

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407252129243.png)

类似于一个线程可以发送多个信号，接收信号的线程可以根据不同的信号做出不同的事情
## 消息邮箱

RT-Thread操作系统的邮箱用于线程间通信，特点是开销比较低，效率较高。邮箱中的每一封邮件只能容纳固定的4字节内容（针对32位处理系统，指针的大小即为4个字节，所以一封邮件恰好能够容纳一个指针）。典型的邮箱也称作交换消息，如下图所示，线程或中断服务例程把一封 4字节长度的邮件发送到邮箱中，而一个或多个线程可以从邮箱中接收这些邮件并进行处理。

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407252150215.png)

这东西相对于上面的机制来说比较复杂,最好根据API学习直接用一下会比较清楚一点

## 消息队列

在实际应用中我们常常遇到一些任务需要和其他任务之间进行数据交流，其实就是不同任
务之间的消息传递。在一般的裸机系统中可以通过全局变量来实现不同应用程序之间的数据交互和消息传递。在操作系统中为了更好的管理不同任务之间的消息传递我们引入消息队列的概念。

消息队列，也就是将多条消息排成的队列形式，是一种常用的线程间通信方式
可以应用在多种场合，线程间的消息交换，使用串口接收不定长数据等。线程可以将一条或多条消息放到消息队列中，同样一个或多个线程可以从消息队列中获得消息；同时消息队列提供异步处理机制可以起到缓冲消息的作用。

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407252220050.png)

使用消息队列实现线程间的异步通信工作，具有以下特性：
- 支持读消息超时机制
- 支持等待方式发送消息
- 允许不同长度（不超过队列节点最大值）任意类型消息
- 支持发送紧急消息
## 内容概括

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407252146580.png)

> 注：使用create API创建的东西，detach无法完全删除，最终还是要delete
> create是动态申请内存，init是静态
## 代码实验

实现：
信号量：线程一打印10次count，线程二控制led1亮一次

互斥量：线程三控制led2亮0.5秒，线程四控制led2亮1秒,表现为LED2亮一长一短

```
/*

 * Copyright (c) 2023, RT-Thread Development Team

 *

 * SPDX-License-Identifier: Apache-2.0

 *

 * Change Logs:

 * Date           Author       Notes

 * 2023-07-06     Supperthomas first version

 * 2023-12-03     Meco Man     support nano version

 */

  

#include <board.h>

#include <rtthread.h>

#include <drv_gpio.h>

#ifndef RT_USING_NANO

#include <rtdevice.h>

#endif /* RT_USING_NANO */

  

#define GPIO_LED_B    GET_PIN(F, 11) //LED1

#define GPIO_LED_R    GET_PIN(F, 12) //LED2

  

/*

信号量：线程一打印10次count，线程二控制led1亮一次

互斥量：线程三控制led2亮0.5秒，线程四控制led2亮1秒,表现为LED2亮一长一短

*/

#define THREAD_PRIORITY         25

#define THREAD_TIMESLICE        5

static rt_sem_t dynamic_sem = RT_NULL;

  

static char thread1_stack[1024];

static struct rt_thread thread1;

static void rt_thread1_entry(void *parameter)

{

    static rt_uint8_t count = 0;

    while (1)

    {

        if (count <= 100)

        {

            count++;

        }

        else

            return;

  

        /* count每计数10次，就释放一次信号量 */

        if (0 == (count % 10))

        {

            rt_kprintf("thread1 release a dynamic semaphore.\n");

            rt_sem_release(dynamic_sem);

        }

        rt_thread_mdelay(500);

    }

}

static char thread2_stack[1024];

static struct rt_thread thread2;

static void rt_thread2_entry(void *parameter)

{

    static rt_err_t result;

    while (1)

    {

        result = rt_sem_take(dynamic_sem, RT_WAITING_FOREVER);

        if (result != RT_EOK)

        {

            rt_kprintf("thread2 take a dynamic semaphore, failed.\n");

            rt_sem_delete(dynamic_sem);

            return;

        }

        else

        {

            rt_pin_write(GPIO_LED_B, PIN_LOW);

            rt_thread_mdelay(1000);

            rt_pin_write(GPIO_LED_B, PIN_HIGH);

            rt_kprintf("LED1 Open\n");

        }

    }

}

  

int semaphore_text(){

    /* 创建一个动态信号量，初始值是0 */

    dynamic_sem = rt_sem_create("dsem", 0, RT_IPC_FLAG_PRIO);

    if (dynamic_sem == RT_NULL)

    {

        rt_kprintf("create dynamic semaphore failed.\n");

        return -1;

    }

    rt_thread_init(&thread1,

                   "thread1",

                   rt_thread1_entry,

                   RT_NULL,

                   &thread1_stack[0],

                   sizeof(thread1_stack),

                   THREAD_PRIORITY, THREAD_TIMESLICE);

    rt_thread_startup(&thread1);

  

    rt_thread_init(&thread2,

                   "thread2",

                   rt_thread2_entry,

                   RT_NULL,

                   &thread2_stack[0],

                   sizeof(thread2_stack),

                   THREAD_PRIORITY, THREAD_TIMESLICE);

    rt_thread_startup(&thread2);

}

  

static rt_mutex_t dynamic_mutex = RT_NULL;

  

static char thread3_stack[1024];

static struct rt_thread thread3;

static void rt_thread_entry3(void *parameter)

{

    while (1)

    {

        rt_mutex_take(dynamic_mutex, RT_WAITING_FOREVER);

        rt_pin_write(GPIO_LED_R, PIN_LOW);

        rt_thread_mdelay(500);

        rt_pin_write(GPIO_LED_R, PIN_HIGH);

        rt_thread_mdelay(500);

        rt_mutex_release(dynamic_mutex);

    }

}

static char thread4_stack[1024];

static struct rt_thread thread4;

static void rt_thread_entry4(void *parameter)

{

    while (1)

    {

  

        rt_mutex_take(dynamic_mutex, RT_WAITING_FOREVER);

  

        rt_pin_write(GPIO_LED_R, PIN_LOW);

        rt_thread_mdelay(1000);

        rt_pin_write(GPIO_LED_R, PIN_HIGH);

        rt_thread_mdelay(500);

        rt_mutex_release(dynamic_mutex);

  

    }

}

  

int mutex_test(void)

{

    dynamic_mutex = rt_mutex_create("dmutex", RT_IPC_FLAG_PRIO);

    if (dynamic_mutex == RT_NULL)

    {

        rt_kprintf("create dynamic mutex failed.\n");

        return -1;

    }

    rt_thread_init(&thread3,

                   "thread3",

                   rt_thread_entry3,

                   RT_NULL,

                   &thread3_stack[0],

                   sizeof(thread3_stack),

                   THREAD_PRIORITY, THREAD_TIMESLICE);

  

    rt_thread_startup(&thread3);

  

    rt_thread_init(&thread4,

                   "thread4",

                   rt_thread_entry4,

                   RT_NULL,

                   &thread4_stack[0],

                   sizeof(thread4_stack),

                   THREAD_PRIORITY - 1, THREAD_TIMESLICE);

    rt_thread_startup(&thread4);

    return 0;

}

  

/* 导出到 msh 命令列表中 */

MSH_CMD_EXPORT(semaphore_text, semaphore text);

MSH_CMD_EXPORT(mutex_test, mutex test);

int main(void)

{

    rt_pin_mode(GPIO_LED_B, PIN_MODE_OUTPUT);

    rt_pin_mode(GPIO_LED_R, PIN_MODE_OUTPUT);

    rt_pin_write(GPIO_LED_B, PIN_HIGH);

    rt_pin_write(GPIO_LED_R, PIN_HIGH);

    return 0;

}
```

编译运行连接串口终端

输入semaphore_text或mutex_test即可查看对应效果