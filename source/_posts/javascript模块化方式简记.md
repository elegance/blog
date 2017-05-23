---
title: javascript模块化方式简记
tags:
  - javascript
  - 模块化
  - CommonJS
  - AMD
  - CMD
date: 2017-05-23 11:19:46
---


##### `CommonJS` 规范
`node.js`是服务端的编程，需要与操作系统、其他应用互动，需要模块化，否则无法编程，而在浏览器其复杂性有限，没有模块化也不是特别大的问题。而`node.js`的模块系统就是操作`CommonJS`规范实现的。
```js
// 文件A
module.exports = {};

// 文件B
var b = require('./A');
```

##### `AMD` 规范
有了服务端模块化后，很自然地，大家就想要客户端模块化。而且最好能与服务端兼容，模块都不用修改，在服务器和浏览器都可以运行。但是**浏览器加载`模块文件`是跨网络的**，加载可能会造成假死，后面的代码无法运行。而服务端是在本地硬盘的，这对服务端不是问题。因此，浏览器端的模块，不能采用"同步加载"（synchronous），只能采用"异步加载"（asynchronous）。这就是AMD规范诞生的背景。

`AMD`是"Asynchronous Module Definition"，意识就是“异步模块定义”。它采用异步方式加载模块，模块的加载不影响他后面语句的运行。所有依赖这个模块的语句，都定义在回调函数中，等模块加载完成后，这个回调函数才会运行。
```js
require(['a', 'b'], function(a, b) { // 依赖前置，提前执行
    a.xx();
    b.xx();
});
```

[`require.js`](http://requirejs.org/)是`AMD`规范的一个实现。

##### `CMD` 规范
`CMD` 是 `SeaJS` 在推广过程中对模块定义的规范化产出
```js
define(function(require, exports, module) {
    var a = require('./a');
    a.xxx();
    ...
    var b = require('./b'); //依赖就近书写
    b.xxx();
});
```


#### 静态加载与动态加载
`es6`之前模块加载的两种方式
* 静态加载：在编译阶段进行，把所有需要的依赖打包到一个文件
* 动态加载：在运行时加载

`AMD`标准是动态加载的代表，而`CommonJS`是静态加载的代表。

`AMD`主要用在浏览器上，是异步加载的，而`NodeJS`在服务端，同步加载的方式更易被接收，所以用的是`CommonJS`。


##### `ES6`
`ES6`采用哪种加载机制了？ `ES6`既希望用简单的声明来完成静态加载，有不愿放弃动态加载的特性，而这两种方式几乎不能简单的同时实现，所以`ES6`提供了两种独立的模块加载方法。

1. 声明的方式
```js
import {foo} from some_module;
```

2. 通过`System.import` API
```js
System.import('some_module)
    .then(some_module => {
        // ...
    })
    .catch(error => {
        // ...
    });
```

模块导出：
```js
// some_module.js
export function abc() {} // export 一个命名 function
export default function() {} // export default function
export num = 123 // export 一个数值
export obj = {}
export { obj as default }

// import
import exp from 'some_module' // default export
import {default as myModule} from 'some_module' // rename
import {abc, num, obj} from 'some_module'
```

参考自：
* [写了十年JS却不知道模块化为何物？](https://blog.wilddog.com/?p=587)
* [Sea.js 与 RequireJS 的异同](https://github.com/seajs/seajs/issues/277)
* [Javascript模块化编程（二）：AMD规范](http://www.ruanyifeng.com/blog/2012/10/asynchronous_module_definition.html)