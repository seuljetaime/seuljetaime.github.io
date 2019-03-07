---
title: k8s初步搭建
date: 2019-01-11 11:18:51
tags: k8s
---

本文只介绍初步搭建k8s单节点的步骤，没有深入。

本文以CentOS 7 虚拟机为环境进行编写。



<!-- more -->

# 介绍

请见[官方链接](https://kubernetes.io/zh/docs/concepts/overview/what-is-kubernetes/)，有中文描述。



Kubernetes 容器管理系统。

**功能：**

1. 服务监控
2. 负载均衡

## 概念

node：节点。master、worker

pod：相关的服务整合成一个pod



master: kube-apiserver、kube-controller-manager、kube-scheduler

worker:kubelet、kube-proxy



# 搭建

## kubectl

[官方链接](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

Use the Kubernetes command-line tool, [kubectl](https://kubernetes.io/docs/user-guide/kubectl/), to deploy and manage applications on Kubernetes. Using kubectl, you can inspect cluster resources; create, delete, and update components; look at your new cluster; and bring up example apps.

1. 下载安装，请自行解决此url无法访问的问题

    ```shell
    curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
    
    # 下载特定版本
    https://storage.googleapis.com/kubernetes-release/release/v1.13.2/bin/linux/amd64/kubectl
    ```

2. 修改权限，让kubectl可执行

   ```
   chmod +x kubectl
   ```


3. 将可执行程序移动到PATH目录下，方便运行

   ```
   mv kubectl /usr/local/bin/kubectl
   ```

4. 验证命令及版本

   ```
   kubectl version
   ```



## Minikube

1. 下载并授权可执行。也可以将&&拆分成两句

   ```shell
   curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \
     && chmod +x minikube
   ```

2. 移动到PATH目录，如果使用浏览器下载的，没有将minikube-linux-amd64重命名为minikube，请使用第二条命令

   ```shell
   mv minikube /usr/local/bin/minikube
   
   # 没有重命名则使用这条命令
   mv minikube-linux-amd64 /usr/local/bin/minikube
   ```

3. 验证命令及版本

   ```
   # 不要漏掉chmod +x minikube
   [root@localhost ~]# minikube version
   minikube version: v0.32.0
   ```

## Docker

kubernetes与Docker18.09有问题，此时请使用Docker 18.06。

推荐版本请见`https://kubernetes.io/docs/setup/cri/#docker`

```
On each of your machines, install Docker. Version 18.06 is recommended
```

如果能连接外网，可以自行配置网络yum安装特定版本，请参照`https://docs.docker.com/install/linux/docker-ce/centos/`自行安装。本文章介绍的是本地手动安装，**CentOS 7 ISO本地更新源请先自行配置，需额外安装包**。

1. 手动下载Docker`https://download.docker.com/linux/centos/7/x86_64/stable/Packages/`，下载`docker-ce-18.06.1.ce-3.el7.x86_64.rpm   `

2. 下载`container-selinux`, 本地更新源中没有，需要网络下载`http://mirror.centos.org/centos/7/extras/x86_64/Packages/`查找container-selinux，下载最新版

3. 安装Docker

```
yum install docker-ce-18.06.1.ce-3.el7.x86_64.rpm container-selinux-2.74-1.el7.noarch.rpm
```

### 启动Docker服务

```
# 启动
systemctl start docker

# 查看状态
systemctl status docker

# 设置开机启动
systemctl enable docker

# 关闭服务
systemctl stop docker
```



## 启动minikube及配置

需先启动好docker服务

1. 启动
    ```
    minikube start --vm-driver=none
    
    # 会去下载 kubeadm、kubelet
    # https://storage.googleapis.com/kubernetes-release/release/v1.12.4/bin/linux/amd64/kubeadm
    # https://storage.googleapis.com/kubernetes-release/release/v1.12.4/bin/linux/amd64/kubelet
    ```

2. 如果在上一步不能下载到这两个，请按这一步手动下载。否则可跳过这一步.

   + 使用能访问storage.googleapis.com的电脑下载以下2个文件，**如果版本不对应你现在的版本，请自行修改**

   ```
   https://storage.googleapis.com/kubernetes-release/release/v1.12.4/bin/linux/amd64/kubelet
   
   https://storage.googleapis.com/kubernetes-release/release/v1.12.4/bin/linux/amd64/kubeadm
   ```

   + 将2个文件放置到k8s环境下的 ~/.minikube/cache/v1.12.4/。如有需要，请自行修改版本
   + 重新执行minikube start --vm-driver=none

3. 如果遇到如下错误：

    ```
    [ERROR ImagePull]: failed to pull image k8s.gcr.io/kube-apiserver:v1.12.4: output: Error response from daemon: Get https://k8s.gcr.io/v2/: dial tcp: lookup k8s.gcr.io on [::1]:53: read udp [::1]:39248->[::1]:53: read: connection refused
    ```

    + 我是通过本机的docker，不是k8s环境。 然后设置docker-machine的代理，通过本机docker挂代理下载这些镜像。

    ```
    docker-machine ssh default
    sudo -i
    
    # 编辑/var/lib/boot2docker/profile，在最后加入。如果使用的是Windows的ShadowSocks，则在右键中启用“允许其他设备连入”
    export HTTP_PROXY=http://192.168.99.1:1080
    export HTTPS_PROXY=http://192.168.99.1:1080
    
    # 保存退出，并退出root及docker-machine，之后重新docker-machine
    docker-machine restart default
    ```

    + docker pull 那些image
    + `docker save k8s.gcr.io/coredns > coredns`  打包归档这个image。其他image类似
    + 上传打包好的各个image
    + `docker load < coredns`，加载image
    + `docker images`可以看到载入的image
    + 重新执行`minikube start --vm-driver=none`



## minikube监控

1. 创建dashboard服务

   `kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/master/aio/deploy/recommended/kubernetes-dashboard.yaml`， 访问不了这个文件的话可以先下载到文件中

2. 如果查看到

   ```
   [root@localhost ~]# kubectl get pods --namespace=kube-system
   
   kubernetes-dashboard-65c76f6c97-mxlq7   0/1     CrashLoopBackOff   84         22d
   ```

   可以查看日志

   ```
   kubectl logs kubernetes-dashboard-65c76f6c97-mxlq7 --namespace=kube-system
   
   Error while initializing connection to Kubernetes apiserver...
   Get https://10.96.0.1:443/version: dial tcp 10.96.0.1:443: connect: no route to host
   ```

   使用下列命令重启服务

   ```
   systemctl stop kubelet
   systemctl stop docker
   iptables --flush
   iptables -tnat --flush
   systemctl start kubelet
   systemctl start docker
   ```

3. 使用虚拟机环境中的浏览器访问dashboard

   ```
   kubectl proxy
   
   http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/
   ```

4. 创建dashboard的token

   ```
   kubectl create -f dashboard-adminuser.yaml
   
   kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
   ```

   

启动` minikube dashboard`



## 其他命令

```
# 查看kube系统的pod
kubectl get pods --namespace=kube-system

kubectl get deployments --namespace=kube-system
```



如果遇到dns

```
kubectl -n kube-system edit configmap coredns

删除里面的loop
```

