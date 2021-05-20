---
title: KubeSphere 常用操作

categories:
- 部署运维
- Kubernetes 手册

date: 2021-05-13
---
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

## 污点管理
**1. 选择一个节点**
![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210514121835.png)

**2. 选中污点管理**
![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210514121921.png)

**3. 删除污点**
![](https://cdn.jsdelivr.net/gh/zhangqinghua/hexo_image/20210514121951.png)

删除污点后，可以看到 pod 部署到 master 节点上来了。

```bash
zhangqinghua$ kubectl get pods -o wide
NAME                       READY   STATUS    RESTARTS   AGE     IP             NODE                      NOMINATED NODE   READINESS GATES
tomcat6-5f7ccf4cb9-25p6t   1/1     Running   0          5m23s   10.244.0.8     izwz9go2hn3kv068o5wpdpz   <none>           <none>
tomcat6-5f7ccf4cb9-5gcdl   1/1     Running   0          5m23s   10.244.0.7     izwz9go2hn3kv068o5wpdpz   <none>           <none>
tomcat6-5f7ccf4cb9-78b5h   1/1     Running   0          5m23s   10.244.0.9     izwz9go2hn3kv068o5wpdpz   <none>           <none>
tomcat6-5f7ccf4cb9-7xgj7   1/1     Running   0          51m     10.244.1.181   easybyte-dev-server       <none>           <none>
tomcat6-5f7ccf4cb9-n5qhc   1/1     Running   0          5m23s   10.244.0.6     izwz9go2hn3kv068o5wpdpz   <none>           <none>
tomcat6-5f7ccf4cb9-wn7ml   1/1     Running   0          5m23s   10.244.1.204   easybyte-dev-server       <none>           <none>
```