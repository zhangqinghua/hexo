---
title: KubeSphere 部署应用

categories:
- 部署运维
- Kubernetes 手册

date: 2021-05-13
---

## KubeSphere 部署 Nacos

## KubeSphere 部署 Nacos

## KubeSphere 部署 Vue
1. 现将 Vue 编译称为 dist 静态文件。
1. Docker 将 dist 静态文件打包称为 Nginx 容器。
1. 访问 Nginx 80 端口。

#### 1. 配置 Jenkinsfile
```Jenkinsfile
pipeline {
  agent {
    node {
      label 'nodejs'
    }

  }
  stages {
    stage('拉取代码') {
      steps {
        container('nodejs') {
          git(url: 'https://code.aliyun.com/easybyte-web/easybyte-sys.git', credentialsId: 'aliyuncode-id', branch: 'master', changelog: true, poll: false)
        }
      }
    }
    
    stage('安装依赖') {
      steps {
        container('nodejs') {
          sh 'npm install -g cnpm --registry=https://registry.npm.taobao.org'
          sh 'cnpm i --no-package-lock'
        }
      }
    }
    
    stage('构建项目') {
      steps {
        container('nodejs') {
          sh 'yarn build'
          sh 'ls -a'
        }
      }
    }
    
    stage('打包项目') {
      steps {
        container('nodejs') {
          sh 'docker build -f Dockerfile -t $APP_NAME:$BUILD_NUMBER .'
          
          sh 'docker images'
        }
      }
    }
    
    stage('发布开发') {
      agent none
      steps {
        kubernetesDeploy(enableConfigSubstitution: true, deleteResource: false, kubeconfigId: 'demo-kubeconfig', configs: 'k8s-deploy.yml')
      }
    }
  }

  environment {
    APP_NAME = 'easybyte-front-merchant'
    CREDENTIAL_ID = 'aliyuncode-id'
    BRANCH = 'master'
  }
}
```

#### 2. Dockerfile
```Dockerfile
# Alpine Linux是：一个基于musl libc和busybox、面向安全的轻量级Linux发行版。
# ngixn 打包大小为 133 MB，nginx:stable-alpine 只有 20 MB。
FROM nginx:stable-alpine

# 将dist文件中的内容复制到 /usr/share/nginx/html/merchant 这个目录下面
# 前端访问时使用 http://dev.easybyte-hk.com/merchant/ > http://localhost:80/merchant
COPY dist/  /usr/share/nginx/html/merchant
```

#### 3. Kube Pod 资源清单
```yml
# 服务配置
kind: Service
apiVersion: v1
metadata:
  name: $APP_NAME
  namespace: easybyte
  labels:
    app: $APP_NAME
spec:
  ports:
    - name: http
      protocol: TCP
      # 容器内部端口，服务端口
      targetPort: 8080
      # pod 端口，容器外部端口，可以重复（每个pod都有一个独立的ip）
      port: 8080
  selector:
    app: $APP_NAME
  type: NodePort
  sessionAffinity: None


# 部署配置
---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: $APP_NAME
  namespace: easybyte
  labels:
    app: $APP_NAME
spec:
  replicas: 1
  selector:
    matchLabels:
      app: $APP_NAME
  template:
    metadata:
      labels:
        app: $APP_NAME
    spec:
      containers:
        - name: $APP_NAME
          image: $APP_NAME:$BUILD_NUMBER
          ports:
            - containerPort: 80
              protocol: TCP
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          # 拉取镜像策略，当本地不存在时拉取。
          imagePullPolicy: IfNotPresent
      # 容器自动自动（Always：当容器失效时，由kubelet自动重启该容器。OnFailure：当容器终止运行且退出码不为0时，由kubelet自动重启该容器。Never：不论容器运行状态如何，kubelet都不会重启该容器。）
      restartPolicy: Always
      # 优雅停机毫秒
      terminationGracePeriodSeconds: 30
  # 发布策略（滚动更新）
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 100%
      maxUnavailable: 100%
```

## 常见问题
#### 给 SpringBoot 应用动态传参
在部署 SpringBoot 项目时，需要动态的制定启动参数例如：

```bash
java -jar /app.jar --server.port=80 --spring.cloud.nacos.config.namespace=$NACOS_GROUP
```

这时候可以在 pod 资源文件配置：

```yml
spec:
  template:
    spec:
      containers:
        - name: $APP_NAME
          image: $APP_NAME:$BUILD_NUMBER
          command:  ["java", "-jar", "/app.jar", "--server.port=80", "--spring.cloud.nacos.config.namespace=$NACOS_GROUP"]
```