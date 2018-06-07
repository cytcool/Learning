## 如何计算一个Bitmap占用内存的大小，怎么保证加载Bitmap不产生内存溢出？

- Bitmap占用内存大小=宽度像素*(inTagrgetDensity/inDensity)*高度像素*(inTargetDensity/inDensity)*一个像素所占的内存
- 这里inDensity表示目标图片的，inTargetDensity表示目标屏幕的dpi，所以你可以发现inDensity和inTargetDensity会对Bitmap的宽高进行拉伸，进而改变Bitmap占用内存的大小

#### 在Bitmap里有两个获取内存占用大小的方法

- getByteCount():API 12加入，代表存储Bitmap的像素需要的最少内存
- getAllocationBytCount():API 19加入，代表在内存中为Bitmap分配的内存大小，代替了getBytCount()

#### 在不复用Bitmap时，getByteCount()和getAllocationByteCount()返回的结果是一样的。在通过复用Bitmap来解码图片时，那么getByteCount()表示新解码图片占用内存的大小，getAllocationByteCount()表示被复用Bitmap真实占用的内存大小

### 为了保证在加载Bitmap的时候不产生内存溢出，可以受用BitmapFactory进行图片压缩，主要有以下几个参数：

- BitmapFactory.Options.inPreferredConfig：将AGRB_8888改为RGB_565，改变编码方式，节约内存
- BitmapFactory.Options.inSampleSize：缩放比例，可以参考Luban那个库，根据图片宽高计算出适合的缩放比例
- BitmapFactory.Options.inPurgeable：让系统可以内存不足时回收内存

## Android如何在不压缩的情况下加载高清大图

- 使用BitmapRegionDecoder进行布局加载

## Android里的内存缓存和磁盘缓存是怎么实现的

- 内存缓存基于LruCache实现，磁盘缓存基于DiskLruCache实现，这两个类都基于Lru算法和LinkedHashMap来实现

#### LRU算法可以用一句话来描述：

- LRU是Least Recently Used的缩写，最近最久未使用算法，从它的名字可以看出，它的核心原则是如果一个数据在最近一段时间没有使用到，那么它在将来被访问到的可能性也很小，则这类数据项会被优先淘汰掉

#### LruCache的原理

- 利用LinkedHashMap持有对象的强引用，按照Lru算法进行对象淘汰。具体来说假设我们从表尾访问数据，在表头删除数据，当访问的数据项在链表中存在时，则将该数据项移动到表尾，否则在表尾新建一个数据项。当链表容量超过一定阈值，则溢出表头的数据

#### 为什么会选择LinkedHashMap呢？

- LinkedHashMap的构造函数里有个布尔参数accessOrder，当它为true时，LinkedHashMap会以访问顺序为序排列元素，否则以插入顺序为序排序元素

## PathClassLoader与DexClassLoader有什么区别？

- PathClassLoader：只能加载已经安装到Android系统的APK文件，即/data/app目录，Android默认的类加载器
- DexClassLoader：可以加载任意目录下的dex、jar、apk、zip文件

## 如何优化WebView，如何提高WebView的加载速度？

#### 为什么WebView加载会慢

- 这是因为在客户端中，加载H5页面之前，需要先初始化WebView，在WebView完全初始化完成之前，后续的界面加载过程都是被阻塞的

### 优化手段围绕以下两个点进行：

- 1、预加载WebView
- 2、加载WebView的同时，请求H5页面数据

### 常见的优化方法：

- 1、全局WebView
- 2、客户端代理页面请求。WebView初始化完成后向客户端请求数据
- 3、asset存放离线包

### 除此之外还有一些其他的优化手段

- 脚本执行慢，可以让脚本最后运行，不阻塞页面解析
- DNS与链接慢，可以让客户端复用使用的域名与连接
- React框架代码执行慢，可以将这部分代码拆分出来，提前进行解析

## Java和JS的相互调用怎么实现，有做过什么优化吗？

## JNI了解吗？，Java与C++如何相互调用

### Java调用C++

- 1、在Java中声明Native方法(即需要调用的本地方法)
- 2、编译上述Java源文件javac(得到.class文件)
- 3、通过javah命令导出JNI的头文件(.h文件)
- 4、使用Java需要交互的本地代码实现Java中声明的Native方法
- 5、编译.so库文件
- 6、通过Java命令执行Java程序，最终实现Java调用本地文件

### C++调用Java

