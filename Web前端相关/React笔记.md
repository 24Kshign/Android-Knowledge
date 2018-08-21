## React笔记

### 1、注意
在代码中任何使用到JavaScriptXML（JSX）的地方，最后一个script标签都要加上type="text/babel"属性，这是因为React独有的JSX语法，跟JavaScript不兼容：

```
<script type="text/babel"></script>
```

### 2、语法

#### 2.1、ReactDOM.render方法
ReactDOM.render是React的最基本方法，用于将模板转为HTML语言，并插入指定的DOM节点。

```
ReactDOM.render(
  <h1>Hello, world!</h1>,
  document.getElementById('example')
);
```
上述代码是将一个h1标签插入到example节点下。

### 3、JSX语法

```
var names = ['Alice', 'Emily', 'Kate'];

ReactDOM.render(
  <div>
  {
    names.map(function (name) {
      return <div>Hello, {name}!</div>
    })
  }
  </div>,
  document.getElementById('example')
);
```

语法规则：遇到HTML标签（以<开头），就用HTML规则解析；遇到代码块（以{开头），就用JavaScript规则解析。

### 4、组件
```
var HelloMessage = React.createClass({
  render: function() {
    return <h1>Hello {this.props.name}</h1>;
  }
});

ReactDOM.render(
  <HelloMessage name="John" />,
  document.getElementById('example')
);
```
使用React.createClass方法来生成一个组件类，每一个组件类都必须有自己的render方法，用于输出组件，这里需要注意的地方有：

>(1)、组件类的第一个字母必须大写，否则会报错
>
>(2)、组件类只能包含一个顶层标签，否则也会报错

添加组件属性时需要注意，class属性需要写成className，for属性要写成htmlFor
### 5、PropTypes
#### 5.1 组件类的PropTypes属性，就是用来验证组件实例的属性是否符合要求

```
var MyTitle = React.createClass({
  propTypes: {
    title: React.PropTypes.string.isRequired,
  },

  render: function() {
     return <h1> {this.props.title} </h1>;
   }
});
```
上面例子中就是验证属性的值是否是字符串，如果不是那么就会在控制台报错

#### 5.2 getDefaultProps
用来设置组件属性的默认值，在没有接收到传过来值的情况下，会有一个默认值

```
var MyTitle = React.createClass({
  getDefaultProps : function () {
    return {
      title : 'Hello World'
    };
  },

  render: function() {
     return <h1> {this.props.title} </h1>;
   }
});

ReactDOM.render(
  <MyTitle />,
  document.body
);
```
上述默认为Hello World
