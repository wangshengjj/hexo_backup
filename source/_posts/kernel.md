---
title: Centos7升级内核版本教程
date: 2023-05-26 15:47:36.961
updated: 2023-05-26 16:00:05.231
categories: 
- 服务器搭建
- 我的水货
- 杂谈
- 技术
- 笔记
- centos
- linux教程
tags: 
- centos
- linux
- centos7
- kernel
- centos升级
---

# Centos7升级内核版本教程

>**`Centos7`默认的内核版本太低了，已经不适应现在的大多数软件了**
>**所以本期教程，带大家如何给`Centos7`升级内核版本**

- 教程用到的系统版本：`Centos7.9`

## 一、检查自己系统的发行版本

>**不会吧不会吧，既然你刷到这个教程，不会连自己系统的，发行版本是什么都不知道吧！**

```
[root@node5 ~]# cat /etc/redhat-release 
CentOS Linux release 7.9.2009 (Core)
```

### 查看内核版本

```
[root@node5 ~]# uname -r
3.10.0-1160.el7.x86_64
```

## 二、导入elrepo公钥

```
[root@node5 ~]# rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
```

## 三、安装elrepo源

```
[root@node5 ~]# yum install -y https://www.elrepo.org/elrepo-release-7.el7.elrepo.noarch.rpm
```

## 四、下载最新版内核

>**`kernel-ml`是最新的稳定版本**
>**`kernel-lt`是长期支持版本**
>**我个人可能更偏向于长期支持版本，因为他更稳定一些**

```
[root@node5 ~]# yum --enablerepo=elrepo-kernel install -y kernel-lt
```

>**执行完上面的命令，大概要等`30分钟-1小时`不等，请耐心等待，国外网站下载确实慢很多**

## 五、修改系统默认内核

### 1.查看系统所有可用内核

>**请务必记住不同版本内核的编号！**

```
[root@node5 ~]# awk -F \' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg
0 : CentOS Linux (6.3.4-1.el7.elrepo.x86_64) 7 (Core)
1 : CentOS Linux (5.4.243-1.el7.elrepo.x86_64) 7 (Core)
2 : CentOS Linux (3.10.0-1160.el7.x86_64) 7 (Core)
3 : CentOS Linux (0-rescue-69b30c004d1a456fb5a27f5306213572) 7 (Core)
```

### 2.修改文件/etc/default/grub(重要)

>**`kernel 6.x`版本需要下载`kernel-ml`版本**
>**在我的配置文件中是`数字1`使用的是`kernel 5.4.243`版本，如果想使用最新的`kernel 6.3.4`版本，就需要修改为`数字0`**

```
[root@node5 ~]# vim /etc/default/grub
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=1	#修改这里的参数，我这里的0对应上面刚刚查到的内核编号
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap rhgb quiet"
GRUB_DISABLE_RECOVERY="true"
```

### 3.重新生成grub文件(重要)

```
[root@node5 ~]# grub2-mkconfig -o /boot/grub2/grub.cfg
```

### 4.重启系统

```
[root@node5 ~]# reboot
```

## 六、查看升级后的内核版本

>**恭喜你，内核升级成功啦！**

```
[root@node5 ~]# uname -r
5.4.243-1.el7.elrepo.x86_64
```
