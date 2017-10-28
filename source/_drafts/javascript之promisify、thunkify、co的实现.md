---
title: javascript之promisify、thunkify、co的实现
tags:
---

参考：
* [generator-async](http://es6.ruanyifeng.com/#docs/generator-async)
* [gist-thunkify](https://gist.github.com/elegance/ef15299f8fb49cd7f982008afb3eca72)
* [gist-promisify](https://gist.github.com/elegance/9fe20250c84238d78299040ff3e40702)

promise => async/await 使用

thunkify => co => ES6 generator 使用



event loop的理解，callback => promise => generator => CSP ，Channel, Goroutines

CSP => [Clojure 风格的 JavaScript 并发编程](https://blog.oyanglul.us/javascript/clojure-core.async-essence-in-native-javascript.html)