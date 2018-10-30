## Web实现手风琴效果

### 序言

今天给大家带来一个手风琴效果的图片，效果如下图所示：

![](http://ooaap25kv.bkt.clouddn.com/18-10-26/85041570.jpg)

### 实现

我们来简单分析一下，这里一共有两种效果，鼠标移动到某一列表项时，这个列表项会变大，里面的图片和文字也会变大，所以这个地方我们得写两套样式用来切换，然后就是监听鼠标的事件了，总体来说和之前写的 `tabBar` 功能也有点类似，下面来看看 `Html` 代码：

```
<div id="header" class="container">

	<ul>
		<li class="big">
			<a>
				<img src="http://gtms01.alicdn.com/tps/i1/T1Lvt3Fv4kXXbA5QAK-195-120.jpg_Q90.jpg" />
				<div class="description">
					<h3 style="color:#6f9400">聚美妆</h3>
					<p>聚美妆1/2周年庆</p>
					<p class="price"><strong>1</strong><i>折起</i></p>
				</div>
				<s class="line"></s>
				<i class="mask"></i>
			</a>
		</li>
	</ul>

</div>
```

这里我只是写了列表项中的其中一项，剩下的几项复制粘贴一下就好了，接下来看看CSS的代码：

```
.container {
    width: 938px;
    height: 128px;
    margin: 0 auto;
    border: 1px solid #cccccc;
}

.container li{
    width: 156px;
    height: 128px;
    float: left;
    overflow: hidden;
}

.container li a{
    width: 156px;
    height: 128px;
    position: relative;
    display: block;
    overflow: hidden;
    text-decoration: none;
}

.container li img{
    height: 72px;
    width: 117px;
    position: absolute;
    bottom: 0;
    right: -15px;
}

.description {
    width: 136px;
    padding: 4px 10px;
    position: absolute;
    top: 0;
    left: 0;
}

.description h3{
    font-size: 14px;
    font-weight: 700;
}

.description p{
    color:#868686;
    font-size: 12px;
    height: 22px;
    width: 136px;
    overflow: hidden;
    line-height: 22px;
}

.description .price {
    font-size: 14px;
    font-style: italic;
    color: #fa2a5d;
    height: 35px;
}

.container .line{
    position: absolute;
    right: 0;
    width: 0;
    border: 1px dashed #cacaca;
    height: 128px;
}

.container .mask{
    position: absolute;
    top: 0;
    left: 0;
    height: 128px;
    width: 156px;
    background-color: #000;
    opacity: 0;
}

.container:hover .mask {
    opacity: 0.15;
}


/* 以下是变大之后的 */

.container li.big, li.big a{
    width: 314px;
    height: 128px;
}

.container li.big img {
    width: 195px;
    height: 120px;
    right: 0;
    bottom: 0;
}

.container li.big .description {
    width: 290px;
}

.container li.big h3 {
    font-size: 18px;
}

.container li.big .description p {
    font-size: 14px;
    width: 166px;
}

.container li.big p.price {
    font-size: 16px;
    padding-top: 7px;
}

.container li.big a:hover .mask{
    opacity: 0;
}
.container li:hover{
    width: 312px;
}
```

这里用到了一个新的布局，`relative` 相对定位布局，这个相对是相对于父布局的，可以设置 `top`，`left`，`bottom`，`right`是个方向上相对父布局的距离，然后再给子布局设置绝对定位布局，这样的话就表示子布局只会跟着父布局改动而改动了。然后还有一个遮罩层的概念，就是给某个元素设置 `opacity` 属性，这个的取值是0-1，一个透明度的变化，最后再来定义一个 `big` 的样式，就算完成了；那么我们怎么给布局动态的添加 `class`属性呢，来看看 `Js` 的实现：

```
window.onload = function () {
    var outer = document.getElementById('header');
    var lis = outer.getElementsByTagName('li');
    if (null != lis) {
        for (var i=0; i<lis.length; i++) {
            lis[i].onmouseover = function () {
                for(var j=0;j<lis.length;j++) {
                    var aa = lis[j]
                    aa.classList.remove('big')
                }
                this.classList.add('big');
            }
        }
    }
}
```

首先，还是监听鼠标的移动事件，然后将所有的列表项 `class='big'` 的属性给移除掉，然后再给当前鼠标移动到的列表项设置 `class='big'` 属性，这样就达到了一个动态更换 `class` 的效果。
