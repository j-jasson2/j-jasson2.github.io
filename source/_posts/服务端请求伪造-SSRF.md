---
title: 服务器端请求伪造-SSRF
tags: 
  - web安全
  - CSRF
  - SSRF
  - 服务器端请求伪造
categories: web安全
keywords: 'SSRF,服务器端请求伪造,任意文件读取,文件读取,实战演示'
description: 从CSRF漏洞学习cookie对于web安全的危害
cover: https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F27a30abf3c1075c2dcff9371a6d8905e.png&refer=http%3A%2F%2Fimg-blog.csdnimg.cn&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1630914926&t=2db2045c1dd34b138e02c2edd50efca7
date: 2020-11-28 14:05:18
---

<meta name="referrer" content="no-referrer"/>

最初认识SSRF会与CSRF跨站请求伪造很容易混淆。二者虽然名字很像，但是实际运用却千差万别。SSRF是构造形成由服务器发起请求，攻击内网主机的漏洞；而CSRF则是构造形成由客户端发起请求，攻击服务器的漏洞。

# 概念
SSRF(Server-Side Request Forgery)，服务器端请求伪造，利用漏洞伪造服务器端发起请求，从而突破客户端获取不到数据限制。	
攻击者往往可以通过SSRF访问到目标服务器所在的内网资源，这些资源是正常用户访问不到的，而且因为SSRF可以支持伪协议，所以还存在直接通过Gopher协议写入webshell的危害。
# SSRF的危害
>1.内外网的端口和服务扫描
2.主机本地敏感数据的读取
3.内外网主机应用程序漏洞的利用
4.内外网Web站点漏洞的利用 

# SSRF漏洞可能出现的地方
因为SSRF是服务器端请求伪造，那么可以看出，SSRF漏洞出现的位置，肯定是服务器加载资源处。或者说，当服务器有向外加载资源，就有可能存在SSRF漏洞。例如：
>1、图片加载/下载：通过URL地址加载或下载图片
2、图片/文章收藏功能：从URL地址中取出title以及文本的内容作为显示
3、导出功能：从服务器中加载文件进行导出
4、在线翻译
5、一些api接口提供的加载其他url功能
# 实战渗透SSRF漏洞
1、首先打开指定站点，这里看到打开页面是通过加载模块进行回显的
![upload successful](/images/13/1.png) 

2、直接通过file伪协议获取文件内容发现显示模块不存在
![upload successful](/images/13/2.png)  

3、定位存在漏洞的代码文件：app/setting/controller/ApiAdminDomainSettings.php
![upload successful](/images/13/3.png) 

4、分析代码
```
（1）通过input传参postAddress
（2）将postAddress赋予$api
（3）$api经过$options处理，最后赋予给$ch
（4）执行curl_exec
```
5、传参获取数据库文件内容，里面获取了用户名root，密码root。
```
postAddress=file:///E:\New File\WWW\phpStudy\WWW\app\database.php&url=test&id=test
```
![upload successful](/images/13/4.png)  

![upload successful](/images/13/5.png)  

6、读取php配置文件php.ini
![upload successful](/images/13/6.png)  

