---
title: javascript之常见问题与技巧
categories:
- javascript语言
tags:
- javascript基础
- 技巧
---

### 技巧：replace替代循环
比如你要对页面上的div1,div2,div3 等进行同样的操作，不需要split成数组、考虑兼容的forEach
```javascript
"div1 div2 div3".replace(/[^ ]+/g,function(a){
    console.log(a);
    return a;
});
```

### 理解：变量提升
```javascript
//来自 Nettuts+ 的一段代码，生动的阐述了 JavaScript 中变量声明提升规则
var myvar = 'myvalue';

(function() {
    alert(myval); //undefined
    var myval = 'local value';
})();
```

### 函数参数默认值
定义一个函数，经常会需要给参数默认值的情况，常会看到以下这种方式：
```
function testFunc(name, age, isMale) {
    name = name || '张三'; //没有名称就给默认值 张三
    age = age || 18;    // 没有传入年龄就给默认年龄 18
    isMale = isMale || true; //没有传入性别就默认是true 男性 
    console.log('我的名字叫做' + name + ', 今年 ' + age + '岁了, 我是一位' + (isMale ? '男性' : '女性'));
}
```
看下来似乎没有什么问题，代码也比较简短，但是如果有这样的一个需求，需要输出这样的信息`我的名字叫做小薇，今年0岁了，我是一位女性`的信息？ 
```
testFunc('小薇', 0, false); // 打印出的结果却是这个：我的名字叫做小薇, 今年 18岁了, 我是一位男性
```
很明显这样子做是有问题的，问题就在于使用 `||` 时，左侧会有一个Boolean转换的判断， `空字符串`、`0或NaN`、`undefined`、`false` 如果你的使用值需可能是其中的一种，那你就不要使用`||`这种方式来设置默认值了。

正确的做法是使用`typeof`：
``` javascript
function multiply(a, b) {
    b = typeof b !== 'undefined' ? b : 1;  //设定b的默认值为1
    return a * b;
}
```

### this对象
1. global this
浏览器中全局对象是 `window`，`this`等价于`window`， 即`this === window`

nodejs的全局对象是 `global`， `this`等价于`global`，即`this === global`

2. function this
函数中的`this`:
    1. 如果不是用`new`调用的，在函数里使用`this`指代全局范围的`this`。
    2. 使用严格模式，函数中的`this`就会变成`undefined`，将会抛出错误。
    3. 如果使用`new`调用，`this`会变成一个新的值与全局范围的`this`脱离关系
``` javascript
var foo = 'bar';

function testThis() {
    //"use strict"
    this.foo = 'foo';
}

console.log('without the use "new":')
console.log(this.foo); //Output: bar
testThis();
console.log(this.foo); //Output: foo

console.log('use "new":')
foo = 'bar';
console.log(this.foo); //Output: bar
var instance = new testThis();
console.log(this.foo); //Output: bar
console.log(instance.foo); //Output: foo
```

3. object this
在一个对象的一个函数里，可以通过`this`来引用这个对象的其他属性。
``` javascript
var obj = {
    name: 'hello',
    say: function() { 
        console.log(this.name);
    }
};

obj.say();  //Output: hello
```
只有**相同直接父元素**的属性才能通过`this`
``` javascript
var obj = {
    name: 'hello',
    subObj: {
        subName: 'subHello',
        say: function() {
            console.log(this.subName);
            console.log(this.name); // 这里是无法找到父元素中的name，undefined
        }
    }
};

obj.subObj.say();  //Output: subHello、undefined
```
最后来个Demo测试下(下面是在浏览器上做的测试):
``` javascript
var obj = {
  id: "xyz",
  printId: function() {
    console.log('The id is '+ this.id + ', ', this);
  }
};
obj.printId();                     // 1. expect, The id is xyz,  Object {id: "xyz"}
var callback = obj.printId;
callback();                        // 2. unexpect, The id is undefined,  Window {external: ....
setTimeout(obj.printId, 1000);     // 3. unexpect, The id is undefined,  Window {external: ....
```
第1种符合预期，这和我们之前上面的例子是一样的。

第2种调用的只是一个赋值，`this`指向全局这里即`window`

第3种`this`指向`window`全局

再理解下**this关键字的指向实在函数调用的时候定义的**

要时上面的函数都达到我们的预期，我们可以采用`bind`的方式实现。比如`var callback = obj.printId.bind(obj)`
