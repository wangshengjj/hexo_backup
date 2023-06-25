---
title: 【容器应用系列教程】基于Kubernetes1.20.7部署高可用集群(Centos7)
date: 2023-06-10 16:16:48.303
updated: 2023-06-18 16:12:54.756
sticky: 3
categories: 
- k8s
- 笔记
- 集群
- keepalived
- haproxy
- 容器
tags: 
- k8s
- kubernetes
- linux
- 高可用集群
- centos7
---

# 【容器应用系列教程】基于Kubernetes1.20.7部署高可用集群(Centos7)

>**关于单`Master`节点`Kubernetes`集群教程：https://www.wsjj.top/archives/138**

## 一、环境描述

### 1.主机规划

|主机名|IP地址|VIP|安装的软件|系统版本|配置|
|-------|-------|-------|-------|-------|-------|
|k8s-master01.linux.com|192.168.140.10|192.168.140.100|Docker、Kubernetes、LVS、HAproxy、Keepalived|Centos7.9|最低2c2g|
|k8s-master02.linux.com|192.168.140.11|192.168.140.100|Docker、Kubernetes、LVS、HAproxy、Keepalived|Centos7.9|最低2c2g|
|k8s-master03.linux.com|192.168.140.12|192.168.140.100|Docker、Kubernetes、LVS、HAproxy、Keepalived|Centos7.9|最低2c2g|
|k8s-node01.linux.com|192.168.140.13|空|Docker、Kubernetes、LVS|Centos7.9|最低2c2g|
|k8s-node02.linux.com|192.168.140.14|空|Docker、Kubernetes、LVS|Centos7.9|最低2c2g|

### 2.软件规划

- `kubernetes-1.20.7`版本
- `docker-19.03`版本

### 3.网段规划

- `Pod`网段: `172.168.0.0/16`
- `Service`网段: `10.96.0.0/16`

## 二、前期准备(重要)

### 1.五台服务器关闭防火墙和SElinux、配置时间同步

>**过程省略**

#### 配置时间同步计划任务

```
[root@k8s-master01 ~]# yum install -y ntpdate
[root@k8s-master02 ~]# yum install -y ntpdate
[root@k8s-master03 ~]# yum install -y ntpdate
[root@k8s-node01 ~]# yum install -y ntpdate
[root@k8s-node02 ~]# yum install -y ntpdate
```

```
[root@k8s-master01 ~]# crontab -e
*/30 * * * *   /usr/sbin/ntpdate  120.25.115.20 &> /dev/null
[root@k8s-master02 ~]# crontab -e
*/30 * * * *   /usr/sbin/ntpdate  120.25.115.20 &> /dev/null
[root@k8s-master03 ~]# crontab -e
*/30 * * * *   /usr/sbin/ntpdate  120.25.115.20 &> /dev/null
[root@k8s-node01 ~]# crontab -e
*/30 * * * *   /usr/sbin/ntpdate  120.25.115.20 &> /dev/null
[root@k8s-node02 ~]# crontab -e
*/30 * * * *   /usr/sbin/ntpdate  120.25.115.20 &> /dev/null
```

### 2.所有主机关闭`SSH`的`DNS`解析(优化SSH连接速度)

```
[root@k8s-master01 ~]# sed -ri 's|#UseDNS yes|UseDNS no|' /etc/ssh/sshd_config
[root@k8s-master01 ~]# systemctl restart sshd
[root@k8s-master02 ~]# sed -ri 's|#UseDNS yes|UseDNS no|' /etc/ssh/sshd_config
[root@k8s-master02 ~]# systemctl restart sshd
[root@k8s-master03 ~]# sed -ri 's|#UseDNS yes|UseDNS no|' /etc/ssh/sshd_config
[root@k8s-master03 ~]# systemctl restart sshd
[root@k8s-node01 ~]# sed -ri 's|#UseDNS yes|UseDNS no|' /etc/ssh/sshd_config
[root@k8s-node01 ~]# systemctl restart sshd
[root@k8s-node02 ~]# sed -ri 's|#UseDNS yes|UseDNS no|' /etc/ssh/sshd_config
[root@k8s-node02 ~]# systemctl restart sshd
```

### 3.所有主机配置免密`SSH`连接(重要)

```
[root@k8s-master01 ~]# ssh-keygen -t rsa
[root@k8s-master01 ~]# mv /root/.ssh/id_rsa.pub /root/.ssh/authorized_keys

[root@k8s-master01 ~]# for i in 11 12 13 14
> do
> scp -r /root/.ssh/ root@192.168.140.$i:/root/
> done
```

