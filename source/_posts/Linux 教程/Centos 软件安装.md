---
title: Centos 软件安装

categories:
- Linux 教程

date: 2020-07-09
---
## 包管理器
包管理器是在电脑中自动安装、配制、卸载和升级软件包的工具组合，使用包管理器可以大大简化在 Linux 发行版中安装软件的过程。

在 Linux 发行版中，几乎每一个发行版都有自己的包管理器。常见的有：
1. 管理 deb 软件包的 dpkg 以及它的前端 apt（使用于Debian、Ubuntu）。
1. rpm 包管理器以及它的前端 dnf（使用于 Fedora）、前端 yum（使用于 Red Hat Enterprise Linux）、前端 ZYpp（使用于 openSUSE）、前端 urpmi（使用于 Mandriva Linux、Mageia）等。



## rpm 命令
rpm 可以用来安装、卸载、升级、查询、校验软件。

#### 查询软件
```bash
# 查询软件是否安装
[root@vultrguest ~]# rpm -q nginx
package nginx is not installed

[root@vultrguest ~]# rpm -q openssh
openssh-8.0p1-4.el8_1.x86_64

# 查询软件是否安装（名称模糊查询）
[root@vultrguest ~]# rpm -qa | grep -i openssh
openssh-server-8.0p1-4.el8_1.x86_64
openssh-clients-8.0p1-4.el8_1.x86_64
openssh-8.0p1-4.el8_1.x86_64

# 列出所有安装过的软件
[root@vultrguest ~]# rpm -qa
authselect-compat-1.1-2.el8.x86_64
kmod-25-16.el8.x86_64
geolite2-country-20180605-1.el8.noarch
...

# 列出软件的安装位置
[root@vultrguest ~]# rpm -ql yum
/etc/dnf/protected.d/yum.conf
/etc/yum.conf
/etc/yum/pluginconf.d
...
```
#### 安装软件
```bash
# 安装指定软件 i: install v: verbose 显示步骤 h: 带进度条
[root@vultrguest ~]# rpm -ivh your-package.rpm

# 忽略报错，强制安装
[root@vultrguest ~]# rpm --force -ivh your-package.rpm 
```

#### 卸载软件
```bash
# 卸载指定软件。
[root@vultrguest ~]# rpm -e tree  

# 卸载指定软件。如果软件有依赖关系，需要加上 --nodeps 不检查依赖强制删除
[root@vultrguest ~]# rpm --nodeps -e tree      

# 卸载指定软件。（名称模糊匹配）
[root@vultrguest ~]# rpm -qa | grep httpd
httpd-2.2.3-31.el5.centos.4
httpd-manual-2.2.3-31.el5.centos.4
```

## yum 命令
yum 全称 Yellowdog update Modifier，是 rpm 的前端程序，可解决软件包相关依赖性，可在多个库之间定位软件包。yum repo 存储了众多 rpm 包和它们的相关元数据文件。

#### 更新源
如果使用安装软件时提示 `No match for argument: screen` 找不到此软件，则可能是没有配置软件包仓库，可以通过安装 EPEL 源解决。

```bash
[root@vultrguest ~]# yum install epel-release
```
#### 查询软件
```bash
# 显示安装包信息
[root@vultrguest ~]# yum info mysql
Last metadata expiration check: 0:30:40 ago on Thu 09 Jul 2020 03:21:40 PM UTC.
Available Packages
Name         : mysql
Version      : 8.0.17
Release      : 3.module_el8.0.0+181+899d6349
Architecture : x86_64
Size         : 11 M
Source       : mysql-8.0.17-3.module_el8.0.0+181+899d6349.src.rpm
Repository   : AppStream
Summary      : MySQL client programs and shared libraries
URL          : http://www.mysql.com
License      : GPLv2 with exceptions and LGPLv2 and BSD
Description  : MySQL is a multi-user, multi-threaded SQL database server. MySQL is a
             : client/server implementation consisting of a server daemon (mysqld)
             : and many different client programs and libraries. The base package
             : contains the standard MySQL client programs and generic MySQL files.

# （未安装）模糊查询软件 或者 yum list | grep mysql 模糊匹配名字
[root@vultrguest ~]# yum list mysql*
Last metadata expiration check: 0:31:46 ago on Thu 09 Jul 2020 03:21:40 PM UTC.
Available Packages
MySQL-zrm.noarch                                                                       3.0-23.el8                                                                                              epel     
mysql.x86_64                                                                           8.0.17-3.module_el8.0.0+181+899d6349                                                                    AppStream
mysql-common.x86_64                                                                    8.0.17-3.module_el8.0.0+181+899d6349                                                                    AppStream
mysql-devel.x86_64                                                                     8.0.17-3.module_el8.0.0+181+899d6349                                                                    AppStream
mysql-errmsg.x86_64                                                                    8.0.17-3.module_el8.0.0+181+899d6349                                                                    AppStream
mysql-libs.x86_64                                                                      8.0.17-3.module_el8.0.0+181+899d6349                                                                    AppStream
mysql-server.x86_64                                                                    8.0.17-3.module_el8.0.0+181+899d6349                                                                    AppStream
mysql-test.x86_64                                                                      8.0.17-3.module_el8.0.0+181+899d6349                                                                    AppStream
mysqltuner.noarch                                                                      1.7.17-2.git.f18a3ef.el8                                                                                epel     

# （已安装）模糊查询软件
[root@vultrguest ~]# yum list installed mysql*
Installed Packages
mysql.x86_64                                                                          8.0.17-3.module_el8.0.0+181+899d6349                                                                    @AppStream
mysql-common.x86_64                                                                   8.0.17-3.module_el8.0.0+181+899d6349                                                                    @AppStream

# （已安装）模糊查询软件，模糊匹配名字
[root@vultrguest ~]# yum list installed | grep ssh
libssh.x86_64                                 0.9.0-4.el8                                    @anaconda    
libssh-config.noarch                          0.9.0-4.el8                                    @anaconda    
openssh.x86_64                                8.0p1-4.el8_1                                  @anaconda    
openssh-clients.x86_64                        8.0p1-4.el8_1                                  @anaconda    
openssh-server.x86_64                         8.0p1-4.el8_1                                  @anaconda    
```

#### 安装软件
```bash
# 安装指定软件
[root@vultrguest ~]# yum install mysql
Last metadata expiration check: 0:41:26 ago on Thu 09 Jul 2020 03:21:40 PM UTC.
...

# 安装指定软件和相关依赖
[root@vultrguest ~]# yum install -y mysql
Last metadata expiration check: 0:41:26 ago on Thu 09 Jul 2020 03:21:40 PM UTC.
...
```
#### 升级软件

#### 卸载软件
```bash
# 卸载所有相关的软件
[root@vultrguest ~]# yum rm mysql*
Dependencies resolved.
...

Removed:
  mariadb-connector-c-config-3.0.7-1.el8.noarch                     mecab-0.996-1.module_el8.0.0+41+ca30bab6.9.x86_64                 mysql-8.0.17-3.module_el8.0.0+181+899d6349.x86_64                
  mysql-common-8.0.17-3.module_el8.0.0+181+899d6349.x86_64          mysql-errmsg-8.0.17-3.module_el8.0.0+181+899d6349.x86_64          mysql-server-8.0.17-3.module_el8.0.0+181+899d6349.x86_64         
  protobuf-lite-3.5.0-7.el8.x86_64                                 

Complete!

```

## 常见问题
