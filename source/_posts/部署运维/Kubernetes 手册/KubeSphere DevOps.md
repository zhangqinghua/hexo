---
title: KubeSphere DevOps

categories:
- 部署运维
- Kubernetes 手册

date: 2021-05-13 07:00:06
---
## 限制 DevOps 运行在指定节点

参考：[为依赖项缓存设置 CI 节点](https://kubesphere.com.cn/docs/devops-user-guide/how-to-use/set-ci-node/)

> 经过测试不能指定在 master 节点上。

## Vue 应用
大体流程：
1. 将 Vue 项目编译成为 dist 文件；
1. 将 dist 打包成为 Docker Nginx 镜像；
1. 上传到阿里云容器镜像仓库；
1. 部署到集群中；
1. 配置网关路由公网访问；

#### jenkins 配置
```Jenkinsfile
pipeline {

  environment {
    // 项目名称
    APP_NAME = 'easybyte-front-sys'
    // 项目版本
    APP_VERSION = sh(script: "echo `date '+%Y%m%d%H%M%S'`", returnStdout: true).trim()
    // 项目空间
    APP_NAME_SPACE = 'easybyte-dev'
    
    // 代码仓库地址
    GIT_URL = 'https://code.aliyun.com/easybyte-web/easybyte-sys.git'
    // 代码版本
    GIT_BRANCH = 'master'
    // 代码凭证
    GIT_CREDENTIAL = 'aliyuncode-credential'

    // 镜像仓库地址
    DOCKER_REGISTRY = 'registry.cn-shenzhen.aliyuncs.com'
    // 镜像命名空间
    DOCKER_NAMESPACE = 'easybyte'
    // 镜像仓库凭证
    DOCKER_CREDENTIAL = 'aliyunrepository-credential'
    
    // 资源清单所在位置
    DEPLOY_CONFIGS = "Deploy.yml"
    // 集群的Id
    DEPLOY_KUBECONFIID = 'kubeconfig-credential'
  }

  agent {
    node {
      label 'nodejs'
    }
  }
  stages {
    stage('基本信息') {
      steps {
        sh 'echo -e "\n APP_NAME: $APP_NAME \n NAME_SPACE: $NAME_SPACE \n APP_VERSION: $APP_VERSION \n GIT_URL: $GIT_URL \n GIT_BRANCH: $GIT_BRANCH \n GIT_CREDENTIAL: $GIT_CREDENTIAL \n DOCKER_REGISTRY: $DOCKER_REGISTRY \n DOCKER_NAMESPACE: $DOCKER_NAMESPACE \n DOCKER_CREDENTIAL: $DOCKER_CREDENTIAL \n DEPLOY_CONFIGS: $DEPLOY_CONFIGS \n DEPLOY_KUBECONFIID: $DEPLOY_KUBECONFIID"'
      }
    }
    
    stage('拉取代码') {
      steps {
        container('nodejs') {
          git(url: "$GIT_URL", credentialsId: "$GIT_CREDENTIAL", branch: "$GIT_BRANCH", changelog: true, poll: false)
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

    stage('编译项目') {
      steps {
        container('nodejs') {
          sh 'yarn build'
          sh 'ls -a'
        }
      }
    }

    stage('构建镜像') {
      steps {
        container('nodejs') {
          sh 'docker build -f Dockerfile -t $DOCKER_REGISTRY/$DOCKER_NAMESPACE/$APP_NAME:$APP_VERSION .'
        }
      }
    }

    stage('上传镜像') {
      steps {
        container('nodejs') {
          withCredentials([usernamePassword(passwordVariable : 'DOCKER_PASSWORD' ,usernameVariable : 'DOCKER_USERNAME' ,credentialsId : "$DOCKER_CREDENTIAL" ,)]) {
            sh 'echo "$DOCKER_PASSWORD" | docker login $DOCKER_REGISTRY -u "$DOCKER_USERNAME" --password-stdin'
            sh 'docker push $DOCKER_REGISTRY/$DOCKER_NAMESPACE/$APP_NAME:$APP_VERSION'
          }
        }
      }
    }

    stage('部署应用') {
      steps {
        // enableConfigSubstitution 是否开启变量替换（即替换Deploy.yml里面的配置）
        // kubeconfigId             集群的Id
        // configs                  资源文件所在位置
        kubernetesDeploy(enableConfigSubstitution: true, deleteResource: false, kubeconfigId: "$DEPLOY_KUBECONFIID", configs: "$DEPLOY_CONFIGS")
      }
    }
  }
}
```

#### Dockerfile
```Dockerfile
# Alpine Linux是：一个基于musl libc和busybox、面向安全的轻量级Linux发行版。
# ngixn 打包大小为 133 MB，nginx:stable-alpine 只有 20 MB。
FROM nginx:stable-alpine

# 将dist文件中的内容复制到 /usr/share/nginx/html/merchant 这个目录下面
# 前端访问时使用 http://dev.easybyte-hk.com/merchant/ > http://localhost:80/merchant
COPY dist/  /usr/share/nginx/html/merchant
```

#### Kube Pod 资源清单
```yml
# 服务配置
kind: Service
apiVersion: v1
metadata:
  name: $APP_NAME
  namespace: $NAME_SPACE
  labels:
    app: $APP_NAME
spec:
  ports:
    - name: http
      protocol: TCP
      # 容器内部端口，服务端口
      targetPort: 80
      # pod 端口，容器外部端口，可以重复（每个pod都有一个独立的ip）
      port: 80
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
  namespace: $NAME_SPACE
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
          image: $DOCKER_REGISTRY/$DOCKER_NAMESPACE/$APP_NAME:$VERSION
          ports:
            - containerPort: 80
              protocol: TCP
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          # 拉取镜像策略，当本地不存在时拉取。
          imagePullPolicy: Always
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

## SpringCloud 应用

#### jenkins 配置
单模块部署：

```Jenkinsfile
pipeline {

  parameters {
    choice(name: 'APP_NAME',
           choices: ['easybyte-log',
                     'easybyte-pay',
                     'easybyte-auth',
                     'easybyte-base',
                     'easybyte-flow',
                     'easybyte-cart',
                     'easybyte-store',
                     'easybyte-order',
                     'easybyte-media',
                     'easybyte-stats',
                     'easybyte-member',
                     'easybyte-market',
                     'easybyte-swagger',
                     'easybyte-product',
                     'easybyte-gateway',
                     'easybyte-merchant',
                     'easybyte-consumer',
                     'easybyte-template',
                     'easybyte-scheduled'],
                     description: '选择部署的模块')
  }

  environment {
    // 项目版本
    APP_VERSION = sh(script: "echo `date '+%Y%m%d%H%M%S'`", returnStdout: true).trim()
    // 项目空间
    APP_NAME_SPACE = 'easybyte-dev'
    // Nacos 配置分组
    NACOS_GROUP = '153cda3e-322c-4ff1-8aa0-d1240784dbd2'

    // 代码仓库地址
    GIT_URL = 'https://code.aliyun.com/easybyte-java/easybyte.git'
    // 代码版本
    GIT_BRANCH = 'master'
    // 代码凭证
    GIT_CREDENTIAL = 'aliyuncode-credential'

    // 镜像仓库地址
    DOCKER_REGISTRY = 'registry.cn-shenzhen.aliyuncs.com'
    // 镜像命名空间
    DOCKER_NAMESPACE = 'easybyte'
    // 镜像仓库凭证
    DOCKER_CREDENTIAL = 'aliyunrepository-credential'

    // 资源清单所在位置
    DEPLOY_CONFIGS = "Deploy.yml"
    // 集群的Id
    DEPLOY_KUBECONFIID = 'kubeconfig-credential'
  }

  agent {
    node {
      label 'maven'
    }
  }
  stages {
    stage('基本信息') {
      steps {
        sh 'echo -e "\n APP_NAME: $APP_NAME \n APP_NAME_SPACE: $APP_NAME_SPACE \n APP_VERSION: $APP_VERSION \n NACOS_GROUP: $NACOS_GROUP \n GIT_URL: $GIT_URL \n GIT_BRANCH: $GIT_BRANCH \n GIT_CREDENTIAL: $GIT_CREDENTIAL \n DOCKER_REGISTRY: $DOCKER_REGISTRY \n DOCKER_NAMESPACE: $DOCKER_NAMESPACE \n DOCKER_CREDENTIAL: $DOCKER_CREDENTIAL \n DEPLOY_CONFIGS: $DEPLOY_CONFIGS \n DEPLOY_KUBECONFIID: $DEPLOY_KUBECONFIID"'
      }
    }

    stage('拉取代码') {
      steps {
        container('maven') {
          git(url: "$GIT_URL", credentialsId: "$GIT_CREDENTIAL", branch: "$GIT_BRANCH", changelog: true, poll: false)
        }
      }
    }

    stage('编译项目') {
      steps {
        container('maven') {
          sh 'mvn clean package -DskipTests -pl $APP_NAME -am'
        }
      }
    }

    stage('构建镜像') {
      steps {
        container('maven') {
          sh "docker build -f Dockerfile -t $DOCKER_REGISTRY/$DOCKER_NAMESPACE/$APP_NAME:$APP_VERSION --build-arg APP_NAME=$APP_NAME ."
        }
      }
    }

    stage('上传镜像') {
      steps {
        container('maven') {
          withCredentials([usernamePassword(passwordVariable : 'DOCKER_PASSWORD' ,usernameVariable : 'DOCKER_USERNAME' ,credentialsId : "$DOCKER_CREDENTIAL" ,)]) {
            sh 'echo "$DOCKER_PASSWORD" | docker login $DOCKER_REGISTRY -u "$DOCKER_USERNAME" --password-stdin'
            sh 'docker push $DOCKER_REGISTRY/$DOCKER_NAMESPACE/$APP_NAME:$APP_VERSION'
          }
        }
      }
    }

    stage('部署应用') {
      steps {
        // enableConfigSubstitution 是否开启变量替换（即替换Deploy.yml里面的配置）
        // kubeconfigId             集群的Id
        // configs                  资源文件所在位置
        kubernetesDeploy(enableConfigSubstitution: true, deleteResource: false, kubeconfigId: "$DEPLOY_KUBECONFIID", configs: "$DEPLOY_CONFIGS")
      }
    }
  }
}
```

全部模块一起部署：

```Jenkinsfile
def MODULES = ['easybyte-log',
               'easybyte-pay',
               'easybyte-auth',
               'easybyte-base',
               'easybyte-flow',
               'easybyte-cart',
               'easybyte-store',
               'easybyte-order',
               'easybyte-media',
               'easybyte-stats',
               'easybyte-member',
               'easybyte-market',
               'easybyte-swagger',
               'easybyte-product',
               'easybyte-gateway',
               'easybyte-merchant',
               'easybyte-consumer',
               'easybyte-template',
               'easybyte-scheduled'];

pipeline {
  environment {
    // 项目版本
    APP_VERSION = sh(script: "echo `date '+%Y%m%d%H%M%S'`", returnStdout: true).trim()
    // 项目空间
    APP_NAME_SPACE = 'easybyte-dev'
    // Nacos 配置分组
    NACOS_GROUP = '153cda3e-322c-4ff1-8aa0-d1240784dbd2'

    // 代码仓库地址
    GIT_URL = 'https://code.aliyun.com/easybyte-java/easybyte.git'
    // 代码版本
    GIT_BRANCH = 'master'
    // 代码凭证
    GIT_CREDENTIAL = 'aliyuncode-credential'

    // 镜像仓库地址
    DOCKER_REGISTRY = 'registry.cn-shenzhen.aliyuncs.com'
    // 镜像命名空间
    DOCKER_NAMESPACE = 'easybyte'
    // 镜像仓库凭证
    DOCKER_CREDENTIAL = 'aliyunrepository-credential'

    // 资源清单所在位置
    DEPLOY_CONFIGS = "Deploy.yml"
    // 集群的Id
    DEPLOY_KUBECONFIID = 'kubeconfig-credential'
  }

  agent {
    node {
      label 'maven'
    }
  }
  stages {
    stage('基本信息') {
      steps {
        sh 'echo -e "\n APP_NAME: $APP_NAME \n APP_NAME_SPACE: $APP_NAME_SPACE \n APP_VERSION: $APP_VERSION \n NACOS_GROUP: $NACOS_GROUP \n GIT_URL: $GIT_URL \n GIT_BRANCH: $GIT_BRANCH \n GIT_CREDENTIAL: $GIT_CREDENTIAL \n DOCKER_REGISTRY: $DOCKER_REGISTRY \n DOCKER_NAMESPACE: $DOCKER_NAMESPACE \n DOCKER_CREDENTIAL: $DOCKER_CREDENTIAL \n DEPLOY_CONFIGS: $DEPLOY_CONFIGS \n DEPLOY_KUBECONFIID: $DEPLOY_KUBECONFIID"'
      }
    }

    stage('拉取代码') {
      steps {
        container('maven') {
          git(url: "$GIT_URL", credentialsId: "$GIT_CREDENTIAL", branch: "$GIT_BRANCH", changelog: true, poll: false)
        }
      }
    }

    stage('编译项目') {
      steps {
        container('maven') {
          sh 'mvn clean package -DskipTests'
        }
      }
    }

    stage('构建镜像') {
      steps {
        container('maven') {
          script {
            for (String MODULE : MODULES) {
              sh "docker build -f Dockerfile -t $DOCKER_REGISTRY/$DOCKER_NAMESPACE/$MODULE:$APP_VERSION --build-arg APP_NAME=$MODULE ."
            }
          }
        }
      }
    }

    stage('上传镜像') {
      steps {
        container('maven') {
          withCredentials([usernamePassword(passwordVariable : 'DOCKER_PASSWORD' ,usernameVariable : 'DOCKER_USERNAME' ,credentialsId : "$DOCKER_CREDENTIAL" ,)]) {
            sh 'echo "$DOCKER_PASSWORD" | docker login $DOCKER_REGISTRY -u "$DOCKER_USERNAME" --password-stdin'
              script {
                for (String MODULE : MODULES) {
                  sh "docker push $DOCKER_REGISTRY/$DOCKER_NAMESPACE/$MODULE:$APP_VERSION"
                }
              }
          }
        }
      }
    }

    stage('部署应用') {
      steps {
        container('maven') {
          script {
            for (String MODULE : MODULES) {
              // kubernetesDeploy 好像只能识别全局变量 env
              env.APP_NAME = MODULE
              // enableConfigSubstitution 是否开启变量替换（即替换Deploy.yml里面的配置）
              // kubeconfigId             集群的Id
              // configs                  资源文件所在位置
              kubernetesDeploy(enableConfigSubstitution: true, deleteResource: false, kubeconfigId: "$DEPLOY_KUBECONFIID", configs: "$DEPLOY_CONFIGS")
              sleep 20
            }
          }
        }
      }
    }
  }
}
```

#### Dockerfile
```Dockerfile
# 1. 加载Easybyte通用库和运行环境（lib有70MB Java应用环境100MB）
FROM        zhangqinghua/easybyte-lib:0.0.6

# 2.接收从外边传来的参数 easybyte-auth
ARG         APP_NAME

# 3. 把应用包复制进容器（排除了lib的应用体积大概在1MB左右）复制本地的renren-fast.jar文件到容器/目录并改名app.jar easybyte-auth/target/easybyte-auth.jar -> /app.jar
ADD         ${APP_NAME}/target/${APP_NAME}.jar /app.jar

# 4. 对外开放的端口，8070给Txlcn用
EXPOSE      80

# 5. 启动应用，指定外部通用库目录 在应用的启动参数，设置 -XX:+UseContainerSupport，设置-XX:MaxRAMPercentage=75.0，这样为其他进程（debug、监控）留下足够的内存空间，又不会太浪费RAM。
ENTRYPOINT  ["java", "-Dloader.path=/lib", "-jar", "/app.jar", "--server.port=80", "--spring.cloud.nacos.config.namespace=???"]
```

#### Kube Pod 资源清单
```yml
# 服务配置
kind: Service
apiVersion: v1
metadata:
  # 应用名称：easybyte-auth
  name: $APP_NAME
  # 命名空间（这里区分不同的环境）：easybyte-dev、easybyte-test、easybyte-prod
  namespace: $NAME_SPACE
  labels:
    app: $APP_NAME
spec:
  ports:
    - protocol: TCP
      # 容器内部端口，服务端口
      targetPort: 80
      # pod 端口，容器外部端口，可以重复（每个pod都有一个独立的ip）
      port: 80
  selector:
    app: $APP_NAME
  sessionAffinity: None

# 部署配置
---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: $APP_NAME
  namespace: $NAME_SPACE
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
          image: $DOCKER_REGISTRY/$DOCKER_NAMESPACE/$APP_NAME:$VERSION
          command: [ "java",
                     "-Dloader.path=/lib",
                     "-XX:+UseContainerSupport",
                     "-XX:MaxRAMPercentage=75.0",
                     "-jar",
                     "/app.jar",
                     "--server.port=80",
                     "--spring.cloud.nacos.config.namespace=$NACOS_GROUP"]

          # args: ["--spring.cloud.nacos.config.namespace=$NACOS_GROUP"]
          ports:
            - containerPort: 80
              protocol: TCP
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          # 拉取镜像策略，当本地不存在时拉取。
          imagePullPolicy: IfNotPresent
          # 设置时区，Docker默认的是伦敦时区
          env:
            - name: TZ
              value: Asia/Shanghai
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

#### 自定义运行 pod 资源文件
```bash
stage('deploy app to multi cluster') {
    steps {
        container('nodejs') {
            script {
                withCredentials([kubeconfigFile(credentialsId: 'multi-cluster', variable: 'KUBECONFIG')]) {
                    sh 'envsubst < devops-go-sample/manifest/multi-cluster-deploy.yaml | kubectl apply -f -'
                }
            }
        }
    }
}
```

#### Jenkinsfile 使用 choice 参数时，修改没有生效
场景：Jenkinsfile 使用了 choice 参数，总是不生效，且 `kubesphere-system.ks-controller-manager` 报错。

```bash
zhangqinghua$ kubectl get pods -n kubesphere-system
kubesphere-system              cluster-agent-76d4f5466-j9shm                                     1/1     Running            0          15d
kubesphere-system              ks-apiserver-689d5cccbf-psnz4                                     1/1     Running            0          6h12m
kubesphere-system              ks-console-6858bfd78c-zkf4j                                       1/1     Running            0          31d
kubesphere-system              ks-controller-manager-7fcb88968-gtdk2                             0/1     CrashLoopBackOff   3          3m14s
kubesphere-system              ks-installer-566dc4c9bd-t7slb                                     1/1     Running            0          7h25m
kubesphere-system              minio-5f49b564fd-tnq7k                                            1/1     Running            2          30d
kubesphere-system              openldap-0                                                        1/1     Running            2          31d
kubesphere-system              tower-8647b6f6d4-2crzd                                            1/1     Running            8          25d
```
原因：Kubesphere bug。
解决：？

参考：[相关Issus](https://github.com/kubesphere/kubesphere/issues?q=is%3Aissue+panic+is%3Aclosed+author%3ALinuxSuRen)

后续：经过多次测试，这种写法是 OK 的（编辑后运行 2 次才生效）

```
parameters {
  choice(name: 'APP_NAME', choices: ['a' , 'b' , 'c'], description: '123')
}
```

#### 流水线更新应用，deployment安装失败
场景：Internal error occurred: failed calling webhook “logsidecar-injector.logging.kubesphere.io”: Post https://logsidecar-injector-admission.kubesphere-logging-system.svc:443/?timeout=30s: service “logsidecar-injector-admission” not found

原因：????

解决：kubectl delete MutatingWebhookConfiguration logsidecar-injector-admission-mutate 删除掉这个3.0.0版本的对象

参考：https://kubesphere.com.cn/forum/d/1978-deployment