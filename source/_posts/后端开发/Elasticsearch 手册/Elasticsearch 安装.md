---
title: Elasticsearch 安装

categories:
- 后端开发
- Elasticsearch 手册

date: 2021-04-18 99:99:99
---
## Docker 安装
#### 拉取镜像
```bash
zhangqinghua$ docker pull elasticsearch:7.4.2
7.4.2: Pulling from library/elasticsearch
d8d02d457314: Pull complete 
...
Digest: sha256:543bf7a3d61781bad337d31e6cc5895f16b55aed4da48f40c346352420927f74
Status: Downloaded newer image for elasticsearch:7.4.2
docker.io/library/elasticsearch:7.4.2
```

#### 创建挂载目录
创建挂载目录：
```bash
mkdir /data/elasticsearch/config

mkdir /data/elasticsearch/data
```

创建 elasticsearch 配置文件： 

```bash
echo /data/elasticsearch/config/elasticsearch.yml
# 允许任意IP访问elasticsearch
echo "http.host:0.0.0.0" >> /data/elasticsearch/config/elasticsearch.yml
```

#### 运行容器

执行容器命令：   

```bash
zhangqinghua$ docker run --name elasticsearch001 \
                         -p 9200:9200 -p 9300:9300 \
                         -e "discovery.type=single-node" \
                         -e ES_JAVA_OPTS="-Xms64m -Xmx128m" \
                         -v /data/elasticsearch/config/elaticsearch.yml:/usr/share/elasticsearch/config/elaticsearch.yml \
                         -v /data/elasticsearch/data:/usr/share/elasticsearch/data \
                         -v /data/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
                         -d elasticsearch:7.4.2
```

参数说明：
1. `-p 9200:9200`
1. `-p 9300:9300`
1. `-e "discovery.type=single-node"`
   运行模式：单机
1. `-e ES_JAVA_OPTS="-Xms64m -Xmx128m"`
   重要：如果没有指定内存，elasticsearch 启动会将所有内存占用，服务器就卡死了

## Docker 安装 Kibana
#### 拉取镜像
```bash
zhangqinghua$ docker pull kibana:7.4.2
7.4.2: Pulling from library/kibana
d8d02d457314: Already exists 
bc64069ca967: Pull complete 
...
Digest: sha256:355f9c979dc9cdac3ff9a75a817b8b7660575e492bf7dbe796e705168f167efc
Status: Downloaded newer image for kibana:7.4.2
docker.io/library/kibana:7.4.2
```

#### 启动容器
```bash
$ docker run --name kibana001 -e ELASTICSEARCH_HOSTS=http://10.100.32.124:9200 -p 5601:5601 -d kibana:7.4.2
fa6a5d36792226e786489044295bd055508c7b2c5b837b15e510e4d66f62fe34
```