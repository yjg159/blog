---
title: Javascript的原型
date: 2016-06-21 14:26:01
tags: 
- javascript
categories: 
- 前端
---
### Javascript的原型

> 声明:此文章转载自网络。如需转载请标注原文地址。

当我开始学习JavaScript的对象模型时，第一反应就是难以置信。我完全被它的原型本质给弄糊涂了，毕竟这是我头一次遇到以原型为基础的语言。因为JS中有构造函数这个概念，所以我看不出使用原型能给JS带来任何的好处。我敢说你们中的大部分人也有同样的经历。

但是当我更多的使用JavaScript后，我不仅开始理解它的对象模型，甚至还喜欢上了它的一部分。感谢JavaScript让我见识到了原型语言的优雅与灵活。现在的我十分推崇原型语言，原因就是相对于以类为基础的语言，原型语言的有着更简单、更灵活的对象模型。
<!-- more -->
##### Javascript中的原型

绝大部分的指南或者教程在开始讲解JavaScript对象时，总是直接从「构造函数」讲起。我觉得这是错误的，因为过早的引入相对复杂的概念，只能让JavaScript看起来更复杂，学起来更迷惑。所以让我们暂时抛开这个概念，先讲讲原型的基础。

##### 原型链(又叫做原型继承)

JavaScript中的所有对象都有原型。当对象收到一个消息(译者注：在这里可以理解为属性查找或方法调用)，JavaScript便尝试先从对象自身查找属性，如果查不到，那么该消息就会传递给对象的原型并以此类推。这个工作机制很像以类为基础的语言中的单根继承。

