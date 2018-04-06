# HandlerThread

## handlerThread是什么

### 一、handlerThread产生背景

- 开启Thread子线程进行耗时操作
- 多次创建和销毁线程是很耗系统资源的

### 二、handlerThread是什么

- handler+thread+looper
- 是一个thread内部有looper

### 三、handlerThread的特点

- HandlerThread本质上是一个线程类，它继承了Thread
- HandlerThread有自己的**内部Looper**对象，可以进行**Looper循环**
- 通过获取HandlerThread的Looper对象传递给Handler对象，可以在handleMessage方法中执行**异步任务**
- 优点是不会有堵塞，减少了对性能的消耗，缺点是不同同时进行多任务的处理，需要等待进行处理，处理效率较低
- 与线程池注重并发不同，HandlerThread是一个串行队列，HandlerThread背后只有一个线程
- HandlerThread将Looper转到子线程中处理，可以分担MainLooper的工作量，降低了主线程的压力，使主界面更流畅
- HandlerThread拥有自己的消息队列，它不会干扰或阻塞UI线程


### 四、HandlerThread常规使用步骤

- 1、创建实例对象

```
//参数是标记当前线程的名字
HandlerThread handlerThread = new HandlerThread("downloadImage");
```
- 2、启动HandlerThread线程

```
handlerThread.start();
```
- 3、构建循环消息处理机制

```
handler = new Handler( myHandlerThread.getLooper() ){
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
            //这个方法是运行在 handler-thread 线程中的 ，可以执行耗时操作
            Log.d( "handler " , "消息： " + msg.what + "  线程： " + Thread.currentThread().getName()  ) ;
            }
    };
//在主线程给handler发送消息
handler.sendEmptyMessage( 1 ) ;
        new Thread(new Runnable() {
            @Override
            public void run() {
             //在子线程给handler发送数据
             handler.sendEmptyMessage( 2 ) ;
            }
        }).start() ;
}
```

- 4、退出循环，释放资源

```
 @Override
    protected void onDestroy() {
        super.onDestroy();

        //释放资源
        myHandlerThread.quit() ;
    }
```
## HandlerThread源码分析

```
public class HandlerThread extends Thread {
    int mPriority;
    int mTid = -1;
    Looper mLooper;

    public HandlerThread(String name) {
        super(name);
        mPriority = Process.THREAD_PRIORITY_DEFAULT;
    }

    //也可以指定线程的优先级，注意使用的是 android.os.Process 而不是 java.lang.Thread 的优先级！
    public HandlerThread(String name, int priority) {
        super(name);
        mPriority = priority;
    }

    // 子类需要重写的方法，在这里做一些执行前的初始化工作
    protected void onLooperPrepared() {
    }

    //获取当前线程的 Looper
    //如果线程不是正常运行的就返回 null
    //如果线程启动后，Looper 还没创建，就 wait() 等待 创建 Looper 后 notify
    public Looper getLooper() {
        if (!isAlive()) {
            return null;
        }

        synchronized (this) {
            while (isAlive() && mLooper == null) {    //循环等待
                try {
                    wait();
                } catch (InterruptedException e) {
                }
            }
        }
        return mLooper;
    }

    //调用 start() 后就会执行的 run()
    @Override
    public void run() {
        mTid = Process.myTid();
        Looper.prepare();            //帮我们创建了 Looepr
        synchronized (this) {
            mLooper = Looper.myLooper();
            notifyAll();    //Looper 已经创建，唤醒阻塞在获取 Looper 的线程
        }
        Process.setThreadPriority(mPriority);
        onLooperPrepared();    
        Looper.loop();        //开始循环
        mTid = -1;
    }

    public boolean quit() {
        Looper looper = getLooper();
        if (looper != null) {
            looper.quit();
            return true;
        }
        return false;
    }

    public boolean quitSafely() {
        Looper looper = getLooper();
        if (looper != null) {
            looper.quitSafely();
            return true;
        }
        return false;
    }

    public int getThreadId() {
        return mTid;
    }
}
```
