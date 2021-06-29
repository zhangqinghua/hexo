---
title: KubeSphere 常见问题

categories:
- 部署运维
- Kubernetes 手册

date: 2021-05-13 07:00:10
---
#### KubeSphere 卸载卡死
场景：KubeSphere 卸载卡死，一直不动。
原因：？？？
解决：重启服务器，重新卸载，等个 30 分钟左右。

#### KubeSphere devops 部署 minio 失败
场景：KubeSphere 安装 devops 服务时，minio 一直没安装上，提示：

```
...
TASK [common : debug] **********************************************************
ok: [localhost] => {
    "msg": [
        "1. check the storage configuration and storage server",
        "2. make sure the DNS address in /etc/resolv.conf is available",
        "3. execute 'kubectl logs -n kubesphere-system -l job-name=minio-make-bucket-job' to watch logs",
        "4. execute 'helm -n kubesphere-system uninstall ks-minio && kubectl -n kubesphere-system delete job minio-make-bucket-job'",
        "5. Restart the installer pod in kubesphere-system namespace"
    ]
}

TASK [common : fail] ***********************************************************
fatal: [localhost]: FAILED! => {"changed": false, "msg": "It is suggested to refer to the above methods for troubleshooting problems ."}

PLAY RECAP *********************************************************************
localhost                  : ok=40   changed=25   unreachable=0    failed=1    skipped=81   rescued=0    ignored=0  
```

原因：大概是有一个节点有污点，通过 describe 可以看到报错信息：

```bash
zhangqinghua$ kubectl describe pod  minio-make-bucket-job-2p5pr --namespace kubesphere-system
...
Events:
  Type     Reason            Age                 From               Message
  ----     ------            ----                ----               -------
  Warning  FailedScheduling  72s (x26 over 27m)  default-scheduler  0/2 nodes are available: 1 Insufficient cpu, 1 node(s) had taints that the pod didn't tolerate.
```

解决：开放一个节点的污点即可。

#### 创建 DevOps 项目失败
https://kubesphere.com.cn/forum/d/2811

原因：实际上是安装报错了，没有安装成功。

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210515222342.png)

解决：卸载掉 jenkins 再重新安装。

```bash
# 1. 删除旧的 Jenkins
zhangqinghus$ helm delete ks-jenkins
release "ks-jenkins" deleted

# 2. 删除 cc 中 devops 的状态：删掉 status.devops
zhangqinghus$ kubectl edit cc -n kubesphere-system ks-installer

# 3. 重启 ks-installer 
zhangqinghus$ kubectl rollout restart deploy -n kubesphere-system ks-installer
```

下面这样子才是安装成功的：

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210515222656.png)

#### DevOps 部署项目失败
场景：到 deploy to Kubernetes 阶段时报错：

```
Starting Kubernetes deployment
Loading configuration: /home/jenkins/agent/workspace/test-cicd928zj/test123/easybyte-auth/deploy/easybyte-auth.yml
ERROR: ERROR: java.lang.RuntimeException: io.kubernetes.client.openapi.ApiException: Unprocessable Entity
hudson.remoting.ProxyException: java.lang.RuntimeException: io.kubernetes.client.openapi.ApiException: Unprocessable Entity
```

原因：拉到报错最后面，提示参数问题。

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210516220443.png)

解决：修复参数问题。

#### 您需要联系平台管理员或者集群管理员为企业空间授权集群的访问权限
场景：在开启多集群功能后，企业空间无法查看项目。
原因：集群可见性。
解决：开发集群可见行。
参考：https://kubesphere.com.cn/docs/cluster-administration/cluster-settings/cluster-visibility-and-authorization/

#### 彻底禁用 ES
场景：在开启日志系统后，ES 也随着安装，但是禁用日志系统后，ES 没有停掉。
原因：日志系统、事件系统、审计日志均使用到 ES。即使将它们禁用，ES 还是不会自动关闭，需要手工删除。
解决：将使用到 ES 的组件禁用后，手工执行 `kubectl delete ns xxx`。