#### 测试免密`SSH`

```
[root@k8s-master01 ~]# for i in 10 11 12 13 14
> do
> ssh root@192.168.140.$i hostname; date
> done
k8s-master01.linux.com
2023年 06月 10日 星期六 16:49:40 CST
k8s-master02.linux.com
2023年 06月 10日 星期六 16:49:40 CST
k8s-master03.linux.com
2023年 06月 10日 星期六 16:49:40 CST
k8s-node01.linux.com
2023年 06月 10日 星期六 16:49:40 CST
k8s-node02.linux.com
2023年 06月 10日 星期六 16:49:40 CST
```

### 4.所有主机添加主机名解析(重要)

```
[root@k8s-master01 ~]# vim /etc/hosts
#在配置文件底部添加以下内容
192.168.140.10 k8s-master01.linux.com k8s-master01
192.168.140.11 k8s-master02.linux.com k8s-master02
192.168.140.12 k8s-master03.linux.com k8s-master03
192.168.140.13 k8s-node01.linux.com k8s-node01
192.168.140.14 k8s-node02.linux.com k8s-node02
192.168.140.100 k8s-master-vip
```

```
[root@k8s-master01 ~]# for i in 11 12 13 14
> do
> scp -r /etc/hosts root@192.168.140.$i:/etc/
> done
```

### 5.所有主机禁用`SWAP`分区

```
[root@k8s-master01 ~]# for i in 10 11 12 13 14
> do
> ssh root@192.168.140.$i swapoff -a; free -m; sysctl -w vm.swappiness=0
> done

              total        used        free      shared  buff/cache   available
Mem:           2755         192        2165           9         397        2409
Swap:             0           0           0
vm.swappiness = 0
              total        used        free      shared  buff/cache   available
Mem:           2755         192        2165           9         396        2409
Swap:             0           0           0
vm.swappiness = 0
              total        used        free      shared  buff/cache   available
Mem:           2755         192        2165           9         396        2409
Swap:             0           0           0
vm.swappiness = 0
              total        used        free      shared  buff/cache   available
Mem:           2755         192        2165           9         396        2410
Swap:             0           0           0
vm.swappiness = 0
              total        used        free      shared  buff/cache   available
Mem:           2755         192        2166           9         396        2410
Swap:             0           0           0
vm.swappiness = 0
```

#### 注释掉`SWAP`分区的挂载信息

```
[root@k8s-master01 ~]# vim /etc/fstab
# /dev/mapper/centos-swap swap                    swap    defaults        0 0
[root@k8s-master01 ~]# mount -a

[root@k8s-master02 ~]# vim /etc/fstab
# /dev/mapper/centos-swap swap                    swap    defaults        0 0
[root@k8s-master02 ~]# mount -a

[root@k8s-master03 ~]# vim /etc/fstab
# /dev/mapper/centos-swap swap                    swap    defaults        0 0
[root@k8s-master03 ~]# mount -a

[root@k8s-node01 ~]# vim /etc/fstab
# /dev/mapper/centos-swap swap                    swap    defaults        0 0
[root@k8s-node01 ~]# mount -a

[root@k8s-node02 ~]# vim /etc/fstab
# /dev/mapper/centos-swap swap                    swap    defaults        0 0
[root@k8s-node02 ~]# mount -a
```

### 6.所有主机调整资源限制

```
[root@k8s-master01 ~]# for i in 10 11 12 13 14
> do
> ssh root@192.168.140.$i ulimit -SHn 65535
> done
```

- `nofile`最大文件数
- `nproc`最大进程数
- `memlock`内页的页面锁

```
[root@k8s-master01 ~]# vim /etc/security/limits.conf
#在文件末尾添加以下内容
* soft nofile 655360
* hard nofile 131072
* soft nproc 655350
* hard nproc 655350
* soft memlock unlimited
* hard memlock unlimited
```

```
[root@k8s-master01 ~]# for i in 11 12 13 14
> do
> scp -r /etc/security/limits.conf root@192.168.140.$i:/etc/security/
> done
```

### 7.配置软件安装源

>**`Centos base源`、`Centos epel源`、`Docker源`、`kubernetes源`**

```
[root@k8s-master01 ~]# wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
[root@k8s-master01 ~]# wget -O /etc/yum.repos.d/epel.repo https://mirrors.aliyun.com/repo/epel-7.repo
```

