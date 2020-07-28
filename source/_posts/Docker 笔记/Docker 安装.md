---
title: Docker 安装

categories:
- Docker 笔记

date: 2020-07-08
---
Docker 运行需要占据 50MB 的内存。

## Centos 人工安装
#### 卸载旧版本（如果安装过旧版本的话）
```bash
[root@vultr ~]# sudo yum remove docker  docker-common docker-selinux docker-engine
Failed to set locale, defaulting to C
No match for argument: docker
No match for argument: docker-common
No match for argument: docker-selinux
No match for argument: docker-engine
No packages marked for removal.
Dependencies resolved.
Nothing to do.
Complete!
```

#### 安装需要的软件包
yum-util 提供 yum-config-manage r功能，另外两个是 devicemapper 驱动依赖的。

```bash
[root@vultr ~]# sudo yum install -y yum-utils device-mapper-persistent-data lvm2
Failed to set locale, defaulting to C
Last metadata expiration check: 0:04:08 ago on Tue Oct 22 13:46:56 2019.
Package dnf-utils-4.0.2.2-3.el8.noarch is already installed.
Package device-mapper-persistent-data-0.7.6-1.el8.x86_64 is already installed.
Package lvm2-8:2.03.02-6.el8.x86_64 is already installed.
Dependencies resolved.
Nothing to do.
Complete!
```

#### 设置yum源
```bash
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

```
#### 查看仓库中所有的 Docker 版本
```bash
[root@vultr ~]# yum list docker-ce --showduplicates | sort -r
Failed to set locale, defaulting to C
docker-ce.x86_64            3:19.03.4-3.el7                     docker-ce-stable
docker-ce.x86_64            3:19.03.3-3.el7                     docker-ce-stable
docker-ce.x86_64            3:19.03.2-3.el7                     docker-ce-stable
docker-ce.x86_64            3:19.03.1-3.el7                     docker-ce-stable
docker-ce.x86_64            3:19.03.0-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.9-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.8-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.7-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.6-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.5-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.4-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.3-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.2-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.1-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.0-3.el7                     docker-ce-stable
docker-ce.x86_64            18.06.3.ce-3.el7                    docker-ce-stable
docker-ce.x86_64            18.06.2.ce-3.el7                    docker-ce-stable
docker-ce.x86_64            18.06.1.ce-3.el7                    docker-ce-stable
docker-ce.x86_64            18.06.0.ce-3.el7                    docker-ce-stable
docker-ce.x86_64            18.03.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            18.03.0.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.12.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.12.0.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.09.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.09.0.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.06.2.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.06.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.06.0.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.03.3.ce-1.el7                    docker-ce-stable
docker-ce.x86_64            17.03.2.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.03.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.03.0.ce-1.el7.centos             docker-ce-stable
Last metadata expiration check: 0:05:32 ago on Tue Oct 22 13:46:56 2019.
Available Packages
```

#### 安装 Docker
```bash
[root@vultr ~]# sudo yum install -y docker-ce-18.03.1.ce-1.el7.centos
```

#### 启动 Docker 并加入开机启动
```bash
[root@vultr ~]# sudo systemctl start docker
[root@vultr ~]# sudo systemctl enable docker
```

#### 验证安装是否成功
有 client 和 service 两部分表示 docker 安装启动都成功了。

```bash
	[root@vultr ~]# docker version
	Client:
	 Version:	17.12.1-ce
	 API version:	1.35
	 Go version:	go1.9.4
	 Git commit:	7390fc6
	 Built:	Tue Feb 27 22:15:20 2018
	 OS/Arch:	linux/amd64
	
	Server:
	 Engine:
	  Version:	17.12.1-ce
	  API version:	1.35 (minimum version 1.12)
	  Go version:	go1.9.4
	  Git commit:	7390fc6
	  Built:	Tue Feb 27 22:17:54 2018
	  OS/Arch:	linux/amd64
	  Experimental:	false
```

