# IntentService

## 简介


```
public abstract class IntentService extends Service {...}
```
- IntentService是一个**抽象类**，继承了**Service**
- IntentService 使用工作线程逐一处理所有启动请求。如果你不需要在 Service 中执行并发任务，IntentService 是最好的选择

## IntentService源码分析


```
public abstract class IntentService extends Service {
    private volatile Looper mServiceLooper;
    private volatile ServiceHandler mServiceHandler;
    private String mName;
    private boolean mRedelivery;

    //内部创建的 Handler
    private final class ServiceHandler extends Handler {
        public ServiceHandler(Looper looper) {
            super(looper);
        }

        @Override
        public void handleMessage(Message msg) {
            //调用这个方法处理数据
            onHandleIntent((Intent)msg.obj);
            //处理完就自尽了
            stopSelf(msg.arg1);
        }
    }

    //子类需要重写的构造函数，参数是服务的名称
    public IntentService(String name) {
        super();
        mName = name;
    }

    //设置当前服务被意外关闭后是否重新
    //如果设置为 true，onStartCommand() 方法将返回 Service.START_REDELIVER_INTENT，这样当
    //当前进程在 onHandleIntent() 方法返回前销毁时，会重启进程，重新使用之前的 Intent 启动这个服务
    //(如果有多个 Intent，只会使用最后的一个)
    //如果设置为 false，onStartCommand() 方法返回 Service.START_NOT_STICKY，当进程销毁后也不重启服务
    public void setIntentRedelivery(boolean enabled) {
        mRedelivery = enabled;
    }

    @Override
    public void onCreate() {
        super.onCreate();
        //创建时启动一个 HandlerThread
        HandlerThread thread = new HandlerThread("IntentService[" + mName + "]");
        thread.start();

        //拿到 HandlerThread 中的 Looper，然后创建一个子线程中的 Handler
        mServiceLooper = thread.getLooper();
        mServiceHandler = new ServiceHandler(mServiceLooper);
    }

    @Override
    public void onStart(@Nullable Intent intent, int startId) {
        //将 intent 和 startId 以消息的形式发送到 Handler
        Message msg = mServiceHandler.obtainMessage();
        msg.arg1 = startId;
        msg.obj = intent;
        mServiceHandler.sendMessage(msg);
    }

    /**
     * You should not override this method for your IntentService. Instead,
     * override {@link #onHandleIntent}, which the system calls when the IntentService
     * receives a start request.
     * @see android.app.Service#onStartCommand
     */
    @Override
    public int onStartCommand(@Nullable Intent intent, int flags, int startId) {
        onStart(intent, startId);
        return mRedelivery ? START_REDELIVER_INTENT : START_NOT_STICKY;
    }

    @Override
    public void onDestroy() {
        mServiceLooper.quit();    //值得学习的，在销毁时退出 Looper
    }

    @Override
    @Nullable
    public IBinder onBind(Intent intent) {
        return null;
    }

    @WorkerThread
    protected abstract void onHandleIntent(@Nullable Intent intent);
}
```

### 从上述代码可以看到，Inte做了以下工作

- 创建了一个HandlerThread默认的工作线程
- 使用HandlerThread的Looper创建了一个Handler，这个Handler执行在子线程
- 在onStartCommand()中调用onStart(),然后再onStart()中奖intent和startId以消息的形式发送到Handler
- 在Handler中将消息队列中的Intent按顺序传递给onHandleIntent()方法
- 在处理完所有启动请求后自动停止服务，不需要我们调用stopSelf()

```
public void handleMessage(Message msg) {
    onHandleIntent((Intent)msg.obj);
    stopSelf(msg.arg1);
}
```

- 虽然在handleMessage()方法中只调用了一次onHandleIntent()方法然后就调用了stopSelf()方法
- 仔细看下可以发现，这个stopSelf()方法传递了一个 id，这个id是启动服务时IActivityManager 分配的 id，当我们调用stopSelf(id)方法结束服务时，IActivityManager 会对比当前 id 是否为最新启动该服务的id，如果是就关闭服务

```
public final void stopSelf(int startId) {
    if (mActivityManager == null) {
        return;
    }
    try {
        mActivityManager.stopServiceToken(
                new ComponentName(this, mClassName), mToken, startId);
    } catch (RemoteException ex) {
    }
}
```
- 所以只有当最后一次启动IntentService的任务执行完毕才会关闭这个服务
- 此外还要注意的是，IntentService 中除了 onHandleIntent方法其他都是运行在主线程的。

## 总结

- 在第一次启动 IntentService 后，IntentService仍然可以接受新的请求，接受到的新的请求被放入了工作队列中，等待被串行执行
- 使用 IntentService显著简化了启动服务的实现，如果您决定还重写其他回调方法（如onCreate()、onStartCommand()或onDestroy()），请确保调用超类实现，以便IntentService能够妥善处理工作线程的生命周期。
- 由于大多数启动服务都不必同时处理多个请求（实际上，这种多线程情况可能很危险），因此使用 IntentService类实现服务也许是最好的选择。

### 一句话总结IntentService

- 优先级比较高的、用于串行执行异步任务、会自尽的 Service