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
TASK [common : KubeSphere | Creating manifests] ********************************
ok: [localhost] => (item={'name': 'custom-values-minio', 'file': 'custom-values-minio.yaml'})

TASK [common : KubeSphere | Checking minio] ************************************
changed: [localhost]

TASK [common : KubeSphere | Deploying minio] ***********************************
changed: [localhost]

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
Name:           minio-make-bucket-job-2p5pr
Namespace:      kubesphere-system
Priority:       0
Node:           <none>
Labels:         app=minio-job
                controller-uid=42d3a7bf-587c-4d92-bdc7-692256a25c08
                job-name=minio-make-bucket-job
                release=ks-minio
Annotations:    <none>
Status:         Pending
IP:             
IPs:            <none>
Controlled By:  Job/minio-make-bucket-job
Containers:
  minio-mc:
    Image:      minio/mc:RELEASE.2019-08-07T23-14-43Z
    Port:       <none>
    Host Port:  <none>
    Command:
      /bin/sh
      /config/initialize
    Requests:
      cpu:     250m
      memory:  256Mi
    Environment:
      MINIO_ENDPOINT:  minio
      MINIO_PORT:      9000
    Mounts:
      /config from minio-configuration (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-7j7f7 (ro)
Conditions:
  Type           Status
  PodScheduled   False 
Volumes:
  minio-configuration:
    Type:                Projected (a volume that contains injected data from multiple sources)
    ConfigMapName:       minio
    ConfigMapOptional:   <nil>
    SecretName:          minio
    SecretOptionalName:  <nil>
  default-token-7j7f7:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-7j7f7
    Optional:    false
QoS Class:       Burstable
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason            Age                 From               Message
  ----     ------            ----                ----               -------
  Warning  FailedScheduling  72s (x26 over 27m)  default-scheduler  0/2 nodes are available: 1 Insufficient cpu, 1 node(s) had taints that the pod didn't tolerate.
```
解决：开放一个节点的污点即可。

#### 创建DevOps项目失败
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