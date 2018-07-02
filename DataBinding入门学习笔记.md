
## DataBinding学习笔记

### 学习网站

#### [DataBinding使用全面详解](https://www.jianshu.com/p/ba4982be30f8)

### 

#### 1、DataBinding介绍

    DataBinding是一个数据绑定框架，是Google对MVVM在Android上的一种实现，可以直接绑定数据到xml中，并实现自动刷新。
    以前我们在Activity里写很多的findViewById，现在如果我们使用DataBinding，就可以抛弃findViewById。DataBinding
    主要解决了两个问题：
- 需要多次使用findViewById，损害了应用性能且令人厌烦
- 更新UI数据需切换至UI线程，将数据分解映射到各个view比较麻烦
    


#### 2、基本使用

##### 2.1、DataBinding的导入

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

##### 2.2、摆脱findviewbyid及绑定
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

##### 2.3、绑定model数据
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

##### 2.4、事件的绑定

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
    3.android:onClick="@{()->event.cilck3(title）}"
    
[注]：()->event.cilck3(title)是lambda表达式写法，
也可以写成：(view)->event.cilck3(title),前面(view)表示onClick方法的传递的参数，
如果event.click3()方法中不需要用到view参数，可以将view省略。


#### 3、基本原理

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


#### 3、性能介绍

    1、0反射
    2、findViewById需要遍历整个viewgroup，而现在只需要做一次就可以初始化所有需要的view
    3、使用位标记来检验更新（dirtyFlags）
    4、数据改变在下一次批量更新才会触发操作
    5、表达式缓存，同一次刷新中不会重复计算
    6、自动帮助我们进行空指针的避免，比如说@{employee.firstName}，如果employee是null的话，employee.firstName则会被赋默认值（null）。
    int的话，则是0。
    
#### 4、Observable

    一个纯净的Java ViewModel类被更新后，并不会让UI去更新。而数据绑定后，我们当然会希望数据变更后UI会即时刷新，
    Observable就是为此而生的概念：
    
##### 4.1、类继承BaseObservable:
   
```
   private static class User extends BaseObservable {
   private String firstName;
   private String lastName;
   @Bindable
   public String getFirstName() {
       return this.firstName;
   }
   @Bindable
   public String getLastName() {
       return this.lastName;
   }
   public void setFirstName(String firstName) {
       this.firstName = firstName;
       notifyPropertyChanged(BR.firstName);
   }
   public void setLastName(String lastName) {
       this.lastName = lastName;
       notifyPropertyChanged(BR.lastName);
   }
}
```
    BaseObservable提供了一系列notify函数（其实就是notifyChange和notifyPropertyChanged），前者会刷新所有的值域，
    后者则只更新对应BR的flag，该BR的生成通过注释@Bindable生成，在上面的实例代码中，我们可以看到两个get方法被注释上了，
    所以我们可以通过BR访问到它们并进行特定属性改变的notify。
    
##### 4.2、Observable Fields:
    
如果所有要绑定的都需要创建Observable类，那也太麻烦了。所以Data Binding还提供了一系列Observable，包括 ObservableBoolean,
ObservableByte, ObservableChar, ObservableShort, ObservableInt, ObservableLong, ObservableFloat, ObservableDouble, 
和ObservableParcelable。我们还能通过ObservableField泛型来申明其他类型，如
    
```
   private static class User {
   public final ObservableField<String> firstName =
       new ObservableField<>();
   public final ObservableField<String> lastName =
       new ObservableField<>();
   public final ObservableInt age = new ObservableInt();
   }
```
在xml中，使用方法和普通的String，int一样，只是会自动刷新，但在java中访问则会相对麻烦：
    
```
user.firstName.set("Google");
int age = user.age.get();
```
相对来说，每次要get/set还是挺麻烦，私以为还不如直接去继承BaseObservable。
    
    
    
 
    






	

