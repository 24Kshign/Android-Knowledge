## ConstraintLayout最强布局解析

`ConstraintLayout`布局出来已经很久了，刚出来那会儿就想尝试一下的，结果半天都没适应，前两天看到一篇`ConstraintLayout`实战的文章，看完之后发现这布局贼鸡儿好用啊，日常开发中的大多数布局使用它都可以完成，而且也不需要嵌套。

### 介绍

`ConstraintLayout`又称约束布局，是谷歌在2016年开发者大会上推出的，之后在`Android Studio`上成为了默认布局，该布局能减少布局的层级嵌套，我们都知道，`View`嵌套的越多，在加载的过程中解析起来就越费时间，该布局几乎能做到`LinearLayout`和`RelativeLayout`嵌套完成的任何布局，下面跟着一波小`demo`来深入了解谷歌推荐的`ConstraintLayout`。

### 用法简介

#### 1、xxx_toyyyOf属性

`xxx`是当前的控件，`yyy`是指定控件，这个指定控件也可以是容器本身（parent）

`ConstraintLayout`中有以下多种这样的属性：

- **layout_constraintLeft_toLeftOf**

- **layout_constraintLeft_toRightOf**

- **layout_constraintRight_toLeftOf**

- **layout_constraintRight_toRightOf**

- **layout_constraintTop_toTopOf**

- **layout_constraintTop_toBottomOf**

- **layout_constraintBottom_toTopOf**

- **layout_constraintBottom_toBottomOf**

- **layout_constraintBaseline_toBaselineOf**

- **layout_constraintStart_toEndOf**

- **layout_constraintStart_toStartOf**

- **layout_constraintEnd_toStartOf**

- **layout_constraintEnd_toEndOf**

