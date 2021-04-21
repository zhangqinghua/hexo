---
title: Nginx 动静分离

categories:
- 部署运维
- Nginx 教程

date: 2020-09-02 00:00:55
---
在 Web 开发中，通常来说，动态资源其实就是指那些后台资源，而静态资源就是指 HTML，JavaScript，CSS，img 等文件。

一般来说，都需要将动态资源和静态资源分开，将静态资源部署在 Nginx 上，当一个请求来的时候，如果是静态资源的请求，就直接到 Nginx 配置的静态资源目录下面获取资源，如果是动态资源的请求，Nginx 利用反向代理的原理，把请求转发给后台应用去处理，从而实现动静分离。

在使用前后端分离之后，可以很大程度的提升静态资源的访问速度，同时在开过程中也可以让前后端开发并行可以有效的提高开发时间，也可以有些的减少联调时间。

静态资源并发 = worker_processes * worker_connections / 2。

动态资源并发 = worker_processes * worker_connections / 4。

## Nginx 配置

```conf
server {
    listen       10000;
    server_name  localhost;

    # 拦截后台请求
    location / {
        proxy_pass http://localhost:8888;
        proxy_set_header X-Real-IP $remote_addr;
    }

    # 拦截静态资源
    location ~ .*\.(html|htm|gif|jpg|jpeg|bmp|png|ico|js|css)$ {
        root /Users/dalaoyang/Downloads/static;
    }
}
```