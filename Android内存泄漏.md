# Android内存泄漏

## 内存泄漏与内存溢出

### 内存泄漏定义

- 内存泄漏是当程序不再使用到的内存时，释放内存失败而产生了无用的内存消耗。
- 内存泄漏并不是指物理上的内存消失，这里的内存泄漏是值由程序分配的内存但是由于程序逻辑错误而导致程序失去了对该内存的控制，使得内存浪费。

### 内存溢出定义

- 内存溢出：程序向系统申请的内存空间超出了系统能给的

#### 大量的内存泄漏会导致内存溢出（OOM）

## 内存

- Java是在JVM所虚拟出的内存环境中运行的，JVM的内存可分为三个区域：堆（heap）、栈（stack）和方法区（method）

### 栈（stack）

- 是简单的数据结构，LIFO（后进先出）
- 栈中只存放基本类型和对象的引用（并不是对象）

### 堆（heap）

- 堆内存用于存放由new创建的对象和数组
- 在堆中分配的内存，由java虚拟机自动垃圾回收器来管理
- JVM只有一个堆区（heap）被所有线程共享
- 堆中不存放基本类型和对象引用，只存放对象本身

### 方法区（method）

- 方法区又叫静态区，跟堆一样，被所有的线程共享
- 方法区包含所有的class和static变量

## 内存泄漏原因分析

- 在JAVA中JVM的栈记录了方法的调用，每个线程拥有一个栈。在线程的运行过程当中，执行到一个新的方法调用，就在栈中增加一个内存单元，即**帧(frame)**。在frame中，保存有该方法**调用的参数、局部变量和返回地址**
- 然而JAVA中的**局部变量**只能是**基本类型变量(int)**，或者**对象的引用**。所以在栈中只存放基本类型变量和对象的引用。引用的对象保存在堆中。
- 当某方法运行结束时，该方法对应的frame将会从栈中删除，frame中所有局部变量和参数所占有的空间也随之释放。线程回到原方法继续执行，当所有的栈都清空的时候，程序也就随之运行结束。
- 而对于堆内存，堆存放着**普通变量**。**在JAVA中堆内存不会随着方法的结束而清空**，所以在方法中定义了局部变量，在方法结束后变量依然存活在堆中。

### 综上所述

- 栈(stack)可以自行清除不用的内存空间。但是如果我们不停的创建新对象，堆(heap)的内存空间就会被消耗尽
- 所以JAVA引入了**垃圾回收**(garbage collection，简称GC)去处理堆内存的回收，但**如果对象一直被引用无法被回收，造成内存的浪费，无法再被使用**。所以**对象无法被GC回收**就是造成内存泄露的原因！

## 垃圾回收机制

- 垃圾回收（GC）可以自动清空堆中不再使用的对象
- 在Java中对象是通过引用使用的，如果再没有指向该对象，那么该对象就无从处理或调用该对象，这样的对象称为**不可到达**
- 垃圾回收用于释放不可到达的对象所占据的内存

### 垃圾回收实现思想

- 遍历栈中所有的对象的引用，再遍历一遍堆中的对象，因为栈中的对象的引用执行完毕就删除，所以可以通过栈中的对象的引用，查找到堆中没有被指向的对象，这些对象就是不可到达的对象，对其进行垃圾回收
- 如果持有对象的强引用，垃圾回收器是无法再内存中回收这个对象


## 内存泄漏类别

### 单例导致内存泄漏

- 因为单例的静态特性使得它的生命周期同应用的生命周期一样长，如果一个对象已经没有用处，但是单例还持有它的引用，那么整个应用程序的生命周期它都不能正常被回收，导致内存泄漏


```
public class AppSettings {

    private static AppSettings sInstance;
    private Context mContext;

    private AppSettings(Context context) {
        this.mContext = context;
    }

    public static AppSettings getInstance(Context context) {
        if (sInstance == null) {
            sInstance = new AppSettings(context);
        }
        return sInstance;
    }
}

```

- 在调用getInstance(Context context)方法的时候传入的context参数是Activity、Service等上下文，就会导致内存泄露。
- 当我们启动一个Activity，并调用getInstance(Context context)方法去获取AppSettings的单例，传入Activity.this作为context，这样AppSettings类的单例sInstance就持有了Activity的引用，当我们退出Activity时，该Activity就没有用了，但是因为sIntance作为静态单例（在应用程序的整个生命周期中存在）会继续持有这个Activity的引用，导致这个Activity对象无法被回收释放，这就造成了内存泄露。
- 优化：

