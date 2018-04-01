# Java反射

## 一、编译时vs运行时

- 编译时：将Java代码编译成.class文件的过程
- 运行时：就是Java虚拟机执行.class文件的过程

**PS：两者的区别是编译时涉及内存，而运行时不涉及内存**

### 编译时类型和运行时类型

- 编译时类型：编译时类型由声明该变量时使用的类型决定
- 运行时类型：运行时类型由实际赋给该变量的对象决定

```
Animal animal = new Dog();
```
- Animal类是编译时类型
- Dog()类是运行时类型

### 动态绑定-调用引用实例的方法

- 1、在编译时，是调用声明类型的成员方法（多态的实现原理），也就是所谓的编译时类型的方法
- 2、到了运行时，调用的是实际类型的成员方法，也就是所谓的运行时类型的方法
- 3、对于调用引用实例的成员变量，无论是编译时还是运行时，均是调用编译时类型的成员变量

## 二、什么是反射

- 在==运行状态==中，对于任意一个类，都能够知道这个类的所有属性和方法;对于任意一个对象，都能够调用它的任意一个方法和属性
- 本质就是这种在==运行状态==下动态获取信息和动态调用对象和方法

## 三、class类

- .class文件
- Class对象

## 四、反射的应用

- 方式一：

```
Person person = new Person();
Class<? extends Person> personClazz01 = person.getClass();
```

- 方式二：

```
Class<?> personClazz02 = Class.forName("Person");
```

- 方式三：

```
Class<? extends Person> personClazz03 = Person.class;
```

- 用反射实例化对象

```
public class TestDemo{
    public static void main(String[] args)throws Exception{
    Class<?> cls = Class.forName("java.utils.Date");//直接使用字符串描述要使用的类
    Object obj=cls.newInstance();//实例化对象，等价：new java.utils.Date()
    System.out.println(obj);
}
```


## 五、Android中反射的运用

- 1、通过原始的Java反射机制的方式调用资源
- 2、Activity的启动过程中Activity的对象的创建




 