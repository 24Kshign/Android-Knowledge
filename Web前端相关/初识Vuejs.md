## 初识Vuejs

### 介绍

Vue是一套用于构建用户界面的渐进式框架。Vue 被设计为可以自底向上逐层应用。Vue 的核心库只关注视图层，不仅易于上手，还便于与第三方库或既有项目整合。另一方面，当与现代化的工具链以及各种支持类库结合使用时，Vue 也完全能够为复杂的单页应用提供驱动。

### 用法

#### 1、Vue实例

```
<body>
  <div id="root">{{msg}}</div>

  <script>
    new Vue({
      el: "#root",
      data: {
        msg: "hello world"
      },
      methods{
      
      }
    })
  </script>
</body>
```
上述代码中，在script标签中实例化了一个Vue对象，该Vue对象和id为root的div标签绑定了，所以可以通过Vue来操作这个div标签。data中存放的数据可供div调用，这样可以动态的修改div中的数据；methods中用于存放各种方法，供标签调用。div标签又被称作挂载点；{{}}被称作插值表达式。

#### 2、Vue中的数据、事件和方法

- **v-text：**用于操作纯文本，它会替代显示对应的数据对象上的值，它可以用{{}}来简写。

```
<body>
  <div id="root">
  	 <h1 v-text="number"></h1>
  </div>

  <script>
    new Vue({
      el: "#root",
      data: {
        msg: "hello world",
        number: "123"
      }
    })
  </script>
</body>
```

- **v-html：**用于输出html，它与v-text区别在于v-text输出的是纯文本，浏览器不会对其再进行html解析，但v-html会将其当html标签解析后输出。

```
<body>
  <div id="root">
  	 <h1 v-text="number"></h1>
  	 <h1 v-html="content"></h1>
  </div>

  <script>
    new Vue({
      el: "#root",
      data: {
        msg: "hello world",
        number: "123",
        content: "<h1>hello world</h1>"
      }
    })
  </script>
</body>
```

- **v-on:事件名称：**用来绑定一个事件监听器，通过它调用我们Vue实例中定义的方法。v-on:可以简写成@符号。

```
<body>

  <div id="root">
    <div @click="handleClick">{{content}}</div>
  </div>

  <script>
    new Vue({
      el: "#root",
      data: {
        content: "hello"
      },
      methods: {
        handleClick: function() {
          this.content = "world"
        }
      }
    })
  </script>
</body>
```

#### 3、属性绑定和双向数据绑定

- **v-bind:属性名称：**它可以往元素的属性中绑定数据，也可以动态地根据数据为元素绑定不同的样式。v-bind:可以简写成:符号。

```
<body>

  <div id="root">
    <div v-bind:title="title">{{content}}</div>
  </div>

  <script>
    new Vue({
      el: "#root",
      data: {
        content: "hello world",
        title:"I'm hello world"
      }
    })
  </script>
</body>
```
- **v-model：**该指令绑定的元素就是组件的输出结果。一般用于表单组件，当输入框中的内容改变时，文本也跟着改变。

```
<body>

  <div id="root">
    <div v-bind:title="title">{{content}}</div>
    <input v-model="vcontent" />
    <div>{{vcontent}}</div>
  </div>

  <script>
    new Vue({
      el: "#root",
      data: {
        content: "hello world",
        title:"I'm hello world",
        vcontent:"I'm v-model"
      }
    })
  </script>
</body>
```

#### 4、计算属性和侦听器

- **computed：**比较适合对多个变量或者对象进行处理后返回一个结果值，也就是数多个变量中的某一个值发生了变化则我们监控的这个值也就会发生变化。如果监测的值没有发生改变的话，那么会再次使用这个结果值时不会去再次计算，而是使用上次的缓存值。

```
<body>

  <div id="root">
    <div v-bind:title="title">{{content}}</div>
    <input v-model="vcontent" />
    <div>{{vcontent}}</div>
    姓：<input v-model="firstName" /> 名：<input v-model="lastName" />
    <div>{{fullName}}</div>
  </div>

  <script>
    new Vue({
      el: "#root",
      data: {
        content: "hello world",
        title: "I'm hello world",
        vcontent: "I'm v-model",
        firstName: '',
        lastName: ''
      },
      computed :{
        fullName: function() {
          return this.firstName + ' ' + this.lastName
        }
      }
    })
  </script>
</body>
```

