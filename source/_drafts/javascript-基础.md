---
title: javascript 基础
tags:
---
### 一切皆对象
一切（引用类型）皆对象 （undefined、number、string、boolean属于基础类型，即值类型，引用类型有比如：`Function`、`Array`、`Object`）
》》 string 这个特殊的家伙：String('str') === 'str' === new String('str') ? 

对象即是属性（属性是有名称和值的，即键值对）的集合

### 原型
函数有个`prototype` 属性，这个属性的值是一个对象，其中`constructor` 指向函数本身。

每个对象有一个隐藏属性`__proto__`，这个属性指向的其实就是创建这个对象的函数的`prototype`

[JS学习 Primitive vs Object](http://hackjutsu.com/2016/11/17/JS%E5%AD%A6%E4%B9%A0%20Primitive%20vs%20Object/)

### 闭包
https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/A_re-introduction_to_JavaScript#闭包

### Promise
jquery 也可以优雅 [大白话讲解Promise（三）搞懂jquery中的Promise](http://www.cnblogs.com/lvdabao/p/jquery-deferred.html)

### 点运算符、new运算符、函数执行这三者之间的优先级的问题
参考：[Javascript解惑之 new A.B() 与 new A().B() 的区别](http://blog.csdn.net/cuixiping/article/details/15037061)
```javascript
//这两种写法是等价的  
var d = new A;  
var d = new A();  
  
//但是下面这两种是不同的，不能混淆了：  
var d = new A.B(); //new A.B;  
var d = new A().B(); //new A().B;  
```

###  一个基础题目- 有关变量提升
出自：[这道题--致敬各位10年阿里的前端开发](https://juejin.im/post/58fdb0ddda2f60005dcb4bc1)
```javascript
function Foo() {
    getName = function () { alert (1); };
    return this;
}
Foo.getName = function () { alert (2);};
Foo.prototype.getName = function () { alert (3);};
var getName = function () { alert (4);};
function getName() { alert (5);}

//请写出以下输出结果：
Foo.getName();
getName();
Foo().getName();
getName();
new Foo.getName();
new Foo().getName();
```

### 构造函数的返回
```javascript
function Obj1() {
    this.a = 'a';
    this.b = 'b';
}

function Obj2() {
    this.a = 'a';
    this.b = 'b';
    // return new String('c');
    return 'c';
}

function Obj3() {
    this.a = 'a';
    this.b = 'b';
    return {
        a: 'b',
        b: 'a'
    };
}

console.log(new Obj1());
console.log(new Obj2());
console.log(new Obj3());
```
构造函数被`new`调用时，无`return`时则得到`this`对象，有显示`return`并且返回的是`对象`时，则得到显示返回的对象，否则将忽略显示返回值返回`this`

### 构造块作用域-函数自执行的几种写法
```javascript
// 第一个括号用来包裹函数，第二个括号来触发执行(没有签名的括号会有语法错误)
(function() {
    //alert('xxx');
})();

// 这种代码被压与其它代码压缩合，如果其他人的语句最后没有以分号“；”结尾会发生错误，于是又了以下这种优化：
;(function() { // 这个分号相当于是为前面别人的文件末尾加上一个分号“;”
    //alert('xxx');
})();

// 再后来有人觉得第一层括号可以用单个操作符（+-!~等）来代替可以节省一个字符的空间，于是就有了：
;!function() {
    //alert('xxx');
}();
```

### Function 写法 - https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function
```javascript
// new Function ([arg1[, arg2[, ...argN]],] functionBody)
(function() {

})(Function("return this")); // 此处不用new 跟以构造函数调用是一样的
``` 

### JS 的 new 到底是干什么的？-- 士兵攻击例子
https://zhuanlan.zhihu.com/p/23987456

### this 的值到底是什么？一次说清楚
https://zhuanlan.zhihu.com/p/23804247

### 前端基础功：https://zhuanlan.zhihu.com/study-fe