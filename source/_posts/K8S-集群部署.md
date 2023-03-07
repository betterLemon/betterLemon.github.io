---
title: K8S 集群部署
date: 2023-03-07 10:33:18
tags:
---

# K8S 集群部署 

###  使用kubeasz进行离线集群部署

#### 准备工作

- 直接下载安装工具

  ```
  export release=3.0.0
  curl -C- -fLO --retry 3 https://github.com/easzlab/kubeasz/releases/download/${release}/ezdown
  chmod +x ./ezdown
  ```

- 下载 k8s/docker 安装包 ( 默认下载最新版本 )

  ```
  ./ezdown -D
  ```

- 下载离线系统包

  ```
  ./ezdown -P
  ```

- 在接外网情况下可以直接使用上述命令，否则将在后续的安装包中给出下载好的文件。完成后目录如下

  - `/etc/kubeasz` 包含 kubeasz 版本为 ${release} 的发布代码
  - `/etc/kubeasz/bin` 包含 k8s/etcd/docker/cni 等二进制文件
  - `/etc/kubeasz/down` 包含集群安装时需要的离线容器镜像
  - `/etc/kubeasz/down/packages` 包含集群安装时需要的系统基础软件

#### 集群规划

- 测试环境规划3台服务器做集群

  |      IP       |  k8s   |  hostname  |
  | :-----------: | :----: | :--------: |
  | 10.70.132.55  | master | k8s-master |
  | 10.70.132.39  | worker | k8s-slave1 |
  | 10.70.132.189 | worker | k8s-slave2 |

- 默认配置下容器/kubelet会占用/var的磁盘空间，如果磁盘分区特殊，可以设置config.yml中的容器/kubelet数据目录：`CONTAINERD_STORAGE_DIR` `DOCKER_STORAGE_DIR` `KUBELET_ROOT_DIR`  正常情况下可保持不变

- 三台服务器配置互信免密

  

#### 安装k8s集群

###### 离线模式

- 启动 kubeasz 容器

  ```
  ./ezdown -S
  ```

- 设置允许离线安装

  ```
  sed -i 's/^INSTALL_SOURCE.*$/INSTALL_SOURCE: "offline"/g' /etc/kubeasz/example/config.yml 
  ```

- 若安装单机版

  ```
  docker exec -it kubeasz ezctl start-aio
  ```

