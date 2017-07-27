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
* **SecurityManager**: 安全管理器，即所有与安全有关的操作都会与`SecurityManager`交互；且它管理了所有的`Subject`
* **Subject** : 主体，代表当前“用户”，如爬虫、机器人、真实用户，即`Subject`是用户的抽象概念；
* **Realm** 中包含了用户、角色、权限等信息，是验证主体的数据源，`SecurityMananger`都是从`Realm`中获取安全数据，可以把`Realm`看成DataSource,即安全数据源。

## 身份认证
在shiro中，用户提供 principals（身份）和 credentials（证明）给shiro。

* **principals** 身份，即主体的标识属性，可以是用户名、邮箱，唯一即可。一个主体可以有多个 principal，但只有一个 Primary principal，一般是用户名/手机号。
* **credentials** 证明/凭证，即只有主体才知道的安全值，如密码/证书等。

#### org.apache.shiro.realm.jdbc.JdbcRealm
默认有：`users`、`user_roles`、`roles_permissions` 三个表

#### 多个Realm 时的 Authenticator 及 AuthenticationStrategy
`Authenticator`的职责是验证账号，是Shiro API 中身份验证核心的入口点。如果验证成功则返回 AuthenticationInfo 验证信息；如果验证失败将抛出相应的 AuthenticationException。

SecurityManager 接口继承了 Authenticator，另外还有一个ModularRealmAuthenticator实现，其委托了多个Realm，验证规则通过 AuthenticationStrategy指定接口，默认提供的实现有：
* FirstSuccessfulStrategy 只要有一个Realm 验证成功即可，只返回第一个Realm身份验证成功的认证信息，其他忽略。
* AtLeastOneSuccessfulStrategy 只要有一个Realm验证成功即可，和FirstSuccess不同，返回所有Realm身份验证成功的验证信息。
* AllSuccessfulStrategy 所有Realm验证成功才是成功，且返回所有成功的认证信息。

## 授权
授权，也叫访问控制，即应用控制中谁能访问哪些资源。授权中涉及的几个关键对象：主体（Subject）、资源（Resource）、权限（Permission）、角色（Role）

资源：应用中可以访问的任何东西，如jsp、查看、编辑某些收据。

权限：安全策略中的原子授权单位，通过权限可以表示用户有没有操作某个资源的权利。

角色：角色代表了操作集合，可以理解为权限的集合，一般情况下我们授予用户角色而不是权限。

隐式角色：即直接通过角色来验证用户有没有操作权限，关心的是角色与资源，通常是硬编码，改动成本相对较大。

显示角色：在程序中通过权限控制谁能访问某个资源，关心的是细粒度的权限与资源，调整角色权限时，只需要从角色权限集合中移除/新增即可。