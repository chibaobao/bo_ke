---
title: fedora下的qt安装
date: 2016-10-25 23:41:37
tags: qt
categories: 编程相关
---
1. 解决g++问题
  yum install gcc-c++（163的yum源可能无法解决依赖，可以配置阿里的yum源:http://mirrors.aliyun.com/）
2. 安装库
    yum install libGL 安装库
    yum install libGL-devel
3. 官方下载(下装好g++在装qt，qt会自动检测出g++)
    ./***.run