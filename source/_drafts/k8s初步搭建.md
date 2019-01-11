---
title: k8s初步搭建
date: 2019-01-11 11:18:51
tags: k8s
---

本文只介绍初步搭建k8s集群的步骤，没有深入。
搭建前请自行安装3个Linux虚拟机，本文以CentOS 7为例。



<!-- more -->

k8s是谷歌出品，请先科学上网

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

2. 移动到PATH目录

   ```shell
   mv minikube-linux-amd64 /usr/local/bin/minikube
   ```

3. 验证命令及版本

   ```
   [root@localhost ~]# minikube version
   minikube version: v0.32.0
   ```

   