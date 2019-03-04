---
title: CentOS7安装OpenSSH7.9
tags:
---

# 简介说明
截止至2019-03-01 CentOS 7 最新的OpenSSH rpm包是`openssh-7.4p1-16.el7.x86_64.rpm `

由于[CVE-2018-15919](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-15919)漏洞，需要将OpenSSH升级到7.9，需手动编译安装升级。



# 注意及准备事项

1. 请先在自己的仿真环境进行升级，然后再在正式生产环境升级。网上别人安装的OpenSSL、zlib依赖版本可能与你的不一样
2. 额外开放另一种连接CentOS服务器方式，以防ssh出问题，比如telnet、vnc。并要注意好这些能开机自动开启并开放防火墙，如果是云管理平台，记得云管理平台的安全组端口也需开放。



<!-- more -->

# Telnet

+ telnet传输数据时没有加密。
+ root默认是无法登陆telnet的，如果不额外配置root可登录，可以使用其他用户登录telnet，然后su - root再转过去。
+ 默认端口是23



1. 安装telnet-server

   `yum install telnet-server`

   telnet 这个client看你是否需要，可不安装。CentOS7使用了systemd，不需要安装xinetd

2. 修改telnet默认端口，默认是23，修改成你需要的端口

   ```
   vi /usr/lib/systemd/system/telnet.socket
   
   [Socket]
   ListenStream=23
   ```

3. 开放防火墙，如果防火墙打开了的话。

   **如果是公网访问telnet，可以对防火墙开放特定IP访问telnet。略**

   ```
   # /usr/lib/firewalld/services/telnet.xml 中显示telnet service默认端口是23，没有修改的话，可使用此命令
   firewall-cmd --add-service=telnet --zone=public --permanent
   
   # 如果修改了默认23端口，使用此命令，2889修改成你的
   firewall-cmd --permanent --add-port=2889/tcp
   
   # 重载防火墙
   firewall-cmd --reload
   ```

4. SELinux

   此处关闭SELinux，后面的openssh在启用SELinux状态下登录会提示Access Denied

   ```
   # 永久关闭SELinux
   vi /etc/selinux/config
   将里面的值设置为 SELINUX=disabled
   
   # 临时关闭
   setenforce 0
   ```

   

   下文的供参考

   >  如果启用SELinux并且修改了23端口，需额外设置，不然会报`telnet.socket failed to listen on sockets: Permission denied`。2889是telnet新的端口
   >
   > ```
   > # 判断SELinux状态
   > [root@localhost ~]# getenforce
   > Enforcing
   > # 增强模式表示已启用
   > 
   > semanage port -a -t telnetd_port_t -p tcp 2889
   > 
   > # -bash: semanage: command not found 时需额外安装
   > yum install policycoreutils-python
   > ```
   >
   > 


5. 开机启动telnet服务

   ```
   systemctl enable telnet.socket
   systemctl start telnet.socket
   ```

6. 查看状态及端口是否监听

   ```
   systemctl status telnet.socket
   
   netstat -ntlp | grep 2889
   ss -ntlp | grep 2889
   ```

使用外部telnet client测试， 192.168.99.100是我测试虚拟机的IP

```
telnet 192.168.99.100 2889
# 会提示输入账号、密码，默认root是无法登陆，请使用其他账号，然后su - root跳转过去
```





# OpenSSH

**新安装的ssh端口是默认的22，如果对以前的ssh修改过默认端口，注意你的防火墙。**



1. 下载最新的OpenSSH7.9 `https://www.openssh.com/`，并上传到服务器

2. 备份旧的ssh配置

   ```
   cp -ar /etc/ssh /etc/ssh_bak
   ```

3. 安装编译前依赖，INSTALL中有版本要求，最新的yum rpm包满足要求

   ```
   yum install gcc zlib-devel openssl-devel pam-devel
   ```

4. 解压并安装OpenSSH

   ```
   tar -zxvf openssh-7.9p1.tar.gz
   cd openssh-7.9p1
   ./configure --prefix=/usr --sysconfdir=/etc/ssh --with-md5-passwords --with-zlib --with-pam
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

7. 修改配置文件的权限。或者删除`rm -rf /etc/ssh`

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
   
   # 默认的是on的，可以再次设置，列出以便确认
   chkconfig --list sshd
   chkconfig sshd on
   chkconfig --list sshd
   ```

10. 允许root登录ssh，将/etc/ssh/sshd_config 中PermitRootLogin的值修改为yes，或者在文件最后加一行

    ```
    vi /etc/ssh/sshd_config
    
    PermitRootLogin yes
    UsePAM yes
    ```

11. 重启ssh服务

    ```
    service sshd restart
    ```

12. 验证

    ```
    ssh -V
    ```



# 关闭Telnet

1. 清理防火墙

   ```
   # 如果配置的是默认的23端口并且用的是service
   firewall-cmd --remove-service=telnet --zone=public --permanent
   
   # 如果配置的自定义端口
   firewall-cmd --remove-port=2889/tcp --zone=public --permanent
   
   # 重载使防火墙生效
   firewall-cmd --reload
   ```

2. 卸载telnet-server

   ```
   yum remove telnet-server
   ```

   