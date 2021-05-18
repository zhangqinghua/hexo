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

在该 YAML 文件中，搜寻到 `devops`，将 `enabled` 的 `false` 改为 `true`。完成后，点击右下角的更新，保存配置：

```yml
devops:
    enabled: true # Change "false" to "true"
```

#### 查看安装过程
您可以使用 Web Kubectl 工具执行以下命令来检查安装过程，整个过程需要半个小时左右：

```bash
zhangqinghua$ kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l app=ks-install -o jsonpath='{.items[0].metadata.name}') -f
```

> 您可以通过点击控制台右下角的锤子图标找到 Web Kubectl 工具。

#### 验证安装结果
进入服务组件，检查 DevOps 的状态，可以看到如下类似图片：

![](https://kubesphere.io/images/docs/zh-cn/enable-pluggable-components/kubesphere-devops-system/devops.PNG)

## KubeSphere 3.1 开启应用路由
本文档介绍了如何在 KubeSphere 上创建、使用和编辑应用路由。

#### 准备工作
1. 您需要创建一个企业空间、一个项目以及两个帐户（例如，project-admin 和 project-regular）。在此项目中，project-admin 必须具有 admin 角色，project-regular 必须具有 operator 角色。有关更多信息，请参见创建企业空间、项目、帐户和角色。
1. 若要以 HTTPS 模式访问应用路由，则需要创建密钥用于加密，密钥中需要包含 tls.crt（TLS 证书）和 tls.key（TLS 私钥）。
1. 您需要创建至少一个服务。本文档使用演示服务作为示例，该服务会将 Pod 名称返回给外部请求。

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210515170616.png)


#### 配置访问方式
1. 以 project-admin 身份登录 KubeSphere 的 Web 控制台，然后访问您的项目；
1. 在左侧导航栏中选择项目设置下的高级设置，点击右侧的设置网关；
1. 在出现的设置网关对话框中，将访问方式设置为 NodePort 或 LoadBalancer，然后点击保存；

![](https://v3-1.docs.kubesphere.io/images/docs/zh-cn/project-user-guide/application-workloads/routes/set-gateway.png)

![](https://v3-1.docs.kubesphere.io/images/docs/zh-cn/project-user-guide/application-workloads/routes/access-method-nodeport.png)

> 若将访问方式设置为 LoadBalancer，则可能需要根据插件用户指南在您的环境中启用负载均衡器插件。

#### 创建应用路由
**1. 配置基本信息**
1. 登出 KubeSphere 的 Web 控制台，以 project-regular 身份登录，并访问同一个项目。
1. 选择左侧导航栏应用负载中的应用路由，点击右侧的创建。
1. 在基本信息选项卡中，配置应用路由的基本信息，并点击下一步。
   名称：应用路由的名称，用作此应用路由的唯一标识符。
   别名：应用路由的别名。
   描述信息：应用路由的描述信息。

![](https://v3-1.docs.kubesphere.io/images/docs/zh-cn/project-user-guide/application-workloads/routes/create-route.png)

![](https://v3-1.docs.kubesphere.io/images/docs/zh-cn/project-user-guide/application-workloads/routes/basic-info.png)

**2. 配置路由规则**
1. 在路由规则选项卡中，点击添加路由规则。
1. 选择一种模式来配置路由规则，点击 √，然后点击下一步。

![](https://v3-1.docs.kubesphere.io/images/docs/zh-cn/project-user-guide/application-workloads/routes/auto-generate.png)

**3. 获取域名、服务路径和网关地址**
1. 在左侧导航栏中选择应用负载中的应用路由，点击右侧的应用路由名称。
2. 在规则区域获取域名和服务路径，在详情区域获取网关地址。

![](https://v3-1.docs.kubesphere.io/images/docs/zh-cn/project-user-guide/application-workloads/routes/route-list.png)

![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210515171604.png)

#### 配置域名解析
若在配置路由规则中选择自动生成，则不需要配置域名解析，域名会自动由 nip.io 解析为网关地址。

若在配置路由规则中选择指定域名，则需要在 DNS 服务器配置域名解析，或者在客户端机器上将<路由网关地址> <路由域名>添加到 `etc/hosts` 文件。

这里我们以指定域名域名的例子测试：

**1. 修改 /etc/hosts 文件**
```
# vim /etc/hosts
120.78.223.168  test.easybyte-hk.com
```

**2. 模拟域名请求**
```bash
zhangqinghua$ curl test.easybyte-hk.com:31967
Server address: 10.244.0.223:80
Server name: tea-6dc6465765-wjctx
Date: 15/May/2021:09:23:35 +0000
URI: /
Request ID: 3f9e1694f661eebefc1015c13bf450a7
```

可以看到，应用路由已经成功解析到服务端口上了。

Nginx 反向代理配置：

```
server {
    listen 80;
    # listen 443 ssl;
    server_name prod.easybyte-hk.com;
    index index.html index.htm index.jsp;
    charset utf-8;
    access_log /data/wwwlogs/dev.easybyte-hk.com.log combined;
    location / {
        proxy_http_version 1.1;
        proxy_pass http://120.78.223.168:31967;
        proxy_redirect off;
        proxy_set_header        Host $host:$server_port;
        proxy_set_header        X-Real-IP $remote_addr;
        proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_connect_timeout  3600s;
        proxy_read_timeout  3600s;
        proxy_send_timeout  3600s;
        send_timeout  3600s;
    }
}
```

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