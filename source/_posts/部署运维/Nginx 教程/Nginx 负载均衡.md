---
title: Nginx 负载均衡

categories:
- 部署运维
- Nginx 教程

date: 2020-09-02 00:00:56
---
Nginx模块一般被分成三大类：handler、filter和upstream。前面的章节中，读者已经了解了handler、filter。利用这两类模块，可以使Nginx轻松完成任何单机工作。而本章介绍的upstream模块，将使Nginx跨越单机的限制，完成网络数据的接收、处理和转发。

从本质上说，upstream属于handler，只是它不产生自己的内容，而是通过请求后端服务器得到内容，所以才称为upstream（上游）。请求并取得响应内容的整个过程已经被封装到Nginx内部，所以upstream模块只需要开发若干回调函数，完成构造请求和解析响应等具体的工作。

## 轮询
将客户端发起的请求，平均分配给每一台服务器。

```conf
upstream my_server{
    server ncthz.top:8080;
    server ncthz.top:8081;
}
server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;

	location / {
        proxy_pass http://my_server/;
    }
}
```

## 权重
会将客户端的请求，根据服务器的权重值不同，分配不同的数量。

```conf
upstream my_server{
    server ncthz.top:8080 weight=8;     # 将8/10的请求转发到此服务。 
    server ncthz.top:8081 weight=2;     # 将2/10的请求转发到此服务。
}
server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;

	location / {
        proxy_pass http://my_server/;	#tomcat首页
    }
}
```

## IP Hash
基于发起请求的客户端的IP地址不同，它始终会将请求发送到指定的服务器上。就是说如果这个客户端的请求的IP地址不变，那么处理请求的服务器将一直是同一个。

```conf
upstream my_server{
	ip_hash;
    server ncthz.top:8080 weight=10;
    server ncthz.top:8081 weight=2;
}
server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;

	location / {
        proxy_pass http://my_server/;
    }
}
```

