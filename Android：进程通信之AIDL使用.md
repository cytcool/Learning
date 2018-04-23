# Android：进程通信之AIDL使用

## AIDL是什么

- AIDL（Android接口定义语言）是Android提供的一种进程间通信（IPC）机制
- 我们可以利用它定义客户端与服务端使用进程间通信（IPC）进行相互通信时都认可的编程接口
- 在Android上，一个进程通常无法访问另一个进程的内存，尽管如此，进程需要将其对象分解成操作系统能够识别的原语，并将对象编组成跨越边界的对象

## AIDL文件支持的数据类型

- 基本数据类型（int、Long、char、boolean、double等）
- String和CharSequence
- List：只支持ArrayList，里面每个元素都必须能够被AIDL支持
- Map：只支持HashMap，里面的每个元素都必须被AIDL支持，包括Key和Value
- Parcelable：所有实现了Parcelable接口的对象
- AIDL：所有的AIDL接口本身也可以在AIDL文件中使用

## AIDL如何编写

### 1、创建AIDL

- 1、创建要操作的实体类，实现Parcelable接口，以便序列化/反序列化
```
import android.os.Parcel;
import android.os.Parcelable;

public class Person implements Parcelable {
    private String mName;

    public Person(String name) {
        mName = name;
    }

    protected Person(Parcel in) {
        mName = in.readString();
    }

    public static final Creator<Person> CREATOR = new Creator<Person>() {
        @Override
        public Person createFromParcel(Parcel in) {
            return new Person(in);
        }

        @Override
        public Person[] newArray(int size) {
            return new Person[size];
        }
    };

    @Override
    public int describeContents() {
        return 0;
    }

    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeString(mName);
    }

    @Override
    public String toString() {
        return "Person{" +
                "mName='" + mName + '\'' +
                '}';
    }
}
```
- 实现 Parcelable接口是为了后序跨进程通信时使用。
- 自定义的Parcelable对象和AIDL对象必须要显示import进来，不管它们是否和当前的AIDL文件位于同一个包内
- 2、新建AIDL文件夹，在其中创建接口AIDL文件以及实体类的映射AIDL文件
- 在**main**文件夹下新建AIDL文件夹，使用的包名要和**java**文件夹的包名一致
- 先创建实体类的映射aidl文件，Person.aidl;
```
// Person.aidl
package net.sxkeji.shixinandroiddemo2.bean;

//还要和声明的实体类在一个包里
parcelable Person;
```
- 在其中声明映射的实体类名称与类型（注意：Person.aidl的包名要和实体类包名一致）
- 然后创建接口aidl文件，IMyAidl.aidl
```
// IMyAidl.aidl
package net.sxkeji.shixinandroiddemo2;

import net.sxkeji.shixinandroiddemo2.bean.Person;

interface IMyAidl {
    /**
     * 除了基本数据类型，其他类型的参数都需要标上方向类型：in(输入), out(输出), inout(输入输出)
     */
    void addPerson(in Person person);

    List<Person> getPersonList();
}
```
- AIDL中除了基本数据类型，其他类型的参数必须标上方向：in、out或者inout
- 3、Make Project，生成Binder的Java文件

### 2、服务端

- 创建Service，在其中创建上面生成的Binder对象实例，实现接口定义的方法
- 在onBind()中返回
```
public class MyAidlService extends Service {
    private final String TAG = this.getClass().getSimpleName();

    private ArrayList<Person> mPersons;

    /**
     * 创建生成的本地 Binder 对象，实现 AIDL 制定的方法
     */
    private IBinder mIBinder = new IMyAidl.Stub() {

        @Override
        public void addPerson(Person person) throws RemoteException {
            mPersons.add(person);
        }

        @Override
        public List<Person> getPersonList() throws RemoteException {
            return mPersons;
        }
    };

    /**
     * 客户端与服务端绑定时的回调，返回 mIBinder 后客户端就可以通过它远程调用服务端的方法，即实现了通讯
     * @param intent
     * @return
     */
    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        mPersons = new ArrayList<>();
        LogUtils.d(TAG, "MyAidlService onBind");
        return mIBinder;
    }
}
```
- 在Manifest文件中声明：
```
<service
    android:name="net.sxkeji.shixinandroiddemo2.service.MyAidlService"
    android:enabled="true"
    android:exported="true"
    android:process=":aidl"/>
```
- 服务端实现了接口，在onBind()中返回这个Binder，客户端拿到就可以操作数据了

### 3、客户端

- 实现==ServiceConnection==接口，在其中拿到AIDL类
```
private IMyAidl mAidl;

private ServiceConnection mConnection = new ServiceConnection() {
    @Override
    public void onServiceConnected(ComponentName name, IBinder service) {
        //连接后拿到 Binder，转换成 AIDL，在不同进程会返回个代理
        mAidl = IMyAidl.Stub.asInterface(service);
    }

    @Override
    public void onServiceDisconnected(ComponentName name) {
        mAidl = null;
    }
};
```
在 Activity中创建一个服务连接对象，在其中调用 **IMyAidl.Stub.asInterface()** 方法将 Binder 转为 AIDL 类。


- bindService()绑定服务
```
Intent intent1 = new Intent(getApplicationContext(), MyAidlService.class);
bindService(intent1, mConnection, BIND_AUTO_CREATE);
```
要执行 IPC，必须使用 bindService() 将应用绑定到服务上。
- 调用AIDL类中定义好的操作请求
```
@OnClick(R.id.btn_add_person)
public void addPerson() {
    Random random = new Random();
    Person person = new Person("shixin" + random.nextInt(10));

    try {
        mAidl.addPerson(person);
        List<Person> personList = mAidl.getPersonList();
        mTvResult.setText(personList.toString());
    } catch (RemoteException e) {
        e.printStackTrace();
    }
}
```




