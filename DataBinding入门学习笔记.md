
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
在布局中是通过@{}来绑定数据的，{}中是布局中该控件属性对应的数据类型数据，同时还可以支持运算符运算和静态方法调用和转换，这个后面会介绍

###### 2.MainActivity文件 :  
页面的.json只能设置 window 相关的配置项，以决定本页面的窗口表现，所以无需写 window 这个键。  
属性如下：
```
public class MainActivity extends AppCompatActivity {

    private ActivityMainBinding binding;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        //通过DataBInding加载布局都会对应的生成一个对象，如ActivityMainBinding，对象名在布局文件名称后加上了一个后缀Binding
        binding = DataBindingUtil.setContentView(MainActivity.this, R.layout.activity_main);

        //2.绑定基本数据类型及String
        binding.setContent("对String类型数据的绑定");//setContent就是给布局文件text属性绑定的content设置值
        binding.setEnabled(false);//设置enabled值为false
        //给控件设置点击事件，发现其实点击无效，因为在布局文件中给cilckable属性绑定了enabled,在代码中设置enabled值为false，所以点击事件无效
        binding.mainTv2.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Toast.makeText(MainActivity.this, "我被点击了", Toast.LENGTH_SHORT).show();
            }
        });

    }
}
```

##### 1.4、绑定model数据
###### 1.Model数据类型 :

```
public class User {
    private String text;

    public User(String text) {
        this.text = text;
    }

    public String getText() {
        return text;
    }

    public void setText(String text) {
        this.text = text;
    }
}
```

###### 2.布局文件 :

```
<?xml version="1.0" encoding="utf-8"?><!--布局以layout作为根布局-->
<layout>

    <data>
        <!--绑定Model数据2中形式，一中是导入该类型的，一种指定类型的全称，和java一样-->
        <!--  方式一 -->
        <variable
            name="user"
            type="www.zhang.com.databinding.User" />
        <!--  方式二 -->
        <!--<import type="www.zhang.com.databinding.User" />-->
        <!--<variable-->
            <!--name="user"-->
            <!--type="User" />-->

    </data>
    <!--我们需要展示的布局-->
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">

        <Button
            android:id="@+id/main_btn3"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@{user.text}" /><!--这里user.text相当于user.getText()-->
    </LinearLayout>
</layout>
```

###### 3.MainActivity文件 :

```
public class MainActivity extends AppCompatActivity {

    private ActivityMainBinding binding;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        //通过DataBInding加载布局都会对应的生成一个对象，如ActivityMainBinding，对象名在布局文件名称后加上了一个后缀Binding
        binding = DataBindingUtil.setContentView(MainActivity.this, R.layout.activity_main);

        //3.绑定model对象数据
        User user = new User("绑定Model数据类型");
        binding.setUser(user);//或者 binding.setVariable(BR.user,user);
    }
}
```
[注]：binding设置数据有两种方式:1.binding.setUser(user) 2.binding.setVariable(BR.user,user)–采用BR指定

##### 1.5、事件的绑定

```
<Button
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:onClick="@{event.click}"
                android:text="@{title}" />
```
事件的多种写法：

    1.android:onClick="@{event.click1}" 
    2.android:onClick="@{event::click2}" 
    3.android:onClick="@{()->event.cilck3(title4 
    
[注]：()->event.cilck3(title4)是lambda表达式写法，
也可以写成：(view)->event.cilck3(title4),前面(view)表示onClick方法的传递的参数，
如果event.click3()方法中不需要用到view参数，可以将view省略。


#### 2、基本原理

    一个Activity会有一个Window对象，而一个Window对象也有一个DecorView。DecorView是一个ViewGroup，布局文件都是通过
    inflate转化为view，加入到DecorView中，可以说DecorView是最大的根布局，而这个android.R.id.content正是它的id。
    DataBinding通过获取这个根布局，然后通过for循环将里面的控件一个个return出去，然后在生成的实体类再一个个获取。
    这样子的效率比直接findViewByid要效率的多，因为每次findViewByid都需要进行一次for循环在ViewGroup里面来寻找指定id名的控件。
    
生成ActivityMainBinding实例并绑定过程，主要有三个过程：：

第一步就是Inflate 处理后的布局文件，由于现在activity_main.xml文件与普通的layout文件一样。现在DataBindingUtil将会Inflate activity_main.xml文件，得到一个ViewGroup变量root。

第二步就是生成ActivityMainBinding实例对象，DataBindingUtil会将这个变量root传递给ActivityMainBinding的构造方法，生成一个ActivityMainBinding的实例，就是我们在onCreate方法中获取的binding对象。在该构造方法中，ActivityMainBinding会首先遍历root，根据各个View的Tag或者id，初始化自己的fields，完成后，ActivityMainBinding将会把之前加到各个View上的Tags清空。最后，构造方法调用invalidateAll引发数据绑定。

第三步就是进行数据绑定。在这一步中，ActivityMainBinding将会计算各个view上的binding表达式，然后赋值给view相应的属性。
代码如下（省略了细节）：
```
@Override 
protected void executeBindings() { 
  java.lang.String firstNameUser = null; 
  java.lang.String lastNameUser = null; 
  com.like4hub.www.databindingtest.User user = mUser; 

  if (user != null) { 
     // read user.firstName 
     firstNameUser = user.getFirstName(); 
     // read user.lastName 
     lastNameUser = user.getLastName(); 
   } 

   TextViewBindingAdapter.setText(this.firstname, firstNameUser); 
   TextViewBindingAdapter.setText(this.lastname, lastNameUser);
   ImageViewBindingAdapter.setImageDrawable(this.mboundView3,
   getDrawableFromResource(R.drawable.avatar_pure)); 
}
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


