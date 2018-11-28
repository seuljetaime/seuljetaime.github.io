---
title: npm设置
date: 2018-11-15 15:58:29
tags:
---

# NPM网络设置

1. 设置registry

    ```
    # 单次临时设置
    npm --registry=https://registry.npm.taobao.org install -g hexo-cli
    
    # 持久化设置，并显示设置的值
    npm config get registry
    npm config set registry https://registry.npm.taobao.org
    npm config get registry
    ```

2. cnpm

   ```
   npm install -g cnpm --registry=https://registry.npm.taobao.org
   cnpm install hexo-cli
   ```

3. 单次临时设置npm代理proxy

   ```
   npm install -g hexo-cli --proxy http://server:port --https-proxy http://server:port
   ```

4. 全局npm设置proxy

    ```
    # 查看设置
    npm config ls -l
    # 如果是Linux可以用grep过滤
    npm config ls -l | grep proxy
    # 查看特定配置，比如proxy
    npm config get proxy
    
    # 代理不需要账号密码，需要将server、port修改成对应的值
    npm config set proxy http://server:port
    npm config set https-proxy http://server:port
    
    # 代码需要账号密码
    npm config set proxy http://username:password@server:port
    npm config set https-proxy http://username:pawword@server:port
    
    # 取消设置代理，delete或rm都可
    npm config delete proxy
    npm config rm https-proxy
    
    # 全局设置，全局的在以上命令加上--global就可以，--global也可以简写成-g
    npm config --global set proxy http://server:port
    npm config --global set https-proxy http://server:port
    npm config -g rm proxy
    npm config -g rm https-proxy
    ```


