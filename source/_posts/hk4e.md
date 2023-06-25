---
title: 基于Kubernetes环境搭建hk4e原神真端教程
date: 2023-06-24 10:12:50.108
updated: 2023-06-25 10:45:51.495
sticky: 5
categories: 
- 服务器搭建
- k8s
- 集群
- 容器
- 应用
tags: 
- 原神私服
- 原神
- k8s
- kubernetes
- hk4e
- 原神真端
---

# 基于`Kubernetes`环境搭建`hk4e`原神真端教程

>**仅供学习交流使用，如果侵犯到你的合法权利，请联系邮件删除，或评论。我将会在24h内删除。**
>**教程用到的文件下载链接：https://pan.baidu.com/s/1pJTUu1KjyAU7duIvqYwD-Q?pwd=6zv9**

## 一、环境介绍

|主机名|IP地址|软件|
|-------|-------|-------|
|master|192.168.31.200|kubernetes1.20.7、Docker-ce19.03|
|node01|192.168.31.201|kubernetes1.20.7、Docker-ce19.03|
|node02|192.168.31.203|kubernetes1.20.7、Docker-ce19.03|
|NFS|192.168.31.202|NFS网络存储|

>**本教程基于`hk4e`真端`3.2`版本，把实体机部署转到容器环境。**
>**`Kubernetes`简称`K8s`，以下教程部分将以`K8s`为称**

