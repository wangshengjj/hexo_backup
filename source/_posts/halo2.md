---
title: 使用Docker容器环境部署halo2博客+MySQL
date: 2023-06-04 13:10:10.111
updated: 2023-06-04 13:23:00.203
categories: 
- 服务器搭建
- 我的水货
- 杂谈
- 技术
- 虚拟机
tags: 
- docker
- centos
- mysql
- 容器
- halo
- halo2
---

# 使用Docker容器环境部署halo2博客+MySQL

>**关于`Halo`博客官网：https://halo.run/**
>**本教程参考了`Halo`官方文档：https://docs.halo.run/getting-started/install/docker**

## 一、环境准备

>**由于本人公网服务器有限，所以拿虚拟机和局域网演示**

|主机名|IP|软件|
|-------|-------|-------|
|server.linux.com|192.168.140.10|Docker、halo2、MySQL|
|slave.linux.com|192.168.140.11|Docker、MySQL|

## 二、给两台服务器安装`Docker`

>**安装步骤省略**
>**关于Docker的安装教程：https://www.wsjj.top/archives/132**

## 三、准备`MySQL`主从复制环境

|主机名|IP|软件名|
|-------|-------|-------|
|server.linux.com|192.168.140.10|MySQL主库|
|slave.linux.com|192.168.140.11|MySQL从库|

### 1.准备`MySQL`配置文件和数据目录

>**如果想让`Docker`运行`MySQL`，并且开启二进制日志，是需要提前准备个配置文件**

```
[root@server ~]# mkdir -p /mysql_master/{data,etc}
[root@slave ~]# mkdir -p /mysql_slave/{data,etc}
```

```
[root@server ~]# vim /mysql_master/etc/my.cnf

[mysqld]
server_id=10
log_bin=master
gtid_mode=on
enforce_gtid_consistency=true
```

```
[root@slave ~]# vim /mysql_slave/etc/my.cnf

[mysqld]
server_id=11
log_bin=slave
gtid_mode=on
enforce_gtid_consistency=true
```

```
[root@server ~]# docker run -itd -e MYSQL_ROOT_PASSWORD="WWW.1.com" -v /mysql_master/data:/var/lib/mysql -v /mysql_master/etc/my.cnf:/etc/my.cnf --restart=always --name=masterDB -p 3306:3306 mysql:5.7
```

```
[root@slave ~]# docker run -itd -e MYSQL_ROOT_PASSWORD="WWW.1.com" -v /mysql_slave/data:/var/lib/mysql -v /mysql_slave/etc/my.cnf:/etc/my.cnf --restart=always --name=slaveDB -p 3306:3306 mysql:5.7
```

- `-it`模拟终端
- `-d`后台保持运行
- `-e`传递参数，用于指定`MySQL`内`root`用户的密码
- `-v`用于MySQL做数据永久保存
- `--restart`重启后自动启动容器
- `--name`给容器指定一个名字
- `-p`映射端口，用于从库连接和未来的主从切换

### 2.配置主从环境用户、准备`halo`库和用户

>**需要在主库操作！**
>**关于`MySQL`主从复制教程：https://www.wsjj.top/archives/76**

```
[root@server ~]# docker exec -it masterDB /bin/bash
bash-4.2# mysql -uroot -pWWW.1.com
```

```
mysql> grant replication slave on *.* to 'repluser'@'192.168.140.%' identified by 'WWW.
1.com';
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> create database halo2 charset utf8mb4;
Query OK, 1 row affected (0.00 sec)

mysql> grant all on halo2.* to 'halo2'@'%' identified by 'WWW.1.com';
Query OK, 0 rows affected, 1 warning (0.00 sec)
```

### 3.回到从库，配置主从复制

```
[root@slave ~]# docker exec -it slaveDB /bin/bash
bash-4.2# mysql -uroot -pWWW.1.com
```

```
mysql> change master to
    -> master_host="192.168.140.10",
    -> master_user="repluser",
    -> master_password="WWW.1.com",
    -> master_auto_position=1;
Query OK, 0 rows affected, 2 warnings (0.01 sec)

mysql> start slave;	#启动slave
Query OK, 0 rows affected (0.00 sec)

mysql> show slave status\G;	#查看状态
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.140.10
                  Master_User: repluser
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: master.000003
          Read_Master_Log_Pos: 964
               Relay_Log_File: d8ff05681692-relay-bin.000003
                Relay_Log_Pos: 1171
        Relay_Master_Log_File: master.000003
             Slave_IO_Running: Yes	#正常运行
            Slave_SQL_Running: Yes	#正常运行
```

## 四、安装halo2

### 1.创建数据目录

```
[root@server ~]# mkdir -p /root/.halo2
```

### 2.创建容器

>**`halo2`官方部署文档：https://docs.halo.run/getting-started/install/docker**

```
docker run \
  -it -d \
  --name halo \
  -p 8090:8090 \
  -v ~/.halo2:/root/.halo2 \
  --link=masterDB:masterdb \
  --restart=always \
  halohub/halo:2.6 \
  --halo.external-url=http://localhost:8090/ \
  --halo.security.initializer.superadminusername=admin \
  --halo.security.initializer.superadminpassword=password \
  --spring.r2dbc.url=r2dbc:pool:mysql://masterdb:3306/halo2 \
  --spring.r2dbc.username=halo2 \
  --spring.r2dbc.password=WWW.1.com
```

- `--link`=`容器名`:`容器别名`
	- 用于连接数据库
	- 当你指定了一个别名后，在下面的数据库连接地址，就可以写成容器别名，这样子`halo`就会以容器别名的方式连接其他容器
### 3.浏览器测试

![halo2_01](https://www.wsjj.top/upload/2023/06/halo2_01.png)

![halo2_02](https://www.wsjj.top/upload/2023/06/halo2_02.png)

>**到这里，我们的博客就算部署完啦！**

## 特别鸣谢

**[Halo文档](https://docs.halo.run/)**
