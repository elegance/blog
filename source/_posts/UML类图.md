---
title: UML类图
date: 2017-02-04 11:01:43
tags:
---

![](/images/uml-example/uml.png)

#### 1. 类， `动物`矩形框代表一个类(Class)，类分为三层
* 第一层显示类的名称，如果是抽象类，则用斜体表示
* 第二层显示类的特性，通常是字段和属性
* 第三层是累的操作，通常是方法和行为，注意前面的符号，`+`表示`public`，`-`表示`private`，`#`表示`protected`

<div style="margin: auto;">
    <img src="/images/uml-example/1.png"/>
</div>

#### 2. 接口， `飞翔`表示一个接口
1. 第一种：
    * 第一行是接口名称，顶部有`<<interface>>`
    * 第二行是接口方法

2. 另一种接口表示方法：
    * 棒棒糖表示法，就是唐老鸭实现了‘讲人话’接口

<div style="margin: auto;">
    <img src="/images/uml-example/2.png"/>
</div>

#### 3. 继承
* `空心三角` + `实线`，鸟继承自动物

<div style="margin: auto;">
    <img src="/images/uml-example/3.png"/>
</div>

#### 4. 实现接口
* `空心三角` + `虚线`， 大雁实线了飞翔接口

<div style="margin: auto;">
    <img src="/images/uml-example/4.png"/>
</div>

#### 5. 关联关系
* `实线箭头`表示关联关系，企鹅要知道气候的变化

<div style="margin: auto;">
    <img src="/images/uml-example/5.png"/>
</div>

#### 6. 聚合关系（Aggregation）
* `空心菱形` + `实线箭头`，大雁是群居动物，每只大雁都是属于一个雁群，一个雁群可以有多只大雁，所以他们就是聚合(Aggregation)关系。**聚合表示一种弱‘拥有’关系，体现A对象可包含B对象，但B对象不是A对象的一部分。**

<div style="margin: auto;">
    <img src="/images/uml-example/6.png"/>
    <img src="/images/uml-example/7.png"/>
</div>

#### 7. 组合/合成关系（Composition）
* `实心菱形` + `实线箭头`，**组合关系是一种强拥有关系，体现了严格的部分和整体的关系。** 鸟和其翅膀是组合关系，他们是部分和整体的关系，生命周期相同。连线两端的数字称为基数，表明这一端的类可以有几个实例。关联关系、聚合关系也可以有基数。

<div style="margin: auto;">
    <img src="/images/uml-example/8.png"/>
</div>

#### 8. 依赖关系
* `虚线箭头`表示依赖关系，动物有新陈代谢、能繁殖的特征，需要依赖氧气、水等。所以动物依赖氧气和水。

<div style="margin: auto;">
    <img src="/images/uml-example/9.png"/>
    <img src="/images/uml-example/10.png"/>
</div>