```
[root@k8s-master01 ~]# vim /etc/yum.repos.d/docker.repo
[docker-ce-stable]
name=Docker CE Stable - $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/$releasever/$basearch/stable
enabled=1
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-stable-debuginfo]
name=Docker CE Stable - Debuginfo $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/$releasever/debug-$basearch/stable
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-stable-source]
name=Docker CE Stable - Sources
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/$releasever/source/stable
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-test]
name=Docker CE Test - $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/$releasever/$basearch/test
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-test-debuginfo]
name=Docker CE Test - Debuginfo $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/$releasever/debug-$basearch/test
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-test-source]
name=Docker CE Test - Sources
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/$releasever/source/test
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-nightly]
name=Docker CE Nightly - $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/$releasever/$basearch/nightly
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-nightly-debuginfo]
name=Docker CE Nightly - Debuginfo $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/$releasever/debug-$basearch/nightly
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[docker-ce-nightly-source]
name=Docker CE Nightly - Sources
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/$releasever/source/nightly
enabled=0
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg
```

```
[root@k8s-master01 ~]# vim /etc/yum.repos.d/k8s.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=0
```

#### 把安装源拷贝给其他机器

```
[root@k8s-master01 ~]# for i in 11 12 13 14
> do
> scp -r /etc/yum.repos.d/*.repo root@192.168.140.$i:/etc/yum.repos.d/
> done
```

#### 建立新的`YUM`缓存

```
[root@k8s-master01 ~]# yum clean all && yum makecache fast
[root@k8s-master02 ~]# yum clean all && yum makecache fast
[root@k8s-master03 ~]# yum clean all && yum makecache fast
[root@k8s-node01 ~]# yum clean all && yum makecache fast
[root@k8s-node02 ~]# yum clean all && yum makecache fast
```

### 8.升级系统到最新版

```
[root@k8s-master01 ~]# yum update -y
[root@k8s-master02 ~]# yum update -y
[root@k8s-master03 ~]# yum update -y
[root@k8s-node01 ~]# yum update -y
[root@k8s-node02 ~]# yum update -y
```

```
[root@k8s-master01 ~]# init 6
[root@k8s-master02 ~]# init 6
[root@k8s-master03 ~]# init 6
[root@k8s-node01 ~]# init 6
[root@k8s-node02 ~]# init 6
```

### 9.升级系统内核(可选的)

>**关于`Centos7`升级内核教程：https://www.wsjj.top/archives/kernel**

```
[root@k8s-master01 ~]# for i in 10 11 12 13 14
> do
> ssh root@192.168.140.$i uname -r
> done

5.4.246-1.el7.elrepo.x86_64
5.4.246-1.el7.elrepo.x86_64
5.4.246-1.el7.elrepo.x86_64
5.4.246-1.el7.elrepo.x86_64
5.4.246-1.el7.elrepo.x86_64
```

## 三、调整系统参数

### 1.所有主机安装`IPVS`

```
[root@k8s-master01 ~]# for i in 10 11 12 13 14
> do
> ssh root@192.168.140.$i yum install ipvsadm ipset sysstat conntrack libseccomp -y
> done
```

#### 所有主机加载`IPVS`模块

```
[root@k8s-master01 ~]# vim /etc/modules-load.d/ipvs.conf
#把以下内容复制到文件内
ip_vs
ip_vs_lc
ip_vs_wlc
ip_vs_rr
ip_vs_wrr
ip_vs_lblc
ip_vs_lblcr
ip_vs_dh
ip_vs_sh
ip_vs_fo
ip_vs_nq
ip_vs_sed
ip_vs_ftp
ip_vs_sh
nf_conntrack_ipv4	#如果使用高版本内核5.x版本，请更名为nf_conntrack
ip_tables
ip_set
xt_set
ipt_set
ipt_rpfilter
ipt_REJECT
ipip
```

```
[root@k8s-master01 ~]# for i in 11 12 13 14
> do
> scp -r /etc/modules-load.d/ipvs.conf root@192.168.140.$i:/etc/modules-load.d/
> done
```

```
[root@k8s-master01 ~]# for i in 10 11 12 13 14
> do
> ssh root@192.168.140.$i systemctl enable --now systemd-modules-load
> done
```

### 2.所有主机调整内核参数