```
private AppSettings(Context context) {
    this.mContext = context.getApplicationContext();
}
```
- 全局的上下文Application Context就是应用程序的上下文，和单例的生命周期一样长，这样就避免了内存泄漏。
- 单例模式对应应用程序的生命周期，所以我们在构造单例的时候尽量避免使用Activity的上下文，而是使用Application的上下文。

### 静态变量导致内存泄漏

- 静态变量存储在方法区，它的生命周期从类加载开始，到整个进程结束。一旦静态变量初始化后，它所持有的引用只有等到进程结束才会释放。


```
//在Activity中为了避免重复的创建info，将sInfo作为静态变量
public class MainActivity extends AppCompatActivity {

    private static Info sInfo;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        if (sInfo != null) {
            sInfo = new Info(this);
        }
    }
}

class Info {
    public Info(Activity activity) {
    }
}

```
- Info作为Activity的静态成员，并且持有Activity的引用，但是sInfo作为静态变量，生命周期肯定比Activity长。所以当Activity退出后，sInfo仍然引用了Activity，Activity不能被回收，这就导致了内存泄露。
- 静态持有很多时候都有可能因为其使用的生命周期不一致而导致内存泄露，所以我们在新建静态持有的变量的时候需要多考虑一下各个成员之间的引用关系，并且尽量少地使用静态持有的变量，以避免发生内存泄露
- 在适当的时候讲静态量重置为null，使其不再持有引用，这样也可以避免内存泄露。

### 非静态内部类导致内存泄漏

- 非静态内部类（包括匿名内部类）默认就会持有外部类的引用，当非静态内部类对象的生命周期比外部类对象的生命周期长时，就会导致内存泄露。
- 非静态内部类导致的内存泄露在Android开发中有一种典型的场景就是使用Handler，很多开发者在使用Handler是这样写的：

```
public class MainActivity extends AppCompatActivity {
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        start();
    }

    private void start() {
        Message msg = Message.obtain();
        msg.what = 1;
        mHandler.sendMessage(msg);
    }

    private Handler mHandler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            if (msg.what == 1) {
                // 做相应逻辑
            }
        }
    };
}

```
- mHandler虽未作为静态变量持有Activity医用，但mHandler作为成员变量保存在发送的消息msg中，即msg持有mHandler的引用，而mHandler是Activity的非静态内部类实例，即mHandler持有Activity的引用
- msg被发送后先放到消息队列MessageQueue中，然后等待Looper处理消息，当Activity退出后，msg可能仍然存在于消息队列MessageQueue中未处理或正在处理，这就会导致Activity无法被回收，以致发生Activity的内存泄漏

- 如果要使用内部类，又要避免内存泄漏，一般会采用**静态内部类+弱引用**的方式


```
private static class MyHandler extends Handler {

        private WeakReference<MainActivity> activityWeakReference;

        public MyHandler(MainActivity activity) {
            activityWeakReference = new WeakReference<>(activity);
        }

        @Override
        public void handleMessage(Message msg) {
            MainActivity activity = activityWeakReference.get();
            if (activity != null) {
                if (msg.what == 1) {
                    // 做相应逻辑
                }
            }
        }

```
- Handler通过弱引用的方式持有Activity，当GC执行垃圾回收时，遇到Activity就会回收并释放所占据的内存单元。这样就不会发生内存泄露了。
- 上面的做法确实避免了Activity导致的内存泄露，发送的msg不再已经没有持有Activity的引用了，但是msg还是有可能存在消息队列MessageQueue中，所以更好的是在Activity销毁时就将mHandler的回调和发送的消息给移除掉。
```
@Override
protected void onDestroy() {
    super.onDestroy();
    mHandler.removeCallbacksAndMessages(null);
}
```

### 非静态内部类造成内存泄漏还可能是使用Thread或者AsyncTask

#### Activity中直接new一个子线程Thread


```
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        new Thread(new Runnable() {
            @Override
            public void run() {
                // 模拟相应耗时逻辑
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }
}

```

#### Activity中直接新建AsyncTask异步任务


```
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        new AsyncTask<Void, Void, Void>() {
            @Override
            protected Void doInBackground(Void... params) {
                // 模拟相应耗时逻辑
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                return null;
            }
        }.execute();
    }
}

```

- 这种两种方式新建的子线程Thread和AsyncTask都是匿名内部类对象，默认就隐式的持有外部Activity的引用，导致Activity内存泄露
- 解决方式：使用静态内部类+弱应用的方式

