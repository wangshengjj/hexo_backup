---
title: Centos7安装并配置Kubernetes(K8s)
date: 2022-11-20 17:39:25.572
updated: 2023-01-04 17:36:25.853
sticky: 3
categories: 
- 服务器搭建
- 技术
- Docker
- 虚拟机
- k8s
tags: 
- docker
- 服务器搭建
- linux搭建服务器
- centos
- 虚拟机
- k8s
- kubernetes
---

# Centos7安装并配置Kubernetes(K8s)

**关于K8s介绍：**

Kubernetes 是一个全新的基于容器技术的分布式架构解决方案，是 Google 开源的一个容器集群管理系统，Kubernetes 简称 K8S。

Kubernetes 是一个一站式的完备的分布式系统开发和支撑平台，更是一个开放平台，对现有的编程语言、编程框架、中间件没有任何侵入性。

Kubernetes 提供了完善的管理工具，这些工具涵盖了开发、部署测试、运维监控在内的各个环节。

Kubernetes 具有完备的集群管理能力，包括多层次的安全防护和准入机制、多租户应用支撑能力、透明的服务注册和服务发现机制、内建智能负载均衡器、强大的故障发现和自我修复能力、服务滚动升级和在线扩容能力、可扩展的资源自动调度机制、多粒度的资源配额管理能力。

Kubernetes 官方文档：https://kubernetes.io/zh/

