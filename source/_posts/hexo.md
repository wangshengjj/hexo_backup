---
title: hexo博客搭建教程
date: 2022-06-23 14:27:15.146
updated: 2022-08-20 14:44:31.306
categories: 
- 服务器搭建
tags: 
- 服务器搭建
- hexo博客
---

# hexo博客搭建教程

**仅供学习交流使用，如果侵犯到你的合法权利，请联系邮件删除，或评论。我将会在24h内删除。**

## 一、选择服务器和域名

我这里用腾讯云服务器做演示

[服务器购买链接](https://cloud.tencent.com/act/pro/618season?channel=sp&fromSource=gwzcw.6633950.6633950.6633950&utm_medium=cpc&utm_id=gwzcw.6633950.6633950.6633950&bd_vid=11065626160413136653)

新用户可以选择45一年的轻量级服务器

![](https://www.wangshengjj.work/upload/2022/06/微信图片_20220613164726.png)

购买域名

[域名购买链接](https://cloud.tencent.com/act/domainsales?channel=sp&fromSource=gwzcw.6463875.6463875.6463875&utm_medium=cpc&utm_id=gwzcw.6463875.6463875.6463875&bd_vid=9026323095797005744)

这里推荐购买.top和.cc的域名（主要是次年续费便宜），可以根据自己网站的名称去选择注册

![](https://www.wangshengjj.work/upload/2022/06/微信图片_20220613165407.png)

## 二.开始部署服务器

来到腾讯云控制台

[点我跳转到腾讯云控制台](https://console.cloud.tencent.com/)

![](https://www.wangshengjj.work/upload/2022/06/微信图片_20220613165945.png)

#### 1.开始部署服务器

进入我们的服务器管理，选择“新建”后会跳转到新的页面（服务器创建过程中会慢一点，请耐心等待）

![](https://www.wangshengjj.work/upload/2022/06/微信图片_20220613174620.png)

![](https://www.wangshengjj.work/upload/2022/06/微信图片_20220623131212.png)

**设置防火墙规则（这一步非常重要！）**

我们需要开放8888端口和4000端口

![](https://www.wangshengjj.work/upload/2022/06/微信图片_20220623132815.png)

**1.安装宝塔面板**

我们部署好服务器后，先更改服务器密码

![](https://www.wangshengjj.work/upload/2022/06/微信图片_20220623131446.png)

然后使用Windows自带的远程桌面工具，连接我们的服务器

![](https://www.wangshengjj.work/upload/2022/06/微信图片_20220623131736.png)

如果提示输入账号和密码，请按以下输入

```
用户：Administrator
```

```
密码：刚才我们已经改好的密码
```

我们去宝塔官网下载面板（选择Windows版）

[宝塔官网](https://www.bt.cn/new/download.html)

~~安装步骤省略~~

浏览器访问外网管理地址即可

![](https://www.wangshengjj.work/upload/2022/06/微信图片_20220613171440.png)

选择安装Nginx套件

![微信图片_20220623162456](https://www.wangshengjj.work/upload/2022/06/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20220623162456.png)

#### 2.安装nodejs和git

[nodejs下载地址](http://nodejs.cn/download/)

[git下载地址](https://git-scm.com/download/win)

**下载打开后，一直点下一步安装即可**

在桌面创建一个data文件夹，里面再创建一个名为blog的文件夹

我们右键，选择git bash

![](https://www.wangshengjj.work/upload/2022/06/微信图片_20220623133922.png)

依次输入以下代码（检查环境和版本）

```
node -v
```

```
npm -v
```

```
git -version
```

![](https://www.wangshengjj.work/upload/2022/06/微信图片_20220623134129.png)

#### 3.开始部署hexo博客

npm下载慢的话，可以下载淘宝下载源cnpm

```
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

**1.安装hexo**

```
npm install hexo-cli -g
```

或者

```
cnpm install hexo-cli -g
```

安装完成后可以选择使用hexo -v查看版本信息

**2.初始化**

```
hexo init
```

![](https://www.wangshengjj.work/upload/2022/06/微信图片_20220623135134.png)

**3.启动博客**

生成

```
hexo g
```

启动

```
hexo s
```

停止

**Ctrl+C**

**4.测试**

浏览器访问**服务器IP+4000端口**测试

![](https://www.wangshengjj.work/upload/2022/06/微信图片_20220623135822.png)

## 三、配置域名访问

1.首先你需要提前准备好一个域名（最好已经备过案的，否则只能等待备案完成后才能访问，备案期间只能使用IP+端口号访问！！！）

![](https://www.wangshengjj.work/upload/2022/06/微信图片_20220613173251.png)

![](https://www.wangshengjj.work/upload/2022/06/微信图片_20220613173407.png)

2.配置反向代理

![](https://www.wangshengjj.work/upload/2022/06/微信图片_20220613174029.png)

![微信图片_20220623140119](https://www.wangshengjj.work/upload/2022/06/微信图片_20220623140119.png)

**以上全部配置完成后，即可使用域名访问博客页面**

### 拓展1：配置SSL证书（https访问）

可以使用腾讯云一键注册一个ssl证书，添加到宝塔面板即可！

**下载的证书的时候，选择Nginx证书！！！**

![](https://www.wangshengjj.work/upload/2022/06/微信图片_20220613180016.png)

![](https://www.wangshengjj.work/upload/2022/06/微信图片_20220613175749.png)

### 拓展2：认识hexo博客基本操作

1. 生成：**hexo g**
1. 启动：**hexo s**
1. 清理：**hexo clean**
1. 建议每次修改操作之后，先依次运行**hexo clean**，然后再运行**hexo g**和**hexo s**

![](https://www.wangshengjj.work/upload/2022/06/微信图片_20220623141259.png)