## Centos 一键安装
Docker 官方为了简化安装流程，提供了一套安装脚本：

```bash
[root@vultrguest ~]# sudo curl -fsSL get.docker.com | bash
# Executing docker install script, commit: 26ff363bcf3b3f5a00498ac43694bf1c7d9ce16c
+ sh -c 'yum install -y -q yum-utils'
Failed to set locale, defaulting to C.UTF-8
+ sh -c 'yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo'
Failed to set locale, defaulting to C.UTF-8
Adding repo from: https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
+ '[' stable '!=' stable ']'
+ sh -c 'yum makecache'
Failed to set locale, defaulting to C.UTF-8
CentOS-8 - AppStream                                                                                                           16 kB/s | 4.3 kB     00:00    
CentOS-8 - Base                                                                                                                14 kB/s | 3.9 kB     00:00    
CentOS-8 - Extras                                                                                                             6.9 kB/s | 1.5 kB     00:00    
Docker CE Stable - x86_64                                                                                                     8.1 kB/s | 3.5 kB     00:00    
Metadata cache created.
+ '[' -n '' ']'
+ sh -c 'yum install -y -q docker-ce'
Failed to set locale, defaulting to C.UTF-8
warning: /var/cache/dnf/docker-ce-stable-3e5647bf4960c796/packages/docker-ce-19.03.12-3.el7.x86_64.rpm: Header V4 RSA/SHA512 Signature, key ID 621e9f35: NOKEY
Importing GPG key 0x621E9F35:
 Userid     : "Docker Release (CE rpm) <docker@docker.com>"
 Fingerprint: 060A 61C5 1B55 8A7F 742B 77AA C52F EB6B 621E 9F35
 From       : https://mirrors.aliyun.com/docker-ce/linux/centos/gpg
If you would like to use Docker as a non-root user, you should now consider
adding your user to the "docker" group with something like:

  sudo usermod -aG docker your-user

Remember that you will have to log out and back in for this to take effect!

WARNING: Adding a user to the "docker" group will grant the ability to run
         containers which can be used to obtain root privileges on the
         docker host.
         Refer to https://docs.docker.com/engine/security/security/#docker-daemon-attack-surface
         for more information.
```

国内用户还可以使用阿里云或 daocloud 提供的脚本安装：
```bash
[root@vultrguest ~]# sudo curl -sSL https://get.daocloud.io/docker | sh

[root@vultrguest ~]# sudo curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```

## MacOS 安装
Mac 用户可以使用 brew cask 安装。
```bash
brew cask install docker
```

## 国内镜像加速器
国内访问 Docker Hub 有时会遇到困难，此时可以配置镜像加速器。国内很多云服务商都提供了加速器服务。

针对 Docker 客户端版本大于 1.10.0 的用户 您可以通过修改 daemon 配置文件 /etc/docker/daemon.json（没有时新建该文件）来使用加速器：

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://xxx.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

修改后：

```bash
{
    "registry-mirrors": ["<your accelerate address>"]
}
```

## 常见问题
1. requires containerd.io >= 1.2.2-3
CentOS 8.0 安装 docker 报错：Problem: package docker-ce-3:19.03.8-3.el7.x86_64 requires containerd.io >= 1.2.2-3。

这是因为 containerd.io 版本过低，需要更新。

```bash
[root@localhost ~]# yum install -y wget
[root@localhost ~]# wget https://download.docker.com/linux/centos/7/x86_64/edge/Packages/containerd.io-1.2.6-3.3.el7.x86_64.rpm
[root@localhost ~]# yum install -y  containerd.io-1.2.6-3.3.el7.x86_64.rpm
```

1. ERROR: Unsupported distribution 'amzn'
问题：亚马逊云安装Docker提示这个异常。。。
原因：软件库太老。
解决：`sudo yum update -y && sudo yum install docker -y`