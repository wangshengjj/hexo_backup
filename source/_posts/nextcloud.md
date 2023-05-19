---
title: 私人网盘nextcloud搭建教程
date: 2022-05-10 19:31:35.0
updated: 2022-08-21 14:43:54.67
categories: 
- 私人网盘
tags: 
- 私人网盘 nextcloud
---



# 私人网盘搭建教程
## 声明：仅供学习交流使用，如果侵犯到你的合法权利，请联系邮件删除，或评论。我将会在24h内删除。
## 一、准备阶段

请确认您的Linux是Centos7版本，本教程兼容Centos7！！！

您的系统已经连上了网络（本文默认您已经连上了网络）

教程中所用到的文件下载链接：https://pan.baidu.com/s/1CbS5scJvpV0sv2PEsCpvDQ?pwd=d5zq 
提取码：d5zq

## 二、开始搭建

### 1.挂载阿里云yum源

```
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```

清理旧yum

```
yum clean all
```

打包新yum

```
yum makecache
```

### 2.解决一些依赖性

```
yum install -y epel-release yum-utils unzip curl wget \ bash-completion policycoreutils-python mlocate bzip2
```

### 3.安装Apache web服务

```
yum install -y httpd
```

### 4.编写必要的配置文件

**如果您的系统vim无法打开，请使用vi命令**

```
vim /etc/httpd/conf.d/nextcloud.conf
```

**以下是插入的内容**

```
<VirtualHost *:80>
  DocumentRoot /var/www/html/
  ServerName  your.server.com

<Directory "/var/www/html/">
  Require all granted
  AllowOverride All
  Options FollowSymLinks MultiViews
</Directory>
</VirtualHost>
```

### 5.设置web服务开机自启运行

```
systemctl enable httpd.service
```

```
systemctl start httpd.service
```

### 6.安装PHP模块

```
yum install -y centos-release-scl
```

```
yum install -y rh-php72 rh-php72-php rh-php72-php-gd rh-php72-php-mbstring \ rh-php72-php-intl rh-php72-php-pecl-apcu rh-php72-php-mysqlnd rh-php72-php-pecl-redis \ rh-php72-php-opcache rh-php72-php-imagick
```

### 7.创建一些必要的链接

```
ln -s /opt/rh/httpd24/root/etc/httpd/conf.d/rh-php72-php.conf /etc/httpd/conf.d/
```

```
ln -s /opt/rh/httpd24/root/etc/httpd/conf.modules.d/15-rh-php72-php.conf /etc/httpd/conf.modules.d/
```

```
ln -s /opt/rh/httpd24/root/etc/httpd/modules/librh-php72-php7.so /etc/httpd/modules/
```

```
ln -s /opt/rh/rh-php72/root/bin/php /usr/bin/php
```

### 8.安装数据库

```
yum install -y mariadb mariadb-server
```

### 9.设置数据库开机自启动，并且手动启动数据库

```
systemctl enable mariadb.service
```

```
systemctl start mariadb.service
```

## 三、开始部署私人网盘服务器

### 1.下载并解压nextcloud主程序文件

**文章首页已经提供了下载链接，如果您还是没有下载请跳转**

链接：https://pan.baidu.com/s/1CbS5scJvpV0sv2PEsCpvDQ?pwd=d5zq 
提取码：d5zq

本文建议您把下载好的2个文件（一个本体，一个MD5）**默认放在root家目录下**

#### 一.MD5检验文件完整性

```
md5sum -c nextcloud-15.0.5.zip.md5 < nextcloud-15.0.5.zip
```

#### 二.解压并且移动到web服务目录下

```
unzip nextcloud-15.0.5.zip
```

```
cd nextcloud
```

```
cp -R * /var/www/html/
```

#### 三.创建数据文件目录

```
mkdir /var/www/html/data
```

### 2.对目录设置权限（确保web可以正常访问）

```
chown -R apache:apache /var/www/html
```

#### 一.重启web服务

```
systemctl restart httpd.service
```

#### 二.设置防火墙

```
firewall-cmd --zone=public --add-service=http --permanent
```

```
firewall-cmd --reload
```

## 四、设置selinux安全上下文

```
semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/html/data(/.*)?'
```

```
semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/html/config(/.*)?'
```

```
semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/html/apps(/.*)?'
```

```
semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/html/.htaccess'
```

```
semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/html/.user.ini'
```

```
semanage fcontext -a -t httpd_sys_rw_content_t '/var/www/html/3rdparty/aws/aws-sdk-php/src/data/logs(/.*)?'
```

```
restorecon -R '/var/www/html/'
```

```
setsebool -P httpd_can_network_connect on
```

**到这一步，服务器就搭建成功了，可以用浏览器访问您服务器的IP地址开始配置了！**

## 五、拓展

给服务器添加信任IP，防止换一个IP地址访问服务器后提示**不受信任的链接**

```
vi /var/www/html/config/config.php
```

按**照格**式把您的第二个，或者第三个，或者您的域名添加进去即可

#### 添加完成后重启服务器

```
systemctl start httpd.service
```

**如果重启失败了，说明您的上一步添加信任IP操作不对**