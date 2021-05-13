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
zhangqinghua$ wget https://get.helm.sh/helm-v2.17.0-linux-amd64.tar.gz
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
zhangqinghua$ helm init --service-account=tiller --tiller-image=jessestuart/tiller:v2.16.3 --history-max 300
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

确认 Master 节点是否有Taint，如下看到 Master 节点有 Taint。

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

接下来安装 



#### 最终效果

## 常见问题
**Error: Unable to find a match: kubelet-1.17.3 kubeadm-1.17.3 kubectl-1.17.3**
场景：执行 yum 命令 `yum install -y kubelet-1.17.3 kubeadm-1.17.3 kubectl-1.17.3` 报错.
原因：没有配置 k8s 的 yum 源
解决：https://blog.csdn.net/m0_37556444/article/details/86494294
