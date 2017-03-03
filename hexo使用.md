---
title: hexo使用
date: 2016-10-09 20:13:31
tags: hexo
categories: 博客搭建
---

hexo的安装和常用命令
# hexo 的使用

## 安装

 查看hexo官网: [帮助文档](https://hexo.io/zh-cn/docs/index.html)
 
## 使用
- 创建
   $ hexo init <folder>
   $ cd <folder>
   $ npm install

- 本地服务器启动
   hexo server
   hexo server -p  5000   指定端口号为5000
- 新建文件 
	hexo new file
- 生成静态文件
	hexo generate	
- 同步到github
  -  hexo deploy --generate

## 错误：not found: git
   - 提交到github报错：ERROR Deployer not found: git
   - 解决：npm install hexo-deployer-git --save
## 配置文件

- 基本配置
title: 标题
subtitle: 介绍（在副标题位置）
description:
author: 作者
language: zh-Hans  语言：中文
timezone:

- 绑定github(提前配置好sshkey)
deploy:
  type: git
  repo: git@github.com:chibaobao/chibaobao.github.io.git
  
## 解决图片问题（本地与web端地址无法一致）

- 首先确认 _config.yml在中有 post_asset_folder:true
- hexo 目录，执行

```
npm install https://github.com/CodeFalling/hexo-asset-image --save 
```

- 在source/_posts目录下建立与博客文件同名文件夹
    - 如博客文章叫aaa.md则建立aaa文件夹 
    - 将图片添加到aaa文件夹   