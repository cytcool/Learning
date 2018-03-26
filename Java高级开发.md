# Java高级开发

### Lambda表达式(函数式表达式)


```
interface IMessage{
    public void print();
}

public static void main(String[] args){
    IMessage msg = new IMessage(){
        public void print(){
          System.out.println("Hello World!")；
        }
    }
}
```

```
//使用Lambda表达式,只有一条语句的情况
public static void main(String[] args){
    IMessage msg = () -> System.out.println("Hello World!")；
}

```

```
//使用Lambda表达式，有多条语句情况
public static void main(String[] args){
    IMessage msg = () ->{
    System.out.println("Hello World!")
    System.out.println("Hello World!")
    System.out.println("Hello World!")
    }；
}

```



```
@FuntionalInterface
//新的注释 规定了接口只能有一个抽象方法
interface IMessage{
    public void print();
}
```


```
//带参方法
interface IMessage{
    public void add(int x,int y);
}
public static void main(String[] args){
    IMessage msg = (p1,p2) -> p1+p2;
    //p1,p2参数名随便定义即可
    System.out.println(msg.add(10,20));
}
```

### 方法引用

**引用的本质就是别名**

1. 引用静态方法：类名称 ::static 方法名称
2. 引用某个对象的方法：实例化对象 :: 普通方法名称
3. 引用某个特定类的方法：类名称 :: 普通方法名称
4. 引用构造方法：类名称 ::new。


```
//1、引用静态方法
interface IUtil<P,R>{
    public R zhuanhuan(P p);
}
public class TestDemo{
    public static void main(String[] args){
        IUtil<Integer,String> iu = String :: valueOf;
        String str = iu.zhuanhuan(1000);//相当于 String.valueOf(1000);
    
    
    }
}
//就相当于为方法 valueOf() 起了别名

```

```
//2、引用某个对象的方法
interface IUtil<P,R>{
    public R zhuanhuan();
}
public class TestDemo{
    public static void main(String[] args){
        IUtil<String> iu = "hello" :: toUpperCase;
        System.out.println(iu.zhuanhuan);//相当于"hello".toUpperCase;
    
    
    }
}
```


```
3、引用类中的普通方法
interface IUtil<P,R>{
    public R compare(P p1,P p2);
}
public class TestDemo{
    public static void main(String[] args){
        IUtil<Integer,String> iu = String :: compareTo; 
        System.out.println(iu.compare("H","h");
    
    }
}
```

```
//4、引用构造方法
class Person{
    String name;
    int age;
    public Person(String name,int age){
        this.name = name;
        this.age = age;
    }
    public String toString(){
        return "name="+name+" age="+age
    }
}
interface IUtil<R,FP,SP>{
    public R create(FP p1,SP p2);
}
public class TestDemo{
    public static void main(String[] args){
        IUtil<Person,String,Integer> iu = Person :: new; 
        System.out.println(iu.create("张三","20");
    
    }
}
```

### Java多线程实现

#### 实现一个多线程的主类
- 继承一个Thread类
- 【推荐】 实现Runnable、Callable接口


#### 继承Thread类实现多线程

- 启动多线程的唯一方法：public void start();
- 通过start()方法来调用run()方法，而不是直接调用run()方法;
- 每个线程只能启动一次，不能启动多次，否则会抛异常;

- 注释：Thread类中有一个start0()方法，该方法只声明别未实现，该方法是由不同版本的JVM来实现private native void start0();
- Thread类的核心功能是进行线程的启动，但是如果一个类直接继承了Thread类所造成的问题是单继承局限;

#### Runnable接口实现多线程


```
@FunctionalInterface
public interface Runnable{
    public void run();
}
//该注解标识该接口只能有一个抽象方法
//接口中只有一个run()方法
//run()方法没有返回值
```
##### 实现多线程的启动

```
public class TestDemo{
    public static void main(String[] args){
       MyThread mt1 = new MyThread("线程A");
       MyThread mt2 = new MyThread("线程B");
       new Thread(mt1).start();
       new Thread(mt2).start();
    }
}
//采用内部类方法调用
//多线程的启动永远都是Thread类的start()方法
```

- Runnable接口可以使用匿名内部类
- 也可以使用Lambda表达式

#### Thread类与Runnable接口的区别

##### Thread类的定义
- public class Thread extends Object implement Runnable
- Thread类是Runnable接口的子类

##### Thread类中的run()方法

```
//源代码

private Runnable target

public void run(){
    if(target!=null)
        target.run();
}
```


```
//源代码
public Thread(Runnable target){
    init(null,target,"Thread-"+nextThreadNum(),0);
}
```
##### Runnable接口能更好的实现数据共享的操作

#### 线程的运行状态(面试题)

 [link](http://blog.51cto.com/lavasoft/99153)
 
#### Callable接口实现多线程


```
@FunctionalInterface
public interface Callable<V>{
    public V call() throws Exception;
}
```

##### 启动并取得线程结果
```
class MyThread implements java.util.concurrent.Callable<String>{

    @Override
    public String call() throws Exception {
        for (int i = 0; i < 20 ; i++) {
            System.out.println("卖票，i="+i);
        }
        return "票卖完了，下次请早";
    }
}
public class Main {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        FutureTask<String> task = new FutureTask<>(new MyThread());
        //通过FutureTask构造方法取得线程返回值
        new Thread(task).start();
        System.out.println(task.get());

    }
}

```


##### 与Runnable接口的区别
- Runnable的run()方法没有返回值，而Callable的run()方法有返回值


#### 线程的同步与死锁


#### 生产者与消费者

##### 面试题

请解释sleep()与wait的区别

- sleep()是Thread类中定义的方法，到了一定的时间后该休眠的线程可以自动唤醒
- wait()是Object类中定义的方法，如果要想唤醒，必须使用notify()、notifyAll()

### Java常用类库

#### StringBuffer

##### 面试题

请解释String、StringBuffer与StringBuilder的区别

- String的内容不可以修改，StringBuffer与StringBuilder可以修改;
- StringBuffer采用同步操作，属于线程安全操作，而StringBuilder采用异步操作，属于线程不安全操作

#### Runntime类

##### 面试题

什么是gc?如何处理

- gc(Garbage Collector):垃圾收集器，用于释放无用的内存空间;
- gc有两种处理形式，一种自动不定期调用，另一种是使用Runntime类中的gc()方法手工调用;

#### System类

##### 面试题

请解释 final、finally、finalize的区别
- final是一个关键字，用于定义不能被继承的父类，不能被覆写的方法和常量;
- finally是异常处理的统一出口;
- finalize是Object类中的一个方法，用于在对象回收前进行调用

#### 对象克隆

- 要想实现对象克隆，需要被克隆的对象所在的类一定要实现Cloneable接口;
- Cloneable接口没有任何抽象方法，该接口只是一个标识接口，表示一种能力;
- 还需要在对象所在的类定义覆写clone()方法;


### Java类集

#### ArrayList与Vector区别

- ArrayList处理形式：异步处理，性能更高;
- Vector处理形式：同步处理，性能降低;
- ArrayList非线程安全;
- Vector线程安全;
- ArrayList输出形式：Iterator、ListIterator、foreach
- Vector输出形式：Iterator、ListIterator、foreach、Enumeration

#### ArrayList与LinkedList区别

- ArrayList封装的是一个数组
- LinkedList封装的是一个链表
- ArrayList的时间复杂度是1
- LinkedList的时间复杂度是n