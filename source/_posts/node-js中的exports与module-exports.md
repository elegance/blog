---
title: node.js中的exports与module.exports
draft: true
date: 2016-10-13 17:16:26
tags:
---
Rererence for: https://cnodejs.org/topic/5231a630101e574521e45ef8

### 理解以下
1. `module.exports` 初始值为一个空对象`{}`
2. `exports` 是指向`module.exports`的引用
3. `require(xxx)`返回的是`module.exports`而不是`exports`

如果`module.exports` 指向新的对象时，`exports` 断开了与 `module.exports` 的引用，那么通过 `exports = module.exports` 让 exports 重新指向 module.exports 即可