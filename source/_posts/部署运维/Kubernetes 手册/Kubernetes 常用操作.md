---
title: Kubernetes 常用操作

categories:
- 部署运维
- Kubernetes 手册

date: 2021-05-13 09:00:08
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

## 查看 pod 日志
```bash
zhangqinghua$ kubectl logs tiller-deploy-bc4f597d8-j7rgh --namespace kube-system
Error from server (BadRequest): container "tiller" in pod "tiller-deploy-bc4f597d8-j7rgh" is waiting to start: trying and failing to pull image
```

## 查看 pod 信息
```bash
zhangqinghua$ kubectl describe pod tiller-deploy-bc4f597d8-j7rgh --namespace kube-system
Name:         tiller-deploy-bc4f597d8-j7rgh
Namespace:    kube-system
Priority:     0
Node:         izwz9go2hn3kv068o5wpdpz/172.27.243.201
Start Time:   Sat, 15 May 2021 19:37:43 +0800
Labels:       app=helm
              name=tiller
              pod-template-hash=bc4f597d8
Annotations:  <none>
Status:       Pending
IP:           10.244.0.227
IPs:
  IP:           10.244.0.227
Controlled By:  ReplicaSet/tiller-deploy-bc4f597d8
Containers:
  tiller:
    Container ID:   
    Image:          gcr.io/kubernetes-helm/tiller:v2.16.3
    Image ID:       
    Ports:          44134/TCP, 44135/TCP
    Host Ports:     0/TCP, 0/TCP
    State:          Waiting
      Reason:       ImagePullBackOff
    Ready:          False
    Restart Count:  0
    Liveness:       http-get http://:44135/liveness delay=1s timeout=1s period=10s #success=1 #failure=3
    Readiness:      http-get http://:44135/readiness delay=1s timeout=1s period=10s #success=1 #failure=3
    Environment:
      TILLER_NAMESPACE:    kube-system
      TILLER_HISTORY_MAX:  300
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from tiller-token-4fc68 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             False 
  ContainersReady   False 
  PodScheduled      True 
Volumes:
  tiller-token-4fc68:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  tiller-token-4fc68
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason     Age                    From                              Message
  ----     ------     ----                   ----                              -------
  Normal   Scheduled  6m27s                  default-scheduler                 Successfully assigned kube-system/tiller-deploy-bc4f597d8-j7rgh to izwz9go2hn3kv068o5wpdpz
  Normal   Pulling    4m15s (x4 over 6m26s)  kubelet, izwz9go2hn3kv068o5wpdpz  Pulling image "gcr.io/kubernetes-helm/tiller:v2.16.3"
  Warning  Failed     4m (x4 over 6m11s)     kubelet, izwz9go2hn3kv068o5wpdpz  Failed to pull image "gcr.io/kubernetes-helm/tiller:v2.16.3": rpc error: code = Unknown desc = Error response from daemon: Get https://gcr.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
  Warning  Failed     4m (x4 over 6m11s)     kubelet, izwz9go2hn3kv068o5wpdpz  Error: ErrImagePull
  Normal   BackOff    3m23s (x7 over 6m11s)  kubelet, izwz9go2hn3kv068o5wpdpz  Back-off pulling image "gcr.io/kubernetes-helm/tiller:v2.16.3"
  Warning  Failed     75s (x15 over 6m11s)   kubelet, izwz9go2hn3kv068o5wpdpz  Error: ImagePullBackOff
```

## 删除 pod
```bash
zhangqinghua$ kubectl delete pod tiller-deploy-bc4f597d8-j7rgh --namespace kube-system 
pod "tiller-deploy-bc4f597d8-j7rgh" deleted
```

## 修改 pod 端口范围
参考：[k8s 修改端口访问范围](https://blog.csdn.net/qq_39378657/article/details/111992316)