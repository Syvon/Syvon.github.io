---
layout:     post
title:      "JavaScript Inheritance"
subtitle:   ""
date:       2017-02-11
author:     "WunWun"
header-img: "img/in-post/js-the-good-parts/Javascript-good-parts.png"
tags:
    - JavaScript
---


JavaScript是一门弱类型语言，从不需要类型转换。对象继承关系变得无关紧要。对于一个对象来说重要的是它能做什么，而不是它从哪里来。

JavaScript是一门基于原型的语言，这意味着对象直接从其他对象继承。

## 伪类

当一个函数对象被创建时，Function构造器产生的函数对象会运行类似这样的一些代码：

```
this.prototype = {constructor: this};
```

新函数对象被赋予一个prototype属性，其值包含一个constructor属性且属性值为该新函数对象。该prototype对象是存放继承特征的地方。因为JavaScript语言没有提供一种方法去确定哪个函数是打算用来作构造器的，所以每个函数都会得到一个prototype对象。constructor属性没什么用，重要的是prototype对象。

我们可以定义一个构造器并扩充它的原型。

```
var Mammal = function (name) {
    this.name = name;
};
Mammal.prototype.get_name = function () {
    return this.name;
};
Mammal.prototype.says = function () {
    return this.saying || '';
};
```

现在我们可以构造一个实例。

```
var myMammal = new Mammal('Herb the Mammal');
var name = myMammal.get_name();    //"Herb the Mammal"
```

我们可以构造另一个伪类来继承Mammal，这是通过定义它的constructor函数并替换它的prototype为一个Mammal的实例来实现的。

```
var Cat = function (name) {
    this.name = name;
    this.saying = 'meow';
};
Cat.prototype = new Mammal();
Cat.prototype.purr = function (n) {
    var i,
        s = '';
    for (i = 0; i < n; i++) {
        if (s) {
            s += '-';
        }
        s += 'r';
    }
    return s;
}
Cat.prototype.get_name = function () {
    return this.says() + ' ' + this.name + ' ' + this.says();
};

var myCat = new Cat('mimi');
var says = myCat.says();    //"meow"
var purr = myCat.purr(5);   //"r-r-r-r-r"
var name = myCat.get_name();//"meow mimi meow"
```

## 对象说明符

有时候，构造器要接受一大串参数，记住参数的顺序非常困难。这种情况下，如果我们在编写构造器时让它接受一个**简单的对象说明符**，可能会更加友好。与其这样写：

```
var myObject = maker(f, l, m, c, s);
```

不如这样写：

```
var myObject = maker({
    first: f,
    middle: m,
    last: l,
    state: s,
    city: c
    });
```

现在多个参数可以按任何顺序排列，如果构造器会聪明地使用默认值，一些参数可以忽略掉。

## 原型

基于原型的继承相比于基于类的继承，在概念上更为简单：一个新对象可以继承一个旧对象的属性。

让我们先用对象字面量去构造一个有用的对象：

```
var myMammal = {
    name : 'Herb the Mammal',
    get_name : function () {
        return this.name;
    },
    says : function () {
        return this.saying || '';
    }
};
```

一旦有了一个想要的对象，接下来可以定制新的实例：

```
var myCat = Object.create(myMammal);
myCat.name = 'mimi';
myCat.saying = 'meow';
myCat.purr = function (n) {
    var i,
        s = '';
    for (i = 0; i < n; i++) {
        if (s) {
            s += '-';
        }
        s += 'r';
    }
    return s;
};
myCat.get_name = function () {
    return this.says() + ' ' + this.name + ' ' + this.says();
};
```

这是一种差异化继承。通过定制一个新的对象，我们指明它与所基于的基本对象的区别。

## 函数化

应用模块模式保护私有属性。下面的例子中，`name`和`saying`属性是完全私有的。只有通过`get_name`和`says`两个特权方法才可以访问它们。

```
var mammal = function (spec) {
    var that = {};

    that.get_name = function () {
        return spec.name;
    };
    that.says = function () {
        return spec.saying || '';
    };

    return that;
};
var myMammal = mammal({name : 'Herb'});
```

在伪类模式里，构造器函数Cat不得不重复构造器Mammal已经完成的工作。在函数化模式中不再需要了，因为构造器Cat将会调用构造器Mammal，让Mammal去做对象创建中的大部分工作，Cat只需要关注自身差异即可。

```
var cat = function (spec) {
    spec.saying = spec.saying || 'meow';
    var that = mammal(spec);

    that.purr = function (n) {
        var i,
            s = '';
        for (i = 0; i < n; i++) {
            if (s) {
                s += '-';
            }
            s += 'r';
        }
        return s;
    };
    that.get_name = function () {
        return this.says() + ' ' + this.name + ' ' + this.says();
    };

    return that;
};
var myCat = cat({name : 'mimi'});
```

#### 著作权声明
  
作者 [陈兴旺](http://weibo.com/xingwangchan)，首次发布于 [WunWun Blog](http://iwun.github.io/)，转载请保留以上链接
