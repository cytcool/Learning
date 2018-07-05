## Android面试题 布局DrawableView

### Android布局面试问题

#### Android中常用的布局有哪些

- FrameLayout
- RelativeLayout
- LinearLayout
- AbsoluteLayout
- TableLayout
- GrideLayout

#### 谈谈UI中，Padding和Margin有什么区别？

- android:padding是站在父View的角度描述问题，它规定它里面的内容必须与这个父View边界的距离
- android:margin是站在自己的角度描述问题，规定自己和其他的View之间的距离，如果同一级只有一个View，那么它的效果基本上就和padding一样了

#### 使用权重如何让一个空间的宽度为父控件的1/3?

- 可以在水平方向的LinearLayout中社会weightSum为3，然后让其子控件的weight为1，那么该子控件就是父控件的1/3

#### Android中布局的优化措施有哪些？

- 1、尽可能减少布局的嵌套层级：可以使用sdk提供的hierarchyviewer工具分析视图树，帮助我们发现没有用到的布局
- 2、不用设置不必要的背景，避免过度绘制，比如父控件设置了背景色，子控件完全将父控件覆盖的情况下，那么父控件就没有必要设置背景
- 3、使用<include>标签复用相同的布局代码
- 4、使用<merge>标签减少视图层次结构

##### 标签主要由两种用法

- 因为所有的Activity视图的根节点都是FrameLayout，因此如果我们的自定义的布局也是FrameLayout的时候那么可以使用merge替换
- 当应用include或者viewSub标签从外部导入xml结构时，可以将被导入的xml用merge作为根节点表示，这样当被嵌入父级结构中后可以很好的将它所包含的子集融合到父级结构中，而不会出现冗余的节点
- <merge>只能作为xml布局的根元素
- 通过<ViewStub>实现View的延迟加载

#### android：layout_gravity和android：gravity的区别？

- 第一个是让该布局在其父控件中的布局方式
- 第二个是该布局其字对象的布局方式

##### 关于LinearLayout的权重算法

- 布局大小=剩余空间大小权重所占比例+设定的宽度

#### ScrollView嵌套ListView方式除了测量还有什么方法

- 1、手动设置ListView高度：经过测试发现，在xml中直接指定ListView的高度，是可以解决这个问题的，但是ListView中的数据是可变的，实际高度还需要实际测量
- 2、使用单个ListView取代ScrollView中所有内容：如果满足头布局和脚布局的UI设计，直接使用ListView替代ScrollView
- 3、使用LinearLayout取代ListView：既然ListView不能适应ScrollView，那就换一个可以适应ScrollView的控件，如果想继续使用已经定义好的Adapter，只需要自定义一个类继承自LinearLayout，为其加上对BaseAdapter的适配
- 自定义可适应ScrollView的ListView：这个方法和上面的方法是异曲同工的
- 如果一定要使用ListView，只能自定义一个类继承自ListView，通过重写其onMeasure达到对ScrollView适配的效果

#### dp和px之间的关系

- dp：是dip的简写，指密度无关的像素，指一个抽象意义上的像素，程序用它来定义界面元素。一个与密度无关的，在逻辑尺寸上，与一个位于像素密度为160dpi的屏幕上的像素是一致的。
- 要把密度无关像素转换为屏幕像素，可以用这样的一个简单公式：pixels = dips*(density/160)
```
/**
* 根据手机的分辨率从 px(像素) 的单位 转成为 dp */
public static int px2dip(Context context, float pxValue) {
final float scale = context.getResources().getDisplayMetrics().density; return (int) (pxValue / scale + 0.5f);
}
/**
* 根据手机的分辨率从 dip 的单位 转成为 px(像素) */
public static int dip2px(Context context, float dpValue) {
final float scale = context.getResources().getDisplayMetrics().density; return (int) (dpValue * scale + 0.5f);
}
```

#### 什么是屏幕尺寸、屏幕分辨率、屏幕像素密度

- 屏幕尺寸是指屏幕对角线的长度
- 屏幕分辨率是指在横纵向上的像素点数，单位是px
- 屏幕像素密度是指每英寸上的像素点数，单位是dpi

#### Android样式和主题

- 样式(styles)：Android允许在外部样式文件中定义Android应用程序的Look和Feel，你可以将定义好的样式应用在不同的视图Views上。你可以在XML文件中定义样式，并将这些样式运用到不同的组件上。使用XML这种方式定义样式，只需要配置一些通用的属性，以后如果需要修改样式，可以集中修改
- 属性(Attribute)：你可以将单个属性应用到Android样式上，通常会在自定义View的时候，自定义属性
- 主题(Theme)：主题相比单个视图而言，是应用到整个Activity或者Application的样式

