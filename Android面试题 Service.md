## Android面试题 Service

### Service是否在main Thread中执行，Service里面是否能执行耗时的操作

- 默认情况下，如果没有显式地指Service所运行的进程，Service和Activity都是运行在当前app所在的进程main Thread(UI主线程)里面
- Service里面不能执行耗时的操作
- 特殊情况，可以在清单文件配置Service执行所在的进程，让Service在另外的进程中执行
```
<service android:name="com.baidu.location.f" 
android:enabled="true" android:process=":remote" />

```

### Activity怎么和Service绑定，怎么在Activity中启动自己对应的Service

- Activity通过bindService(Intent service，ServiceConnection conn，int flags)跟Service进行绑定，当绑定成功的时候Service会将代理对象通过回调的形式传给conn，这样我们就拿到了Service提供的服务代理对象
- 在Activity可以通过startService和bindService方法启动Service。
- 一般情况下如果想获取Service的服务对象，那么需要通过bindService()方法，比如音乐播放器，第三方支付等。如果仅仅是为了开启一个后台服务那么可以使用startService()方法

### Service的生命周期

- Service有绑定模式和非绑定模式，以及这两种模式的混合使用方式。不同的使用方式生命周期方法也不同

#### 非绑定模式

- 当第一次调用startService的时候执行的方法依次是onCreate()->onStartCommand()，当Service关闭的时候调用onDestroy()方法

#### 绑定模式

- 第一次bindService()的时候，执行的方法为onCreate()->onBind()，解除绑定的时候会执行onUnbind()->onDestory()


#### 注意

- 上面的两种生命周期都是在相对单纯的模式下的情形
- 我们在开发的过程中还必须注意Service实例只会有一个，也就是说如果当前要启动的Service已经存在了那么久不会再次创建该Service，当然也不会调用onCreate()方法了
- 一个Service可以被多个客户进行绑定，只有所有的绑定对象都执行了onBind()方法后该Service才会销毁
- 如果有一个客户执行了onStart()方法，那么这个时候如果所有的bind客户都执行了onUnBind()，该Service也不会销毁

### IntentService

#### 简介

- IntentService是Service的子类，比普通的Service增加了额外的功能
- Service不会专门启动一条单独的进程，Service与它所在应用位于同一个进程中，即主线程
- Service也不是专门一条新线程，因此不应该在Service中直接处理耗时的任务

#### 特征

- 会创建独立的工作线程来处理所有的Intent请求
- 会创建独立的工作线程来处理onHandleIntent()方法实现的代码，无需处理多线程问题
- 所有请求处理完成后，IntentService会自动停止，无需调用stopSelf()方法停止Service
- 为Service的onBind()提供默认实现，返回null
- 为Service的onStartCommand()提供默认实现，将请求的Intent添加到队列中

### Activity、Intent、Service是什么关系

- 他们都是Android开发中使用频率最高的类
- Activity和Service都是Android四大组件之一，都是Context类的子类，ContextWrapper的子类
- Activity负责用户界面的显示和交互，Service负责后台任务的处理
- Activity和Service之间可以通过Intent传递数据，因此可以把Intent看做是两者的通信使者

### Service和Activity在同一个线程吗

- 对于同一App来说默认情况下是在同一个线程中的main Thread中

### Service里面可以弹Toast吗

- 可以的。
- 弹Toast有一个条件就是得有一个Context上下文，而Service本身就是Context的子类，因此在Service里面弹Toast是可以的，比如我们可以在Service中完成下载任务后弹出一个Toast通知用户

### 什么是Service以及描述下它的生命周期。Service有哪些启动方法，有什么区别，怎么停用Service？

### 两种启动方式

#### 通过startService

- Service会经历onCreate()到onStartCommand()，然后处于运行状态，stopService的时候调用onDestroy()方法
- 如果是调用者自己直接退出而没有调用stopService的话，Service会一直在后台运行

#### 通过bindService

