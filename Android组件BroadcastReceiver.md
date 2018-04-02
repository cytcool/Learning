# BroadcastReceiver

- Broadcast（广播）：是一种广泛运用的再应用程序之间传输信息的机制
- BroadcastReceiver（广播接收者）：是对发送出来的广播进行过滤接收并响应的一类组件，它就是用来接收来自系统和应用中的广播

## 用途

- 当开机完成后系统会产生一条广播
- 当网络状态改变时系统会产生一条广播
- 当电池电量改变时，系统会产生一条广播
- 等等

### 使用方法

#### 发送

- 把信息装入一个Intent对象（如Action，Category）
- 通过调用相应的方法将Intent对象以广播方式发送出去
- sendBroadcast() 普通广播
- sendOrderBroadcast() 有序广播
- sendStickyBroadcast() 异步广播

#### 接收

- 当Intent发送以后，所有已经注册的BroadcastReceiver会检查注册时的IntentFilter是否与发送的Intent相匹配，若匹配则就会调用BroadcastReceiver的onReceiver()方法。所以当我们定义一个BroadcastReceiver的时候，都需要实现onReceive()方法
- BroadcastReceiver需要注册（静态注册、代码注册）

### 注意

- BroadcastReceiver生命周期只有十秒左右
- 在BroadcastReceiver里不能做一些比较耗时的操作
- 应该通过发送Intent给Service，由Service来完成
- 不能使用子线程

## 广播的种类

- 普通广播：所有监听该广播的广播接收者都可以监听到该广播
- 有序广播：按照接收者的优先级顺序接收广播，优先级别在intent-filter中的priority中声明，-1000到1000之间，值越大，优先级越高。可以终止广播意图的继续传播。接收者可以篡改内容
- 异步广播：不能将处理结果传给下一个接收者，无法终止广播 
- 本地广播：不能被其他程序广播接收到，解决了广播的安全性问题

### 普通广播特点

- 同级别接收先后是随机的（无序）
- 级别低的后收到广播
- 接收器==不能截断广播==的继续传播也不能处理广播
- 同级别动态注册高于静态注册

### 有序广播特点：

- 同级别接收顺序是随机的
- 能截断广播的继续传播，高级别的广播接收器收到该广播后，可以决定把该广播是否截断
- 接收器能截断广播的继续传播，也能处理广播
- 同级别动态注册高于静态注册 
- **截断广播** ==abortBroadcast()==

#### 广播动态注册
- 在onCreate()方法中进行注册

```
   protected void onCreate(Bundle savedInstanceState){
   super.onCreate(savedInstanceState);
   setContentView(R.layout.activity_main);
   IntentFilter intentfilter = new IntentFilter("BC_One");
   BC2 bc2 = new BC2();
   registerReceiver(bc2,intentfilter);
}
```
- 动态注册使用完后需要进行手动销毁

### 本地广播

- 本地广播主要是使用了一个==LocalBroadcastManager==来对广播进行管理，并提供了发送广播和注册广播接收器的方法
- 用法：

```
public class MainActivity extends AppCompatActivity{
    private IntentFilter intentFilter;
    private LocalReceiver localReceiver;
    private LocalBroadcastManager localBroadcastManager;
    
    @Override
    protected void onCreate(Bundle savedInstanceState){
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main);
        localBroadcastManager = LocalBroadcastManager.getInstance(this);//获取实例
        Button button = (Button) findViewById(R.id.button);
        button.setOnClickListener(new View.OnClickListener(){
            @Override
            public void onClick(View v){
                Intent intent = new Intent("com.example.broadcasttest.LOCAL_BROADCAST");
                localBroadcastManager.sendBroadcast(intent);//发送广播
                
            }
        });
        intentFilter = new IntentFilter();
        intentFilter.addAction("com.example.broadcasttest.LOCAL_BROADCAST");
        localReceiver = new LocalReceiver();
        localBroadcastManager.registerReceiver(localReceiver,intentFilter);//注册本地广播监听器
    }
}
```
- 本地广播是无法通过静态注册的方式来接收的，因为静态注册主要是为了让程序在未启动的情况下也能收到广播，而发送本地广播时，程序肯定已经启动了
