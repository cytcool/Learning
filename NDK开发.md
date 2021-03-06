# NDK开发

## 基本概念

### JNI

- 定义：JNI是Java Native Interface的缩写，它提供了若干的API实现了Java和其他语言的通信，允许Java代码和其他语言写的代码进行交互

### 为什么要使用JNI？

- 效率上c/c++语言效率更高
- 代码移植，复用已经存在的C代码
- java反编译比C语言容易

### 跨平台概念

- 跨平台概念即不依赖于操作系统，也不依赖硬件环境，一个操作系统下开发的应用，放到另一个操作系统下依然可以运行

### JNI的副作用

- 程序不再跨平台，要想跨平台，必须在不同的系统环境下重新编译本地语言部分
- 程序不再是绝对安全的，本地的不当使用可能导致整个程序崩溃

### Android的五层架构

- 1、application：应用层
- 2、application framework：应用框架层
- 3、libraries和dalvik：函数库和虚拟机层
- 4、HAL（硬件抽象层）
- 5、linux kernel

### JNI与NDK的区别

- NDK（Native Development Kit）：Android NDK是一套允许您使用原生代码语言实现部分应用的工具集
- JNI是一种技术，NDK是一个工具

### 静态库和动态库

#### 库

- 库是写好的现有的，成熟的，可以复用的代码集合
- 本质上来说库是一种可执行代码的二进制形式，可以被操作系统载入内存执行
- 库有两种：静态库(.a .lib)和动态库(.so .dll)

#### 两者的区别

- 二者的不同点在于代码被载入的时刻不同
- 静态库的代码在编译过程中已经被载入可执行程序，因此体积较大
- 共享库的代码是在可执行程序时才载入内存的，在编译过程中仅简单的引用，因此代码体积较小

#### 动态库为什么又叫做共享库？

- 不同的应用程序如果调用相同的库，那么在内存里只需要有一份该共享库的实例