- **watch：**侦听器指的是去监听某一个数据（data里面）或者计算属性（computed里面）的变化，一旦数据发生了变化，就能回调。

```
<body>

  <div id="root">
    <div v-bind:title="title">{{content}}</div>
    <input v-model="vcontent" />
    <div>{{vcontent}}</div>
    姓：<input v-model="firstName" /> 名：<input v-model="lastName" />
    <div>{{fullName}}</div>
    <div>{{count}}</div>
  </div>

  <script>
    new Vue({
      el: "#root",
      data: {
        content: "hello world",
        title: "I'm hello world",
        vcontent: "I'm v-model",
        firstName: '',
        lastName: '',
        count: 0
      },
      computed: {
        fullName: function() {
          return this.firstName + ' ' + this.lastName;
        }
      },
      watch: {
        fullName: function() {
          this.count++;
        }
      }
    })
  </script>
</body>
```

#### 5、三个常见指令

- **v-if：**控制元素是否存在（显示）的，直接把元素从DOM树中移除或者添加到DOM树中，性能略差。
- **v-show：**控制元素是否显示的，通过改变display属性来控制元素是否显示，性能略好。

```
<body>

  <div id="root">
    <div v-bind:title="title">{{content}}</div>
    <input v-model="vcontent" />
    <div>{{vcontent}}</div>
    姓：<input v-model="firstName" /> 名：<input v-model="lastName" />
    <div>{{fullName}}</div>
    <div v-if="show">{{count}}</div>
    <button @click="handleClick">toggon</button>
  </div>

  <script>
    new Vue({
      el: "#root",
      data: {
        content: "hello world",
        title: "I'm hello world",
        vcontent: "I'm v-model",
        firstName: '',
        lastName: '',
        count: 0,
        show:true
      },
      methods: {
        handleClick: function() {
          this.show=!this.show;
        }
      },
      computed: {
        fullName: function() {
          return this.firstName + ' ' + this.lastName;
        }
      },
      watch: {
        fullName: function() {
          this.count++;
        }
      }
    })
  </script>
</body>
```

- **v-for：**指令根据一组数组的选项列表进行渲染。v-for 指令需要使用 item in items 形式的特殊语法，items 是源数据数组并且 item 是数组元素迭代的别名。

```
<body>

  <div id="root">
    <div v-bind:title="title">{{content}}</div>
    <input v-model="vcontent" />
    <div>{{vcontent}}</div>
    姓：<input v-model="firstName" /> 名：<input v-model="lastName" />
    <div>{{fullName}}</div>
    <div v-if="show">{{count}}</div>
    <button @click="handleClick">toggon</button>
    <ul>
      <li v-for="(item, index) of list" :key="index">
        {{item}}
      </li>
    </ul>
  </div>

  <script>
    new Vue({
      el: "#root",
      data: {
        content: "hello world",
        title: "I'm hello world",
        vcontent: "I'm v-model",
        firstName: '',
        lastName: '',
        count: 0,
        show:true,
        list:[1,2,3]
      },
      methods: {
        handleClick: function() {
          this.show=!this.show;
        }
      },
      computed: {
        fullName: function() {
          return this.firstName + ' ' + this.lastName;
        }
      },
      watch: {
        fullName: function() {
          this.count++;
        }
      }
    })
  </script>
</body>
```

### 组件的使用

组件是可复用的Vue实例，且带有一个名字，在Vue中分为全局组件和局部组件，局部组件需要在Vue对象中注册才能被使用。

#### 1、全局组件的定义

```
<body>
	<todo-item></todo-item>
</body>

<script>
	Vue.component('todo-item',{
		template: '<li>item</li>'
	})
	
	new Vue({
		
	})
</script>
```

#### 2、局部组件的定义

```
<body>
	<todo-item></todo-item>
</body>

<script>
	var TodoItem={
		template='<li>item</li>';
	}
	
	new Vue({
		//注册局部组件，然后才能在外部使用
		components:{
			'todo-item':TodoItem
		}
	})
</script>
```