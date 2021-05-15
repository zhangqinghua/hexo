---
title: KubeSphere 安装

categories:
- 部署运维
- Kubernetes 手册

date: 2021-05-13
---
## KubeSphere 2.1 安装


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

## KubeSphere 3.1 启用 DevOps
#### 进入集群管理
以 admin 身份登录控制台，点击左上角的平台管理，选择集群管理。

![](https://kubesphere.io/images/docs/zh-cn/enable-pluggable-components/kubesphere-devops-system/clusters-management.png)

#### 选择资源 CRD
点击自定义资源 CRD，在搜索栏中输入 clusterconfiguration，点击搜索结果查看其详细页面
![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210515162318.png)

> 自定义资源定义（CRD）允许用户在不增加额外 API 服务器的情况下创建一种新的资源类型，用户可以像使用其他 Kubernetes 原生对象一样使用这些自定义资源。

#### 编辑配置文件
在资源列表中，点击 ks-installer 右边的三个点，选择编辑配置文件。

![](https://kubesphere.io/images/docs/zh-cn/enable-pluggable-components/kubesphere-devops-system/edit-yaml.PNG)

在该 YAML 文件中，搜寻到 devops，将 enabled 的 false 改为 true。完成后，点击右下角的更新，保存配置：

```yml
devops:
    enabled: true # Change "false" to "true"
```

#### 查看安装过程
您可以使用 Web Kubectl 工具执行以下命令来检查安装过程：

```bash
zhangqinghua$ kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l app=ks-install -o jsonpath='{.items[0].metadata.name}') -f
```

> 您可以通过点击控制台右下角的锤子图标找到 Web Kubectl 工具。

## 卸载 KubeSphere 2.1
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

> 直接执行此脚本可能会卡死，重启服务器后再试一下即可。等半个小时左右即可。