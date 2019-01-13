---
title: k8s初步搭建
date: 2019-01-11 11:18:51
tags: k8s
---

本文只介绍初步搭建k8s集群的步骤，没有深入。
搭建前请自行安装3个Linux虚拟机，本文以CentOS 7为例。



<!-- more -->

# 介绍

请见[官方链接](https://kubernetes.io/zh/docs/concepts/overview/what-is-kubernetes/)，有中文描述。



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

2. 移动到PATH目录，如果使用浏览器下载的，没有将minikube-linux-amd64重命名为minikube，请自行修改下面的命令

   ```shell
   mv minikube-linux-amd64 /usr/local/bin/minikube
   ```

3. 验证命令及版本

   ```
   # 不要漏掉chmod +x minikube
   [root@localhost ~]# minikube version
   minikube version: v0.32.0
   ```

## Docker

如果能连接外网，可以自行配置网络yum安装，请参照`https://docs.docker.com/install/linux/docker-ce/centos/`自行安装。本文章介绍的是本地手动安装，CentOS 7 ISO本地更新源请先自行配置，需额外安装包。

1. 手动下载Docker`https://download.docker.com/linux/centos/7/x86_64/stable/Packages/`，此时最新版是`docker-ce-18.09.1-3.el7.x86_64.rpm   `，需下载containerd.io、docker-ce、docker-ce-cli

2. 下载`container-selinux`, 本地更新源中没有，需要网络下载`http://mirror.centos.org/centos/7/extras/x86_64/Packages/`查找container-selinux，下载最新版

3. 安装Docker

```
yum install container-selinux-2.74-1.el7.noarch.rpm docker-ce-18.09.1-3.el7.x86_64.rpm containerd.io-1.2.2-3.el7.x86_64.rpm docker-ce-cli-18.09.1-3.el7.x86_64.rpm
```



## kubelet、kubeadm

**可选**。minikube启动时会去 `https://storage.googleapis.com` 下载kubelet、kubeadm。如果你的minikube环境不能访问此链接的话，需执行这一步。

1. 从可访问storage.googleapis.com机器下载kubelet、kubeadm
2. 下载后上传到minikube所在的环境
3. 配置minikube所在机器的hosts，将storage.googleapis.com指向自己本机
4. 在本机用python启动一个https服务器

## 启动及配置

1. 下载kubelet、kubeadm

   ```
   https://storage.googleapis.com/kubernetes-release/release/v1.12.4/bin/linux/amd64/kubelet
   https://storage.googleapis.com/kubernetes-release/release/v1.12.4/bin/linux/amd64/kubeadm
   ```


1. 启动
    ```
    minikube start --vm-driver=none
    ```





# 增强

## kubectl shell autocompletion

未验证bash-completion是否是额外yum更新源的

```
yum install bash-completion -y

# 手动刷新以启用功能
source <(kubectl completion bash)

# 自动加载
echo "source <(kubectl completion bash)" >> ~/.bashrc
```

