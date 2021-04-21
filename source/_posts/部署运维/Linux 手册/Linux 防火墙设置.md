---
title: Linux 防火墙设置

categories:
- 部署运维
- Linux 手册

date: 2021-04-08 00:40:56
---

## systemctl
`systemctl` 是 CentOS 7 的服务管理工具中主要的工具，它融合之前 `service` 和 `chkconfig` 的功能于一体。

```bash
# 启动一个服务
systemctl start firewalld.service
# 关闭一个服务
systemctl stop firewalld.service
# 重启一个服务
systemctl restart firewalld.service
# 显示一个服务的状态
systemctl status firewalld.service
# 在开机时启用一个服务
systemctl enable firewalld.service
# 在开机时禁用一个服务
systemctl disable firewalld.service
# 查看服务是否开机启动
systemctl is-enabled firewalld.service
# 查看已启动的服务列表
systemctl list-unit-files|grep enabled
# 查看启动失败的服务列表
systemctl --failed
```

## firewall基本使用
```bash
# 启动
systemctl start firewalld
# 关闭
systemctl stop firewalld
# 查看状态
systemctl status firewalld 
# 开机启用
systemctl enable firewalld
# 开机禁用
systemctl disable firewalld
```

## firewalld-cmd命令
```bash
# 查看版本
firewall-cmd --version
# 查看帮助
firewall-cmd --help
# 显示状态
firewall-cmd --state
# 查看所有打开的端口
firewall-cmd --zone=public --list-ports
# 更新防火墙规则
firewall-cmd --reload
# 查看区域信息
firewall-cmd --get-active-zones
# 查看指定接口所属区域
firewall-cmd --get-zone-of-interface=eth0
# 拒绝所有包
firewall-cmd --panic-on
# 取消拒绝状态
firewall-cmd --panic-off
# 查看是否拒绝
firewall-cmd --query-panic

# 查看某个端口是否开放
firewall-cmd --zone=public --query-port=80/tcp
# 查看所有开启的端口
firewall-cmd --permanent --zone=public --list-ports
# 开启一个端口（--permanent永久生效，没有此参数重启后失效）
firewall-cmd --zone=public --add-port=80/tcp --permanent 
# 关闭一个端口
firewall-cmd --zone= public --remove-port=80/tcp --permanent


# 查看某个服务是否开放
firewall-cmd --zone=public --query-service=https
# 查看所有开启的服务
firewall-cmd --permanent --zone=public --list-services
# 开启一个服务（--permanent永久生效，没有此参数重启后失效）
firewall-cmd --zone=public --add-service=https --permanent
# 关闭一个服务
firewall-cmd --zone=public --remove-service=https --permanent

# 重新加载配置（修改后要重新加载配置才生效）
firewall-cmd --reload
```