---
title: halo博客搭建教程
date: 2022-06-13 18:02:05.77
updated: 2022-11-12 16:03:03.054
categories: 
- 服务器搭建
tags: 
- 服务器搭建
- linux搭建服务器
- halo博客
---

# halo博客搭建教程

#### 写在文章前面：本教程部分内容基于[幻幻博客](https://blog.xhxx.cc/)

## halo博客升级和备份教程 [点我传送](https://www.wangshengjj.work/archives/8)

**仅供学习交流使用，如果侵犯到你的合法权利，请联系邮件删除，或评论。我将会在24h内删除。**

### 一、选择服务器和域名

我这里用腾讯云服务器做演示

[服务器购买链接](https://cloud.tencent.com/act/pro/618season?channel=sp&fromSource=gwzcw.6633950.6633950.6633950&utm_medium=cpc&utm_id=gwzcw.6633950.6633950.6633950&bd_vid=11065626160413136653)

新用户可以选择45一年的轻量级服务器

![](https://www.wangshengjj.work/upload/2022/06/微信图片_20220613164726.png)

购买域名

[域名购买链接](https://cloud.tencent.com/act/domainsales?channel=sp&fromSource=gwzcw.6463875.6463875.6463875&utm_medium=cpc&utm_id=gwzcw.6463875.6463875.6463875&bd_vid=9026323095797005744)

这里推荐购买.top和.cc的域名（主要是次年续费便宜），可以根据自己网站的名称去选择注册

![](https://www.wangshengjj.work/upload/2022/06/微信图片_20220613165407.png)

### 二、开始准备服务器

来到腾讯云控制台

[点我跳转到腾讯云控制台](https://console.cloud.tencent.com/)

![](https://www.wangshengjj.work/upload/2022/06/微信图片_20220613165945.png)

#### 1.开始部署服务器

进入我们的服务器管理，选择“新建”后会跳转到新的页面（服务器创建过程中会慢一点，请耐心等待）

![](https://www.wangshengjj.work/upload/2022/06/微信图片_20220613174620.png)

![](https://www.wangshengjj.work/upload/2022/06/微信图片_20220613170450.png)

设置防火墙策略

开放8888端口和8090端口

![微信图片_20220613182231](https://www.wangshengjj.work/upload/2022/06/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20220613182231.png)

#### 2.获取宝塔面板地址和账号密码

我们创建完成后，点击登录即可

![](https://www.wangshengjj.work/upload/2022/06/微信图片_20220613170919.png)

在终端输入以下代码

```
sudo /etc/init.d/bt default
```

![](https://www.wangshengjj.work/upload/2022/06/微信图片_20220613171144.png)

我复制好网址，就可以访问了！

![](https://www.wangshengjj.work/upload/2022/06/微信图片_20220613171440.png)

我们登录进去以后选择“推荐安装”

![](https://www.wangshengjj.work/upload/2022/06/825eec95f9da2.png)

### 三、配置环境

1.下载并且安装Docker环境

![](https://www.wangshengjj.work/upload/2022/06/微信图片_20220613172236.png)

2.创建halo目录（回到终端页面）

```
mkdir ~/.halo && cd ~/.halo
```

3.拉取并运行halo(目前最新版本1.6.0，如果有新版本，可以删除版本号拉取最新latest版本)

```
docker run -it -d --name halo -p 8090:8090 -v ~/.halo:/root/.halo --restart=unless-stopped halohub/halo:1.6.0
```

**拓展：**

> 1. -it 开启输入功能并连接伪终端
> 2. -d 保持后台运行容器
> 3. --name 为容器指定一个名字
> 4. -p 映射指定端口
> 5. -v 映射宿主机上的目录
> 6. --restart 在Docker开机自启后自动启动容器

4.浏览器访问：**http:// 服务器IP:8090** 即可进入halo博客的安装引导界面

### 四、配置域名访问

1.首先你需要提前准备好一个域名（最好已经备过案的，否则只能等待备案完成后才能访问，备案期间只能使用IP+端口号访问！！！）

![](https://www.wangshengjj.work/upload/2022/06/微信图片_20220613173251.png)

![](https://www.wangshengjj.work/upload/2022/06/微信图片_20220613173407.png)

2.配置反向代理

![](https://www.wangshengjj.work/upload/2022/06/微信图片_20220613174029.png)

![](https://www.wangshengjj.work/upload/2022/06/微信图片_20220613174032.png)

**以上全部配置完成后，即可使用域名访问博客页面**

### 五、拓展：配置SSL证书（https访问）

可以使用腾讯云一键注册一个ssl证书，添加到宝塔面板即可！

**下载的证书的时候，选择Nginx证书！！！**

![](https://www.wangshengjj.work/upload/2022/06/微信图片_20220613180016.png)

![](https://www.wangshengjj.work/upload/2022/06/微信图片_20220613175749.png)

**管理页面（仪表盘）**

浏览器访问域名或者IP+端口号，成功进入主页以后，在后面加上"/admin"即可进到管理页面

![微信图片_20220623142303](https://www.wangshengjj.work/upload/2022/06/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20220623142303.png)

### 特别鸣谢

[halo官方文档](https://docs.halo.run/)

[幻幻博客](https://blog.xhxx.cc/)