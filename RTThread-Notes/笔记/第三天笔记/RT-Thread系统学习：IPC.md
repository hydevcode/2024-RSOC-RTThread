

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

#### 二值信号量相关API

##### 创建信号量

```
rt_sem_t rt_sem_create(const char *name,rt_uint32_t value, rt_uint8_t flag);
```

>当调用这个函数时，系统将先从对象管理器中分配一个semaphore对象，并初始化这个对象，然后初始化父类IPC对象以及与semaphore相关的部分。
>在创建信号量指定的参数中，信号量标志参数决定了当信号量不可用时，多个线程等待的排队方式。
>FLAG可以设置为两种方式：
>当选择RT_IPC_FLAG_FIFO（先进先出）方式时，那么等待线程队列将按照先进先出的方式排队，先进入的线程将先获得等待的信号量；
>当选择RT_IPC_FLAG_PRIO（优先级等待）方式时，等待线程队列将按照优先级进行排队优先级高的等待线程将先获得等待的信号量。

##### 删除信号量

```
rt_err_t rt_sem_delete(rt_sem_t sem); 
```

>系统不再使用信号量时，可通过删除信号量以释放系统资源。
>当调用这个函数时，系统将删除这个信号量。
>如果删除该信号量时，有线程正在等待该信号量，那么删除操作会先唤醒等待在该信号量上的线程（等待线程的返回值是－RT_ERROR），然后再释放信号量的内存资源。

##### 初始化信号量

```
rt_err_t rt_sem_init(rt_sem_t sem,
	const char *name,
	rt_uint32_t value,
	rt_uint8_t flag)
```

>当调用这个函数时，系统将对这个semaphore对象进行初始化，然后初始化IPC对象以及与 semaphore相关的部分。信号量标志可用上面创建信号量函数里提到的标志。

##### 创建信号量

```
rt_sem_t rt_sem_create(const char *name,rt_uint32_t value, rt_uint8_t flag);
```

>当调用这个函数时，系统将先从对象管理器中分配一个semaphore对象，并初始化这个对象，然后初始化父类IPC对象以及与semaphore相关的部分。
>在创建信号量指定的参数中，信号量标志参数决定了当信号量不可用时，多个线程等待的排队方式。
>当选择RT_IPC_FLAG_FIFO（先进先出）方式时，那么等待线程队列将按照先进先出的方式排队，先进入的线程将先获得等待的信号量；
>当选择RT_IPC_FLAG_PRIO（优先级等待）方式时，等待线程队列将按照优先级进行排队优先级高的等待线程将先获得等待的信号量。

##### 初始化信号量

```
rt_err_t rt_sem_init(rt_sem_t sem,
	const char *name,
	rt_uint32_t value,
	rt_uint8_t flag)
```

>当调用这个函数时，系统将对这个semaphore对象进行初始化，然后初始化IPC对象以及与 semaphore相关的部分。信号量标志可用上面创建信号量函数里提到的标志。

##### 脱离信号量

```
rt_err_t rt_sem_detach(rt_sem_t sem);
```

>脱离信号量就是让信号量对象从内核对象管理器中脱离，适用于静态初始化的信号量。
>使用该函数后，内核先唤醒所有挂在该信号量等待队列上的线程，然后将该信号量从内核对象管理器中脱离。原来挂起在信号量上的等待线程将获得-RT_ERROR的返回值。

##### 获取信号量

```
rt_err_t rt_sem_take (rt_sem_t sem,rt_int32_t time);
```

> 线程通过获取信号量来获得信号量资源实例，当信号量值大于零时，线程将获得信号量，并且相应的信号量值会减1。
>在调用这个函数时，如果信号量的值等于零，那么说明当前信号量资源实例不可用，申请该信号量的线程将根据time参数的情况选择直接返回、或挂起等待一段时间、或永久等待，直到其他线程或中断释放该信号量。如果在参数time指定的时间内依然得不到信号量，线程将超时返回，返回值是-RT_ETIMEOUT。

##### 无等待获取信号量

```
rt_err_t rt_sem_trytake(rt_sem_t sem);
```

