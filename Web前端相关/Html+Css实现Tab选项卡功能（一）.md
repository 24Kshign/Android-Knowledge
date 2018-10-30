## Html+Css实现Tab选项卡功能（一）

### 序言

前段时间学习了一下Html+Css+Js相关知识，感觉不用的话很容易忘记，因此打算每周有空的时候都会写一些小型的demo，也希望能给想入门Web前端的Android程序猿们一些建议，好了话不多说，代码为敬：

![](http://ooaap25kv.bkt.clouddn.com/18-10-24/37644265.jpg)

### 实现

就是这样一个简单的效果，来看一下html代码：

```
<body>
    <div class="tabBar">
        <ul>
            <li><a href="#">首页</a></li>
            <li><a href="#">关于</a></li>
            <li><a href="#">我的</a></li>
            <li><a href="#">购物车</a></li>
            <li><a href="#">个人中心</a></li>
        </ul>
    </div>
</body>
```

是不是很简单，`html`代码里面只管放东西进去就好了，需要什么就放什么，在`html`中，通常用`ul`标签来实现无需列表，而`li`标签通常被用来定义列表项目，所以它们俩是配套使用的，塞完东西之后我们要去摆放和设置一些样式

![没有加css样式](http://ooaap25kv.bkt.clouddn.com/18-10-24/85361508.jpg)

要想好看，接下来看看`css`代码：

```
.tabBar{
    width: 600px;
    height: 50px;
    background-color: #ffffff;
    margin: 10px auto;
}

.tabBar ul {
    width:510px;
    margin: 0 auto;
}

.tabBar ul li {
    width: 100px;
    height: 50px;
    float: left;
    list-style: none;
}

.tabBar ul li a{
    width: 100px;
    height: 50px;
    text-align: center;
    margin: 0 1px;
    display: block;
    background-color: #e1e1e1;
    line-height: 50px;
    text-decoration: none;
    color: #000000;
}

.tabBar ul li a:hover{
    color: #ffffff;
    background-color: #f60;
}
```

貌似看起来也很简单，解释一下，`list-style: none;`这句代码是去除`<li>`前面的圆点的，`text-decoration: none;`这句是去除`<a>`标签的下划线的，然后我们对`<li>`标签使用`float: left;`属性，这样就能使得列表横向排列了。代码中有一些居中的写法，我们来看看：

- `margin: 0 auto;` || `margin: 10px auto;`这样是让布局水平居中，这其实是一个缩写，想了解更多的可以翻到底下去看我以前写的文章

- `line-height: 50px;`这种可以让布局垂直居中，这个高度设置为父布局的高度就行了

- `text-align: center;`这种一般是文本居中的方式。


### 小白请先看这里

#### [1、初识前端](https://www.jianshu.com/p/e3021fb2d85c)

#### [2、初识JavaScript](https://www.jianshu.com/p/d84b66839570)

#### [3、Android开发人员不得不学习的JavaScript基础（一）](https://www.jianshu.com/p/4a1c75990f84)

#### [4、Android开发人员不得不学习的JavaScript基础（二）](https://www.jianshu.com/p/b33068c297be)

#### [5、Android开发人员不得不学习的Vue.js基础](https://www.jianshu.com/p/1d8e33406fe9)


### 公众号

欢迎关注我的个人公众号【IT先森养成记】，专注大前端技术分享，包含Android，Java基础，Kotlin，HTML，CSS，JS等技术；在这里你能得到的不止是技术上的提升，还有一些学习经验以及志同道合的朋友，赶快加入我们，一起学习，一起进化吧！！！

![公众号：IT先森养成记](http://ooaap25kv.bkt.clouddn.com/18-10-24/75189748.jpg)