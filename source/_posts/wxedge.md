---
title: Centos7网心云搭建教程
date: 2023-01-07 10:57:09.841
updated: 2023-01-07 10:58:15.386
categories: 
- 服务器搭建
- 技术
- Docker
- 虚拟机
tags: 
- docker
- 服务器搭建
- 技术
- centos
- 虚拟机
- wxedge
- 网心云
- 容器魔方
---

# Centos7网心云搭建教程

**仅供学习交流使用，如果侵犯到你的合法权利，请联系邮件删除，或评论。我将会在24h内删除。**

关于容器魔方(网心云)：

「容器魔方」由网心云推出的一款docker容器镜像软件，通过简单安装后即可快速加入网心云共享计算生态网络，为网心科技星域云贡献带宽和存储资源，用户根据每日的贡献量可获得相应的现金收益回报。

**关于Docker的[安装教程](https://www.wangshengjj.work/archives/13)**

# 部署容器魔方

### 拉取镜像

```
docker run -d --name=wxedge --restart=always --privileged --net=host --tmpfs /run --tmpfs /tmp -v 你的磁盘路径:/storage:rw registry.hub.docker.com/onething1/wxedge
```

磁盘路径为你需要存放文件的位置(推荐大于50G)

### 停止容器

```
docker stop wxedge
```

### 删除容器

```
docker rm wxedge
```

### 删除镜像

```
docker rmi onething1/wxedge
```

上面的命令报错用下面这个

```
docker rmi registry.hub.docker.com/onething1/wxedge
```

### 特别鸣谢

[网心云官方文档](https://help.onethingcloud.com/caa9/a0fe/b6b3)

[容器魔方官方升级文档](https://help.onethingcloud.com/7cb4/6bc9)


