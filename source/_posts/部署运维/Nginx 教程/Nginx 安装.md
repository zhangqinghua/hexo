---
title: Nginx 安装

categories:
- 部署运维
- Nginx 教程

date: 2020-09-02 00:00:59
---
Centos 安装 Nginx 主要有 2 种方式，一种是源码包安装，另外一种是 yum 安装。

另外我们也可以把 Nginx 安装的 Docker 容器种。

## 源码包安装
1. 下载源码并解压
```shell
sudo mkdir /home/downloads
sudo cd /home/downloads
sudo wget http://nginx.org/download/nginx-1.8.0.tar.gz

sudo tar -zxvf nginx-1.8.0.tar.gz
sudo cd nginx-1.8.0  
```

1. 编译安装
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
1. 测试安装
```bash
# 检测配置文件
sudo /usr/local/nginx/sbin/nginx -t
# 启动
sudo /usr/local/nginx/sbin/nginx
# 停止
sudo /usr/local/nginx/sbin/nginx –s stop
```
1. 设置 systemctl 服务
```bash
# 在系统服务目录里创建 nginx.service 文件。
sudo vi /lib/systemd/system/nginx.service
```

nginx.service 内容如下：
```
[Unit]                                              # 服务的说明
Description=nginx                                   # 描述服务
After=network.target                                # 描述服务类别
  
[Service]                                           # 服务运行参数的设置，注意：[Service]的启动、重启、停止命令全部要求使用绝对路径
Type=forking                                        # 是后台运行的形式
ExecStart=/usr/local/nginx/sbin/nginx               # 为服务的具体运行命令
ExecReload=/usr/local/nginx/sbin/nginx -s reload    # 为重启命令
ExecStop=/usr/local/nginx/sbin/nginx -s quit        # 为停止命令
PrivateTmp=true                                     # 表示给服务分配独立的临时空间
  
[Install]                                           # 运行级别下服务安装的相关设置，可设置为多用户，即系统运行级别为3
WantedBy=multi-user.target  
```

1. 常用 systemctl 命令。
```bash
# 设置开机启动
systemctl enable nginx.service
# 停止开机自启动
systemctl disable nginx.service
# 查看服务当前状态
systemctl status nginx.service
# 启动 nginx 服务
systemctl start nginx.service　
# 停止 nginx 服务
systemctl stop nginx.service　
# 重新启动服务
systemctl restart nginx.service　
# 查看所有已启动的服务
systemctl list-units --type=service
```

## yum 安装
#### 添加源

默认情况 Centos7 中无 Nginx 的源，需要通过以下命令添加：

```bash
sudo rpm -Uvh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm
```

#### 安装 Nginx

通过 `yum search nginx` 看看是否已经添加源成功。如果成功则执行下列命令安装 Nginx：

```bash
sudo yum install -y nginx
```

#### 启动 Nginx 并设置开机自动运行

```bash
sudo systemctl start nginx.service
sudo systemctl enable nginx.service
```

#### Nginx 的默认路径
1. 配置路径
   `/etc/nginx/`

1. PID 目录
   `/var/run/nginx.pid`

1. 站点目录
   `/usr/share/nginx/html`

1. 访问日志
   `/var/log/nginx/access.log`

1. 错误日志
   `/var/log/nginx/error.log`

## docker 安装
1、取最新版的 Nginx 镜像。

```bash
docker pull nginx:latest
```

2、查看本地镜像。

执行命令后可以看到我们已经安装了最新版本的 Nginx 镜像。

```bash
docker images
```

3、运行容器。

安装完成后，我们可以使用以下命令来运行 nginx 容器：

```bash
docker run --name nginx-test -p 8080:80 -d nginx
```

参数说明：
1. `--name nginx-test`
   容器名称。

1. `-p 8080:80`
   端口进行映射，将本地 8080 端口映射到容器内部的 80 端口。

1. `-d nginx`
   设置容器在在后台一直运行。

## docker-compose 安装
1、在 /opt 目录下创建 docker_nginx 目录。

```bash
#在/opt目录下创建docker_nginx目录
cd /opt
mkdir docker_nginx
```

2、创建docker-compose.yml文件并编写下面的内容，保存退出

```bash
vim docker-compose.yml
```

```conf
version: '3.1'
services: 
  nginx:
    restart: always
    image: daocloud.io/library/nginx:latest
    container_name: nginx
    ports: 
      - 80:80
```

3、执行 docker-compose 命令编译运行。

```bash
docker-compose up -d
```