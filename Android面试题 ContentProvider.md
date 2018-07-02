## ContentProvider常见知识点

### 1、ContentProvider是如何实现数据共享的

- 在Android中如果想将自己应用的数据(一般多为数据库中的数据)提供给第三方应用，只能通过ContentProvider来实现了
- ContentProvider是应用程序之间共享数据的接口。
- 使用的时候首先自定义一个类继承ContentProvider，然后覆写query，insert，update，delete等方法。因为其是四大组件之一，因此必须在AndroidManifest文件中进行注册
- 把自己的数据通过uri的形式共享出去
- Android系统下不同程序数据默认是不能共享访问，需要去实现一个类去继承ContentProvider
```
public class PersonContentProvider extends ContentProvider{    
    public boolean onCreate(){
    } 
    query(Uri, String[], String, String[], String)；
    insert(Uri, ContentValues)；
    update(Uri, ContentValues, String, String[]) ；
    delete(Uri, String,  String[])；
}    
```

- 第三方可以通过ContentResolver来访问该Provider

### Android的数据存储方式

- 1、File存储
- 2、SharedPreference存储
- 3、ContentProvider存储
- 4、SQLiteDataBase存储
- 5、网络存储

#### 为什么要用ContentProvider？它和sql的实现上有什么差别？

- ContentProvider屏蔽了数据存储的细节，内部实现对用户完全透明，用户只需要关心操作数据的uri就可以了。
- ContentProvider可以实现不同app之间共享，而Sql也有增删改查的方法
- 但是sql只能查询本应用下的数据库
- 而ContentProvider还可以去增删改查本地文件，xml文件的读取等

#### 说说ContentProvider、ContentResolver、ContentObserver之间的关系

- 1、ContentProvider：内容提供者，用于对外提供数据
- 2、ContentResolver.notifyChange(uri)：发出消息
- 3、ContentResolver：内容解析者，用于获取内容提供者提供的数据
- 4、ContentObserver：内容监听器，可以监听数据的改变状态
- 5、ContentResolver.registerContentObserver()：监听消息

#### Uri介绍

- 1、每一个ContentProvider都拥有一个公共的URI，这个URI用于表示这个ContentProvider所提供的数据
- 2、Android所提供的ContentProvider都存放在android.provider包中，将其分为A,B,C,D四个部分：

A：标准前缀，用来说明一个ContentProvider控制这些数据，无法改变的;“content://”

B：URI的标识，用于唯一标识这个ContentProvider，外部调用者可以根据这个标识来找到它，它定义了是哪个ContentProvider提供这些数据。对于第三方应用程序，为了保证URI标识的唯一性，它必须是一个完整的、小写的雷鸣。这个标识在元素的authorities属性中说明：一般是定义该ContentProvider的包.类的名称

C：路径(path)，通俗的讲就是你要操作的数据库中表的名字，或者你也可以自己定义，记得在使用的时候保持一致就可以了"content://com.bing.provider.myprovider/tablename"

D：如果URI中包含标识需要获取的记录的ID，则就返回该ID对应的数据，如果没有ID，就标识返回全部； "content://com.bing.provider.myprovider/tablename/#" #表示数据id。

#### 如何访问asserts资源目录下的资源库

- 把数据库db赋值到/data/data/packagename/databases/目录下，然后直接就能访问了

### 2、Intent详解

#### Android Intent的使用

- Intent是一种运行时绑定的消息机制，而三大组件Activity、Service和BroadcastReceiver都是被消息激活的，这种消息就是Intent
- 一个Intent对象包括六个属性：组件名(Component Name)、动作(Action)、数据(Data)、分类(Category)、额外消息(Extra)和标志(Flag)
- 在一个Android应用中，主要由一些组件组成，(Activity、Service、ContentProvider等)在这些组件之间的通信中，由Intent协助完成
- Intent负责对应用中一次操作的动作、动作涉及数据、附加数据进行描述，Android则根据此Intent的描述，负责找到对应的组件，将Intent传递给调用的组件，完成组件的调用，Intent在这里起着实现调用者与被调用者之间的解耦作用
- Intent在传递过程中，要找到目标消费者(另一个Activity，IntentReceiver或Service)，也就是Intent的响应者，有两种方法进行匹配

