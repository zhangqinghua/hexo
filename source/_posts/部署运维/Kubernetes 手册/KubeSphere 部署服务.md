---
title: KubeSphere 部署服务

categories:
- 部署运维
- Kubernetes 手册

date: 2021-05-13
---
## 部署 Nacos
#### 创建服务
![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210517115621.png)

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210517115844.png)

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210517120013.png)

#### 选择镜像
![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210517120254.png)

配置参考下面的：

```bash
zhangqinghua$ docker run --name nacos-quick -e MODE=standalone -p 8849:8848 \
                         -e SPRING_DATASOURCE_PLATFORM=mysql \
                         -e MYSQL_SERVICE_HOST=47.119.139.41 \
                         -e MYSQL_SERVICE_PORT=3306 \
                         -e MYSQL_SERVICE_DB_NAME=nacos_config \
                         -e MYSQL_SERVICE_USER=easybyte \
                         -e MYSQL_SERVICE_PASSWORD=easybyte \
                         -d nacos/nacos-server:1.4.2
```

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210517120428.png)

## 部署 Nginx