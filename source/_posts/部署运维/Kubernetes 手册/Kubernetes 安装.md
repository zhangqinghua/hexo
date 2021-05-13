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

#### 最终效果

## 常见问题
**Error: Unable to find a match: kubelet-1.17.3 kubeadm-1.17.3 kubectl-1.17.3**
场景：执行 yum 命令 `yum install -y kubelet-1.17.3 kubeadm-1.17.3 kubectl-1.17.3` 报错.
原因：没有配置 k8s 的 yum 源
解决：https://blog.csdn.net/m0_37556444/article/details/86494294
