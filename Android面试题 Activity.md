## Android面试题 Activity

### Android四大组件简介

#### 请简要介绍Android的四大组件

- 1、Activity用来显示Android的程序界面，一个应用往往有多个界面，所以一个应用中会有多个Activity
- 2、Service没有界面的后台服务，会一直运行在后台，常被用来做数据处理，也可以做一些定时的任务
- 3、BroadcastReceiver是Android的广播接收器，在广播机制中充当广播的接收者的作用，Android中充满了各种广播，所有需要有选择地接收一些有用的广播，然后处理这些广播
- 4、ContentProvider可直译为内容提供者，它是用在不同的应用程序之间共享数据时，可以把一个应用的数据提供给其他的应用使用

#### Android中Activity，Intent，ContentProvider，Service各有什么区别

- 1、Activity：活动，是最基本的Android应用程序组件。一个活动就是一个单独的屏幕，每个活动都被实现为一个独立的类，并且从活动基类继承而来的
- 2、Intent：意图，描述应用想干什么，最重要的部分是动作和动作对应的数据
- 3、ContentProvider：内容提供器，Android应用程序能够将它们的数据保存到文件、SQLite数据库中，甚至是任何有效的设备中。当你想将你的应用数据和其他应用共享时，内容提供器接可以发挥作用了
- 4、Service：服务，具有一段较长生命周期且没有用户界面的程序

#### Manifest.xml文件中主要包括哪些信息？

- 1、manifest：根节点，描述了package中所有的内容
- 2、user-permission：请求你的package正常运作所需赋予的安全许可
- 3、permission：声明了安全可许来限制哪些程序能使用package中的组件和功能
- 4、instrumentation：声明了用来测试此package或其他package指令组件的代码
- 5、application：包含package中application级别组件声明的根节点
- 6、activity：Activity是用来与用户交互的主要工具
- 7、receiver：IntentService能使得application获得数据的改变或者发生的的操作，即使它当前不在运行
- 8、Service：Service是能在后台运行任意时间的组件
- 9、provider：ContentProvider是用来管理持久化数据并发布给其他应用程序使用的组件

### Activity常见知识点

#### 1、什么是Activity？

- 四大组件之一，一个用户交互界面对应一个Activity setContentView()//要显示的布局 button.setOnclickListener{},activity是Context的子类，同时实现了window.callback和keyevent.callback，可以处理与窗体用户交互的事件，我们开发常用的有：FragmentActivityListActivity、PreferenceActivity，TabActivity等

#### 2、Activity生命周期

- Activity从创建到销毁有多种状态，从一种状态到另一种状态时会激发相应的回调方法，这些回调方法包括：onCreate(),onStart(),onResume(),onPause(),onStop(),onRestart(),onDestroy()
- 这些方法都是两两对应的,onCreate 创建与 onDestroy 销毁; 
onStart 可见与 onStop 不可见;onResume 可编辑(即焦点)与 onPause;如果界面有共同的特点或者功能的时候,还会自己定义一个 BaseActivity. 进度对话框的显示与销毁 。 

##### 方法作用
- onCreate：Activity 创建时的初始化工作 如设置页面的ContentView、接收参数等.

- onRestart：在Activity的onStop后重新启动时，回调onRestart

- onStart：正在启动，即将开始，没有出现在前台，还无法和用户交互，可以理解为已经初始化完成，但是处于后台我们暂时没法看见。

- onResume：可见了并且处于激活态,处于前台，和onStart最大的不同就是onStart是在后台已经初始化完，但是无法交互。

- onPause：失去焦点不可以交互、处于后台。

- onStop：即将停止，做一些稍微重量级回收类的工作

- onDestory：Activity即将被销毁，需要们做一些回收和资源释放类的工作。

##### 提醒

- 从Activity A,打开另一个Activity B，B这个Activity是在A的onPause()执行后才变成可见状态的，所以为了不影响B的显示，最好不要再onPause里执行一些耗时操作，可以考虑将这些操作方到onStop里，这时B已经可见了，(A->onPause后才B->onCreate,onStart,onResume)

