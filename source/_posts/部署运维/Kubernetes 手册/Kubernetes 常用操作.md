---
title: Kubernetes 常用操作

categories:
- 部署运维
- Kubernetes 手册

date: 2021-05-13
---
## Kubernetes 集群部署一个 Tomcat 服务
#### 创建 prod
```bash
zhangqinghua$ kubectl create deployment tomcat6 --image=tomcat:6.0.53-jre8
deployment.apps/tomcat6 created
```

#### 查询 prod
查询所有的资源：

```bash
zhangqinghua$ kubectl get all
NAME                           READY   STATUS    RESTARTS   AGE
pod/tomcat6-5f7ccf4cb9-f8269   0/1     Pending   0          17s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   26m

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/tomcat6   0/1     1            0           17s

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/tomcat6-5f7ccf4cb9   1         1         0       17s
```

使用 `-o wide` 查询更丰富的信息：

```bash
zhangqinghua$ kubectl get pods -o wide
NAME                       READY   STATUS    RESTARTS   AGE    IP       NODE     NOMINATED NODE   READINESS GATES
tomcat6-5f7ccf4cb9-f8269   0/1     Pending   0          106s   <none>   <none>   <none>           <none>
```

> READY 0/1 表示此 prod 在集群内有 1 个容器，其中 0 个准备好了。NODE 表示部署在哪个节点上。