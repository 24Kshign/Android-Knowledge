
## DataBinding学习笔记

### 学习网站

#### [DataBinding使用全面详解](https://www.jianshu.com/p/ba4982be30f8)

### 

#### 1、基本使用
##### 1.1、DataBinding介绍

    DataBinding是一个数据绑定框架，以前我们在Activity里写很多的findViewById，现在如果我们使用DataBinding，
    就可以抛弃findViewById。DataBinding主要解决了两个问题：
- 需要多次使用findViewById，损害了应用性能且令人厌烦
- 更新UI数据需切换至UI线程，将数据分解映射到各个view比较麻烦


##### 1.2、DataBinding的导入

在应用的build.gradle文件中添加如下代码：
```
android {
    ...

    //导入dataBinding支持
    dataBinding{
        enabled true
    }

    ...
}
```

##### 1.3、摆脱findviewbyid及绑定
###### 1.布局文件 :
	
```
<?xml version="1.0" encoding="utf-8"?><!--布局以layout作为根布局-->
<layout>

    <data>
        <!--绑定基本数据类型及String-->
        <!--name:   和java代码中的对象是类似的，名字自定义-->
        <!--type:   和java代码中的类型是一致的-->
        <variable
            name="content"
            type="String" />

        <variable
            name="enabled"
            type="boolean" />
    </data>
    <!--我们需要展示的布局-->
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">

        <!--绑定基本数据类型及String的使用是通过   @{数据类型的对象}  通过对应数据类型的控制显示-->
        <Button
            android:id="@+id/main_tv2"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:clickable="@{enabled}"
            android:text="@{content}" />
    </LinearLayout>
</layout>
```
###### 2.page.json :  
页面的.json只能设置 window 相关的配置项，以决定本页面的窗口表现，所以无需写 window 这个键。  
属性如下：
```
{
“navigationBarBackgroundColor”: #000,
“navigationBarTextStyle”: black/white,
“navigationBarTitleText”: string,
“backgroundColor”: #ddd,
“backgroundTextStyle”: dark/light,
“enablePullDownRefresh”: boolean,
“disableScroll”: boolean //设置为 true 则页面整体不能上下滚动；只在 page.json 中有效，无法在 app.json 中设置该项(默认 false )
}
```
###### 3.app.js :  
开发工具会生成一个默认的程序框架，其中程序的主流程代码包含在app.js中。一般会用来设置全局变量，比如用户信息等。  

```
{
globalData:{    //定义需要传输的全局对象
    userInfo:null,    
    test:"test"    
}   

var test = getApp().globalData.test;  //获取  
console.log(test)   
```
###### 4.app.wxss :  
app.wxss 是整个小程序的公共样式表。我们可以在页面组件的 class 属性上直接使用 app.wxss 中声明的样式规则。如果页面有自己的样式表， 则会覆盖公共样式表。  
```
.container {
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: space-between;
  padding: 0rpx 0;
  box-sizing: border-box;
}
```
#### 2、基础语法（着重部分）
##### 2.1、小程序的页面标签
标签中的自定义属性必须以 data- 开头  

```
<view></view>    类似于div标签，表示一个容器  

<image src='/'><image>   类似于 <img> 标签，表示一张图片   /表示根目录，不指定宽高的情况下默认宽度300px、高度225px  

<text></text>    类似于<span></span>标签，表示行内文本只有被该标签包围的文本才能被长按选中text标签可以嵌套text标签，会直接解析转义字符

<swiper><swiper>   图片轮播，样式和属性作用在swiper标签上  
```
	
##### 2.2、小程序的数据赋值

请求回来的数据，使用 this.setData() 方法，传入需要赋值到data数据的变量  
小程序总是会读取data对象里的数据来进行页面的数据绑定，这个动作是在onLoad事件之后执行的
```
var data = '获取到的数据'

this.setData(data)
```

