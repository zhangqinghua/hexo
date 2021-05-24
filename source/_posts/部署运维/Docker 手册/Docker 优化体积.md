---
title: Docker 优化体积

categories:
- 部署运维
- Docker 手册

date: 2021-05-23
---
## 上传实验
#### 没有附加数据
Dockerfile 没有配置额外的数据。

```Dockerfile
FROM openjdk:8-jre-alpine

CMD ["echo", "123"]
```

`push` 到 dockerhub 上。
```bash
zhangqinghua$ docker push zhangqinghua/test:0.0.7
The push refers to repository [docker.io/zhangqinghua/test]
edd61588d126: Layer already exists 
9b9b7f3d56a0: Layer already exists 
f1b5933fe4b5: Layer already exists 
0.0.7: digest: sha256:e0e3d759e97e3b804d8b3d0333fce57a5770aec5124047e13a84335b10f82c9a size: 947
```

可以看到，大小大概是 947kb 左右。

#### 添加 2kb

## 附：Docker SpringBot 优化体积