![](http://ooaap25kv.bkt.clouddn.com/18-9-3/98495697.jpg)

参照上图给出的解释，以上属性都可以这样用，有点类似`RelativeLayout`中的`toLeftOf`，`toRightOf`，上面的属性中还有一个关于`Baseline`的，我们通过另外一张图来了解一下：

![](http://ooaap25kv.bkt.clouddn.com/18-9-4/78875139.jpg)


`Baseline`是控件中文字的基准线，这里可以理解为参照某个控件中的文字底部对齐，来看看样式：

![](http://ooaap25kv.bkt.clouddn.com/18-9-3/75559787.jpg)

如果不加基准线对齐的话，那么`ButtonA`的位置就在容器的左上角。

#### 2、设置margin边距

边距，和传统的布局是一样的用法，但是这里要注意的是，必须要设置自己的相对位置（先要指定自己在容器中的位置，可以是相对容器的，也可以是相对某个控件的），如果不设置的话，那么设置`margin`是无效的，大家可以试试，在一个`ConstraintLayout`布局中放一个按钮，除了边距之外什么都不设置，这样是没有效果的，因为你没有在布局中给它设置相对位置。

#### 3、隐藏空间设置边距

ConstraintLayout中有以下多种这样的属性：

- **layout_goneMarginStart**

- **layout_goneMarginEnd**

- **layout_goneMarginLeft**

- **layout_goneMarginTop**

- **layout_goneMarginRight**

- **layout_goneMarginBottom**

平常我们在开发过程中会遇到这样一个问题，当一个控件被设置成`gone`之后，与之关联的控件的位置常常也会发生改变，来看看样式：

![](http://ooaap25kv.bkt.clouddn.com/18-9-3/35375631.jpg)

平常我们写标题栏的时候应该都遇到过右边放两个按钮的情况，而且是可以控制显示隐藏的，当最右边的按钮隐藏之后，左边的按钮也要距离右边有一个边距，这种情况下我们就可以使用上面这些属性来配置布局。

#### 4、居中和偏移

- 水平居中

```
app:layout_constraintLeft_toLeftOf="parent"
app:layout_constraintRight_toRightOf="parent"
```

![](http://ooaap25kv.bkt.clouddn.com/18-9-4/83163284.jpg)

这个很好理解，设置与容器的左边和右边分别对齐，这样的话就能让控件水平居中了，同理垂直居中和中心对齐也是这样。可能有人会吐槽了，写这么多还不如我用`LinearLayout`和`RelativeLayout`写一句代码来的快呢？老铁还真是个急性子，接着往下看。

- 垂直居中

```
app:layout_constraintTop_toTopOf="parent"
app:layout_constraintBottom_toBottomOf="parent"
```

- 中心

```
app:layout_constraintLeft_toLeftOf="parent"
app:layout_constraintRight_toRightOf="parent"
app:layout_constraintTop_toTopOf="parent"
app:layout_constraintBottom_toBottomOf="parent"
```

下面来实现一个这样的功能，让一个控件在距离左边和距离右边的比例是1:3，来看看正经写法：

![](http://ooaap25kv.bkt.clouddn.com/18-9-4/23123049.jpg)


```
<LinearLayout>

	<View
	    android:layout_width="0dp"
	    android:layout_height="match_parent"
	    android:layout_weight="1" />
	    
	<Button
	    android:layout_width="100dp"
	    android:layout_height="match_parent" />
	
	<View
	    android:layout_width="0dp"
	    android:layout_height="match_parent"
	    android:layout_weight="3" />

</LinearLayout>
```
相信大多数老铁都会这么写，那么我们现在来看看不正经的写法：

```
<android.support.constraint.ConstraintLayout>

	<Button 
	    android:id="@+id/button"
	    app:layout_constraintHorizontal_bias="0.33"
	    app:layout_constraintLeft_toLeftOf="parent"
	    app:layout_constraintRight_toRightOf="parent />
	    
</android.support.constraint.ConstraintLayout>
```
看到这个是不是惊呆了，一行代码就搞定了，简直不要太简洁。

#### 5、CircleRadius角度定位（在版本1.1中加入）

![](http://ooaap25kv.bkt.clouddn.com/18-9-4/42338153.jpg)
![](http://ooaap25kv.bkt.clouddn.com/18-9-4/92041542.jpg)

官网给出的解释是，你可以以角度和距离约束窗口小部件中心相对于另一个窗口小部件中心。可能有些人看不太懂，我也没看懂（哈哈，LZ你是来搞笑的吗），但是看官网给出的图我大概明白是什么意思了，简单来说就是可以根据两个控件的中心来形成约束关系，然后可以通过设置角度来控制这个约束关系（还看不懂的话那就来实践一把）

![](http://ooaap25kv.bkt.clouddn.com/18-9-3/6881503.jpg)

`layout_constraintCircle`表示相对某一指定控件，上图中表示相对`ButtonA`；`layout_constraintCircleRadius`表示该控件的中心点到指定控件的中心点的距离（两点之间，直线最短）；`layout_constraintCircleAngle`表示该控件的中心点在指定控件垂直方向上的角度（范围是0-360），具体看上图中标示的。

#### 6、尺寸约束

在`ConstraintLayout`布局中，你可以设置布局的最大和最小尺寸，而且你可通过三种方式来设置控件的大小：

- **特定数值，比如123dp**

- **使用`wrap_content`，控件将自己计算大小**

- **使用0dp，相当于`MATCH_CONSTRAINT`**

**注意：`match_parent`官方不建议在`ConstraintLayout`布局中使用，可以通过设置`MATCH_CONSTRAINT`（真实数值是0dp）配合约束来定义布局**

下面我们来看一个例子：

![](http://ooaap25kv.bkt.clouddn.com/18-9-3/97692654.jpg)

`ButtonA`是固定宽度且靠左，给`ButtonB`设置了约束，刚开始预期的是设置`ButtonB`的宽度慢慢增大，超过`ButtonA`之后不管设置多大都像`ButtonC`和`ButtonD`一样，但是`ButtonA`却把`ButtonB`覆盖了，显然这不是我们需要的，这时候`MATCH_CONSTRAINT`的作用就能体现出来了，怎么理解这个`MATCH_CONSTRAINT`，我们可以理解成为了配合约束布局而代替了`match_parent`

#### 7、设置宽高比例

在使用百分比布局时，有两种形式可以设置：

- `layout_constraintDimensionRatio`，给宽或者高其中一个设置为0dp，然后设置该属性是一个比例，宽和高的比（相对那个已知长度的），可能说的有点绕，下面我们直接看demo：

![](http://ooaap25kv.bkt.clouddn.com/18-9-3/20926220.jpg)

- `app:layout_constraintDimensionRatio`，设置宽和高都为0dp，然后设置改属性的值为`H,x:y` 或者 `W,x:y`，看一下demo

![](http://ooaap25kv.bkt.clouddn.com/18-9-3/3306578.jpg)

#### 8、Chains（链）

链条在同一方向上（水平或者垂直）为一组互相关联的控件作统一管理，并且链由链头（链的第一个元素）设置的属性控制，链头是水平链的最左侧的元素，是垂直链的最顶部的元素。我们来看看一些链的样式：

![](http://ooaap25kv.bkt.clouddn.com/18-9-4/4038136.jpg)

我们只需要设置`layout_constraintHorizontal_chainStyle`或者`layout_constraintVertical_chainStyle`的属性就好了，而这个属性有以下几种配置：

- **`CHAIN_SPREAD`模式：元素将展开（默认样式）**

- **加权链`CHAIN_SPREAD`模式：如果给元素的宽或者高设置了`MATCH_CONSTRAINT`(0dp)，它们将分割宽高方向上的可用空间**

- **`CHAIN_SPREAD_INSIDE`模式：类似于SPREAD，但链的端点不会分散**

- **`CHAIN_PACKED`模式：链条的元素将被捆绑在一起。然后，子项的水平或垂直偏差属性将影响该链元素的定位**

#### 9、辅助布局Guildline

这是`ConstraintLayout`布局特有的功能，你可以用它来辅助你完成布局，类似于高中数学图形学中的辅助线，只不过这条辅助线只有两个方向，水平和垂直：

- **当设置线的方向为`horizontal`时，辅助线的高度为0，宽度是容器的宽度。**

- **当设置线的方向为`vertical`时，辅助线的宽度为0，高度时容器的高度。**

我们来看看`Guildline`的样式（需要注意的是，辅助线是不可见的，只有在预览的时候才能通过鼠标选中可见）：

![](http://ooaap25kv.bkt.clouddn.com/18-9-4/18069466.jpg)

`Guildline`有三个常用的属性，用来控制辅助线的位置：

- **`layout_constraintGuide_begin`：指定辅助线距离左边或者顶部的距离**

- **`layout_constraintGuide_end`：指定辅助线距离右边或者底部的距离**

- **`layout_constraintGuide_percent`：指定在父控件中的宽度或高度的百分比**



### 代码

以上demo的代码全都上传至[ConstraintLayout最强布局解析](https://github.com/24Kshign/ConstraintLayoutTest)，有兴趣的童鞋可以自行下载看看


### 参考

[官网
ConstraintLayout讲解](https://developer.android.com/reference/android/support/constraint/ConstraintLayout)

[泓洋_ConstraintLayout 完全解析](https://blog.csdn.net/lmj623565791/article/details/78011599?utm_source=tuicool&utm_medium=referral)

[实战篇ConstraintLayout的崛起之路](https://www.jianshu.com/p/a74557359882)
