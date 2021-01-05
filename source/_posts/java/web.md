# 网络协议

## osi五层网络协议

## HTTP中GET和POST的区别



## 用Java写一个简单的静态文件的HTTP服务器

## 完整的网络请求过程是什么？

浏览器先检查本地缓存，没有向服务器发出请求，返回的静态资源添加了`Last-Modified`头信息。

后续的请求会被浏览器加上`If-Modified-Since`头信息，服务端资源如果没有修改，就返回304。浏览器从本地缓存获取资源。这就是http缓存协商。

## HttpServlet为什么不能重写service方法？

保留默认实现的缓存协商机制。如果要自定义内容变化的时间，就重写getLastModified这个方法。

禁用没有重写的post、head等方法，来提高安全性。

## cookie被禁用，如何实现session

## 什么是DNS 、记录类型:A记录、CNAME记录、AAAA记录等



## TCP保证可靠传输的三次握手过程是什么？

在TCP的连接中，数据流必须以正确的顺序送达对方。TCP的可靠性是通过顺序编号和确认（ACK）来实现的。TCP 连接是通过三次握手进行初始化的。三次握手的目的是同步连接双方的序列号和确认号并交换 TCP 窗口大小信息。第一次是客户端发起连接；第二次表示服务器收到了客户端的请求；第三次表示客户端收到了服务器的反馈。

## http/1.0 http/1.1 http/2之间的区别？

## 什么是WebSockets？

WebSocket是一种计算机通信协议，通过单个TCP连接提供全双工通信信道。

![img](https://mmbiz.qpic.cn/mmbiz_png/z1ViaEyjXTiauez9697via0LaAH28VBpfT1NkYicKQgxxLK1ER8QqWy4IImfH8iafEiayzwcqBqzyiagdOPm87ZGeayqg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1&retryload=1)

- WebSocket是双向的 -使用WebSocket客户端或服务器可以发起消息发送。
- WebSocket是全双工的 -客户端和服务器通信是相互独立的。
- 单个TCP连接 -初始连接使用HTTP，然后将此连接升级到基于套接字的连接。然后这个单一连接用于所有未来的通信
- Light -与http相比，WebSocket消息数据交换要轻得多。


\79. http 响应码 301 和 302 代表的是什么？有什么区别？

\80. forward 和 redirect 的区别？

\81. 简述 tcp 和 udp的区别？

\82. tcp 为什么要三次握手，两次不行吗？为什么？

\83. 说一下 tcp 粘包是怎么产生的？

\84. OSI 的七层模型都有哪些？

\85. get 和 post 请求有哪些区别？

\86. 如何实现跨域？

\87. 说一下 JSONP 实现原理？


\64. jsp 和 servlet 有什么区别？

\65. jsp 有哪些内置对象？作用分别是什么？

\66. 说一下 jsp 的 4 种作用域？

\67. session 和 cookie 有什么区别？

\68. 说一下 session 的工作原理？

\69. 如果客户端禁止 cookie 能实现 session 还能用吗？

\70. spring mvc 和 struts 的区别是什么？

\71. 如何避免 sql 注入？

\72. 什么是 XSS 攻击，如何避免？

\73. 什么是 CSRF 攻击，如何避免？

# 网络安全

## XSS

XSS的防御

## CSRF

## 注入攻击

SQL注入、XML注入、CRLF注入

## 文件上传漏洞

## 加密与解密

对称加密、非对称加密、哈希算法、加盐哈希算法

MD5，SHA1、DES、AES、RSA、DSA

彩虹表



## DDOS攻击

DOS攻击、DDOS攻击

memcached为什么可以导致DDos攻击、什么是反射型DDoS

如何通过Hash碰撞进行DOS攻击

## SSL、TLS，HTTPS

## 用openssl签一个证书部署到apache或nginx