#### 3、如何保存Activity的状态？

- Activity的状态通常情况下系统会自动保存的，只有当我们需要保存额外的数据时才需要使用到这样的功能
- 一般来说，调用onPause()和onStop()方法后的Activity实例仍然存在内存中，Activity的所有信息和状态数据不会消息，当Activity重新回到前台之后，所有的改变都会得到保留
- 但是当系统内存不足时，调用onPause()和onStop()方法后的Activity可能会被系统销毁，此时内存中就不会存在该Activity的实例对象了
- 如果之后这个Activity重新回到前台，之前所作的改变就会消失
- 为了避免这种情况的发生，我们可以覆写onSaveInstanceState()方法
- onSaveInstanceState()方法接受一个Bundle类型的参数，开发者可以将状态数据存储到这个Bundle对象中，这样即使Activity被系统销毁，当用户重新启动这个Activity而调用的onCreate()方法时，上述的Bundle对象会作为实参传递给onCreate()方法，开发者可以从Bundle对象中取出保存的数据，然后利用这些数据将Activity恢复到被销毁之前的状态
- 需要注意的是：onSaveInstanceState()方法并一定会被调用的，因为有些场景不需要保存状态数据，比如用户按下BACK键退出Activity时，用户显然想要关闭这个Activity，此时是没有必要保存数据以供下次恢复，也就是onSaveInstanceState()方法不会被调用
- 如果调用onSaveInstanceState()方法，调用将发生在onPause()或onStop()之前

#### 4、两个Activity之间跳转时必然会执行的是哪几个方法？

- 一般情况下，比如两个Activity A，B，当在A里面激活B组件的时候，A会调用onPause()方法，然后B调用onCreate()、onStart()、onResume()
- 这个时候B覆盖了窗体，A会调用onStop()方法；如果B是透明的，或者是对话款高德样式，就不会调用A的onStop()方法

#### 5、横竖屏切换时Activity的生命周期

- 此时的生命周期跟清单文件里的配置有关系
- 1、不设置Activity的Android：configChange时，切屏会重新调用各个生命周期，默认首先销毁当前Activity，然后重新加载
- 2、设置Activity的Android：configChange="orientation|keyboardHidden|screenSize"时，切屏不会重新调用各个生命周期，只会执行onConfigurationChanged方法

#### 6、如何将一个Activity设置成窗口的样式？

- 只需要给Activity配置如下属性：android：theme="@android:style/Theme.Dialog"

#### 7、如何退出Activity？如何安全退出已调用多个Activity的Application？

- 1、通常情况用户退出一个Activity只需按返回键，想退出Activity直接调用finish()方法就行
- 2、发送特定广播：在需要结束应用时，发送一个特定的广播，每个Activity收到广播后，关闭即可(//给某个activity注册接受,接受广播的意图registerReceiver(receiver, filter) 
//如果接受到的是关闭activity的广播activity的finish()掉)
- 3、递归退出：就调用finish()方法把当前的关闭，在打开新的Activity时使用startActivityForResult()，然后自己加标志，在onActivityResult中处理，递归关闭
- 4、其实也可以通过intent的flag来实现Intent.setFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP)激活一个新的Activity，此时如果该任务栈中已经有该Activity，那么系统会把这个Activity上面的所有Activity干掉。其实相当于给Activity配置的启动模式为SingleTop
- 5、记录打开的Activity：每打开一个Activity，就记录下来，在需要退出时，关闭每一个Activity

#### 8、Activity的四种启动模式，singleTop和singleTask区别是什么？一般书签的使用模式是singleTop，那为什么不适用singleTask？

