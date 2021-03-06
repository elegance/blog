---
title: 前端与后端的HTTP通信
categories:
  - HTTP
  - 通信
date: 2017-07-12 15:10:20
tags:
---

理解了，不用了又淡忘了，渐渐模糊混淆，感觉耗费大把的经历。

## 前端与后端的通信方式
通信，其实也就是网络通信，所以从`Chrome`的控制台的`Network`可以看到前端与后端的通信，如下图：
![](http://wx3.sinaimg.cn/mw690/929194b4gy1fhfr4aok4rj20au00udfn.jpg)

我暂时打算记录一下开发经常碰到的问题。

## HTTP 通信
这里的介绍不会一应俱全，如果有需要你可以去参考一些权威指南。跟着自己的思考，带着一些问题来记录。

#### HTTP 是无状态
就像你在围墙外面，我在围墙里面，我们通过扔纸条的方式来通信，我不知道你是谁。
无状态带来的问题很明显，如果有两个人站在墙外，我不知道你是谁，那我们就不能聊一些私密的话题了。
在互联网，乃至整个现实世界，要绝对的确认一个人的身份是不可能的，网络上的信息可以被截取、模仿，现实世界有人工智能，就像那句“我不是李开复,我是人工智能。”

#### HTTP 请求与响应
请求与响应就类似传递的信件，一部分是正文，一部分是必要的附加信息比如地址、身份等等。所以HTTP的“信件”分为了head、body部分。像一封信一样，虽然分为head、body但是他们是在一个报文内的。

HTTP/1.1 head 是文本(ASCII编码), body 可以是文本，也可以是二进制。
HTTP/2 则是一个彻底的二进制协议，头、数据体都是二进制，并且统称为“帧”（frame）：头信息帧、数据帧，因为帧可以方便的扩展，解析更为方便，HTTP/2已经定义了近十种帧。

来看下一个Ajax完整的请求报文：
```
POST https://www.tzb360.com/tzb-api/api/public/login HTTP/1.1
Host: www.tzb360.com
Connection: keep-alive
Content-Length: 47
Origin: https://www.tzb360.com
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.86 Safari/537.36
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
Accept: */*
X-Requested-With: XMLHttpRequest
Referer: https://www.tzb360.com/html/common/login.html
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.8
Cookie: userId=test01; io=j8nS4_Ph2Jaq1aSWAFX4

loginName=cqtl123&password=tk110234&imgVefCode=
```
可以看出：
* **14行前的是请求头，即head；14行之后的是请求体，即body**
* **14行本身是分隔，其内容是`空行`，即CR+LF（Carriage Return 回车， Line Feed 换行）**
    * 打字机，在纸张上打印字时，分为纵向移动、横向移动。纸张的一行打满后，横向位置回到起点，即携带纸张的车子回到起点即回车；纵方向上向下移动一行，即换行。
    * unix 结尾只有换行“\n”，window下是“\r\n”。现象就是：win下的文件在unix下会出现“^M”符号；unix下的文件到win下会连接成一行。
    * [关于回车与换行的参考](http://www.ruanyifeng.com/blog/2006/04/post_213.html?bsh_bid=705296311)
* **请求地址、cookie、请求内容长度等等都是在请求头内的**

响应头报文如下：
```
HTTP/1.1 200 OK
Server: nginx/1.9.9
Date: Tue, 11 Jul 2017 06:02:51 GMT
Content-Type: application/json
Content-Length: 928
Connection: keep-alive
Set-Cookie: TZB_SESSIONID=5788439a2c524880a5f79c6b81e97fcf; Path=/; HttpOnly

{"code":"0000","data":{},"msg":"操作成功"}
```
同样可以看出，也是使用`\r\n`作为头、体的分隔符的。

#### 发起请求与后台接收请求
编码方面这前端请求主要用：`XMLHttpRequest/fetch`来实现，后台用`Spring MVC`来实现，只涉及到POST与GET方式的请求。
首先有必要讲下请求头中易忘、易混淆的字段：
1. `Content-type` ，内容类型，即描述body，作为请求头时，一般有以下4种分类来描述body：
    * `multipart/form-data;` 类似页面表单，支持多字段、多类型。此类型请求体中会有对每个子段的的描述，就像请求体中分了多个子的请求头、请求体。
    * `application/x-www-form-urlencoded` “=”号相隔的ascii键值串，非ascii字符与特殊字符需要转码，js中可以使用`encodeURIComponent`。请求体格式：`a=value-a&b=value-b`
    * raw: `text/plain`、`application/json`、`text/xml`，这一类请求意味着请求体是一块整体，告诉服务器按照`Content-type`来解析这块整体。
    * binary: 可以上传单个文件，其body中存储了二进制内容，默认没有设置`Content-type`。
2. Request Method 
    * POST/PUT/PATCH 可以包含请求体，而其他如**GET是不包括请求体的**。页面上的`form`如果`不科学`的使用，会引起误解，比如`method=GET`，提交时会忽略action问号后的参数，自动将表单内的input转换为地址的QueryString。当`method=POST`提交时，如果action的QueryString与表单内的字段都会被提交，后台可以从两个来源中得到同样的参数，而可能值不同。

所以有必要强调的是，一般意义上的参数取法是当后台判断请求方法是`GET`时，取参从QueryString中取，当判断请求的方法是`POST`时，取参从FormData中取。后台**可以**取到`POST`请求的QueryString，GET请求是无body的。

错误的请求头设置，会导致后台不能正常的接收数据，请根据具体的场景选择请求方式、`Content-type`。

开发中会碰到一些复杂嵌套的对象需要传输，一般是前端设置`ContentType: application/json` 也就是body是一个raw，作为一个整体，后端使用`@RequestBody`描述对象。

###### XMLHttpRequest 与 fetch
使用`jquery`发起ajax请求时，会有`data`部分的参数设置，jquery发现请求是get时，会把data对象作为head的queryString提交，如果是POST，则会将data放置到body部分。

`fetch`方法默认情况下不会发送本地的cookie到服务器，注意如果需要依赖cookie，需要配置`credentials`，其配置值有：`omit`、`same-origin`、`include`，其意分别为不携带、同源携带、一直携带。更多关于fetch的信息可以参考[MDN-GlobalFetch](https://developer.mozilla.org/zh-CN/docs/Web/API/GlobalFetch/fetch)

#### HTTP 协议版本
这里再简单的记录下HTTP的几个版本的区别: （需要明白协议的实现要客户端、服务端共同完成，即新的浏览器、新的Web服务（Nginx/Apache/IIs等））
* HTTP/1.0 每个TCP只能发送一个请求。发完数据就关闭
* HTTP/1.1 (当今主流) 
    * 持久连接`Connection: keep-alive`，TCP默认不关闭，可以供多个请求复用，不用手动声明。连接没有活动，客户端就可以主动关闭连接了。规范的请求是，客户端发送最后一个请求时发送`Connection: close`，明确要求服务器关闭TCP连接。一般对于同一个域名，浏览器允许建立6个持久连接。
    * 管道机制（pipelining），同一个TCP中可发送多个请求
    * Content-Length字段，一个TCP连接中有多个请求或响应，用长度区分数据边界
    * 分块传输编码，`Content-length`的前提是在传输之前就得知道要传输的数据长度，对于一些耗时久、数据块大的操作来说，意味长时等待，这不太合理。更好的办法是产生一块数据，发送一块数据。使用`Transfer-Encoding: chunked`来表明数据是数量未定的数据块组成，每个非空数据块之前，会有一个16进制的数值，表示这个块的长度，最后一个大小为0的块，表示本轮数据传输完毕。下面有个`chunked`的响应例子：
    ```
    HTTP/1.1 200
    Content-Type: application/json;charset=UTF-8
    Transfer-Encoding: chunked
    Date: Wed, 12 Jul 2017 05:47:23 GMT

    7a
    {"paramters":{"key-a":["value-a"],"a":["value-a"],"b":["value-b"]},"queryString":"key-a=value-a","autoInject-a":"value-a"}
    0
    ```
* HTTP/2 
    * 二进制协议
    * 多工，复用的TCP不要求请求与应答顺序一一对应，避免了“队头阻塞”，这样能达到双向、实时，即多工（Multiplexing）
    * 数据流，一个连接内的数据包，可能归属不同的请求或响应，每个请求或响应的所有数据包，称之为一个数据流（stream）。每个stream都有一个编号，客户端请求的stream编号为奇数，服务端发起的stream为偶数。
    * 头信息压缩，无状态导致很多字段都是重复的，比如Cookie和UserAgent等。因此引入了头信息压缩机制（header compression），一方面压缩传输，一方面server与client共同维护一张头表，所有字段存入这个表，生成一个索引号，只发索引号来减少传输。
    * 服务器推送，允许服务器未经请求，主动向客户端发送资源。以前是请求-返回网页，解析网页源码，请求网页依赖的静态资源；而现在可以主动把这些依赖的静态资源随着网页一起发给客户端。
