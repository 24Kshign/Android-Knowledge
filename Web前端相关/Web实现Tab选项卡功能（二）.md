## Web实现Tab选项卡功能（二）


##### [Web实现Tab选项卡功能（一）](https://www.jianshu.com/p/64fba48bd5cd)


### 序言

废话就不多说了，直接上图，今天带来一个`Tab`选项卡 + 二级菜单的功能，相信在各大网站上也很常见吧！

![](http://ooaap25kv.bkt.clouddn.com/18-10-25/19633381.jpg)

### 实现

先来看看 `html`代码：

```
<body>
    <div class="tabBar">
        <ul class="nav">
            <li><a href="#">首页</a>
                <ul class="subNav">
                    <li><a href="#">首页二级菜单</a></li>
                    <li><a href="#">首页二级菜单</a></li>
                    <li><a href="#">首页二级菜单</a></li>
                    <li><a href="#">首页二级菜单</a></li>
                </ul>
            </li>
            <li><a href="#">关于</a>
                <ul class="subNav">
                    <li><a href="#">关于二级菜单</a></li>
                    <li><a href="#">关于二级菜单</a></li>
                    <li><a href="#">关于二级菜单</a></li>
                    <li><a href="#">关于二级菜单</a></li>
                </ul>
            </li>
            <li><a href="#">我的</a>
                <ul class="subNav">
                    <li><a href="#">我的二级菜单</a></li>
                    <li><a href="#">我的二级菜单</a></li>
                    <li><a href="#">我的二级菜单</a></li>
                    <li><a href="#">我的二级菜单</a></li>
                </ul>
            </li>
            <li><a href="#">购物车</a>
                <ul class="subNav">
                    <li><a href="#">购物车二级菜单</a></li>
                    <li><a href="#">购物车二级菜单</a></li>
                    <li><a href="#">购物车二级菜单</a></li>
                    <li><a href="#">购物车二级菜单</a></li>
                </ul>
            </li>
            <li><a href="#">个人中心</a>
                <ul class="subNav">
                    <li><a href="#">个人中心二级菜单</a></li>
                    <li><a href="#">个人中心二级菜单</a></li>
                    <li><a href="#">个人中心二级菜单</a></li>
                    <li><a href="#">个人中心二级菜单</a></li>
                </ul>
            </li>
        </ul>
    </div>
</body>
```

是不是跟昨天差不多，只不过在每一个一级菜单里面添加了一个二级菜单，效果是一样的，同样，我们来看看没有经过 `CSS` 洗礼的原型图：

![](http://ooaap25kv.bkt.clouddn.com/18-10-25/76806867.jpg)

这样肯定不是我们想看到的，下面我们来看一下 `CSS` 部分的代码：

```
.tabBar{
    width: 800px;
    height: 50px;
    background-color: #ffffff;
    margin: 10px auto;
}

.nav {
    width:760px;
    margin: 0 auto;
}

li {
    list-style: none;
}

a{
    text-decoration: none;
    text-align: center;
    display: block;
}

.nav li {
    width: 150px;
    height: 50px;
    float: left;
}

.nav li a{
    width: 150px;
    height: 50px;
    margin: 0 1px;
    background-color: #e1e1e1;
    line-height: 50px;
    color: #000000;
}

.nav li a:hover{
    color: #ffffff;
    background-color: #f60;
}

.subNav{
    height: 0px;
    overflow: hidden;
}

.subNav li a{
    width: 150px;
    height: 50px;
    text-align: center;
    background-color: #f7f7f7;
    line-height: 50px;
    color: #666666;
}

.subNav li a:hover{
    color: #ffffff;
    background-color: #f60;
}
```

大部分CSS代码还是和昨天写的一样，讲一下新用到的东西，我们在正常情况下是不显示二级菜单的，所以这里我们使用`overflow: hidden;`来实现，这个属性的意思就是当内容溢出元素框时，我就隐藏起来，它不止 `hidden` 这一个值可以设置，还有其他的；而且要将 `Subnav` 的高度设置成0，这样才能完全隐藏二级菜单，其他设置和之前一样；那么怎么让鼠标移动到某个 `tabBar` 元素之后，显示该元素下的二级菜单呢？我们都知道，在 `Web` 中，`Js` 被用来处理用户与网页客户端的交互，所以这就要借助我们的 `Js` 来完成这件事了，上代码：

```
window.onload = function() {
    var lis = document.getElementsByTagName('li');
    for (var i=0; i<lis.length;i++){
        lis[i].onmouseover = function () {
            var nav = this.getElementsByTagName('ul')[0];
            if (null != nav) {
                var that = nav;
                clearInterval(that.time);
                that.time = setInterval(function() {
                    that.style.height = that.offsetHeight + 16 + "px";
                    if (that.offsetHeight >= 200){
                        clearInterval(that.time);
                    }
                }, 20)
            }
        }
        lis[i].onmouseleave = function () {
            var nav = this.getElementsByTagName('ul')[0];
            if (null != nav) {
                var that = nav;
                clearInterval(that.time);
                that.time = setInterval(function () {
                    that.style.height = that.offsetHeight - 16 + "px";
                    if (that.offsetHeight <= 0) {
                        that.style.height = 0 +"px";
                        clearInterval(that.time);
                    }
                }, 20)
            }
        }
    }
}
```

`window.onload` 方法是页面在加载完之后就会被立即执行，这样避免代码中一些对象未被加载就被操作了，然后获取 `li` 元素，并对每个 `li` 元素设置鼠标监听事件，当鼠标经过的时候，循环执行一个函数，一个增加 `subNav` 的高度的函数，当增加的高度超过了二级菜单中的高度时，取消函数的执行，这样就能形成一个打开的动画，同理，在鼠标移除监听事件中，也循环执行减去 `subNav` 高度的函数，这样就能形成一个关闭动画。