>当用户不想在申请的信号量上挂起线程进行等待时，可以使用无等待方式获取信号量。
>这个函数与rt_sem_take(sem，RT_WAITING_NO）的作用相同，即当线程申请的信号量资源实例不可用的时候，它不会等待在该信号量上，而是直接返回－RTETIMEOUT。

##### 释放信号量

```
rt_err_t rt_sem_release(rt_sem_t sem);
```

>释放信号量可以唤醒挂起在该信号量上的线程
>当信号量的值等于零时，并且有线程等待这个信号量时，释放信号量将唤醒等待在该信号量线程队列中的第一个线程，由它获取信号量；否则将把信号量的值加1。

## 互斥信号量

互斥量又叫相互排斥的信号量，是一种特殊的二值信号量。它和信号量不同的是，它支持：
- 互斥量所有权；
- 递归访问;
- 防止优先级翻转的特性。

> 例子：还是以上面那个蛋糕为例子，当桌子上有蛋糕，而又有很多小孩子想吃蛋糕，这时年纪最大的小孩子吃的时候，其他小孩子就没法去吃了，但是仔细想想，好像这跟二值信号量差不多了，查了一下，二值信号量也可以实现互斥，但是不如互斥信号量那样严格地控制访问，那其实以后基本上使用互斥量不就没问题了？而且哪怕二值信号量多用于同步，但是互斥量好像也能用于同步？这里不太懂他们之间严格的区别，如果有具体的例子就好了比如说只有二值信号量才能解决的问题


### 优先级翻转

> 有优先级为A、B和C的三个线程，优先级A>B>C。线程A，B处于挂起状态，等待某一事件触发，线程C正在运行，此时线程C开始使用某一共享资源M。在使用过程中，线程A等待的事件到来，线程A转为就绪态，因为它比线程C优先级高，所以立即执行。但是当线程A要使用共享资源M时，由于其正在被线程C使用，因此线程A被挂起切换到线程C运行。如果此时线程B等待的事件到来，则线程B转为就绪态。由于线程B的优先级比线程C高，且线程B没有用到共享资源 M，因此线程B开始运行，直到其运行完毕，线程C才开始运行。只有当线程C释放共享资源M后，线程A才得以执行。在这种情况下，优先级发生了翻转：线程B先于线程A运行。这样便不能保证高优先级线程的响应时间。

说白了就是A优先级比C的高，当A线程执行的时候，C就让给A执行，A需要访问共享资源时发现被C用着，就切回C执行，这就是优先级翻转

之后轮到B执行，B优先级也高过C，B不需要访问共享资源，那就直接抢过来直到B运行完才轮到C

就这样C的执行时间被拖得很长，A也被卡住了，高优先级A线程响应时间不能保证

![](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407251933718.png)

那么RT-Thread是怎么解决这个问题的

当A线程抢到执行权的时候，需要访问共享资源，于是让给C先跑，但是不能给B抢去了，于是把C的优先级暂时提高，直到C跑完，在接着让A，B线程抢

![](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407251933935.png)

### 互斥信号量相关API

#### 创建互斥量

```
rt_mutex_t rt_mutex_create (const char* name, rt_uint8_t flag);
```

>调用 rt_mutex_create 函数创建一个互斥量，它的名字由 name 所指定。当调用这个函数时，系统将先从对象管理器中分配一个mutex对象，并初始化这个对象，然后初始化父类IPC对象以及与mutex相关的部分。**互斥量的flag标志已经作废** ，无论用户选择RT_IPC_FLAG_PRIO还是 RTIPCFLAGFIFO，内核均按照RTIPCFLAGPRIO处理。

#### 删除互斥量

```
rt_err_t rt_mutex_delete(rt_mutex_t mutex);
```

>当不再使用互斥量时，通过删除互斥量以释放系统资源，适用于动态创建的互斥量。
>当删除一个互斥量时，所有等待此互斥量的线程都将被唤醒，等待线程获得的返回值是
>－RT_ERROR。然后系统将该互斥量从内核对象管理器链表中删除并释放互斥量占用的内存空间。

#### 初始化互斥量

```
rt_err_t rt_mutex_init (rt_mutex_t mutex， const char* name,rt_uint8_t flag); 
```

> 静态互斥量对象的内存是在系统编译时由编译器分配的，一般放于读写数据段或未初始化数据段中。在使用这类静态互斥量对象前，需要先进行初始化。
>使用该函数接口时，需指定互斥量对象的句柄（即指向互斥量控制块的指针），互斥量名称以及互斥量标志。互斥量标志可用上面创建互斥量函数里提到的标志。

#### 脱离互斥量

```
rt_err_t rt_mutex_detach （rt_mutex_t mutex);
```

>当使用该函数接口后，内核先唤醒所有挂在该互斥量上的线程（线程的返回是-RTERROR），然后系统将该互斥量从内核对象管理器中脱离。

#### 获取互斥量

```
rt_err_t rt_mutex_take (rt_mutex_t mutex,rt_int32_t time);
```

>线程获取了互斥量，那么线程就有了对该互斥量的所有权，即某一个时刻一个互斥量只能被一个线程持有。
>如果互斥量没有被其他线程控制，那么申请该互斥量的线程将成功获得该互斥量。如果互斥量已经被当前线程线程控制，则该互斥量的持有计数加1，当前线程也不会挂起等待。如果互斥量已经被其他线程占有，则当前线程在该互斥量上挂起等待，直到其他线程释放它或者等待时间超过指定的超时时间。

#### 无等待获取互斥量

```
rt_err_t rt_mutex_trytake(rt_mutex_t mutex);
```

>线程获取了互斥量，那么线程就有了对该互斥量的所有权，即某一个时刻一个互斥量只能被一个线程持有。
>如果互斥量没有被其他线程控制，那么申请该互斥量的线程将成功获得该互斥量。如果互斥量已经被当前线程线程控制，则该互斥量的持有计数加1，当前线程也不会挂起等待。如果互斥量已经被其他线程占有，则当前线程在该互斥量上挂起等待，直到其他线程释放它或者等待时间超过指定的超时时间。

#### 释放取互斥量

```
rt_err_t rt_mutex_release(rt_mutex_tmutex);
```

>当线程完成互斥资源的访问后，应尽快释放它占据的互斥量，使得其他线程能及时获取该互斥量。
>使用该函数接口时，只有已经拥有互斥量控制权的线程才能释放它，每释放一次该互斥量，它的持有计数就减1。
>当该互斥量的持有计数为零时（即持有线程已经释放所有的持有操作），它变为可用等待在该互斥量上的线程将被唤醒。
>如果线程的运行优先级被互斥量提升，那么当互斥量被释放后线程恢复为持有互斥量前的优先级。

## 事件集

上面说的信号量都只能处理单个信号也就是有信号和没信号,接下来看下事件集
 
事件集是一个32bit的数，每个事件用一个bit位代表；触发方式有与触发、或触发
发送：可以从中断或者线程中进行发送
接收：线程接收，条件检查（逻辑与方式、逻辑或方式）

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407252129243.png)

