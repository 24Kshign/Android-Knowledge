## Android开发人员初识JavaScript

JavaScript是一种脚本语言；网页，以及基于H5的手机app等都靠JavaScript来驱动；更简单的来说，JavaScript就像是一种运行在浏览器中的解释型语言。

### 变量

在JavaScript中，定义变量需要使用var关键字，语法如下：

```
var 变量名
```
变量名要遵循命名规则：

- 变量必须使用字母、下划线或者美元开始
- 可以使用任意多个英文字母、数字、下划线或者美元符号组成
- 不能使用JavaScript关键词与保留字作为变量名

![摘自慕课网](http://img.mukewang.com/529c07c000014f5103080447.jpg)

### 函数
和其他语言一样，JavaScript同样具有函数，在JavaScript中如何定义一个函数呢：

```
function 函数名()
{
     函数代码;
}
```
函数的定义遵循以下规则：

- 一定要使用关键字function来定义函数
- “函数名”不要使用中文

### 消息对话框

在JavaScript中，消息对话框有三种：

##### 1、alert警告框

![](http://ooaap25kv.bkt.clouddn.com/18-8-7/23074058.jpg)

>**注意：**
>
>(1)、在点击对话框"确定"按钮前，不能进行任何其它操作。
>
>(2)、消息对话框通常可以用于调试程序。
>
>(3)、alert输出内容，可以是字符串或变量，与document.write 相似。

##### 2、confirm确认框

confirm消息对话框通常用于允许用户做选择的动作，如：“你对吗？”等。弹出对话框(包括一个确定按钮和一个取消按钮)。

```
用法：
confirm(str);

参数说明：
str：在消息对话框中要显示的文本
返回值: Boolean值

返回值：
当用户点击"确定"按钮时，返回true
当用户点击"取消"按钮时，返回false
```
![](http://ooaap25kv.bkt.clouddn.com/18-8-7/66497542.jpg)

##### 3、prompt提问框

prompt弹出消息对话框,通常用于询问一些需要与用户交互的信息。弹出消息对话框（包含一个确定按钮、取消按钮与一个文本输入框）。

```
用法：
prompt(str1, str2);
prompt(str1);

参数说明：
str1: 要显示在消息对话框中的文本，不可修改
str2：文本框中的默认内容，可修改，也可为空

返回值：
当用户点击确定按钮时，文本框中的内容将作为函数返回值
当用户点击取消按钮时，将返回null
```
![](http://ooaap25kv.bkt.clouddn.com/18-8-7/54850731.jpg)

### 打开新的窗口

使用window.open()方法可以打开一个已经存在或者新建的浏览器窗口。

```
window.open([URL], [窗口名称], [参数字符串])
```

##### 参数说明：

**1、URL：**

可选参数，在窗口中要显示网页的网址或路径。如果省略这个参数，或者它的值是空字符串，那么窗口就不显示任何文档。

**2、窗口名称：**

可选参数，被打开窗口的名称。

>(1).该名称由字母、数字和下划线字符组成。
>
>(2)."_top"、"_blank"、"_self"具有特殊意义的名称。
>
       _blank：在新窗口显示目标网页
       _self：在当前窗口显示目标网页
       _top：框架网页中在上部窗口中显示目标网页
       
>(3).相同 name 的窗口只能创建一个，要想创建多个窗口则 name 不能相同。
   
>(4).name 不能包含有空格。

**3、参数字符串**

![摘自慕课网](http://img.mukewang.com/52e3677900013d6a05020261.jpg)

### 文档对象模型DOM

##### 1、通过ID来获取元素

在HTML中，元素的id是唯一的，那么我们可以通过id来获取某一元素，然后对标签进行动态操作。

```
document.getElementById(“id”) ;

获取的结果为null或者[object HTMLParagraphElement]
```
**注意：这里获取到的元素是一个对象，如果想对元素进行操作，可以通过元素的属性或者方法即可。**

##### 2、替换HTML元素内容

通过innerHTML属性就可以获取或替换 HTML 元素的内容。

```
Object.innerHTML = xxx;
```
Object是获取的元素对象，如通过document.getElementById("ID")获取的元素。

##### 3、改变HTML样式

HTML DOM 允许 JavaScript 改变 HTML 元素的样式。

```
Object.style.元素属性 = new style;
```
Object是获取的元素对象，如通过document.getElementById("id")获取的元素。可以通过修改以下属性来改变HTML的样式：

![摘自慕课网](http://img.mukewang.com/52e4d4240001dd6c04850229.jpg)

**注意：该表只是一小部分CSS样式属性，其它样式也可以通过该方法设置和修改。**

##### 4、显示与隐藏

在网页中，我们经常可以看到某个元素显示和隐藏的效果，是通过display属性来实现的。

```
Object.style.display = "none";    //隐藏元素

bject.style.display = "block";    //显示元素
```
Object是获取的元素对象，如通过document.getElementById("id")获取的元素。

##### 5、控制类名

通过className属性设置或返回元素的class属性。

```
object.className = "css样式";
```
通常使用该属性为某个元素动态改变css样式。