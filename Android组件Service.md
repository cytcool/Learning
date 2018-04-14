# Service

## 定义

- 后台运行，不可见，没有界面
- 优先级高于Activity

## 用途

- 播放音乐、记录地理信息位置的改变、监听某种动作...

## 注意

- 运行在主线程，不能用它来做耗时的请求或动作
- 可以在服务中开一个线程，在线程中做耗时动作

## 类型

### 1、本地服务

- 应用程序内部
- startService,stopService,stopSelf,stopSelfResult
- bindService,unbindService

### 2、远程服务

- Android系统内务的应用程序之间
- 定义IBinder接口

## 生命周期

### Unboundedservice

- startService()-> onCreate()-> onStartCommand()-> Service running -> onDestroy()

#### 特点

- 服务跟启动源没有任何联系
- 无法得到服务对象

### Boundedservice

- bindService()-> onBind()-> Clients are bound to service-> onUnbind()-> onDestroy()



#### 特点

- 通过IBinder接口实例，返回一个ServiceConnection对象给启动源
- 通过ServiceConnection对象的相关方法可以得到Service对象

## Service的两种启动方式

### 1.启动状态

- 1.调用startService()方法启动
- 2.启动状态下的Service将会在后台一直运行，即便主应用退出依旧在运行
- 直到自身通过调用stopSelf()结束工作，或者由另一个组件通过调用stopService()来停止
- 这种状态下的Service一般只负责执行任务，不会直接将结果返回调用方
- 比如后台下载数据或者处理文件
```
Intent intent = new Intent(this, HelloService.class);
startService(intent);
```
- startService() 的方式启动服务时，传递 intent 是组件与服务唯一的通信方式


### 2.绑定状态

- 1.调用bindService()启动
- 2.绑定状态下的服务可以和调用组件交互，比如发送请求、获取结果
- 3.这种情况下就可能涉及到IPC
- 4.一个服务可以绑定多个组件，有绑定的组件才会运行，绑定的组件全部取消绑定后就销毁（同生同死）

- 启动服务如果需要返回结果，有两种选择：再调用bindService()绑定服务或者为传递的Intent中添加一个广播，服务端给广播发送结果
- 绑定服务
```
private ServiceConnection mConnection = new ServiceConnection() {
    @Override
    public void onServiceConnected(ComponentName name, IBinder service) {
        mAidl = IMyAidl.Stub.asInterface(service);
    }

    @Override
    public void onServiceDisconnected(ComponentName name) {
        mAidl = null;
    }
};
Intent intent1 = new Intent(getApplicationContext(), MyAidlService.class);
bindService(intent1, mConnection, BIND_AUTO_CREATE);
```
- 添加广播
```
//1. 在启动服务的组件中构建广播的 PendingIntent，以 bundle 的形式添加到 intent 中，然后启动服务
private void starServiceWithBroadcast(){
    PendingIntent pendingIntent = PendingIntent.getBroadcast(this, 0, new Intent(this, RepeatReceiver.class), 0);
    Bundle bundle = new Bundle();
    bundle.putParcelable("receiver", pendingIntent);
    Intent intent = new Intent(getApplicationContext(), MyAidlService.class);
    intent.putExtras(bundle);
    startService(intent);
}
//2.在你的 Service 中拿出 intent 携带的 PendingIntent，在完成任务后将结果发送出去
public class DataRequestService extends Service {

   private final class ServiceHandler extends Handler {
      public ServiceHandler(Looper looper) {
         super(looper);
      }

      @Override
      public void handleMessage(Message msg) {      
         Bundle bundle = msg.getData();
         PendingIntent receiver = bundle.getParcelable("receiver");
         try {            
            Intent intent = new Intent();
            Bundle b = new Bundle();
            intent.putExtras(b);
            receiver.send(getApplicationContext(), status, intent);
         } catch (CanceledException e) {         
         e.printStackTrace();
         }         
      }
   }

   @Override
   public void onStart(Intent intent, int startId) {
      Bundle bundle = intent.getExtras();
      msg.setData(bundle);
      mServiceHandler.sendMessage(msg);
   }
```

