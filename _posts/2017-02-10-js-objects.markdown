---
layout:     post
title:      "JavaScript Objects"
subtitle:   ""
date:       2017-02-10
author:     "WunWun"
header-img: "img/in-post/js-the-good-parts/Javascript-good-parts.png"
tags:
    - JavaScript
---


JavaScript的简单数据类型包括**数字、字符串、布尔值（true和false）、null值**和**undefined**值。其他所有的值都是对象。

数字、字符串和布尔值“貌似”对象，因为它们拥有方法，但它们是不可变的。JavaScript中的对象是可变的键控集合（keyed collections）。

对象是属性的容器，其中每个属性都拥有名字和值。属性的名字可以是包括空字符串在内的任意字符串。属性值可以是除undefined值之外的任何值。

JavaScript 包含一种原型链的特性，允许对象继承另一个对象的属性。正确地使用它能减少对象初始化时消耗的时间和内存。

## 对象字面量

一个对象字面量就是包围在一对花括号中的零或多个“名/值”对。

```
var empty_object = {}

var stooge = {
    firstName: 'Xingwang',
    lastName: 'Chan'
}
```

在对象字面量中，如果属性名是一个合法的JavaScript标识符且不是保留字，则并不要求用引号括住属性名。

## 检索

要检索对象里包含的值，可以采用在[]后缀中括住一个字符串表达式的方式。如果字符串表达式是一个字符串字面量，而且它是一个合法的JavaScript标识符且不是保留字，则也可以用.表示法代替。优秀考虑使用.表示法，因为它更紧凑且可读性更好。

```
console.log(stooge.firstName); // Xingwang
console.log(stooge['firstName']); // Xingwang
```

如果你尝试检索一个并不存在的成员属性的值，将返回undefined。

```
console.log(stooge.age); // undefined
```

`||`运算符可以用来填充默认值：

```
var age = stooge.age || 25; 
```

尝试从undefined的成员属性中取值将会导致TypeError异常。可以通过`&&`运算符来避免错误。

```
flight.equipment                            // undefined
flight.equipment.model                      // throw "TypeError"
flight.equipment && flight.equipment.model  // undefined
```

## 更新

直接使用赋值语句更新，若不存在这个属性，则作为扩充操作。

```
stooge.firstName = 'Richard'
stooge.nickName = 'wunwun'
```

## 引用

对象通过引用来传递，他们永远不会被复制。

```
var x = stooge
x.hair = 'black'
stooge.hair //"black"
```

## 原型

每一个对象都连接到一个原型对象，并且它可以从中继承属性。所有通过字面量创建的对象都连接到`Object.prototype`，它是JavaScript中的标配对象。

当你创建一个新对象时，你可以选择某个对象作为它的原型。

原型连接在更新时是不起作用的。当对某个对象做出改变时，不会触及该对象的原型。

原型连接只有在检索值的时候才被用到。如果我们尝试去获取对象的某个属性值，但该对象没有此属性名，那么JavaScript会试着从原型对象中获取属性值，如果那个原型对象也没有该属性，那么再从它的原型中寻找，以此类推，直到终点Object.prototype。如果属性完全不存在于原型链中，那么结果就是undefined值。

原型关系是一种动态的关系。如果我们添加一个新的属性到原型中，该属性会立即对所有基于该原型创建的对象可见。

## 反射

检查对象并确定对象有什么属性是很容易的事情，只要试着去检索该属性并验证取得的值。

一个方法是是用typeof操作符，它对确定属性的类型很有帮助。

另一个方法是使用hasOwnProperty方法，若对象拥有独有的属性，它将返回true。hasOwnProperty方法不会检查原型链。


## 枚举

使用for in可以遍历一个对象中的所有属性名，包括原型链上的属性名。可以使用hasOwnProperty过滤原型链上的属性，使用typeof来排除函数。

```
for (var name in anotherStooge) {
    if (anotherStooge.hasOwnProperty(name) && typeof anotherStooge[name] !== 'function') {
        console.log(name + '--->' + anotherStooge[name])
    }
}
```

使用for in枚举一个对象中的所有属性名时，属性名出现的顺序是不确定的。如果想要确保属性以特定的顺序出现，最好的办法就是完全避免使用for in语句，而是创建一个数组，在其中以正确的顺序包含属性名。

## 删除

`delete`可以用来删除对象的属性。若对象包含该属性，则会被移除。它不会触及原型链中的任何对象。

删除对象的属性可能会让来自原型链中的属性暴露出来。

## 减少全局变量污染

最小化使用全局变量的方法之一是为应用只创建一个唯一的全局变量。

```
var MYAPP = {}

MYAPP.stooge = {
    //...
}

MYAPP.flight = {
    //...
}
```

下一章将使用闭包来进行信息隐藏，是另一种有效减少全局污染的方法。

#### 著作权声明
  
作者 [陈兴旺](http://weibo.com/xingwangchan)，首次发布于 [WunWun Blog](http://iwun.github.io/)，转载请保留以上链接
