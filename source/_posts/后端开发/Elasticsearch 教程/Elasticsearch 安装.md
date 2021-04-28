---
title: Elasticsearch 安装

categories:
- 后端开发
- Elasticsearch 教程

date: 2021-04-18 00:98:99
---
## 二进制文件安装
Elastic 需要 Java 8 环境。如果你的机器还没安装 Java，可以参考[这篇文章](https://www.digitalocean.com/community/tutorials/how-to-install-java-with-apt-get-on-debian-8)，注意要保证环境变量 `JAVA_HOME` 正确设置。

#### 下载压缩包
安装完 Java，就可以跟着[官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/zip-targz.html)安装 Elastic。直接下载压缩包比较简单。

```bash
$ wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.5.1.zip
$ unzip elasticsearch-5.5.1.zip
$ cd elasticsearch-5.5.1/ 
```

#### 启动 elasticsearch
接着，进入解压后的目录，运行下面的命令，启动 Elastic。

```bash
$ ./bin/elasticsearch
```

如果这时报错"max virtual memory areas vm.maxmapcount [65530] is too low"，要运行下面的命令。

```bash
$ sudo sysctl -w vm.max_map_count=262144
```

#### 测试请求
如果一切正常，Elastic 就会在默认的9200端口运行。这时，打开另一个命令行窗口，请求该端口，会得到说明信息。

```bash
$ curl localhost:9200

{
  # 节点名字
  "name" : "atntrTf",
  # 在集群中的名字
  "cluster_name" : "elasticsearch",
  # 在集群中的Id
  "cluster_uuid" : "tf9250XhQ6ee4h7YI11anA",
  "version" : {
    "number" : "5.5.1",
    "build_hash" : "19c13d0",
    "build_date" : "2017-07-18T20:44:24.823Z",
    "build_snapshot" : false,
    "lucene_version" : "6.6.0"
  },
  "tagline" : "You Know, for Search"
}
```

上面代码中，请求9200端口，Elastic 返回一个 JSON 对象，包含当前节点、集群、版本等信息。

按下 Ctrl + C，Elastic 就会停止运行。

#### 配置远程访问
默认情况下，Elastic 只允许本机访问，如果需要远程访问，可以修改 Elastic 安装目录的 `config/elasticsearch.yml` 文件，去掉 `network.host`的注释，将它的值改成 `0.0.0.0`，然后重新启动 Elastic。

```conf
network.host: 0.0.0.0
```

上面代码中，设成 `0.0.0.0` 让任何人都可以访问。线上服务不要这样设置，要设成具体的 IP。

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

> ELASTICSEARCH_HOSTS 只要指定宿主机的地址，不能使用 localhsot 或者 192.168.0.1。

#### 查看效果
打开浏览器，输入：http://localhost:5601 查看效果。

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210418234416.png)

## 常见问题
**Exception in thread "main" java.nio.file.NoSuchFileException: /usr/share/elasticsearch/config/jvm.options**
1. 场景：Docker 启动 elasticsearch 失败。
1. 原因：宿主机没有配置 `jvm.options` 文件。
1. 解决：将文件拷出到宿主机，重启。

**Caused By: java.nio.file.AccessDeniedExceion: /usr/share/elasticsearch/data/nodes**
1. 场景：Docker 启动 elasticsearch 失败。
1. 原因：Docker 没有操作宿主机 elasticsearch 挂载的目录权限。
1. 解决：修改目录权限：`chmod -R 777 /elasticsearcch/data/`。

**Unable to revive connection: http://elasticsearch:9200**
1. 场景：Docker 启动 Kibana 提示 Kibana server is not ready yet，查询日志报错。
1. 原因：没有指定 `ELASTICSEARCH_HOSTS` 地址，命令输错。
1. 解决：`docker run --name kibana001 -e ELASTICSEARCH_HOSTS=http://192.168.42.2:9200`。