```
[root@k8s-master01 ~]# vim /etc/sysctl.d/k8s.conf
#复制以下内容到配置文件内
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
fs.may_detach_mounts = 1
vm.overcommit_memory=1
vm.panic_on_oom=0
fs.inotify.max_user_watches=89100
fs.file-max=52706963
fs.nr_open=52706963
net.netfilter.nf_conntrack_max=2310720

net.ipv4.tcp_keepalive_time = 600
net.ipv4.tcp_keepalive_probes = 3
net.ipv4.tcp_keepalive_intvl =15
net.ipv4.tcp_max_tw_buckets = 36000
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_max_orphans = 327680
net.ipv4.tcp_orphan_retries = 3
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 16384
net.ipv4.ip_conntrack_max = 65536
net.ipv4.tcp_max_syn_backlog = 16384
net.ipv4.tcp_timestamps = 0
net.core.somaxconn = 16384
```

```
[root@k8s-master01 ~]# for i in 11 12 13 14
> do
> scp -r /etc/sysctl.d/k8s.conf root@192.168.140.$i:/etc/sysctl.d/
> done
```

```
[root@k8s-master01 ~]# for i in 10 11 12 13 14
> do
> ssh root@192.168.140.$i sysctl --system
> done
```

## 四、所有主机安装需要的软件

>**`Docker-ce-19.03`、`kubeadm-1.20.7`、`kubelet-1.20.7`、`kubectl-1.20.7`**

### 1.所有主机安装`Docker-ce-19.03`

```
[root@k8s-master01 ~]# for i in 10 11 12 13 14
> do
> ssh root@192.168.140.$i yum install -y docker-ce-19.03*
> done
```

```
[root@k8s-master01 ~]# for i in 10 11 12 13 14
> do
> ssh root@192.168.140.$i systemctl enable --now docker
> done
```

#### 配置`Docker`镜像加速

```
[root@k8s-master01 ~]# vim /etc/docker/daemon.json
#复制以下内容到配置文件内
{
"registry-mirrors": ["http://hub-mirror.c.163.com",
"https://docker.mirrors.ustc.edu.cn"]
}
```

#### 把配置文件拷贝给其他机器

```
[root@k8s-master01 ~]# for i in 11 12 13 14
> do
> scp -r /etc/docker/daemon.json root@192.168.140.$i:/etc/docker/
> done
```

#### 重启`Docker`

```
[root@k8s-master01 ~]# systemctl restart docker
[root@k8s-master02 ~]# systemctl restart docker
[root@k8s-master03 ~]# systemctl restart docker
[root@k8s-node01 ~]# systemctl restart docker
[root@k8s-node02 ~]# systemctl restart docker
```

### 2.所有主机安装`kubeadm`

```
[root@k8s-master01 ~]# yum install -y kubeadm-1.20.7 kubelet-1.20.7 kubectl-1.20.7
[root@k8s-master02 ~]# yum install -y kubeadm-1.20.7 kubelet-1.20.7 kubectl-1.20.7
[root@k8s-master03 ~]# yum install -y kubeadm-1.20.7 kubelet-1.20.7 kubectl-1.20.7
[root@k8s-node01 ~]# yum install -y kubeadm-1.20.7 kubelet-1.20.7 kubectl-1.20.7
[root@k8s-node02 ~]# yum install -y kubeadm-1.20.7 kubelet-1.20.7 kubectl-1.20.7
```

### 3.修改`kubelet`默认下载地址为国内镜像

```
[root@k8s-master01 ~]# vim /etc/sysconfig/kubelet
KUBELET_EXTRA_ARGS="--pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/google_containers/pause-amd64:3.2"
```

```
[root@k8s-master01 ~]# for i in 11 12 13 14
> do
> scp -r /etc/sysconfig/kubelet root@192.168.140.$i:/etc/sysconfig/
> done
```

```
[root@k8s-master01 ~]# systemctl enable --now kubelet
[root@k8s-master02 ~]# systemctl enable --now kubelet
[root@k8s-master03 ~]# systemctl enable --now kubelet
[root@k8s-node01 ~]# systemctl enable --now kubelet
[root@k8s-node02 ~]# systemctl enable --now kubelet
```

## 五、在三个`Master`节点配置负载均衡和高可用

### 1.在三个`Master`节点安装`HAproxy`和`keepalived`

```
[root@k8s-master01 ~]# for i in 10 11 12
> do
> ssh root@192.168.140.$i yum install -y haproxy keepalived
> done
```

### 2.修改`HAproxy`配置文件

