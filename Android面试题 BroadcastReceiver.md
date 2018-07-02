## Android面试题 BroadcastReceiver

### BroadcastReceiver简介

- 四大组件之一，主要用于接收系统或者app发送的广播事件
- 广播分为两种：有序广播和无序广播
- 内部通信实现机制：通过Android系统的Binder机制实现通信

#### 无序广播

- 完全异步，逻辑上可以被任何广播接收者接收到
- 优点：效率较高
- 缺点：一个接收者不能将处理结果传递给下一个接收者，并无法终止广播Intent的传播

#### 有序广播

- 按照被接收者的优先级顺序，在被接收者中依次传播
- 每个接收者有权终止广播，当广播被终止，后续的广播接收者将不能接受到广播
- 当A接受到广播后可以对结果对象进行操作，当广播传给B时，B可以从结果对象中取得A存入的数据

#### 最终接收者resultReceiver

- 通过
```
Context.sendOrderedBroadcast(intent, receiverPermission, 
resultReceiver, scheduler, initialCode, initialData, 
initialExtras)
```
我们可以指定resultReceiver广播接收者
- 通常情况下如果比他优先级更高的接收者没有终止广播，那么他的onReceive会被执行两次，第一次是正常的按照优先级顺序执行，第二次是作为最终接收者接收
- 如果比他优先级高的接收者终止了广播，那么他依然能接收到广播
- 在项目中疆场使用广播接收者接收系统通知，比如开机启动、sd挂载、低电量、外拨电话、锁屏等
- Android4.0之后，如果系统自动关闭广播接收者所在进程，在广播中的action跟广播接收者的action匹配时，系统会启动该广播所在的进程，但是如果是用户手动关闭该进程，则不会自启动，只有等用户手动开启
- 广播接收者所在进程如果从来没有启动过，那么广播接收者不会生效

### 在manifest和代码中如何注册和使用BroadcastReceiver

- 在清单文件中注册广播接收者称为静态注册，在代码中注册称为动态注册
- 静态注册的广播接收者只要app在系统中运行则一直可以接收到广播消息
- 动态注册的广播接收者当注册的Activity或者Service销毁了那么就接收不到广播了
```
//静态注册，在清单文件中进行如下配置
<receiver android:name=".BroadcastReceiver1" > <intent-filter>
<action android:name="android.intent.action.CALL" >
            </action>
        </intent-filter>
</receiver> 
```
```
//动态注册，在代码中进行如下注册
receiver = new BroadcastReceiver();
IntentFilter intentFilter = new IntentFilter(); 
intentFilter.addAction(CALL_ACTION); 
context.registerReceiver(receiver, intentFilter);
```
#### 两种注册类型的区别

- 1、静态注册是常驻型，也就是说当应用程序关闭后，如果有信息广播来，程序也会被系统调用自动运行
- 2、动态注册不是常驻型广播，也就是说广播跟随程序的组件(比如Activity)生命周期

### 不同注册方式广播接收器回调onReceive(context,intent)中Context类型不一致

- manifest静态注册的ContextReceiver，回调onReceive(context,intent)中的Context是ReceiverRestrictedContext
- 代码动态注册的ContextReceiver，回调onReceive(context,intent)中的context是Activity Context
- 使用LocalBroadcastManager动态注册的ContextReceiver，回调onReceive(context,intent)中的context是Application Context

### BroadcastReceiver的生命周期

- 广播接收者的生命周期非常短暂，在接收到广播的时候创建，onReceive()方法结束之后销毁
- 广播接收者中不要做一些耗时的工作，否则会弹出ANR错误对话框
- 最好也不要再广播接收这中创建子线程做耗时的操作，因为广播接收这被销毁后进程就成为了空进程，很容易被系统杀掉
- 耗时的较长的工作最好放在服务中完成

### Android引入广播机制的用意

- 从 MVC 的角度考虑(应用程序内)，其实回答这个问题的时候还可以这样问, Android为什么要有那4大组件,现在的移动开发模型基本上也是照搬的web那一套MVC架构,只不过是改了点嫁妆而已。android的四大组件本质上就是为了实现移动或者说嵌入式设备上的MVC架构,它们之间有时候是一种相互依存的关系,有时候又是一种补充关系,引入广播机制可以方便 几大组件的信息和数据交互。 
- 程序间互通消息(例如在自己的应用程序内监听系统来电)
- 效率上(参考UDP的广播协议在局域网的方便性)
- 设计模式(反转控制的一种应用，类似监听者模式)

### 如何让自己的广播只让指定的app接收

- 通过自定义广播权限来保护自己发出的广播
- 在清单文件里receiver西部有这个权限才能收到广播
- 首先需要定义权限
- 然后声明权限
- 这时接收者就能收到发送的广播

