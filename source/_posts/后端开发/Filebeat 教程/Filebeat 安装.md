---
title:  Filebeat 安装

categories:
- 后端开发
- Filebeat 教程

date: 2021-05-06
---

## 二进制包安装
#### 下载二进制文件并解压
```bash
$ curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.2-darwin-x86_64.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 23.2M  100 23.2M    0     0  2124k      0  0:00:11  0:00:11 --:--:-- 4717k
```

下载完成后，解压。

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210506210704.png)

#### 新增配置文件
在当前目录下，新增一个配置文件：

```yml
# vim itcast.yml
filebeat.inputs:  # 采用控制台输入
- type: stdin
  enable: true
output.console:   # 采用控制台输出
  pretty: true
  enable: true
```

#### 启动 Filebeat
执行命令：

```bash
$ ./filebeat -e -c itcast.yml 
2021-05-06T20:58:12.304+0800	INFO	...
...
```

#### 测试 Filebeat
启动完后，Filebeat 会一直等待输入。

```bash
"ms":4}}},"info":{"ephemeral_id":"8ecc6ebe-7b92-4c05-b0fe-c0cfaa1cd83d","uptime":{"ms":420034}},"memstats":{"gc_next":8970528,"memory_alloc":7780096,"memory_total":25356432,"rss":4096},"runtime":{"goroutines":20}},"filebeat":{"harvester":{"open_files":0,"running":1}},"libbeat":{"config":{"module":{"running":0}},"pipeline":{"clients":1,"events":{"active":0}}},"registrar":{"states":{"current":1}},"system":{"load":{"1":2.9111,"15":3.1938,"5":3.168,"norm":{"1":0.7278,"15":0.7985,"5":0.792}}}}}}...

# 输入 Hello
HelloWord

# Filebeat 的响应结果
{
  "@timestamp": "2021-05-06T13:05:58.908Z",
  "@metadata": {
    "beat": "filebeat",
    "type": "_doc",
    "version": "7.4.2"
  },
  "ecs": {
    "version": "1.1.0"
  },
  "host": {
    "name": "zhangqinghuadeiMac.local"
  },
  "agent": {
    "ephemeral_id": "8ecc6ebe-7b92-4c05-b0fe-c0cfaa1cd83d",
    "hostname": "zhangqinghuadeiMac.local",
    "id": "1bf77d2d-68bd-4daa-ad88-528587f9f1d9",
    "version": "7.4.2",
    "type": "filebeat"
  },
  "log": {
    "offset": 0,
    "file": {
      "path": ""
    }
  },
  "message": "HelloWord",
  "input": {
    "type": "stdin"
  }
}
2021-05-06T21:05:59.911+0800	ERROR	file/states.go:112	State for  should have been dropped, but couldn't as state is not finished.
```

可以看到，Filebeat 已经监听到了输入事件，并处理后作了输出。

## 配置 Filebeat 读取多个日志文件
```yml
filebeat.inputs:
- type: log
  enable: true
  paths:
    - /Users/zhangqinghua/Documents/log/*.log
output.console:
  pretty: true
  enable: true
```

## 配置 Filebeat 发送日志到 Elasticsearch
```yml
filebeat.inputs:
- type: log
  enable: true
  paths:
    - /Users/zhangqinghua/Documents/log/*.log
output.elasticsearch:
  hosts: ["localhost:9200"]
```

## 配置 Kibana 显示 Filebeat 收集的日志
**1. Create index pattern**
![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210507101710.png)

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210507101813.png)

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210507101846.png)

**2. Discover Filter**
![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210507101937.png)