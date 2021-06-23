---
title: Kubernetes 安装

categories:
- 部署运维
- Kubernetes 手册

date: 2021-05-13 09:00:10
---
## 部署内网集群
![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210513161436.png)

#### 环境配置
需要在 Master 节点和 Node 节点执行以下操作：

```bash
# 关闭防火墙
zhangqinghua$ sudo systemctl stop firewalld && sudo systemctl disable firewalld   

# 关闭selinux
zhangqinghua$ sudo setenforce 0 && sudo sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config

# 关闭swap
zhangqinghua$ sudo swapoff -a && sudo yes | cp /etc/fstab /etc/fstab_bak && sudo cat /etc/fstab_bak |grep -v swap > /etc/fstab
```

#### 卸载 K8s
参考：[卸载清理K8s环境](https://blog.csdn.net/qq_36246184/article/details/109158027)

#### 卸载 Docker

```bash
# 查看已经安装的docker安装包,列出入校内容
zhangqinghua$ rpm -qa|grep docker
docker.x86_64 2:1.12.6-16.el7.centos @extras
docker-client.x86_64 2:1.12.6-16.el7.centos @extras
docker-common.x86_64 2:1.12.6-16.el7.centos @extra

# 分别删除
zhangqinghua$ yum -y remove docker.x86_64
zhangqinghua$ yum -y remove docker-client.x86_64
zhangqinghua$ yum -y remove docker-common.x86_64

# 删除 Docker 镜像
zhangqinghua$ rm -rf /var/lib/docker
```

#### 安装 Docker
需要在 Master 节点和 Node 节点执行以下操作：

```bash
# 配置Docker的yum源
zhangqinghua$ sudo yum -y install -y yum-utils device-mapper-persistent-data lvm2
zhangqinghua$  sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 更新containerd.io
zhangqinghua$ sudo dnf install -y \
https://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.6-3.3.el7.x86_64.rpm

# 安装 Docker（由于kubeadm对Docker的版本是有要求的，需要安装与Kubernetes匹配的版本）
zhangqinghua$ sudo yum install -y docker-ce-20.10.6 docker-ce-cli-20.10.6 containerd.io

# 设置默认启动
zhangqinghua$ sudo systemctl enable docker & sudo systemctl start docker
```

#### 安装 Kubernetes
需要在 Master 节点和 Node 节点执行以下操作：

```bash
# 配置 yum 源（国内机器需要）
zhangqinghua$ sudo cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
       http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

# 安装 Kubernetes（这里使用1.17.3版本，要兼容Docker）
zhangqinghua$ sudo yum install -y kubelet-1.17.3 kubeadm-1.17.3 kubectl-1.17.3 --disableexcludes=kubernete

# 设置默认启动
zhangqinghua$ sudo systemctl enable --now kubelet
```

#### Master 节点初始化
```bash
# apiserver-advertise-address 公网/内网IP
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
```

初始化完毕后，再执行以下命令：

```bash
zhangqinghua$ mkdir -p $HOME/.kube

zhangqinghua$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

zhangqinghua$ chown $(id -u):$(id -g) $HOME/.kube/config
```

`kubeadm join xxx` 就是 Node 节点加入集群的命令，2 个小时内有效。如果失效了可以再生成一个永久有效的命令：

```bash
zhangqinghua$ kubeadm token create --ttl 0 --print-join-command
kubeadm join 172.27.243.201:6443 --token b52699.ed59k5rb47b2epm3 \
    --discovery-token-ca-cert-hash sha256:02ac09f2b530e4b062585fb6b9d5031b65a8a9ed46e9e291e081bb41fb781315
```

#### 安装 Pod 网络插件
```bash
zhangqinghua$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

如果网络出现问题，关闭 cni0，重启虚拟机继续测试 执行 `watch kubectl get pod -n kube-system -o wide` 监控 pod 进度等 3-10 分钟，完全都是 running 以后再继续。

```bash
zhangqinghua$ ip link set cni0 down
```

#### Node 节点加入集群
只需要在 Node 节点执行上面生成的命令即可。

```bash
zhangqinghua$ kubeadm join 172.27.243.201:6443 --token b52699.ed59k5rb47b2epm3 \
                      --discovery-token-ca-cert-hash sha256:02ac09f2b530e4b062585fb6b9d5031b65a8a9ed46e9e291e081bb41fb781315
W0522 11:57:49.573586 1357475 join.go:346] [preflight] WARNING: JoinControlPane.controlPlane settings will be ignored when control-plane flag is not set.
[preflight] Running pre-flight checks
        [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
        [WARNING FileExisting-tc]: tc not found in system path
        [WARNING SystemVerification]: this Docker version is not on the list of validated versions: 20.10.6. Latest validated version: 19.03
        [WARNING Service-Kubelet]: kubelet service is not enabled, please run 'systemctl enable kubelet.service'
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.17" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster. 
```

#### Node 节点退出集群
**1. 在 Master 节点删除操作**
```bash
# 查询所有的节点信息
zhangqinghua$ kubectl get nodes
NAME                      STATUS   ROLES                  AGE   VERSION
izj6c081jz2b47q77ff3bfz   Ready    <none>                 26m   v1.17.3
k8s-master                Ready    control-plane,master   29m   v1.21.1

# 删除节点
zhangqinghua$ kubectl delete node izj6c081jz2b47q77ff3bfz
node "izj6c081jz2b47q77ff3bfz" deleted
```

**2. 在 Node 节点清空集群数据信息**
```bash
# 重置 kubeadm
zhangqinghua$ kubeadm reset

# 清理数据
zhangqinghua$ rm -rf /etc/cni/net.d
zhangqinghua$ rm -rf $HOME/.kube
zhangqinghua$ ifconfig cni0 down
zhangqinghua$ ip link delete cni0
zhangqinghua$ ifconfig flannel.1 down
zhangqinghua$ ip link delete flannel.1
zhangqinghua$ rm -rf /var/lib/cni/
zhangqinghua$ rm -rf /var/lib/etcd
```

#### 最终效果
```bash
zhangqinghua$ kubectl get nodes
NAME                      STATUS     ROLES    AGE   VERSION
izwz9go2hn3kv068o5wpdpz   NotReady   master   13m   v1.17.3
```

目前 master 状态为 `notready`，等待网络加入完成即可。

## 部署公网集群
#### Master 节点初始化
Master 节点在执行 `kubeadm init` 的时候只需要制定公网 IP 即可。

```bash
zhangqinghua$ kubeadm init \
                      --apiserver-advertise-address=公网IP \
                      --image-repository registry.cn-hangzhou.aliyuncs.com/google_containers \
                      --kubernetes-version v1.17.3 \
                      --service-cidr=10.96.0.0/16 \
                      --pod-network-cidr=10.244.0.0/16
```

阿里云 ECS 网卡没有配置公网ip，因此就无法指定该ip，导致 etcd 无法正常启动，需要特殊处理。

#### 手工修改
```bash
# 47.242.9.184 为公网IP
zhangqinghua$ sudo kubeadm init  --apiserver-advertise-address=47.242.9.184
```

参考：https://zhuanlan.zhihu.com/p/74134318

参考：https://www.cnblogs.com/life-of-coding/p/11879067.html
#### 修复验证
手工修改后，执行 kubectl 命令会报错。

```bash
x509: certificate is valid for 10.96.0.1, 172.18.255.243, not 内网IP
```

解决：加上 `--apiserver-cert-extra-sans=内网IP`

```bash
# 172.25.25.244 为内网IP
zhangqinghua$ sudo kubeadm init  --apiserver-advertise-address=47.242.9.184 --apiserver-cert-extra-sans=172.25.25.244
```
参考：https://www.cnblogs.com/huhyoung/p/9738126.html

#### 证书修复
修复验证后，执行 kubectl 命令会报错。

```
Unable to connect to the server: x509: certificate signed by unknown authority (possibly because of “crypto/rsa: verification error” while trying to verify candidate authority certificate “kubernetes”)
```

执行以下命令即可：

```bash
zhangqinghus$ export KUBECONFIG=/etc/kubernetes/kubelet.conf
```

参考：https://blog.csdn.net/qq_34857250/article/details/82562514

#### 最终效果
再分别加入内网节点（izj6c081jz2b47q77ff3bfz）和公网节点（k8s-master）后，在 Master 可以查看到：

```bash
zhangqinghua$ kubectl get nodes
NAME                      STATUS   ROLES    AGE   VERSION
izj6c081jz2b47q77ff3bez   Ready    master   20m   v1.17.3
izj6c081jz2b47q77ff3bfz   Ready    <none>   18m   v1.17.3
k8s-master                Ready    <none>   14m   v1.17.3
```

查看运行的 pod：

```bash
zhangqinghua$ kubectl get pods -o wide --all-namespaces
NAMESPACE     NAME                                              READY   STATUS              RESTARTS   AGE   IP               NODE                      NOMINATED NODE   READINESS GATES
kube-system   coredns-6955765f44-2pzvw                          0/1     ContainerCreating   0          22m   <none>           izj6c081jz2b47q77ff3bez   <none>           <none>
kube-system   coredns-6955765f44-h8nwd                          0/1     ContainerCreating   0          22m   <none>           izj6c081jz2b47q77ff3bez   <none>           <none>
kube-system   etcd-izj6c081jz2b47q77ff3bez                      1/1     Running             0          22m   172.25.25.244    izj6c081jz2b47q77ff3bez   <none>           <none>
kube-system   kube-apiserver-izj6c081jz2b47q77ff3bez            1/1     Running             0          22m   172.25.25.244    izj6c081jz2b47q77ff3bez   <none>           <none>
kube-system   kube-controller-manager-izj6c081jz2b47q77ff3bez   1/1     Running             0          22m   172.25.25.244    izj6c081jz2b47q77ff3bez   <none>           <none>
kube-system   kube-proxy-42wq6                                  1/1     Running             0          15m   141.164.53.117   k8s-master                <none>           <none>
kube-system   kube-proxy-6s68d                                  1/1     Running             0          22m   172.25.25.244    izj6c081jz2b47q77ff3bez   <none>           <none>
kube-system   kube-proxy-nrqgd                                  1/1     Running             0          20m   172.25.25.245    izj6c081jz2b47q77ff3bfz   <none>           <none>
kube-system   kube-scheduler-izj6c081jz2b47q77ff3bez            1/1     Running             0          22m   172.25.25.244    izj6c081jz2b47q77ff3bez   <none>           <none>
```

测试 2 个节点的 pod 能否 ping 通。

```bash
zhangqinghua$ ping 172.25.25.244
PING 172.25.25.244 (172.25.25.244) 56(84) bytes of data.
64 bytes from 172.25.25.244: icmp_seq=1 ttl=64 time=0.083 ms

zhangqinghua$ ping 141.164.53.117
PING 141.164.53.117 (141.164.53.117) 56(84) bytes of data.
64 bytes from 141.164.53.117: icmp_seq=1 ttl=53 time=37.1 ms
```

## 修改集群
#### 修改 Master 节点名称

#### 修改 Node 节点名称
**1. 在 Master 节点删除想要改名的 Worker 节点**
```bash
zhangqinghua$ kubectl delete node {your worker name name}
```

**2. 在 Worker 节点停止 kubelet 服务**
```bash
zhangqinghua$ systemctl stop kubelet
```

**3. 删除之前通过 csr 请求后产生的证书、秘钥、kubelet.conf 文件**
```bash
# ​ 查找到kubelet.conf的位置，找到之后打开并且查看pki的目录 
zhangqinghua$ find . -name kubelet.conf

zhangqinghua$ rm -rf /etc/kubernetes/pki
```

**4. 修改节点名称**
```bash
zhangqinghua$ hostnamectl set-hostname xxx
```

**4. 在 worker 节点上重启 kubelet 服务**
```bash
zhangqinghua$ systemctl start kubelet
```

## 卸载集群
每个节点都执行一遍。
```bash
zhangqinghua$ sudo yum remove -y kubelet kubeadm kubectl
```

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

**[ERROR Port-10257]: Port 10257 is in use**
场景：进行 kubectl init 的时候报错。
原因：端口被占用。
解决：`kubeadm reset`

**The reset process does not clean your kubeconfig files and you must remove them manually.**
场景：reset master 节点，报错。
解决：`kubectl init`

**Error: error initializing: Looks like "https://kubernetes-charts.storage.googleapis.com"**
场景：安装 Tiller2 失败。
原因：未知。
解决：https://blog.csdn.net/weixin_41806245/article/details/98631199

```bash
zhangqinghua$ helm init --upgrade --tiller-image registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.16.3 --stable-repo-url https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
```

**删除namespace时卡在Terminating状态**
参考：https://blog.csdn.net/weweeeeeeee/article/details/100036417
参考：https://blog.csdn.net/Tilyp/article/details/89226235

**Error from server (Forbidden): namespaces is forbidden: User "system:node:izj6c081jz2b47q77ff3bez" cannot create resource "namespaces" in API group "" at the cluster scope**
原因：集群部署完成后，kubectl 执行写资源没有权限。
原因：apiserver 权限的问题
解决：`export KUBECONFIG=/etc/kubernetes/admin.conf`
参考：https://blog.csdn.net/ywq935/article/details/80109090

**network plugin is not ready: cni config uninitialized**
场景：node 节点加入后，`kubectl get nodes` 提示。

```bash
zhangqinghua$ kubectl get nodes
NAME         STATUS     ROLES    AGE     VERSION
k8s-node1    NotReady   <none>   8m59s   v1.17.3
ks8-master   NotReady   master   10m     v1.17.3

zhangqinghua$ kubectl describe node k8s-node1
  Type             Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----             ------  -----------------                 ------------------                ------                       -------
  MemoryPressure   False   Sat, 22 May 2021 17:50:03 +0800   Sat, 22 May 2021 17:44:52 +0800   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure     False   Sat, 22 May 2021 17:50:03 +0800   Sat, 22 May 2021 17:44:52 +0800   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure      False   Sat, 22 May 2021 17:50:03 +0800   Sat, 22 May 2021 17:44:52 +0800   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready            False   Sat, 22 May 2021 17:50:03 +0800   Sat, 22 May 2021 17:44:52 +0800   KubeletNotReady              runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady message:docker: network plugin is not ready: cni config uninitialized
```

原因：没有按照 cni 模块。
解决：安装 fluent
参考：https://blog.csdn.net/weixin_40161254/article/details/112203691

**从节点 pod id ping 不通**
场景：安装集群之后，从节点的 pod ping 不通。

```bash
zhangqinghua$ kubectl get pods -o wide --all-namespaces
NAMESPACE                      NAME                                               READY   STATUS    RESTARTS   AGE   IP              NODE          NOMINATED NODE   READINESS GATES
kube-federation-system         kubefed-admission-webhook-7c55679bdf-nc45s         1/1     Running   0          8d    10.244.1.18     kube-node01   <none>           <none>
kube-federation-system         kubefed-controller-manager-5db964797d-z42bs        1/1     Running   0          8d    10.244.1.19     kube-node01   <none>           <none>
kube-system                    coredns-7f9c544f75-5pz9r                           1/1     Running   0          8d    10.244.0.2      kube-master   <none>           <none>

zhangqinghua$ ping 10.244.1.18
PING 10.244.1.18 (10.244.1.18) 56(84) bytes of data.
```

原因：Master 节点的防火墙开了，导致从节点连不上。
解决：关掉 Master 的防火墙。

**kubelet 启动异常：kubelet.go:2263] node "kube-node01" not found**
场景：从节点启动 kubelet 报错：`May 31 10:44:33 kube-node01 kubelet[1427]: E0531 10:44:33.161795    1427 kubelet.go:2263] node "kube-node01" not found`。
原因：Master 节点的防火墙开了，导致从节点连不上。
解决：关掉 Master 的防火墙。

**从节点加入失败： error execution phase preflight: couldn't validate the identity of the API Server: abort connecting to API servers after timeout**
场景：从节点加入一直没有进度。

```bash
[root@kube-node01 ~]# kubeadm join 172.25.25.244:6443 --token d3qa8m.9live0v3z817t6km     --discovery-token-ca-cert-hash sha256:fff8c69e23cce925eefa5e00f564c3ff71d68a2ec87086b8853dedc782e26522
W0531 10:50:58.497804    2566 join.go:346] [preflight] WARNING: JoinControlPane.controlPlane settings will be ignored when control-plane flag is not set.
[preflight] Running pre-flight checks
        [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
        [WARNING FileExisting-tc]: tc not found in system path
        [WARNING SystemVerification]: this Docker version is not on the list of validated versions: 20.10.6. Latest validated version: 19.03
        [WARNING Hostname]: hostname "kube-node01" could not be reached
        [WARNING Hostname]: hostname "kube-node01": lookup kube-node01 on 100.100.2.136:53: no such host
        [WARNING Service-Kubelet]: kubelet service is not enabled, please run 'systemctl enable kubelet.service'
error execution phase preflight: couldn't validate the identity of the API Server: abort connecting to API servers after timeout of 5m0s
To see the stack trace of this error execute with --v=5 or higher
```

原因：Master 节点的防火墙开了，导致从节点连不上。
解决：关掉 Master 的防火墙。

**yum 安装 Docker 失败：Error: Unable to find a match: docker-ce**
场景：
