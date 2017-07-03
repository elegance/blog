---
title: Apache shiro
tags:
---

Shiro 主要关注点:
* **Authentication** : 身份认证，登录；即: 要知道某个人能不能做某事，我们首先得知道这个人是谁，身份认证/登录了，我们就知道是谁了。
* **Authorization**: 授权，权限认证；即知道了用户是谁，来检验用户是否能做某事。
* **Session Manager**: 会话管理，用户进行身份认证后，在没有退出之前，它所有的信息都在会话中，可以是Web环境，也可以是非Web环境。
* **Cryptography**: 加密，保护数据的安全性。如密码加密存储到数据库，而不是明文存储。

## 几个概念 concept
* **Subject** : 主体，代表当前“用户”，如爬虫、机器人、真实用户，即`Subject`是用户的抽象概念；
* **SecurityManager**: 安全管理器，即所有与安全有关的操作都会与`SecurityManager`交互；且它管理了所有的`Subject`
* **Realm** 中包含了用户、角色、权限等信息，`SecurityMananger`都是从`Realm`中获取安全数据，可以把`Realm`看成DataSource,即安全数据源。