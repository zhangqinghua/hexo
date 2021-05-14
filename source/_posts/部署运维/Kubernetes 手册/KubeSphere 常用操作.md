---
title: KubeSphere 常用操作

categories:
- 部署运维
- Kubernetes 手册

date: 2021-05-13
---
#### 污点管理
**1. 选择一个节点**
![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210514121835.png)

**2. 选中污点管理**
![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210514121921.png)

**3. 删除污点**
![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210514121951.png)

删除污点后，可以看到 pod 部署到 master 节点上来了。

```bash
zhangqinghua$ kubectl get pods -o wide
NAME                       READY   STATUS    RESTARTS   AGE     IP             NODE                      NOMINATED NODE   READINESS GATES
tomcat6-5f7ccf4cb9-25p6t   1/1     Running   0          5m23s   10.244.0.8     izwz9go2hn3kv068o5wpdpz   <none>           <none>
tomcat6-5f7ccf4cb9-5gcdl   1/1     Running   0          5m23s   10.244.0.7     izwz9go2hn3kv068o5wpdpz   <none>           <none>
tomcat6-5f7ccf4cb9-78b5h   1/1     Running   0          5m23s   10.244.0.9     izwz9go2hn3kv068o5wpdpz   <none>           <none>
tomcat6-5f7ccf4cb9-7xgj7   1/1     Running   0          51m     10.244.1.181   easybyte-dev-server       <none>           <none>
tomcat6-5f7ccf4cb9-n5qhc   1/1     Running   0          5m23s   10.244.0.6     izwz9go2hn3kv068o5wpdpz   <none>           <none>
tomcat6-5f7ccf4cb9-wn7ml   1/1     Running   0          5m23s   10.244.1.204   easybyte-dev-server       <none>           <none>
```