参考：https://kubesphere.com.cn/forum/d/4860-logging

参考：https://kubesphere.com.cn/docs/faq/observability/logging/

#### 部署多集群环境，ks-apiserver-xxx 报错
场景：配置 Host 集群，ks-apiserver-xxx 一直处于 CrashLoopBackOff 状态， 查看日志提示：

```bash
zhangqinghua$ kubectl logs -f  ks-apiserver-65cdffb5d9-p2bzm -n kubesphere-system
Error: JWT secret MUST not be empty
2021/06/15 10:19:03 JWT secret MUST not be empty
```

原因：ks-installer 配置的 `spec.authentication.jwtSecret: null`。但是这本身是host集群，估计是 bug。
解决：配置 `spec.authentication.jwtSecret: gfIwilcc0WjNGKJ5DLeksf2JKfcLgTZU` 随便设置值，再删除 ks-installer 让它重新安装即可。

#### Host 集群添加 Member 集群，无法生产 agent.yaml 代理文件。
场景：Host 集群通过代理方式添加 Member 集群，无法生成 agent.yaml 代理文件，提示：

```
Not Found
cannot generate agent deployment yaml for member cluster because tower.kubesphere-system.svc service has no public address, please check tower.kubesphere-system.svc status, or set address mannually in ClusterConfiguration
```

原因：ks-installer 需要配置 `multicluster.proxyPublishAddress`。 

解决：将 `proxyPublishAddress` 的值添加到 ks-installer 的配置文件中，参考：https://kubesphere.com.cn/docs/multicluster-management/enable-multicluster/agent-connection/。

```bash
zhangqinghua$ kubectl -n kubesphere-system get svc
NAME                    TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)           AGE
ks-apiserver            ClusterIP      10.96.215.133   <none>        80/TCP            30d
ks-console              NodePort       10.96.138.81    <none>        80:30880/TCP      30d
ks-controller-manager   ClusterIP      10.96.164.104   <none>        443/TCP           30d
mc-hongkong             ClusterIP      10.96.0.191     <none>        6443/TCP,80/TCP   2m38s
minio                   ClusterIP      10.96.121.171   <none>        9000/TCP          30d
openldap                ClusterIP      None            <none>        389/TCP           30d
tower                   LoadBalancer   10.96.246.61    <pending>     8080:31347/TCP    76m

zhangqinghua$ kubectl edit cc ks-installer -n kubesphere-system
multicluster:
    clusterRole: host
    proxyPublishAddress: http://139.198.120.120:31347 # 139.198.120.120 是 Host 集群的公网地址
```

