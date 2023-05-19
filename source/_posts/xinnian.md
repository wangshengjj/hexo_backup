---
title: 给你的网站挂上灯笼
date: 2022-12-31 10:21:53.601
updated: 2023-02-08 11:54:08.238
categories: 
- 我的水货
- 技术
tags: 
- halo博客
- 技术
- 新年
---

# 给网站挂上灯笼教程

**本教程代码原作者[TomyJan](https://tomys.top)**

原文直达[链接](https://blog.tomys.top/2021-02/2021newyear/)

仅供学习交流使用，如果侵犯到你的合法权利，请联系邮件删除，或评论。我将会在24h内删除。

废话不多直接上效果图

![微信图片_20221231101907](https://www.wangshengjj.work/upload/2022/12/%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20221231101907.png)

找个地方随便加入head标签内就行

```
<!-- 灯笼1 -->
<div class="deng-box">
	<div class="deng">
		<div class="xian"></div>
		<div class="deng-a">
			<div class="deng-b"><div class="deng-t">年</div></div>
		</div>
		<div class="shui shui-a"><div class="shui-c"></div><div class="shui-b"></div></div>
	</div>
</div>
 
<!-- 灯笼2 -->
<div class="deng-box1">
	<div class="deng">
		<div class="xian"></div>
		<div class="deng-a">
			<div class="deng-b"><div class="deng-t">新</div></div>
		</div>
		<div class="shui shui-a"><div class="shui-c"></div><div class="shui-b"></div></div>
	</div>
</div>
 
 
<style type="text/css">
.deng-box {
	position: fixed;
	top: -40px;
	right: -20px;
	z-index: 999;
}
 
.deng-box1 {
	position: fixed;
	top: -30px;
	right: 10px;
	z-index: 999;
}
 
.deng-box1 .deng {
	position: relative;
	width: 120px;
	height: 90px;
	margin: 50px;
	background: #d8000f;
	background: rgba(216, 0, 15, 0.8);
	border-radius: 50% 50%;
	-webkit-transform-origin: 50% -100px;
	-webkit-animation: swing 5s infinite ease-in-out;
	box-shadow: -5px 5px 30px 4px rgba(252, 144, 61, 1);
}
 
.deng {
	position: relative;
	width: 120px;
	height: 90px;
	margin: 50px;
	background: #d8000f;
	background: rgba(216, 0, 15, 0.8);
	border-radius: 50% 50%;
	-webkit-transform-origin: 50% -100px;
	-webkit-animation: swing 3s infinite ease-in-out;
	box-shadow: -5px 5px 50px 4px rgba(250, 108, 0, 1);
}
 
.deng-a {
	width: 100px;
	height: 90px;
	background: #d8000f;
	background: rgba(216, 0, 15, 0.1);
	margin: 12px 8px 8px 8px;
	border-radius: 50% 50%;
	border: 2px solid #dc8f03;
}
 
.deng-b {
	width: 45px;
	height: 90px;
	background: #d8000f;
	background: rgba(216, 0, 15, 0.1);
	margin: -4px 8px 8px 26px;
	border-radius: 50% 50%;
	border: 2px solid #dc8f03;
}
 
.xian {
	position: absolute;
	top: -20px;
	left: 60px;
	width: 2px;
	height: 20px;
	background: #dc8f03;
}
 
.shui-a {
	position: relative;
	width: 5px;
	height: 20px;
	margin: -5px 0 0 59px;
	-webkit-animation: swing 4s infinite ease-in-out;
	-webkit-transform-origin: 50% -45px;
	background: #ffa500;
	border-radius: 0 0 5px 5px;
}
 
.shui-b {
	position: absolute;
	top: 14px;
	left: -2px;
	width: 10px;
	height: 10px;
	background: #dc8f03;
	border-radius: 50%;
}
 
.shui-c {
	position: absolute;
	top: 18px;
	left: -2px;
	width: 10px;
	height: 35px;
	background: #ffa500;
	border-radius: 0 0 0 5px;
}
 
.deng:before {
	position: absolute;
	top: -7px;
	left: 29px;
	height: 12px;
	width: 60px;
	content: " ";
	display: block;
	z-index: 999;
	border-radius: 5px 5px 0 0;
	border: solid 1px #dc8f03;
	background: #ffa500;
	background: linear-gradient(to right, #dc8f03, #ffa500, #dc8f03, #ffa500, #dc8f03);
}
 
.deng:after {
	position: absolute;
	bottom: -7px;
	left: 10px;
	height: 12px;
	width: 60px;
	content: " ";
	display: block;
	margin-left: 20px;
	border-radius: 0 0 5px 5px;
	border: solid 1px #dc8f03;
	background: #ffa500;
	background: linear-gradient(to right, #dc8f03, #ffa500, #dc8f03, #ffa500, #dc8f03);
}
 
.deng-t {
	font-family: 华文行楷,Arial,Lucida Grande,Tahoma,sans-serif;
	font-size: 3.2rem;
	color: #dc8f03;
	font-weight: bold;
	line-height: 85px;
	text-align: center;
}
 
.night .deng-t, 
.night .deng-box, 
.night .deng-box1 {
	background: transparent !important;
}
 
@-moz-keyframes swing {
	0% {
		-moz-transform: rotate(-10deg)
	}
 
	50% {
		-moz-transform: rotate(10deg)
	}
 
	100% {
		-moz-transform: rotate(-10deg)
	}
}
 
@-webkit-keyframes swing {
	0% {
		-webkit-transform: rotate(-10deg)
	}
 
	50% {
		-webkit-transform: rotate(10deg)
	}
 
	100% {
		-webkit-transform: rotate(-10deg)
	}
}
</style>
```