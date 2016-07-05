---
title: JavaScript比较运算符
date: 2016-07-05 10:26:12
tags:
- javascript
categories: 
- 前端
---
### 本文介绍的是JavaScript中的比较运算符使用过程中我们需要注意的地方
<!-- more -->
当我们队Number做比较时，我们可以通过比较运算符得到一个布尔值：

```javascript
1 > 2; //false
9 >= 3; //true
5 == 5; //true	
```

实际上，JavaScript允许对任意数据类型做比较：

```javascript
false == 0;//true
false === 0;//false
```

要特别注意相等运算符`==`。JavaScript在设计时，有两种比较运算符：

第一种是`==`比较，它会自动转换数据类型再比较，很多时候，会得到非常诡异的结果；

第二种是`===`比较，它不会自动转换数据类型，如果数据类型不一致，返回`false`，如果一致，再比较。

由于JavaScript的这个设计缺陷，_不要_使用`==`比较，始终坚持`===`比较。

另一个例外是`NaN`这个特殊的Number与所有其他值都不相等，包括它自己：

```javascript
NaN === NaN; //false
```

唯一能判断`NaN`的方法是通过`isNaN()`函数：

```javascript
isNaN(NaN); //true
```

最后要注意浮点数的比较：

```javascript
1 / 3 === (1 - 2 / 3); //false
```

这不是JavaScript的设计缺陷。浮点数在运算过程中会产生误差，因为计算机无法精确表示无限循环小数。要比较两个浮点数是否相等，只能计算它们之差的绝对值，看是否小于某个阈值：

```javascript
Math.abs(1 / 3 - (1 - 2 / 3)) < 0.0000001; //true
```

