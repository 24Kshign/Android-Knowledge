## Android开发人员不得不学习的CSS3基础

`CSS3` 是 `CSS` 的升级版，和 `CSS` 一样都是控制网页的样式和布局的，但是新增了很多种属性便于我们更好的给 `Html` 元素设置样式。

### 边框

用 `CSS3`，你可以创建圆角边框，添加阴影框，边框有以下几个属性：

- border-radius

- box-shadow

- border-image

**1、CSS3的圆角边框**

```
div {
	// 语法
	// border-radius:top-left top-right bottom-right bottom-left;
	
	border:2px solid;
	border-radius:25px;
}
```

`border-radius` 中有四个值，分别是左上角，右上角，右下角，左下角；你也可以单独设置：

|     值     |    说明     |
| ----------|:-----------:|
|  border-top-left-radius|   定义了左上角的弧度   |
|  border-top-right-radius|   定义了右上角的弧度   |
|  border-bottom-right-radius|   定义了右下角的弧度   |
|  border-bottom-left-radius|   定义了左下角的弧度   |


**2、CSS3阴影**

```
// 语法：
// box-shadow: h-shadow v-shadow blur spread color inset;

div {
	box-shadow: 10px 10px 5px #888888;
}
```

你可以通过设置 `box-shadow` 属性来给元素设置阴影，以下是一些属性值：

|     值     |    说明     |
| ----------|:-----------:|
| h-shadow  |  必需的。水平阴影的位置。允许负值   |
| v-shadow  |  必需的。垂直阴影的位置。允许负值   |
|   blur    |  可选。模糊距离   |
|  spread   |  可选。阴影的大小  |
|   color   |  可选。阴影的颜色。   |
|   inset   |  可选。从外层的阴影（开始时）改变阴影内侧阴影   |

**3、CSS3边界图片**

```
div {
	// 语法：
	// border-image: source slice width outset repeat|initial|inherit;
	
	border-image:url(border.png) 30 30 round;
	-webkit-border-image:url(border.png) 30 30 round; /* Safari 5 and older */
	-o-border-image:url(border.png) 30 30 round; /* Opera */
}
```

你还可以用图片给元素添加边框，需要注意的是，这个要适配不同的浏览器：

|     值     |    说明     |
| ----------|:-----------:|
|  source   |   用于指定要用于绘制边框的图像的位置   |
|  slice   |   图像边界向内偏移   |
|  width   |   图像边界的宽度   |
|  outset   |   用于指定在边框外部绘制 border-image-area 的量   |
|  repeat   |   用于设置图像边界是否应重复（repeat）、拉伸（stretch）或铺满（round）。   |

### 过渡

在 `CSS3`中，我们为了给某个元素从某种样式转变到另一种样式添加效果，可以使用 `transition`属性即可：

```
div {
    transition-property: width;
    transition-duration: 1s;
    transition-timing-function: linear;
    transition-delay: 2s;
    /* Safari */
    -webkit-transition-property:width;
    -webkit-transition-duration:1s;
    -webkit-transition-timing-function:linear;
    -webkit-transition-delay:2s;
}
```
这里同样需要做适配，还可以简写，把所有的属性写在一起，类似于：

```
div {
	transition: width 1s linear 2s;
	/* Safari */
    -webkit-transition:width 1s linear 2s;
}
```

|     值     |    说明     |
| ----------|:-----------:|
|  transition-property   |   规定应用过渡的 CSS 属性的名称。   |
|  transition-duration   |   定义过渡效果花费的时间。默认是 0。   |
|  transition-timing-function   |   规定过渡效果的时间曲线。默认是 "ease"。   |
|  transition-delay   |   规定过渡效果何时开始。默认是 0。   |

### 弹性盒子

是一种当页面需要适应不同的屏幕大小以及设备类型时确保元素拥有恰当的行为的布局方式。引入弹性盒布局模型的目的是提供一种更加有效的方式来对一个容器中的子元素进行排列、对齐和分配空白空间。

