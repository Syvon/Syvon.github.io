---
layout:     post
title:      "JavaScript Functions"
subtitle:   ""
date:       2017-02-10
author:     "WunWun"
header-img: "img/in-post/js-the-good-parts/Javascript-good-parts.png"
tags:
    - JavaScript
---


## 函数对象

JavaScript 中的函数就是对象。对象是"名/值"对的集合并拥有一个连接到原型对象的隐藏连接。对象字面量产生的对象连接到`Object.prototype`。函数对象连接到`Function.prototype`（该原型对象本身连接到Object.prototype）。每个函数对象在创建时会附加两个隐藏属性：函数的上下文和实现函数行为的代码。

函数对象在创建时也随配有一个prototype属性。它的值是一个拥有constructor属性且值即为该函数的对象。

因为函数是对象，所以可以像任何其他的值一样被使用。函数可以保存在变量、对象和数组中。函数可以被当做参数传递给其他函数，函数也可以再返回函数。函数也可以拥有方法。

函数的与众不同之处在于可以被调用。

## 函数字面量

函数对象通过函数字面量来创建。

```
var add = function(a, b) {
    return a + b
}
```

函数字面量包含4部分，分别是：保留字 `function`、函数名（若省略，被称为匿名函数）、参数、花括号中的语句。

## 调用

调用一个函数会暂停当前函数的执行，传递控制权和参数给新函数。除了声明时定义的形式参数，还有两个附加参数：`this`和`arguments`。参数this在面向对象编程中非常重要，它的值取决于调用的模式。JavaScript中一共有4中调用模式：**方法调用模式**、**函数调用模式**、**构造器调用模式**、**apply调用模式**。

实参和形参个数不匹配时，不会有运行时错误。实参过多时，超出的实参被忽略。形参过多时，缺失的值被替换为undefined。

### 方法调用模式

当一个函数被保存为对象的一个属性时，我们称它为一个方法。**当一个方法被调用时，this被绑定到该对象**。

```
var myObject = {
    value: 0,
    increment: function(inc) {
        this.value += typeof inc === 'number' ? inc : 1
    }
}

myObject.increment()
console.log(myObject.value) //1

myObject.increment(3)
console.log(myObject.value) //4
```

方法可以使用this访问自己所属的对象，所以它能从对象中取值或对对象进行修改。this到对象的绑定发生在调用的时候。

### 函数调用模式

当一个函数并非一个对象的属性时，那么它就是被当做一个函数来调用的。

**此时this被绑定到全局对象。**

可以在函数内创建一个属性并赋值为this来解决这个问题。如下：

```
var add = function(a, b) {
    return a + b
}

myObject.double = function() {
    var that = this
    var helper = function() {
        that.value = add(that.value, that.value)
    }
    helper()    //以函数的形式调用helper
}

myObject.double()
console.log(myObject.value) //8
```

### 构造器调用模式

如果在一个函数前面带上new来调用，那么背地里将会创建一个连接到该函数的prototype成员的新对象，同时this会被绑定到那个新对象上。

```
//创建构造器函数
var Quo = function(string) {
    this.status = string
}

//给Que的所有实例提供一个公共方法
Quo.prototype.getStatus = function() {
    return this.status
}

//实例化
var myQuo = new Quo('confused')

console.log(myQuo.getStatus()) //confused
```

一个函数，如果创建的目的就是希望结合new前缀来调用，那它就被称为**构造器函数**。

### apply调用模式

apply方法让我们构建一个参数数组传递给调用函数。它也允许我们选择this的值。apply方法接受两个参数，第一个是要绑定给this的值，第二个是参数数组。

```
var arr = [3, 4]
var sum = add.apply(null, arr)
console.log(sum) //7

var statusObject = {
    status: 'fighting'
}

var status = Quo.prototype.getStatus.apply(statusObject)
console.log(status) //fighting
```

## 参数

当函数被调用时，会得到一个arguments数组。*通过此参数可以访问所有它被调用时传递给它的参数列表*，包括那些没有被分配给函数声明时定义的形参的多余参数。这使得编写一个无须指定参数个数的函数成为可能。

```
var sum = function() {
    var i, sum = 0
    for (i = 0; i < arguments.length; i++) {
        sum += arguments[i]
    }
    return sum
}
console.log(sum(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)) //55
```

因语言的设计错误，arguments并不是一个真正的数组。是一个“类似数组”的对象。有length属性，但没有任何数组的方法。

## 返回

函数执行时遇到关闭函数体的`}`时结束。然后把控制权交还给调用该函数的程序。

`return`可以使函数提前返回，不在执行余下的语句。

函数总是会返回一个值，若没有指定，则返回`undefined`。

