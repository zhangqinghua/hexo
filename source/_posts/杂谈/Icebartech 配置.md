---
title: Icebartech 常用配置
tags:
- Icebartech

categories:
- 杂谈

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

    