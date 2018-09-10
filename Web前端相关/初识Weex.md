## 初识Weex

Weex 是一个使用 Web 开发体验来开发高性能原生应用的框架。

### 环境搭建

环境搭建在这里就不详细讲述了，大家可以去官网瞅瞅。

[Weex环境搭建](http://weex.apache.org/cn/guide/set-up-env.html)

### Weex 通用样式

#### 1、适配和缩略

1.1、Weex 对于长度值目前只支持像素值，不支持相对单位(em，rem)；适配以 750px 标准，详细可以参考：[weex-html5 组件进阶](https://yq.aliyun.com/articles/61067)

1.2、设定边框，border 目前不支持简写，类似`border: 1px solid #ff0000;`的组合写法，需要单独分开写。

1.3、设定背景颜色，background 目前不支持类似这样`background: red`的写法，需要写全。

#### 2、定位属性

2.1、weex 支持 position 定位，`relative | absolute | fixed | sticky`，默认值为 relative。

2.2、weex 目前不支持 z-index 设置元素的层级关系，但靠后的元素层级更高，因此，对于层级高的元素，可将其排列在后面。

2.3、如果定位元素超过容器边界，在 Android 下，超出部分将不可见，原因在于 Android 端元素 overflow 默认值为 hidden。

#### 3、文本属性

`color;` `font-size;` `font-weight;` `font-style;` `font-family;` `text-decoration;` `text-align;` `text-overflow`(内容超长时的省略样式); lines(指定文本行数)

#### 4、其他属性

4.1、weex 支持线性渐变（linear-gradient），不支持径向渐变（radial-gradient）

4.2、weex 中 box-shadow 仅仅支持 iOS。

4.3、目前<image>组件无法定义一个或几个角的border-radius，只对 iOS 有效，对 Android 无效。

4.4、weex 中，flexbox 是默认并且唯一的布局模型，每个元素都默认拥有了`display:flex`属性。

### Weex 内建组件

#### 1、a 组件

1.1、Weex 中 a 组件定义了指向 weex 页面打包后的一个 js 地址。

1.2、组件中无法添加文本，需要在其中 text 组件才能添加文本。

1.3、此组件支持除了自己外的所有 Weex 组件做为子组件。

1.4、支持所有通用样式。

1.5、请不要为 a 组件添加 click 事件。

```
<a href="../dist/index.web.js">
	<text>点击跳转</text>
</a>

注意：在weex中，a标签中直接写文字是没用的，需要嵌套一个<text>标签，而且a标签不能跳转
一个html链接。
```

#### 2、web 组件

2.1、web 组件用于在页面中嵌入一张网页；src 用以指定资源地址。

2.2、不支持任何子组件

2.3、pagestart，web 组件开始加载时执行。

2.4、pagefinish，web 组件完成加载时执行。

2.5、error，web 组件加载错误时执行。

```
<template>
	<div class="content">
		<web class="web" 
			:src="src" 
			v-on:pagestart="start"
			@pagefinish="finish"
			@error="error">
		</web>
	</div>
</template>

<script>
	export default{
		data:{
			src:'http:www.baidu.com'
		},
		methos:{
			start:function(e){
				console.log("start");
			},
			finish(e){    //简写，省略function
				console.log("finish");
			},
			error(e){
				console.log("error");
			}
		}
	}
</script>

<style>
	.content{
		width="750px;
		height:600px;
	}
	
	.web{
		width="750px;
		height:600px;
	}
</style>
```

#### 3、weex 内建模块 webview 模块

3.1、一系列 web 组件的操作接口。可以通过调用 this.$refs.el 来获取元素的引用。

3.2、goBack(webElement)，加载历史记录里的前一个资源地址。

3.2、goForward(webElement)，加载历史记录里的下一个资源地址。

3.3、reload(webElement)，刷新当前页面。