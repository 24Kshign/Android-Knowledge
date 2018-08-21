## JavaScript基础（二）

### 对象

书接上文，在JavaScript中，有很多常用的对象，这一节中我们继续讲对象。

#### 1、window对象

window对象是BOM的核心，window对象指当前的浏览器窗口，window对象中有很多种方法供开发者调用：

|    方法   |    描述   | 
|    :-:   |    :-:   |
|    alert()  |   显示带有一段消息和一个确认按钮的警告框  |
|    prompt()  |   显示可提示用户输入的对话框  |
|    confirm()  |   显示带有一段消息以及确认按钮和取消按钮的对话框  |
|    open()  |   打开一个新的浏览器窗口，或者查找一个已命名的窗口  |
|    close()  |   关闭浏览器窗口  |
|    print()  |   打印当前窗口内容  |
|    focus()  |   把键盘焦点给予一个窗口  |
|    blur()  |   把键盘焦点从顶层窗口移开  |
|    movebBy()  |   可相对窗口的当前坐标把它移动到指定的像素  |
|    moveTo()  |   把窗口的左上角移动到一个指定的坐标  |
|    resizeBy()  |   按照指定的像素调整窗口的大小  |
|    resizeTo()  |   把窗口的大小调整到指定的宽和高  |
|    scrollBy()  |   按照指定的像素值来滚动内容  |
|    scrollTo()  |   把内容滚动到指定位置  |
|    setInterval() |   每隔指定的时间执行代码  |
|    setTimeOut()  |   在指定的延迟时间之后来执行代码  |
|    clearInterval()  |   取消setInterval的值  |
|    clearTimeout()  |   取消setTimeOut的值  |

#### 2、history对象

history对象记录了用户曾经浏览过的页面(URL)，并可以实现浏览器前进与后退相似导航的功能。需要注意的是从窗口被打开的那一刻开始记录，每个浏览器窗口、每个标签页乃至每个框架，都有自己的history对象与特定的window对象关联。

|    方法/属性   |    描述   | 
|    :-:   |    :-:   |
|   length   |    返回浏览器历史列表中的URL数量   |
|   back()   |    加载history列表中的前一个URL   |
|   forward()   |    加载history列表中的下一个URL   |
|   go()   |    加载history列表中的某个具体的页面   |

#### 3、location对象

location用于获取或设置窗体的URL，并且可以用于解析URL。我们先看看location对象属性图示：

