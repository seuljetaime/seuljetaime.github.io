---
title: gitlab使用外部nginx
tags:
---





# 介绍

本文介绍Gitlab-CE 9.3.6 使用外部nginx的步骤，请先在测试环境验证此步骤。

# 参考链接

[Using a non-bundled web-server](https://docs.gitlab.com/omnibus/settings/nginx.html#using-a-non-bundled-web-server) 注意最顶上可以选文档的版本，但最早的也只能选择到`10.3`



# 操作步骤

## 源码安装Nginx

```
yum install gcc pcre-devel.x86_64 zlib-devel.x86_64

tar -zxvf nginx-xxxx.tar.gz
./configure --prefix=/home/user1/nginx

make & make install
```



## 配置步骤

1. 启动外置的nginx

2. 

3. 禁用内置的nginx，编辑`/etc/gitlab/gitlab.rb`

   ```
   nginx['enable'] = false
   ```

   