![deodesign](http://cdn1.w3cplus.com/cdn/farfuture/-BKlGu1jItePrxyLq30qg_DWdQMcnM9N7X6ZQ25xVrU/mtime:1421035169/sites/default/files/styles/print_image/public/blogs/2013/js-prototypes-1.jpg)

原型链的长度没有限制，但通常来说，过长的原型链会造成代码维护与理解上的困难，因此不值得推荐。

#####  \_\_proto\_\_对象

理解JavaScript原型链最简单的方式就是通过\_\_proto\_\_属性。可惜的是，在ES 6之前，\_\_proto\_\_并不是JavaScript中的标准接口，所以一定不要在工作代码中使用它。尽管有这样的限制，这个属性使我们讲解原型更容易了。

```javascript
// 让我们先创建一个alien对象
var alien = {
  kind: 'alien'
}

// 然后是一个person对象
var person = {
  kind: 'person'
}

// 最后是一个叫做zack的对象
var zack = {};

// 设置alien为zack的原型
zack.__proto__ = alien

// zack现在与alien关联了起来
//  它继承了alien的所有属性
console.log(zack.kind); //=> ‘alien’

// 接下来将person设置为zack的原型
zack.__proto__ = person

// 这时候zack又和person关联在一起了
console.log(zack.kind); //=> ‘person’	
```


如你所见，\_\_proto\_\_属性的含义与用法简单明了。即便我们不能在工作的代码中使用\_\_proto\_\_，但我想上面那些例子已经为你理解JavaScript对象模型打下了坚实的基础。

你可以像下面这么操作，来判断一个对象是否是另一个对象的原型：

```javascript
console.log(alien.isPrototypeOf(zack))
//=> true
```



##### 原型查找是动态的

你可以在任何时候为原型增加新属性，原型链查找机制会如你所愿找到这些新属性。

```javascript
var person = {}

var zack = {}
zack.__proto__ = person

//当前zack没有kind属性
console.log(zack.kind); //=> undefined

// 让我们在person上增加kind属性
person.kind = 'person'

// 现在有反应了，因为它从person上找到了kind属性
console.log(zack.kind); //=> 'person'	
```

##### 新建/更新的属性被赋值给了对象，而不是它的原型

如果你更新一个在原型上已经存在的属性会发生什么？我们来试试：

```javascript
var person = {
  kind: 'person'
}

var zack = {}
zack.__proto__ = person

zack.kind = 'zack'

console.log(zack.kind); //=> 'zack'
// zack 现在拥有kind属性

console.log(person.kind); //=> 'person'
// person上的属性没有被修改	
```

注意：[kind]属性已经同时存在于person和zack上了。

##### Object.create

前面解释过使用__proto__为对象设置原型这个办法并不通用。所以我们介绍另外一种简单方法：Object.create()。这是在ES 5中新增的方法，对于那些古老的浏览器或者JS引擎，可以使用[es5-shim](https://github.com/kriskowal/es5-shim)来模拟。

```javascript
var person = {
  kind: 'person'
}

//  创建一个原型为person的新对象
var zack = Object.create(person);
  
console.log(zack.kind); // => ‘person’	
```

你可以向Object.create()方法中传入一个对象，用来给新生成的对象设置特定的属性：

```javascript
var zack = Object.create(person, {age: {value:  13} });
console.log(zack.age); // => ‘13’	
```

是不是觉得传进去的对象很麻烦？实际上它必须是这个样子。详细可以查看[文档](https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/Object/create)。

##### Object.getPrototype

可以使用Object.getPrototypeOf()方法来获得对象的原型：

```javascript
ar zack = Object.create(person);
Object.getPrototypeOf(zack); //=> person	
```

这里可没有Object.setPorototype这样设置原型的方法。

##### 构造函数

在JavaScript中，构造函数是用来创建原型链最常用的方法。构造函数如此流行的原因在于：它是创建类型的唯一途径。另外一个值得考虑的因素是大部分引擎都对构造函数进行了高度优化。

很不幸的是，构造函数容易让人困惑，在我看来，它是让初学者认为JavaScript难以理解的主要原因。但构造函数是JS语言中的一个重要部分，我们必须对其深刻理解。

##### 作为构造器的函数

在JavaScript中，创建函数的实例可以这么来做：

```javascript
function Foo(){}

var foo = new Foo();

//foo 现在是Foo的一个实例
console.log(foo instanceof Foo ) //=> true	
```

对函数使用关键字new，使得函数从本质上来讲表现的像工厂，即函数能够生成新的对象。后面会讲到，新创建的对象通过函数的原型与函数保持关联。在JavaScript中，我们将这个对象称为函数的实例。

##### 隐式赋值的「this」

当我们使用「new」，JavaScript以「this」关键字的形式向函数中注入了新创建对象的隐式引用。在函数运行结尾处也会隐式的返回该引用。

当我们这么做：

```javascript
function Foo() {
  this.kind = ‘foo’
}

var foo = new Foo(); 
foo.kind //=> ‘foo’	
```

实际上会相当于这么做：

```javascript
function Foo() {
  var this = {}; // 这里的this是无效的，我们这么做只是为了演示
  this.__proto__ = Foo.prototype;
  
  this.kind = ‘foo’
  
  return this;
}	
```

需要注意的是：只有在使用「new」的时候，「this」才会被赋值为新创建的对象。如果你忘记书写「new」，那么「this」会指向全局对象。忘记new是造成众多bug的原因，所以千万不要忘记写new。

有一个惯例我十分喜欢，那就是如果一个函数被用作构造函数，那么它的函数名首字母要大写，如果你忘记写new关键字，就会很容易发觉。

##### 「函数原型」

JavaScript中的每个函数都有一个特殊属性「prototype」。

```javascript
function Foo(){
}

Foo.prototype	
```

让人有些不敢相信的是：此「prototype」并非函数真正的原型(__proto__)。

```javascript
foo.__proto__ === foo.prototype //=> false	
```

如果「prototype」术语的含义如此不明确，可想而知会对使用它的人们产生多大的困扰。有一个澄清的方法我认为不错，那就是始终将函数的「prototype」属性称为「函数的原型」，永远不要称其为「原型」。

「prototype」属性指向一个对象，这个对象就是当对函数使用「new」时创建的实例的原型。听起来糊涂？用例子来解释是最容易不过的了：

```javascript
function Person(name) {
  this.name = name;
}

// 函数person拥有一个prototype属性
// 我们可以为函数的原型增加新属性
Person.prototype.kind = ‘person’

// 当我们使用new来生成新对象时
var zack = new Person(‘Zack’);

// 新对象的原型将指向person.prototype
zack.__proto__ == Person.prototype //=> true

//  在这个新对象中，我们能够访问在Person.prototype上定义的属性
zack.kind //=> person	
```

这就是关于JavaScript对象模型的绝大部分内容了。理解__proto__与function.prototype是如何关联的将会给你带来无尽的满足感，当然也可能相反。

出处：

英文原文：[http://sporto.github.com/blog/2013/02/22/a-plain-english-guide-to-javascript-prototypes/](http://sporto.github.com/blog/2013/02/22/a-plain-english-guide-to-javascript-prototypes/)

中文译文：[http://www.w3cplus.com/js/a-plain-english-guide-to-javascript-prototypes.html](http://www.w3cplus.com/js/a-plain-english-guide-to-javascript-prototypes.html)