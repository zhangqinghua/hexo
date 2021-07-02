---
title: Docker 常用操作

categories:
- 部署运维
- Docker 手册

date: 2021-04-22
---
#### 批量删除镜像
参考：[docker批量删除镜像](https://blog.csdn.net/benben_2015/article/details/90081468)

```bash
zhangqinghua$ docker rmi -f $(docker images | grep "none" | awk '{print $3}')
```

#### 修改容器启动配置参数
有时候，我们创建容器时忘了添加参数 `--restart=always`，当 Docker 重启时，容器未能自动启动。现在要添加该参数怎么办呢，方法有二：

**1. Docker 命令修改**

```bash
zhangqinghua$ docker container update --restart=always 容器名字
```

**2. 配置文件修改**

1. 首先停止容器，不然无法修改配置文件
1. 配置文件路径为：/var/lib/docker/containers/容器ID
1. 在该目录下找到一个文件 hostconfig.json ，找到该文件中关键字 RestartPolicy
1. 修改前配置："RestartPolicy":{"Name":"no","MaximumRetryCount":0}
1. 修改后配置："RestartPolicy":{"Name":"always","MaximumRetryCount":0}
1. 最后启动容器。

hostconfig.json：

```json
{
    "Binds": [
        "/data/elasticsearch/config/elaticsearch.yml:/usr/share/elasticsearch/config/elaticsearch.yml", 
        "/data/elasticsearch/data:/usr/share/elasticsearch/data", 
        "/data/elasticsearch/plugins:/usr/share/elasticsearch/plugins"
    ], 
    "ContainerIDFile": "", 
    "LogConfig": {
        "Type": "json-file", 
        "Config": { }
    }, 
    "NetworkMode": "default", 
    "PortBindings": {
        "9200/tcp": [
            {
                "HostIp": "", 
                "HostPort": "9200"
            }
        ], 
        "9300/tcp": [
            {
                "HostIp": "", 
                "HostPort": "9300"
            }
        ]
    }, 
    "RestartPolicy": {
        "Name": "no", 
        "MaximumRetryCount": 0
    }, 
    "AutoRemove": false, 
    "VolumeDriver": "", 
    "VolumesFrom": null, 
    "CapAdd": null, 
    "CapDrop": null, 
    "CgroupnsMode": "host", 
    "Dns": [ ], 
    "DnsOptions": [ ], 
    "DnsSearch": [ ], 
    "ExtraHosts": null, 
    "GroupAdd": null, 
    "IpcMode": "private", 
    "Cgroup": "", 
    "Links": null, 
    "OomScoreAdj": 0, 
    "PidMode": "", 
    "Privileged": false, 
    "PublishAllPorts": false, 
    "ReadonlyRootfs": false, 
    "SecurityOpt": null, 
    "UTSMode": "", 
    "UsernsMode": "", 
    "ShmSize": 67108864, 
    "Runtime": "runc", 
    "ConsoleSize": [
        0, 
        0
    ], 
    "Isolation": "", 
    "CpuShares": 0, 
    "Memory": 0, 
    "NanoCpus": 0, 
    "CgroupParent": "", 
    "BlkioWeight": 0, 
    "BlkioWeightDevice": [ ], 
    "BlkioDeviceReadBps": null, 
    "BlkioDeviceWriteBps": null, 
    "BlkioDeviceReadIOps": null, 
    "BlkioDeviceWriteIOps": null, 
    "CpuPeriod": 0, 
    "CpuQuota": 0, 
    "CpuRealtimePeriod": 0, 
    "CpuRealtimeRuntime": 0, 
    "CpusetCpus": "", 
    "CpusetMems": "", 
    "Devices": [ ], 
    "DeviceCgroupRules": null, 
    "DeviceRequests": null, 
    "KernelMemory": 0, 
    "KernelMemoryTCP": 0, 
    "MemoryReservation": 0, 
    "MemorySwap": 0, 
    "MemorySwappiness": null, 
    "OomKillDisable": false, 
    "PidsLimit": null, 
    "Ulimits": null, 
    "CpuCount": 0, 
    "CpuPercent": 0, 
    "IOMaximumIOps": 0, 
    "IOMaximumBandwidth": 0, 
    "MaskedPaths": [
        "/proc/asound", 
        "/proc/acpi", 
        "/proc/kcore", 
        "/proc/keys", 
        "/proc/latency_stats", 
        "/proc/timer_list", 
        "/proc/timer_stats", 
        "/proc/sched_debug", 
        "/proc/scsi", 
        "/sys/firmware"
    ], 
    "ReadonlyPaths": [
        "/proc/bus", 
        "/proc/fs", 
        "/proc/irq", 
        "/proc/sys", 
        "/proc/sysrq-trigger"
    ]
}
```