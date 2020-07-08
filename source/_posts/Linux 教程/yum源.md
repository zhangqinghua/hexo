---
title: yum 源

categories:
- Linux 教程

date: 2020-07-08
---

## 修改源
CentOS 安装后默认官方源，速度可能不是很快，这个时候就需要更改为国内的源了，这里以阿里源 为例，进行脚本展示：

```bash
#!/bin/bash
# by liuxg
# 2019.05.15
# aliyun_repo.sh


# 获得当前 CentOS 系统发行版本号
# 第一个 awk 后边必须换行, 目前未查到原因
releasetmp=`cat /etc/redhat-release | awk '{match($0,"release ")
 print substr($0,RSTART+RLENGTH)}' | awk -F '.' '{print $1}'`
echo $releasetmp
sleep 5


yum install wget -y
# 备份原文件  应该添加检测是否原来有备份文件, 有的话应该备份为别名文件  此处省略
cp /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak 
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-$releasetmp.repo
if [[ $? -eq 0 ]];then
    echo -e "\033[32m# yum 源已成功更新为 aliyun_repo #\033[0m"; 
    sleep 3; 
else
    echo -e "\033[31m# yum 源未成功更新为 aliyun_repo #\n3s 后退出...\033[0m";
    exit;
fi

# 添加EPEL源
wget -P /etc/yum.repos.d/ http://mirrors.aliyun.com/repo/epel-$releasetmp.repo 

# 重建缓存
yum clean all
yum makecache

# 自动更新包列表，可选择注释该行
yum update -y; 
```