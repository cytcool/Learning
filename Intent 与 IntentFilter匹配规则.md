# Intent与IntentFilter匹配规则

## Intent

- Intent是一个消息传递对象，可以使用它启动其他应用组件完成特定的任务

- 我们可以通过Intent来启动以下三个组件：Activity、Service、BroadcaseReceiver

### Activity

```
public void startActivity(Intent intent)
```

### Service


```
public ComponentName startService(Intent service)
public boolean bindService(Intent service, ServiceConnection conn, int flags)
```

### BroadcastReceiver

```
public void sendBroadcast(Intent intent)
public void sendOrderedBroadcast(Intent intent, String receiverPermission)
sendStickyBroadcast(Intent intent)
```

## Intent携带的信息


```
private String mAction;
private Uri mData;
private String mType;
private String mPackage;
private ComponentName mComponent;
private int mFlags;
private ArraySet<String> mCategories;
private Bundle mExtras;
private Rect mSourceBounds;
private Intent mSelector;
private ClipData mClipData;
private int mContentUserHint = UserHandle.USER_CURRENT;
```

#### 组件名称mComponent

- 可以使用setComponent()、setClass()、setClassName()或Intent构造函数设置组件名称
- 如果没有名称就是隐式的Intent

#### 要进行的操作mAction

- 可以使用系统定义好的，也可以自定义
- 可以使用setAction()或Intent构造函数为Intent指定操作

#### 数据mData

- 待操作数据或者数据的类型等信息
- 要仅设置数据URI，请调用setData()
- 要仅设置MIME类型，请调用setType()
- 如果同时设置以上两点，就使用setDataAndType()同时显式设置二者

#### 类别mCategories

- 表示Intent属于哪个类别
- 一个Intent可以属于多个类别，如果不生命，就属于默认的类别default
- 可以使用addCategory()指定类别

#### 附加数据mExtras

- Intent可以携带完成请求操作所需数据，格式为键值对
- 可以使用各种putExtra()方式添加数据
- 也可以创建一个包含所有数据的Bundle对象，然后使用putExtras()将Bundle插入Intent中

#### 标志位mFlags

- 标志位可以指示Android系统如何启动Activity以及启动之后如何处理
- 可以使用addFlags()方法添加标志位

### 注：

- 1.启动 Service时应该始终指定组件名称。否则无法确定哪项服务会响应Intent，且用户无法看到哪项服务已启动。
- 2.若要同时设置 URI 和 MIME 类型，请勿调用 setData() 和 setType()，因为它们会互相抵消彼此的值。 
- 3.Intent 类将为标准化的数据类型指定多个 EXTRA_* 常量。例如，使用 ACTION_SEND 创建用于发送电子邮件的 Intent 时，可以使用 EXTRA_EMAIL 键指定“目标”收件人，并使用 EXTRA_SUBJECT 键指定“主题”。

## Intent的类型

- 显式Intent
- 隐式Intent

### 隐式Intent

- 隐式Intent不直接指明要启动的组件，而是通过指定要进行的操作，让系统为我们找出匹配的组件

```
Uri uri = Uri.parse("smsto:18789999999");
Intent intent = new Intent();
intent.setAction(Intent.ACTION_SENDTO);
intent.setData(uri);
intent.putExtra("sms_body", "Hello");
startActivity(intent);
```
- 上述代码构建了一个Intent，然后为它设置了action、data和extra数据，然后调用了startActivity()
- 接着系统将检查已安装的所有应用，确定哪些应用能够处理这种Intent

#### 如果含ACTION_SENDTO操作并携带短信数据的Intent

- 如果只有一个应用能够处理，则该应用将立即打开并为其提供Intent
- 如果多个Activity接收Intent，则系统将显示一个对话框，使用户能够选取要使用的应用


#### 在检查每个Activity能否处理Intent的过程中，需要访问Intent过滤器(IntenFilter)

## Intent过滤器IntentFilter

- 我们可以在AndroidManifest.xml中给Activity设置一个IntentFilter属性
```
<activity
    android:name=".activity.launchmode.SingleTaskActivity"
    android:alwaysRetainTaskState="true"
    android:label="singleTask"
    android:launchMode="singleTask"
    android:taskAffinity="top.shixinzhang.task2">
    <intent-filter>
        <action android:name="top.shixinzhang.action.test"/>
        <category android:name="top.shixinzhang.category.test"/>
        <data android:mimeType="text/plain"/>
    </intent-filter>
</activity>
```
- IntentFilter中可以设置action、category和data三种过滤信息，每一种信息都可以有多个
- 一个Activity也可以有多个IntentFilter，相当于多了几个过滤器，被筛选到的可能就更大了