#### 如何将Activity中的Window的背景图设置为空

- getWindow().setBackgroundDrawbe(null)
- Android的默认背景不为空

### 布局适配

- 在Android中有4种普遍尺寸：小Small、普通Normal、大Large、超大XLarge
- 常见的普遍分辨率：低精度ldpi、中精度mdpi、高精度hdpi、超高精度xhdpi、1080p xxhdpi

#### 基本设置

- 在Manifest中添加子元素
- 在android：anyDensity="true"时，应用程序安装在不同密度的终端上时，程序会分别加载xxhdpi、xhdpi、hdpi、mdpi、ldpi文件夹中的资源
- 相反，如果设置为false，即便在文件夹下拥有相同资源，应用不会自动地区相应文件夹下寻找资源

#### 适配方案

- 1、使用wrap_content、math_parent、weight wrap_content：根据控件的内容设置控件的尺寸
- 2、使用相对布局，禁用绝对布局
- 3、创建不同的layout：每一种layout需要保存在相应的资源目录中，目录以-为后缀命名。例如：对大尺寸屏幕，一个唯一的layout文件应该保存在res/layout-large中
- 4、使用9-patch PNG图片：当我们需要使图片在拉伸后还能保持一定的显示效果，比如：不能使图片中的重要像素拉伸，不能使内容区域受到拉伸的影响，我们可以使用.9.png图来实现

### Android Drawable

- Drawable属于轻量级的、使用也很简单，Android把可绘制的对象抽象为Drawable，不同的图形图像资源代表着不同的drawable类型，在实际开发过程中使用@drawable来使用drawable资源

#### Android5.0新特性-使用SVG图片资源

- SVG的全程是Scalable Vector Graphics，叫可缩放矢量图形。它和位图(Bitmap)相对，SVG不会像位图一样因为缩放而让图片质量下降

##### 优点

- 图片的完美适配：SVG图像在放大或改变尺寸的情况下其图形质量不会有所损失。这样我们大大减少了适配所需要的多种分辨率图片，而且能够让图片完美适配多种分辨率，减少了APK包大小并提升了用户体验
- 尺寸的减少：SVG是使用XML文件来描述的，这种文本格式的图片尺寸很小，而且便于修改
- 设计上的轻便：在设计方面我们可以任意修改SVG图片的颜色，这对于某些情况下需要同一张图像但不同的颜色图片时非常方便，只需要修改fill颜色就可以了

### View初步了解

#### View是什么

- 简单来说：View是Android系统在屏幕上的视觉呈现，也就是说你在手机屏幕上看到的东西都是View

#### View是如何绘制出来的

- View的绘制流程是从ViewRoot的performTraversal()方法开始，依次经过measure()、layout()和draw()三个过程才最终将一个View绘制出来

#### View是怎么呈现在界面上的

- Android中的视图都是通过Window来呈现的，不管Activity、Dialog还是Toast它们都有一个Window，然后通过WindowManager来管理View。Window和顶级View--DecorView的通信时依赖ViewRoot完成的

#### 关于Android View控件的理解

- Android中控件大致被分为两类ViewGroup和View
- ViewGroup作为容器管理View
- Android视图是类似于Dom树的架构
- 父视图负责测量定位绘制等操作，我们经常在用的findViewById方法代价昂贵的原因，就是因为它负责自上而下遍历整棵控件树，来寻找View实例，在重复操作中尽量少用。
- 现在在用的很多控件都是直接或者间接继承自View的

#### View和ViewGroup什么区别

- Android的UI界面都是由View和ViewGroup及其派生类组合而成的
- 其中，View是所有UI组件的基类
- ViewGroup是容纳这些组件的容器，其本身就是从View派生出来的
- 需要注意的是嵌套次数最好不要超过10层，否则会降低效率

#### Android View刷新机制

- 在Android的布局体系中，父View负责刷新，布局显示子View，而当子View需要刷新时，则是通知父View来完成

#### RelativeLayout和LinearLayout性能比较

- RelativeLayout会让子View调用两次onMeasure，LinearLayout在有weight时，也会调用子View2次onMeasure
- RelativeLayout的子View如果高度和RelativeLayout不同，则会引发效率问题，当子View很复杂时，这个问题会更加严重。如果可以，尽量使用padding代替margin
- 在不影响层级深度的情况下，使用LinearLayout和FrameLayout而不是RelativeLayout

#### Android UI界面架构理解

- 每个Activity，Dialog，Toast都包含一个PhoneWindow对象，PhoneWindow设置DecorView为应用窗口的根视图