- 多节点集群安装

  - 先执行上面的单节点安装 docker exec 命令，然后进入docker 容器内 **docker exec -it kubeasz bash**

  - kubeasz 容器会映射宿主机的目录，所以能容器环境下执行或使用 /etc/kubeasz 目录下的文件

    ```
    cd /etc/kubeasz
    ./ezctl new k8s-01  这里表示创建一个名称为 k8s-01 的集群，名字可自取，这一步会初始化一些基本的配置文件出来
    ```

  - 修改配置文件 /etc/kubeasz/clusters/k8s-01/hosts， 注意以下配置，对应集群规划进行修改

    ```
    [etcd]
    10.70.132.39
    10.70.132.55
    10.70.132.189
    
    # master node(s)
    [kube_master]
    10.70.132.55
    
    # work node(s)
    [kube_node]
    10.70.132.189
    10.70.132.39
    
    # [optional] harbor server, a private docker registry
    # 'NEW_INSTALL': 'true' to install a harbor server; 'false' to integrate with existed one
    [harbor]
    10.70.132.39 NEW_INSTALL=true
    
    
    # 集群服务器之间是否已经安装了NTP 服务器？如果没有的话，请安装 chrony 做节点间时间同步，修改成主节点IP即可若有则注释即可
    [chrony] 
    #192.168.1.1
    
    ```

  - 修改配置文件 /etc/kubeasz/clusters/k8s-01/config.yml

    - 离线情况下

      ```
      INSTALL_SOURCE: "offline"
      ```

      

    - **DNS 相关的配置项**，查看宿主机的配置文件 /etc/resolv.conf 中的 nameserver 是否为空（一般内部系统不存在 DNS 服务）；如果为空，需将 ENABLE_LOCAL_DNS_CACHE 设置为 false，否则 local_dns_cache 这个 Pod 会一直 Crash，详见[issue](https://link.zhihu.com/?target=https%3A//github.com/easzlab/kubeasz/issues/1010)中的说明。如果这个配置为 true 的话，集群会使用 [NodeLocal DNSCache](https://link.zhihu.com/?target=https%3A//kubernetes.io/zh/docs/tasks/administer-cluster/nodelocaldns/) 提升集群 DNS 性能，集群规模不大的话，可以不开启此功能

      ```
      dns_install: "yes"
      corednsVer: "__coredns__"
      ENABLE_LOCAL_DNS_CACHE: true
      dnsNodeCacheVer: "__dns_node_cache__"
      LOCAL_DNS_CACHE: "10.70.132.10"
      ```

    - Harbor配置 

      ```
      HARBOR_VER: "v2.1.3"
      HARBOR_DOMAIN: "harbor.qihoo.com"
      HARBOR_TLS_PORT: 8443
      ```

      

  - 执行安装 

    ```sh
    ./ezctl setup k8s-01 all  #即可一键完成所有步骤安装
    
    #也可以分步安装，对每一步进行验证
    # ezctl setup k8s-01 01
    # ezctl setup k8s-01 02
    # ezctl setup k8s-01 03
    # ezctl setup k8s-01 04
    ```

  - ezctl 命令行使用介绍可查看

    [帮助文档]: https://github.com/easzlab/kubeasz/blob/master/docs/setup/ezctl.md

  

###### 	在线模式

- 安装python2.7 ,已安装可忽略

  - centos系统

    ```shell
    # 更新yum，安装python ,已安装可忽略
    yum update
    yum install python -y
    ```

  - Ubuntu 

    ```shell
    apt-get update && apt-get upgrade -y && apt-get dist-upgrade -y
    # 安装python2 , 已安装可忽略
    apt-get install python2.7
    # Ubuntu16.04可能需要配置以下软连接
    ln -s /usr/bin/python2.7 /usr/bin/python
    ```

- 安装ansible

  ```shell
  # 注意pip 21.0以后不再支持python2和python3.5，需要如下安装
  # To install pip for Python 2.7 install it from https://bootstrap.pypa.io/2.7/ :
  curl -O https://bootstrap.pypa.io/pip/2.7/get-pip.py
  python get-pip.py
  python -m pip install --upgrade "pip < 21.0"
   
  # pip安装ansible(国内如果安装太慢可以直接用pip阿里云加速)
  pip install ansible -i https://mirrors.aliyun.com/pypi/simple/
  ```

- ssh免密配置，已经手动配置了互信免密机制可忽略,在主节点进行操作

  ```shell
  # 更安全 Ed25519 算法
  ssh-keygen -t ed25519 -N '' -f ~/.ssh/id_ed25519
  # 或者传统 RSA 算法
  ssh-keygen -t rsa -b 2048 -N '' -f ~/.ssh/id_rsa
  
  ssh-copy-id $IPs #$IPs为所有节点地址包括自身，按照提示输入yes 和root密码
  ```

  

- 完成上述步骤后，在主节点初始化集群基本配置

  ```
  ./ezctl new k8s-01 
  ```

  

- 修改配置文件，和离线模式内的修改方式一致，完成修改后，执行一键安装脚本

  ```
  ./ezctl setup k8s-01 all
  ```



#### 完成安装后进行验证，在节点上执行

```shell
# 查看节点信息
kubectl get node -owide

#查看默认pod信息，观察是否存在重启，失败等
kubectl get pod -n kube-system

```



### kuboard 大屏组件安装(可选)

Kuboard 是一款免费的 Kubernetes 管理工具，提供了丰富节点管理，部署等能力，可视化界面操作，简单便捷

安装步骤参考 

[在k8s中安装kuboard-v3]: https://kuboard.cn/install/v3/install-in-k8s.html#%E5%9C%A8-k8s-%E4%B8%AD%E5%AE%89%E8%A3%85-kuboard-v3

