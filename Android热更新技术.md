# Android热更新技术

### 什么是热更新

- “热”通常是指程序运行过程中，比如：SD卡热插拔，SIM卡热插拔，所以热更新就是指在程序运行过程中更新程序代码

### Android热更新产生背景

- 更快的修复BUG
- 数据和策略的微调

### 热更新技术的用途

- ==线上紧急BUG修复==
- 零碎功能上线（布局微调，策略的切换）
- 本地数据更新（城市公交，驾考助手）

### 热修复和传统发版的优缺点对比

#### 传统发版

- 流程复杂
- 用户主动更新
- 流量大
- 渠道包多
- 审核周期长
- 覆盖率低
- 无限制

#### 热修复

- 流程简单
- 用户无感知
- 流量小
- 只需一个补丁包
- 无需审核
- 覆盖率高
- 功能受限

### 常见的Android热修复框架

- AndFix
- Nuwa，RocooFix
- Tinker 微信
- Robust 美团 基于Google

### 对热修复的需求

- 方法修改
- 增加方法
- 修改res资源
- 新增图片
- 新增四大组件
- 修改so库

### 三类底层

- Native Hook
- ClassLoader
- Dex替换

#### Native Hook(AndFix)

- 通过C/C++层方法替换实现方法热修复

##### 实现步骤

- 比较新apk和旧apk
- 找到修改的类和方法，并用注解打上标记，生成补丁文件
- 加载补丁文件，找到被标记的方法
- 调用native方法hook替换方法

#### ClassLoader(Nuwa)

- 基于multidex方案切入的热修复方案

##### 实现步骤

- 找到所有修改的类
- 生成新的dex文件
- 用反射强行把心的dex放到dexElements数组最前面
- 同名类系统会优先加载修复后的，原先的会被丢弃

#### Dex替换(Tinker)

##### 实现步骤

- 比较新apk和旧apk
- 生成差分包
- App加载差分包生成新Dex文件
- 全量替换Dex文件
- 重新启动app生效


## Native Hook

### 原理

- 通过DexDiffer().diff()方法比较两个apk,找到所有修改的方法
- 为修改的方法加上自定义注解@MethodReplace(clazz="com.xx.TestClass",method="test")
- 签名打包信息,生成.apatch文件
- 加载补丁,读取所有修改的方法
- 调用native void replaceMethod()底层函数替换方法
- extern void dalvik_replaceMethod();/extern void art_replaceMethod();

## ClassLoader

### 原理

- 对比代码找到所有修改过的类
- 把这些类放进一个新生成的Dex文件中
- 用反射强行把新的**Dex**放到**dexElements**数组最前面
- 代码注入防止类被打上 ==IS_PREVERIFIED==标志
- dexElements数组中存在同名类，系统会优先加载前面的，原先的类被抛弃

#### IS_PREVERIFIED标志

- 是类加载的一种优化策略，目的是提高性能
- 虚拟机在启动的时候，如**果verify**选项被打开，就会执行**dvmVerifyClass**进行校验
- 如果一个类的**stati**c方法、**private**方法、**构造方法**中直接引用的类（第一层）和这个类是在同一个Dex中，那这个类就会被打上==IS_PREVERIFIED==标记
- 在DexOpt优化Dex时，如果类被打上IS_PREVERIFIED标记，那就会对类进行校验，如果这个类引用其他Dex中的类，就会报错
- 解决方法：在所有类的构造方法中加入一行代码
```
System.out.println(AntilazyLoad.class);
//AntilazyLoad是被打包在一个单独的hack.dex中
```
- 字节码插入

## Dex替换 Tinker

### 原理

- 比较两个apk的Dex,通过diff算法生成差分包
- 把补丁包下发给app(主动或被动)
- App在加载补丁文件之前会对补丁文件进行检查
- 补丁文件通过检查后app加载补丁(文件合成和Dex替换)
- 重启App修复生效  
