---
title: KubeSphere 安装

categories:
- 部署运维
- Kubernetes 手册

date: 2021-05-13 07:00:10
---
## 预先工作
#### 安装 Helm
```bash
zhangqinghua$ wget https://get.helm.sh/helm-v2.16.3-linux-amd64.tar.gz

zhangqinghua$ tar -zxvf helm-v2.16.3-linux-amd64.tar.gz

zhangqinghua$ mv linux-amd64/helm /usr/local/bin/helm

zhangqinghua$ helm version
Client: &version.Version{SemVer:"v2.17.0", GitCommit:"a690bad98af45b015bd3da1a41f6218b1a451dbe", GitTreeState:"clean"}
Error: could not find tiller
```

#### 创建 Helm 权限
先创建角色配置：

```yml
# vim helm-rbac.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects: 
- kind: ServiceAccount
  name: tiller
  namespace: kube-system
```

应用权限配置：

```bash
zhangqinghua$ kubectl apply -f helm-rbac.yaml
```

#### 安装 tiller
```bash
# 国内访问不了gcr.io就使用 registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.16.3
zhangqinghua$ helm init --service-account=tiller --tiller-image=gcr.io/kubernetes-helm/tiller:v2.16.3 --history-max 300
...

Tiller (the Helm server-side component) has been installed into your Kubernetes Cluster.

Please note: by default, Tiller is deployed with an insecure 'allow unauthenticated users' policy.
To prevent this, run `helm init` with the --tiller-tls-verify flag.
For more information on securing your installation see: https://v2.helm.sh/docs/securing_installation/

zhangqinghua$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                              READY   STATUS    RESTARTS   AGE
default       tomcat6-5f7ccf4cb9-f8269                          1/1     Running   0          3h32m
kube-system   coredns-7f9c544f75-2hsq2                          1/1     Running   0          3h57m
kube-system   coredns-7f9c544f75-v8g8z                          1/1     Running   0          3h57m
kube-system   etcd-izwz9go2hn3kv068o5wpdpz                      1/1     Running   0          3h57m
kube-system   kube-apiserver-izwz9go2hn3kv068o5wpdpz            1/1     Running   0          3h57m
kube-system   kube-controller-manager-izwz9go2hn3kv068o5wpdpz   1/1     Running   0          3h57m
kube-system   kube-flannel-ds-5t4sk                             1/1     Running   0          3h18m
kube-system   kube-flannel-ds-k9hnj                             1/1     Running   0          3h38m
kube-system   kube-proxy-4q852                                  1/1     Running   0          3h18m
kube-system   kube-proxy-6bvww                                  1/1     Running   0          3h57m
kube-system   kube-scheduler-izwz9go2hn3kv068o5wpdpz            1/1     Running   0          3h57m
kube-system   tiller-deploy-6ffcfbc8df-c8rbp                    1/1     Running   0          5m40s
```

整个过程大概需要 10 分钟。

#### 设置默认存储类型

确认 Master 节点是否有 Taint，如下看到 Master 节点有 Taint。

```bash
zhangqinghua$ kubectl describe node izwz9go2hn3kv068o5wpdp | grep Taint
Taints:             node-role.kubernetes.io/master:NoSchedule
```

如果有 Taint，则需要去掉：

```bash
zhangqinghua$ kubectl taint nodes izwz9go2hn3kv068o5wpdpz node-role.kubernetes.io/master:NoSchedule-
node/izwz9go2hn3kv068o5wpdpz untainted
```

再来查询，可以看到 Taint 没有了：

```bash
zhangqinghua$ kubectl describe node izwz9go2hn3kv068o5wpdp | grep Taint
Taints:             <none>
```

#### 创建 OpenESB 命名空间
```bash
zhangqinghua$ kubectl create ns openebs
namespace/openebs created

zhangqinghua$ kubectl get ns
NAME              STATUS   AGE
default           Active   6h18m
kube-node-lease   Active   6h18m
kube-public       Active   6h18m
kube-system       Active   6h18m
openebs           Active   20s
```

