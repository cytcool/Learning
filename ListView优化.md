# ListView优化

- 在adapter的getView()方法中尽量少用逻辑
- 尽最大可能避免GC
- 滑动的时候不加载图片
- 将ListView的scrollingCache和animateCache设置为false
- item的布局层级越少越好，避免引用半透明元素
- 使用ViewHolder

## 在adapter中的getView方法中尽量使用逻辑

### 优化前的getView()


```
@Override
public View getView(int position, View convertView, ViewGroup paramViewGroup) {
        Object current_event = mObjects.get(position);
        ViewHolder holder = null;
        if (convertView == null) {
                holder = new ViewHolder();
                convertView = inflater.inflate(R.layout.row_event, null);
                holder.ThreeDimension = (ImageView) convertView.findViewById(R.id.ThreeDim);
                holder.EventPoster = (ImageView) convertView.findViewById(R.id.EventPoster);
                convertView.setTag(holder);

        } else {
                holder = (ViewHolder) convertView.getTag();
        }

       //在这里进行逻辑判断，这是有问题的 
        if (doesSomeComplexChecking()) {
                holder.ThreeDimention.setVisibility(View.VISIBLE);
        } else {
                holder.ThreeDimention.setVisibility(View.GONE); 
        }

        // 这是设置image的参数，每次getView方法执行时都会执行这段代码，这显然是有问题的
        RelativeLayout.LayoutParams imageParams = new RelativeLayout.LayoutParams(measuredwidth, rowHeight);
        holder.EventPoster.setLayoutParams(imageParams);

        return convertView;
}
```

### 优化后的getView()


```
@Override
public View getView(int position, View convertView, ViewGroup paramViewGroup) {
    Object object = mObjects.get(position);
    ViewHolder holder = null;

    if (convertView == null) {
            holder = new ViewHolder();
            convertView = inflater.inflate(R.layout.row_event, null);
            holder.ThreeDimension = (ImageView) convertView.findViewById(R.id.ThreeDim);
            holder.EventPoster = (ImageView) convertView.findViewById(R.id.EventPoster);
            //设置参数提到这里，只有第一次的时候会执行，之后会复用 
            RelativeLayout.LayoutParams imageParams = new RelativeLayout.LayoutParams(measuredwidth, rowHeight);
            holder.EventPoster.setLayoutParams(imageParams);
            convertView.setTag(holder);
    } else {
            holder = (ViewHolder) convertView.getTag();
    }

    // 我们直接通过对象的getter方法代替刚才那些逻辑判断，那些逻辑判断放到别的地方去执行了
    holder.ThreeDimension.setVisibility(object.getVisibility());

    return convertView;
}
```

## GC垃圾回收器

- 当你创建了大量对象的时候，GC就会频繁的执行，所以在getVIew()方法中不要创建很多的对象
- 最好的优化是不要再ViewHolder以外创建任何对象

### 如果频发出现“GC has freed somememory”，请检查

- item布局的层级是否太深
- getView()方法中是否有大量对象存在
- ListView的布局属性

## 加载图片

- 如果ListView中需要显示从网络上下载的图片，不要再ListView，这会使ListView变得卡段，所以我们需要在监听器里面监听ListView的状态，如果滑动的时候，停止加载图片，如果没有滑动，则开始加载图片


```
listView.setOnScrollListener(new OnScrollListener() {

            @Override
            public void onScrollStateChanged(AbsListView listView, int scrollState) {
                    //停止加载图片 
                    if (scrollState == AbsListView.OnScrollListener.SCROLL_STATE_FLING) {
                            imageLoader.stopProcessingQueue();
                    } else {
                    //开始加载图片
                            imageLoader.startProcessingQueue();
                    }
            }

            @Override
            public void onScroll(AbsListView view, int firstVisibleItem, int visibleItemCount, int totalItemCount) {
                    // TODO Auto-generated method stub

            }
    });

```

## 将ListView的scrollingCache和animateCache设置为false

- **scrollingCache**: scrollingCache本质上是drawing cache，你可以让一个View将他自己的drawing保存在cache中（保存为一个bitmap），这样下次再显示View的时候就不用重画了，而是从cache中取出。默认情况下drawing cahce是禁用的，因为它太耗内存了，但是它确实比重画来的更加平滑。而在ListView中，scrollingCache是默认开启的，我们可以手动将它关闭。
- **animateCache**:ListView默认开启了animateCache，这会消耗大量的内存，因此会频繁调用GC，我们可以手动将它关闭掉


```
<ListView
        android:id="@android:id/list"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:scrollingCache="false"
        android:animationCache="false"
        android:visibility="gone" />
```

## ListView的四种优化

### convertView的复用（优化加载布局）

- 在Adapter类的getView方法中通过判断convertView是否为null，是的话就需要在创建一个视图出来，然后给视图设置数据，最后将这个视图返回给底层，呈现给用户；如果不为null的话，其他新的view可以通过复用的方式使用已经消失的条目view，重新设置上数据然后展现出来。


```
@Override
public View getView(int position, View convertView, ViewGroup parent) {
    if (convertView == null) {

//如果当前的convertView为null，则通过inflate产生一个view

    convertView = View.inflate(context, R.layout.layout_pic_item,null);
    }

    TextView tvDis = (TextView) convertView.findViewById(R.id.tv_item_picture_desc);

    tvDis.setText("设置数据");

    return convertView;
}
```

