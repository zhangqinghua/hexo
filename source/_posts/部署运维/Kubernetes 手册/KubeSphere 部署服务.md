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

#### 参考配置
```conf
kind: Service
apiVersion: v1
metadata:
  name: nacos-server
  namespace: easybyte-api
  labels:
    app: nacos-server
    version: v1
  annotations:
    kubesphere.io/alias-name: 注册中心
    kubesphere.io/creator: admin
    kubesphere.io/description: 注册中心、服务发现和配置中心
    kubesphere.io/serviceType: statelessservice
spec:
  ports:
    - name: tcp-8848
      protocol: TCP
      port: 8848
      targetPort: 8848
      nodePort: 31029
  selector:
    app: nacos-server
  clusterIP: 10.96.156.4
  type: NodePort
  sessionAffinity: None
  externalTrafficPolicy: Cluster

---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nacos-server
  namespace: easybyte-api
  labels:
    app: nacos-server
  annotations:
    deployment.kubernetes.io/revision: '3'
    kubesphere.io/creator: admin
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nacos-server
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nacos-server
      annotations:
        kubesphere.io/restartedAt: '2021-05-17T09:41:02.330Z'
    spec:
      containers:
        - name: nacos-server
          image: 'nacos/nacos-server:1.4.2'
          ports:
            - name: tcp-8848
              containerPort: 8848
              protocol: TCP
          env:
            - name: MODE
              value: standalone
            - name: SPRING_DATASOURCE_PLATFORM
              value: mysql
            - name: MYSQL_SERVICE_HOST
              value: rm-wz9snunp381a6zza4mo.mysql.rds.aliyuncs.com
            - name: MYSQL_SERVICE_PORT
              value: '3306'
            - name: MYSQL_SERVICE_DB_NAME
              value: nacos_config
            - name: MYSQL_SERVICE_USER
              value: easybyte
            - name: MYSQL_SERVICE_PASSWORD
              value: izDN4p)w9(s%#!K
          resources: {}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          imagePullPolicy: IfNotPresent
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      dnsPolicy: ClusterFirst
      serviceAccountName: default
      serviceAccount: default
      securityContext: {}
      affinity: {}
      schedulerName: default-scheduler
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%
      maxSurge: 25%
  revisionHistoryLimit: 10
  progressDeadlineSeconds: 600
```

## 部署 Nginx

## 部署 TxLCN TM
参考：https://hub.docker.com/r/johnnywjh/txlcn.tm

## 常见问题
#### 变量引用不生效或者流水线引用变量报错
原因：在引用变量时，常见的错误是没区分单双引号。单引号并不替换变量，双引号才会替换变量，这与 Shell 一致。

#### Docker 时间不一致问题
1. 修改 DockerFile 构建文件
1. 或者在 pod 文件上懂手脚：https://blog.51cto.com/u_14268033/2463671