### 两种状态下服务的生命周期

- **onCreate**只在创建时调用一次，一旦服务启动后，就不会再调用了
- **onStartCommand**必须返回整型数，它用于表示在服务停止时系统如何处理，有以下三个值：**START_NOT_STICKY** : 服务终止时不会重建，比较安全；**START_STICKY** : 服务终止时重建并调用 onStartCommand() ，但传递的 intent为空，适用于不需要参数的服务；**START_REDELIVER_INTENT**:和START_STICKY类似，但会将之前接收到的intent传递给重建服务的onStartCommand() 方法，适用于必须立即恢复的紧急任务
- **onBind**返回一个IBinder，客户端拿到后就可以和服务通信

### 停止服务

- 使用startService()方式启动的服务，除非系统必须回收内存资源，否则不会停止
- stopService()：Context的方法，外部组件调用，调用后系统会尽快销毁服务
- stopSelf()：Service的方法，效果和stopService() 一样
- stopSelf(int)：Service的方法，它的特别之处在于参数和启动时的id一致才会被终止，也就是说如果在终止前又收到新的调用，就不会停止

### 前台服务

- 仅当内存过低且必须回收系统资源以供具有用户焦点的 Activity使用时，Android系统才会强制停止服务。 如果将服务绑定到具有用户焦点的Activity，则它不太可能会终止；如果将服务声明为在前台运行，则它几乎永远不会终止。
- 为了降低 Service被回收的可能，有时候我们需要把服务声明为前台的，这样在内存不足时，系统也不会考虑将其终止，因为在系统看来它正在与用户进行交互。

```
//使用startForeground()
//源码
public final void startForeground(int id, Notification notification) {
    try {
        mActivityManager.setServiceForeground(
                new ComponentName(this, mClassName), mToken, id,
                notification, 0);
    } catch (RemoteException ex) {
    }
}
```

```
//实例
PendingIntent contentIntent = PendingIntent.getActivity(this, 0,
        new Intent(this, ServiceTestActivity.class), 0);

Notification notification = new Notification.Builder(this)
        .setSmallIcon(R.drawable.icon_ez)
        .setTicker(text)
        .setWhen(System.currentTimeMillis())
        .setContentTitle(getText(R.string.local_service_label))
        .setContentText(text)
        .setContentIntent(contentIntent)
        .build();

startForeground(NOTIFICATION, notification);
```

### Android5.0后需要显示启动Service

- 在 5.0 以后为了确保应用的安全性，系统强制要求使用显式Intent启动或绑定Service，否则运行时会报错
- 同时建议不要为服务声明Intent过滤器，因为它会导致启动哪个服务的不确定性。
- 除此外还可以为 Service 添加 android:exported 属性并将其设置为“false”，确保服务仅适用于你的应用。这可以有效阻止其他应用启动您的服务。
```
Intent intent = new Intent();
intent.setComponent(new ComponentName("top.shixinzhang.myapplication", "top.shixinzhang.myapplication.service.MyAidlService"));
bindService(intent, mConnection, BIND_AUTO_CREATE);

Intent intent1 = new Intent(getApplicationContext(), MyAidlService.class);
bindService(intent1, mConnection, BIND_AUTO_CREATE);
```

### Service中弹出Dialog

- Service 中可以弹Toast和Notification来提示用户，这也符合 Service的特点，默默无闻地后台奉献，没有界面，提示也是比较轻量级的
- 官方文档是不可以的弹出Dialog，毕竟在其他应用中弹出自己应用的对话框，有些不人性化，官方希望类似的场景采用Notification来解决
- 有些场景下可以弹出：
```
//修改 Dialog 的类型为系统提示

dialog.getWindow().setType((WindowManager.LayoutParams.TYPE_SYSTEM_ALERT))
```

```
//修改 Activity 的主题为 Dialog，然后启动一个 Activity
<style name="AppTheme.Dialg" parent="Theme.AppCompat.Light.Dialog">

</style>
```
