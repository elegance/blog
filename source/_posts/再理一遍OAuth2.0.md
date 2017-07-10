---
title: 再理一遍OAuth2.0
tags:
    - 安全
    - 授权
categories:
    - WEB
    - 安全
---
OAuth在我印象中已经啃过几次了。大多数时候，好记性还是不如烂笔头，对自己的记忆力太过自信了，当时理解了，觉的妙，但是现在只记得一个妙字了，至于其它的已经忘的差不多了...

所以最好趁自己刚理解的那一刻，赶紧的尽量用自己组织的语言记录下来。

## 概念

首先，想到`OAuth 2.0`，脑海中先弹出**三方交互**的概念，三方即如下：
* **资源提供者**
* **用户(资源拥有者)**
* **获取资源者**

在来挨个解释下，**资源提供者**，好比新浪微博提供了发微博、获取微博列表，百度网盘提供了存储文件、访问文件列表的服务，其中新浪微博、百度网盘都是资源的提供者。**用户**就是资源拥有者，具体的某条微博、网盘里的某个照片都是具体用户的，没有得到用户允许，资源提供者不可供其他应用使用。**获取资源者** 就是一些非资源提供者的应用想要访问这些资源，比如一款冲印照片的应用需要访问你网盘里的照片。

`资源提供者`、`获取资源者`、`用户(资源拥有者)`，为了引用方便，我们先将分别简称其为：`B1`、`B2`、`C`。站在`B1`的角度来说，它定义了OAuth服务，交互细节都有它来设计，整个流程都需要依赖它，其一般是拥有众多用户的高可用服务。对于`B2`来说，它“觊觎”`B1`庞大的用户群以及用户资源(为用户省心，避免再注册于在上传资源)，它的最终目的是通过`Access Token`拿到想要的资源。对于`C`来说，它其实既是`B1`的用户，也是`B2`的用户，`C`在整个过程中只需要选中同意还是不同意。

搞清楚了这些概念后，如果你要做应用，最好确定下你是要做**提供者的应用** 还是**获取者的应用**，前者相当于是要在基于现有服务，整理好资源提供一整套`OAuth 2.0`的Server，后者作为一个`OAuth 2.0`的Client相对较为简单。