### 未取消注册或回调导致内存泄漏

- 比如在Activity中注册广播，如果在Activity销毁后不取消注册，这个广播就一直存在系统中，同上面所说的非静态内部类一样持有Activity引用，导致内存泄漏
- 所以注册广播后在Activity销毁后一定要取消注册

```
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        this.registerReceiver(mReceiver, new IntentFilter());
    }

    private BroadcastReceiver mReceiver = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            // 接收到广播需要做的逻辑
        }
    };

    @Override
    protected void onDestroy() {
        super.onDestroy();
        this.unregisterReceiver(mReceiver);
    }
}
```

### Time和TimerTask导致内存泄漏

- Timer和TimerTask在Android中通常会被用来做一些计时或循环任务，比如实现无限轮播的ViewPager

```
public class MainActivity extends AppCompatActivity {

    private ViewPager mViewPager;
    private PagerAdapter mAdapter;
    private Timer mTimer;
    private TimerTask mTimerTask;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        init();
        mTimer.schedule(mTimerTask, 3000, 3000);
    }

    private void init() {
        mViewPager = (ViewPager) findViewById(R.id.view_pager);
        mAdapter = new ViewPagerAdapter();
        mViewPager.setAdapter(mAdapter);

        mTimer = new Timer();
        mTimerTask = new TimerTask() {
            @Override
            public void run() {
                MainActivity.this.runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        loopViewpager();
                    }
                });
            }
        };
    }

    private void loopViewpager() {
        if (mAdapter.getCount() > 0) {
            int curPos = mViewPager.getCurrentItem();
            curPos = (++curPos) % mAdapter.getCount();
            mViewPager.setCurrentItem(curPos);
        }
    }

    private void stopLoopViewPager() {
        if (mTimer != null) {
            mTimer.cancel();
            mTimer.purge();
            mTimer = null;
        }
        if (mTimerTask != null) {
            mTimerTask.cancel();
            mTimerTask = null;
        }
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        stopLoopViewPager();
    }
}

```

- 当我们Activity销毁的时，有可能Timer还在继续等待执行TimerTask，它持有Activity的引用不能被回收，因此当我们Activity销毁的时候要立即cancel掉Timer和TimerTask，以避免发生内存泄漏。

### 集合中的对象未清理造成内存泄漏

- 如果一个对象放入到**ArrayList、HashMap**等集合中，这个集合就会持有该对象的引用。当我们不再需要这个对象时，也并没有将它从集合中移除，这样只要集合还在使用（而此对象已经无用了），这个对象就造成了内存泄露
- 如果集合被静态引用的话，集合里面那些没有用的对象更会造成内存泄露了
- 所以在使用集合时要及时将不用的对象从集合remove，或者clear集合，以避免内存泄漏。

### 资源未关闭或释放 导致内存泄漏

- 在使用IO、File流或者Sqlite、Cursor等资源时要及时关闭。这些资源在进行读写操作时通常都使用了缓冲，如果及时不关闭，这些缓冲对象就会一直被占用而得不到释放，以致发生内存泄露。
- 因此我们在不需要使用它们的时候就及时关闭，以便缓冲能及时得到释放，从而避免内存泄露。

### 属性动画造成内存泄漏

- 动画同样是一个耗时任务，比如在Activity中启动了属性动画（ObjectAnimator），但是在销毁的时候，没有调用cancle方法，虽然我们看不到动画了，但是这个动画依然会不断地播放下去，动画引用所在的控件，所在的控件引用Activity，这就造成Activity无法正常释放。
- 因此同样要在Activity销毁的时候cancel掉属性动画，避免发生内存泄漏。
```
@Override
protected void onDestroy() {
    super.onDestroy();
    mAnimator.cancel();
}
```

### WebView造成内存泄漏

- 因为WebView在加载网页后会长期占用内存而不能被释放，因此我们在Activity销毁后要调用它的destory()方法来销毁它以释放内存。
- Webview下面的Callback持有Activity引用，造成Webview内存无法释放，即使是调用了Webview.destory()等方法都无法解决问题（Android5.1之后）
- 所以在销毁WebView之前需要先将WebView从父容器中移除，然后在销毁WebView

```
@Override
protected void onDestroy() {
    super.onDestroy();
    // 先从父控件中移除WebView
    mWebViewContainer.removeView(mWebView);
    mWebView.stopLoading();
    mWebView.getSettings().setJavaScriptEnabled(false);
    mWebView.clearHistory();
    mWebView.removeAllViews();
    mWebView.destroy();
}

```
