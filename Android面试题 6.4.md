### Android Handler机制是做什么，原理了解吗？

#### 主要涉及的角色如下所示：

- Message：消息，分为硬件产生的消息（例如：按钮、触摸）和软件产生的消息
- MessageQueue：消息队列，主要用来向消息池添加消息和取走消息
- Looper：消息循环器，主要用来把消息分发给相应的处理者
- Handler：消息处理器，主要向消息队列发送各种消息以及处理各种消息

#### 整个消息的循环流程还是比较清晰的，具体来说：

- 1、Handler通过sendMessage()发送消息Message到消息队列MessageQueue
- 2、Looper通过loop()不断提取触发条件的Message，并将Message交给对应的target handler来处理
- 3、target handler调用自身的handleMessage()方法来处理Message
- 事实上，在整个消息循环的过程中，并不只有java层参与，很多重要的工作都是在C++层来完成
- 在四个主要的类中MessageQueue是Java层与C++层维系的桥梁，MessageQueue与Looper相关功能通过MessageQueue的Native方法来完成

### Android Binder机制是做什么，为什么选用Binder，原理了解吗？

- Android Binder是用来做进程通信的，因为Android的各个应用以及系统服务都运行在独立的进程中，它们的通信都依赖于Binder

#### 为什么选用Binder，我们知道Android基于Linux内核，Linux现有的进程通信手段有以下几种：

- 1、管道：在创建时分配一个页大小的内存，缓存区大小比较有限
- 2、消息队列：要将信息复制两次，额外的CPu消耗；不合适频繁或信息量大的通信
- 3、共享内存：无需复制，共享缓冲区直接附加到进程虚拟地址空间，速度快；但进程间的同步问题操作系统无法实现，必须各进程利用同步工具解决
- 4、套接字：作为更通用的接口，传输效率低，主要用于互通及其或跨网络的通信
- 5、信号量：常作为一种锁机制，防止某进程正在访问共享资源时，其他进程也访问该资源。因此，主要作为进程间以及同一进程内不同线程之间的同步手段
- 6、信号：不适用于信息交换，更适用于进程中断控制，比如非法内存访问，杀死某个进程等

#### 既然有现有的IPC方式，为什么要重新设计一套Binder机制，主要是出于以上三个方面的考量：

- 高性能：从数据拷贝次数来看Binder只需要进行一次内存拷贝，而管道、消息队列、Socket都需要两次，共享内存不需要拷贝，Binder的性能仅此于共享内存
- 稳定性：上面说到共享内存性能优于Binder，那为什么不适用共享内存呢，因为共享内存需要处理并发同步问题，控制负责，容易出现死锁和资源竞争，稳定性较差。而Binder基于C/S架构，客户端与服务端彼此独立，稳定性较好
- 安全性：Android为每个应用分配了UID，用来作为鉴别进程的重要标志，Android内部也依赖这个UID进行权限管理，包括6.0以前的固定权限和6.0以后的动态权限，传统IPC只能由用户在数据包里填入UID/PID，这个标记完全是在用户控件控制，没有放在内核控件，因此有被恶意篡改的可能，因此Binder的安全性更高

### 描述一下Activity的生命周期，这些生命周期是如何管理的？

- 1、比方说点击跳转一个新Activity，这个时候Activity会入栈，同时它的生命周期也会从onCreate()到onResume()开始变化，这个过程是在ActivityStack里完成的，ActivityStack是运行在Server进程里的，这个时候Server进程就通过ApplicationThread的代理对象ApplicationThreadProxy向运行在app进程ApplicationThread发起操作请求
- 2、ApplicationThread接收到操作请求后，因为它是运行在app进程里的其他线程里，所以ApplicationThread需要通过Handler向主线程ActivityThread发送操作消息
- 3、主线程接收到ApplicationThread发出的消息后，调用主线程ActivityThread执行响应的操作，并回调Activity相应的周期方法

### Activity的通信方式有哪些？

- startActivityForResult
- EventBus
- LocalBroadcastReceiver

### Android应用里有几种Context对象

- Context是一个抽象类，它的具体实现类是ContextImpl，ContextWrapper是个包装类，内部的成员变量mBase指向的也是个ContextImpl对象，ContextImpl完成了实际的功能，Activity、Service与Application都直接或者间接的继承ContextWrapper

### 描述一下进程和Application的生命周期？

#### 一个安装的应用对应一个LoadedApk对象，对应一个Application对象，对于四大组件，Application的创建和获取方式也是不尽相同的

- Activity：通过LoadedApk的makeApplication()方法创建
- Service：通过LoadedApk的makeApplication()方法创建
- 静态广播：通过其回调方法onReceive()方法的第一个参数指向Application
- ContentProvider：无法获取Application，因此此时Application不一定已经初始化

### Android哪些情况会导致内存泄漏，如何分析内存泄漏？

- 持有==静态的Context==(Activity)引用
- 持==有静态的View==引用
- 内部类&匿名内部类实例无法释放（有延迟时间等等），而**内部类又持有外部类的强引用，导致外部类无法释放**，这种匿名内部类常见于==监听器、Handler、Thread、TimerTask==
- 资源使用完成后没有关闭，例如：==BroadcastReceiver、ContentObserver，File，Cursor，Stream，Bitmap==
- 不正确的单例模式，比如单例持有Activity
- **集合类内存泄漏**，如果一个集合类是静态的（缓存HashMap),只有添加方法，没有对应的删除方法，会导致引用无法被释放，引发内存泄漏
- 错误的覆写了==finalize==()方法，finalize()方法执行不确定，可能会导致引用无法被释放