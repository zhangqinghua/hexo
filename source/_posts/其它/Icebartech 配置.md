---
title: Icebartech 常用配置
tags:
- Icebartech

categories:
- 其它

date: 2019-08-27
---

测试

## 云效配置
1. Java 配置
```bash
/data/deploy/Java/icebartech-cloudnote/package.tgz

set -e;
if [ -f "/data/deploy/Java/icebartech-cloudnote/deploy.sh" ]; then /data/deploy/Java/icebartech-cloudnote/deploy.sh stop; fi;
mkdir -p /data/deploy/Java/icebartech-cloudnote;
tar xf /data/deploy/Java/icebartech-cloudnote/package.tgz -o -C /data/deploy/Java/icebartech-cloudnote;
chmod +x  /data/deploy/Java/icebartech-cloudnote/deploy.sh;
/data/deploy/Java/icebartech-cloudnote/deploy.sh start
```

1. Web 打包配置
```bash
/data/deploy/Web/icebartech-home/icebartech-home-sys/package.tgz

set -e;
mkdir -p /data/deploy/Web/icebartech-home/icebartech-home-sys/dist;
tar xf /data/deploy/Web/icebartech-home/icebartech-home-sys/package.tgz -o -C /data/deploy/Web/icebartech-home/icebartech-home-sys/dist;
```

1. Web 源码配置
```bash
/data/deploy/Web/icebartech-home/icebartech-home-sys/package.tgz

set -e;
mkdir -p /data/deploy/Web/icebartech-home/icebartech-home-sys;
tar xf /data/deploy/Web/icebartech-home/icebartech-home-sys/package.tgz -o -C /data/deploy/Web/icebartech-home/icebartech-home-sys;
```

## 项目数据

1. 阿里云服务器
    120.76.156.141
    root CQypEz5W
    ftp ice REtaCf
    redis Yah6lkNoHm1VtoeZChyzinevnPtkrUa6
    Mysql root 3fBsKHwVuwo1gspj

    120.76.100.114
    root CzjdqsaW
    redis ZUravFit3fxQJ0VlzwxW8UEmEmK3LfVE
    Mysql root o7dP6ftMSOqKKsRP

    120.76.98.47
    root VsrmIb1s
    redis z0fMIYf3KbGxm2bAml8Acqe71JeIehC8
    Mysql root BgQ0EJktyVNdAqWG

    120.76.102.155
    root kwAPYE6o
    redis XKKojoTM2hC4jHEqQDVRvvXUX6BthPLY
    Mysql root BgQ0EJktyVNdAqWG
    jenkins admin 9TlbyqHS

    120.77.246.50
    root Bo9rDeUA

1. 手机膜
    深圳服务器
    香港服务器

1. 爱美丽

1. 名博
    https://mingbo.mib2019.com/api/swagger-ui.html
    https://mingbo.mib2019.com/sys

1. 奥尼
    服务器1 ssh -22 root@218.17.171.114 aoni.com 
    服务器2 ssh -23 root@218.17.171.114 aoni.com 

## 体育中心

#### git
体育中心后端                      git@code.aliyun.com:icebartech-java/icebartech-sportscenter.git     
体育中心PC端                        git@code.aliyun.com:icebartech-web/icebartech-sportscenter-pc.git   
体育中心后台管理                    git@code.aliyun.com:icebartech-web/icebartech-sportscenter-sys.git  
体育中心i深圳 （嵌入APP的网页）     git@code.aliyun.com:icebartech-web/icebartech-sportscenter-isz.git  
体育中心手机端（从手机浏览器打开）  git@code.aliyun.com:icebartech-web/icebartech-sportscenter-h5.git   


#### 网址
PC端        http://isz.sztyzx.com.cn/pc/ 
i深圳       http://isz.sztyzx.com.cn/sz/ 
手机端      http://isz.sztyzx.com.cn/wechat/ 
后台管理    http://isz.sztyzx.com.cn/sys/loginPage
接口文档    http://isz.sztyzx.com.cn/api/swagger-ui.html

#### 部署
交与[云效](https://rdc.aliyun.com/project/258421?spm=0.mix_pipeline.0.0.3fc01c05a48ynN)管理，提交代码自动部署。

Java端使用dev配置。

#### Redis
database: 0
host: 120.76.102.155
port: 6379
password: XKKojoTM2hC4jHEqQDVRvvXUX6BthPLY

#### 数据库
url: jdbc:mysql://icebartech-external.mysql.rds.aliyuncs.com:3366/sportscenter
username: sportscenter
password: sportscenter

#### 阿里云
账号    30467132
密码    Tyzx1234

#### 服务器
root@47.112.226.9 Bg360123456，阿里云已开放22、80、443、3306、6379等端口。

#### Java
Java系通过yum安装的java-1.8.0-openjdk*。

#### Nginx
Nginx系源码安装，下载包位于/home/downlaods/nginx-1.8.0，安装位于/usr/local/nginx。

配置文件位于/usr/local/nginx/conf/nginx.conf、/data/vhost/*。

```bash
# 启用
/usr/local/nginx/sbin/nginx

# 校验配置文件
/usr/local/nginx/sbin/nginx -t

# 重启
/usr/local/nginx/sbin/nginx -s reload
```
