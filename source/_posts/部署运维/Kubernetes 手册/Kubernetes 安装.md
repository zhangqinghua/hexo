---
title: Kubernetes 安装

categories:
- 部署运维
- Kubernetes 手册

date: 2021-05-13
---
## k8s 集群安装
前置要求：
1. 一台或多台机器，操作系统 CentOS7.x-86_x64
1. 硬件配置：2GB 或更多 RAM，2 个 CPU 或更多 CPU，硬盘 30GB 或更多
1. 集群中所有机器之间网络互通
1. 可以访问外网，需要拉取镜像
1. 禁止 swap 分区

部署步骤：
1. 在所有节点上安装 Docker 和 kubeadm
2. 部署 Kubernetes Master
3. 部署容器网络插件
4. 部署 Kubernetes Node，将节点加入 Kubernetes 集群中
5. 部署 Dashboard Web 页面，可视化查看 Kubernetes 资源

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210513161436.png)

#### 在所有节点上安装 Docker 和 kubeadm

#### 部署 Kubernetes Master
**master 节点初始化**
```bash
zhangqinghua$ kubeadm init \
                       --apiserver-advertise-address=172.27.243.201 \
                       --image-repository registry.cn-hangzhou.aliyuncs.com/google_containers \
                       --kubernetes-version v1.17.3 \
                       --service-cidr=10.96.0.0/16 \
                       --pod-network-cidr=10.244.0.0/16

...
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.27.243.201:6443 --token b52699.ed59k5rb47b2epm3 \
    --discovery-token-ca-cert-hash sha256:02ac09f2b530e4b062585fb6b9d5031b65a8a9ed46e9e291e081bb41fb781315 

zhangqinghua$ 
```

参数说明：
1. `apiserver-advertise-address`
   本机 IP 地址。
1. `image-repository`
   由于默认拉取镜像地址 `k8s.gcr.io` 国内无法访问，这里指定阿里云镜像仓库地址。可以手动按照我们的 images.sh 先拉取镜像，地址变为 `registry.aliyuncs.com/google_containers` 也可以。
1. `kubernetes-version`
   版本号，跟上面 一致。
1. `service-cidr` 和 `pod-network-cidr` 
   参考默认的即可。

运行完成后，根据提示需要执行 3 个命令：

```bash
zhangqinghua$ mkdir -p $HOME/.kube

zhangqinghua$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

zhangqinghua$ chown $(id -u):$(id -g) $HOME/.kube/config
```

如果 worker 节点需要加入主节点，只需要把 `kubeadm join` 命令在副节点执行一遍即可，一般情况下这个命令 2 个小时内有效。如果失效了可以再生成一个永久有效的命令：

```bash
zhangqinghua$ kebeadm token create --ttl 0 --print-join-command
kubeadm join 172.27.243.201:6443 --token b52699.ed59k5rb47b2epm3 \
    --discovery-token-ca-cert-hash sha256:02ac09f2b530e4b062585fb6b9d5031b65a8a9ed46e9e291e081bb41fb781315 
```

**测试 kubectl（主节点）**
```bash
zhangqinghua$ kubectl get nodes
NAME                      STATUS     ROLES    AGE   VERSION
izwz9go2hn3kv068o5wpdpz   NotReady   master   13m   v1.17.3
```

目前 master 状态为 `notready`，等待网络加入完成即可。

#### 部署容器网络插件

#### 部署 Kubernetes Node，将节点加入 Kubernetes 集群中

#### 部署 Dashboard Web 页面，可视化查看 Kubernetes 资源


#### 安装 KubeSphere
**1. 安装 Helm**
```bash
zhangqinghua$ wget https://get.helm.sh/helm-v2.16.3-linux-amd64.tar.gz
...
zhangqinghua$ ll
-rw-r--r-- 1 root root 25097357 May 13 20:07 helm-v2.17.0-linux-amd64.tar.gz

zhangqinghua$ tar -zxvf helm-v2.17.0-linux-amd64.tar.gz 
linux-amd64/
linux-amd64/README.md
linux-amd64/LICENSE
linux-amd64/helm
linux-amd64/tiller

zhangqinghua$ helm version
Client: &version.Version{SemVer:"v2.17.0", GitCommit:"a690bad98af45b015bd3da1a41f6218b1a451dbe", GitTreeState:"clean"}
Error: could not find tiller
```

**2. 安装 tiller**
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

然后执行命令：

```bash
zhangqinghua$ helm init --service-account=tiller --tiller-image=registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.16.3   --history-max 300
Creating /root/.helm 
Creating /root/.helm/repository 
Creating /root/.helm/repository/cache 
Creating /root/.helm/repository/local 
Creating /root/.helm/plugins 
Creating /root/.helm/starters 
Creating /root/.helm/cache/archive 
Creating /root/.helm/repository/repositories.yaml 
Adding stable repo with URL: https://charts.helm.sh/stable 
Adding local repo with URL: http://127.0.0.1:8879/charts 
$HELM_HOME has been configured at /root/.helm.

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

**3. 设置默认存储类型**

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

**4. 创建 OpenESB 命名空间**
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

**5. Helm 安装 OpenESB**
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

**6. 将 openebs-hostpath 设置为 默认的 StorageClass**
```bash
zhangqinghua$ kubectl patch storageclass openebs-hostpath -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
storageclass.storage.k8s.io/openebs-hostpath patched
```

**7. 恢复 Master 节点的 Taint**
至此，OpenEBS 的 LocalPV 已作为默认的存储类型创建成功。由于在文档开头手动去掉 了 master 节点的 Taint，我们可以在安装完 OpenEBS 后将 master 节点 Taint 加上，避 免业务相关的工作负载调度到 master 节点抢占 master 资源。
```bash
zhangqinghua$ kubectl taint nodes k8s-node1 node-role.kubernetes.io=master:NoSchedule
```

#### KubeSphere 最小化安装
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

#### 最终效果


## 常见问题
**Error: Unable to find a match: kubelet-1.17.3 kubeadm-1.17.3 kubectl-1.17.3**
场景：执行 yum 命令 `yum install -y kubelet-1.17.3 kubeadm-1.17.3 kubectl-1.17.3` 报错.
原因：没有配置 k8s 的 yum 源
解决：https://blog.csdn.net/m0_37556444/article/details/86494294

**Error: incompatible versions client[v2.17.0] server[v2.16.3]**
场景：Helm 安装服务的时候报错。
原因：Helm 和 Tiller 的版本不一致。
解决：卸载掉 Tiller 重新安装：https://www.jianshu.com/p/d0cdbb49569b

**fatal: [localhost]: FAILED! => {“changed”: true,**
场景：最小化安装 KubeSphere 时，查看日志得到的报错。
原因：Helm 版本不对，helm的版本不匹配导致的。当前安装的版本是v2.17.0，重新安装的版本是v2.16.3。
解决：卸载 Helm 和 Tiller 重新安装：https://blog.csdn.net/qq_30019911/article/details/113747673