如果获取到的数据是一个数组，那么需要传入一个对象  
```
this.setData({ local_key: data});    // 这样在data里面就相当于有一个数组local_key

```

##### 2.3、小程序的显示隐藏
通过使用wx:if = ‘值’ 以及wx:else 来进行显示和隐藏的判断
```
 <image src='/images/b.jpg' wx:if="true"></image>
```

##### 2.4、小程序的循环输出
使用<block></block>标签将循环的内容包裹

wx:for = '{{ 循环数组 }}'   wx:for-item = '循环的值'   wx:key = 'key值'   wx:for-index = '下标'

循环的值默认是 item , 这里的 wx:for-item也可以省略不写，  wx:for-index 默认是 index  
```
var local_key = [1,2,3,4,5,6,7]

<block wx:for="{{local_key}}" wx:for-item="item" wx:key="unique">
    <view  class='list'>
      {{item}}
    </view>
</block>
```
##### 2.5、小程序的时间绑定
可以使用bind和catch绑定  
bindtap或者bind:tap 不阻止冒泡  
catchtap或者catch:tap 阻止冒泡  
```
<text class='gofont' bind:tap="go">开启小程序之旅</text>
```

##### 2.6、小程序的页面跳转
1.跳转后可以返回：  

```
wx.navigateTo({
url: '跳转页面相对路径',
})
}
```

2.跳转后不能返回： 

```
wx.redirectTo({
url: '跳转页面相对路径',
})
```

3.跳转到有tab切换的页面：  
```
wx.switchTab({
url: '../post/post',
})
```
##### 2.7、小程序的跳转传参  
举个例子：
在wxml中：  
```
<view bindtap="shoppingCar" data-index="{{index}}">兑换</view>
```
在对应的js文件中：
```
 //跳转到购物车页面
  shoppingCar: function (e) {
    var index = e.target.dataset.index; //获取到了传参的值并做跳转
    wx.navigateTo({
      url: '../detail/detail?id='+index,
    })
  }
```

在跳转到的detail.js文件中：
```
Page({
  data:{
    title:''
  },
  onLoad:function(options){
    // 页面初始化 options为页面跳转所带来的参数
    this.setData({
        title:options.title //就是传过来的值
    })
  },
})
```

##### 2.8、小程序的设置缓存 
小程序的缓存最多不能超过7M
同步方法设置或修改：　key表示缓存变量名，string/object表示缓存的内容，可以是 string/object 中的一种类型:
```
wx.setStorageSync(key, string/object)
```

同步方法获取缓存：key表示缓存变量名,获取后赋值给变量:  
```
const value = wx.getStorageSync(key)
```

同步方法删除指定缓存：key表示缓存变量名:
```
wx.removeStorageSync(key)
```

同步方法删除所有缓存：
```
wx.clearStorageSync()
```

##### 2.8、小程序的弹窗
#### [参考](https://blog.csdn.net/gao_xu_520/article/details/71084162?locationNum=1&fps=1)

#### 3、开发中遇到的问题
##### 3.1、textarea层级最高,使用z-index也无效
示例：  
   当我在textarea的控件上方需要弹出个选项窗口，我把该窗口的层级设为最高时，弹出弹窗也还是会显示底层的textarea的文字，  
也就是说textarea层级最高，无论z-index设置成多少都无作用。
解决办法：
   通过使用临时变量保存textarea中的内容，在弹出时设置textarea内容为空，隐藏时重新赋值。
   
##### 3.1、tab页面只能使用switchTab
页面要返回/跳转至tabbar的某一页面，可用：
```
wx.switchTab({  
      url: '../b/b'  
    }); 
```
注意switchTab只能跳转到带有tab的页面，不能跳转到不带tab的页面

##### 3.2、苹果小屏幕手机会出现布局错乱的情况
在出现错乱的控件所对应的css中，将高度height改为固定高度或者去掉，最好不写上height:100%。


