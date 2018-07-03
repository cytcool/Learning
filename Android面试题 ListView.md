## Android面试题 ListView

### 基础问题

#### 1、当ListView数据集改变后，如何更新ListView

- 使用该ListView的adapter的notifyDataSetChanged()方法，该方法会使ListView重新绘制

#### 2、ListView如何实现分页加载

- 1、设置ListView的滚动监听器：setOnScrollListener()；在监听器中有两个方法：滚动状态发生改变的方法onScrollStateChanged()和ListView被滚动时调用的方法onScroll()
- 2、在滚动状态发生改变的方法中，有三种状态：手指按下移动的状态：SCROLL_STATE_TOUCH_SCROLL(触摸滑动);惯性滑动：SCROLL_STATE_FLING(滑翔静止状态);静止状态：SCROLL_STATE_IDLE(静止);

#### 3、ListView如何显示多种类型的Item

- ListView显示的每个条目都是通过baseAdapter的getView(int position,View convertView,ViewGroup parent)来展示的，理论上我们完全可以让每个条目都是不同类型的View
- adapter提供了getViewTypeCount()和getItemViewType(int position)两个方法。在getView方法中可以根据不同的viewtype加载不同的布局文件

#### 4、ListView如何定位到指定位置

- 可以通过ListView提供的lv.setSelection(ListView.getPosition())方法

#### 5、在ListView中设置Selector为null会报空指针

- mListView.setSelector(null)//空指针
- mListView.setSelector(new ColorDrawable(Color.TRANSPARENT));

#### 6、ListView中主要由两种监听器

- OnItemClickListener：处理视图中单个条目的点击事件
- OnScrollListener：监听滚动变化，可以用于视图滚动中加载数据

#### 7、ListView中图片错位问题

- 每次getView能给对象一个标识，在异步加载完成时比较标识与当前行item的标识是否一致，一致则显示，否则不做处理即可

```
// 给 ImageView 设置一个 tag
holder.img.setTag(imgUrl);
// 预设一个图片
holder.img.setImageResource(R.drawable.ic_launcher);

// 通过 tag 来防止图片错位
if (imageView.getTag() != null &&imageView.getTag().equals(imageUrl)) {
imageView.setImageBitmap(result);
}
```

#### 8、ListView中多线程更新问题

- 使用Handler负责统一刷新
- ListView带有进度条的每个Item，多个子线程刷新各自的进度，如果子线程很多那么刷新就会变得很频繁，我们可以由一个Handler负责统一刷新，这样我们就要以增加一些额外条件限制刷新的次数和条件

### ListView优化问题

#### 1、ListView如何提高其效率？

- 当convertView为空时，用setTag()方法为每个View绑定一个存放控件的ViewHolder对象
- 当convertView不为空时，重复利用已经创建的View的时候，使用getTag()方法获取绑定的ViewHolder对象，这就避免了findViewById对控件的层层查询，而是快速定位到控件
- 复用convertView，使用历史的View，提升效率200%
- 自定义静态类ViewHolder，减少findViewById的次数，提升效率50%
- 异步加载数据，分页加载数据
- 使用WeakReference引用ImageView对象

#### 2、ListView中加载优化图片

##### (1)处理图片的方式

- 如果ListView中定义的Item中有涉及到大量图片的，一定要对图片进行处理，因为图片占的内存是ListView中最头疼的

###### 方法

- 1、不要直接拿路径去循环BitmapFactory.decodeFile；使用Options保存图片大小；不要加载图片到内存
- 2、对图片一定要经过边界压缩尤其是比较大的图片；如果图片时后台服务器处理好的那就不需要了
- 3、在ListView中取图片时不要直接拿个路径去取图片，而是以WeakReference(使用WeakReference代替强引用)。比如可以使用WeakReference.mContextRef)、SoftReference、WeakHashMap等来存储图片信息
- 4、在getView中做图片转换时，产生的中间变量一定要及时释放

##### (2)异步加载图片基本思想

- 1、先从内存缓存中获取图片显示(内存缓冲)
- 2、获取不到的话从SD卡里获取(SD卡缓冲)
- 3、都获取不到的话从网络下载图片并保存到SD卡同时加入内存并显示

### 优化基本原理

- 1、先从内存中加载，没有则开启线程从SD卡或网络中获取，这里从SD卡获取图片时放在子线程里执行的，否则快速滑屏的话不够流畅
- 2、于此同时，在adpter里有个busy变量，表示ListView是否处于滑动状态，如果是滑动状态则仅从内存中获取图片，没有的话无需再开启线程去外村或网络获取图片
- 3、ImageLoader里的线程使用了线程池，从而避免了过多线程频繁创建和销毁，。如果每次总是new一个线程去执行这是非常不可取的，好一点的用AsyncTask类，其实内部也是用到了线程池。在从网络获取图片时，先是将其保存到SD卡，然后再加载到内存，这么做得好处是在加载到内存时可以做个岩梭处理，以减少图片所在内存

#### ListView实现Item局部刷新

```
private void updateView(int itemIndex) {
          //得到第一个可显示控件的位置，
          int visiblePosition = mListView.getFirstVisiblePosition();
          //只有当要更新的view在可见的位置时才更新，不可见时，跳过不更新
          if (itemIndex - visiblePosition >= 0) {
              //得到要更新的item的view
            View view = mListView.getChildAt(itemIndex - visiblePosition);
              //调用adapter更新界面
             mAdapter.updateView(view, itemIndex);
        }
}
```

### ListView的原理

#### 用MVC思路来理解ListView的三个重要成员

- ListView：用来展示列表的View(V)
- Adapter：用来把数据映射到ListView上(C)
- 数据：Item显示的数据(M)


- ListView针对每个Item，要求adapter"返回一个视图"(getView)，也就是说ListView在开始绘制的时候，系统首先调用getCount()函数，根据他的返回值得到ListView的长度，然后根据这个长度，调用getView()一行一行的绘制ListView的每一项。如果getCount()的返回值是0的话，列表一行都不会显示，如果返回1，就只显示一行，如果我们有更多的item为每个Item创建一个新的VIew是不可能的，实际上Android早已经缓存了这些视图



