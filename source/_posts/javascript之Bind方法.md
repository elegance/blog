---
title: javascript之Bind方法
date: 2016-06-29 16:30:54
categories:
- javascript语言

tags:
- javascript函数
- javascript基础
---

作用：**对于给定函数，创建具有与原始函数相同的主体的绑定函数。在绑定函数中，this 对象将解析为传入的对象。绑定函数具有指定的初始参数。**

语法：`function.bind(thisArg[,arg1[,arg2[,argN]]])`

例1：注意`checkNumericRange `后面不要使用**箭头函数**的方式，因为**this**在箭头函数内已经绑定作用域了，同样`call()`、`apply()`在执行箭头函数会忽略传入的第一参数，无法对`this`进行绑定
``` javascript
var checkNumericRange = function(value) {
    if (typeof value !== 'number') {
        return false;
    } else {
        return value >= this.minnum && value <= this.maxnum;
    }
};

var range = { minnum: 10, maxnum: 20 };

var boundCheckNumericRange = checkNumericRange.bind(range);

console.log(boundCheckNumericRange(12)); // true
console.log(boundCheckNumericRange(21)); // false
```
例2：下面例子中`this`与包含原始原始方法的对象不同
``` javascript
var originalObject = {
    minnum: 50,
    maxnum: 100,
    checkNumericRange: function (value) {
        console.log('This is me?', originalObject === this); // originalobject is true, bind method is false

        if (typeof value !== 'number') {
            return false;
        } else {
            return value >= this.minnum && value <= this.maxnum;
        }
    }
};

console.log(originalObject.checkNumericRange(10)); // false (compare with originalobject)

var range = { minnum: 10, maxnum: 20 };

var boundObjectWithRange = originalObject.checkNumericRange.bind(range);

console.log(boundObjectWithRange(10)); // true (expect)

```

例3：下面的例子演示如何使用`arg1[arg2, [argN]]`
``` javascript
var displayArgs = function (val1, val2, val3, val4) {
    console.log(val1 + ' ' + val2 + ' ' + val3 + ' ' + val4);
};

var emptyObject = {};

var displayArgs2 = displayArgs.bind(emptyObject, 12, 'a');

displayArgs2('b', 'c'); // Output: 12 a b c
```
