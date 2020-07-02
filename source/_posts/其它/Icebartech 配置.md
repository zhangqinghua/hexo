---
title: Icebartech 常用配置
tags:
- Icebartech

categories:
- 其它

date: 2019-08-27
---


## 云效配置
1. Java 配置
```bash
/data/deploy/Java/icebartech-cloudnote/package.tgz

set -e;
if [ -f "/data/deploy/Java/icebartech-cloudnote/deploy.sh" ]; 
    then /data/deploy/Java/icebartech-cloudnote/deploy.sh stop; 
fi;
mkdir -p /data/deploy/Java/icebartech-cloudnote;
tar xf /data/deploy/Java/icebartech-cloudnote/package.tgz -o -C /data/deploy/Java/icebartech-cloudnote;
chmod +x  /data/deploy/Java/icebartech-cloudnote/deploy.sh;
/data/deploy/Java/icebartech-cloudnote/deploy.sh start
```

1. Web 打包配置
```bash
/data/deploy/Web/icebartech-bookstore/icebartech-bookstore-sys/package.tgz

set -e;
mkdir -p /data/deploy/Web/icebartech-bookstore/icebartech-bookstore-sys/dist;
tar xf /data/deploy/Web/icebartech-bookstore/icebartech-bookstore-sys/package.tgz -o -C /data/deploy/Web/icebartech-bookstore/icebartech-bookstore-sys/dist;
```

1. Web 源码配置
```bash
/data/deploy/Web/icebartech-home/icebartech-home-sys/package.tgz

set -e;
mkdir -p /data/deploy/Web/icebartech-home/icebartech-home-sys;
tar xf /data/deploy/Web/home/home-sys/package.tgz -o -C /data/deploy/Web/home/home-sys;
```

1. Web 打包配置（release）
```bash
code.language=scripts
build.output=dist/*
```

1. Web 源码配置（release）
```bash
code.language=node10.x

build.output=dist
build.command=sudo cnpm install babel-loader --save && sudo cnpm install && sudo cnpm run build

```

## Agent操作
```bash
# 启动
/home/staragent/bin/staragentctl restart;
# 重启
/home/staragent/bin/staragentctl restart;
# 查看状态
/home/staragent/bin/staragentctl status;
# 卸载
1. /home/staragent/bin/staragentctl stop;
2. rm -rf /home/staragent;
3. rm /usr/sbin/staragent_sn
```

## 服务器
|IP|账号|密码|备注|
| :- |
|120.76.156.141|root|bwwIQb2mcGAGPlIL|冰棍官网服务器|
|8.129.214.203|root|8nLDwEhu@#EYXSmO|开发服务器一|
|120.77.246.50|root|Bo9rDeUA|开发服务器二|
|120.79.178.208|root|Bg360123456|洗车佬_DEV_服务器|
|120.76.102.155|root|kwAPYE6o|Jenkins、禅道等服务器|
|120.76.98.47|root|h#PkP#@0Zl^Veh0T|GitLab 服务器|
## 体育中心
git
体育中心后端                        git@code.aliyun.com:icebartech-java/icebartech-sportscenter.git     
体育中心PC端                        git@code.aliyun.com:icebartech-web/icebartech-sportscenter-pc.git   
体育中心后台管理                    git@code.aliyun.com:icebartech-web/icebartech-sportscenter-sys.git  
体育中心i深圳 （嵌入APP的网页）     git@code.aliyun.com:icebartech-web/icebartech-sportscenter-isz.git  
体育中心手机端（从手机浏览器打开）  git@code.aliyun.com:icebartech-web/icebartech-sportscenter-h5.git   


网址
PC端        http://isz.sztyzx.com.cn/pc/
i深圳       http://isz.sztyzx.com.cn/sz/ 
手机端      http://isz.sztyzx.com.cn/wechat/ 
后台管理    http://isz.sztyzx.com.cn/sys/loginPage
接口文档    http://isz.sztyzx.com.cn/api/swagger-ui.html

部署
交与[云效](https://rdc.aliyun.com/project/258421?spm=0.mix_pipeline.0.0.3fc01c05a48ynN)管理，提交代码自动部署。

Java端使用dev配置。

Web端提交dist自动部署。

Redis
database: 0
host: 120.76.102.155
port: 6379
password: XKKojoTM2hC4jHEqQDVRvvXUX6BthPLY

数据库
url: jdbc:mysql://icebartech-external.mysql.rds.aliyuncs.com:3366/sportscenter
username: sportscenter
password: sportscenter

阿里云
账号    30467132
密码    Tyzx1234

服务器
root@47.112.226.9 Bg360123456，阿里云已开放22、80、443、3306、6379等端口。

Java
Java系通过yum安装的java-1.8.0-openjdk*。

Nginx
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

## 官网
|120.76.156.141|root|bwwIQb2mcGAGPlIL|冰棍官网服务器|

#### 前端
https://www.icebartech.com/
https://admin.icebartech.com/admin


svn://120.76.102.155/icebar/common/icebar-www
svn://120.76.102.155/icebar/production/WebCode/icebar

jenkins: 正式环境 icebar

后端启动：/data/deploy/Java/tomcat/bin/startup.sh

nginx: /data/vhost/www.ice....

nginx -t
nginx -s relaod


#### 后台
http://admin.icebartech.com/admin/loginIndex

/usr/local/tomcat

启动：./usr/local/tomcat/bin/startup.sh

usr/local/nginx/conf/vhost/admin.icebartech.com.conf 

## 盆栽种植
微服务项目。

Git
Java: git@code.aliyun.com:icebartech-java/icebartech-potplant.git

网址
接口文档：http://potplant.xmzuozhuang.com/api.doc.html

管理后台：http://potplant.xmzuozhuang.com/sys/loginPage

阿里云
登录名：慢乐科技

密码：906eddga

服务器
IP 120.79.144.2(公)    实例密码：Bg360123456

部署
Java：阿里云云效，提交代码后，点击对应的应用流水线部署
前端：提交代码自动部署

数据库
url: jdbc:mysql://120.79.144.2:3306/potplant
username: root
password: 123456

Redis
启用、重启
/usr/local/redis/bin/redis-server /usr/local/redis/etc/redis.conf 
密码：Bg360123456 port: 6379

## 餐饮
https://catering.hanhuimedia.com/sys/loginPage

餐饮平台
服务器
47.112.136.181
root W0S5bJfc

tQgu6LrPMS3Xgb4

## 书展
现在的网址是这个：https://api.shenzhenshuzhan.com/pc/#/
他要 www.shenzhenshuzhan.com 这个也能访问

账号：shenzhenshuzhan
密码：10086123...

120.24.156.114       
root/123Lahmy!c

---数据库 ---
120.24.156.114 
wcmall 
wcmall

## 建筑
后台登录账号密码：测试用户：13164106093 123456
后台地址：http://www.szzhiyiruanjian.com/#/login

47.112.12.47

重置密码：Bg360123456