类似于一个线程可以发送多个信号，接收信号的线程可以根据不同的信号做出不同的事情

### 事件集相关API

#### 创建事件

```
rt_event_t rt_event_create(const char* name,rt_uint8_t flag);
```

>调用该函数接口时，系统会从对象管理器中分配事件集对象，并初始化这个对象，然后初始化父类 IPC 对象。

#### 删除事件

```
rt_err_t rt_event_delete(rt_event_t event);
```

>系统不再使用rt_event_create()创建的事件集对象时，通过删除事件集对象控制块来释放系统资
源。
>在调用rt_event_delete 函数删除一个事件集对象时，应该确保该事件集不再被使用。在删除前会唤醒所有挂起在该事件集上的线程（线程的返回值是－RT_ERROR），然后释放事件集对象占用的内存块。

#### 初始化事件

```
rt_err_t rt_event_init(rt_event_t event, const char* name, rt_uint8_t flag);
```

>静态事件集对象的内存是在系统编译时由编译器分配的，一般放于读写数据段或未初始化数据段中。在使用静态事件集对象前，需要先行对它进行初始化操作。
调用该接口时，需指定静态事件集对象的句柄（即指向事件集控制块的指针），然后系统会初始化事件集对象，并加入到系统对象容器中进行管理。

#### 脱离事件

