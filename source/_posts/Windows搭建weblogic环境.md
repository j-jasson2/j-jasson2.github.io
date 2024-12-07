---
title: Windows搭建weblogic环境
tags: 
  - 环境搭建
  - weblogic
  - windows
categories: 环境搭建
keywords: '环境搭建,weblogic,windows'
description: Windows搭建weblogic环境
cover: https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fwww.51wendang.com%2Fpic%2F10b9ee610cba11bb1d02938e%2F5-810-jpg_6-1080-0-0-1080.jpg&refer=http%3A%2F%2Fwww.51wendang.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1630917022&t=a4900175111cf1e7c26c7a5ed713c7af
date: 2020-09-30 10:45:41
---

介于网络上weblogic漏洞环境基本上都是通过docker+vulhub进行搭建的，而有的小伙伴想要通过windows来搭建而没有教程，下面我将向大家介绍如何通过windows来搭建weblogic环境。


1、下载好weblogic的安装文件，双击运行
下载地址：https://www.oracle.com/middleware/technologies/weblogic.html
![upload successful](/images/16/1.png) 

2、读条结束弹出安装界面
![upload successful](/images/16/2.png) 

3、点击下一步，设置目录
![upload successful](/images/16/3.png) 

4、设置好目录后，进入设置更新界面，这里将勾去掉，不更新。
![upload successful](/images/16/4.png) 

5、下一步，安装类型选择典型即可
![upload successful](/images/16/5.png) 

6、安装目录，这里我选择默认，也可以自行调整
![upload successful](/images/16/6.png) 

7、快捷方式，直接下一步
![upload successful](/images/16/7.png) 

8、安装概要，不配置，下一步
![upload successful](/images/16/8.png) 

9、开始安装
![upload successful](/images/16/9.png) 

10、安装成功
![upload successful](/images/16/10.png) 

11、点击完成，弹出quickstart
![upload successful](/images/16/11.png) 

12、点击getting started with WebLogic Server 10.3.6，创建新的域
![upload successful](/images/16/12.png) 

13、点击下一步，这里默认即可
![upload successful](/images/16/13.png) 

14、域名，就用默认的
![upload successful](/images/16/14.png) 

15、下一步，设置名称和口令
![upload successful](/images/16/15.png) 

16、设置好之后点击下一步，这里选择生产模式
![upload successful](/images/16/16.png) 

17、下一步，可选设置，这里不选。
![upload successful](/images/16/17.png) 

18、配置概要，默认即可，点击创建
![upload successful](/images/16/18.png) 

19、完成
![upload successful](/images/16/19.png) 

20、安装完成后，在这个路径下可以开启weblogic
```
C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Oracle Enterprise Pack for Eclipse\User Projects\base_domain
```
22、双击打开，输入之前设置的用户名和密码，启动服务
![upload successful](/images/16/20.png) 

23、打开192.168.137.1:7001/console/，成功访问管理登录界面
![upload successful](/images/16/21.png) 