### IntentFIlter的匹配规则

#### 1.action的匹配规则

- action可以理解为一个组件具备功能、可以进行什么操作。系统为我们提供了很多内置的action，当然也可以自定义
- 一个Intent-filter可以有多个action，就好比一个人有多种才能
```
<intent-filter>
    <action android:name="android.intent.action.EDIT" />
    <action android:name="android.intent.action.VIEW" />
    ...
</intent-filter>
```
- Intent中的action至少有一个与过滤器的匹配，才能调用这个过滤器所在的组件，否则无法命中
- 注意：区分大小写

#### 2.category的匹配规则

- category即分类，和action一样，一个过滤器可以包含多个分类
```
<intent-filter>
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    ...
</intent-filter>
```
- 和action匹配规则（有一个即可）不同的是，category匹配时，要求你的Intent的category必须和过滤器中声明的完全匹配
- 以上述 intentFilter 为例，startActivity(intent)中的 intent 的分类不能**是android.intent.category.DEFAULT** 和 **android.intent.category.BROWSABLE** 以外的。
- Android 会自动将 **android.intent.category.DEFAULT** 类别传递给**startActivity()**和**startActivityForResult() **的所有隐式Intent。 
- 因此即使startActivity(intent)中不传任何分类，也可以命中上述过滤器。
- 注意：自定义分类时不要忘记在 AndroidManifest.xml 中添加 android.intent.category.DEFAULT，原因就是上面提到的，系统会为 startActivity() 中添加这个分类。

#### 3.data的匹配规则

- data表示该组件可以支持的数据格式与类型
- 同样，一个过滤器也可以有多个data
```
<intent-filter>
    <data android:mimeType="video/mpeg" android:scheme="http" ... />
    <data android:mimeType="audio/mpeg" android:scheme="http" ... />
    ...
</intent-filter>
```
- data由两个部分组成：**mimeType**和**scheme**
- mimeType指的是支持的数据类型与格式，常见有ext/plain、image/jpeg、video/* 、audio/*
- scheme就是常见的URI格式：
```
<scheme>://<host>:<port>/<path>
```
- 在Intent-Filter中，声明scheme必须从前往后，逐步缩小范围
- 你可以只声明一个协议，这表示该协议下的所有数据你都可以处理；同样也可以只声明主机地址，这表示使用该协议，访问该主机下的所有数据你都可以处理。。

##### scheme的具体部分介绍

- scheme协议类型：最重要，协议类型决定了如何访问数据，比如是本地还是网络
- host主机：第二重要，主机地址决定了具体ip
- port端口：第三重要，一个主机可能有多个网卡端口，有了端口才能访问到具体
- path具体路径：最后一级，表示要访问的文件夹路径

##### scheme和mimeType组成一个data。而data的匹配规则就是：intent中的data至少可以匹配过滤器中的一个

- 如果Intent-Filter只声明了**scheme**，那Intent中必须只包含scheme并且至少和Intent-filter中的一个scheme匹配才可以
- 如果Intent-Filter只声明了**mimeType**，那Intent中除了type要和Intent-Filter一致，还需要额外包含content或者file的scheme才行，因为Intent-Filter默认包含这两个scheme
- 如果 intent-filter 同时声明了多个 scheme 和 mimeType，那你的 intent 至少要完全匹配其中的一组

##### 注意： intent-filter 默认的 content 或者 file 的 scheme ，它表示默认组件能够从文件中或内容提供程序获得本地数据。

### 总结过滤郭泽

#### 1.擅长什么开发，UI、网络、音视频？ （对应 action）

- 至少具备要求中的一条才可以

#### 2.是哪类程序员，求知欲强、自我驱动？（对应 category ） 

- 必须和要求完全一致才可以

#### 3.使用什么工具开发，AS、Eclipse、记事本？(对应 data) 

- 至少具备要求中的一条才可以

### 注意：

- 如果当前设备中没有能够匹配你发送到 startActivity() 的隐式 Intent，则调用将会失败，且应用会崩溃。
- 因此我们需要对 Intent 对象调用**resolveActivity**：
如果结果为非空，则至少有一个应用能够处理该 Intent，且可以安全调用 startActivity()
如果结果为空，则不应使用该 Intent

```
Intent sendIntent = new Intent();
sendIntent.setAction(Intent.ACTION_SEND);
sendIntent.putExtra(Intent.EXTRA_TEXT, textMessage);
sendIntent.setType("text/plain");

//验证当前 Intent 是否可以被处理
if (sendIntent.resolveActivity(getPackageManager()) != null) {
    startActivity(sendIntent);
}
```
