# Activity的生命周期

## 典型情况下的生命周期

### 7个生命周期

- onCreate:Activity正在被创建（初始化布局和数据）
- onRestart：Activity正在重新启动，从不可见变为可见状态（从Home或者新Activity返回旧Activity）
- onStart：Activity正在被启动，已经显示出来，但是没有出现在前台（无法和用户交互）
- onResume：Activity已经可见了，显示到前台（可以交互）
- onPause：Activity正在被停止（可以做存储数据、停止动画等操作，但不能做耗时操作，因为onPause执行完才会执行新Activity的onResume）
- onStop：Activity即将停止（可以做稍微重量级的回收，但同样不能太耗时）
- onDestroy：Activity即将被销毁（回收和最终的资源释放）

### 一些特殊情况

- 1、A中启动B，如果B是透明主题，A的onStop不会被调用
- 2、从B中返回A，A的生命周期：onRestart-> onStart-> onResume
- 3、onStart和onStop在该Activity是否在可见时回调，而onResume和onPause则在Activity是否在前台时回调（可见：只是说显示，但不一定是用户可以看到、交互;前台：就是看得见、摸得着）
- 4、如何实现点击返回键，Activity的onDestroy不被执行

```
//在所在的 Activity 中重写 onKeyDown() 方法，拦截返回事件，然后调用 moveTaskToBack() 方法：
@Override
public boolean onKeyDown(int keyCode, KeyEvent event) {
    if (keyCode == KeyEvent.KEYCODE_BACK){
        moveTaskToBack(true);   //将当前 Activity 的 Task 放到 Activity 栈的后边
        return false;
    }
    return super.onKeyDown(keyCode, event);
}
```
- 5、Activity的启动流程简述：(1)Instrumentation处理启动Activity的请求，然后通过Binder将请求发给AMS，(2)AMS维护者一个ActivityStack并负责栈中Activity的状态管理，(3)AMS通过ActivityThread去同步Activity的状态，从而完成生命周期的调用

### 异常情况下的生命周期
- 系统回收或者当前设备Configuration改变导致Activity被销毁重建的情况

### 异常状态保存/恢复方法

- 在系统配置发生改变时，默认情况下Activity会被销毁重建
- 异常终止的情况下会调用onSaveInstanceState()方法，重新创建后会调用onRestoreInstanceState()
- 状态保存调用顺序：onPause-> onSaveInstanceState-> onStop
- 状态回复调用顺序：onStart-> onSaveInstanceState-> onResume
- 数据通过键值对的形式保存到Bundle中
- 数据恢复在onCreate或者onRestoreInstance中进行都可以，但是官方文件建议在onRestoreInstance中，因为它被调用时bundle一定是有值的，不需要判断

### 系统自动做得保存/恢复工作

- 在Activity的异常情况下，系统会给这两个保存、恢复方法中为我们做一定的工作，比如保存当前Activity的视图结构（View的状态）

#### Activity异常终止时，系统保存View状态的流程简述

- Activity调用onSaveInstanceState保存数据
- 然后Activity委托Window保存数据
- Window再委托上面的顶级容器保存数据
- 顶级容器（一般来说是DecorView）再一一通知它的子元素保存数据
- 委托思想：上层委托下层去处理一件事，比如这里的数据恢复，还有View的绘制过程、事件分发等

### 系统内存不足时，优先杀死低优先级的Activity

#### Activity的三中优先级，从高到低顺序

- 1、前台Activity（正在和用户交互，优先级最高，最不可能被回收）
- 2、可见但非前台（比如弹出Dialog的Activity）
- 3、后台Activity（已经暂停，执行了onStop，优先级最低）

### 指定在某些配置改变时Activity不重建

- 我们可以在AndroidManifest.xml中配置android:configChanges来指定该Activity在哪些系统配置改变时不重新简历

#### 配置项很多，常用的四个：

```
android:configChanges="screenSize|orientation|keyboardHidden|locale"
```
- **screenSize|orientation**：指的是在屏幕旋转和尺寸改变时不重新创建
- **keyboardHidden**：指的是可用键盘的改变
- **locale**：指的是系统语言切换

现在，当其中一个配置发生变化时，Activity 不会重启。相反，Activity 会调用 onConfigurationChanged()方法，并且向此方法传递 Configuration 对象，这个对象代表当前所有配置，你可以根据不同配置进行不同的处理：

```
@Override
public void onConfigurationChanged(Configuration newConfig) {
    super.onConfigurationChanged(newConfig);

    // Checks the orientation of the screen
    if (newConfig.orientation == Configuration.ORIENTATION_LANDSCAPE) {
        Toast.makeText(this, "landscape", Toast.LENGTH_SHORT).show();
    } else if (newConfig.orientation == Configuration.ORIENTATION_PORTRAIT){
        Toast.makeText(this, "portrait", Toast.LENGTH_SHORT).show();
    }
}
```
- 如果在配置改变时仍使用旧的状态，则可以不实现 onConfigurationChanged()。