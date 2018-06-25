## 浅谈Android MVC、MVP和MVVM

### 一、MVC(Model,View,Controller)

- MVC模式是最经典开发模式之一，它分为三个部分Model，View，Controller

#### 模型层(Model)

- 数据模型，是对客观事物的抽象

#### 视图层(View)

- 用户界面，是model的具体表现形式

#### 控制器层(Controller)

- 业务逻辑，主要负责与model与view打交道

#### 使用场景

- 适用于功能较少，业务逻辑较少的项目

### MVC的优缺点

#### 优点

- 1、业务逻辑全部分离到Controller中，模块化程度高
- 2、观察者模式可以做到多视图同时更新

#### 缺点

- 1、Model和View之间是直接进行交互，就必然会导致Model和View之间的耦合
- 2、所有逻辑都写在Controller层，导致Controller层特别臃肿

### 二、MVP(Model,View,Presenter)

- MVC架构方式的变种，使用Presenter来代替Controller，而且改变了数据的流向，View和Model之间不再直接进行交互，而是全部通过Presenter来进行。MVP模式就是把MVC模式中的Controller换成了Presenter
- 在MVP当中，Presenter可以同时操作VIew和Model，View需要提供一组对界面操作的接口给Presenter进行调用；
- Model仍然通过事件广播自己的变更，但由Presenter监听而不是View
- View与Model不发生联系，都通过Presenter传递

#### 适用场景

- 视图界面不是很多的项目中

### MVP的优缺点

#### 优点：

- 1、模型与视图完全分离，我们可以修改视图而不影响模型
- 2、可以更高效地适用模型，因为所有的交互都发生在一个地方--Presenter内部
- 3、我们可以将一个Presenter用于多个视图，而不需要改变Presenter的逻辑。这个特性非常的有用，因为视图的变化总是比模型的变化频繁
- 4、如果我们在逻辑放在Presenter中，那么我们就可以脱离用户接口来测试这些逻辑(单元测试)

#### 缺点：

- Presenter作为桥梁协调View和Model，就会导致Presenter变得很臃肿，维护比较困难

### 三、MVVM(Model,View,ViewModel)

- MVVM其实是对MVP的一种改良，它将Presenter替换成了ViewModel，并通过双向的数据绑定来实现视图和数据的交互

#### 适用场景

- 适用于界面展示的数据较多的项目

### MVVM的优缺点

#### 优点：

- 1、低耦合。视图(View)可以独立于Model变化和修改，一个ViewModel可以绑定到不同的View上，当View变化的时候Model可以不变，当Model变化的时候View也可以不变
- 2、可重用性。你可以把一些视图逻辑放在一个ViewModel里面，让很多View重用这个视图逻辑
- 3、独立开发。开发人员可以专注于业务逻辑和数据的开发(ViewModel)，设计人员可以专注于页面设计，使用Expression Blend可以很容易设计界面并生成xml代码
- 4、可测试。界面素来是比较难于测试，而现在测试可以针对ViewModel来写
- 5、提高可维护性。解决了MVP大量的手动View和Model同步的问题，提供双向绑定机制。提高了代码的可维护性

#### 缺点：

- 1、过于简单的图形界面不适用
- 2、对于大型的图形应用程序，视图状态较多，ViewModel的构建和维护的成本都会比较高
- 3、数据绑定的声明是指令式地写在View的模板当中，这些内容是没办法去打断点debug的
- 4、目前这种架构方式的实现方式比较不完善规范，常见的就是DataBinding框架