#### Helm 安装 OpenESB
若集群已安装了 Helm，可通过 Helm 命令来安装 OpenEBS：
```bash
zhangqinghua$ helm install --namespace openebs --name openebs stable/openebs --version 1.5.0
NAME:   openebs
LAST DEPLOYED: Thu May 13 23:11:59 2021
NAMESPACE: openebs
STATUS: DEPLOYED

RESOURCES:
==> v1/ClusterRole
NAME     AGE
openebs  0s

...

NOTES:
The OpenEBS has been installed. Check its status by running:
$ kubectl get pods -n openebs

For dynamically creating OpenEBS Volumes, you can either create a new StorageClass or
use one of the default storage classes provided by OpenEBS.

Use `kubectl get sc` to see the list of installed OpenEBS StorageClasses. A sample
PVC spec using `openebs-jiva-default` StorageClass is given below:

---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: demo-vol-claim
spec:
  storageClassName: openebs-jiva-default
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5G
---

For more information, please visit http://docs.openebs.io/.

Please note that, OpenEBS uses iSCSI for connecting applications with the
OpenEBS Volumes and your nodes should have the iSCSI initiator installed.
```

然后就一直等安装完成（一般 5 - 30 分钟左右）：

```bash
zhangqinghua$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                              READY   STATUS    RESTARTS   AGE
default       tomcat6-5f7ccf4cb9-f8269                          1/1     Running   0          6h57m
kube-system   coredns-7f9c544f75-2hsq2                          1/1     Running   0          7h22m
kube-system   coredns-7f9c544f75-v8g8z                          1/1     Running   0          7h22m
kube-system   etcd-izwz9go2hn3kv068o5wpdpz                      1/1     Running   0          7h22m
kube-system   kube-apiserver-izwz9go2hn3kv068o5wpdpz            1/1     Running   0          7h22m
kube-system   kube-controller-manager-izwz9go2hn3kv068o5wpdpz   1/1     Running   0          7h22m
kube-system   kube-flannel-ds-5t4sk                             1/1     Running   0          6h43m
kube-system   kube-flannel-ds-k9hnj                             1/1     Running   0          7h3m
kube-system   kube-proxy-4q852                                  1/1     Running   0          6h43m
kube-system   kube-proxy-6bvww                                  1/1     Running   0          7h22m
kube-system   kube-scheduler-izwz9go2hn3kv068o5wpdpz            1/1     Running   0          7h22m
kube-system   tiller-deploy-59665c97b6-85mwc                    1/1     Running   0          4m38s
openebs       maya-apiserver-7f664b95bb-b2ptv                   1/1     Running   0          6m3s
openebs       openebs-admission-server-85dcbc7979-g5dfl         1/1     Running   0          6m39s
openebs       openebs-apiserver-bc55cd99b-mtgnb                 1/1     Running   0          31m
openebs       openebs-localpv-provisioner-85ff89dd44-5ql55      1/1     Running   0          31m
openebs       openebs-ndm-operator-87df44d9-sbfsw               1/1     Running   1          31m
openebs       openebs-ndm-sjrjn                                 1/1     Running   0          31m
openebs       openebs-ndm-sq9ds                                 1/1     Running   0          31m
openebs       openebs-provisioner-7f86c6bb64-br2pb              1/1     Running   0          31m
openebs       openebs-snapshot-operator-54b9c886bf-j5xp2        2/2     Running   0          31m
```

查看效果：

```bash
zhangqinghua$ kubectl get sc -n openebs
NAME                        PROVISIONER                                                RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
openebs-device              openebs.io/local                                           Delete          WaitForFirstConsumer   false                  16m
openebs-hostpath            openebs.io/local                                           Delete          WaitForFirstConsumer   false                  16m
openebs-jiva-default        openebs.io/provisioner-iscsi                               Delete          Immediate              false                  16m
openebs-snapshot-promoter   volumesnapshot.external-storage.k8s.io/snapshot-promoter   Delete          Immediate              false                  16m
```

#### 将 openebs-hostpath 设置为 默认的 StorageClass**
```bash
zhangqinghua$ kubectl patch storageclass openebs-hostpath -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
storageclass.storage.k8s.io/openebs-hostpath patched
```

#### 恢复 Master 节点的 Taint**
至此，OpenEBS 的 LocalPV 已作为默认的存储类型创建成功。由于在文档开头手动去掉 了 master 节点的 Taint，我们可以在安装完 OpenEBS 后将 master 节点 Taint 加上，避 免业务相关的工作负载调度到 master 节点抢占 master 资源。

```bash
zhangqinghua$ kubectl taint nodes k8s-node1 node-role.kubernetes.io=master:NoSchedule
```