```
div {
	display: flex;  //将该div内部设置成弹性布局
	flex-diretion: row; //设置弹性布局的方向是横向的
}
```

弹性布局的方向有以下四种：

- row：横向从左到右排列（左对齐），默认的排列方式。

- row-reverse：反转横向排列（右对齐，从后往前排，最后一项排在最前面。

- column：纵向排列。

- column-reverse：反转纵向排列，从后往前排，最后一项排在最上面。

除此之外，弹性布局还有两种居中的方式：

**1、justify-content**

`justify-content` 内容对齐，把弹性项沿着弹性容器的主轴线（main axis）对齐。

```
justify-content: flex-start | flex-end | center | space-between | space-around
```

- **flex-start：**
弹性项目向行头紧挨着填充。这个是默认值。第一个弹性项的main-start外边距边线被放置在该行的main-start边线，而后续弹性项依次平齐摆放。

- **flex-end：**
弹性项目向行尾紧挨着填充。第一个弹性项的main-end外边距边线被放置在该行的main-end边线，而后续弹性项依次平齐摆放。

- **center：**
弹性项目居中紧挨着填充。（如果剩余的自由空间是负的，则弹性项目将在两个方向上同时溢出）。

- **space-between：**
弹性项目平均分布在该行上。如果剩余空间为负或者只有一个弹性项，则该值等同于flex-start。否则，第1个弹性项的外边距和行的main-start边线对齐，而最后1个弹性项的外边距和行的main-end边线对齐，然后剩余的弹性项分布在该行上，相邻项目的间隔相等。

- **space-around：**
弹性项目平均分布在该行上，两边留有一半的间隔空间。如果剩余空间为负或者只有一个弹性项，则该值等同于center。否则，弹性项目沿该行分布，且彼此间隔相等（比如是20px），同时首尾两边和弹性容器之间留有一半的间隔（1/2*20px=10px）。

**2、align-items**

`align-items` 设置或检索弹性盒子元素在侧轴（纵轴）方向上的对齐方式。

```
align-items: flex-start | flex-end | center | baseline | stretch
```

- **flex-start：**
弹性盒子元素的侧轴（纵轴）起始位置的边界紧靠住该行的侧轴起始边界。

- **flex-end：**
弹性盒子元素的侧轴（纵轴）起始位置的边界紧靠住该行的侧轴结束边界。

- **center：**
弹性盒子元素在该行的侧轴（纵轴）上居中放置。（如果该行的尺寸小于弹性盒子元素的尺寸，则会向两个方向溢出相同的长度）。

- **baseline：**
如弹性盒子元素的行内轴与侧轴为同一条，则该值与'flex-start'等效。其它情况下，该值将参与基线对齐。

- **stretch：**
如果指定侧轴大小的属性值为'auto'，则其值会使项目的边距盒的尺寸尽可能接近所在行的尺寸，但同时会遵照'min/max-width/height'属性的限制。

以下是 `CSS3` 弹性盒子常用的属性：

|     值     |    说明     |
| ----------|:-----------:|
|   flex-direction  |   指定了弹性容器中子元素的排列方式  |
|   justify-content  |   设置弹性盒子元素在主轴（横轴）方向上的对齐方式。  |
|   align-items  |   设置弹性盒子元素在侧轴（纵轴）方向上的对齐方式。  |
|   flex-wrap  |   设置弹性盒子的子元素超出父容器时是否换行。  |
|   align-content  |   修改 flex-wrap 属性的行为，类似 align-items, 但不是设置子元素对齐，而是设置行对齐  |
|   flex-flow  |   flex-direction 和 flex-wrap 的简写  |
|   order  |   设置弹性盒子的子元素排列顺序。  |
|   align-self  |   在弹性子元素上使用。覆盖容器的 align-items 属性。  |
|   flex  |   设置弹性盒子的子元素如何分配空间。  |