- 关于`K8s`环境介绍：
- `k8s`是一个容器编排工具，它由`Google`公司开源，由`Go`语言编写，它的作用类似于`Docker-compose`，它和`Docker-compose`最大的区别在于，`k8s`可以管理多台机器，而`Docker-compose`只能管理`1`台。
- `K8s`拥有服务发现和负载均衡的功能，同时`K8s`也支持存储编排功能
- `K8s`可以实现资源调度的分配，自动部署更新和回滚。
- `K8s`支持副本集功能，可以实现容器的自我修复
- 更多关于`K8s`的介绍，请观看这期教程[【容器应用系列教程】Kubernetes介绍](https://www.wsjj.top/archives/137)

## 二、部署`Kubernetes`环境

>**部署过程省略**
>**关于`K8s`搭建教程：https://www.wsjj.top/archives/138**

### 1.修改`Kube-apiserver`的端口范围

>**此操作，需要部署完`K8s`集群后再做修改！！！**
>**默认端口范围在`30001-32767`**

```
[root@master ~]# vim /etc/kubernetes/manifests/kube-apiserver.yaml
#配置文件并不完整，仅展示修改部分
containers:
  - command:
    - kube-apiserver
    - --service-node-port-range=20000-32767	#指定NodePort端口范围，需要和下面的其他参数对其
```

>**修改完配置文件后，`k8s`集群会自动重启，请等待重启完毕**

```
[root@master ~]# kubectl get pods -A -o wide
NAMESPACE       NAME                                        READY   STATUS      RESTARTS   AGE   IP               NODE     NOMINATED NODE   READINESS GATES
kube-system     calico-kube-controllers-5f6d4b864b-5vdvx    1/1     Running     0          42m   192.168.31.201   node01   <none>           <none>
kube-system     calico-node-4rspn                           1/1     Running     0          42m   192.168.31.200   master   <none>           <none>
kube-system     calico-node-smvkk                           1/1     Running     1          42m   192.168.31.201   node01   <none>           <none>
kube-system     calico-node-xtm4s                           1/1     Running     0          42m   192.168.31.203   node02   <none>           <none>
kube-system     coredns-54d67798b7-5qx8t                    1/1     Running     0          51m   192.168.140.66   node02   <none>           <none>
kube-system     coredns-54d67798b7-ntffw                    1/1     Running     0          51m   192.168.140.65   node02   <none>           <none>
kube-system     etcd-master                                 1/1     Running     0          51m   192.168.31.200   master   <none>           <none>
kube-system     kube-apiserver-master                       1/1     Running     0          51m   192.168.31.200   master   <none>           <none>
kube-system     kube-controller-manager-master              1/1     Running     0          51m   192.168.31.200   master   <none>           <none>
kube-system     kube-proxy-7v68r                            1/1     R
```

### 2.部署`nginx-ingress`插件

>**这个插件主要用于基于服务名的发布，发布我们`sdk`服务器的连接地址**
>**如果不需要部署这个插件，那么一会就需要修改`sdkserver`的端口类型为`NodePort`，直接端口映射到物理机的端口上**
>**教程用到的文件下载链接：https://pan.baidu.com/s/1pJTUu1KjyAU7duIvqYwD-Q?pwd=6zv9**

#### 准备`yml`文件

![k8s-hk4e02](https://www.wsjj.top/upload/2023/06/k8s-hk4e02.png)

#### 准备镜像

>**这一步需要再`2`个`node`节点上操作**
>**如果你的网络条件良好，这一步可以略过
>如果你的网络条件较差，我在镜像包里，准备了用到的镜像文件**

![k8s-hk4e04](https://www.wsjj.top/upload/2023/06/k8s-hk4e04.png)

```
[root@node01 ~]# docker load -i nginx-ingress-controller_v1.3.1.tar
[root@node01 ~]# docker load -i kube-webhook-certgen_v1.3.0.tar

[root@node02 ~]# docker load -i nginx-ingress-controller_v1.3.1.tar
[root@node02 ~]# docker load -i kube-webhook-certgen_v1.3.0.tar
```

#### 回到`master`节点，部署`nginx-ingress`

```
[root@master ~]# kubectl create -f ingress-nginx-deploy_v1.3.1.yaml
```

#### 部署完，可查看所有`Pod`状态

```
[root@master ~]# kubectl get pods -A -o wide
NAMESPACE       NAME                                        READY   STATUS      RESTARTS   AGE   IP               NODE     NOMINATED NODE   READINESS GATES
ingress-nginx   ingress-nginx-admission-create-ld8d2        0/1     Completed   0          45s   192.168.140.70   node02   <none>           <none>
ingress-nginx   ingress-nginx-admission-patch-9b2lx         0/1     Completed   1          45s   192.168.140.69   node02   <none>           <none>
ingress-nginx   ingress-nginx-controller-64ff7cd7cf-hz6ps   1/1     Running     0          45s   192.168.31.203   node02   <none>           <none>
kube-system     calico-kube-controllers-5f6d4b864b-5vdvx    1/1     Running     0          42m   192.168.31.201   node01   <none>           <none>
kube-system     calico-node-4rspn                           1/1     Running     0          42m   192.168.31.200   master   <none>           <none>
kube-system     calico-node-smvkk                           1/1     Running     1          42m   192.168.31.201   node01   <none>           <none>
kube-system     calico-node-xtm4s                           1/1     Running     0          42m   192.168.31.203   node02   <none>           <none>
kube-system     coredns-54d67798b7-5qx8t                    1/1     Running     0          51m   192.168.140.66   node02   <none>           <none>
kube-system     coredns-54d67798b7-ntffw                    1/1     Running     0          51m   192.168.140.65   node02   <none>           <none>
kube-system     etcd-master                                 1/1     Running     0          51m   192.168.31.200   master   <none>           <none>
kube-system     kube-apiserver-master                       1/1     Running     0          51m   192.168.31.200   master   <none>           <none>
kube-system     kube-controller-manager-master              1/1     Running     0          51m   192.168.31.200   master   <none>           <none>
kube-system     kube-proxy-7v68r                            1/1     R
```

## 三、准备`NFS`网络存储共享

>**`NFS`搭建过程省略，请自行准备下面几个挂载点。**
>**关于`NFS`搭建教程：https://www.wsjj.top/archives/62**
>**`NFS`比较重要，负责存储真端的所有数据，还有后续数据库做持久存储的重要存储媒介**

```
[root@nfs ~]# exportfs -rav
exporting 192.168.31.0/24:/hk4e_data
exporting 192.168.31.0/24:/hk4e_mongodb
exporting 192.168.31.0/24:/hk4e_redis
exporting 192.168.31.0/24:/hk4e_slave_data
exporting 192.168.31.0/24:/hk4e_master_data
```

### 1.准备数据文件

>**教程用到的文件下载地址：https://pan.baidu.com/s/1pJTUu1KjyAU7duIvqYwD-Q?pwd=6zv9**

![k8s-hk4e01](https://www.wsjj.top/upload/2023/06/k8s-hk4e01.png)

>**把压缩包里的所有目录个文件，拷贝到`hk4e_data`目录下**

```
[root@nfs ~]# cd /hk4e_data
[root@nfs hk4e_data]# ls
hk4e-gameserver_data.zip

[root@nfs hk4e_data]# unzip hk4e-gameserver_data.zip
[root@nfs hk4e_data]# mv hk4e-gameserver_data/* ./
[root@nfs hk4e_data]# chmod a+x ./
[root@nfs hk4e_data]# rm -rf hk4e-gameserver_data.zip
```

## 四、准备搭建用的镜像

>**我们的`k8s`使用`Docker`作为容器引擎**
>**需要在所有`node`节点导入教程用到的镜像**

![k8s-hk4e03](https://www.wsjj.top/upload/2023/06/k8s-hk4e03.png)

### 1.导入镜像

```
[root@node02 ~]# docker load -i tar包名
```

### 2.关于`hk4e-sdkserver-ubuntu`镜像和`hk4e-gameserver-ubuntu`镜像(可选的)

>**如果您出现了导入失败的情况**
>**我在文件里准备了`Docker build`用到的所有文件，可以自己手动在`node`节点上构建镜像**
>**注意：导出的镜像名和标签，建议跟我一样**
>**如果不同，则需要修改`hk4e-sdkserver.yml`和`hk4e-gameserver.yml`的`image`后面的镜像名**

![k8s-hk4e05](https://www.wsjj.top/upload/2023/06/k8s-hk4e05.png)

![k8s-hk4e06](https://www.wsjj.top/upload/2023/06/k8s-hk4e06.png)

```
[root@node02 Docker build]# ls
Dockerfile  jdk-17_linux-x64_bin.tar.gz

[root@node02 Docker build]# docker build -t hk4e-sdkserver-ubuntu:v0.2 ./
[+] Building 15.7s (9/9) FINISHED                                                   
 => [internal] load build definition from Dockerfile                           0.0s
 => => transferring dockerfile: 456B                                           0.0s
 => [internal] load .dockerignore                                              0.0s
 => => transferring context: 2B                                                0.0s
 => [internal] load metadata for docker.io/library/ubuntu:20.04                0.0s
 => [internal] load build context                                              2.0s
 => => transferring context: 181.75MB                                          2.0s
 => [1/4] FROM docker.io/library/ubuntu:20.04                                  0.0s
 => => resolve docker.io/library/ubuntu:20.04                                  0.0s
 => [2/4] WORKDIR /sdkserver                                                   0.1s
 => [3/4] RUN sed -i s@/archive.ubuntu.com/@/mirrors.aliyun.com/@g /etc/apt/  10.9s
 => [4/4] ADD jdk-17_linux-x64_bin.tar.gz /usr/local/                          3.5s
 => exporting to image                                                         1.0s 
 => => exporting layers                                                        1.0s 
 => => writing image sha256:802940fa7aff0ccc8b3ab56018d3361d4a29c32a30ab09d02  0.0s 
 => => naming to docker.io/library/hk4e-sdkserver-ubuntu:v0.2                  0.0s
```

```
[root@node02 Docker build]# docker build -t hk4e-gameserver-ubuntu:v0.2 ./
```

## 五、部署`hk4e`真端的数据库

### 1.创建命名空间(重要)

>**后续我们所有的服务和`Pod`，都在这个命名空间内部署**

```
[root@master ~]# kubectl create ns genshin
```

### 2.部署主从`MySQL`数据库

#### A.准备主数据库的`PV`和`PVC`

```
[root@master ~]# vim hk4e-master-pv-pvc.yml

apiVersion: v1
kind: PersistentVolume
metadata:
   name: hk4e-master-db-pv
   namespace: genshin	#指定命名空间
spec:
  capacity:
      storage: 10G	#PV卷大小，建议和NFS那边保持一致
  accessModes:
       - ReadWriteMany
  persistentVolumeReclaimPolicy: Recycle
  nfs:
      server: 192.168.31.202	#NFS服务器地址
      path: /hk4e_master_data	#NFS共享出来的目录名称
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
     name: hk4e-master-db-pvc
     namespace: genshin	#指定命名空间
spec:
     accessModes:
        - ReadWriteMany
     resources:
        requests:
            storage: 10G	#PVC卷大小，建议和NFS那边保持一致
```

```
[root@master ~]# kubectl create -f hk4e-master-pv-pvc.yml
```

#### B.准备从数据库的`PV`和`PVC`

```
[root@master ~]# vim hk4e-slave-pv-pvc.yml

apiVersion: v1
kind: PersistentVolume
metadata:
   name: hk4e-slave-db-pv
   namespace: genshin	#指定命名空间
spec:
  capacity:
      storage: 10G	#PV卷大小，建议和NFS那边保持一致
  accessModes:
       - ReadWriteMany
  persistentVolumeReclaimPolicy: Recycle
  nfs:
      server: 192.168.31.202	#NFS服务器地址
      path: /hk4e_slave_data	#NFS共享出来的目录名称
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
     name: hk4e-slave-db-pvc
     namespace: genshin	#指定命名空间
spec:
     accessModes:
        - ReadWriteMany
     resources:
        requests:
            storage: 10G	#PVC卷大小，建议和NFS那边保持一致
```

```
[root@master ~]# kubectl create -f hk4e-slave-pv-pvc.yml
```

##### 查看`PV`和`PVC`

```
[root@master hk4e-master-db-mysql_yaml]# kubectl get pv
NAME                CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                        STORAGECLASS   REASON   AGE
hk4e-master-db-pv   10G        RWX            Recycle          Bound    genshin/hk4e-master-db-pvc                           2m52s
hk4e-slave-db-pv    10G        RWX            Recycle          Bound    genshin/hk4e-slave-db-pvc                            8s

[root@master hk4e-master-db-mysql_yaml]# kubectl get pvc -n genshin
NAME                 STATUS   VOLUME              CAPACITY   ACCESS MODES   STORAGECLASS   AGE
hk4e-master-db-pvc   Bound    hk4e-master-db-pv   10G        RWX                           2m59s
hk4e-slave-db-pvc    Bound    hk4e-slave-db-pv    10G        RWX                           15s
```

#### C.创建主库用到的`ConfigMap`

```
[root@master ~]# vim hk4e-master-db-config.yml

apiVersion: v1
kind: ConfigMap
metadata:
    name: hk4e-master-db-config
    namespace: genshin
data:
    my.cnf: |
        [mysqld]
        server_id=10
        log_bin=master
        gtid_mode=on
        enforce_gtid_consistency=true
        collation-server = utf8_general_ci
        character-set-server = utf8
```

```
[root@master ~]# kubectl craete -f hk4e-master-db-config.yml
```

#### D.创建从库用到的`ConfigMap`

```
[root@master ~]# vim hk4e-slave-db-config.yml

apiVersion: v1
kind: ConfigMap
metadata:
    name: hk4e-slave-db-config
    namespace: genshin
data:
    my.cnf: |
        [mysqld]
        server_id=11
        log_bin=slave
        gtid_mode=on
        enforce_gtid_consistency=true
        collation-server = utf8_general_ci
        character-set-server = utf8
```

```
[root@master ~]# kubectl craete -f hk4e-slave-db-config.yml
```

##### 查看所有`ConfigMap`

```
[root@master hk4e-master-db-mysql_yaml]# kubectl get cm -n genshin
NAME                    DATA   AGE
hk4e-master-db-config   1      87s
hk4e-slave-db-config    1      28s
kube-root-ca.crt        1      32m
```

#### E.创建主库`Pod`

```
[root@master ~]# vim hk4e-master-db.yml

apiVersion: apps/v1
kind: StatefulSet	#有状态负载类型
metadata:
    name: hk4e-master-mysql
    namespace: genshin	#指定命名空间
spec:   
    replicas: 1	#指定保持1个副本集
    selector:
        matchLabels:
            app: hk4e-master-mysql  
    serviceName: "hk4e-master-mysql"	#指定下面的服务名
    template:
        metadata:
            labels:
                app: hk4e-master-mysql
        spec:  
            nodeName: node01	#指定调度到哪个节点上
            containers:
            - name: hk4e-master-mysql
              image: mysql:5.7	#镜像名
              imagePullPolicy: IfNotPresent	#拉取镜像策略
              ports:
              - containerPort: 3306
              env:	#传递环境变量
              - name: MYSQL_ROOT_PASSWORD
                value: "f2c340a9-bf06-4345-9654-00b074b92fe8"	#数据库root用户密码
              - name: MYSQL_USER
                value: "work"	#创建work用户
              - name: MYSQL_PASSWORD
                value: "GenshinImpactOffline2022"	#创建work用户的密码
              resources:   	#配置资源限制
                requests:
                  memory: "1G"
                  cpu: "1"
                limits:
                  memory: "2G"
                  cpu: "2"
              volumeMounts:
                  - name: mysql-data-volume	#挂载到数据目录
                    mountPath: /var/lib/mysql
                  - name: mysql-config-volume	#挂载配置文件
                    mountPath: /etc/mysql/conf.d
                    readOnly: true
            volumes: 
                - name: mysql-data-volume	#数据卷名
                  persistentVolumeClaim:	#挂载PVC
                    claimName: hk4e-master-db-pvc
                - name: mysql-config-volume	#数据卷名
                  projected:
                    sources:
                    - configMap:	#挂载cm
                        name: hk4e-master-db-config  
---
apiVersion: v1
kind: Service	#创建服务
metadata:
    name: hk4e-master-mysql	#服务名
    namespace: genshin	#命名空间
spec:
    ports:
    - port: 3306	#容器内部通信的端口
    selector:
      app: hk4e-master-mysql
```

```
[root@master ~]# kubectl create -f hk4e-master-db.yml
```

#### F.创建从库`Pod`

```
[root@master ~]# vim hk4e-slave-db.yml

apiVersion: apps/v1
kind: StatefulSet	#有状态负载类型
metadata:
    name: hk4e-slave-mysql
    namespace: genshin	#指定命名空间
spec:   
    replicas: 1	#指定保持1个副本集
    selector:
        matchLabels:
            app: hk4e-slave-mysql  
    serviceName: "hk4e-slave-mysql"	#指定下面的服务名
    template:
        metadata:
            labels:
                app: hk4e-slave-mysql
        spec:  
            nodeName: node01	#指定调度到哪个节点上
            containers:
            - name: hk4e-slave-mysql
              image: mysql:5.7	#镜像名
              imagePullPolicy: IfNotPresent	#拉取镜像策略
              ports:
              - containerPort: 3306
              env:	#传递环境变量
              - name: MYSQL_ROOT_PASSWORD
                value: "f2c340a9-bf06-4345-9654-00b074b92fe8"	#数据库root用户密码
              resources:   	#配置资源限制
                requests:
                  memory: "1G"
                  cpu: "1"
                limits:
                  memory: "1G"
                  cpu: "1"
              volumeMounts:
                  - name: mysql-data-volume	#挂载到数据目录
                    mountPath: /var/lib/mysql
                  - name: mysql-config-volume	#挂载配置文件
                    mountPath: /etc/mysql/conf.d
                    readOnly: true
            volumes: 
                - name: mysql-data-volume	#数据卷名
                  persistentVolumeClaim:	#挂载PVC
                    claimName: hk4e-slave-db-pvc
                - name: mysql-config-volume	#数据卷名
                  projected:
                    sources:
                    - configMap:	#挂载cm
                        name: hk4e-slave-db-config  
---
apiVersion: v1
kind: Service	#创建服务
metadata:
    name: hk4e-slave-mysql	#服务名
    namespace: genshin	#命名空间
spec:
    ports:
    - port: 3306	#容器内部通信的端口
    selector:
      app: hk4e-slave-mysql
```

```
[root@master ~]# kubectl create -f hk4e-slave-db.yml
```

##### 查看数据库`Pod`

```
[root@master hk4e-master-db-mysql_yaml]# kubectl get pod -n genshin
NAME                  READY   STATUS    RESTARTS   AGE
hk4e-master-mysql-0   1/1     Running   0          34m
hk4e-slave-mysql-0    1/1     Running   0          34m
```

##### 查看`NFS`服务器上的数据目录

```
[root@nfs ~]# ls /hk4e_master_data/
auto.cnf    client-cert.pem  ibdata1      ibtmp1         master.000003  mysql.sock          public_key.pem   sys
ca-key.pem  client-key.pem   ib_logfile0  master.000001  master.index   performance_schema  server-cert.pem
ca.pem      ib_buffer_pool   ib_logfile1  master.000002  mysql          private_key.pem     server-key.pem
[root@nfs ~]# ls /hk4e_slave_data/
auto.cnf    client-cert.pem  ibdata1      ibtmp1      performance_schema  server-cert.pem  slave.000002  sys
ca-key.pem  client-key.pem   ib_logfile0  mysql       private_key.pem     server-key.pem   slave.000003
ca.pem      ib_buffer_pool   ib_logfile1  mysql.sock  public_key.pem      slave.000001     slave.index
```

#### G.进入主库，配置主从复制的用户和`hk4e`的数据库

```
[root@master hk4e-master-db-mysql_yaml]# kubectl exec -it hk4e-master-mysql-0 -n genshin -- bash
```

```
bash-4.2# mysql -uroot -pf2c340a9-bf06-4345-9654-00b074b92fe8

mysql> grant replication slave on *.* to 'repluser'@'%' identified by 'WWW.1.com';
Query OK, 0 rows affected, 1 warning (0.11 sec)

mysql> GRANT ALL PRIVILEGES ON *.* TO 'work'@'127.0.0.1' IDENTIFIED BY 'GenshinImpactOffline2022' WITH GRANT OPTION;
Query OK, 0 rows affected, 1 warning (0.02 sec)

mysql> GRANT ALL PRIVILEGES ON *.* TO 'work'@'localhost' IDENTIFIED BY 'GenshinImpactOffline2022' WITH GRANT OPTION;
Query OK, 0 rows affected, 2 warnings (0.01 sec)

mysql> FLUSH   PRIVILEGES;
Query OK, 0 rows affected (0.04 sec)

mysql> quit
Bye
```

#### H.创建`hk4e`用到的库，和导入数据

![k8s-hk4e07](https://www.wsjj.top/upload/2023/06/k8s-hk4e07.png)

##### 先把用到的数据导入到`NFS`上

>**一定要创建完数据库后再导入，否则会报错。**
>**或者使用`kubectl cp`命令，拷贝文件到容器内部，用法和`docker cp`一样**

```
[root@nfs hk4e_master_data]# ls | grep mysql_sql
mysql_sql
```

##### 回到容器内部执行脚本，一键导入数据库

```
bash-4.2# bash /var/lib/mysql/mysql_sql/sql.sh
bash-4.2# rm -rf /var/lib/mysql/mysql_sql
bash-4.2# exit	#退出容器
exit
[root@master hk4e-master-db-mysql_yaml]#
```

#### I.进入从库，配置主从复制环境

```
[root@master hk4e-master-db-mysql_yaml]# kubectl exec -it hk4e-slave-mysql-0 -n genshin -- bash
bash-4.2# mysql -uroot -pf2c340a9-bf06-4345-9654-00b074b92fe8
```

```
mysql> change master to
    -> master_host="hk4e-master-mysql",
    -> master_user="repluser",
    -> master_password="WWW.1.com",
    -> master_auto_position=1;
Query OK, 0 rows affected, 2 warnings (0.10 sec)

mysql> start slave;
Query OK, 0 rows affected (0.00 sec)
```

##### 查看主从复制状态

```
mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: hk4e-master-mysql
                  Master_User: repluser
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: master.000003
          Read_Master_Log_Pos: 136422
               Relay_Log_File: hk4e-slave-mysql-0-relay-bin.000003
                Relay_Log_Pos: 136629
        Relay_Master_Log_File: master.000003
             Slave_IO_Running: Yes	#IO线程正常
            Slave_SQL_Running: Yes	#SQL线程正常
            
mysql> exit
Bye
bash-4.2# exit
exit
```

### 3.部署`redis`

#### A.创建`redis`的`PV`和`PVC`

```
[root@master hk4e-redis_yaml]# vim hk4e-redis-pv-pvc.yml

apiVersion: v1
kind: PersistentVolume
metadata:
   name: hk4e-redis-db-pv
   namespace: genshin
spec:
  capacity:
      storage: 10G
  accessModes:
       - ReadWriteMany
  persistentVolumeReclaimPolicy: Recycle
  nfs:
      server: 192.168.31.202
      path: /hk4e_redis
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
     name: hk4e-redis-db-pvc
     namespace: genshin
spec:
     accessModes:
        - ReadWriteMany
     resources:
        requests:
            storage: 10G
```

```
[root@master hk4e-redis_yaml]# kubectl create -f hk4e-redis-pv-pvc.yml
```

```
[root@master hk4e-redis_yaml]# kubectl get pv 
NAME                CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                        STORAGECLASS   REASON   AGE
hk4e-master-db-pv   10G        RWX            Recycle          Bound       genshin/hk4e-master-db-pvc                           85m
hk4e-redis-db-pv    10G        RWX            Recycle          Available                                                        8s
hk4e-slave-db-pv    10G        RWX            Recycle          Bound       genshin/hk4e-slave-db-pvc                            82m
[root@master hk4e-redis_yaml]# kubectl get pvc -n genshin
NAME                 STATUS   VOLUME              CAPACITY   ACCESS MODES   STORAGECLASS   AGE
hk4e-master-db-pvc   Bound    hk4e-master-db-pv   10G        RWX                           85m
hk4e-redis-db-pvc    Bound    hk4e-redis-db-pv    10G        RWX                           14s
hk4e-slave-db-pvc    Bound    hk4e-slave-db-pv    10G        RWX                           82m
```

#### B.部署`redis`用到的`ConfigMap`

```
[root@master hk4e-redis_yaml]# vim hk4e-redis-config.yml

apiVersion: v1
kind: ConfigMap
metadata:
    name: hk4e-redis-config
    namespace: genshin
data:
    redis.conf: |
        dir /srv/redis
        bind 0.0.0.0
        appendonly yes
        protected-mode yes
        port 6379
        tcp-backlog 800
        timeout 0
        tcp-keepalive 300
        daemonize no
        supervised no
        pidfile /srv/redis/redis-server.pid
        loglevel warning
        logfile /srv/redis/redis-server.log
        databases 168
        always-show-logo yes
        save 900 1
        save 300 10
        save 60 10000
        requirepass GenshinImpactOffline2022
```

```
[root@master hk4e-redis_yaml]# kubectl create -f hk4e-redis-config.yml
configmap/hk4e-redis-config created
[root@master hk4e-redis_yaml]# kubectl get cm -n genshin
NAME                    DATA   AGE
hk4e-master-db-config   1      82m
hk4e-redis-config       1      9s
hk4e-slave-db-config    1      81m
kube-root-ca.crt        1      114m
```

#### C.部署`Redis`的`Pod`

```
[root@master hk4e-redis_yaml]# vim hk4e-redis.yml 

apiVersion: apps/v1
kind: StatefulSet       #有状态负载
metadata:
  name: hk4e-redis
  namespace: genshin
  labels:
    app: hk4e-redis
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hk4e-redis
  serviceName: "hk4e-redis"
  template:
    metadata:
      labels:
        app: hk4e-redis
    spec:
      nodeName: node01	#指定调度到哪个节点上
      containers:
      - name: hk4e-redis
        image: redis
        command:
          - "sh"
          - "-c"
          - "redis-server /usr/local/etc/redis/redis.conf"
        ports:
        - containerPort: 6379
        resources:
          limits:	#资源限制
            cpu: 1000m
            memory: 1024Mi
          requests:
            cpu: 1000m
            memory: 1024Mi
        livenessProbe:
          tcpSocket:
            port: 6379
          initialDelaySeconds: 300
          timeoutSeconds: 1
          periodSeconds: 10
          successThreshold: 1
          failureThreshold: 3
        readinessProbe:
          tcpSocket:
            port: 6379
          initialDelaySeconds: 5
          timeoutSeconds: 1
          periodSeconds: 10
          successThreshold: 1
          failureThreshold: 3
        volumeMounts:
        - name: hk4e-redis-config
          mountPath:  /usr/local/etc/redis/redis.conf       #配置文件挂载到容器内
          subPath: redis.conf
        - name: hk4e-redis-data-volume
          mountPath: /srv/redis       #配置文件里指定的数据目录
      volumes:
      - name: hk4e-redis-config
        configMap:
          name: hk4e-redis-config    #挂载配置文件
      - name: hk4e-redis-data-volume
        persistentVolumeClaim:
            claimName: hk4e-redis-db-pvc     #挂载PVC
---
apiVersion: v1
kind: Service
metadata:
  name: hk4e-redis
  namespace: genshin
  labels:
    app: hk4e-redis
spec:
  ports:
    - name: tcp
      port: 6379
  selector:
    app: hk4e-redis
```

```
[root@master hk4e-redis_yaml]# kubectl create -f hk4e-redis.yml 
statefulset.apps/hk4e-redis created
service/hk4e-redis created
```

```
[root@master ~]# kubectl get pods -n genshin
NAME                  READY   STATUS    RESTARTS   AGE
hk4e-master-mysql-0   1/1     Running   0          65m
hk4e-redis-0          1/1     Running   0          91s
hk4e-slave-mysql-0    1/1     Running   0          65m
```

### 4.部署`MongoDB`

#### A.准备`MongoDB`的`PV`和`PVC`

```
[root@master hk4e-mongodb_yaml]# vim hk4e-mongodb-pv-pvc.yml

apiVersion: v1
kind: PersistentVolume
metadata:
   name: hk4e-mongodb-db-pv
   namespace: genshin
spec:
  capacity:
      storage: 10G
  accessModes:
       - ReadWriteMany
  persistentVolumeReclaimPolicy: Recycle
  nfs:
      server: 192.168.31.202
      path: /hk4e_mongodb
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
     name: hk4e-mongodb-db-pvc
     namespace: genshin
spec:
     accessModes:
        - ReadWriteMany
     resources:
        requests:
            storage: 10G
```

```
[root@master hk4e-mongodb_yaml]# kubectl create -f hk4e-mongodb-pv-pvc.yml 
persistentvolume/hk4e-mongodb-db-pv created
persistentvolumeclaim/hk4e-mongodb-db-pvc created

[root@master hk4e-mongodb_yaml]# kubectl get pv
NAME                 CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                         STORAGECLASS   REASON   AGE
hk4e-master-db-pv    10G        RWX            Recycle          Bound    genshin/hk4e-master-db-pvc                            92m
hk4e-mongodb-db-pv   10G        RWX            Recycle          Bound    genshin/hk4e-mongodb-db-pvc                           8s
hk4e-redis-db-pv     10G        RWX            Recycle          Bound    genshin/hk4e-redis-db-pvc                             7m21s
hk4e-slave-db-pv     10G        RWX            Recycle          Bound    genshin/hk4e-slave-db-pvc                             90m
[root@master hk4e-mongodb_yaml]# kubectl get pv -n genshin
NAME                 CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                         STORAGECLASS   REASON   AGE
hk4e-master-db-pv    10G        RWX            Recycle          Bound    genshin/hk4e-master-db-pvc                            92m
hk4e-mongodb-db-pv   10G        RWX            Recycle          Bound    genshin/hk4e-mongodb-db-pvc                           14s
hk4e-redis-db-pv     10G        RWX            Recycle          Bound    genshin/hk4e-redis-db-pvc                             7m27s
hk4e-slave-db-pv     10G        RWX            Recycle          Bound    genshin/hk4e-slave-db-pvc
```

#### B.创建`MongoDB`的`Pod`

```
[root@master hk4e-mongodb_yaml]# vim hk4e-mongodb.yml

apiVersion: apps/v1
kind: Deployment     
metadata:
    name: hk4e-mongodb
    namespace: genshin
spec:   
    replicas: 1
    selector:
        matchLabels:
            app: hk4e-mongodb 
    template:
        metadata:
            labels:
                app: hk4e-mongodb
        spec:  
            nodeName: node01	#指定调度到哪个节点上
            containers:
            - name: hk4e-mongodb
              image: mongo:4.0	#镜像
              imagePullPolicy: IfNotPresent
              ports:
              - containerPort: 27017
              resources:   
                requests:
                  memory: "1G"
                  cpu: "1"
                limits:
                  memory: "1G"
                  cpu: "1"
              volumeMounts:
                  - name: hk4e-mongodb-data
                    mountPath: /data/db
            volumes:
              - name: hk4e-mongodb-data
                persistentVolumeClaim:
                  claimName: hk4e-mongodb-db-pvc
---
apiVersion: v1
kind: Service
metadata:
    name: hk4e-mongodb
    namespace: genshin
spec:
    ports:
    - port: 27017
    selector:
      app: hk4e-mongodb
```

```
[root@master hk4e-mongodb_yaml]# kubectl create -f hk4e-mongodb.yml 
deployment.apps/hk4e-mongodb created
service/hk4e-mongodb created

[root@master hk4e-mongodb_yaml]# kubectl get pod -n genshin
NAME                           READY   STATUS    RESTARTS   AGE
hk4e-master-mysql-0            1/1     Running   0          76m
hk4e-mongodb-5d7fc87d9-b4ql5   1/1     Running   0          43s
hk4e-redis-0                   1/1     Running   0          12m
hk4e-slave-mysql-0             1/1     Running   0          76m
```

## 六、部署`hk4e`的`sdkserver`

### 1.准备`sdkserver`的`PV`和`PVC`

```
[root@master hk4e-sdkserver_yaml]# vim hk4e-sdkserver-pv-pvc.yml

apiVersion: v1
kind: PersistentVolume
metadata:
   name: hk4e-sdkserver-pv
   namespace: genshin
spec:
  capacity:
      storage: 10G
  accessModes:
       - ReadWriteMany
  persistentVolumeReclaimPolicy: Recycle
  nfs:
      server: 192.168.31.202
      path: /hk4e_data/sdkserver
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
     name: hk4e-sdkserver-pvc
     namespace: genshin
spec:
     accessModes:
        - ReadWriteMany
     resources:
        requests:
            storage: 10G
```

```
[root@master hk4e-sdkserver_yaml]# kubectl create -f hk4e-sdkserver-pv-pvc.yml
persistentvolume/hk4e-sdkserver-pv created
persistentvolumeclaim/hk4e-sdkserver-pvc created

[root@master hk4e-sdkserver_yaml]# kubectl get pv
NAME                 CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                         STORAGECLASS   REASON   AGE
hk4e-master-db-pv    10G        RWX            Recycle          Bound    genshin/hk4e-master-db-pvc                            108m
hk4e-mongodb-db-pv   10G        RWX            Recycle          Bound    genshin/hk4e-mongodb-db-pvc                           16m
hk4e-redis-db-pv     10G        RWX            Recycle          Bound    genshin/hk4e-redis-db-pvc                             23m
hk4e-sdkserver-pv    10G        RWX            Recycle          Bound    genshin/hk4e-sdkserver-pvc                            6s
hk4e-slave-db-pv     10G        RWX            Recycle          Bound    genshin/hk4e-slave-db-pvc

[root@master hk4e-sdkserver_yaml]# kubectl get pvc -n genshin
NAME                  STATUS   VOLUME               CAPACITY   ACCESS MODES   STORAGECLASS   AGE
hk4e-master-db-pvc    Bound    hk4e-master-db-pv    10G        RWX                           108m
hk4e-mongodb-db-pvc   Bound    hk4e-mongodb-db-pv   10G        RWX                           16m
hk4e-redis-db-pvc     Bound    hk4e-redis-db-pv     10G        RWX                           23m
hk4e-sdkserver-pvc    Bound    hk4e-sdkserver-pv    10G        RWX                           15s
hk4e-slave-db-pvc     Bound    hk4e-slave-db-pv     10G        RWX                           106m
```

### 2.修改`sdk`的`config,json`

>**前往`NFS`修改**

```
[root@nfs hk4e_data]# vim sdkserver/config.json

"dispatch": {
      "regions": [
        {
          "Name": "os_usa",
          "Title": "Grasscutter",
          "type": "DEV_PUBLIC",
          "DispatchUrl": "http://192.168.31.203:20001/query_cur_region"	#这个IP是后端node02节点的物理机IP，稍后我将把gameserver调度到node02身上
        }
      ],
      "logRequests": "NONE"
```

### 3.创建`sdkserver`的`pod`

```
[root@master hk4e-sdkserver_yaml]# vim hk4e-sdkserver.yml

apiVersion: apps/v1
kind: Deployment     
metadata:
    name: hk4e-sdkserver
    namespace: genshin
spec:   
    replicas: 1
    selector:
        matchLabels:
            app: hk4e-sdkserver 
    template:
        metadata:
            labels:
                app: hk4e-sdkserver
        spec:  
            nodeName: node02	#指定调度到哪个节点上
            containers:
            - name: hk4e-sdkserver
              image: hk4e-sdkserver-ubuntu:v0.2	#指定镜像
              imagePullPolicy: IfNotPresent
              # tty: true
              ports:
              - containerPort: 2888
              resources:   
                requests:	#做资源限制
                  memory: "1G"
                  cpu: "1"
                limits:
                  memory: "1G"
                  cpu: "1"
              volumeMounts:
                  - name: hk4e-sdkserver-data
                    mountPath: /sdkserver
            volumes:
              - name: hk4e-sdkserver-data
                persistentVolumeClaim:
                  claimName: hk4e-sdkserver-pvc
---
apiVersion: v1
kind: Service
metadata:
    name: hk4e-sdkserver
    namespace: genshin
spec:
    ports:
    - port: 2888
    selector:
      app: hk4e-sdkserver
```

```
[root@master hk4e-sdkserver_yaml]# kubectl create -f hk4e-sdkserver.yml
deployment.apps/hk4e-sdkserver created
service/hk4e-sdkserver created
[root@master hk4e-sdkserver_yaml]# kubectl get pod -n genshin
NAME                              READY   STATUS    RESTARTS   AGE
hk4e-master-mysql-0               1/1     Running   0          88m
hk4e-mongodb-5d7fc87d9-b4ql5      1/1     Running   0          12m
hk4e-redis-0                      1/1     Running   0          23m
hk4e-sdkserver-6dbb86f5cc-kv2lg   1/1     Running   0          28s
hk4e-slave-mysql-0                1/1     Running   0          88m
```

```
[root@master hk4e-sdkserver_yaml]# kubectl logs hk4e-sdkserver-6dbb86f5cc-kv2lg -n genshin
05:44:37 <ERROR:Crypto> An error occurred while loading keys.
java.lang.NullPointerException: Cannot invoke "java.util.List.iterator()" because the return value of "org.nofs.utils.FileUtils.getPathsFromResource(String)" is null
	at org.nofs.utils.Crypto.loadKeys(Crypto.java:40)
	at org.nofs.Grasscutter.main(Grasscutter.java:92)
05:44:37 <INFO:Grasscutter> Starting Grasscutter...
05:44:37 <INFO:Grasscutter> Game version: 3.2.0
05:44:37 <INFO:Grasscutter> Grasscutter version: 3.2.0-dev-01ce6a9
05:44:38 <INFO:HttpServer> [Dispatch] Dispatch server started at hk4e-sdkserver:2888
```

### 4.创建`Ingress`服务，发布`2888`端口

```
[root@master hk4e-sdkserver_yaml]# vim hk4e-sdkserver-ingress.yml

apiVersion: extensions/v1beta1
kind: Ingress 
metadata:
  name: hk4e-sdkserver-ingress
  namespace: genshin
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules: 
  - host: sdk.linux.com	#发布到哪个域名上
    http:
      paths: 
      - path:  
        backend: 
          serviceName: hk4e-sdkserver	#指定要发布服务的服务名
          servicePort: 2888	#被发布的端口
```

```
[root@master hk4e-sdkserver_yaml]# kubectl create -f hk4e-sdkserver-ingress.yml 
Warning: extensions/v1beta1 Ingress is deprecated in v1.14+, unavailable in v1.22+; use networking.k8s.io/v1 Ingress
ingress.extensions/hk4e-sdkserver-ingress created
[root@master hk4e-sdkserver_yaml]# kubectl get ingress -n genshin
NAME                     CLASS    HOSTS           ADDRESS   PORTS   AGE
hk4e-sdkserver-ingress   <none>   sdk.linux.com             80      14s
```

#### 查看`ingress`跑在哪台`node`节点上

>**如果是真实的服务器，直接域名解析到相应的公网`IP`上即可**

```
[root@master hk4e-sdkserver_yaml]# kubectl get pod -A -o wide
NAMESPACE       NAME                                        READY   STATUS      RESTARTS   AGE     IP                NODE     NOMINATED NODE   READINESS GATES
genshin         hk4e-master-mysql-0                         1/1     Running     0          95m     192.168.196.129   node01   <none>           <none>
genshin         hk4e-mongodb-5d7fc87d9-b4ql5                1/1     Running     0          19m     192.168.140.71    node02   <none>           <none>
genshin         hk4e-redis-0                                1/1     Running     0          31m     192.168.196.131   node01   <none>           <none>
genshin         hk4e-sdkserver-6dbb86f5cc-kv2lg             1/1     Running     0          7m56s   192.168.140.72    node02   <none>           <none>
genshin         hk4e-slave-mysql-0                          1/1     Running     0          95m     192.168.196.130   node01   <none>           <none>
ingress-nginx   ingress-nginx-controller-64ff7cd7cf-hz6ps   1/1     Running     0          162m    192.168.31.203    node02   <none>           <none>
#我这里跑在node02身上
```

#### 修改hosts文件(如果公网有域名，不用修改)

>`C:\Windows\System32\drivers\etc\hosts`

![k8s-hk4e08](https://www.wsjj.top/upload/2023/06/k8s-hk4e08.png)

![k8s-hk4e09](https://www.wsjj.top/upload/2023/06/k8s-hk4e09.png)

## 七、部署`hk4e`的`gameserver`

### 1.准备`gameserver`的`PV`和`PVC`

```
[root@master hk4e-gameserver_yaml]# vim hk4e-gameserver-pv-pvc.yml

apiVersion: v1
kind: PersistentVolume
metadata:
   name: hk4e-gameserver-pv
   namespace: genshin
spec:
  capacity:
      storage: 50G
  accessModes:
       - ReadWriteMany
  persistentVolumeReclaimPolicy: Recycle
  nfs:
      server: 192.168.31.202
      path: /hk4e_data
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
     name: hk4e-gameserver-pvc
     namespace: genshin
spec:
     accessModes:
        - ReadWriteMany
     resources:
        requests:
            storage: 50G
```

```
[root@master hk4e-gameserver_yaml]# kubectl create -f hk4e-gameserver-pv-pvc.yml 
persistentvolume/hk4e-gameserver-pv created
persistentvolumeclaim/hk4e-gameserver-pvc created

[root@master hk4e-gameserver_yaml]# kubectl get pv
NAME                 CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                         STORAGECLASS   REASON   AGE
hk4e-gameserver-pv   50G        RWX            Recycle          Bound    genshin/hk4e-gameserver-pvc                           7s
hk4e-master-db-pv    10G        RWX            Recycle          Bound    genshin/hk4e-master-db-pvc                            128m
hk4e-mongodb-db-pv   10G        RWX            Recycle          Bound    genshin/hk4e-mongodb-db-pvc                           36m
hk4e-redis-db-pv     10G        RWX            Recycle          Bound    genshin/hk4e-redis-db-pvc                             43m
hk4e-sdkserver-pv    10G        RWX            Recycle          Bound    genshin/hk4e-sdkserver-pvc                            19m
hk4e-slave-db-pv     10G        RWX            Recycle          Bound    genshin/hk4e-slave-db-pvc                             125m
[root@master hk4e-gameserver_yaml]# kubectl get pvc -n genshin
NAME                  STATUS   VOLUME               CAPACITY   ACCESS MODES   STORAGECLASS   AGE
hk4e-gameserver-pvc   Bound    hk4e-gameserver-pv   50G        RWX                           13s
hk4e-master-db-pvc    Bound    hk4e-master-db-pv    10G        RWX                           128m
hk4e-mongodb-db-pvc   Bound    hk4e-mongodb-db-pv   10G        RWX                           36m
hk4e-redis-db-pvc     Bound    hk4e-redis-db-pv     10G        RWX                           43m
hk4e-sdkserver-pvc    Bound    hk4e-sdkserver-pv    10G        RWX                           20m
hk4e-slave-db-pvc     Bound    hk4e-slave-db-pv     10G        RWX                           126m
```

### 2.修改`hk4e`的运行地址

>**前往`NFS`服务器上`hk4e_data`目录操作**
>**如果是内网游玩，`IP`地址都填写写本机`IP`的内网地址即可**

```
find ./ -name "*.xml"  -exec sed -i "s/REPLACE_IT_TO_YOUR_DEVICE_IP/你的内网IP/g" {} \;

find . -name "*.xml"  -exec sed -i "s/REPLACE_IT_TO_YOUR_ACCESS_IP/你的公网IP/g" {} \;

find . -name "*.php"  -exec sed -i "s/123.123.123.123/你的公网IP/g" {} \;
```

### 3.部署`gameserver`的`pod`

```
[root@master hk4e-gameserver_yaml]# vim hk4e-gameserver.yml

apiVersion: apps/v1
kind: Deployment     
metadata:
    name: hk4e-gameserver
    namespace: genshin
spec:   
    replicas: 1
    selector:
        matchLabels:
            app: hk4e-gameserver 
    template:
        metadata:
            labels:
                app: hk4e-gameserver
        spec:  
            nodeName: node02	#指定调度到哪个节点上
            # hostNetwork: true
            containers:
            - name: hk4e-gameserver
              image: hk4e-gameserver-ubuntu:v0.2	#自定义镜像
              imagePullPolicy: IfNotPresent
              # tty: true
              command:	#启动容器的命令
                - "/bin/sh"
                - "-c"
                - "/gameserver/startserver.sh"
              resources:   #配置资源限制
                requests:
                  memory: "16G"
                  cpu: "8"
                limits:
                  memory: "20G"
                  cpu: "12"
              volumeMounts:
                  - name: hk4e-gameserver-data
                    mountPath: /gameserver
            volumes:
              - name: hk4e-gameserver-data
                persistentVolumeClaim:
                  claimName: hk4e-gameserver-pvc
---
apiVersion: v1
kind: Service
metadata:
    name: hk4e-gameserver
    namespace: genshin
spec:
    type: NodePort	#用于发布宿主机身上的端口
    ports:
    - name: port-0-tcp
      port: 20001
      protocol: TCP
      nodePort: 20001
    - name: port-0-udp
      port: 20001
      protocol: UDP
      nodePort: 20001
    - name: port-1-tcp
      port: 20011
      protocol: TCP
      nodePort: 20011
    - name: port-1-udp
      port: 20011
      protocol: UDP
      nodePort: 20011
    - name: port-2-tcp
      port: 20021
      protocol: TCP
      nodePort: 20021
    - name: port-2-udp
      port: 20021
      protocol: UDP
      nodePort: 20021
    - name: port-3-tcp
      port: 20031
      protocol: TCP
      nodePort: 20031
    - name: port-3-udp
      port: 20031
      protocol: UDP
      nodePort: 20031
    - name: port-4-tcp
      port: 20041
      protocol: TCP
      nodePort: 20041
    - name: port-4-udp
      port: 20041
      protocol: UDP
      nodePort: 20041
    - name: port-5-tcp
      port: 20051
      protocol: TCP
      nodePort: 20051
    - name: port-5-udp
      port: 20051
      protocol: UDP
      nodePort: 20051
    - name: port-6-tcp
      port: 20061
      protocol: TCP
      nodePort: 20061
    - name: port-6-udp
      port: 20061
      protocol: UDP
      nodePort: 20061
    - name: port-7-tcp
      port: 20071
      protocol: TCP
      nodePort: 20071
    - name: port-7-udp
      port: 20071
      protocol: UDP
      nodePort: 20071
    - name: port-8-tcp
      port: 20081
      protocol: TCP
      nodePort: 20081
    - name: port-8-udp
      port: 20081
      protocol: UDP
      nodePort: 20081
    selector:
      app: hk4e-gameserver
```

```
[root@master hk4e-gameserver_yaml]# kubectl create -f hk4e-gameserver.yml

[root@master hk4e-gameserver_yaml]# kubectl get pods -n genshin
NAME                              READY   STATUS    RESTARTS   AGE
hk4e-gameserver-8dcdd6b47-j9wnt   1/1     Running   0          26s
hk4e-master-mysql-0               1/1     Running   0          110m
hk4e-mongodb-5d7fc87d9-b4ql5      1/1     Running   0          34m
hk4e-redis-0                      1/1     Running   0          46m
hk4e-sdkserver-6dbb86f5cc-kv2lg   1/1     Running   0          22m
hk4e-slave-mysql-0                1/1     Running   0          110m
```

### 4.查看服务

```
[root@master hk4e-gameserver_yaml]# kubectl get svc -n genshin
NAME                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                                                                                                                                                                                                                                                                                           AGE
hk4e-gameserver     NodePort    172.16.133.113   <none>        20001:20001/TCP,20001:20001/UDP,20011:20011/TCP,20011:20011/UDP,20021:20021/TCP,20021:20021/UDP,20031:20031/TCP,20031:20031/UDP,20041:20041/TCP,20041:20041/UDP,20051:20051/TCP,20051:20051/UDP,20061:20061/TCP,20061:20061/UDP,20071:20071/TCP,20071:20071/UDP,20081:20081/TCP,20081:20081/UDP   5m50s
hk4e-master-mysql   ClusterIP   172.16.242.133   <none>        3306/TCP                                                                                                                                                                                                                                                                                          114m
hk4e-mongodb        ClusterIP   172.16.153.180   <none>        27017/TCP                                                                                                                                                                                                                                                                                         40m
hk4e-redis          ClusterIP   172.16.177.173   <none>        6379/TCP                                                                                                                                                                                                                                                                                          49m
hk4e-sdkserver      ClusterIP   172.16.29.198    <none>        2888/TCP                                                                                                                                                                                                                                                                                          26m
hk4e-slave-mysql    ClusterIP   172.16.186.228   <none>        3306/TCP                                                                                                                                                                                                                                                                                          113m
```

## 八、登录游戏测试

![k8s-hk4e10](https://www.wsjj.top/upload/2023/06/k8s-hk4e10.png)

![k8s-hk4e11](https://www.wsjj.top/upload/2023/06/k8s-hk4e11.png)