```
[root@k8s-master01 ~]# vim /etc/haproxy/haproxy.cfg

global
  maxconn  2000
  ulimit-n  16384
  log  127.0.0.1 local0 err
  stats timeout 30s

defaults
  log global
  mode  http
  option  httplog
  timeout connect 5000
  timeout client  50000
  timeout server  50000
  timeout http-request 15s
  timeout http-keep-alive 15s

frontend monitor-in
  bind *:33305
  mode http
  option httplog
  monitor-uri /monitor

frontend k8s-master
  bind 0.0.0.0:16443
  bind 127.0.0.1:16443
  mode tcp
  option tcplog
  tcp-request inspect-delay 5s
  default_backend k8s-master

backend k8s-master
  mode tcp	#四层调度
  option tcplog
  option tcp-check
  balance roundrobin
  default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 250 maxqueue 256 weight 100
  server k8s-master01	192.168.140.10:6443  check
  server k8s-master02	192.168.140.11:6443  check
  server k8s-master03	192.168.140.12:6443  check	#我们三个Master节点的IP
```

#### 拷贝配置文件给另外两台`master`节点

```
[root@k8s-master01 ~]# scp -r /etc/haproxy/haproxy.cfg root@192.168.140.11:/etc/haproxy/
haproxy.cfg                                    100%  850   735.5KB/s   00:00    
[root@k8s-master01 ~]# scp -r /etc/haproxy/haproxy.cfg root@192.168.140.12:/etc/haproxy/
haproxy.cfg                                    100%  850   635.7KB/s   00:00
```

#### 启动`HAproxy`

```
[root@k8s-master01 ~]# systemctl enable --now haproxy
[root@k8s-master02 ~]# systemctl enable --now haproxy
[root@k8s-master03 ~]# systemctl enable --now haproxy
```

```
[root@k8s-master01 ~]# netstat -tunlp | grep haproxy
tcp        0      0 0.0.0.0:33305           0.0.0.0:*               LISTEN      3104/haproxy        
tcp        0      0 127.0.0.1:16443         0.0.0.0:*               LISTEN      3104/haproxy        
tcp        0      0 0.0.0.0:16443           0.0.0.0:*               LISTEN      3104/haproxy        
udp        0      0 0.0.0.0:44001           0.0.0.0:*                           3104/haproxy
```

### 3.配置`keepalived`高可用

#### 编写高可用脚本

>**监测`kube-apiserver`状态**

```
[root@k8s-master01 ~]# vim /etc/keepalived/check_apiserver.sh
#!/bin/bash

err=0
for k in $(seq 1 3)
do
    check_code=$(pgrep haproxy)
    if [[ $check_code == "" ]]; then
        err=$(expr $err + 1)
        sleep 1
        continue
    else
        err=0
        break
    fi
done

if [[ $err != "0" ]]; then
    echo "systemctl stop keepalived"
    /usr/bin/systemctl stop keepalived
    exit 1
else
    exit 0
fi
```

```
[root@k8s-master01 ~]# chmod a+x /etc/keepalived/check_apiserver.sh
```

#### 将脚本拷贝给另外两台`master`节点

```
[root@k8s-master01 ~]# rsync -av /etc/keepalived/check_apiserver.sh root@192.168.140.11:/etc/keepalived/
[root@k8s-master01 ~]# rsync -av /etc/keepalived/check_apiserver.sh root@192.168.140.12:/etc/keepalived/
```

#### 修改主节点的`keepalived`配置文件

|主机名|IP地址|VIP|节点|
|-------|-------|-------|-------|
|k8s-master01.linux.com|192.168.140.10|192.168.140.100|主节点MASTER|
|k8s-master02.linux.com|192.168.140.11|192.168.140.100|备用节点BACKUP|
|k8s-master03.linux.com|192.168.140.12|192.168.140.100|备用节点BACKUP|

```
[root@k8s-master01 ~]# vim /etc/keepalived/keepalived.conf
! Configuration File for keepalived

global_defs {
    router_id LVS_DEVEL
script_user root
    enable_script_security
}
vrrp_script chk_apiserver {
    script "/etc/keepalived/check_apiserver.sh"
    interval 5
    weight -5
    fall 2  
    rise 1
}
vrrp_instance VI_1 {
    state MASTER
    interface ens33
    virtual_router_id 51
    priority 101
    advert_int 2
    authentication {
        auth_type PASS
        auth_pass redhat
    }
    virtual_ipaddress {
        192.168.140.100
    }
    track_script {
       chk_apiserver
    }
}
```

