---
title: nginx
date: 2018-12-05 16:35:56
tags:

---

# 依赖

```
yum install gcc pcre-devel.x86_64 zlib-devel.x86_64
```



# 添加用户

```
groupadd -r nginx
useradd -s /sbin/nologin -g nginx -r nginx
```



# 安装命令

```
tar -zxvf nginx-xxxx.tar.gz

# 如果不安装到默认目录，可以配置 --prefix
./configure --prefix=/home/user1/nginx

# 如果要启用https，configure增加参数，并用yum安装 openssl-devel
./configure --prefix=/home/user1/nginx --with-http_ssl_module

# 如果要以其他用户启动nginx，则配置--user  --group
./configure --prefix=/home/user1/nginx --user=nginx --group=nginx


 ./configure --prefix=/opt/gitlab_external_nginx --user=nginx --group=nginx --with-http_ssl_module --with-http_gzip_static_module

make & make install
```



# 修改Banner

```
vi src/http/ngx_http_header_filter_module.c (lines 48 and 49)

static char ngx_http_server_string[] = "Server: MyDomain.com" CRLF;
static char ngx_http_server_full_string[] = "Server: MyDomain.com" CRLF;
```





# 日志切割

cat /etc/logrotate.d/nginx

```
/var/log/nginx/*.log {
        daily
        missingok
        rotate 52
        compress
        delaycompress
        notifempty
        create 640 nginx adm
        sharedscripts
        postrotate
                if [ -f /var/run/nginx.pid ]; then
                        kill -USR1 `cat /var/run/nginx.pid`
                fi
        endscript
}
```



# 自启动服务

ln -s /opt/gitlab_external_nginx/sbin/nginx /usr/sbin/nginx



cat /usr/lib/systemd/system/nginx.service

```
[Unit]
Description=nginx - high performance web server
Documentation=http://nginx.org/en/docs/
After=network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target

[Service]
Type=forking
PIDFile=/var/run/nginx.pid
ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx.conf
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s TERM $MAINPID

[Install]
WantedBy=multi-user.target
```



systemctl daemon-reload