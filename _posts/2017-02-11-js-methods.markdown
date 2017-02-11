---
layout:     post
title:      "JavaScript Methods"
subtitle:   ""
date:       2017-02-11
author:     "WunWun"
header-img: "img/in-post/js-the-good-parts/Javascript-good-parts.png"
tags:
    - JavaScript
---


JavaScript提供了一套小型的可用在标准类型上的标准方法集。

## Array

* array.concat(item...)

`array.concat(item...)`方法会产生一个新数组，它包含一份array的浅复制（shallow copy）并把一个或多个参数item附加在其后。**如果参数item是一个数组，那么它的每个元素会被分别添加**。

```
var a = ['a', 'b', 'c'];
var b = ['x', 'y', 'z'];
var c = a.concat(b, true);
// c为['a', 'b', 'c', 'x', 'y', 'z', true]
```

* array.join(separator)

`array.join(separator)`方法把一个array构造成一个字符串。默认的separator是逗号‘，’。目前在大多数情况下，对字符串连接建议首选使用+运算符，因为相比join方法，+运算符的性能更高。

```
var a = ['a', 'b', 'c'];
var c = a.join('');    // c为‘abc’
```

* array.pop()

pop方法移除array中的最后一个元素并返回该元素。如果array是empty，它会返回undefined。

```
var a = ['a', 'b', 'c'];
var c = a.pop()    //a 是['a', 'b']；c是'c'
```

* array.push(item...)

push方法把一个或多个参数item附加到一个数组的尾部。与concat方法不同的是，该方法会修改array。如果参数item是一个数组，它会把参数数组作为单个元素整个添加到数组中，并返回这个array的新长度值。

```
var a = ['a', 'b', 'c'];
var b = ['x', 'y', 'z']; 
var c = a.push(b, true)    //a 是['a', 'b', 'c', ['x', 'y', 'z'], true]；c是5
```

* array.reverse()

reverse方法反转array里的元素的顺序，并返回array本身：

```
var a = ['a', 'b', 'c'];
var b = a.reverse();
// a和b都是['c', 'b', 'a']
```

* array.shift()

shift方法移除数组array中的第1个元素并返回该元素。如果数组array为空，则会返回undefined。shift操作通常要比pop慢得多.

```
var a = ['a', 'b', 'c'];
var c = a.shift();    //a是[ 'b', 'c']; c是'a'
```

* array.slice(start, end)

slice(start, end)方法对array中的一段做浅复制。

```
var a = ['a', 'b', 'c'];
var b = a.slice(0, 1);    //['a']
var c = a.slice(1);       //['b', 'c']
var d = a.slice(1, 2);    //['b']
```

* array.sort(comparefn)

sort(comparefn)方法对array中的内容进行排序。默认比较函数把要排序的元素都视为字符串。可以使用自己的比较函数来替换默认的比较函数。你的比较函数应该接受两个参数，并且如果这两个参数相等则返回0，如果第1个参数应该排列在前面，则返回一个负数，如果第2个参数应该排列在前面，则返回一个正数。

```
var n = [4, 18, 15, 16, 23, 42];
n.sort();    //n是[15, 16, 18, 23, 4, 42]
n.sort(function (a, b) {
    return a - b;
});    //n是[4, 15, 16, 18, 23, 42]
```

* array.splice(start, deleteCount, item...)

splice(start, deleteCount, item...)方法从array中移除一个或多个元素，并用新的item替换它们。其返回一个包含被移除元素的数组。

```
var a = ['a', 'b', 'c'];
var r = a.splice(1, 1, 'ache', 'bug');
//a是["a", "ache", "bug", "c"]
//r是["b"]
```

* array.unshift(item...)

unshift(item...)方法把元素添加到数组中，但它是把item插入到array的开始部分而不是尾部，返回array的新的length。

```
var a = ['a', 'b', 'c'];
var r = a.unshift('?', '@');
//a是["?", "@", "a", "b", "c"]
//r是5
```

## Function

* function.apply(thisArg, argArray)

function.apply(thisArg, argArray)方法调用function，传递一个会被绑定到this上的对象和一个可选的数组作为参数。

## Number

* number.toExponential()

把number转换成一个指数形式的字符串。

```
Math.PI.toExponential(0);   //  "3e+0"
Math.PI.toExponential(2);   //  "3.14e+0"
Math.PI.toExponential(7);   //  "3.1415927e+0"
Math.PI.toExponential(16);  //  "3.1415926535897931e+0"
Math.PI.toExponential();    //  "3.141592653589793e+0"
```

* number.toFixed()

把number转换成一个十进制数形式的字符串。可选参数fractionDigits控制其小数点后的数字位数，必须在0～20之间，默认为0。

```
Math.PI.toFixed(0);   //  "3"
Math.PI.toFixed(2);   //  "3.14"
Math.PI.toFixed(7);   //  "3.1415927"
Math.PI.toFixed(16);  //  "3.1415926535897931"
Math.PI.toFixed();    //  "3"
```

* number.toString()

把number转换成为一个字符串。可选参数radix控制基数，默认为10。

```
Math.PI.toString(2);   // "11.001001000011111101101010100010001000010110100011"
Math.PI.toString(8);   //  "3.1103755242102643"
Math.PI.toString(16);  //  "3.243f6a8885a3"
Math.PI.toString();    //  "3.141592653589793"
```

## String

* string.charAt(pos)

string.charAt(pos)方法返回在string中pos位置处的字符。

```
var name = 'Curly';
var initial = name.charAt(0);    // 'C'
```

* string.charCodeAt(pos)

string.charCodeAt(pos)方法返回，以整数形式表示的在string中pos位置处的字符的字符码位。

```
var name = 'Curly';
var initial = name.charCodeAt(0);    // 67
```

* string.indexOf(searchString, position)

string.indexOf(searchString, position)方法在string内查找另一个字符串searchString。如果找到，返回第一个匹配字符的位置，否则返回-1。可选参数position可设置从string的某个指定位置开始查找。

```
var text = 'Mississippi';
var p = text.indexOf('ss');    // 2
p = text.indexOf('ss', 3);     // 5
p = text.indexOf('ss', 6);     // -1
```

* string.lastIndexOf(searchString, position)

string.lastIndexOf(searchString, position)方法和indexOf方法类似，只不过它是从该字符串的末尾开始查找而不是从开头。

* string.replace(searchValue, replaceValue)

被用来在正则表达式和字符串直接比较，然后用新的子串来替换被匹配的子串。

* string.slice(start, end)

摘取一个字符串区域，返回一个新的字符串。如果start参数是负数，它将与string.length相加。end参数是可选的，且默认值是string.length。

```
var text = 'and in it he says "Any damn fool could';
var a = text.slice(18);    //  ""Any damn fool could"
var b = text.slice(0, 3);  //  "and"
var c = text.slice(-5);    //  "could"
```

* string.split(separator, limit)

把string分割成片段来创建一个字符串数组。可选参数limit可以限制被分隔的片段数量。

```
var ip = '192.168.1.0';
var b = ip.split('.');        //  ["192", "168", "1", "0"]

var c = '|a|b|c|'.split('|')  //  ["", "a", "b", "c", ""]
```

#### 著作权声明
  
作者 [陈兴旺](http://weibo.com/xingwangchan)，首次发布于 [WunWun Blog](http://iwun.github.io/)，转载请保留以上链接
