---
title: k8s初步搭建
date: 2019-01-11 11:18:51
tags: k8s
---

本文只介绍初步搭建k8s单节点的步骤，没有深入。

本文以CentOS 7 虚拟机为环境进行编写。

也可以使用[kubernetes.io的互动教程](https://kubernetes.io/zh/docs/tutorials/kubernetes-basics/cluster-interactive/)快速模拟搭建。



<!-- more -->

# 介绍

请见[官方链接](https://kubernetes.io/zh/docs/concepts/overview/what-is-kubernetes/)，有中文描述。



> Kubernetes 是一个跨主机集群的 [开源的容器调度平台，它可以自动化应用容器的部署、扩展和操作](http://www.slideshare.net/BrianGrant11/wso2con-us-2015-kubernetes-a-platform-for-automating-deployment-scaling-and-operations) , 提供以容器为中心的基础架构。
>
> 使用 Kubernetes, 您可以快速高效地响应客户需求:
>
> - 快速、可预测地部署您的应用程序
> - 拥有即时扩展应用程序的能力
> - 不影响现有业务的情况下，无缝地发布新功能
> - 优化硬件资源，降低成本
>
> 我们的目标是构建一个软件和工具的生态系统，以减轻您在公共云或私有云运行应用程序的负担。
>
> #### Kubernetes 具有如下特点:
>
> - **便携性**: 无论公有云、私有云、混合云还是多云架构都全面支持
> - **可扩展**: 它是模块化、可插拔、可挂载、可组合的，支持各种形式的扩展
> - **自修复**: 它可以自保持应用状态、可自重启、自复制、自缩放的，通过声明式语法提供了强大的自修复能力
>
> Kubernetes 项目由 Google 公司在 2014 年启动。Kubernetes 建立在 [Google 公司超过十余年的运维经验基础之上，Google 所有的应用都运行在容器上](https://research.google.com/pubs/pub43438.html), 再与社区中最好的想法和实践相结合，也许它是最受欢迎的容器平台。



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



启动minikube

```
minikube start
```

kubectl版本

```
kubectl version
```

查看集群

```
kubectl cluster-info
```

查看节点

```
kubectl get nodes
```

创建deployment,

> We want to run the app on a specific port so we add the `--port`

```
kubectl run kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1 --port=8080
```

获取deployment

```
kubectl get deployments
```

获取pods

```
kubectl get pods
```

获取pod的描述

```
kubectl describe pods
```

查看pod日志

```
kubectl logs $POD_NAME
```

查看pod ENV

```
kubectl exec $POD_NAME env
```

pod的交互模式

```
kubectl exec -ti $POD_NAME bash
```

创建新service，并指定NodePort

```
kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080
```

查看service描述

```
kubectl describe services/kubernetes-bootcamp
```

访问NodePort资源

```
curl http://$NODE_IP:$SERVICE_Node_Port
```

查看deployment

```
kubectl describe deployment
```

根据label获取pod、service

```
kubectl get pods -l run=kubernetes-bootcamp
kubectl get services -l run=kubernetes-bootcamp
```

给pod添加label

```
kubectl label pod $POD_NAME app=v1
```

根据label删除service

```
kubectl delete service -l run=kubernetes-bootcamp
```

修改冗余份数

```
kubectl scale deployments/kubernetes-bootcamp --replicas=4
```

滚动更新

```
kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=jocatalin/kubernetes-bootcamp:v2
```

滚动状态

```
kubectl rollout status deployments/kubernetes-bootcamp
```

回滚

```
kubectl rollout undo deployments/kubernetes-bootcamp
```



# kubeadm 

[kubeadm官方安装链接](https://kubernetes.io/docs/setup/independent/install-kubeadm/)

官方安装要求：

- One or more machines running one of:
  - Ubuntu 16.04+
  - Debian 9
  - CentOS 7
  - RHEL 7
  - Fedora 25/26 (best-effort)
  - HypriotOS v1.0.1+
  - Container Linux (tested with 1800.6.0)
- 2 GB or more of RAM per machine (any less will leave little room for your apps)
- 2 CPUs or more
- Full network connectivity between all machines in the cluster (public or private network is fine)
- Unique hostname, MAC address, and product_uuid for every node. See [here](https://kubernetes.io/docs/setup/independent/install-kubeadm/#verify-the-mac-address-and-product-uuid-are-unique-for-every-node) for more details.
- Certain ports are open on your machines. See [here](https://kubernetes.io/docs/setup/independent/install-kubeadm/#check-required-ports) for more details.
- Swap disabled. You **MUST** disable swap in order for the kubelet to work properly.



## master

1. 配置本地更新源

      + Virutalbox配置静态IP

        ```
        cd /etc/sysconfig/network-scripts/
        
        # 将下面的文件改成对应的
        vi ifcfg-enp0s8
        
        # 配置如下几项，网关改成对应的。如果你原本是dhcp获取的，可以用ip addr看到原来的配置
        BOOTPROTO=static
        IPADDR=192.168.99.105
        NETMASK=255.255.255.0
        GATEWAY=192.168.99.1
        ONBOOT=yes
        ```

        

2. 禁用防火墙

     ```
      systemctl disable firewalld.service
      systemctl stop firewalld.service
     ```

3. 设置主机名

     ```
     hostnamectl set-hostname master
     echo "127.0.0.1 master" >> /etc/hosts
     ```

4. 禁用swap

   ```bash
   # 查看swap
   swapon --show
   free -h
   
   vi /etc/fstab
   # 注释掉swap那行
   
   # 需要重启，或者可临时关闭
   swapoff -a
   ```

5. 禁用SELinux

      ```
      # 永久关闭SELinux
      vi /etc/selinux/config
      设置为 SELINUX=disabled
      
      # 需要重启，不重启临时禁用使用
      setenforce 0 
      ```

6. 安装docker并设置自启动

      ```bash
      yum install docker-ce-18.06.1.ce-3.el7.x86_64.rpm container-selinux-2.74-1.el7.noarch.rpm
      [root@master ~]# systemctl enable docker.service
      Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
      [root@master ~]# systemctl start docker.service
      [root@master ~]# systemctl status docker.service
      ```

### 安装kubeadm

[官方中文安装文档](https://kubernetes.io/zh/docs/setup/independent/install-kubeadm/)

> ```bash
> cat <<EOF > /etc/yum.repos.d/kubernetes.repo
> [kubernetes]
> name=Kubernetes
> baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
> enabled=1
> gpgcheck=1
> repo_gpgcheck=1
> gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
> exclude=kube*
> EOF
> 
> # 将 SELinux 设置为 permissive 模式(将其禁用)
> setenforce 0
> sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
> 
> yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
> 
> systemctl enable kubelet && systemctl start kubelet
> ```

1. 配置更新源或者在第二步时使用离线下载

   官方的`packages.cloud.google.com`更新源可能无法访问，可以使用阿里的`mirrors.aliyun.com/kubernetes`。

   [阿里镜像](https://opsx.alibaba.com/mirror) 在列表中找到kubernetes，列表的最右有Help说明怎么配置yum源

   ```bash
   cat <<EOF > /etc/yum.repos.d/kubernetes.repo
   [kubernetes]
   name=Kubernetes
   baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
   enabled=1
   gpgcheck=1
   repo_gpgcheck=1
   gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
   EOF
   ```

2. 安装kubeadm、kubelet、kubectl并启用服务

   ```bash
   yum install -y kubelet kubeadm kubectl
   systemctl enable kubelet && systemctl start kubelet
   ```

   如果是离线服务器可以从外部服务器仅下载，然后上传到离线服务器。--destdir自行指定

   ```
   yum install yum-utils
   yumdownloader --resolve kubelet kubeadm kubectl --destdir=/root/k8s_rpm
   rpm -Uvh /root/k8s_rpm/*.rpm
   systemctl enable kubelet && systemctl start kubelet
   ```

3. 文档中有说明cgroup，如果不是请自行按文档设置

   [在 Master 节点上配置 kubelet 所需的 cgroup 驱动](https://kubernetes.io/zh/docs/setup/independent/install-kubeadm/#%E5%9C%A8-master-%E8%8A%82%E7%82%B9%E4%B8%8A%E9%85%8D%E7%BD%AE-kubelet-%E6%89%80%E9%9C%80%E7%9A%84-cgroup-%E9%A9%B1%E5%8A%A8)

   ```bash
   [root@master ~]# docker info | grep Cgroup
   Cgroup Driver: cgroupfs
   ```

### 创建master

[官方中文文档](https://kubernetes.io/zh/docs/setup/independent/create-cluster-kubeadm/)

[官方英文文档](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/)  英文文档比中文文档新，里面有记录1.13.x版

本文使用单master，使用root安装

```bash
[root@master ~]# kubeadm init
I0314 03:50:30.587984   13231 version.go:94] could not fetch a Kubernetes version from the internet: unable to get URL "https://dl.k8s.io/release/stable-1.txt": Get https://dl.k8s.io/release/stable-1.txt: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
I0314 03:50:30.588396   13231 version.go:95] falling back to the local client version: v1.13.4
[init] Using Kubernetes version: v1.13.4
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
error execution phase preflight: [preflight] Some fatal errors occurred:
        [ERROR ImagePull]: failed to pull image k8s.gcr.io/kube-apiserver:v1.13.4: output: Error response from daemon: Get https://k8s.gcr.io/v2/: net/http: TLS handshake timeout
, error: exit status 1
```

https://dl.k8s.io/release/stable-1.txt  这个可能无法访问，请自行解决，如果版本和你local client的版本不一样，考虑是否升级。也可以使用`kubeadm init --kubernetes-version v1.13.4` 指定具体版本，跳过此网络请求。

后面的docker pull image是从https://k8s.gcr.io，有两种处理方式：

1. 改docker 挂代理，在上文中有说明

2. 修改成从阿里pull

   

本文使用从阿里获取，并提供导出docker image命令，用于离线服务器离线导入

由于下文还需设置网络，本文选用Flannel，需额外增加-pod-network-cidr=10.244.0.0/16。增加之前需按文档设置

```
sysctl net.bridge.bridge-nf-call-iptables=1
```

> For `flannel` to work correctly, you must pass 
>
> `--pod-network-cidr=10.244.0.0/16` to `kubeadm init`.
>
> Set `/proc/sys/net/bridge/bridge-nf-call-iptables` to `1` by running `sysctl net.bridge.bridge-nf-call-iptables=1` to pass bridged IPv4 traffic to iptables’ chains. This is a requirement for some CNI plugins to work, for more information please see [here](https://kubernetes.io/docs/concepts/cluster-administration/network-plugins/#network-plugin-requirements).



kubeadm不指定api-server的话，会用默认的网卡进行通信

```bash
[root@master ~]# ip route
default via 10.0.2.2 dev enp0s3 proto dhcp metric 100
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1
192.168.56.0/24 dev enp0s8 proto kernel scope link src 192.168.56.101 metric 101  
```

可以看到虚拟机中CentOS的默认default使用的是enp0s3，我们要用的是内网的enp0s8的地址`192.168.56.101`,kubeadm init时就需指定api-server的地址



**实际执行的命令**

```bash
kubeadm init --image-repository registry.aliyuncs.com/google_containers --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address 192.168.56.101
```

然后在等着下载，看网络速度，有几个100～200M的image，可以开另一个终端执行docker images看到有下载。

有遇到阿里的下载错误，请重新执行让再下载。





最后会提示

```bash
Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join 192.168.56.101:6443 --token qpazvy.ra54yk4zwjj5rvi0 --discovery-token-ca-cert-hash sha256:d9479075baa585015df4b58932fef2a525e99ba2978cfa1d85239f24846e79df
```



```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

查看node状态

```bash
[root@master ~]# kubectl get nodes
NAME     STATUS     ROLES    AGE   VERSION
master   NotReady   master   19m   v1.13.4
```

如果显示`NotReady`，可以查看日志

```
journalctl -fu kubelet

Mar 14 04:30:26 master kubelet[14979]: W0314 04:30:26.781580   14979 cni.go:203] Unable to update cni config: No networks found in /etc/cni/net.d
Mar 14 04:30:26 master kubelet[14979]: E0314 04:30:26.781870   14979 kubelet.go:2192] Container runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady message:docker: network plugin is not ready: cni config uninitialized

```

### 配置网络

上文最后的日志显示网络插件未准备好，

```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/a70459be0084506e4ec919aa1c114638878db11b/Documentation/kube-flannel.yml
```

kube-flannel.yml中的image是`quay.io/coreos/flannel:v0.11.0-amd64`，如果不能下载，请设置docker pull代理，或者从其他人克隆的库获取。

```bash
wget https://raw.githubusercontent.com/coreos/flannel/a70459be0084506e4ec919aa1c114638878db11b/Documentation/kube-flannel.yml

docker load < flannel.tar
kubectl apply -f kube-flannel.yml
```



## work node

按master的步骤安装到kubeadm，但不要执行kubeadm init

1. 在master节点查看token及sha256

   ```
   kubeadm token list
   
   openssl x509 -in /etc/kubernetes/pki/ca.crt -noout -pubkey | openssl rsa -pubin -outform DER 2>/dev/null | sha256sum | cut -d' ' -f1
   ```

2. 在node节点执行

   ```
   kubeadm join --token <token> <master-ip>:<master-port> --discovery-token-ca-cert-hash sha256:<hash>
   ```

3. 在master节点执行

   ```
   [root@master ~]# kubectl get nodes
   NAME     STATUS     ROLES    AGE     VERSION
   master   Ready      master   7h10m   v1.13.4
   node1    NotReady   <none>   43s     v1.13.4
   ```

   node节点有问题，在node节点使用下面命令查询日志信息

   ```
   journalctl -fu kubelet
   
   # ImagePullBackOff: Back-off pulling image "quay.io/coreos/flannel:v0.11.0-amd64"
   # 发现是flannel没有下载下来，使用手动load的形式加载docker image
   
   # 重启kubelet
   
   ```

   也可以使用下面命令查看pod的状态

   ```
   kubectl get pod --all-namespaces -o wide
   
   kubectl describe pod
   ```



## Ingress



## FAQ




kubectl 命令自动补全功能