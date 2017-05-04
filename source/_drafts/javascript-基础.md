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