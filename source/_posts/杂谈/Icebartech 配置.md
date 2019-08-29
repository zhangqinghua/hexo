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