- Service会运行onCreate()，然后是调用onBind，这个时候调用者和Service绑定在一起。调用者退出了，Service就会调用onUnbind->onDestroy()方法
- 所谓的绑定在一起就是共存亡了。
- 如果先是 bind 了,那么 start 的时候就直接运行 Service 的 onStart 方法,如 果先是 start,那么 bind 的时候就直接运行 onBind 方法。 
- 如果 service 运行期间调用了 bindService,这时候再调用 stopService 的话,service 是不会调用 onDestroy 方法的,service 就 stop 不掉了,只能调用 UnbindService, service 就会被销毁 
- 如果一个 service 通过 startService 被 start 之后,多次调用 startService 的 话,service 会多次调用 onStart 方法。多次调用 stopService 的话,service 只会调用一次 onDestroyed 方法。
- 如果一个 service 通过 bindService 被 start 之后,多次调用 bindService 的话, service 只会调用一次 onBind 方法。 
多次调用 unbindService 的话会抛出异常

### 在Service的生命周期方法onStartCommand()可不可以执行网络操作？如何在Service中执行网络操作？

- 可以直接在Service中执行网络操作，在onStartCommand()方法中可以执行网络操作

### 如何提高Service的优先级

- 我们可以用setForeground(true)来设置Service的优先级

#### 为什么是foreground

- 因为默认启动的Service是被标记为后台进程，当前运行的Activity一般被标记为前台进程，也就是说给Service设置了foreground那么它就和正在运行的Activity类似优先级得到了一定的提高
- 但这并不能保证你的Service永远不被杀掉，只是提高了它的优先级

### Service如何定时执行

- 当启动Service进行后台任务的时候，我们一般的做法是启动一个线程，然后通过sleep方法来控制进行定时的任务，但这样容易被系统回收
- Service被回收时我们不能控制的，但我们可以控制Service的重启活动，在Service的onStartCommand方法中可以返回一个参数来控制重启活动
- 使用AlarmManager，根据AlarmManager的工作原理，AlarmManager会定时的发出一条广播，然后在自己的项目里注册这个广播，重写onReceive()方法，在这个方法里面启动一个Service，然后在Service里面进行网络的访问操作，当获取到新消息的时候进行推送，同时再设置一个AlarmManager进行下一次的轮询，当本次轮询结束的时候可以stopself结束该Service。这样即使这次的轮询失败了，也不会影响到下一次的轮询，这样就能保证推送任务不会中断

### Service的onStartCommand方法有几种返回值？各代表什么意思？

- START_STICKY：如果Service进程被kill掉，保留Service的状态为开始状态，但不保留递送的Intent对象。随后系统会尝试重新创建Service，由于服务状态为开始状态，所以创建服务后一定会调用onStartCommand(Intent,int,int)方法。如果在此期间没有任何启动命令被传递到Service，那么参数Intent将为null
- START_NOT_STICKY："非粘性的"，使用这个返回值时，如果在执行完onStartCommand后，服务被异常kill掉，系统不会自动重启该服务
- START_REDELIVER_INTENT：重传Intent，使用这个返回值时，如果在执行完onStartCommand后，服务被异常kill掉，系统会自动重启该服务，并将Intent的值传入
- START_STICKY_COMPATIBILITY：START_STICKY的兼容版本，但不保证服务被kill后一定能重启

### Service的onRebind(Intent)方法在什么情况下会执行？

- 如果在onUNbind()方法返回true的情况下会执行，否则不执行

### Activity调用Service中的方法都有哪些方式？

#### Binder

- 通过Binder接口的形式实现，当Activity绑定Service成功的时候，Activity会在ServiceConnection类的onServiceConnected()回调方法中获取到Service的onBind()方法return过来的Binder的子类，然后通过对象调用方法

#### AIDL

- AIDL比较适合当客户端和服务端不在同一个应用下的场景

#### Messenger

- 它引用了一个Handler对象，以便其他能够向它发送消息(使用mMessenger.send(Message msg)方法)。该类允许跨进程间基于Message的通信(即两个进程间可以通过Message进行通信)，在服务端使用Handler创建一个Messenger，客户端持有这个Messenger就可以与服务端通信了。一个Messenger不能同时双向发送，两个就能双向发送了

### Service如何给Activity发送给Message？

- Service和Activity如果想护发Message就必须使用Messenger机制

### IntentService适用场景

- IntentService内置的是HandlerThread作为异步线程，每一个交给IntentService的任务都将以队列的方式逐个被执行到，一旦队列中有某个任务执行时间过长，那么久会导致后续的任务都会被延迟处理
- 正在运行的IntentService的程序相比起纯粹的后台程序更不容易被系统杀死，该程序的优先级是介于前台程序与纯后台程序之间