若函数调用时在前面加上了new前缀，且返回值不是一个对象的时候，则返回`this`（该新对象）。

## 异常

JavaScript提供了一套异常处理机制。异常是干扰程序的正常流畅的不寻常（但并非完全是出乎意料的）的事故。

```
var add2 = function(a, b) {
    if (typeof a !== 'number' || typeof b !== 'number') {
        throw {
            name: 'TypeError',
            message: 'add needs numbers'
        }
    }
    return a + b
}
console.log(add2(2, 3)) //5
console.log(add2('a', 6))
```

`throw`语句中断函数的执行。抛出一个`exception`对象，该对象包含一个用来识别异常类型的name属性和一个描述性的message属性。也可以自定义其他属性。

```
var try_it = function() {
    try {
        add2('a')
    } catch (e) {
        console.log(e.name + ': ' + e.message)
    }
}
try_it() //TypeError: add needs numbers
```

如果在`try`代码块内抛出一个异常，控制权就会跳转到它的`catch`语句中。

## 作用域

作用域控制着变量与参数的可见性及生命周期。它减少了名称冲突，并提供了自动内存管理。

在一个函数内部任何位置定义的变量，在该函数内部任何地方都可见。

JavaScript没有块级作用域，有函数作用域。

最好是在函数体的顶部，声明函数中可能用到的所有变量。

## 闭包

作用域的好处是内部函数可以访问定义他们的外部函数的参数和变量（除了this和arguments）。

之前，我们构造了一个myObject对象，它拥有一个value属性和一个increment方法。*假定我们希望保护该值不会被非法更改*。

和以对象字面量形式去初始化myObject不同，我们通过调用一个函数的形式去初始化myObject，该函数会返回一个对象字面量。函数里定义了一个value变量。*该变量对increment和getValue方法总是可用的，但函数的作用域使得它对其他程序来说是不可见的*。

```
var myObject = (function(){
    var value = 0;

    return {
        increment: function (inc) {
            value += typeof inc === 'number' ? inc : 1;
        },
        getValue: function () {
            return value;
        }
    };
    }());
```

*我们并没有把一个函数赋值给myObject。我们是把调用该函数后返回的结果赋值给它。*注意最后一行的()。**该函数返回一个包含两个方法的的对象，并且这些方法继续享有访问value变量的特权。**

下面改造前面的Quo，这个版本的status是私有属性 。

```
var quo = function(status) {
    return {
        get_status: function() {
            return status
        }
    }
}

var myQuo = quo('amazed')
console.log(myQuo.get_status()) //amazed
```

## 回调

假设有这么一个序列，由用户交互行为触发，向服务器发送请求，最终显示服务器的响应。最自然的写法可能是这样的：

```
request = prepare_the_request();
response = send_request_synchronously(request);
display(response);
```

这种方式的问题在于，网络上的同步请求会导致客户端进入假死状态。如果网络传输或服务器很慢，响应会慢到让人不可接受。

更好的方式是发起异步请求。提供一个当服务器的响应到达时随即触发的回调函数。

```
request = prepare_the_request();
send_request_asynchronously(request,function (response) {
    display(response);
    });
```

## 模块

可以使用函数和闭包来构造模块。模块是一个提供接口却隐藏状态与实现的函数或对象。

模块模式的一般形式是：一个定义了私有变量和函数的函数；利用闭包创建可以访问私有变量和函数的特权函数；最后返回这个特权函数，或者把他们保存到一个可访问到的地方。

```
var serial_maker = function () {

    var prefix = '';
    var seq = 0;

    return {
        set_prefix: function (p) {
            prefix = String(p);
        },
        set_seq: function (s) {
            seq = s;
        },
        gensym: function () {
            var result = prefix + seq;
            seq += 1;
            return result;
        }
    };
};

var seqer = serial_maker();
seqer.set_prefix('Q');
seqer.set_seq(1000);
var unique = seqer.gensym();
```

## 级联

有一些方法没有返回值。如果让方法返回this而不是默认的undefined，就可以启用级联，即连续调用。

## 柯里化（curry）

柯里化，是把多参数函数转换为一系列单参数函数并进行调用的技术。

## 记忆（memoization）

函数可以将先前操作的结果记录在某个对象里，从而避免无谓的重复运算。这种优化被称为记忆（memoization）。

```
var fibonacci = function () {
    var memo = [0,1];
    var fib = function (n) {
        var result = memo[n];
        if (typeof result !== 'number') {
            result = fib(n-1) + fib(n-2);
            memo[n] = result;
        }
        return result;
    };
    console.log('test');
    return fib;
}();
```

#### 著作权声明
  
作者 [陈兴旺](http://weibo.com/xingwangchan)，首次发布于 [WunWun Blog](http://iwun.github.io/)，转载请保留以上链接