![摘自慕课网](http://img.mukewang.com/53605c5a0001b26909900216.jpg)

下面是location对象的一些属性以及方法：

|    方法/属性   |    描述   | 
|    :-:   |    :-:   |
|    hash    |   设置或返回从#号开始的URL（锚）   |
|    host    |   设置或返回主机名和当前URL的端口号    |
|    hostname    |   设置或返回当前URL的主机名    |
|    href    |   设置或返回完整的URL    |
|    pathname    |   设置或返回从#号开始的URL（锚）    |
|    port    |   设置或返回当前URL的端口号    |
|    protocol    |   设置或返回当前URL的协议）    |
|    search    |   设置或返回从?号开始的URL（查询部分）    |
|    assign()    |   加载新的文档    |
|    reload()    |   重新加载当前文档    |
|    replace()    |   用心的文档替换当前文档    |

#### 4、navigator对象

navigator对象包含有关浏览器的信息，通常用于检测浏览器与操作系统的版本。以下是navigator对象的一些属性：

|   属性   |    描述   | 
|    :-:   |    :-:   |
|   appCodeName  |   浏览器代码名的字符串表示   |
|   appName  |   返回浏览器名称   |
|   appVersion  |   返回浏览器的平台和版本信息   |
|   platform  |   返回运行浏览器的操作系统平台   |
|   userAgent  |   返回由客户机发送服务器的user-agent头部值   |

**4.1、userAgent**

返回用户代理头的字符串表示(就是包括浏览器版本信息等的字符串)；几种浏览的user_agent，像360的兼容模式用的是IE、极速模式用的是chrom的内核。可以使用userAgent属性来判断使用的是什么浏览器：

![摘自慕课网](http://img.mukewang.com/535a3a4a0001e03f06870189.jpg)

#### 5、screen对象

screen对象用于获取用户的屏幕信息，以下是screen对象的属性

|   属性   |    描述   | 
|    :-:   |    :-:   |
|   avaiHeight  |   窗口可以使用的屏幕高度，单位为像素   |
|   avaiWidth  |   窗口可以使用的屏幕宽度，单位为像素   |
| colorDepth | 用户浏览器表示的颜色位数，通常为32位（每像素的位数）(IE浏览器不支持) |
|   pixelDepth  |   窗口可以使用的屏幕高度，单位为像素   |
|   height  |   屏幕的高度，单位为像素   |
|   width  |   屏幕的宽度，单位为像素   |


#### 5、DOM对象

文档对象模型DOM（Document Object Model）定义访问和处理HTML文档的标准方法。DOM 将HTML文档呈现为带有元素、属性和文本的树结构（节点树）。将HTML代码分解为DOM节点层次如图所示：

![](http://img.mukewang.com/5375ca7e0001dd8d04830279.jpg)

HTML文档是由节点构成的集合，DOM节点有以下几种：

>**5.1、元素节点：**上图中html、body、p等都是元素节点，即标签。
>
>**5.2、文本节点：**向用户展示的内容，入li中的JavaScript、DOM、CSS等文本。
>
>**5.3、属性节点：**元素属性，如a标签的链接属性href="http:xxx.xxx.xxx"。

**节点属性如下表：**

|   属性   |    说明   | 
|    :-:   |    :-:   |
|   nodeName     |    返回一个字符串，其内容是给定节点的名字   |
|   nodeType     |    返回一个整数，这个数值代表给定节点的类型   |
|   nodeValue    |   返回给定节点的当前值    |

- nodeName 属性: 节点的名称，是只读的。
 	- 元素节点的 nodeName 与标签名相同
	- 属性节点的 nodeName 是属性的名称
	- 文本节点的 nodeName 永远是 #text
	- 文档节点的 nodeName 永远是 #document
- nodeValue 属性：节点的值
	- 元素节点的 nodeValue 是 undefined 或 null
	- 文本节点的 nodeValue 是文本自身
	- 属性节点的 nodeValue 是属性的值
- nodeType 属性: 节点的类型，是只读的。以下常用的几种结点类型：

|   元素类型   |    节点类型   | 
|    :-:   |    :-:   |
|   元素  |    1  |
|   属性  |    2  |
|   文本  |    3  |
|   注释  |    8  |
|   文档  |    9  |

**遍历节点树：**

|   方法   |    说明   | 
|    :-:   |    :-:   |
|   childNodes    |   返回一个数组，这个数组由给定元素节点的子节点   |
|   firstChild    |   返回第一个子节点   |
|   lastChild    |   返回最后一个节点   |
|   parentNode    |   返回一个给定节点的父节点   |
|   nextSibling    |   返回给定节点的下一个节点   |
|   previousSibling    |   返回给定节点的下一个节点   |

**DOM操作：**

|   方法   |    说明   | 
|    :-:   |    :-:   |
|   createElement(ele)   |    创建一个新的元素节点   |
|   createTextNode()   |    创建一个包含着给定文本的新文本节点   |
|   appendChild()   |  指定节点的最后一个节点列表之后添加一个新的子节点  |
|   insertBefore()   |    将一个给定节点插入到一个给定元素节点的给定子节点前面   |
|   removeChild()   |    从一个给定元素中删除字子节点   |
|   replaceChild(ele)   |    把一个给定元素里的一个子节点替换成另外一个节点   |

>**5.4、getElementsByName()方法**，返回带有指定名称的节点对象的集合。
>
>- 因为文档中的name属性可能不唯一，所有getElementsByName() 方法返回的是元素的数组，而不是一个元素。
>
>- 和数组类似也有length属性，可以和访问数组一样的方法来访问，从0开始。
>
>**5.5、getElementsByTagName()方法**，返回带有指定标签名的节点对象的集合。返回元素的顺序是它们在文档中的顺序。
>
>- Tagname是标签的名称，如p、a、img等标签名。
>
>- 和数组类似也有length属性，可以和访问数组一样的方法来访问，所以从0开始。