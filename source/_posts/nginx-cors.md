---
title: nginx127.0.0.1与localhost跨域问题
header-img: /img/header_img/post-bg-desk.jpg
catalog: true
top: 0
tags:
  - nginx
categories:
  - work
abbrlink: 9f48
date: 2021-10-28 00:00:00
subtitle:
---
### nginx127.0.0.1与localhost跨域问题

前端vue，后端springboot，本地联调为解决跨域问题使用nginx反向代理，nginx配置如下：

```js
server {
    listen 8090;
    server_name localhost;
    #前端根路由
    location /aclui { 
    	proxy_pass   http://127.0.0.1:8031/;
    }
    #接口根路由
    location /aclrs {
    	proxy_pass   http://127.0.0.1:8032/;
    }
    ...
}
```

本地工程启动后，访问`localhost:8090/aclui`，页面显示正常，但是后台接口调用全部失败，F12发现接口请求状态码返回200，但是response为：`this request has no response data`
后端工程控制台报错如下：

```java
401 Unauthorized: [{"code":"401","content": { "unauthorized":"token is invalidate"}}] 
```

nginx error.log日志中有如下记录：

```verilog
2021/11/04 18:08:44 [error] 2540#8504: *7200 upstream prematurely closed connection while reading response header from upstream, client: 127.0.0.1, server: localhost, request: "GET /aclrs/acl/option/set?option=2 HTTP/1.1", upstream: "http://127.0.0.1:8096//acl/option/set?option=2", host: "localhost:8090", referrer: "http://localhost:8090/aclui/view/acl/option/aclOptionCfg.html
```

由于nginx中配置的是`127.0.0.1`，实际访问的地址为`localhost`，这种情况也会存在跨域的问题，造成`this request has no response data`错误，将本地访问地址修改为`127.0.0.1:8090/aclui`即可解决。