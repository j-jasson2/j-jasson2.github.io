---
title: 如何搭建自己的dnslog服务器
tags: 
  - dnslog
  - vps
  - 域名
  - dnslog搭建
categories: 环境搭建
keywords: 'dnslog,vps,域名,dnslog搭建'
description: 如何搭建自己的dnslog服务器
cover: /images/18/8.png
date: 2020-12-08 10:58:18
---

Dnslog即dns日志，通过记录访问了的域名或者子域名。	
在渗透测试过程中，攻击者往往无法直接获取相应的数据，那么就可以通过将数据拼接到dns中，将数据外带出来。	

# 下面介绍如何搭建dnslog服务器
首先需要有一个可以修改dns设置的域名以及一个vps
推荐通过阿里云购买
## 域名ajie.xxxx
## vps：1.1.1.1

在服务器中需要进行如下设置	
1、设置dns修改
![upload successful](/images/18/1.png) 

2、设置域名解析	
两个NS记录，一个A记录	
![upload successful](/images/18/2.png)  
3、服务器安装go环境	
wget https://dl.google.com/go/go1.10.3.linux-amd64.tar.gz	
sudo tar -C /usr/local -xzf go1.10.3.linux-amd64.tar.gz	
vi /etc/profile	
末尾添加export PATH=$PATH:/usr/local/go/bin	
![upload successful](/images/18/3.png) 

4、安装dnslog服务器	
下载地址：	
https://github.com/lanyi1998/DNSlog-GO/releases/tag/1.2	
我选择的是linux系统
![upload successful](/images/18/4.png) 

5、解压获取里面的文件，修改其中的config.ini	
Port = 8080 //HTTP监听端口	
Token = ajie //API token，相当于密码	
ConsoleDisable = false //禁用web控制台，设置为true以后无法访问web页面，只能通过API获取数据	
![upload successful](/images/18/5.png) 

6、开启dnslog服务
./main
![upload successful](/images/18/6.png)

7、访问8080端口，可以查看dnslog的web页面
![upload successful](/images/18/7.png)

8、测试dns解析，成功记录
![upload successful](/images/18/8.png)  