####  1、显示意图： 伪代码
```
public TestB extents Activity  
{  
 .........  
};  
 public class Test extends Activity  
{  
     ......  
     public void switchActivity()  
     {  
            Intent i = new Intent(Test.this, TestB.class);  
            this.startActivity(i);  
     }  
} 
```

#### 2、隐式意图

- 隐式匹配，首先要匹配Intent的几项值：Action，Category，Data/Type，Component(如果填写了Component就是上例中的Test.class，这就形成了显示匹配了)

##### 匹配规则为最大匹配规则

- 1、如果你填写了Action，如果一个程序的Manifest.xml中的某个一个Activity的IntentFilter段中定义了包含了相同的Action，那么这个Intent就与这个目标Action匹配，如果这个Filter段中没有定义Type，Category，那么这个Activity就匹配了。但是如果手机中有两个以上的程序匹配，那么久会弹出一个对话框来提示说明
- Action的值在Android中有很多预定义，如果你想直接转到你自己定义的Intent接收者，你可以在接收者的IntentFilter中加入一个自定义的Action值(同时要设定Category值为"android.intent.category.DEFAULT")，在你的Intent中设定该值为Intent的Action，就直接能跳转到你自己的Intent接收者中，因为这个Action在系统中是唯一的
- 2、data/type，你可以用Uri来作为data，比如Uri uri = Uri.parse("http://www.google.com")；
```
Intent i = new Intent(Intent.ACTION_VIEW,uri)
//手机的Intent分发过程中，会根据http://www.google.com的scheme判断出数据类型type
//手机的Brower则能匹配它，在Bro味儿的Manifest.xml中的IntentFilter首先由ACTION_VIEW Action,也能处理http:的type
```

- 3、分类Category，一般不要再Intent中设置它，如果你写Intent的接收者，就在Manifest.xml的Activity的IntentFilter中包含Android.category.DEFAULT，这样所有不设置Category(Intent.addCategory(String c);)的Intent都会与这个Category匹配
- 4、extra(附加信息)：是其他所有附加信息的集合，使用extras可以为组件提供扩展信息，比如：如果要执行“发送电子邮件”这个动作，可以将电子邮件的标题、正文等保存在extras里，传给电子邮件发送组件

##### IntentFilter(Intent过滤器)，为什么要引入

- 对于显示Intent，它的接收者已被指定，所以系统会自动把这个Intent发送给指定的组件。
- 但是对于隐式Intent，由于并没有指定其组件名属性，所以系统不知道该把它发给哪个组件名，于是系统就直接将其发送出去，算是所有的组件都有权接收，这就需要定义一个组件可以接收到哪些Intent，所以就引入了IntentFilter(Intent过滤器)

##### Intent传递数据时，可以传递哪些类型数据？

- Intent可以传递的数据类型非常的丰富，java的基本数据类型和String以及他们的数组形式都可以，除此之外还可以传递实现了Serializable和Parcelable接口的对象

##### Serializable和Parcelable的区别

- 1、在注重内存的时候，Parcelable类比Serializable性能高，所以推荐使用Parcelable类
- 2、Serializable在序列化的时候会产生大量的临时变量，从而引起频繁的GC
- 3、Parcelable不能使用在要将数据存储在磁盘上的情况。尽管Serializable效率低点，但在这种情况下，还是建议使用Serializable实现

##### Serializable和Parcelable的实现

- 1、Serializable的实现，只需要继承Serializable计科，这只是给对象打了个标记，系统会自动将其序列化
- 2、Parcelablel的实现，需要在类中添加一个静态成员变量CREATOR，这个变量需要继承Parcelable.Creator接口
