---
title: 复现：通达OA任意以管理员身份进入后台
tags: 
  - 通达OA
  - 未授权访问
  - 漏洞利用
  - 实战演示
  - 漏洞复现
categories: 漏洞复现
keywords: '通达OA,未授权访问,漏洞利用,实战演示,漏洞复现'
description: 复现：通达OA任意以管理员身份进入后台
cover: https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fwww.tongda2000.com%2Foa%2FMYOA2015%2Ffree%2Fstyle%2Fimages%2Fslide%2FSport%2F4.jpg&refer=http%3A%2F%2Fwww.tongda2000.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1630915199&t=d53ae596832d4a1c4d980fc3fae686dd
date: 2020-05-13 13:33:52
---

日前通达OA爆出一个大漏洞，用户可以在登录界面，通过修改cookie的方式，以管理员身份登录后台，这里我就来复现一次。
#### 通达OA
说到通达OA，大家应该都不陌生吧。	
通达OA（Office Anywhere网络智能办公系统）是某公司自主研发的协同办公自动化软件，是与中国企业管理实践相结合形成的综合管理办公平台。	
说白了，通达OA是一个网站模板，有很多网站在使用。	
POC地址：https://github.com/NS-Sp4ce/TongDaOA-Fake-User

#### 中华人民共和国网络安全法
在这里提醒一下，作为一名白帽子，大家一定不要做一些违法乱纪的事。文章开头先学习一下中华人民共和国网络安全法：

![upload successful](/images/pasted-5.png)

![upload successful](/images/pasted-6.png)

![upload successful](/images/pasted-7.png)

#### 复现步骤
1.首先打开网络空间搜索引擎FOFA（非常强大）	
地址：https://fofa.so/
![upload successful](/images/pasted-71.png)

2.在搜索栏处搜索输入“通达OA”


![upload successful](/images/pasted-72.png)

会看到很多可以选择的以ip或域名访问的界面，这里基本上都是以通达OA搭建的网站。

3.随便打开一个，会出现一个登录界面

![upload successful](/images/pasted-73.png)

4.通过运行前面给的poc获取一个新的cookie

![upload successful](/images/pasted-74.png)

![upload successful](/images/pasted-75.png)

5.修改cookie为新生成的cookie（修改cookie的方法有很多，这里我用的是插件）


![upload successful](/images/pasted-76.png)

6.修改好了之后，访问url下的general目录下的index.php文件
```
example：http：//www.xxx.com/general/index.php
```
7.可以看到，已经成功以管理员身份登录后台

![upload successful](/images/pasted-77.png)

#### Over...