#### 配置第一个备用节点`keepalived`配置文件

```
[root@k8s-master02 ~]# vim /etc/keepalived/keepalived.conf 
! Configuration File for keepalived

global_defs {
    router_id LVS_DEVEL
script_user root
    enable_script_security
}
vrrp_script chk_apiserver {
    script "/etc/keepalived/check_apiserver.sh"
    interval 5
    weight -5
    fall 2  
    rise 1
}
vrrp_instance VI_1 {
    state BACKUP	#修改模式为备用
    interface ens33
    virtual_router_id 51
    priority 100	#降低权重
    advert_int 2
    authentication {
        auth_type PASS
        auth_pass redhat
    }
    virtual_ipaddress {
        192.168.140.100
    }
    track_script {
       chk_apiserver
    }
}
```

#### 配置第二个备用节点`keepalived`配置文件

```
[root@k8s-master03 ~]# vim /etc/keepalived/keepalived.conf 
! Configuration File for keepalived

global_defs {
    router_id LVS_DEVEL
script_user root
    enable_script_security
}
vrrp_script chk_apiserver {
    script "/etc/keepalived/check_apiserver.sh"
    interval 5
    weight -5
    fall 2  
    rise 1
}
vrrp_instance VI_1 {
    state BACKUP	#修改模式为备用
    interface ens33
    virtual_router_id 51
    priority 90	#降低权重
    advert_int 2
    authentication {
        auth_type PASS
        auth_pass redhat
    }
    virtual_ipaddress {
        192.168.140.100
    }
    track_script {
       chk_apiserver
    }
}
```

#### 启动三个`Master`节点的`keepalived`

```
[root@k8s-master01 ~]# systemctl enable --now keepalived
[root@k8s-master02 ~]# systemctl enable --now keepalived
[root@k8s-master03 ~]# systemctl enable --now keepalived
```

#### 外部测试`VIP`连通性

```
C:\Users\wangshengjj>ping 192.168.140.100

正在 Ping 192.168.140.100 具有 32 字节的数据:
来自 192.168.140.100 的回复: 字节=32 时间<1ms TTL=64
来自 192.168.140.100 的回复: 字节=32 时间<1ms TTL=64
来自 192.168.140.100 的回复: 字节=32 时间<1ms TTL=64
来自 192.168.140.100 的回复: 字节=32 时间<1ms TTL=64
```

## 六、初始化kubernetes集群

### 1.在主节点`k8s-Master01`准备初始化文件

```
[root@k8s-master01 ~]# kubeadm config print init-defaults > new.yaml
```

```
[root@k8s-master01 ~]# vim new.yaml
#配置文件不完整，仅展示修改部分
localAPIEndpoint:
  advertiseAddress: 192.168.140.10	#指定主API server地址
  bindPort: 6443

apiServer:
  certSANs:
  - 192.168.140.100	#指定证书的签发地址，这里是VIP地址
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controlPlaneEndpoint: 192.168.140.100:16443	#指定主节点地址和端口，同样是VIP和HAProxy的端口
controllerManager: {}

imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers	#指定镜像仓库为国内地址
kind: ClusterConfiguration
kubernetesVersion: v1.20.7	#指定版本信息
networking:
  dnsDomain: cluster.local
  podSubnet: 172.168.0.0/16	#根据规划指定pod网段
  serviceSubnet: 10.96.0.0/12
scheduler: {}
```

#### 下载`Master`节点初始化用到的镜像

```
[root@k8s-master01 ~]# kubeadm config images pull --config /root/new.yaml
```

#### 初始化集群

>**这一步生成的`token`每个人都不一样，请注意！**

```
[root@k8s-master01 ~]# kubeadm init --config /root/new.yaml  --upload-certs

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf	#提示我们配置环境变量

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of the control-plane node running the following command on each as root:
	#用于其他Master节点加入集群
  kubeadm join 192.168.140.100:16443 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:ad8316cb681fea34bf81f1bab6fc2045e386e30347038ff555140bb18ec4417a \
    --control-plane --certificate-key 04ee7ab588714943029e1238f43898c1103af38942b7bca808a7f4dae37ddff9

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:
	#用于Node节点加入集群
kubeadm join 192.168.140.100:16443 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:ad8316cb681fea34bf81f1bab6fc2045e386e30347038ff555140bb18ec4417a
```

#### 配置环境变量