### 广播优先级对无序广播生效吗

- 生效

### 动态注册的广播优先级谁高

- 谁先注册谁的优先级高

### 如何判断当前BroadcastReceiver接收到的是有序广播还是无序广播？

- 在BroadcastReceiver类中onReceive()方法中，可以调用 Boolean b = isOrderedBroadcast()来判断接收到的广播是否为有序广播

### 粘性广播有什么作用，怎么使用

- 粘性广播主要为了解决，在发送完广播之后，动态注册的接收者，也能够收到广播
- 举个例子：首先发送一个广播，接收者是通过程序中某个按钮动态注册的，如果不是粘性广播，注册完接收者肯定无法收到广播了，这是通过发送粘性广播就能够在动态注册接收者后也能收到广播
```
//发送粘性广播
Public void sendStickyBroadCast(){ 
          Intent intent=new Intent(); 
           intent.setAction(“com.example.receiver.action”);
           intent.putExtra(“name”,”tom”); 
           this.sendStickyBroadCast(intent); 
} 
```
- 发送粘性广播还需要发送粘性广播的权限：
```
<uses-permission android:name="android.permission.BROADCAST_STICKY" />
```

### LocalBroadcastManager(局部通知管理器)

- 在android-support-v4.jar中引入了LocalBroadcastManager
- LocalBroadcastManager：除了能够解决BroadcastReceiver进程间安全性问题外，相对Context操作的BroadcastReceiver而言还具有更高的运行效率
- 本地广播通过LocalBroadcastManager.getInstance(context).sendBroadcast(intent)发送广播
- LocalBroadcastManager.getInstance(context).registerReceiver注册服务，通过LocalBroadcastManager.getInstance(context).unregisterReceiver取消注册服务，其他同普通广播。 
- BroadcastReceiver的通信机制是通过Binder机制实现的，LocalBroadcastManager的核心实现实际是Handler，只是利用到了IntentFilter的match功能，因为Handler实现的应用内的通信，自然安全性更好，效率更高
- 通常使用BroadcastReceiver进行工作线程的任务结果通知也好，还是进程间安全性问题，容易引起性能问题，那么使用LocalBroadcastManager.getInstance有效提高了安全性和性能
- LocalBroadcastManager方式发送的应用内广播，只能通过LocalBroadcastManager动态注册的ContextReceiver才有可能接收到（静态注册或其他方式动态注册的ContextReceiver是接收不到的）

#### 应用场景

- App内各个组件通信(单个或多个线程之间)，局部刷新等
- 不同App之间的组件之间消息通信
- 接收系统广播

#### 安全性

- BroadcastReceiver可以方便应用程序和系统、应用程序之间、应用程序内的通信，所以对单个应用程序而言BroadcastReceiver是存在组件安全性问题的
- 这就让组件暴露在外，建议开发过程中不要暴露内部组件，如果有特殊需求也需进行权限控制，使用这些组件时需要申请相应权限

##### 权限控制(permission)

- 发送某个广播时系统会将发送的Intent与系统中所有注册的BroadcastReceiver的IntentFilter进行匹配，若匹配成功则执行相应的onReceive函数
- 通过类似sendBroadcast(Intent,String)的接口在发送广播时指定接收者必须必备的permission，或通过Intent.setPackage设置广播仅对某个程序有效

##### 声明android：exported="false"

- android：exported是Android中的四大组件Activity、Service、BroadcastReceiver、ContentProvider四大组件都会有的一个属性
- 主要作用：是否支持其它应用程序调用当前组件，默认值：如果包含intent-filter默认值为true;没有intent-filter默认值为false。如果组件无须跨进程交互，则不应设置exported属性为true
- 当Activity中该属性用来标示：当前Activity是否可以被另一个Application的组件启动；true允许被启动；false不允许被启动
- 当MyService的exported属性为true时，将可以被其他应用调用，等其他组件类似

##### LocalBroadcastManager

- 只会将广播限定在当前应用程序中

##### 使用protectionLevel

###### 有以下四类级别：

- normal：低风险权限，只要申请了就可以使用(在AndroidManifest.xml中添加标签)，安装时不需要用户确认
- dangerous：高风险权限，安装时需要用户的确认才可以使用
- signature：只有当申请权限的应用程序的数字签名与生命此权限的应用程序的数字签名相同时(如果是申请系统权限，则需要与系统签名相同)，才能将权限授给它
- signatureOrSystem：签名相同，或者申请权限的应用为系统应用(在system.image中)才能拥有。如果需要对自己的应用程序进行访问控制，则可以通过AndroidManifest.xml中添加标签，将其属性中的protectionLevel设置为上述四类级别中的某种来实现

