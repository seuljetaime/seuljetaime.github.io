---
title: CentOS7安装OpenSSH7.9
tags:
---

# 简介说明
截止至2019-03-01 CentOS 7 最新的OpenSSH rpm包是`openssh-7.4p1-16.el7.x86_64.rpm `

由于[CVE-2018-15919](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-15919)漏洞，需要将OpenSSH升级到7.9，需手动编译安装升级。



# 注意及准备事项

1. 请先在自己的仿真环境进行升级，然后再在正式生产环境升级。网上别人安装的OpenSSL、zlib依赖版本可能与你的不一样
2. 额外开放另一种连接CentOS服务器方式，以防ssh出问题，比如telnet、vnc。并要注意好这些能开机自动开启并开放防火墙



# Telnet

root默认是无法登陆telnet的，如果不额外配置root可登录，可以使用其他用户登录telnet，然后su - root再转过去。

1. 安装telnet-server

   `yum install telnet-server`

2. 开放防火墙，如果防火墙打开了的话

   ```
   firewall-cmd --add-service=telnet --zone=public --permanent
   firewall-cmd --reload
   ```

3. 开放SELinux，如果启用了的话。**略**

4. 开机启动telnet服务

   ```
   systemctl enable telnet.socket
   systemctl start telnet.socket
   ```




# OpenSSH

1. 下载最新的OpenSSH7.9 `https://www.openssh.com/`，并上传到服务器

2. 备份旧的ssh配置

   ```
   cp -ar /etc/ssh /etc/ssh_bak
   ```

3. 安装编译前依赖

   ```
   yum install gcc zlib-devel openssl-devel
   ```

4. 解压并安装OpenSSH

   ```
   tar -zxvf openssh-7.9p1.tar.gz
   ./configure --prefix=/usr --sysconfdir=/etc/ssh
   ```

5. make但不make install

   ```
   make
   ```

6. 卸载旧版OpenSSH rpm

   ```
   rpm -qa|grep openssh
   rpm -e --nodeps `rpm -qa | grep openssh`
   rpm -qa |grep openssh
   ```

7. 修改配置文件的权限

   ```
   chmod 600 /etc/ssh/ssh_host_rsa_key
   chmod 600 /etc/ssh/ssh_host_ecdsa_key
   chmod 600 /etc/ssh/ssh_host_ed25519_key
   ```

8. make install

   ```
   make install
   ```

9. 配置开机启动

   ```
   cp contrib/redhat/sshd.init /etc/init.d/sshd
   chkconfig --add sshd
   ```

10. 允许root登录ssh，将/etc/ssh/sshd_config 中PermitRootLogin的值修改为yes

    ```
    vi /etc/ssh/sshd_config
    
    PermitRootLogin yes
    ```

11. 重启ssh服务

    ```
    service sshd restart
    ```

    