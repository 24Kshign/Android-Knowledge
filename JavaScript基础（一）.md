## JavaScript基础（一）

### 操作符

在JavaScript中，有很多种操作符，算术操作符、赋值操作符、比较操作符以及逻辑操作符

#### 1.1、算术操作符：

`+`，`-`，`*`，`/`，除了加号(`+`)之外，其他都是按照四则运算大方式来进行，而加号(`+`)在字符串中可以作为连接符来使用，这个和Java是一样的。

![摘自慕课网](http://img.mukewang.com/52b9227d0001acc703000174.jpg)

#### 1.2、赋值操作符：

`=`操作符不是等于，而是赋值操作

#### 1.3、比较操作符：

|   操作符  |    描述   | 
|    :-:   |    :-:   |
|     <    |    小于   |
|     >    |    大于   |
|     <=    |    小于等于   |
|     >=    |    大于等于   |
|     ==    |    等于（比较值是否相同）   |
|     ===    |    等于（比较值和值的类型是否相同）   |
|     !=    |    不等于（同==）   |
|     !==    |    不等于（同===）   |

JavaScript中的比较操作符的用法和Java是一样的。但是需要注意的是表格中的`==`与`===`的不同用法。

#### 1.4、操作符优先级：

算术操作符 > 比较操作符 > 逻辑操作符 > 赋值操作符；如果同级的运算是按从左到右次序进行,多层括号由里向外。


### 数组

在JavaScript中，定义一个数组是没有类型的，也就是说可以往数组中添加任何类型的数据。

#### 1、定义

```
var myarray=new Array(); //没有指定数组的长度

var myarray= new Array(8); //创建一个长度为8的数组
```
**注意：**

- 创建的新数组是空数组，没有值，如输出，则显示undefined。
- 虽然创建数组时，指定了长度，但实际上数组都是变长的，也就是说即使指定了长度为8，仍然可以将元素存储在规定长度以外。

#### 2、数组长度属性

使用`array.length`属性来获取数组的长度，因为数组的索引总是由0开始，所以一个数组的上下限分别是：0和length-1；同时，在JavaScript中数组的length属性是可变的。

#### 3、二维数组

在JavaScript中，二维数组用myarray[x][y]来表示，声明二维数组有两种方式：

**方式一：**

```
var myarr=new Array();  //先声明一维 
for(var i=0;i<2;i++){   //一维长度为2
   myarr[i]=new Array();  //再声明二维 
   for(var j=0;j<3;j++){   //二维长度为3
      myarr[i][j]=i+j;   // 赋值，每个数组元素的值为i+j
   }
}
```

**方式二：**

```
var Myarr = [[0 , 1 , 2 ],[1 , 2 , 3]]
```

### 函数

在JavaScript中，函数的使用需要注意以下几点事项：

#### 1、函数调用

**方式一：在JS代码中直接调用**

```
<script>
	var sum;
	function add2(){
   		sum = 3 + 2;
   		alert(sum);
	}
	add2();
</script>
```

**方式二：在HTML标签中调用**

```
<script>
	var sum;
	function add2(){
		sum = 3 + 2;
		alert(sum);
	}
</script>

<body>
	<input type="button" value="按钮" onclick="add2()" />
</body>
```

#### 2、函数参数

**2.1、无参：**

```
<script>
	var sum;
	function add2(){
   		sum = 3 + 2;
   		alert(sum);
	}
</script>
```

**2.2、有参**

```
<script>
	var sum;
	function add2(x,y){
   		sum = x + y;
   		alert(sum);
	}
</script>
```
**注意：有参函数中，参数类型是没有类型的（好随意的语言啊>_<）**

#### 3、函数返回值

这里需要注意的是，在JavaScript中，函数的定义是没有返回值类型这一说的，不像Java里面，任何一个函数都需要指明返回值类型。

```
<script>
	var sum;
	function add2(x,y){
   		sum = x + y;
   		return sum;
	}
</script>
```

上面的函数中，你为它增加一条return语句，它就是一个有返回值的函数了，如果去掉return语句，那么它就是一个无返回值的函数（不得不又说一句，JS真随意!!!）


### 事件

JavaScript 创建动态页面。事件是可以被 JavaScript 侦测到的行为。 网页中的每个元素都可以产生某些可以触发 JavaScript 函数或程序的事件。

|   事件  |    说明  | 
|    :-:   |    :-:   |
|     onclick    |    鼠标单击事件   |
|     onmouseover    |    鼠标经过事件   |
|     onmouseout    |    鼠标移开事件   |
|     onchange    |    文本框内容改变事件   |
|     onselect    |    文本框内容被选中事件   |
|     onfocus    |    光标聚集   |
|     onblur    |    光标离开   |
|     onload    |    网页导入   |
|     onunload    |    关闭网页   |


### 对象

在JavaScript中也是有对象这一说的，和Java中的对象差不多。JavaScript中的所有事物都是对象，如:字符串、数值、数组、函数等，每个对象带有属性和方法。

**对象的属性：**反映该对象某些特定的性质的，如：字符串的长度、图像的长宽等；

**对象的方法：**能够在对象上执行的动作。例如，表单的“提交”(Submit)，时间的“获取”(getYear)等；

#### 1、Date对象

在JavaScript中，Date对象被用来存储/获取日期，该对象有以下方法/属性：

|    方法名称     |     功能描述  |
|       :-:      |      :-:    |
|   get/setDate()  |    返回/设置日期    |
|   get/setFullYear()  |    返回/设置年份，用四位数表示    |
|   get/setYear  |    返回/设置年份    |
|   get/setMonth  |    返回/设置月份<br>0：一月  ...  11：十二月，所以要加1    |
|   get/setHours  |    返回/设置小时  24小时制    |
|   get/setMinutes()  |    返回/设置分钟    |
|   get/setSeconds()  |    返回/设置秒    |
|   get/setTimes()  |    返回/设置时间（毫秒单位）    |
|   getDay()  |    返回星期<br>0：周一  ...  6：周日    |

#### 2、String对象

任何一门语言都离不开对String的操作，JavaScript也不例外，该对象有以下方法/属性：

|    方法名称     |     功能描述  |
|       :-:      |      :-:    |
|  String.charAt(1)  |  获取字符串中第1个位置的字符  |
|  String.indexOf('x')  | 或取字符x在字符串中第1次出现的位置，若没找到，返回-1   |
|  String.charAt(1)  |  获取字符串中第1个位置的字符|
|  String.split(separator,limit)| 第一个参数是字符串分割的参照字符，第二个参数表示最多切割多少次，可以不设置，该方法返回的是一个数组|
|  String.substring(start,end)| 第一个参数表示开始位置，第二个参数表示结束位置，可省略，返回的是从start到end-1位置的子串|
|  String.substr(start,length)| 第一个参数表示开始位置，第二个参数表示裁剪几个字符，可省略，返回的是从start到start+length位置的子串|

#### 3、Math对象

在JavaScript，Math对象提供对数据的数学计算。Math对象中有属性，也有方法，以下是Math对象中的属性：

|    属性名称     |     功能描述  |
|       :-:      |      :-:    |
|     E    | 返回算数常量e，即自然对数的底数（约为2.718）  |
|     LN2    | 返回2的自然对数（约等于0.693）  |
|     LN10    | 返回10的自然对数（约等于2.302）  |
|     LOG2E    | 返回以2为底E的对数（约等于1.442）  |
|     LOG10E    | 返回以10为底E的对数（约等于0.434）  |
|     PI    | 返回圆周率（约等于3.14159）  |
|     SQRT1_2    | 返回2的平方根的倒数（约等于0.707）  |
|     SQRT2    | 返回2的平方根（约等于1.414）  |

在JavaScript中，Math对象有以下方法：

|    属性名称     |     功能描述   |
|       :-:      |      :-:      |
|    abs(x)      |  返回x的绝对值  |
|    acos(x)      |  返回x的反余弦值  |
|    asin(x)      |  返回x的反正弦值  |
|    atan(x)      |  返回x的反正切值  |
|    atan2(y,x)      |  返回x轴到点(x,y)的角度（以弧度为单位）  |
|    ceil(x)      |  对x进行上舍入（向上取整）  |
|    floor(x)      |  对x进行下舍入（向下取整）  |
|    sin(x)      |  返回x的正弦值  |
|    cos(x)      |  返回x的余弦值  |
|    tan(x)      |  返回x的正切值  |
|    exp(x)      |  返回e的x次幂  |
|    log(x)      |  返回x的自然对数（底为e）  |
|    max(x,y)      |  返回x和y中的最大值  |
|    min(x,y)      |  返回x和y中的最小值  |
|    pow(x,y)      |  返回x的y次幂  |
|    random()      |  返回0-1之间的随机数  |
|    round(x)      |  返回x的四舍五入值  |
|    sqrt(x)      |  返回x的平方根  |


#### 4、数组Array对象

数组在上面已经介绍过了，就不再做过多的介绍了，下面来看看JavaScript中数组的一些方法以及属性：

|    属性/方法名称     |     功能描述   |
|       :-:      |      :-:      |
|    Array.length     |  返回数组长度    |
|    Array.contact(arr1,...arrn)  |  连接两个或多个数组并返回一个最终的数组    |
|    Array.join(seprator)   |  把数组的所有元素放入一个字符串，元素通过指定的字符seprator进行连接，返回一个字符串  |
|    Array.pop()   |   删除并返回数组的最后一项   |
|    Array.push(x1,...xn)   |   向数组的末尾添加一个或更多元素，并返回新数组的长度   |
|    Array.reverse()   |   颠倒数组中元素的顺序并返回一个数组   |
|    Array.shift()   |   删除并返回数组中的第一个元素   |
|    Array.slice(start,end)   |   从某个已有的数组返回选定元素，第一个参数表示从start处开始到end处结束，如果start为-1，那么从倒数第一个元素开始，以此类推，返回一个子串   |
|    Array.sort(function)   |   对元素进行排序操作，并返回一个数组，参数是可选的，但是必须是一个函数（升序，降序）   |
|    Array.splice(index,howmaney,item1...itemn)   |   删除元素，并向数组添加新的元素，第一个参数表示在数组中删除/添加的位置，第二个参数表示需要删除的元素的数量，后面是添加的参数   |
|    Array.toString()   |   把数组转化为字符串并返回   |





