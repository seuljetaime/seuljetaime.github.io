---
title: nginx
date: 2018-12-05 16:35:56
tags:

---

# 依赖

```
yum install gcc pcre-devel.x86_64 zlib-devel.x86_64
```

# 安装命令

```
tar -zxvf nginx-xxxx.tar.gz

# 如果不安装到默认目录，可以配置 --prefix
./configure --prefix=/home/user1/nginx

# 如果要启用https，configure增加参数
./configure --prefix=/home/user1/nginx --with-http_ssl_module

make & make install
```