```
[root@k8s-master01 ~]# vim /etc/profile
#在配置文件末尾添加内容
export KUBECONFIG=/etc/kubernetes/admin.conf

[root@k8s-master01 ~]# source /etc/profile	#让环境变量生效
```

#### 查看所有节点

```
[root@k8s-master01 ~]# kubectl get nodes
NAME                     STATUS     ROLES                  AGE     VERSION
k8s-master01.linux.com   NotReady   control-plane,master   5m20s   v1.20.7
```

### 2.将其他`Master`节点加入集群

>**在加入集群过程，需要下载镜像，速度可能较慢，请耐心等待**

```
[root@k8s-master02 ~]# kubeadm join 192.168.140.100:16443 --token abcdef.0123456789abcdef \
>     --discovery-token-ca-cert-hash sha256:ad8316cb681fea34bf81f1bab6fc2045e386e30347038ff555140bb18ec4417a \
>     --control-plane --certificate-key 04ee7ab588714943029e1238f43898c1103af38942b7bca808a7f4dae37ddff9
[root@k8s-master03 ~]# kubeadm join 192.168.140.100:16443 --token abcdef.0123456789abcdef \
>     --discovery-token-ca-cert-hash sha256:ad8316cb681fea34bf81f1bab6fc2045e386e30347038ff555140bb18ec4417a \
>     --control-plane --certificate-key 04ee7ab588714943029e1238f43898c1103af38942b7bca808a7f4dae37ddff9
```

#### 回到主节点查看所有节点

>**可以看到`2`个备用`Master`节点已经加入了**

```
[root@k8s-master01 ~]# kubectl get nodes
NAME                     STATUS     ROLES                  AGE     VERSION
k8s-master01.linux.com   NotReady   control-plane,master   9m49s   v1.20.7
k8s-master02.linux.com   NotReady   control-plane,master   72s     v1.20.7
k8s-master03.linux.com   NotReady   control-plane,master   17s     v1.20.7
```

### 3.将`Node`节点加入集群

```
[root@k8s-node01 ~]# kubeadm join 192.168.140.100:16443 --token abcdef.0123456789abcdef \
>     --discovery-token-ca-cert-hash sha256:ad8316cb681fea34bf81f1bab6fc2045e386e30347038ff555140bb18ec4417a
[root@k8s-node02 ~]# kubeadm join 192.168.140.100:16443 --token abcdef.0123456789abcdef \
>     --discovery-token-ca-cert-hash sha256:ad8316cb681fea34bf81f1bab6fc2045e386e30347038ff555140bb18ec4417a
```

#### 回到主节点查看所有节点

>**此时可以看到所有节点都加入到了集群，但是所有节点都是`NotReady`状态**
>**这是由于我们还没有配置跨物理机的网络造成的，我这里使用`calico`进行演示**

```
[root@k8s-master01 ~]# kubectl get nodes
NAME                     STATUS     ROLES                  AGE     VERSION
k8s-master01.linux.com   NotReady   control-plane,master   13m     v1.20.7
k8s-master02.linux.com   NotReady   control-plane,master   4m47s   v1.20.7
k8s-master03.linux.com   NotReady   control-plane,master   3m52s   v1.20.7
k8s-node01.linux.com     NotReady   <none>                 44s     v1.20.7
k8s-node02.linux.com     NotReady   <none>                 45s     v1.20.7
```

## 七、部署calico网络，实现容器之间的网络通信

- `calico`基于`BGP`协议实现通信
- 只需要在`k8s-master01`节点上进行操作即可

### 1.准备`calico`部署文件

>**下载连接：https://pan.baidu.com/s/14c8lth005DuGjcsBUCJrOA?pwd=ezn7**

### 2.指定`etcd`数据库连接地址

```
[root@k8s-master01 ~]# sed -i 's#etcd_endpoints: "http://<ETCD_IP>:<ETCD_PORT>"#etcd_endpoints: "https://192.168.140.10:2379,https://192.168.140.11:2379,https://192.168.140.12:2379"#g' /root/calico-etcd.yaml
```

### 3.配置证书相关信息

#### 指定证书内容

```
[root@k8s-master01 ~]# ETCD_CA=`cat /etc/kubernetes/pki/etcd/ca.crt | base64 | tr -d '\n'`
[root@k8s-master01 ~]# ETCD_CERT=`cat /etc/kubernetes/pki/etcd/server.crt | base64 | tr -d '\n'`
[root@k8s-master01 ~]# ETCD_KEY=`cat /etc/kubernetes/pki/etcd/server.key | base64 | tr -d '\n'`
[root@k8s-master01 ~]# sed -i "s@# etcd-key: null@etcd-key: ${ETCD_KEY}@g; s@# etcd-cert: null@etcd-cert: ${ETCD_CERT}@g; s@# etcd-ca: null@etcd-ca: ${ETCD_CA}@g" /root/calico-etcd.yaml
```

