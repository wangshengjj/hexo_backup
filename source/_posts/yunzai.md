---
title: TRSS-Yunzai搭建教程
date: 2023-04-09 15:40:55.883
updated: 2023-04-09 17:53:49.094
sticky: 5
categories:
- 服务器搭建
- yunzai
- 云崽
tags: 
- yunzai
- 云崽
- 原神
- 群机器人
---

# TRSS-Yunzai搭建教程

### 前言：

TRSS-Yunzai_by[时雨◎星空](https://trss.me) and [Rain kavik's Blog](https://rainkavik.com/archives/339/)

在QQ群，QQ频道，微信群使用Miao-Yunzai。
此教程所有资源均来自互联网，请勿进行任何商业性行为，仅供交流学习使用，如有侵权请联系，会立即删除。

以下教程都是为 `windows`准备的。

`go-cqhttp`和 `TRSS-Yunzai` 有liunx版本，你自己可以折磨。

`ComWeChat Client` 协议端，即微信，只能在windows上运行。

因网路问题，项目地址除了QQ和微信的协议端，都是Gitee地址，因为两个协议端目前都没有Gitee仓库 `(我没发现)`。

教程的环境文件我都存储在以下的网盘里面，你可以直接打包下载。

如果教程有问题，可通过频道联系 `Zyy.小钰 `[点击加入频道](https://pd.qq.com/s/6mkgksvwm)

文章最后我写了MC面板的托管，如果有需要，请查看。
同时在这里征集其他网盘的链接。

* [非非的个人网盘](https://cloud.rainkavik.com/s/xRMHB)

YunZai：

[TRSS-Yunzai](https://gitee.com/TimeRainStarSky/Yunzai)：gocq-Yunzai

[miao-plugin](https://gitee.com/yoimiya-kokomi/miao-plugin)：喵喵插件

QQ：

[go-cqhttp](https://docs.go-cqhttp.org/)：基于 `Mirai` 以及 `MiraiGo` 的 ` OneBot Golang` 原生实现

微信：

[ComWeChat Client](https://justundertaker.github.io/ComWeChatBotClient/) ：`ComWeChatRobot`的客户端封装，支持 `onebot12`通信协议。

### 准备工作

如果你的电脑没有任何环境，请直接下载网盘的 `全部文件 初次使用下这个.zip`

![](https://gitee.com/Yuer-QAQ/drawing-bed/blob/master/TRSS-Yunzai/1.jpg)![全部文件](https://i.328888.xyz/2023/04/09/iRLbH5.png)

如果你有搭建过Yunzai，那你的服务器或者电脑应该有Node.js、Git、Redis，你就不需要下载全部文件了，根据需求下载即可，我会在后续教程中提到你要下载的东西。

### 安装环境

##### 安装 `Node.js`

[Node.js官网下载](https://pd.qq.com/s/6mkgksvwm)

解压 `全部文件 初次使用下这个.zip`
运行 `环境 - node-v16.19.1-x64_2.msi` 无脑下一步即可。

##### 安装 `Git`

[Git官网下载](https://git-scm.com/downloads)

运行 `环境 - Git-2.40.0-64-bit.exe` 无脑下一步即可。

##### 安装 `Redis`

[Redis官网下载](https://redis.io/download/)

创建一个文件夹，命名为 `TRSS-Yunzai`

将 `环境 - Redis-x64-5.0.14.1_2.zip` 解压到文件夹内，如图。

![示例](https://i.328888.xyz/2023/04/09/iRLLJL.png)

打开 `Redis-x64-5.0.14.1_2 `运行 `redis-server.exe `。

`redis-server.exe` 运行成功如图，机器人运行期间请`不要关闭此窗口`。

![Redis](https://i.328888.xyz/2023/04/09/iRLszw.png)

### 安装TRSS-Yunzai

打开 `TRSS-Yunzai`文件夹，在地址栏输出 `cmd`，回车确认。

或者`按住Shift+鼠标右键`选择在`Powershell`打开，Windows11用户可以直接在当前窗口`鼠标右键`选择`在终端中打开`即可。

![打开cmd](https://i.328888.xyz/2023/04/09/iRLu18.png)

依次输入以下命令

##### 1.克隆项目并安装 `miao-plugin`

```
git clone --depth 1 https://gitee.com/TimeRainStarSky/Yunzai
```

```
cd Yunzai
```

```
git clone --depth 1 https://gitee.com/yoimiya-kokomi/miao-plugin plugins/miao-plugin
```

##### 2.安装[pnpm](https://gitee.com/link?target=https%3A%2F%2Fpnpm.io%2Fzh%2Finstallation)，已安装的跳过

```
npm --registry=https://registry.npmmirror.com install pnpm -g
```

##### 3.安装依赖

```
pnpm config set registry https://registry.npmmirror.com
```

```
pnpm install -P
```

##### 4.安装插件包 `(可选)`

我个人推荐先安装 `xiaoyao-cvs-plugin`，因为QQ频道和微信可以快捷使用 `#扫码登录`来获取ck。

以下所有插件包均在 `Yunzai`根目录安装，如果你按照教程来，安装依赖后可直接运行代码块内的指令。

[xiaoyao-cvs-plugin](https://gitee.com/Ctrlcvs/xiaoyao-cvs-plugin)：

```
git clone https://gitee.com/Ctrlcvs/xiaoyao-cvs-plugin.git ./plugins/xiaoyao-cvs-plugin/
```

[Guoba-Plugin](https://gitee.com/guoba-yunzai/guoba-plugin)：

安装锅巴插件可以进行修改一些默认配置。
使用方法：发送 `#锅巴登录`，具体看仓库地址

```
git clone --depth=1 https://gitee.com/guoba-yunzai/guoba-plugin.git ./plugins/Guoba-Plugin/
```

```
pnpm install --filter=guoba-plugin
```

##### 5.启动 `Yunzai-Bot`

```
node app
```

这里先设置为你的QQ账号，频道id和微信号不一样，需要全部连接完成才可以获取你的频道id和微信号。

![启动云崽](https://i.328888.xyz/2023/04/09/iRMORV.png)

### 配置[go-cqhttp](https://docs.go-cqhttp.org/) `(QQ协议框架)`

1.解压 `go-cqhttp`

打开 `QQ - go-cqhttp_windows_amd64.zip`

将其解压到 `TRSS-Yunzai`下

![解压go-cqhttp](https://i.328888.xyz/2023/04/09/iRMN6z.png)

2.配置 `go-cqhttp`

打开 `go-cqhttp_windows_amd64` 文件夹

运行 `go-cqhttp.exe`，点击三次确定！

![释放安全启动脚本](https://i.328888.xyz/2023/04/09/iRMRCa.png)

随后看到目录多了一个 `go-cqhttp.bat`

![运行脚本](https://i.328888.xyz/2023/04/09/iRM7Vv.png)

运行他，输入 `3`！回车确定，然后关闭当前窗口。

![生成配置文件](https://i.328888.xyz/2023/04/09/iRMFl8.png)

关闭后，根目录多了一个 `config.yml `用记事本打开他。

以下为必改项：

```
uin: 1233456 # QQ账号
password: '' # 密码为空时使用扫码登录
post-format: string
universal: ws://your_websocket_universal.server
```

扫码登录只能使用安卓手表协议，推荐使用密码登录。

修改成以下，QQ账号和密码自己修改为你的QQ机器人小号。

```
post-format: array
universal: ws://localhost:2536/go-cqhttp
```

修改完成记得保存。

随后再次运行 `go-cqhttp.bat`

随后输入 `1`，选择 `自动提交`

把链接复制到浏览器打开或者按 `Ctrl+鼠标单击链接`，完成滑块验证。

![请输入图片描述](https://i.328888.xyz/2023/04/09/iRPVYz.png)

如果有设备锁，请自己通过验证，如不出意外，此时你应该看到。
![请输入图片描述](https://i.328888.xyz/2023/04/09/iRPkyU.png)

你可以在频道和QQ群正常使用机器人了。

注：机器人如果需要在频道使用，需要把该子频道的消息免打扰关闭掉，否则机器人无法接收消息。关于在频道设置自己为主人，请看[#关于主人](#关于主人)。

### 配置[ComWeChat-Client](https://justundertaker.github.io/ComWeChatBotClient/) `(WeChat协议框架)`

##### 1.安装 `WeChat`

打开 `WeChat`文件夹，运行其中的 `WeChatSetup-3.7.0.30.exe` 必须此版本，不要用其他版本。

安装目录选择TRSS-Yunzai，安装完成不要打开。

##### 2.禁用自动升级

打开文件夹内的 `禁用pc微信自动升级补丁.exe`

![](https://i.328888.xyz/2023/04/09/iRPK4p.png)

点击 `一键禁用。`如果成功，则继续下一步。

如果失败，点击 `操作说明`进行手动禁用更新。

##### 3.配置ComWeChat-Client

解压 `ComWeChat-Client-v0.0.6.zip`到TRSS-Yunzai

以记事本打开 `.env` 修改以下配置。

修改前

```
websocekt_type = "Backward"
websocket_url = ["ws://localhost:2536/ComWeChat"]
```

修改后：

```
websocekt_type = "Backward"
websocket_url = ["ws://localhost:2536/ComWeChat"]
```

修改完成保存关闭。

##### 4.安装com组件

运行 `install.bat`

```
#看到以下提示即安装成功

Active code page: 65001
安装com组件成功!
Press any key to continue . . .
```

##### 5.运行ComWeChat-Client

运行 `ComWeChat-Client-v0.0.6.exe`

此时会让你扫码登录微信，自行配置即可。

登录完成看到以下画面即可成功。

![成功示例](https://i.328888.xyz/2023/04/09/iRPLNU.png)

### 关于主人

##### QQ频道

处于同一子频道，发消息，查看 `go-cqhttp`窗口。

![频道id](https://i.328888.xyz/2023/04/09/iRP7mo.png)

`144115218791749100`就是我的频道id。

写入到 `Yunzai\config\config\other.yaml `即可。

![](https://i.328888.xyz/2023/04/09/iRPz4V.png)

##### WeChat

需要当主人的QQ号给机器人小号发消息，然后查看 `ComWeChat-Client`窗口，得到微信号，如图。

最后的 `'user_id': 'wxid_zwp8eg5vz5lj22'`中的 `wxid_zwp8eg5vz5lj22` 就是你的微信号，写入到 `Yunzai\config\config\other.yaml `即可。参考上图。

![](https://i.328888.xyz/2023/04/09/iRPLNU.png)

所有教程到此结束，剩下的你可以看可以不看。

### 关于[mcsmanager](https://mcsmanager.com/)

你可以用 `mcsmanager`来托管你的机器人，不需要链接服务器也可以操作机器人。

##### 1.下载 `mcsmanager`

前往[mcsmanager官网](https://mcsmanager.com/)

点击立即下载 - 选择windows版本下载

下载完成解压，运行 `MCSManager-启动器.exe`

随后点击 `启动后台程序`

点击 `访问面板`

![启动面板](https://i.328888.xyz/2023/04/09/iRTZ9H.png)

##### 2.添加实例

点击 `访问面板` 后会面板

初次打开面板会让你创建账户密码。

选择 `应用实例`

点击 `新建实例`

![](https://i.328888.xyz/2023/04/09/iRTyhC.png)

选择 `其他游戏服务端`

选择 `无需选择或已存在文件`

![](https://i.328888.xyz/2023/04/09/iRTlAN.png)

`Redis`配置

![](https://i.328888.xyz/2023/04/09/iRTOZb.png)

`Miao-Yunzai`配置

![](https://i.328888.xyz/2023/04/09/iRTbHq.png)

`go-cqhttp`配置

![](https://i.328888.xyz/2023/04/09/iRTmrp.png)

`ComWeChatBotClient`配置

![](https://i.328888.xyz/2023/04/09/iRTp5U.png)

配置完成后，我们需要去控制台配置事件任务并开启实例。

![](https://i.328888.xyz/2023/04/09/iRTxzy.png)

把 `自动重启`和 `自动启动`打开。

你还可以自己设置计划任务，自行探索。

如果你需要在公网访问面板，请在服务器放行

TCP：23333

TCP：24444

随后输入 `公网IP:23333`即可登录面板，如需要修改端口，自行百度。

### 一些注意事项

如果你装插件包或者插件碰到缺少依赖 `oicq`。
在该插件或者插件包的根目录，打开所有JS文件，搜索其中的 `import { segment } from "oicq"`，全部删掉！

由于作者刚适配，可能会有许多bug，不过不影响使用。

如果教程有问题，可通过频道联系 `Zyy.小钰 `[点击加入频道](https://pd.qq.com/s/6mkgksvwm)
事先声明，本人没有任何义务为任何人解答问题。