## KubeSphere 2.1 安装
执行以下命令进行安装：
```bash
# 需要手工下载，被墙
zhangqinghua$  wget https://raw.githubusercontent.com/kubesphere/ks-installer/v2.1.1/kubesphere-minimal.yaml

zhangqinghua$ kubectl apply -f kubesphere-minimal.yaml
namespace/kubesphere-system created
configmap/ks-installer created
serviceaccount/ks-installer created
clusterrole.rbac.authorization.k8s.io/ks-installer created
clusterrolebinding.rbac.authorization.k8s.io/ks-installer created
deployment.apps/ks-installer created
```

查看安装日志，请耐心等待安装成功。

```bash
zhangqinghua$ kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l app=ks-install -o jsonpath='{.items[0].metadata.name}') -f
2021-05-13T15:55:32Z INFO     : shell-operator v1.0.0-beta.5
2021-05-13T15:55:32Z INFO     : HTTP SERVER Listening on 0.0.0.0:9115
2021-05-13T15:55:32Z INFO     : Use temporary dir: /tmp/shell-operator
2021-05-13T15:55:32Z INFO     : Initialize hooks manager ...
2021-05-13T15:55:32Z INFO     : Search and load hooks ...
2021-05-13T15:55:32Z INFO     : Load hook config from '/hooks/kubesphere/installRunner.py'
2021-05-13T15:55:32Z INFO     : Initializing schedule manager ...
2021-05-13T15:55:32Z INFO     : KUBE Init Kubernetes client
2021-05-13T15:55:32Z INFO     : KUBE-INIT Kubernetes client is configured successfully
2021-05-13T15:55:32Z INFO     : MAIN: run main loop
2021-05-13T15:55:32Z INFO     : MAIN: add onStartup tasks
...
...

Start installing monitoring
**************************************************
task monitoring status is successful
total: 1     completed:1
**************************************************
#####################################################
###              Welcome to KubeSphere!           ###
#####################################################

Console: http://172.27.243.200:30880
Account: admin
Password: P@88w0rd

NOTES：
  1. After logging into the console, please check the
     monitoring status of service components in
     the "Cluster Status". If the service is not
     ready, please wait patiently. You can start
     to use when all components are ready.
  2. Please modify the default password after login.

#####################################################
```

等个 10 多分钟，再次查看：

```bash
zhangqinghua$ kubectl get pods --all-namespaces
NAMESPACE                      NAME                                              READY   STATUS              RESTARTS   AGE
default                        tomcat6-5f7ccf4cb9-f8269                          1/1     Running             0          7h42m
kube-system                    coredns-7f9c544f75-2hsq2                          1/1     Running             0          8h
kube-system                    coredns-7f9c544f75-v8g8z                          1/1     Running             0          8h
kube-system                    etcd-izwz9go2hn3kv068o5wpdpz                      1/1     Running             0          8h
kube-system                    kube-apiserver-izwz9go2hn3kv068o5wpdpz            1/1     Running             0          8h
kube-system                    kube-controller-manager-izwz9go2hn3kv068o5wpdpz   1/1     Running             0          8h
kube-system                    kube-flannel-ds-5t4sk                             1/1     Running             0          7h28m
kube-system                    kube-flannel-ds-k9hnj                             1/1     Running             0          7h48m
kube-system                    kube-proxy-4q852                                  1/1     Running             0          7h28m
kube-system                    kube-proxy-6bvww                                  1/1     Running             0          8h
kube-system                    kube-scheduler-izwz9go2hn3kv068o5wpdpz            1/1     Running             0          8h
kube-system                    tiller-deploy-7b76b656b5-4sp9x                    1/1     Running             0          11m
kubesphere-controls-system     default-http-backend-5d464dd566-f5w6w             1/1     Running             0          3m48s
kubesphere-monitoring-system   kube-state-metrics-566cdbcb48-n9rwr               0/4     ContainerCreating   0          2m58s
kubesphere-monitoring-system   node-exporter-99mjb                               0/2     ContainerCreating   0          2m59s
kubesphere-monitoring-system   node-exporter-b6msl                               2/2     Running             0          2m59s
kubesphere-monitoring-system   prometheus-k8s-0                                  0/3     Pending             0          38s
kubesphere-monitoring-system   prometheus-k8s-system-0                           0/3     Pending             0          38s
kubesphere-monitoring-system   prometheus-operator-6b97679cfd-lrjnm              1/1     Running             0          2m59s
kubesphere-system              ks-account-596657f8c6-6vv8f                       0/1     PodInitializing     0          3m31s
kubesphere-system              ks-apigateway-78bcdc8ffc-kkf9z                    1/1     Running             0          3m34s
kubesphere-system              ks-apiserver-5b548d7c5c-z7xkr                     1/1     Running             0          3m33s
kubesphere-system              ks-console-78bcf96dbf-bxwd4                       1/1     Running             0          3m27s
kubesphere-system              ks-controller-manager-696986f8d9-rlktc            1/1     Running             0          3m30s
kubesphere-system              ks-installer-75b8d89dff-qhhgl                     1/1     Running             0          5m12s
kubesphere-system              openldap-0                                        1/1     Running             0          4m2s
kubesphere-system              redis-6fd6c6d6f9-s9zsr                            1/1     Running             0          4m8s
openebs                        maya-apiserver-7f664b95bb-b2ptv                   1/1     Running             0          51m
openebs                        openebs-admission-server-85dcbc7979-g5dfl         1/1     Running             0          52m
openebs                        openebs-apiserver-bc55cd99b-mtgnb                 1/1     Running             0          77m
openebs                        openebs-localpv-provisioner-85ff89dd44-5ql55      1/1     Running             0          77m
openebs                        openebs-ndm-operator-87df44d9-sbfsw               1/1     Running             1          77m
openebs                        openebs-ndm-sjrjn                                 1/1     Running             0          77m
openebs                        openebs-ndm-sq9ds                                 1/1     Running             0          77m
openebs                        openebs-provisioner-7f86c6bb64-br2pb              1/1     Running             0          77m
openebs                        openebs-snapshot-operator-54b9c886bf-j5xp2        2/2     Running             0          77m
```

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210514100526.png)