#### jaeger-query 一直报错 CrashLoopBackOff
```bash
zhangqinghua$ kubectl get pods -n istio-system
NAME                                       READY   STATUS             RESTARTS   AGE
istio-ingressgateway-78dbc5fbfd-dxpv4      1/1     Running            0          27d
istiod-1-6-10-7db56f875b-v7qx4             1/1     Running            0          27d
jaeger-collector-76bf54b467-p4jzl          1/1     Running            0          27d
jaeger-es-index-cleaner-1621554900-r2hbr   0/1     Completed          0          26d
jaeger-es-index-cleaner-1621727700-85wdn   0/1     Completed          0          24d
jaeger-es-index-cleaner-1621814100-w7xm5   0/1     Completed          0          23d
jaeger-es-index-cleaner-1623801300-76pdj   0/1     Error              0          10h
jaeger-es-index-cleaner-1623801300-gphqd   0/1     Error              0          10h
jaeger-es-index-cleaner-1623801300-mglx2   0/1     Error              0          10h
jaeger-es-index-cleaner-1623801300-nmpl5   0/1     Error              0          10h
jaeger-es-index-cleaner-1623801300-qvmjn   0/1     Error              0          10h
jaeger-es-index-cleaner-1623801300-zj99x   0/1     Error              0          10h
jaeger-operator-7559f9d455-shlrn           1/1     Running            0          27d
jaeger-query-b478c5655-sjqnb               1/2     CrashLoopBackOff   289        24h
kiali-5bbfc48f94-9b5td                     1/1     Running            0          27d
kiali-operator-7d5dc9d766-5v7jd            1/1     Running            0          27d

zhangqinghua$ kubectl describe pod jaeger-query-b478c5655-sjqnb -n istio-system
Name:         jaeger-query-b478c5655-sjqnb
Namespace:    istio-system
Priority:     0
Node:         easybyte-dev-server/172.27.243.200
Start Time:   Tue, 15 Jun 2021 18:06:46 +0800
Labels:       app=jaeger
              app.kubernetes.io/component=query
              app.kubernetes.io/instance=jaeger
              app.kubernetes.io/managed-by=jaeger-operator
              app.kubernetes.io/name=jaeger-query
              app.kubernetes.io/part-of=jaeger
              pod-template-hash=b478c5655
              sidecar.jaegertracing.io/injected=jaeger
Annotations:  linkerd.io/inject: disabled
              prometheus.io/port: 16687
              prometheus.io/scrape: true
              sidecar.istio.io/inject: false
              sidecar.jaegertracing.io/inject: jaeger
Status:       Running
IP:           10.244.1.46
IPs:
  IP:           10.244.1.46
Controlled By:  ReplicaSet/jaeger-query-b478c5655
Containers:
  jaeger-query:
    Container ID:  docker://91b9dc663228625ba00f60484a8f2f5218fceaf5ebc0c7f22a206c4cf63c488e
    Image:         jaegertracing/jaeger-query:1.17
    Image ID:      docker-pullable://jaegertracing/jaeger-query@sha256:98fe9322ba7473e5aa89f50b47d15c1ff076d3ab323e714dc7fbc5b2e416719d
    Ports:         16686/TCP, 16687/TCP
    Host Ports:    0/TCP, 0/TCP
    Args:
      --es.index-prefix=logstash
      --es.server-urls=http://elasticsearch-logging-data.kubesphere-logging-system.svc:9200
      --query.ui-config=/etc/config/ui.json
    State:          Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       Error
      Exit Code:    1
      Started:      Wed, 16 Jun 2021 18:25:57 +0800
      Finished:     Wed, 16 Jun 2021 18:26:02 +0800
    Ready:          False
    Restart Count:  289
    Liveness:       http-get http://:16687/ delay=5s timeout=1s period=15s #success=1 #failure=5
    Readiness:      http-get http://:16687/ delay=1s timeout=1s period=10s #success=1 #failure=3
    Environment:
      SPAN_STORAGE_TYPE:    elasticsearch
      JAEGER_SERVICE_NAME:  jaeger.istio-system
      JAEGER_PROPAGATION:   jaeger,b3
    Mounts:
      /etc/config from jaeger-ui-configuration-volume (ro)
      /var/run/secrets/kubernetes.io/serviceaccount from jaeger-token-lrb7f (ro)
  jaeger-agent:
    Container ID:  docker://457b725faf86c1cf7f58ea9752be11dfac1073fa761f075099d3254c0d3e9780
    Image:         jaegertracing/jaeger-agent:1.17
    Image ID:      docker-pullable://jaegertracing/jaeger-agent@sha256:bb9ff8866bb8b5f1e6bdbfb76d36308a049fdf5bd2aa9234cc83d6df5c9dd22a
    Ports:         5775/UDP, 5778/TCP, 6831/UDP, 6832/UDP, 14271/TCP
    Host Ports:    0/UDP, 0/TCP, 0/UDP, 0/UDP, 0/TCP
    Args:
      --jaeger.tags=cluster=undefined,deployment.name=jaeger-query,pod.namespace=istio-system,pod.name=${POD_NAME:},host.ip=${HOST_IP:},container.name=jaeger-query
      --reporter.grpc.host-port=dns:///jaeger-collector-headless.istio-system:14250
      --reporter.type=grpc
    State:          Running
      Started:      Tue, 15 Jun 2021 18:07:23 +0800
    Ready:          True
    Restart Count:  0
    Environment:
      POD_NAME:  jaeger-query-b478c5655-sjqnb (v1:metadata.name)
      HOST_IP:    (v1:status.hostIP)
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from jaeger-token-lrb7f (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             False 
  ContainersReady   False 
  PodScheduled      True 
Volumes:
  jaeger-ui-configuration-volume:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      jaeger-ui-configuration
    Optional:  false
  jaeger-token-lrb7f:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  jaeger-token-lrb7f
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason   Age                     From                          Message
  ----     ------   ----                    ----                          -------
  Warning  BackOff  2m40s (x6768 over 24h)  kubelet, easybyte-dev-server  Back-off restarting failed container

zhangqinghua$ kubectl logs -f jaeger-query-b478c5655-sjqnb -n istio-system
error: a container name must be specified for pod jaeger-query-b478c5655-sjqnb, choose one of: [jaeger-query jaeger-agent]
```

