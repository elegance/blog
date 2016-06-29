---
title: javascript之apply与call方法
date: 2016-06-29 16:35:54
tags:
- javascript

categories:
- 技术
- 前端
---

#### apply
作用：调用函数，并用指定对象替换函数的 `this` 值，同时用指定数组替换函数的参数。 

语法：`apply([thisArg [,argArray]])`

参数：

* thisArg: 可选，要用作`this`对象的对象。

* argArray：可选，传递给函数的**参数数组**。

例：
```javascript
function callMe(arg1, arg2) {
    var s = '';

    s += 'this value: ' + this;
    s += '\n';
    for (i in callMe.arguments) {
        s += 'arguments: ' + callMe.arguments[i];
        s += '\n';
    }
    return s;
}

console.log('Original function: ')
console.log(callMe(1, 2));

console.log('Function called with apply:')
console.log(callMe.apply('otherObj', [4, 5]));

// Output: 
// Original function: 
// this value: [object Window]
// arguments: 1
// arguments: 2

// Function called with apply: 
// this value: otherObj
// arguments: 4
// arguments: 5

```
#### call
作用：调用一个对象的方法，用另一个对象替换当前对象。

语法：`call([thisObj[, arg1[, arg2[,  [, argN]]]]])`

参数：

* thisObj: 可选。将作为当前对象使用的对象。
* arg1, arg2, , argN: 可选。将被传递到该方法的参数列表。

例：
```javascript
function callMe(arg1, arg2) {
    var s = '';

    s += 'this value:' + this;
    s += '\n';
    for (i in callMe.arguments) {
        s += 'arguments:' + callMe.arguments[i];
        s += '\n';
    }
    return s;
}

console.log('Original function: \n');
console.log(callMe(1, 2));
console.log('\n');

console.log('Function called with call: \n');
console.log(callMe.call(3, 4, 5));

// Output: 
// Original function: 
// this value: [object Window]
// arguments: 1
// arguments: 2

// Function called with call: 
// this value: 3
// arguments: 4
// arguments: 5
```