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