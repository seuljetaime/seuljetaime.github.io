---
title: hexo设置
date: 2018-11-28 14:33:38
tags: hexo
---




# 图片

1. 安装插件`npm install hexo-asset-image --save`
2. 修改hexo根目录配置文件`_config.yml`中的`post_asset_folder`值为true

> 配置post_asset_folder为true后，每次hexo new 或hexo new draft都会在md同级目录创建一个同名的文件夹。 
>
> 1. 如果有些md不打算引入图片，可以采用手动创建.md文件方式。当需要引入图片时，可以再手动创建文件夹
> 2. 或者忽略这些文件夹不管，空文件夹是不会提交到git的

## Markdown 图片语法

`![替代文本](/一般都是markdown同名文件夹/1.jpg)`

## 官方引入图片的描述

[官方资源文件描述链接](https://hexo.io/zh-cn/docs/asset-folders.html)



# Hexo

## 首页文章预览

在markdown中需要切割预览的地方增加一个 `<!-- more -->`



## 启用tags

具体请见next主题的wiki [链接](https://github.com/iissnan/hexo-theme-next/wiki/%E5%88%9B%E5%BB%BA%E6%A0%87%E7%AD%BE%E4%BA%91%E9%A1%B5%E9%9D%A2)



## 常用命令

+ 启动时将草稿也渲染，不渲染草稿就将`--draft`去掉

  ```
  hexo server --draft
  ```

+ 新增草稿文章，新增正式时就将`draft`去掉

  ```
  hexo new draft 替换你的文章名
  ```

+ 草稿发布为正式文章，也可以手动移动文件到`_post`文件夹

  ```
  hexo publish 替换你的文章名
  ```

  