- singleTop跟standard模式比较类似。唯一的区别就是，当跳转的对象是位于栈顶的Activity(应该可以理解为用户眼前所看到的Activity)时，程序将不会生成一个新的Activity实例，而是直接跳到现存于栈顶的那个Activity实例
- 当Activity为singleTop模式时，执行跳转后栈里面依旧只有一个实例，如果现在按返回键程序将直接退出
- singleTask模式和singleInstance模式都是只创建一个实例。在这种模式下，无论跳转的对象是不是位于栈顶的Activity，程序都不会生成一个新的实例(当然前提是栈里面已经有这个实例)。
- 这种模式相当有用，在以后的多Activity开发中，常会因为跳转的关系导致同个页面生成多个实例，这个在用户体验上始终有点不好，而如果你将对应的Activity声明为singleTask模式，这种问题就不复存在

#### 9、Android中的Context，Activity，Application有什么区别？

- 相同：Activity和Application都是Context的子类
- Context：上下文的意思，在实际应用中它起到了管理上下文环境中各个参数和变量的作用，方便我们可以简单的访问到各种资源
- 不同：维护的生命周期不同。Context维护的是当前的Activity的生命周期，Application维护的是整个项目的生命周期

##### 使用Context的时候，小心内存泄漏，防止内存泄漏，注意一下几个方面：

- 1、不要让生命周期长的对象引用Activity的Context，即保证引用Activity的对象要与Activity本身生命周期一样
- 2、对于生命周期长的对象，可以使用application的Context
- 3、避免非静态的内部类，尽量使用静态类，避免生命周期问题，注意内部类对外部对象引用导致的声明周期变化

#### 10、Context是什么

- 1、它描述的是一个应用程序环境的信息，即上下文
- 2、该类是一个抽象类，Android提供了该抽象类的具体实现类(ContextIml)
- 3、通过它我们可以获取应用程序的资源和类，也包括一些应用级别操作，例如：启动一个Activity，发送广播，接收Intent信息等

#### 11、Task&BackTask

- 由同一个应用启动的Activity默认都在同一个任务栈中(Task)
- 任务栈的形式一样遵循后进先出的原则，看图很好理解

#### 12、App A 调用 App B的Activity，这个Task是什么情况

- 1、默认情况：appB的Activity是嵌入到了appA的Task中，但是不影响appB的正常运行，appB有自己的Task
- 2、FLAG_NEW_TASK：appB的Activity不嵌入到appA的Task中，而是加入到appB自己的Task
- 3、FLAG_ACTIVITY_CLEAR_TOP：当Intent对象包含这个标记时，如果在栈中发现存在Activity实例，则清空这个实例之上的Activity，使其处于栈顶
- 4、FLAG_ACTIVITY_SINGLE_TOP：在使用默认的"standard"启动模式下，如果没有再Intent使用到FLAG_ACTIVITY_SINGLE_TOP标记，那么它将关闭后重建，如果使用了这个FLAG_ACTIVITY_SINGLE_TOP标记，则会使用已存在的实例

#### 13、如何获取当前屏幕Activity的对象？

- 使用ActivityLifecycleCallbacks

#### 14、onNewIntent

- 如果IntentActivity处于任务栈的顶端，也就是说之前打开过Activity，现在处于onPause、onStop状态的话，其他应用再发送Intent的话，执行顺序为：onNewIntent，onRestart，onStart，onResume

#### 15、除了用Intent去启动一个Activity，还有其他方法吗？

- 使用adb shell am命令
- 1、am启动一个Activity
- 2、am发送一个广播，使用action

#### 16、Android Service与Activity之间通信的几种方式？

- 1、通过Binder对象：当Activity通过调用bindService(Intent service,ServiceConnection conn,int flags)，得到一个Service的对象，通过这个对象我们可以直接访问Service中的方法
- 2、通过BroadcastReceiver(广播)的形式
- 3、EventBus是一个Android开源事件总线框架

#### 17、TaskAffinity是什么？

- 标识Activity任务栈名称的属性：TaskAffinity，默认为应用包名

#### 18、如果新Activity是透明主题时，旧Activity不会走onStop()