原因：查询命令弄错了，正确的：

```bash
zhangqinghua$ kubectl logs -f jaeger-query-b478c5655-sjqnb jaeger-query -n istio-system
2021/06/17 01:30:51 maxprocs: Leaving GOMAXPROCS=2: CPU quota undefined
{"level":"info","ts":1623893451.6409516,"caller":"flags/service.go:115","msg":"Mounting metrics handler on admin server","route":"/metrics"}
{"level":"info","ts":1623893451.6411955,"caller":"flags/admin.go:115","msg":"Mounting health check on admin server","route":"/"}
{"level":"info","ts":1623893451.6412811,"caller":"flags/admin.go:121","msg":"Starting admin HTTP server","http-port":16687}
{"level":"info","ts":1623893451.641299,"caller":"flags/admin.go:107","msg":"Admin server started","http-port":16687,"health-status":"unavailable"}
{"level":"fatal","ts":1623893456.67678,"caller":"query/main.go:93","msg":"Failed to init storage factory","error":"failed to create primary Elasticsearch client: health check timeout: Head http://elasticsearch-logging-data.kubesphere-logging-system.svc:9200: dial tcp: lookup elasticsearch-logging-data.kubesphere-logging-system.svc on 10.96.0.10:53: no such host: no Elasticsearch node available","stacktrace":"main.main.func1\n\tgithub.com/jaegertracing/jaeger@/cmd/query/main.go:93\ngithub.com/spf13/cobra.(*Command).execute\n\tgithub.com/spf13/cobra@v0.0.3/command.go:762\ngithub.com/spf13/cobra.(*Command).ExecuteC\n\tgithub.com/spf13/cobra@v0.0.3/command.go:852\ngithub.com/spf13/cobra.(*Command).Execute\n\tgithub.com/spf13/cobra@v0.0.3/command.go:800\nmain.main\n\tgithub.com/jaegertracing/jaeger@/cmd/query/main.go:135\nruntime.main\n\truntime/proc.go:203"}
```

解决：ES 没有安装。

参考：https://kubesphere.com.cn/forum/d/4823-istio-system/8

参考：https://kubesphere.com.cn/docs/faq/observability/logging/

#### 一个节点资源不足，触发驱逐。使得另外一个节点也资源不足，形成连锁反应
场景：参考[节点资源不足被驱逐，形成连锁反应。](https://kubesphere.com.cn/forum/d/4955)

原因：集群雪崩，如果节点上调度了大量pod，且pod没有合理的limit限制，节点资源将被耗尽，sshd、kubelet等进程OOM，节点变成 not ready状态，pod重新继续调度到其他节点，新节点也被打挂，引起集群雪崩。

解决：todo 先参考 https://developer.aliyun.com/article/604524