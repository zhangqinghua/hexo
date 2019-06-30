---
title: 安装 Nginx

categories:
- Nginx 教程

date: 2019-06-30
---

## 源码安装
1. 下载源码
```shell
sudo mkdir /home/downloads
sudo cd /home/downloads
sudo wget http://nginx.org/download/nginx-1.8.0.tar.gz
```
2. 解压
```shell
sudo tar -zxvf nginx-1.8.0.tar.gz
cd nginx-1.8.0  
```
3. 编译安装
```shell
# 安装pcre库
sudo yum install -y pcre pcre-devel
# 安装zlib库
sudo yum install -y zlib zlib-devel
# 安装gcc g++（可选）
sudo yum install gcc
sudo yum install gcc-c++
# 安装SSL modules require the OpenSSL library
yum -y install openssl openssl-devel
# 安装nginx
sudo ./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module
sudo make
sudo make install
```
4. 测试
```shell
# 检测配置文件
sudo /usr/local/nginx/sbin/nginx -t
# 启动
sudo /usr/local/nginx/sbin/nginx
# 停止
sudo /usr/local/nginx/sbin/nginx –s stop
```
5. 设置systemctl restart nginx.service服务
...

## OneinStack安装

[OneinStack](https://oneinstack.com/auto/)是一个非常优秀的LNMP一键安装脚本，使用它我们可以很方便的安装Nginx。

复制安装命令然后到ssh粘贴，执行安装。

![OneinStack界面](https://upload-images.jianshu.io/upload_images/14030036-eaf0405642553fcd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/800)
