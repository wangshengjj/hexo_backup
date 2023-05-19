---
title: 利用Docker搭建一个Cloudreve
date: 2022-10-17 20:14:26.112
updated: 2022-10-17 20:21:18.209
categories: 
- 服务器搭建
- 技术
- Docker
tags: 
- 私人网盘 nextcloud
- docker
- 服务器搭建
- linux搭建服务器
- cloudreve
---

# 利用Docker搭建一个Cloudreve

仅供学习交流使用，如果侵犯到你的合法权利，请联系邮件删除，或评论。我将会在24h内删除。

**如果您还未安装Docker请点击我查看[Docker的安装教程](https://www.wangshengjj.work/archives/13)**

## 首先简单介绍一下Cloudreve

Cloudreve 可以让您快速搭建起公私兼备的网盘系统。Cloudreve 在底层支持不同的云存储平台，用户在实际使用时无须关心物理存储方式。你可以使用 Cloudreve 搭建个人用网盘、文件分享系统，亦或是针对大小团体的公有云系统。

## 开始搭建

### 第一步

**创建必要的文件夹**

```
mkdir -vp cloudreve/{uploads,avatar} \
&& touch cloudreve/conf.ini \
&& touch cloudreve/cloudreve.db
```

### 第二步

**拉取并且运行镜像**

```
docker run -d \
-p 5212:5212 \
--mount type=bind,source=/root/cloudreve/conf.ini,target=/cloudreve/conf.ini \
--mount type=bind,source=/root/cloudreve/cloudreve.db,target=/cloudreve/cloudreve.db \
-v /root/cloudreve/uploads:/cloudreve/uploads \
-v /root/cloudreve/avatar:/cloudreve/avatar \
cloudreve/cloudreve:latest
```

**默认站在root家目录下，如有需要请自行更改命令目录！！！**

命令拆分：

- -d 后台运行
- -p 映射端口 
- 5212:5212 外部端口：内部端口
- -v 映射目录
- latest 最新版

待进度条走完浏览器访问，服务器IP+ 5212端口即可

### 特别鸣谢

[Cloudreve官方文档](https://docs.cloudreve.org/getting-started/install)