```
rt_err_t rt_event_detach(rt_event_t event);
```

>系统不再使用rt_event_init()初始化的事件集对象时，通过脱离事件集对象控制块来释放系统资源。脱离事件集是将事件集对象从内核对象管理器中脱离。
>用户调用这个函数时，系统首先唤醒所有挂在该事件集等待队列上的线程（线程的返回值是 -RT_ERROR），然后将该事件集从内核对象管理器中脱离

#### 发送事件

```
rt_err_t rt_event_send(rt_event_t event, rt_uint32_t set); 
```

发送事件函数可以发送事件集中的一个或多个事件。
使用该函数接口时，通过参数set指定的事件标志来设定event事件集对象的事件标志值，然后遍历等待在event事件集对象上的等待线程链表，判断是否有线程的事件激活要求与当前event 对象事件标志值匹配，如果有，则唤醒该线程。

#### 接收时间

```
rt_err_t rt_event_recv(rt_event_t event,
	rt_uint32_t set,
	rt_uint8_t option,
	rt_int32_t timeout,
	rt_uint32_t* recved);
```

内核使用32位的无符号整数来标识事件集，它的每一位代表一个事件，因此一个事件集对象可同时等待接收32个事件，内核可以通过指定选择参数“逻辑与”或“逻辑或”来选择如何激活线
程，使用“逻辑与”参数表示只有当所有等待的事件都发生时才激活线程，而使用“逻辑或" 参数则表示只要有一个等待的事件发生就激活线程。

> 当用户调用这个接口时，系统首先根据set参数和接收选项option来判断它要接收的事件是否发生，如果已经发生，则根据参数option 上是否设置有 RT_EVENT_FLAG_CLEAR 来决定是否重置事件的相应标志位，然后返回（其中recved参数返回接收到的事件）；
> 如果没有发生，则把等待的set和option参数填入线程本身的结构中，然后把线程挂起在此事件上，直到其等待的事件满足条件或等待时间超过指定的超时时间。如果超时时间设置为零，则表示当线程要接受的事件没有满足其要求时就不等待，而直接返回－RT_ETIMEOUT。

> 注：使用create API创建的东西，detach无法完全删除，最终还是要delete
> create是动态申请内存，init是静态
## 消息邮箱

RT-Thread操作系统的邮箱用于线程间通信，特点是开销比较低，效率较高。邮箱中的每一封邮件只能容纳固定的4字节内容（针对32位处理系统，指针的大小即为4个字节，所以一封邮件恰好能够容纳一个指针）。典型的邮箱也称作交换消息，如下图所示，线程或中断服务例程把一封 4字节长度的邮件发送到邮箱中，而一个或多个线程可以从邮箱中接收这些邮件并进行处理。

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407252150215.png)

这东西相对于上面的机制来说比较复杂,最好根据API学习直接用一下会比较清楚一点

### 消息邮箱API

#### 创建消息邮箱

```
rt_mailbox_t rt_mb_create (const char* name, rt_size_t size, rt_uint8_t flag);
```

>创建邮箱对象时会先从对象管理器中分配一个邮箱对象，然后给邮箱动态分配一块内存空间用来存放邮件，这块内存的大小等于邮件大小（4字节）与邮箱容量的乘积，接着初始化接收邮件数目和发送邮件在邮箱中的偏移量。

#### 删除邮箱

```
rt_err_t rt_mb_delete (rt_mailbox_t mb);
```