- 1、从classpath路径下搜索ClassMethod这个类，并返回该类的Class对象
- 2、获取类的默认构造方法ID
- 3、查找实例方法的ID
- 4、创建该类的实例
- 5、调用对象的实例方法
```
JNIEXPORT void JNICALL Java_com_study_jnilearn_AccessMethod_callJavaInstaceMethod  
(JNIEnv *env, jclass cls)  
{  
    jclass clazz = NULL;  
    jobject jobj = NULL;  
    jmethodID mid_construct = NULL;  
    jmethodID mid_instance = NULL;  
    jstring str_arg = NULL;  
    // 1、从classpath路径下搜索ClassMethod这个类，并返回该类的Class对象  
    clazz = (*env)->FindClass(env, "com/study/jnilearn/ClassMethod");  
    if (clazz == NULL) {  
        printf("找不到'com.study.jnilearn.ClassMethod'这个类");  
        return;  
    }  

    // 2、获取类的默认构造方法ID  
    mid_construct = (*env)->GetMethodID(env,clazz, "<init>","()V");  
    if (mid_construct == NULL) {  
        printf("找不到默认的构造方法");  
        return;  
    }  

    // 3、查找实例方法的ID  
    mid_instance = (*env)->GetMethodID(env, clazz, "callInstanceMethod", "(Ljava/lang/String;I)V");  
    if (mid_instance == NULL) {  

        return;  
    }  

    // 4、创建该类的实例  
    jobj = (*env)->NewObject(env,clazz,mid_construct);  
    if (jobj == NULL) {  
        printf("在com.study.jnilearn.ClassMethod类中找不到callInstanceMethod方法");  
        return;  
    }  

    // 5、调用对象的实例方法  
    str_arg = (*env)->NewStringUTF(env,"我是实例方法");  
    (*env)->CallVoidMethod(env,jobj,mid_instance,str_arg,200);  

    // 删除局部引用  
    (*env)->DeleteLocalRef(env,clazz);  
    (*env)->DeleteLocalRef(env,jobj);  
    (*env)->DeleteLocalRef(env,str_arg);  
}  
```

## 了解插件化和热修复吗，它们有什么区别，理解它们的原理吗？

- 插件化：插件化是体现在功能拆分方面，它将某个功能独立提取出来，独立开发，独立测试，再插入到主应用中，依次来扩大较少应用的规模
- 热修复：热修复是体现在bug修复方面的，它实现的是不需要重新发版和重新安装，就可以去修复已知的bug

### 利用PathClassLoader和DexClassLoader去加载与bug类同名的类，替换掉bug类，进而达到修复bug的目的，原理是在app打包的时候阻止类打上CLASS_ISPREVERIFIED标志，然后再热修复的时候动态改变BaseDexClassLoader对象间接引用的dexElements，替换掉旧的类

### 目前热修复框架主要分为两大类：

- Sophix：修改方法指针
- Tinker：修改dex数组元素

## 如何做性能优化

- 1、节制的使用Service，当启动一个Service时，系统总是倾向于保留这个Service依赖的进程，这样会造成系统资源的浪费，可以使用IntentService，执行完成任务后会自动停止
- 2、当界面不可见时释放内存，可以重写Activity的onTrimMemory()方法，然后监听TRIM_MEMORY_UI_HIDDEN这个级别，这个级别说明用户离开了页面，可以考虑释放内存和资源
- 3、避免在Bitmap浪费过多的内存，使用压缩过的图片，也可以使用Fresco等库优化对Bitma显示的管理
- 4、使用优化过的数据集合SparseArray代替HashMap，HashMap为每个键值都提供一个对象入口，使用SpareArray可以免去基本对象类型转换为引用数据类型的时间

## 如何防止过度绘制，如何做布局优化？

- 1、使用include复用布局文件
- 2、使用merge标签避免嵌套布局
- 3、使用stub标签仅在需要的时候展示出来

## 如何提高代码质量

- 1、避免创建不必要的对象，尽可能避免频繁的创建临时对象，例如在for循环内，减少GC的次数
- 2、尽量使用基本数据类型代替引用数据类型
- 3、静态方法调用效率高于动态方法，也可以避免创建额外对象
- 4、对于基本数据类型和String类型的常量要使用static final修饰，这样常量会在dex文件的初始化器中进行初始化，使用的时候可以直接使用
- 5、多使用系统API，例如数组拷贝System.arrayCopy()方法，要比我们用for循环效率快9倍以上，因为系统API很多都是通过底层的汇编模式执行的，效率比较高

## 有没有遇到64k问题，为什么会出现这个问题，如何解决？

- 在DEX文件中，method、field、class等的个数使用short类型来做索引，即两个字节(65535)、method、field、class等均有此限制
- APK在安装过程中会调用dexopt将DEX文件优化成ODEX文件，dexopt使用LinearAlloc来存储应用信息，关于LinearAlloc缓冲区大小，不同的版本经历了4M/8M/16M的限制，超出缓冲区时就会抛出INSTALL_FAILED_DEXOPT错误

### 解决方案是Google的MultiDex方案

## MVC、MVP与MVVM之间的对比分析

- MVC：PC时代就有的架构方案，在Android上也是最早的方案，Activity/Fragment承担了V的角色，也承担了C的角色，小项目开发效率高，但大项目耦合过重，Activity/Fragment类过大
- MVP：为了解决MVC耦合过重的问题，MVP的核心思想就是提供一个Presenter将视图逻辑和业务逻辑相分离，达到解耦的目的
- MVVM：使用ViewModel代替Presenter，实现数据与VIew的双向绑定，这套框架最早使用的data-binding将数据绑定到xml中，这么做在大规模应用的时候是不行的，不过数据绑定是一个很有用的概念，后续Google又推出了ViewModel组件与LiveData组件。ViewModel组件规范了ViewModel所处的地位、生命周期、生产方式以及一个Activity下多个Fragment共享View Model数据的问题。LiveData组件则提供了在Java层面VView订阅ViewModel数据源的实现方案