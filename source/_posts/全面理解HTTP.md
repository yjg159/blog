---
title: 全面理解HTTP
date: 2016-07-19 09:56:26
tags: 
- http
categories: 
- 网络
---
![img](https://raw.githubusercontent.com/yjg159/yjg159.github.io/master/http/1.png)

> 引言：作为一名软件工程Web专业学生，对于HTTP的熟悉掌握是必不可少的，特此做记录，打造自己的HTTP栈。

<!-- more -->
## URL与URI

我们经常接触到的就是URL了，它就是我们访问web的一个字符串地址，那么URI是什么呢？他们是什么关系呢？
URL：uniform resource location 统一资源定位符
URI：uniform resource identifier 统一资源标识符
这也就是说，URI是一种资源的标识；而URL也是一种URI，也是一种资源的标识，但它也指明了如何定位Locate到这个资源。
URI是一种抽象的资源标识，**既可以是绝对的，也可以是相对的**。但是URL是一种URI，它指明了定位的信息，必须是绝对的。

## 报文-通信的桥梁

客户端和服务器端通过相互发送**报文**进行通信，要深刻理解HTTP协议，就需要理解报文的格式和内容。

### 报文的组成

![img](https://raw.githubusercontent.com/yjg159/yjg159.github.io/master/http/2.png)

无论是请求报文还是响应报文都需要有报文首部，当然报文主体并不是必需的。
一般来说，请求报文的格式如下：

![img](https://raw.githubusercontent.com/yjg159/yjg159.github.io/master/http/3.png)

看一下百度网站的请求报文：

![img](https://raw.githubusercontent.com/yjg159/yjg159.github.io/master/http/4.png)

简单的报文形式：

```
GET / HTTP/1.1    //请求行，包含用于请求的方法，请求的URI，HTTP版本
//以下为各种首部字段
Host: www.baidu.com
Connection: keep-alive
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0...
Accept-Encoding: gzip, deflate, sdch
Accept-Language: zh-CN,zh;q=0.8
```

响应报文的格式如下：

![img](https://raw.githubusercontent.com/yjg159/yjg159.github.io/master/http/5.png)

看一下百度网站的响应报文：

![img](https://raw.githubusercontent.com/yjg159/yjg159.github.io/master/http/6.png)

```
HTTP/1.1 200 OK   //状态行，包含表明响应结果的状态码，原因短语和HTTP版本
//以下为各种首部字段
Server: bfe/1.0.8.5
Date: Tue, 06 Oct 2015 14:48:28 GMT
Content-Type: text/html;charset=utf-8
Transfer-Encoding: chunked
Connection: keep-alive
Cache-Control: private
```

## 告知服务器意图的HTTP方法

发送HTTP的方法有许多种，最常用的便是GET和POST，下面就这两种进行详细地说明。

1. **GET**
   GET方法用来请求访问URI所指定的资源，**（我想访问你的某个资源）**并不对服务器上的内容产生任何作用结果；每次GET的内容都是相同的。GET方式把请求所需要的参数放到`URL`中，直接就可以在URL中看见，有大小限制。
2. **POST**
   POST方法用来传输实体主体，目的并不是获取响应的主体内容，**（我要把这条信息告诉你）**，POST方式则是把内容放在`报文内容`中，因此只要报文的内容没有限制，它的大小就没有限制。
3. **总结**
   GET用于获取某个内容，POST用于提交某种数据请求。
   按照使用场景来说，一般用户注册的内容属于私密的，这应该使用POST方式；而针对某一内容的查询，为了快速的响应，可以使用GET方式。

## 无状态协议与Cookie

HTTP是一种无状态协议，也就是每一次发送都是一次新的开始，服务器并不知道也没有必要知道当前连接的客户端是否之前有过交集，那么当需要进行保存用户登录状态时，则出现了麻烦，这个时候使用Cookie来保存状态。
Cookie会根据服务器端发送的响应报文内的一个叫做**Set-Cookie**的首部字段，通知客户端保存Cookie（保存在自己的电脑里），当下次客户端发送请求时，**Cookie值会被添加到请求报文中发送出去。**

## 持久连接

使用浏览器浏览一个包含多张图片的HTML页面时，浏览器会发起多次请求，如图所示：

![img](https://raw.githubusercontent.com/yjg159/yjg159.github.io/master/http/7.png)

显而易见每次请求会造成**无谓的TCP连接建立和断开，增加通信量的开销。**

### 引入持久连接

持久连接的特点是，只要任意一端没有明确提出断开连接，则保持TCP连接状态。目前HTTP/1.1中默认为持久连接。

```
Connection:keep-alive
```

![img](https://raw.githubusercontent.com/yjg159/yjg159.github.io/master/http/8.png)

## 管线化

管线化可以同时并行发送多个请求，不需要一个一个等待响应了。

## 常见的状态码

![img](https://raw.githubusercontent.com/yjg159/yjg159.github.io/master/http/9.png)

## 确保安全的HTTPS

HTTP+加密+认证+完整性保护 = HTTPS
一些登陆界面和购物结算界面使用HTTPS通信，也就是改用`https://`，HTTPS说简单点就是它的通信接口部分被SSL和TLS协议代替而已。

![img](https://raw.githubusercontent.com/yjg159/yjg159.github.io/master/http/10.png)

## 身份认证

有一些网址或者服务需要用户的身份信息，因此需要随时知道这些消息，但是肯定不能每次都让用户输入用户密码，因此关于认证就有下面几种方式：

![img](https://raw.githubusercontent.com/yjg159/yjg159.github.io/master/http/11.png)

在这里主要说一下FormBase认证，也就是**表单认证**。

### 使用Cookie来管理Session

1. 客户端把用户IE和密码等登录信息放入报文的实体部分，以**POST**方式发送给服务器。
2. 服务器进行身份认证，产生SessionID，加入到Set-Cookie内，返回给客户端。
3. 客户端接收到SessionID后，将其加入Cookie，下次请求时，浏览器会自动发送Cookie。

> 在传输过程中，一种安全地保存密码方式是，先利用给密码加盐的方式增加额外信息，再使用散列hash函数计算出散列值后保存。

书籍推荐：《图解HTTP》，轻松理解更全面的HTTP知识。

文／LuckyJing（简书作者）
原文链接：http://www.jianshu.com/p/81632fea327c