当用 rt_mb_create(创建的邮箱不再被使用时，应该删除它来释放相应的系统资源，一旦操作完成，邮箱将被永久性的删除。
删除邮箱时，如果有线程被挂起在该邮箱对象上，内核先唤醒挂起在该邮箱上的所有线程（线程返回值是-RT_ERROR），然后再释放邮箱使用的内存，最后删除邮箱对象。

#### 初始化邮箱

```
rt_err_t rt_mb_init(rt_mailbox_t mb,
	const char* name,
	void* msgpool, 
	rt_size_t size, 
	rt_uint8_t flag)
```

>初始化邮箱跟创建邮箱类似，只是初始化邮箱用于静态邮箱对象的初始化。与创建邮箱不同的是，静态邮箱对象的内存是在系统编译时由编译器分配的，一般放于读写数据段或未初始化数据段中，其余的初始化工作与创建邮箱时相同。
>
>初始化邮箱时，该函数接口需要获得用户已经申请获得的邮箱对象控制块，缓冲区的指针，以及邮箱名称和邮箱容量（能够存储的邮件数）。

#### 脱离邮箱

```
rt_err_t rt_mb_detach(rt_mailbox_t mb); 
```

脱离邮箱将把静态初始化的邮箱对象从内核对象管理器中脱离。
使用该函数接口后，内核先唤醒所有挂在该邮箱上的线程（线程获得返回值是－RT_ERROR），然后将该邮箱对象从内核对象管理器中脱离。

#### 发送邮件

```
rt_err_t rt_mb_send (rt_mailbox_t mb, rt_uint32_t value); 
```

>线程或者中断服务程序可以通过邮箱给其他线程发送邮件。
>
>发送的邮件可以是32位任意格式的数据，一个整型值或者一个指向缓冲区的指针。当邮箱中的邮件已经满时，发送邮件的线程或者中断程序会收到-RT_EFULL的返回值。

#### 等待方式发送邮件

```
rt_err_t rt_mb_send_wait (rt_mailbox_t mb,
	rt_uint32_t value,
	rt_int32_t timeout);
```

>rt_mb_send_wait()与rt_mb_send()的区别在于有等待时间，如果邮箱已经满了，那么发送线程将根据设定的timeout 参数等待邮箱中因为收取邮件而空出空间。
>如果设置的超时时间到达依然没有空出空间，这时发送线程将被唤醒并返回错误码。
>不可以在中断中用

#### 发送紧急邮件

```
rt_err_t rt_mb_urgent (rt_mailbox_t mb, rt_ubase_t value); 
```

>发送紧急邮件的过程与发送邮件几乎一样，唯一的不同是，当发送紧急邮件时，邮件被直接插队放入了邮件队首，这样，接收者就能够优先接收到紧急邮件，从而及时进行处理。

#### 接收邮件

```
rt_err_t rt_mb_recv （rt_mailbox_t mb, rt_uint32_t* value, rt_int32_t timeout);
```

>只有当接收者接收的邮箱中有邮件时，接收者才能立即取到邮件并返回 RT_EOK 的返回值，否则接收线程会根据超时时间设置，或挂起在邮箱的等待线程队列上，或直接返回。
>
>接收邮件时，接收者需指定接收邮件的邮箱句柄，并指定接收到的邮件存放位置以及最多能够等待的超时时间。如果接收时设定了超时，当指定的时间内依然未收到邮件时，将返回 -RT_ETIMEOUT.

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

消息支持发送紧急消息
线程或中断服务例程可以将一条或多条消息放入消息队列中。同样，一个或多个线程也可以从消息队列中获得消息。当有多个消息发送到消息队列时，通常将先进入消息队列的消息先传给线程，也就是说，线程先得到的是最先进入消息队列的消息，即先进先出原则(FIFO)。

在创建消息队列指定的参数中，事件集标志参数决定了当消息不可获取时，多个线程等待的排队方式。
- 当选择RTIPCFLAGFIFO（先进先出）方式时，那么等待线程队列将按照先进先出的方式排队，先进入的线程将先获得等待的消息;
- 当选择RT_IPC_FLAG_PRIO（优先级等待）方式时，等待线程队列将按照优先级进行排队，优先级高的等待线程将先获得等待的消息。

创建消息队列时先从对象管理器中分配一个消息队列对象，然后给消息队列对象分配一块内存空间，组织成空闲消息链表，这块内存的大小=[消息大小 +消息头(用于链表连接) 的大小」＊消息队列最大个数
接着再初始化消息队列；
接口返回RTEOK表示动态消息队列创建成功。

```
rt_mq_t rt_mq_create (const char* name,
// 消息队列名称
rt_size_t msg_size, //消息队列中一条消息的最大长度，单位字节
rt_size_t max_msgs,// 消息队列的最大个数
rt_uint8_t flag) ;//消息队列采用的等待方式
参数flag 可以取如下数值：RT IPC FLAG FIFO(先进先出)或 RT IPC FLAG PRIO(优
先级等待)

```


![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407252259136.png)

#### 发送消息

```
rt_err_t rt_mq_send (rt_mq_t mq，// 消息队列对象的句柄
	void* buffer，// 消息内容
	rt_size_t size)；//消息大小
```
线程或者中断服务程序都可以给消息队列发送消息。当发送消息时，消息队列对象先从空闲消息链表上取下一个空闲消息块，把线程或者中断服务程序发送的消息内容复制到消息块上然后把该消息块挂到消息队列的尾部。当且仅当空闲消息链表上有可用的空闲消息块时，发送者才能成功发送消息；当空闲消息链表上无可用消息块，说明消息队列已满，此时，发送消息的的线程或者中断程序会收到一个错误码（-RTEFULL）。

#### 等待方式发送消息

```
rt_err_t rt_mq_send_wait(rt_mq_t mq,
//消息队列对象的句柄
	const void *buffer, 
	rt_size_t size,
// 消息内容// 消息大小
	rt_int32_t timeout);
// 超时时间
```

等待发送消息的函数接口如下，rt_mq_send_wait()与rt_mq_send()的区别在于有等待时间，如果消息队列已经满了，那么发送线程将根据设定的timeout参数进行等待。如果设置的超时时间到达依然没有空出空间，这时发送线程将被唤醒并返回错误码。

#### 发送消息

```
rt_err_t rt_mq_send (rt_mq_t mq,
//消息队列对象的句柄
void*buffer,
//消息内容
rt_size_tsize)；//消息大小
```

线程或者中断服务程序都可以给消息队列发送消息。当发送消息时，消息队列对象先从空闲消息链表上取下一个空闲消息块，把线程或者中断服务程序发送的消息内容复制到消息块上，然后把该消息块挂到消息队列的尾部。当且仅当空闲消息链表上有可用的空闲消息块时，发送者才能成功发送消息；当空闲消息链表上无可用消息块，说明消息队列已满，此时，发送消息的的线程或者中断程序会收到一个错误码（-RT_EFULL）。发送消息的函数接口如下：

#### 发送紧急信息

```
rt_err_t rt_mq_urgent (rt_mq_t mq,
//消息队列对象的句柄
void* buffer,
// 消息内容
rt_size_t size)；// 消息大小
```

发送紧急消息的过程与发送消息几乎一样，唯一的不同是，当发送紧急消息时，从空闲消息链表上取下来的消息块不是挂到消息队列的队尾，而是挂到队首，这样，接收者就能够优先接收到紧急消息，从而及时进行消息处理。

#### 接收消息

```
rt_err_t rt_mq_recv (rt_mq_t mq,//消息队列对象的句柄
void* buffer,
//消息内容
rt_size_t size,
　//消息大小
rt_int32_t timeout）；//指定的超时时间
```

当消息队列中有消息时，接收者才能接收消息，否则接收者会根据超时时间设置，或挂起在消息队列的等待线程队列上，或直接返回。

接收消息时，接收者需指定存储消息的消息队列对象句柄，并且指定一个内存缓冲区，接收到的消息内容将被复制到该缓冲区里。此外，还需指定未能及时取到消息时的超时时间，接
收一个消息后消息队列上的队首消息被转移到了空闲消息链表的尾部。

## 内容概括

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407252146580.png)


那么接下来尝试一下代码

首先env里面输入menuconfig.exe配置一下
![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407260125978.png)
图片里最上面是路径，然后空格打开这个包，回车进入

![image.png](https://gitee.com/alicization/2024-rsoc-rtthread/raw/master/imgs/202407260126519.png)

把需要的都打开
然后保存退出

由于是在线的库，所以还需要下载
输入pkgs --update



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