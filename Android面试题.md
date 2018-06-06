# Android面试题

### 描述下Android系统结构图，各个层次的作用

- Android应用框架层
- Java系统框架层
- C++系统框架层
- Linux内核层

### Activity与Service通信？

- 可以通过bindService的方式，先在Activity里实现一个==ServiceConnection接口==，并将该接口传递给bindService()方法，在ServiceConnection接口的onServiceConnected()方法里执行相关操作

### Service的生命周期与启动方法有什么区别？

#### 启动方法

- startService():开启Service，调用者退出后Service仍然存在
- bindService():开发Service，调用者退出后Service也随机退出

#### 生命周期

- 只是用startService()启动服务：onCreate->onStartCommand()->onDestory
- 只是用bindService()绑定服务：onCreate->onBind->onUnBind->onDestory
- 同时使用startService()启动服务与bindService()绑定服务：onCreate->onStartCommand->onBind->onUnBind->onDestory

### Service先start再bind如何关闭Service,为什么bindService可以跟Activity生命周期联动？

#### 针对stopService和unbindService：

- 先stopService，则不会调用任何方法，然后当unbindService时，依次调用onUnbind和onDestroy方法
- 先unbindService，则会调用onUnbind，然后当stopService时，会调用onDestroy

#### bindService跟Activity生命周期联动？

### 广播分为哪几种？应用场景是什么？

- 普通广播：调用sendBroadcast()发送，最常用的广播
- 有序广播：调用sendOrderedBroadcast()，发出去的广播会被广播接受者按照顺序接收，广播接收者按照==Prority优先级==排序，Prority属性相同者，动态注册的广播优先，广播接收者还可以选择对广播进行截断和修改

### 广播的两种注册方式有什么区别？

- 静态注册：常驻系统，==不受组件生命周期影响==，即便应用退出，广播还是可以被接收，耗电、占内存
- 动态注册：非常驻，跟随组件的生命变化，组件结束，广播结束。==在组件结束前，需要先移除广播==，否则容易造成内存泄漏

### 广播传输的数据是否有限制，是多少，为什么要限制？

### ContentProvider、ContentResolver与ContentObserver之间的关系是什么？

- ContentProvider：==管理数据，提供数据的增删改查操作==。数据源可以使数据库、文件、XML、网络等，ContentProvider为这些数据的访问提供了统一的接口，可以用来做==进程间数据共享==
- ContentResolver：ContentResolver可以不同URI操作不同的ContentProvider中的数据，外部进程可以通过ContentResolver与ContentProvider进行交互
- ContentObserver：观察ContentProvider中的数据变化，并将变化通知给外界

### 遇到过哪些关于Fragment的问题，如何处理的？

- getActivity()空指针：这种情况一般发生在==异步任务==里调用getActivity()，而Fragment已经onDetach()，此时就会有空指针；解决方案是在Fragment里使用一个全局变量mActivity，在onAttach()方法里赋值，这样可能会引起内存泄漏，但是异步任务没有停止的情况下本身就已经可能内存泄漏，相比直接crash，这种方式显得更妥当一些
- Fragment视图重叠：在类onCreate()的方法加载Fragment，并且没有判断**saveInstance==null或if(findFragmentByTag(mFragmentTag)==null)**，导致重复加载了同一个Fragment导致重叠。(PS:replace情况下，如果没有加入回退栈，则不判断也不会造成重叠，但建议还是统一判断下)
```
@Override 
protected void onCreate(@Nullable Bundle savedInstanceState) {
// 在页面重启时，Fragment会被保存恢复，而此时再加载Fragment会重复加载，导致重叠 ;
    if(saveInstanceState == null){
    // 或者 if(findFragmentByTag(mFragmentTag) == null)
       // 正常情况下去 加载根Fragment 
    } 
}
```

### Android里的Intent传递的数据有大小限制吗，如何解决？

#### Intent传递数据大小的限制大概在1M左右，超过这个限制就会崩溃 处理方式如下：

- 进程内：EventBus，文件缓存，磁盘缓存
- 进程间：通过ContentProvider进行进程数据共享和传递

### 描述一下Android的事件分发机制

- Android事件分发机制的本质：事件从哪个对象发出，经过哪些对象，最终由哪个对象处理了该事件。此处对象指的是Activity、Window与View
- Android的分发顺序：Activity(Window)->ViewGroup->View
- Android事件的分发主要由三个方法完成
```
// 父View调用dispatchTouchEvent()开始分发事件
public boolean dispatchTouchEvent(MotionEvent event){
    boolean consume = false;
    // 父View决定是否拦截事件
    if(onInterceptTouchEvent(event)){
        // 父View调用onTouchEvent(event)消费事件，如果该方法返回true，表示
        // 该View消费了该事件，后续该事件序列的事件（Down、Move、Up）将不会在传递
        // 该其他View。
        consume = onTouchEvent(event);
    }else{
        // 调用子View的dispatchTouchEvent(event)方法继续分发事件
        consume = child.dispatchTouchEvent(event);
    }
    return consume;
}
```

### 描述一下View的绘制原理

#### View的绘制流程主要分为三步：