第二期教程：[Kubernetes(K8s)部署WordPress博客](https://www.wangshengjj.work/archives/23)

## 一、准备阶段

本教程第十步、第十一步、第十二步涉及到的插件[下载链接](https://cloud.wangshengjj.work/s/zYcp)

### 三台虚拟机：

ip:192.168.137.130(根据自己的来)	域名：k8s-master.linux.com	Master	**2核2G**(官方要求最低配置)	系统：Centos7.9

ip:192.168.137.129(根据自己的来)	域名：k8s-node1.linux.com	Node1	**2核2G**(官方要求最低配置)	系统：Centos7.9

ip:192.168.137.128(根据自己的来)	域名：k8s-node2.linux.com	Node2	**2核2G**(官方要求最低配置)	系统：Centos7.9

### 软件版本

kubernetes 1.20.7版本

docker   19.03版本(版本过高会报错)

### 网段

pod网段:    10.88.0.0/16

service网段:  172.16.0.0/16

## 二、关闭SElinux和防火墙(三台虚拟机都要配置)

### 1.关闭防火墙并且设置开机不自启

```
systemctl stop firewalld && systemctl disable firewalld
```

### 2.关闭selinux

```
setenforce 0
```

```
sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
```

### 3.重启

```
reboot
```

### 4.配置yum源和epel源

yum源

```
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```

epel源

```
wget -O /etc/yum.repos.d/epel.repo https://mirrors.aliyun.com/repo/epel-7.repo
```

```
yum clean all && yum makecache fast
```


## 三、修改主机名和域名

### 1.修改主机名

```
[root@k8s-master ~]# hostnamectl set-hostname k8s-master.linux.com
```

```
[root@k8s-node1 ~]# hostnamectl set-hostname k8s-node1.linux.com
```

```
[root@k8s-node2 ~]# hostnamectl set-hostname k8s-node2.linux.com
```

### 2.修改hosts

```
[root@k8s-master ~]# vim /etc/hosts
```

```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.137.130	k8s-master.linux.com	#master的IP和域名
192.168.137.129	k8s-node01.linux.com	#node1的IP和域名
192.168.137.128	k8s-node02.linux.com	#node2的IP和域名
```

```
[root@k8s-master ~]# scp /etc/hosts root@192.168.137.129:/etc/hosts	#把master的hosts文件拷贝给node1
hosts                                                                                                                      100%  267   154.5KB/s   00:00    
[root@k8s-master ~]# scp /etc/hosts root@192.168.137.128:/etc/hosts	#把master的hosts文件拷贝给node2
hosts                                                                                                                      100%  267   154.5KB/s 
```

### 3.使用ping命令测试

```
ping k8s-node1.linux.com
```

```
ping k8s-node8.linux.com
```

## 四、给三台虚拟机配置免密SSH(master上操作)

```
[root@k8s-master ~]# ssh-keygen -t rsa
```

```
[root@k8s-master ~]# mv /root/.ssh/id_rsa.pub /root/.ssh/authorized_keys
```

```
[root@k8s-master ~]# scp -r /root/.ssh/ root@192.168.137.129:/root/	#把秘钥文件拷贝给node1
[root@k8s-master ~]# scp -r /root/.ssh/ root@192.168.137.128:/root/	#把秘钥文件拷贝给node2
```

## 五、配置主机资源

### 1.所有主机禁用swap交换分区(master、node1、node2)

```
[root@k8s-master ~]# swapoff -a
```

```
[root@k8s-master ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:           3932         113        3658          11         160        3599
Swap:             0           0           0
```

```
[root@k8s-master ~]# sysctl -w vm.swappiness=0
vm.swappiness = 0
```

```
[root@k8s-master ~]# sed -ri '/swap/d' /etc/fstab
```

### 2.所有主机调整系统的资源限制(master、node1、node2)

```
[root@k8s-master ~]# ulimit -SHn 65535
```

```
[root@k8s-master ~]# vim /etc/security/limits.conf
```

把下面的添加进去

```
* soft nofile 655360
* hard nofile 131072
* soft nproc 655350
* hard nproc 655350
* soft memlock unlimited
* hard memlock unlimited
```

：wq保存

- nofile
最大文件数
- nproc
最大进程数
- memlock
内页的页面锁

```
[root@k8s-master ~]# scp /etc/security/limits.conf root@192.168.137.129:/etc/security/limits.conf	#拷贝文件到node1
limits.conf                                                                                                                100% 2555     1.4MB/s   00:00    
[root@k8s-master ~]# scp /etc/security/limits.conf root@192.168.137.128:/etc/security/limits.conf	#拷贝文件到node2
limits.conf                                                                                                                100% 2555     2.1MB/s   00:00
```

## 六、配置Docekr仓库和K8s仓库

### 1.配置Docker仓库(阿里云)

```
[root@k8s-master ~]# vim /etc/yum.repos.d/docker.repo
```

把下面内容复制进去

```
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

：wq保存

### 2.配置K8s仓库(阿里云)

```
[root@k8s-master ~]# vim /etc/yum.repos.d/k8s.repo
```

```
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
```

：wq保存

### 3.讲上述2个仓库拷贝到node1和node2

```
[root@k8s-master ~]# scp /etc/yum.repos.d/docker.repo root@192.168.137.129:/etc/yum.repos.d/
docker.repo                                                                                                                100% 2081     1.8MB/s   00:00    
[root@k8s-master ~]# scp /etc/yum.repos.d/docker.repo root@192.168.137.128:/etc/yum.repos.d/
docker.repo                                                                                                                100% 2081     2.8MB/s   00:00    
[root@k8s-master ~]# 
[root@k8s-master ~]# scp /etc/yum.repos.d/k8s.repo root@192.168.137.129:/etc/yum.repos.d/
k8s.repo                                                                                                                   100%  276   360.6KB/s   00:00    
[root@k8s-master ~]# scp /etc/yum.repos.d/k8s.repo root@192.168.137.128:/etc/yum.repos.d/
k8s.repo
```

### 4.升级系统到最新版本(Centos7.9)

```
yum update -y	#三个虚拟机都要执行
```

```
reboot
```

查看系统版本

```
[root@k8s-master ~]# uname -a
Linux k8s-master.linux.com 3.10.0-1160.80.1.el7.x86_64 #1 SMP Tue Nov 8 15:48:59 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
```

## 七、调整系统参数(master、node1、node2)

### 1.安装IPVS

```
[root@k8s-master ~]# yum install ipvsadm ipset sysstat conntrack libseccomp -y	#三个主机都要运行
```

### 2.加载IPVS模块

```
[root@k8s-master ~]# vim /etc/modules-load.d/ipvs.conf
```

```
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
nf_conntrack_ipv4
ip_tables
ip_set
xt_set
ipt_set
ipt_rpfilter
ipt_REJECT
ipip
```

：wq保存

```
[root@k8s-master ~]# scp /etc/modules-load.d/ipvs.conf root@192.168.137.129:/etc/modules-load.d/ipvs.conf	#拷贝到node1
ipvs.conf                                                                                                                  100%  211   123.4KB/s   00:00    
[root@k8s-master ~]# scp /etc/modules-load.d/ipvs.conf root@192.168.137.128:/etc/modules-load.d/ipvs.conf	#拷贝到node2
ipvs.conf
```

```
[root@k8s-master ~]# systemctl enable --now systemd-modules-load	#三台虚拟机都要运行
```

### 4.调整内核参数

```
[root@k8s-master ~]# vim /etc/sysctl.d/k8s.conf
```

```
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

：wq保存

```
[root@k8s-master ~]# scp /etc/sysctl.d/k8s.conf root@192.168.137.129:/etc/sysctl.d/	#拷贝到node1
k8s.conf                                                                                                                   100%  704   654.4KB/s   00:00    
[root@k8s-master ~]# scp /etc/sysctl.d/k8s.conf root@192.168.137.128:/etc/sysctl.d/	#拷贝到node2
k8s.conf
```

```
[root@k8s-master ~]# sysctl --system	#三个虚拟机都要运行
```

## 八、安装软件(master、node1、node2)

### 1.安装Docker

```
[root@k8s-master ~]# yum install -y docker-ce-19.03*	#注意版本
```

```
[root@k8s-master ~]# systemctl enable --now docker
```

**配置镜像加速**

```
[root@k8s-master ~]# vim /etc/docker/daemon.json
```

```
{
"registry-mirrors": ["http://hub-mirror.c.163.com",
"https://docker.mirrors.ustc.edu.cn"]
}
```

：wq保存

**重启服务**

```
[root@k8s-master ~]# systemctl restart docker
```

### 2.安装kubernetes

```
[root@k8s-master ~]# yum install -y kubeadm-1.20.7 kubelet-1.20.7 kubectl-1.20.7
```

```
[root@k8s-master ~]# vim /etc/sysconfig/kubelet
```

```
KUBELET_EXTRA_ARGS="--pod-infra-container-image=registry.cn-hangzhou.aliyuncs.com/google_containers/pause-amd64:3.2"
```

：wq保存

```
[root@k8s-master ~]# systemctl enable --now kubelet
```

## 九、配置master节点的初始文件

```
[root@k8s-master ~]# vim new.yaml
```

```
apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: 7t2weq.bjbawausm0jaxury
  ttl: 96h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 192.168.137.130      #master节点IP
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: k8s-master01
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
  certSANs:
  - 192.168.183.10              #master节点IP
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controlPlaneEndpoint: 192.168.137.130:6443            #master节点IP
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers
kind: ClusterConfiguration
kubernetesVersion: v1.20.7
networking:
  dnsDomain: cluster.local
  podSubnet: 10.88.0.0/16	#之前配置的网段
  serviceSubnet: 172.16.0.0/16	#之前配置的网段
scheduler: {}
```

：wq保存

### 1.在master节点事先下载初始化需要的镜像

```
[root@k8s-master ~]# kubeadm config images pull --config /root/new.yaml
```

### 2.在master节点上初始化集群

```
[root@k8s-master ~]# kubeadm init --config /root/new.yaml --upload-certs
```

输出信息如下图

```
[init] Using Kubernetes version: v1.20.7
[preflight] Running pre-flight checks
	[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
	[WARNING Hostname]: hostname "k8s-master01" could not be reached
	[WARNING Hostname]: hostname "k8s-master01": lookup k8s-master01 on 114.114.114.114:53: no such host
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [k8s-master01 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [172.16.0.1 192.168.183.10]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [k8s-master01 localhost] and IPs [192.168.183.10 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [k8s-master01 localhost] and IPs [192.168.183.10 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 12.026038 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.20" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Storing the certificates in Secret "kubeadm-certs" in the "kube-system" Namespace
[upload-certs] Using certificate key:
4c95b29d3f062b0b005d7f6ddd74dc1b227ad8f76dbe5b5c2a13da883d93d73f
[mark-control-plane] Marking the node k8s-master01 as control-plane by adding the labels "node-role.kubernetes.io/master=''" and "node-role.kubernetes.io/control-plane='' (deprecated)"
[mark-control-plane] Marking the node k8s-master01 as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: 7t2weq.bjbawausm0jaxury
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of the control-plane node running the following command on each as root:

  kubeadm join 192.168.137.130:6443 --token 7t2weq.bjbawausm0jaxury \
    --discovery-token-ca-cert-hash sha256:26b04c860890abb6fabbca8dd194eb089aa6a3811be1fe2edff3b4a08c6bb38c \
    --control-plane --certificate-key 4c95b29d3f062b0b005d7f6ddd74dc1b227ad8f76dbe5b5c2a13da883d93d73f

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.137.130:6443 --token 7t2weq.bjbawausm0jaxury \
    --discovery-token-ca-cert-hash sha256:26b04c860890abb6fabbca8dd194eb089aa6a3811be1fe2edff3b4a08c6bb38c	#一会node加入master的命令，注意：每个人的token不一样！
```

### 3.定义kubeconfig环境变量

```
[root@k8s-master ~]# vim /etc/profile
```

在末尾加上

```
export KUBECONFIG=/etc/kubernetes/admin.conf
```

：wq保存

```
[root@k8s-master ~]# source /etc/profile
```

### 4.将node节点加入master

```
[root@k8s-node01 ~]# kubeadm join 192.168.137.130:6443 --token 7t2weq.bjbawausm0jaxury \
>     --discovery-token-ca-cert-hash sha256:26b04c860890abb6fabbca8dd194eb089aa6a3811be1fe2edff3b4a08c6bb38c	#在node1上运行，注意！每个人的token不一样，看自己终端生成的！
```

```
[root@k8s-node02 ~]# kubeadm join 192.168.137.130:6443 --token 7t2weq.bjbawausm0jaxury \
>     --discovery-token-ca-cert-hash sha256:26b04c860890abb6fabbca8dd194eb089aa6a3811be1fe2edff3b4a08c6bb38c	#在node2上运行，注意！每个人的token不一样，看自己终端生成的！
```

### 5.查看集群节点

```
[root@k8s-master ~]# kubectl get nodes
NAME                   STATUS     ROLES                  AGE     VERSION
k8s-master01           NotReady   control-plane,master   7m55s   v1.20.7
k8s-node01.linux.com   NotReady   <none>                 97s     v1.20.7
k8s-node02.linux.com   NotReady   <none>                 42s     v1.20.7
```

## 十、部署calico网络，实现容器间的通信

- calico基于BGP协议实现通信 

- 只需要在master节点上进行操作即可

```
[root@k8s-master ~]# vim calico-etcd.yaml
```

把下面的内容全部复制进去

```
---
# Source: calico/templates/calico-etcd-secrets.yaml
# The following contains k8s Secrets for use with a TLS enabled etcd cluster.
# For information on populating Secrets, see http://kubernetes.io/docs/user-guide/secrets/
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: calico-etcd-secrets
  namespace: kube-system
data:
  # Populate the following with etcd TLS configuration if desired, but leave blank if
  # not using TLS for etcd.
  # The keys below should be uncommented and the values populated with the base64
  # encoded contents of each file that would be associated with the TLS data.
  # Example command for encoding a file contents: cat <file> | base64 -w 0
  # etcd-key: null
  # etcd-cert: null
  # etcd-ca: null
---
# Source: calico/templates/calico-config.yaml
# This ConfigMap is used to configure a self-hosted Calico installation.
kind: ConfigMap
apiVersion: v1
metadata:
  name: calico-config
  namespace: kube-system
data:
  # Configure this with the location of your etcd cluster.
  etcd_endpoints: "http://<ETCD_IP>:<ETCD_PORT>"
  # If you're using TLS enabled etcd uncomment the following.
  # You must also populate the Secret below with these files.
  etcd_ca: ""   # "/calico-secrets/etcd-ca"
  etcd_cert: "" # "/calico-secrets/etcd-cert"
  etcd_key: ""  # "/calico-secrets/etcd-key"
  # Typha is disabled.
  typha_service_name: "none"
  # Configure the backend to use.
  calico_backend: "bird"
  # Configure the MTU to use for workload interfaces and tunnels.
  # - If Wireguard is enabled, set to your network MTU - 60
  # - Otherwise, if VXLAN or BPF mode is enabled, set to your network MTU - 50
  # - Otherwise, if IPIP is enabled, set to your network MTU - 20
  # - Otherwise, if not using any encapsulation, set to your network MTU.
  veth_mtu: "1440"

  # The CNI network configuration to install on each node. The special
  # values in this config will be automatically populated.
  cni_network_config: |-
    {
      "name": "k8s-pod-network",
      "cniVersion": "0.3.1",
      "plugins": [
        {
          "type": "calico",
          "log_level": "info",
          "etcd_endpoints": "__ETCD_ENDPOINTS__",
          "etcd_key_file": "__ETCD_KEY_FILE__",
          "etcd_cert_file": "__ETCD_CERT_FILE__",
          "etcd_ca_cert_file": "__ETCD_CA_CERT_FILE__",
          "mtu": __CNI_MTU__,
          "ipam": {
              "type": "calico-ipam"
          },
          "policy": {
              "type": "k8s"
          },
          "kubernetes": {
              "kubeconfig": "__KUBECONFIG_FILEPATH__"
          }
        },
        {
          "type": "portmap",
          "snat": true,
          "capabilities": {"portMappings": true}
        },
        {
          "type": "bandwidth",
          "capabilities": {"bandwidth": true}
        }
      ]
    }

---
# Source: calico/templates/calico-kube-controllers-rbac.yaml

# Include a clusterrole for the kube-controllers component,
# and bind it to the calico-kube-controllers serviceaccount.
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: calico-kube-controllers
rules:
  # Pods are monitored for changing labels.
  # The node controller monitors Kubernetes nodes.
  # Namespace and serviceaccount labels are used for policy.
  - apiGroups: [""]
    resources:
      - pods
      - nodes
      - namespaces
      - serviceaccounts
    verbs:
      - watch
      - list
      - get
  # Watch for changes to Kubernetes NetworkPolicies.
  - apiGroups: ["networking.k8s.io"]
    resources:
      - networkpolicies
    verbs:
      - watch
      - list
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: calico-kube-controllers
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: calico-kube-controllers
subjects:
- kind: ServiceAccount
  name: calico-kube-controllers
  namespace: kube-system
---

---
# Source: calico/templates/calico-node-rbac.yaml
# Include a clusterrole for the calico-node DaemonSet,
# and bind it to the calico-node serviceaccount.
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: calico-node
rules:
  # The CNI plugin needs to get pods, nodes, and namespaces.
  - apiGroups: [""]
    resources:
      - pods
      - nodes
      - namespaces
    verbs:
      - get
  - apiGroups: [""]
    resources:
      - endpoints
      - services
    verbs:
      # Used to discover service IPs for advertisement.
      - watch
      - list
  # Pod CIDR auto-detection on kubeadm needs access to config maps.
  - apiGroups: [""]
    resources:
      - configmaps
    verbs:
      - get
  - apiGroups: [""]
    resources:
      - nodes/status
    verbs:
      # Needed for clearing NodeNetworkUnavailable flag.
      - patch

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: calico-node
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: calico-node
subjects:
- kind: ServiceAccount
  name: calico-node
  namespace: kube-system

---
# Source: calico/templates/calico-node.yaml
# This manifest installs the calico-node container, as well
# as the CNI plugins and network config on
# each master and worker node in a Kubernetes cluster.
kind: DaemonSet
apiVersion: apps/v1
metadata:
  name: calico-node
  namespace: kube-system
  labels:
    k8s-app: calico-node
spec:
  selector:
    matchLabels:
      k8s-app: calico-node
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
  template:
    metadata:
      labels:
        k8s-app: calico-node
    spec:
      nodeSelector:
        kubernetes.io/os: linux
      hostNetwork: true
      tolerations:
        # Make sure calico-node gets scheduled on all nodes.
        - effect: NoSchedule
          operator: Exists
        # Mark the pod as a critical add-on for rescheduling.
        - key: CriticalAddonsOnly
          operator: Exists
        - effect: NoExecute
          operator: Exists
      serviceAccountName: calico-node
      # Minimize downtime during a rolling upgrade or deletion; tell Kubernetes to do a "force
      # deletion": https://kubernetes.io/docs/concepts/workloads/pods/pod/#termination-of-pods.
      terminationGracePeriodSeconds: 0
      priorityClassName: system-node-critical
      initContainers:
        # This container installs the CNI binaries
        # and CNI network config file on each node.
        - name: install-cni
          image: registry.cn-beijing.aliyuncs.com/dotbalo/cni:v3.15.3
          command: ["/install-cni.sh"]
          env:
            # Name of the CNI config file to create.
            - name: CNI_CONF_NAME
              value: "10-calico.conflist"
            # The CNI network config to install on each node.
            - name: CNI_NETWORK_CONFIG
              valueFrom:
                configMapKeyRef:
                  name: calico-config
                  key: cni_network_config
            # The location of the etcd cluster.
            - name: ETCD_ENDPOINTS
              valueFrom:
                configMapKeyRef:
                  name: calico-config
                  key: etcd_endpoints
            # CNI MTU Config variable
            - name: CNI_MTU
              valueFrom:
                configMapKeyRef:
                  name: calico-config
                  key: veth_mtu
            # Prevents the container from sleeping forever.
            - name: SLEEP
              value: "false"
          volumeMounts:
            - mountPath: /host/opt/cni/bin
              name: cni-bin-dir
            - mountPath: /host/etc/cni/net.d
              name: cni-net-dir
            - mountPath: /calico-secrets
              name: etcd-certs
          securityContext:
            privileged: true
        # Adds a Flex Volume Driver that creates a per-pod Unix Domain Socket to allow Dikastes
        # to communicate with Felix over the Policy Sync API.
        - name: flexvol-driver
          image: registry.cn-beijing.aliyuncs.com/dotbalo/pod2daemon-flexvol:v3.15.3
          volumeMounts:
          - name: flexvol-driver-host
            mountPath: /host/driver
          securityContext:
            privileged: true
      containers:
        # Runs calico-node container on each Kubernetes node. This
        # container programs network policy and routes on each
        # host.
        - name: calico-node
          image: registry.cn-beijing.aliyuncs.com/dotbalo/node:v3.15.3
          env:
            # The location of the etcd cluster.
            - name: ETCD_ENDPOINTS
              valueFrom:
                configMapKeyRef:
                  name: calico-config
                  key: etcd_endpoints
            # Location of the CA certificate for etcd.
            - name: ETCD_CA_CERT_FILE
              valueFrom:
                configMapKeyRef:
                  name: calico-config
                  key: etcd_ca
            # Location of the client key for etcd.
            - name: ETCD_KEY_FILE
              valueFrom:
                configMapKeyRef:
                  name: calico-config
                  key: etcd_key
            # Location of the client certificate for etcd.
            - name: ETCD_CERT_FILE
              valueFrom:
                configMapKeyRef:
                  name: calico-config
                  key: etcd_cert
            # Set noderef for node controller.
            - name: CALICO_K8S_NODE_REF
              valueFrom:
                fieldRef:
                  fieldPath: spec.nodeName
            # Choose the backend to use.
            - name: CALICO_NETWORKING_BACKEND
              valueFrom:
                configMapKeyRef:
                  name: calico-config
                  key: calico_backend
            # Cluster type to identify the deployment type
            - name: CLUSTER_TYPE
              value: "k8s,bgp"
            # Auto-detect the BGP IP address.
            - name: IP
              value: "autodetect"
            # Enable IPIP
            - name: CALICO_IPV4POOL_IPIP
              value: "Always"
            # Enable or Disable VXLAN on the default IP pool.
            - name: CALICO_IPV4POOL_VXLAN
              value: "Never"
            # Set MTU for tunnel device used if ipip is enabled
            - name: FELIX_IPINIPMTU
              valueFrom:
                configMapKeyRef:
                  name: calico-config
                  key: veth_mtu
            # Set MTU for the VXLAN tunnel device.
            - name: FELIX_VXLANMTU
              valueFrom:
                configMapKeyRef:
                  name: calico-config
                  key: veth_mtu
            # Set MTU for the Wireguard tunnel device.
            - name: FELIX_WIREGUARDMTU
              valueFrom:
                configMapKeyRef:
                  name: calico-config
                  key: veth_mtu
            # The default IPv4 pool to create on startup if none exists. Pod IPs will be
            # chosen from this range. Changing this value after installation will have
            # no effect. This should fall within `--cluster-cidr`.
            # - name: CALICO_IPV4POOL_CIDR
            #   value: "192.168.0.0/16"
            # Disable file logging so `kubectl logs` works.
            - name: CALICO_DISABLE_FILE_LOGGING
              value: "true"
            # Set Felix endpoint to host default action to ACCEPT.
            - name: FELIX_DEFAULTENDPOINTTOHOSTACTION
              value: "ACCEPT"
            # Disable IPv6 on Kubernetes.
            - name: FELIX_IPV6SUPPORT
              value: "false"
            # Set Felix logging to "info"
            - name: FELIX_LOGSEVERITYSCREEN
              value: "info"
            - name: FELIX_HEALTHENABLED
              value: "true"
          securityContext:
            privileged: true
          resources:
            requests:
              cpu: 250m
          livenessProbe:
            exec:
              command:
              - /bin/calico-node
              - -felix-live
              - -bird-live
            periodSeconds: 10
            initialDelaySeconds: 10
            failureThreshold: 6
          readinessProbe:
            exec:
              command:
              - /bin/calico-node
              - -felix-ready
              - -bird-ready
            periodSeconds: 10
          volumeMounts:
            - mountPath: /lib/modules
              name: lib-modules
              readOnly: true
            - mountPath: /run/xtables.lock
              name: xtables-lock
              readOnly: false
            - mountPath: /var/run/calico
              name: var-run-calico
              readOnly: false
            - mountPath: /var/lib/calico
              name: var-lib-calico
              readOnly: false
            - mountPath: /calico-secrets
              name: etcd-certs
            - name: policysync
              mountPath: /var/run/nodeagent
      volumes:
        # Used by calico-node.
        - name: lib-modules
          hostPath:
            path: /lib/modules
        - name: var-run-calico
          hostPath:
            path: /var/run/calico
        - name: var-lib-calico
          hostPath:
            path: /var/lib/calico
        - name: xtables-lock
          hostPath:
            path: /run/xtables.lock
            type: FileOrCreate
        # Used to install CNI.
        - name: cni-bin-dir
          hostPath:
            path: /opt/cni/bin
        - name: cni-net-dir
          hostPath:
            path: /etc/cni/net.d
        # Mount in the etcd TLS secrets with mode 400.
        # See https://kubernetes.io/docs/concepts/configuration/secret/
        - name: etcd-certs
          secret:
            secretName: calico-etcd-secrets
            defaultMode: 0400
        # Used to create per-pod Unix Domain Sockets
        - name: policysync
          hostPath:
            type: DirectoryOrCreate
            path: /var/run/nodeagent
        # Used to install Flex Volume Driver
        - name: flexvol-driver-host
          hostPath:
            type: DirectoryOrCreate
            path: /usr/libexec/kubernetes/kubelet-plugins/volume/exec/nodeagent~uds
---

apiVersion: v1
kind: ServiceAccount
metadata:
  name: calico-node
  namespace: kube-system

---
# Source: calico/templates/calico-kube-controllers.yaml
# See https://github.com/projectcalico/kube-controllers
apiVersion: apps/v1
kind: Deployment
metadata:
  name: calico-kube-controllers
  namespace: kube-system
  labels:
    k8s-app: calico-kube-controllers
spec:
  # The controllers can only have a single active instance.
  replicas: 1
  selector:
    matchLabels:
      k8s-app: calico-kube-controllers
  strategy:
    type: Recreate
  template:
    metadata:
      name: calico-kube-controllers
      namespace: kube-system
      labels:
        k8s-app: calico-kube-controllers
    spec:
      nodeSelector:
        kubernetes.io/os: linux
      tolerations:
        # Mark the pod as a critical add-on for rescheduling.
        - key: CriticalAddonsOnly
          operator: Exists
        - key: node-role.kubernetes.io/master
          effect: NoSchedule
      serviceAccountName: calico-kube-controllers
      priorityClassName: system-cluster-critical
      # The controllers must run in the host network namespace so that
      # it isn't governed by policy that would prevent it from working.
      hostNetwork: true
      containers:
        - name: calico-kube-controllers
          image: registry.cn-beijing.aliyuncs.com/dotbalo/kube-controllers:v3.15.3
          env:
            # The location of the etcd cluster.
            - name: ETCD_ENDPOINTS
              valueFrom:
                configMapKeyRef:
                  name: calico-config
                  key: etcd_endpoints
            # Location of the CA certificate for etcd.
            - name: ETCD_CA_CERT_FILE
              valueFrom:
                configMapKeyRef:
                  name: calico-config
                  key: etcd_ca
            # Location of the client key for etcd.
            - name: ETCD_KEY_FILE
              valueFrom:
                configMapKeyRef:
                  name: calico-config
                  key: etcd_key
            # Location of the client certificate for etcd.
            - name: ETCD_CERT_FILE
              valueFrom:
                configMapKeyRef:
                  name: calico-config
                  key: etcd_cert
            # Choose which controllers to run.
            - name: ENABLED_CONTROLLERS
              value: policy,namespace,serviceaccount,workloadendpoint,node
          volumeMounts:
            # Mount in the etcd TLS secrets.
            - mountPath: /calico-secrets
              name: etcd-certs
          readinessProbe:
            exec:
              command:
              - /usr/bin/check-status
              - -r
      volumes:
        # Mount in the etcd TLS secrets with mode 400.
        # See https://kubernetes.io/docs/concepts/configuration/secret/
        - name: etcd-certs
          secret:
            secretName: calico-etcd-secrets
            defaultMode: 0400

---

apiVersion: v1
kind: ServiceAccount
metadata:
  name: calico-kube-controllers
  namespace: kube-system

---
# Source: calico/templates/calico-typha.yaml

---
# Source: calico/templates/configure-canal.yaml

---
# Source: calico/templates/kdd-crds.yaml
```

：wq保存

### 1.在calico-etcd.yaml指定etcd数据库的连接地址

```
[root@k8s-master ~]# sed -i 's#etcd_endpoints: "http://<ETCD_IP>:<ETCD_PORT>"#etcd_endpoints: "https://192.168.137.130:2379"#g' calico-etcd.yaml	#注意：命令后面的IP地址为master的IP
```

### 2.指定连接etcd数据库使用的证书信息

```
[root@k8s-master ~]# ETCD_CA=`cat /etc/kubernetes/pki/etcd/ca.crt | base64 | tr -d '\n'`
[root@k8s-master ~]# ETCD_CERT=`cat /etc/kubernetes/pki/etcd/server.crt | base64 | tr -d '\n'`
[root@k8s-master ~]# ETCD_KEY=`cat /etc/kubernetes/pki/etcd/server.key | base64 | tr -d '\n'`
[root@k8s-master ~]# sed -i "s@# etcd-key: null@etcd-key: ${ETCD_KEY}@g; s@# etcd-cert: null@etcd-cert: ${ETCD_CERT}@g; s@# etcd-ca: null@etcd-ca: ${ETCD_CA}@g" calico-etcd.yaml
```

```
[root@k8s-master ~]# sed -i 's#etcd_ca: ""#etcd_ca: "/calico-secrets/etcd-ca"#g; s#etcd_cert: ""#etcd_cert: "/calico-secrets/etcd-cert"#g; s#etcd_key: "" #etcd_key: "/calico-secrets/etcd-key" #g' calico-etcd.yaml
```

### 3.指定通信的网段

```
[root@k8s-master ~]# vim calico-etcd.yaml
```

```
346             # no effect. This should fall within `--cluster-cidr`.
347             - name: CALICO_IPV4POOL_CIDR	#把这里的注释删掉，并且和下面的对齐
348               value: "192.168.0.0/16"	#把这里的注释删掉，并且和下面的对齐
349             # Disable file logging so `kubectl logs` works.
350             - name: CALICO_DISABLE_FILE_LOGGING
351               value: "true"
352             # Set Felix endpoint to host default action to ACCEPT.
353             - name: FELIX_DEFAULTENDPOINTTOHOSTACTION
354               value: "ACCEPT"
355             # Disable IPv6 on Kubernetes.
356             - name: FELIX_IPV6SUPPORT
357               value: "false"
358             # Set Felix logging to "info"
359             - name: FELIX_LOGSEVERITYSCREEN
360               value: "info"
361             - name: FELIX_HEALTHENABLED
```

### 4.部署calico网络

```
[root@k8s-master ~]# kubectl create -f calico-etcd.yaml
```

```
[root@k8s-master ~]# kubectl get pods -A -o wide
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE    IP               NODE                   NOMINATED NODE   READINESS GATES
kube-system   calico-kube-controllers-5f6d4b864b-sv9kc   1/1     Running   0          10m    192.168.137.130  k8s-master01           <none>           <none>
kube-system   calico-node-bcsxg                          1/1     Running   0          10m    192.168.137.129   k8s-node01.linux.com   <none>           <none>
kube-system   calico-node-g6k4d                          1/1     Running   0          10m    192.168.137.128   k8s-node02.linux.com   <none>           <none>
kube-system   calico-node-wp8hj                          1/1     Running   0          10m    192.168.137.130   k8s-master01           <none>           <none>
```

### 5.再次查看集群节点的状态

```
[root@k8s-master ~]# kubectl get nodes
NAME                   STATUS   ROLES                  AGE    VERSION
k8s-master01           Ready    control-plane,master   130m   v1.20.7
k8s-node01.linux.com   Ready    <none>                 124m   v1.20.7
k8s-node02.linux.com   Ready    <none>                 123m   v1.20.7
```

## 十一、安装metric插件

- 搜集、监控node节点CPU、内存使用情况 

### 1.将front-proxy-ca.crt证书拷贝到所有node节点

```
[root@k8s-master ~]# scp /etc/kubernetes/pki/front-proxy-ca.crt root@192.168.137.129:/etc/kubernetes/pki/front-proxy-ca.crt	#拷贝到node1节点
front-proxy-ca.crt                                                                                                                       100% 1078   754.4KB/s   00:00    
[root@k8s-master ~]# scp /etc/kubernetes/pki/front-proxy-ca.crt root@192.168.137.128:/etc/kubernetes/pki/front-proxy-ca.crt	#拷贝到node2节点
front-proxy-ca.crt  
```

### 2.部署Metric插件

```
vim comp.yaml
```

复制以下内容

```
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    k8s-app: metrics-server
    rbac.authorization.k8s.io/aggregate-to-admin: "true"
    rbac.authorization.k8s.io/aggregate-to-edit: "true"
    rbac.authorization.k8s.io/aggregate-to-view: "true"
  name: system:aggregated-metrics-reader
rules:
  - apiGroups:
      - metrics.k8s.io
    resources:
      - pods
      - nodes
    verbs:
      - get
      - list
      - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    k8s-app: metrics-server
  name: system:metrics-server
rules:
  - apiGroups:
      - ""
    resources:
      - pods
      - nodes
      - nodes/stats
      - namespaces
      - configmaps
    verbs:
      - get
      - list
      - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server-auth-reader
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: extension-apiserver-authentication-reader
subjects:
  - kind: ServiceAccount
    name: metrics-server
    namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server:system:auth-delegator
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:auth-delegator
subjects:
  - kind: ServiceAccount
    name: metrics-server
    namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: system:metrics-server
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:metrics-server
subjects:
  - kind: ServiceAccount
    name: metrics-server
    namespace: kube-system
---
apiVersion: v1
kind: Service
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
spec:
  ports:
    - name: https
      port: 443
      protocol: TCP
      targetPort: https
  selector:
    k8s-app: metrics-server
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
spec:
  selector:
    matchLabels:
      k8s-app: metrics-server
  strategy:
    rollingUpdate:
      maxUnavailable: 0
  template:
    metadata:
      labels:
        k8s-app: metrics-server
    spec:
      containers:
        - args:
            - --cert-dir=/tmp
            - --secure-port=4443
            - --metric-resolution=30s
            - --kubelet-insecure-tls
            - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
            - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt # change to front-proxy-ca.crt for kubeadm
            - --requestheader-username-headers=X-Remote-User
            - --requestheader-group-headers=X-Remote-Group
            - --requestheader-extra-headers-prefix=X-Remote-Extra-
          image: registry.cn-beijing.aliyuncs.com/dotbalo/metrics-server:v0.4.1
          imagePullPolicy: IfNotPresent
          livenessProbe:
            failureThreshold: 3
            httpGet:
              path: /livez
              port: https
              scheme: HTTPS
            periodSeconds: 10
          name: metrics-server
          ports:
            - containerPort: 4443
              name: https
              protocol: TCP
          readinessProbe:
            failureThreshold: 3
            httpGet:
              path: /readyz
              port: https
              scheme: HTTPS
            periodSeconds: 10
          securityContext:
            readOnlyRootFilesystem: true
            runAsNonRoot: true
            runAsUser: 1000
          volumeMounts:
            - mountPath: /tmp
              name: tmp-dir
            - name: ca-ssl
              mountPath: /etc/kubernetes/pki
      nodeSelector:
        kubernetes.io/os: linux
      priorityClassName: system-cluster-critical
      serviceAccountName: metrics-server
      volumes:
        - emptyDir: {}
          name: tmp-dir
        - name: ca-ssl
          hostPath:
            path: /etc/kubernetes/pki
---
apiVersion: apiregistration.k8s.io/v1
kind: APIService
metadata:
  labels:
    k8s-app: metrics-server
  name: v1beta1.metrics.k8s.io
spec:
  group: metrics.k8s.io
  groupPriorityMinimum: 100
  insecureSkipTLSVerify: true
  service:
    name: metrics-server
    namespace: kube-system
  version: v1beta1
  versionPriority: 100
```

：wq保存

```
[root@k8s-master ~]# kubectl create -f comp.yaml
```

### 3.测试Metric插件

可以等待1分钟左右再测试

```
[root@k8s-master ~]# kubectl top nodes 
NAME                   CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%     
k8s-node02.linux.com   154m         3%     834Mi           22%
```

## 十二、部署dashboard插件

- 提供web UI界面

涉及到的包[下载链接](https://wangshengjj.work/s/wRSz)

### 1.部署dashboard

```
[root@k8s-master ~]# cd dashboard/	#网盘提供下载
[root@k8s-master dashboard]# kubectl create -f ./	#里面有2个yaml格式的文件
```

### 2.将dashboard服务类型修改为nodePort(默认ClusterIP模式，其他机器不能访问)

```
[root@k8s-master dashboard]# kubectl edit svc kubernetes-dashboard -n kubernetes-dashboard
```

```
      1 # Please edit the object below. Lines beginning with a '#' will be ignored,
      2 # and an empty file will abort the edit. If an error occurs while saving this file         will be
      3 # reopened with the relevant failures.
      4 #
      5 apiVersion: v1
      6 kind: Service
      7 metadata:
      8   creationTimestamp: "2022-11-20T07:28:30Z"
      9   labels:
     10     k8s-app: kubernetes-dashboard
     11   name: kubernetes-dashboard
     12   namespace: kubernetes-dashboard
     13   resourceVersion: "6565"
     14   uid: de981c75-46e1-4721-b633-f64e2d7511f2
     15 spec:
     16   clusterIP: 172.16.63.54
     17   clusterIPs:
     18   - 172.16.63.54 
     19   externalTrafficPolicy: Cluster
     20   ports:
     21   - nodePort: 31077
     22     port: 443
     23     protocol: TCP
     24     targetPort: 8443
     25   selector:
     26     k8s-app: kubernetes-dashboard
     27   sessionAffinity: None
     28   type: NodePort	#把这里默认的ClusterIP改成NodePort
     29 status: 
     30   loadBalancer: {}
```

### 3.验证修改结果

```
[root@k8s-master dashboard]# kubectl get service -n kubernetes-dashboard
NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE
dashboard-metrics-scraper   ClusterIP   172.16.188.225   <none>        8000/TCP        8m44s
kubernetes-dashboard        NodePort    172.16.1.18      <none>        443:32205/TCP   8m45s	#注意这里端口443映射到了32205端口
```

### 4.查看dashboard容器运行的主机 

```
[root@k8s-master dashboard]# kubectl get pods -n kubernetes-dashboard -o wide 
NAME                                         READY   STATUS    RESTARTS   AGE   IP                NODE                   NOMINATED NODE   READINESS GATES
dashboard-metrics-scraper-7645f69d8c-hxz8n   1/1     Running   0          11m   192.168.242.130   k8s-node02.linux.com   <none>           <none>
kubernetes-dashboard-78cb679857-8hdnc        1/1     Running   0          11m   192.168.201.193   k8s-node01.linux.com   <none>           <none>	#通过这里我们知道了dashboard服务跑在了node1机器上
```

浏览器访问node1的IP+32205端口即可访问：https://192.168.137.129:32205	#每个人的IP不一样仅供参考

### 5.获取token令牌，登录dashboard

![k8s1](https://www.wangshengjj.work/upload/2022/11/k8s1.png)

```
[root@k8s-master dashboard]# kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
```

```
Name:         admin-user-token-crx2f
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: admin-user
              kubernetes.io/service-account.uid: 289b1d24-93ec-4af0-abdc-99d51dafa133

Type:  kubernetes.io/service-account-token

Data
====
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IlBUNDdadjNDQ3hPbWZ1enlNMGZCU09SNlpZOW9GdkIxckI1LWdWclIwUTgifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLWNyeDJmIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIyODliMWQyNC05M2VjLTRhZjAtYWJkYy05OWQ1MWRhZmExMzMiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06YWRtaW4tdXNlciJ9.FuU7y1LTqlEMSLFMkilOS2-9Q6uaoSYSvn7hr2aS4vGN5CvIeXbWRr-SJKmMTsDGr3ZfQcPc06ixFN1IafSAXZ0Ao6V6dPpuY37DmJ4Uv8Kvinrg1gRZeVSjpeFsTa9cXWod6tDDI7zxEF7byimkGTwXPjgGui2eRwFObu7UzdhAyNZMbALhKM3ot36Acbt8kQoZgkPeZLrbsuy--Qd1tdUH3rirvNEI9v_YDUYx_o5NxMM5OHvrtWConVtenRBYmIllsV4gx_-KQHFwWvx8IAtV4fQFMp5E-hcjOcxhIXkjnUPzr1BlhV68H7yZF2YYkama4y7EKPI6E1hlBYPcHA
#上面的token是我们需要复制到浏览器的！
ca.crt:     1066 bytes
```

### 6.登录成功

把刚刚获取到的token复制进去即可

![k8s2](https://www.wangshengjj.work/upload/2022/11/k8s2.png)

教程未完结，等待补充......
