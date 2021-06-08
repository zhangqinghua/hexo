---
title : Nginx 配置文件

categories:
- 部署运维
- Nginx 手册

date: 2021-05-18
---

#### 配置 http 转 https
**1. 方式一**

这种方法是 http 转发到 https，但是 http 和 https 不能用同一个配置。

```conf
server {
	listen	80;
	listen	www.xxx.com:80; #此处添加你要该链接访问的域名
	server_name  www.xxx.com  alias  xxx.com.alias;
	rewrite ^(.*) https://$server_name$1 permanent;		#此句最关键
}
```

**2. 方式二**

使用同一个端口，http 转 https。

```conf
server {
	listen	80 ssl;
	listen	www.xxx.com:80; 							#此处添加你要该链接访问的域名
	server_name  www.xxx.com  alias  xxx.com.alias;
	error_page 497 https://$host:8080$request_uri;		#此句最关键，重新定义端口
	#error_page 497 https://$http_host$request_uri;		#此句最关键，只是将http改为https，其他不变
}
```

http 和 https 是 tcp 的上层协议，当 nginx 服务器建立 tcp 连接后，根据收到的第一份数据来确定客户端是希望建立 tls 还是 http。nginx 会判断 tcp 请求的首写节内容以进行区分，如果是 0x80 或者 0x16 就可能是 ssl 或者 tls，然后尝试 https 握手。

如果端口开启了 https，但请求过来的并不是，会抛出一个 http 级别的错误，这个错误的状态码是 NGX_HTTP_TO_HTTPS，错误代码 497，然后在返回 response 中会抛出一个 400 错误(因为 497 不是标准状态码，丢给浏览器也没有用)，这时浏览器会显示"400 Bad Request,The plain HTTP request was sent to HTTPS port"。

变量说明（https://www.baidu.com/s?ie=utf-8&f=8&rsv_bp=1 为例）：
1. host：没有端口的 server_name ：www.baidu.com
1. http_host：有端口的 server_name ：www.baidu.com
1. request_uri：server_name 后面的部分 ：/s?ie=utf-8&f=8&rsv_bp=1

#### 配置微信校验文件
参考：[Nginx配置静态访问txt文件（微信校验文件）](https://blog.csdn.net/weixin_47159008/article/details/107151485)

```conf
location /32J7Idp3nE.txt {
	root /data/nginx/weixin/32J7Idp3nE.txt;
}
```

## 反向代理页面
```conf
location /alilog {
	proxy_pass https://g.alicdn.com;
}
```

自己的：https://dev.easybyte-hk.com/alilog/mlog/aplus_v2.js
官方的：https://g.alicdn.com/alilog/mlog/aplus_v2.js

如果代理 https 报错： upstream server temporarily disabled while SSL handshaking to upstream, client: xxx.xxx.xxx.xxx, server: localhost

可以这样子：

```conf
location /a-path/ {
    proxy_pass https://a-address/;
    proxy_set_header Host $proxy_host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_ssl_session_reuse off;
    proxy_ssl_server_name on;
    proxy_ssl_name $proxy_host;
    proxy_ssl_protocols TLSv1.2;
}
```

参考：[Nginx 转发 SSL 的坑](https://blog.csdn.net/ryandong/article/details/108005379)