## KubeSphere 3.1 安装
确保您的机器满足安装的前提条件之后，可以按照以下步骤最小化安装 KubeSphere。

#### 执行安装命令
```bash
zhangqinghua$ kubectl apply -f https://github.com/kubesphere/ks-installer/releases/download/v3.1.0/kubesphere-installer.yaml

zhangqinghua$ kubectl apply -f https://github.com/kubesphere/ks-installer/releases/download/v3.1.0/cluster-configuration.yaml
```

#### 检查安装日志
```bash
zhangqinghua$ kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l app=ks-install -o jsonpath='{.items[0].metadata.name}') -f
```

#### 等待安装完成
使用 `kubectl get pod --all-namespaces` 查看所有 Pod 是否在 KubeSphere 的相关命名空间中正常运行。如果是，请通过以下命令检查控制台的端口（默认为 30880）：

```bash
zhangqinghua$ kubectl get svc/ks-console -n kubesphere-system
NAME         TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
ks-console   NodePort   10.96.138.81   <none>        80:30880/TCP   116m
```

#### 登陆管理后台
确保在安全组中打开了端口 30880，并通过 NodePort (IP:30880) 使用默认帐户和密码 (admin/P@88w0rd) 访问 Web 控制台。

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210515161612.png)

## 启用可插拔组件
https://kubesphere.io/zh/docs/pluggable-components/

## 卸载可插拔组件
https://kubesphere.io/zh/docs/faq/installation/uninstall-pluggable-components/

## 卸载 KubeSphere 2.1 / 3.1
您可以使用 [kubesphere-delete.sh](https://github.com/kubesphere/ks-installer/blob/master/scripts/kubesphere-delete.sh) 将 KubeSphere 从您现有的 Kubernetes 集群中卸载。复制 GitHub 源文件并在本地机器上执行此脚本。

```bash
zhangqinghua$ sudo bash kubesphere-delete.sh
Note:
Delete the KubeSphere cluster, including the module kubesphere-system kubesphere-devops-system kubesphere-monitoring-system kubesphere-logging-system openpitrix-system.
Please reconfirm that you want to delete the KubeSphere cluster.  (yes/no) yes
deployment.apps "ks-installer" deleted
...
...
```

> 1. 卸载意味着 KubeSphere 会从您的 Kubernetes 集群中移除。此操作不可逆并且没有任何备份，请谨慎操作。

> 2. 直接执行此脚本可能会卡死，重启服务器后再试一下即可。等半个小时左右即可。

## 常见问题
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