### ViewHolder的使用（优化加载控件）


- contverView的缺点：就是每次在findviewById，重新找到控件，然后对控件进行赋值，这样会减慢加载的速度
- 创建一个**内部类ViewHolder**，里面的成员变量和view中所包含的组件个数、类型相同，在convertview为null的时候，把findviewbyId找到的控件赋给ViewHolder中对应的变量，就相当于先把它们装进一个容器，下次要用的时候，直接从容器中获取


```
@Override
    public View getView(int position, View convertView, ViewGroup parent) {
        ViewHolder holder;
        View itemView = null;
        if (convertView == null) {
            itemView = View.inflate(context, R.layout.item_news_data, null);
            holder = new ViewHolder(itemView);
            //用setTag的方法把ViewHolder与convertView "绑定"在一起
            itemView.setTag(holder);
        } else {

    //当不为null时，我们让itemView=converView，用getTag方法取出这个itemView对应的holder对象，就可以获取这个itemView对象中的组件

            itemView = convertView;
            holder = (ViewHolder) itemView.getTag();
        }

        return itemView;
    }

}

public class ViewHolder {
    @ViewInject(R.id.iv_item_news_icon)
    private ImageView ivNewsIcon;// 新闻图片
    @ViewInject(R.id.tv_item_news_title)
    private TextView tvNewsTitle;// 新闻标题
   
    public ViewHolder(View itemView) {
        ViewUtils.inject(this, itemView);
    }
}
```
### 使用分段加载 

- 有些情况下我们需要加载网络中的数据，显示到ListView，而往往此时都是数据量比较多的一种情况，如果数据有1000条，没有优化过的ListView都是会一次性把数据全部加载出来的，很显然需要一段时间才能加载出来，我们不可能让用户面对着空白的屏幕等好几分钟
- 那么这时我们可以使用分段加载，比如先设置每次加载数据10条，当用户滑动ListView到底部的时候，我们再加载20条数据出来，然后使用Adapter刷新ListView
- 这样用户只需要等待10条数据的加载时间，这样也可以缓解一次性加载大量数据而导致OOM崩溃的情况。

### 使用分页加载

- 比如说我们将这10万条数据分为1000页，每一页100条数据，每一页加载时都覆盖掉上一页中List集合中的内容，然后每一页内再使用分批加载，这样用户的体验就会相对好一些。


## RecycleBin机制

### 基本原理

- **ActiveView**：ListView中有许多View呈现在UI上，这些View对我们来说是可见的，这些可见的View可以称作OnScreen的View，即在屏幕中能看到的View，也可以叫做ActiveView，因为它们是在UI上可操作的。
- **ScrapView**：当触摸ListView并向上滑动时，ListView上部的一些OnScreen的View位置上移，并移除了ListView的屏幕范围，此时这些OnScreen的View就变得不可见了，不可见的View叫做OffScreen的View，即这些View已经不在屏幕可见范围内了，也可以叫做ScrapView
- ListView会把那些ScrapView（即OffScreen的View）删除，这样就不用绘制这些本来就不可见的View了，同时，ListView会把这些删除的ScrapView放入到RecycleBin中存起来，就像把暂时无用的资源放到回收站一样。
- 当ListView的底部需要显示新的View的时候，会从RecycleBin中取出一个ScrapView，将其作为convertView参数传递给Adapter的getView方法，从而达到View复用的目的，这样就不必在Adapter的getView方法中执行LayoutInflater.inflate()方法了。

## 其他优化方式

### 1、ViewHolder   Tag 必不可少，这个不多说！

### 2、如果自定义Item中有涉及到图片，一定要处理图片

- 不要直接拿个路径就去循环decodeFile();这是找死….用Option保存图片大小、不要加载图片到内存去
- 拿到的图片一定要经过边界压缩
- 在ListView中取图片时也不要直接拿个路径去取图片，而是以**WeakReference**（使用WeakReference代替强引用。比如可以使用WeakReference<Context> mContextRef）、SoftReference、WeakHashMap等的来存储图片信息，是图片信息不是图片哦！
- 尽量避免在BaseAdapter中使用**stati**c来定义全局静态变量，static是Java中的一个关键字，当用它来修饰成员变量时，那么该变量就属于该类，而不是该类的实例用static修饰的变量，它的生命周期是很长的，如果用它来引用一些资源耗费过多的实例（比如Context的情况最多），这时就要尽量避免使用了
- 如果为了满足需求下必须使用Context的话：Context尽量使用**Application Context**，因为Application的Context的**生命周期比较长**，引用它不会出现内存泄露的问题
- 尽量避免在ListView适配器中使用线程，因为线程产生内存泄露的主要原因在于线程生命周期的不可控制
- 使用的自定义ListView中适配数据时使用**AsyncTask**自行开启线程的，这个比用Thread更危险，因为Thread只有在run函数不结束时才出现这种内存泄露问题，然而AsyncTask内部的实现机制是运用了线程执行池，这个类产生的Thread对象的生命周期是不确定的，是应用程序无法控制的，因此如果AsyncTask作为Activity的内部类，就更容易出现内存泄露的问题