## 软件架构概念
有了以上的概念我们就能从宏观上来理解为什么会有OAht2.0的存在。如果要从软件上来实现它，基于以上三者，我们还需要细分一下，如下：
* 资源提供者
    * **HTTP service**: HTTP服务提供商
    * **Resource Server**: 资源服务器，可能你需要整理归类下现有的资源，有的资源可能不需要列入资源服务器。比如[新浪微博的接口分组](http://open.weibo.com/apps/4307576/privilege)
    * **Authorization server**: 认证服务，提供资源的授权服务
* 用户(资源拥有者)
    * **Resource Owner**: 资源拥拥有者，用户
    * **User Agent**: 用户代理，这个就代表着用户，一般就是浏览器、操作界面。
* 获取资源者
    * **Third-part application**： 第三方应用程序

## 应用
* 作为第三方应用，避免注册环节，用微博、微信、QQ等登录/注册
* 作为三方应用，访问用户的资源，比如分享微博，打印网盘照片等

## 运行流程
`OAuth 2.0`的运行流程如下：(referrence from [RFC-6749](https://www.rfc-editor.org/rfc/rfc6749.txt))
```
     +--------+                               +---------------+
     |        |--(A)- Authorization Request ->|   Resource    |
     |        |                               |     Owner     |
     |        |<-(B)-- Authorization Grant ---|               
     |        |                               +---------------+
     |        |
     |        |                               +---------------+
     |        |--(C)-- Authorization Grant -->| Authorization |
     | Client |                               |     Server    |
     |        |<-(D)----- Access Token -------|               |
     |        |                               +---------------+
     |        |
     |        |                               +---------------+
     |        |--(E)----- Access Token ------>|    Resource   
     |        |                               |     Server    |
     |        |<-(F)--- Protected Resource ---|               |
     +--------+                               +---------------+
```
步骤如下：
```
(A) 用户打开客户端后，客户端要求用户授权
(B) 用户同意给予客户端授权
(C) 客户端使用上一步获得的授权，向认证服务器申请令牌。
(D) 认证服务器对客户端进行认证后，确认无误，同意发放令牌
(E) 客户端使用令牌，向资源服务器申请获取资源
(F) 资源服务器确认令牌无误，同意向客户端开放资源
```
简单的来分析下，上面的流程是以请求为载体流转的，最终拿到Access token需要经过几个步骤，但是拿到的Access Token在失效前一直可以重复使用，也就是后面对资源的请求都是请求-应答这种方式。得到Access Token前花费两次“请求-响应”，一次是询问用户是否授权，另外一次是根据授权码请求得到Access Token。这一点与我们平时的普通web应用不同，平时web的认证只需要一次登录“请求-响应”，一般请求携带着用户名、密码，响应头里设置客户端sessionid至cookie。为什么前者需要两次，而后者只需要一次呢？因为后者服务端只需要验证用户名密码是否正确，而前者则需要转个弯，三方应用先向用户申请，申请同意过后再向资源提供的应用申请。

## 授权模式
前面讲到三方应用要向用户申请授权，这个过程要怎么做才合适呢？很显然这个**授权的界面**不能由第三来做，不然就没一点意义了，这个授权的界面都是资源提供方来做的，一般在界面还会提示一些文字让用户确认该界面是资源官方提供的，比如新浪微博:
![](http://wx4.sinaimg.cn/mw690/929194b4gy1fh6smtxhlsj20h00980t5.jpg)
这个提醒大多数普通用户是忽略的，因为现在也很少有黑客用这种方式用来盗取微博的密码了，不过这种方式在仿银行的网站上存在很多。

客户端必须得到用户的授权（authorization grant），才能获取令牌（Access Token），OAuth2.0定义了四种授权方式。
* 授权码模式 （authorization code）
* 简化模式 （implicit）
* 密码模式 （resource owner password credentials）
* 客户端模式 （client credentials）

## 授权码模式
授权码模式（authorization code）是功能最完整、流程最严密的授权模式。它的特点是通过客户端的后台服务器，与“服务提供商”的认证服务器进行互动。
```
     +----------+
     | Resource |
     |   Owner  |
     |          |
     +----------+
          ^
          |
         (B)
     +----|-----+          Client Identifier      +---------------+
     |         -+----(A)-- & Redirection URI ---->|               |
     |  User-   |                                 | Authorization |
     |  Agent  -+----(B)-- User authenticates --->|     Server    |
     |          |                                 |               |
     |         -+----(C)-- Authorization Code ---<|               |
     +-|----|---+                                 +---------------+
       |    |                                         ^      v
      (A)  (C)                                        |      |
       |    |                                         |      |
       ^    v                                         |      |
     +---------+                                      |      |
     |         |>---(D)-- Authorization Code ---------'      |
     |  Client |          & Redirection URI                  |
     |         |                                             |
     |         |<---(E)----- Access Token -------------------'
     +---------+       (w/ Optional Refresh Token)
```
执行步骤：
```
1. 用户访问客户端，后者将其导向认证服务器 ----------------- 比如打开：https://passport.csdn.net/ ，点击“微博登录”，其链接导向的是：https://api.weibo.com/oauth2/authorize?client_id=2601122390&response_type=code&redirect_uri=https%3A%2F%2Fpassport.csdn.net%2Faccount%2Flogin%3Foauth_provider%3DSinaWeiboProvider
2. 用户选择是否给客户端授权
3. 假设用户选择了授权，认证服务器将用户导向至客户端事先指定的“重定向URI”（redirect URI），同时在URI后会附上一个授权码-----------------比如前面的新浪：点击授权，请求api.weibo/oauth2/autorize，响应“302”（重定向），响应头中的Location=https://passport.csdn.net/account/login?oauth_provider=SinaWeiboProvider&code=fbfd1c75c3309bb653a7c0816f919f49 ，即重定向地址。
4. 客户端根据“附带了授权码code的重定向URI”请求，即重定向的实现，客户端的后台服务器上向认证服务器申请令牌-token，这个请求-响应对于客户端与用户是不可见的
5. 认证服务器对授权码和重定向URI确认无误后，向客户端发送访问令牌（access token）和更新令牌（refresh token）
```
看下上面的一些请求细节。
第一步，客户端导向认证服务器的URI的参数：
* clietn_id: 客户端ID，这个一般是三方应用向OAuth服务提供者申请得到的，签发后一般是固定不变的。
* response_type: 授权类型，必选项，此处固定值为code。
* redirect_uri: 表示重定向URI，可选项。
* scope: 申请的权限范围，可选项。
* state: 表示客户端当前状态，自定义之，认证服务器会原封不动的返回这个值。

第二步，授权界面：
* 用户如果在资源提供方还未进行过认证(登录)，那么需要用户先登录认证，会有用户名、密码等登录表单，登录通过后再是授权界面

第三步，授权与否：
* 前面的步骤的假设是，用户选择了授权，界面上有时会有“取消”按钮，当用户拒绝授权时也可以通过URI重定向通知到客户端用户的行为

第四步，带授权码的URI重定向跳转，客户端的后台服务向认证服务器申请token
* 授权码code，为了安全一般具有时效性，通常是10分钟，且使用一次后失效。该code与client_id是一一对应关系，即其他应用使用这个code是无效的，另外重定向的URI也是与之对应相对固定的。
* 申请token的参数：（地址类似：POST //github.com/login/oauth/access_token）
    * grant_type：表示授权模式，必选项，此处为“authorization_code” （与之相呼应的是第一步请求里的 response_type=code）
    * code：表示上一步获得的授权码，必选项
    * redirect_uri：表示重定向URI，必选项，与第一步请求里的redirect_uri保持一致
    * client_id：表示客户端ID，必选项
    * client_secret：三方应用与OAuth服务商协商所得，注意保密，不要放置到客户端。（为什么不用上面的用户授权码code直接作为access token的原因其实也就在这里）

第五步，认证服务器检查token请求，响应参数如下：
* access_token：表示访问令牌
* token_type：表示令牌类型，可以是bearer或mac类型
* expires_in：表示过期时间，单位为秒。如果省略该参数，必须在其他地方设置过期时间
* refresh_token：表示更新令牌，用来获取下一次的访问令牌，可选项。
* scope：表示授权范围，如果与客户端申请的范围一致，可省略。

## 其他三种授权模式
关于这三种模式，这里只会简单的记录下，如果需要详细了解，可以点文章末尾的链接去看阮一峰的文章。
* 简化模式（implicit grant type）
    * 其特点是**跳过了授权码**这个步骤，所有步骤在浏览器中完成。 ——[疑问](https://segmentfault.com/q/1010000008974042)
* 密码模式（Resource Owner Password Credentials Grant）
    * 用户向客户端提供用户名、密码
    * 客户端使用用户提交的信息，提交给认证服务器申请令牌 **通常这个客户端必须是高度可信任的**，不然会有泄密的可能。
* 客户端模式（Client Credentials Grant）
    * 客户端使用自己的名义，而不是用户的名义，向认证服务器认证。其实整个过程不存在授权，只有一个认证，就是服务器认证客户端是可信客户端。

## 实战
[OAuth2集成——《跟我学Shiro》](http://jinnianshilongnian.iteye.com/blog/2038646)

## 关于安全
从以上的实现机制，可以看出最终是为了得到`token`。其实这个`token`与平时web会话客户端的那个 `sessionid`的本质意义是一样的，都是用户的身份标识。
为了`sessionid`的安全我们一般会做两点，后台设置的response cookie是`httpOnly`的，也就是js不可读取保证客户端安全，另外就是使用`https`保证传输通道的安全。按照这个思路我们可以类比下这个`token`该怎么在安全方面有所保障。
* 尽量保证token 不能被三方代码访问
* 请求token IP的白名单

安全问题，需要好好权衡，“三道安全门”你自己进门都会很困难，“一道安全门”足已，重要的是你要保护好自己的钥匙，插在门上不拔，再多安全门也没用。

## 总结
细节总难被记住，单个的web项目的身份认证只需要一个登录请求即可，而OAuth2.0涉及的是三方，会有两次请求，记忆的转弯点在这里，所以我的总结就是**请求授权码，请求token**。

---
## 参考
* [小胡子哥——简述 OAuth 2.0 的运作流程](http://www.barretlee.com/blog/2016/01/10/oauth2-introduce/)
* [知乎——Oauth的access token 安全么?](https://www.zhihu.com/question/20274730)
* [阮一峰——理解OAuth 2.0](http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)
* [小米-振超——指间生活——token类型、协议等](http://www.zhenchao.org/categories/protocol/)