#### 指定证书文件路径

```
[root@k8s-master01 ~]# sed -i 's#etcd_ca: ""#etcd_ca: "/calico-secrets/etcd-ca"#g; s#etcd_cert: ""#etcd_cert: "/calico-secrets/etcd-cert"#g; s#etcd_key: "" #etcd_key: "/calico-secrets/etcd-key" #g' /root/calico-etcd.yaml
```

### 4.修改`pod`网段

```
[root@k8s-master01 ~]# sed -ri -e 's|            # - name: CALICO_IPV4POOL_CIDR|            - name: CALICO_IPV4POOL_CIDR|' -e 's|            #   value: "192.168.0.0/16"|              value: "192.168.0.0/16"|' calico-etcd.yaml
```

### 5.部署`calico`网络

```
[root@k8s-master01 ~]# kubectl apply -f /root/calico-etcd.yaml
```

### 6.再次查看所有节点状态

>**再次查看所有节点状态，都变成了`Ready`**
>**由于虚拟机配置原因，这个过程肯能需要等`1-3`分钟**

```
[root@k8s-master01 ~]# kubectl get nodes
NAME                     STATUS   ROLES                  AGE   VERSION
k8s-master01.linux.com   Ready    control-plane,master   38m   v1.20.7
k8s-master02.linux.com   Ready    control-plane,master   30m   v1.20.7
k8s-master03.linux.com   Ready    control-plane,master   29m   v1.20.7
k8s-node01.linux.com     Ready    <none>                 26m   v1.20.7
k8s-node02.linux.com     Ready    <none>                 26m   v1.20.7
```

### 7.查看所有`pods`状态

>**这个过程取决于虚拟机的配置和网络情况，全变成`running`和`READY`需要`2-3`分钟左右**

```
[root@k8s-master01 ~]# kubectl get pods -n kube-system
NAME                                             READY   STATUS    RESTARTS   AGE
calico-kube-controllers-5f6d4b864b-k4sfs         1/1     Running   0          4m33s
calico-node-8nctk                                1/1     Running   0          4m33s
calico-node-hv4sh                                1/1     Running   0          4m33s
calico-node-nr52x                                1/1     Running   0          4m33s
calico-node-qjk7m                                1/1     Running   0          4m33s
calico-node-zmqtr                                1/1     Running   0          4m33s
coredns-54d67798b7-4bb7m                         1/1     Running   0          40m
coredns-54d67798b7-mtqpz                         1/1     Running   0          40m
etcd-k8s-master01.linux.com                      1/1     Running   0          40m
etcd-k8s-master02.linux.com                      1/1     Running   0          32m
etcd-k8s-master03.linux.com                      1/1     Running   0          29m
kube-apiserver-k8s-master01.linux.com            1/1     Running   0          40m
kube-apiserver-k8s-master02.linux.com            1/1     Running   0          32m
kube-apiserver-k8s-master03.linux.com            1/1     Running   1          29m
kube-controller-manager-k8s-master01.linux.com   1/1     Running   1          40m
kube-controller-manager-k8s-master02.linux.com   1/1     Running   0          32m
kube-controller-manager-k8s-master03.linux.com   1/1     Running   0          30m
kube-proxy-6hccv                                 1/1     Running   0          31m
kube-proxy-9cscl                                 1/1     Running   0          32m
kube-proxy-hq7hv                                 1/1     Running   0          28m
kube-proxy-jkkch                                 1/1     Running   0          40m
kube-proxy-jp9kq                                 1/1     Running   0          28m
kube-scheduler-k8s-master01.linux.com            1/1     Running   1          40m
kube-scheduler-k8s-master02.linux.com            1/1     Running   0          32m
kube-scheduler-k8s-master03.linux.com            1/1     Running   0          30m
```

>**恭喜你，如果你跟到了这里，那么一个`kubernetes`的高可用集群就部署完毕啦！**
>**教程到这里就结束啦，如果你想部署图形化界面或者节点健康监测，可以观看这期教程：[Kubernetes集群部署](https://www.wsjj.top/archives/138)**
