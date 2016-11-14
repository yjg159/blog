---
title: React入门学习
date: 2016-11-14 22:44:42
tags: 
- javascript
- React
categories: 
- 前端
---
#### 一、模板
<!-- more -->
使用React的网页的源码结构：

```javascript
<!DOCTYPE html>
<html>
  <head>
    <script src="../build/react.js"></script>
    <script src="../build/react-dom.js"></script>
    <script src="../build/browser.min.js"></script>
  </head>
  <body>
    <div id="example"></div>
    <script type="text/babel">
      // ** Our code goes here! **
    </script>
  </body>
</html>
```

其中，在`<body>`标签中的`<script>`标签的`type`为`text/babel`，这是React独有的JSX语法，当然也可以使用`text/jsx`，这两种方式的区别在于，后者只能使用ES5的语法，而前者可以使用ES6的语法。

#### 二、ReactDOM.render()

这个方法是React最基本的方法，可以将模板转换为HTML，并插入指定的DOM节点。

```javascript
<div id="example"></div>
  <script type="text/babel">
    ReactDOM.render(
    <h1>Hello, world!</h1>,
    document.getElementById('example')
  );
  </script>
```

#### 三、JSX语法

将HTML直接写在JavaScript语言中，就是jsx的语法，允许js与html混写。

#### 四、组件

React允许讲代码封装成组件，然后像插入普通HTML标签一样直接插入。

React.createClass方法就用于生成一个组件类。

```javascript
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

注意：组件类的第一个字母必须大写，同时组件类只能包含一个顶层标签。在HelloMessage组件加入一个name属性，值为John。组件的属性可以在组件类的this.props对象上获取。添加组件属性，还有需要注意的地方，`class`属性需要写成`className`，`for`属性需要写成`htmlFor`，因为`class`和`for`是JavaScript的保留字。

#### 五、this.props.children

`this.props`对象的属性与组件的属性一一对应，但是有一个例外，`this.props.children`属性。它表示组件的所有子节点。

```javascript
var NotesList = React.createClass({
  render: function() {
    return (
      <ol>
      {
        React.Children.map(this.props.children, function (child) {
          return <li>{child}</li>;
        })
      }
      </ol>
    );
  }
});

ReactDOM.render(
  <NotesList>
    <span>hello</span>
    <span>world</span>
  </NotesList>,
  document.body
);
```

`NodeList`组件有两个`span`子节点，它们都可以通过`this.props.children`读取。

但是，`this.props.children`的值有三种可能，如果没有子节点，就是`undefined`；如果有一个子节点，数据类型就是`object`；如果有多个子节点，数据类型就是`array`。所以，用`this.props.children`的时候要小心。

React提供了一个工具方法`React.Children`来处理`this.props.children`。我们用`React.Children.map`来遍历子节点，而不用担心`this.props.children`的数据类型是`undefined`还是`object`。

#### 六、PropTypes

React组件的属性可以接受任意值，字符串、对象、函数等等都可以。有时候我们需要一种验证机制，验证别人使用组件时，提供的参数是否符合要求。

组件类的`PorpTypes`属性，就是用来验证组件实例的属性是否符合要求。

```javascript
var MyTitle = React.createClass({
  propTypes: {
    title: React.PropTypes.string.isRequired,
  },

  render: function() {
     return <h1> {this.props.title} </h1>;
   }
});
var data = 123;
ReactDOM.render(
  <MyTitle title={data} />,
  document.body
);
```

这样的话，`title`属性就通不过验证了，控制台会显示一行错误信息。

此外，`getDefaultProps`方法可以用来设置组件属性的默认值。

```javascript
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

#### 七、获取真实的DOM节点

组件并不是真实的DOM节点，而是存在于内存之中的一种数据结构，叫做虚拟DOM。只有当它插入文档以后才会变成真实的DOM。根据React的设计，都先在虚拟DOM上发生，然后再将实际变动的部分，反映在真实的DOM上，这种方法可以极大的提高网页的性能表现。

但是，有时候组要组件获取真实的DOM节点，这时就要用到ref属性。

```javascript
var MyComponent = React.createClass({
  handleClick: function() {
    this.refs.myTextInput.focus();
  },
  render: function() {
    return (
      <div>
        <input type="text" ref="myTextInput" />
        <input type="button" value="Focus the text input" onClick={this.handleClick} />
      </div>
    );
  }
});

ReactDOM.render(
  <MyComponent />,
  document.getElementById('example')
);
```

组件 `MyComponent` 的子节点有一个文本输入框，用于获取用户的输入。这时就必须获取真实的 DOM 节点，虚拟 DOM 是拿不到用户输入的。为了做到这一点，文本输入框必须有一个 `ref` 属性，然后 `this.refs.[refName]` 就会返回这个真实的 DOM 节点。

注：由于 `this.refs.[refName]` 属性获取的是真实 DOM ，所以必须等到虚拟 DOM 插入文档以后，才能使用这个属性，否则会报错。上面代码中，通过为组件指定 `Click` 事件的回调函数，确保了只有等到真实 DOM 发生 `Click` 事件之后，才会读取 `this.refs.[refName]` 属性。

#### 八、this.state

React的组件都会有一个初始状态，然后由用户互动导致状态变化，从而触发重新渲染UI。

```javascript
var LikeButton = React.createClass({
  getInitialState: function() {
    return {liked: false};
  },
  handleClick: function(event) {
    this.setState({liked: !this.state.liked});
  },
  render: function() {
    var text = this.state.liked ? 'like' : 'haven\'t liked';
    return (
      <p onClick={this.handleClick}>
        You {text} this. Click to toggle.
      </p>
    );
  }
});

ReactDOM.render(
  <LikeButton />,
  document.getElementById('example')
);
```

`getInitialState` 方法用于定义初始状态，也就是一个对象，这个对象可以通过 `this.state` 属性读取。当用户点击组件，导致状态变化，`this.setState` 方法就修改状态值，每次修改以后，自动调用 `this.render` 方法，再次渲染组件。

由于 `this.props` 和 `this.state` 都用于描述组件的特性，可能会产生混淆。一个简单的区分方法是，`this.props` 表示那些一旦定义，就不再改变的特性，而 `this.state` 是会随着用户互动而产生变化的特性。


