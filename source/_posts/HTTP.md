---
abbrlink: 8c68
title: HTTP状态码
header-img: /img/header_img/post-bg-desk.jpg
catalog: true
top:
tags:
  - http
categories:
  - work
date: 2020-12-09 00:00:00
---
五大类HTTP状态码：

![image-20211209172958837.png](https://s2.loli.net/2021/12/10/EdMxp3t2HmGsweT.png)

1xx
1xx 类状态码属于提示信息，是协议处理中的一种中间状态，实际用到的比较少。
2xx
2xx 类状态码表示服务器成功处理了客户端的请求，也是我们最愿意看到的状态。
「200 OK」是最常见的成功状态码，表示一切正常。如果是非 HEAD 请求，服务器返回的响应头都
会有 body 数据。
「204 No Content」也是常见的成功状态码，与 200 OK 基本相同，但响应头没有 body 数据。
「206 Partial Content」是应用于 HTTP 分块下载或断点续传，表示响应返回的 body 数据并不是资源
的全部，而是其中的一部分，也是服务器处理成功的状态。
3xx
3xx 类状态码表示客户端请求的资源发送了变动，需要客户端用新的 URL 重新发送请求获取资源，
也就是重定向。
「301 Moved Permanently」表示永久重定向，说明请求的资源已经不存在了，需改用新的 URL 再次
访问。
「302 Found」表示临时重定向，说明请求的资源还在，但暂时需要用另一个 URL 来访问。
301 和 302 都会在响应头里使用字段 Location ，指明后续要跳转的 URL，浏览器会自动重定向新的
URL。
「304 Not Modified」不具有跳转的含义，表示资源未修改，重定向已存在的缓冲文件，也称缓存重定
向，用于缓存控制。
4xx
4xx 类状态码表示客户端发送的报文有误，服务器无法处理，也就是错误码的含义。
「400 Bad Request」表示客户端请求的报文有错误，但只是个笼统的错误。
「403 Forbidden」表示服务器禁止访问资源，并不是客户端的请求出错。
「404 Not Found」表示请求的资源在服务器上不存在或未找到，所以无法提供给客户端。
5xx
5xx 类状态码表示客户端请求报文正确，但是服务器处理时内部发生了错误，属于服务器端的错误
码。
「500 Internal Server Error」与 400 类型，是个笼统通用的错误码，服务器发生了什么错误，我们并
不知道。
「501 Not Implemented」表示客户端请求的功能还不支持，类似“即将开业，敬请期待”的意思。
「502 Bad Gateway」通常是服务器作为网关或代理时返回的错误码，表示服务器自身工作正常，访问
后端服务器发生了错误。
「503 Service Unavailable」表示服务器当前很忙，暂时无法响应服务器，类似“网络服务正忙，请稍
后重试”的意思。

```


```