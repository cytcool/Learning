## 描述一下类的加载机制？
- 类的加载就是虚拟机通过一个类的全限定名来获取描述此类的二进制节流，而完成这个加载动作的就是类加载器
- 类和类加载器息息相关，判定两个类是否相等，只有在这两个类被==同一个加载器==加载的情况下才有意义，否则即便是两个类来自同一个Class文件，被不同类加载器加载，它们也是不相等的
- 注：这里的相等性包括Class对象的equals()方法、isAssignableFrom()方法、isInstance()方法的返回结果以及Instance关键字对对象所属关系的判定结果等

### 类加载器可以分为三类：

- 启动类加载器：负责加载<JAVA_HOME>\lib目录下或者被-Xbootclasspath参数所指定的路径的，并且是被虚拟机所识别的库到内存中。
- 扩展类加载器：负责加载<JAVA_HOME>\lib\ext目录下或者被java.ext.dirs系统变量所指定的路径的所有类库到内存中。
- 应用类加载器：负责加载用户类路径上的指定类库，如果应用程序中没有实现自己的类加载器，一般就是这个类加载器去加载应用程序中的类库。

### 当类在加载的时候回使用哪个加载器呢？

- 这里涉及到类加载器的双亲委派模型

#### 双亲委派模型工作流程

- 如果一个类加载器收到了加载类的请求，它不会自己立即去加载类，它会先去请求父类加载器，每个层次的类加载器都是如此。层层传递，直到传递到最高层的类加载器，只有当父类加载器反馈自己无法加载这个类，才会有当前子类加载器去加载该类
```
public abstract class ClassLoader {
    
    protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
            //首先，检查该类是否已经被加载
            Class c = findLoadedClass(name);
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    //先调用父类加载器去加载
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    //如果父类加载器没有加载到该类，则自己去执行加载
                    long t1 = System.nanoTime();
                    c = findClass(name);

                    // this is the defining class loader; record the stats
                }
            }
            return c;
    }
}
```

### 为什么要这么做呢？

- 这是为了要让越基础的类由越高层的类加载器加载，例如Object类，无论哪个类加载器去尝试加载这个类，最终都会传递给最高层的类加载器去加载，前面说过，类的相等性是由类与类加载器共同判定的，这样Object类无论在何种类加载器环境下都是同一个类
- 相反如果没有双亲委派模型，那么每个类加载器都会加载Object类，那么系统中就会出现多个不同的Object类了，如此一来系统的最基础的行为也就无法保证了

## 描述一下GC的原理和回收策略

### 首先，垃圾回收需要解决哪些问题

- 哪些内存需要回收？
- 什么时候回收？
- 如何回收？

#### 这些问题分别对应着引用管理和回收策略等方案

### 提到引用，Java中有四种引用类型：

- 强引用：代码中普遍存在的，只要强引用还在，垃圾收集器就不会回收掉被引用的对象
- 软引用：SoftReference，用来描述还有用但是非必须的对象，当内存不足的时候会回收这类对象
- 弱引用：WeakReference，用来描述非必须对象，弱引用的对象只能生存到下一次GC发生时，当GC发生时，无论内存是否足够，都会回收该对象
- 虚引用：PhantomReference，一个对象是否有虚引用的存在。完全不会对其生存时间产生影响，也无法通过虚引用取得一个对象的引用，它存在的唯一目的是在这个对象被回收时可以收到一个系统通知

#### 不同的引用类型，在做GC时会区别对待，我们平时生成的Java对象，默认都是强引用，也就是说只要强引用还在，GC就不会回收，那么如何判断强引用是否存在呢

- 一个简单的思路：引用计数法，有对这个对象的引用就+1，不再引用就-1，但是这种方式看起来简单没好，但它却不能解决循环引用计数的问题

#### 所以这里使用到了可达性分析算法，用它来判断对象的引用是否存在

- 可达性分析算法通过一系列称为GC Roots的对象作为起始点，从这些节点从上向下搜索，搜索走过的路径称为引用链，当一个对象没有任何引用链与GC Roots连接时就说明此对象不可用，也就是对象不可达

#### GC Roots对象通常包括：

- 虚拟机栈中引用的对象(栈帧中的本地变量表)
- 方法区中类的静态属性引用的对象
- 方法区中常量引用的对象
- Native方法引用的对象

#### 可达性分析算法整个流程

- 第一次标记：对象在经过可达性分析后发现没有雨GC Roots有引用链，则进行第一次标记并进行一次筛选，**筛选条件是**：该对象是否有必要执行finalize()方法。没有覆盖finalize()方法或者finalize()方法已经被执行过都会被认为没有必要执行；**如果有必要执行**：则该对象会被放在一个F-Queue队列，并稍后再由虚拟机建立的低优先级Finalizer线程中触发该对象的finalize()方法，但不保证一定等待它执行结束，因为如果这个对象的finalize()方法发生了死循环或者执行时间较长的情况，会阻塞F-Queue队列里的其他对象，影响GC
- 第二次标记：GC对F-Queue队列里的对象进行第二次标记，如果在第二次标记时该对象又成功被引用，则会被移除即将回收的集合，否则会被回收

## 接口和抽象类有什么区别？

### 共同点

- 1、是上层的抽象类
- 2、都不能被实例化
- 3、都能包含抽象方法，这些抽象的方法用于描述类具备的功能，但是不提供具体的实现

### 区别

- 1、在抽象类中可以写非抽象的方法，从而避免在子类中重复书写他们，这样可以提高代码的复用性，这是抽象类的又是，接口中只能有抽象的方法
- 2、一个类只能继承一个直接父类，这个父类可以使具体的类也可以是抽象类，但是一个类可以实现多个接口

## 内部类、静态内部类在业务中的应用场景是什么？

- 静态内部类：只是为了降低包的深度，方便类的使用，静态内部类适用于包含类当中，但又不依赖于外在的类，不用使用外在类的非静态属性和方法，只是为了方便管理类结构而定义。在创建静态内部类的时候，不需要外部类对象的引用
- 非静态内部类：持有外部类的引用。可以自由使用外部类的所有变量和方法