- 1、onMeasure：测量视图的大小，从顶层父View到子View递归调用到measure()方法，measure()中调用onMeasure()方法，onMeasure()方法完成测量工作
- 2、onLayout：确定视图的位置，从顶层父View到子View递归调用layout()方法，父View将上一步measure()方法得到的子View的布局大小和布局参数，将子View放到合适的位置上
- 3、onDraw：绘制最终的视图，首先ViewRoot创建一个Canvas对象，然后调用onDraw()方法进行绘制。onDraw()方法的绘制流程：1)绘制视图背景。2)绘制画布的图层。3)绘制View内容。4)绘制子视图。5)还原图层。6)绘制滚动条

#### requestLayout()、invalidate()与postInvalidate()有什么区别？

- requestLayout()：该方法会递归调用父窗口的requestLayout()方法，直到触发ViewRootImpl的performTraversals()方法，此时mLayoutRequestde为true，**会触发onMeasure()与onLayout()方法，不一定会触发onDraw()方法**
- invalidate()：该方法递归调用父View的invalidateChildInParent()方法，直到调用ViewRootImpL的invalidateChildInParent()方法，最终触发ViewRootImpL的performTraversals()方法，此时mLayoutRequestede为false，**不会触发onMeasure()与onLayout()方法，此时会触发onDraw()方法**
- postInvalidate()：该方法功能和invalidate()一样，只是它可以在==非UI线程==中调用
- 一般来说需要重新布局就调用requestLayout()方法，需要重新绘制就调用invalidate()方法

### Scroller用过吗，了解它的原理吗？

### 了解APK的打包流程吗？描述一下

#### Android的包文件APK分为两个部分：代码和资源，所以打包方面也分为资源打包和代码打包

- 通过AAPT工具进行资源文件的打包，生成R.java文件
- 通过AIDL工具处理AIDL文件，生成相应的java文件
- 通过javac工具编译项目源码，生成.class文件
- 通过DX工具将所有的Class文件转换成DEX文件，该过程主要完成java字节码转换成Dalvik字节码，压缩常量池以及清楚冗余信息等工作
- 通过APKBuilder工具将资源文件、DEX文件打包成APK文件
- 利用KeyStore对生成的APK文件进行签名

### 了解APK的安装流程吗，描述一下？

- 复制APK到/data/app目录下，解压并扫描安装包
- 资源管理器解析APK里的资源文件
- 解析AndroidManifest文件，并在/data/data目录下创建对应的应用数据目录
- 然后对dex文件进行优化，并保存在dalvik-cache目录下
- 将AndroidManifest文件解析出的四大组件信息注册到PackageManagerService中
- 安装完成，发送广播

### 当点击一个应用图标以后，都发生了什么，描述一下这个过程？

#### 点击应用图标后会启动应用的LauncherActivity，如果LauncherActivity所在的进程没有创建，还会创建新进程，整体的流程就是一个Activity的启动流程

##### 整个流程涉及的主要角色有：

- Instrumentation：监控应用与系统相关的交互行为
- AMS：组件管理调度中心，什么都不干，但是什么都管
- ActivityStarter：Activity启动的控制器，处理Intent与Flag对Activity启动的影响，具体来说：1、寻找符合启动条件的Activity，如果有多个，让用户进行选择；2、校验启动参数的合法性；3、返回int参数，代表Activity是否启动成功
- ActivityStackSupervisor：管理任务栈
- ActivityThread：最终干活的人，是ActivityThread内部类，Activity、Service、BroadcastReceiver的启动、切换、调度等各种操作都在这个类里完成
- 注：ActivityStackSupervisor是高版本才有的类，用来管理多个ActivityStack，早起的版本只有一个ActivityStack对应着手机屏幕，后来高版本支持多屏以后，就有了多个ActivityStack

##### 整个流程主要涉及四个进程

- 调用者进程：如果是在桌面启动应用就是Launcher应用进程
- ActivityManagerService等所在的System Server进程：该进程主要运行着系统服务组件
- Zygote进程：该进程主要用来fork新进程
- 新启动的应用进程：该进程就是用来承载应用运行的进程，它也是应用的主线程（新创建的进程就是主线程），处理组件生命周期、界面绘制等相关事情

##### 整个流程：

- 1、点击桌面应用图标，Launcher进程将启动Activity(MainActivity)的请求以Binder的方式发送给AMS
- 2、AMS接收到启动请求后，交付ActivityStarter处理Intent和Flag等信息，然后再交给ActivityStackSupervisor/ActivityStack处理Activity进栈相关流程，同时以Socket方式请求Zygote进程fork新进程
- 3、Zygote接收到新进程创建请求后fork出新进程
- 4、在新进程里创建ActivityThread对象，新创建的进程就是应用的主线程，在主线程里开启Looper消息循环，开始处理创建Activity
- 5、ActivityThread利用ClassLoader去加载Activity、创建Activity实例，并回调Activity的onCreate()方法，这样便完成了Activity的启动

### BroadcastReceiver与LocalBroadcastReceiver有什么区别？

- BroadcastReceiver是跨应用广播，利用Binder机制实现
- LocalBroadcastReceiver是应用内广播，利用Handler实现，利用了IntentFilter的match功能，提供消息的发布与接收功能，实现应用内通信，效率比较高
