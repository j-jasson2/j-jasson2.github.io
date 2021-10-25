---
title: Nginx中间件漏洞之目录穿越
tags: 
  - Nginx
  - 中间件漏洞
  - web安全
categories: 中间件安全
keywords: 'web安全,Nginx,中间件漏洞'
description: Nginx中间件漏洞之目录穿越
cover: https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fupload-images.jianshu.io%2Fupload_images%2F3778244-65c62b2e2775b20a.png%3FimageMogr2%2Fauto-orient%2Fstrip%257CimageView2%2F2%2Fw%2F1240&refer=http%3A%2F%2Fupload-images.jianshu.io&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1630941831&t=6d3fe55cc7cb43085f42e676981de631
date: 2021-08-06 09:45:41
---

<meta name="referrer" content="no-referrer"/>

# 0x01 复现环境

本地搭建docker+vulhub
```shell
cd vulhub-master 
cd nginx/insecure-configuration 
docker-compose up -d
```
访问[http://127.0.0.1:8081/files/](http://127.0.0.1:8081/files/)显示如下界面即搭建成功
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1626762972928-b9934d33-9b5d-49a7-9710-443d17b6f35d.png#align=left&display=inline&height=250&margin=%5Bobject%2Object%5D&name=image.png&originHeight=500&originWidth=1200&size=45202&status=done&style=none&width=600)

# 0x02 漏洞复现

这个常见于 Nginx 做反向代理的情况，动态的部分被 proxy_pass 传递给后端端口，而静态文件需要 Nginx 来处理。
假设静态文件存储在 /home/ 目录下，而该目录在 url 中名字为 files ，那么就需要用 alias 设置目录的别名：

```
location /files {
    alias /home/;
}
```
当使用如下配置的情况下，就存在目录穿越漏洞，payload如下：
```shell
http://127.0.0.1:8081/files../
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1626763021203-ec62dc04-8a66-41b7-b64b-48a4b58390c3.png#align=left&display=inline&height=568&margin=%5Bobject%2Object%5D&name=image.png&originHeight=1136&originWidth=1980&size=177487&status=done&style=none&width=990)
# 0x03 漏洞分析
正常输入http:ip:port/files/help.txt，访问到file目录下到help.txt文件
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1626763091219-7ec90fec-239a-47c8-9abf-ab5d8ddceed5.png#align=left&display=inline&height=165&margin=%5Bobject%2Object%5D&name=image.png&originHeight=330&originWidth=1132&size=31821&status=done&style=none&width=566)
根据配置情况可以发现，/file的后面没有/，也就是只要输入/file便表示代理到服务器的/home/目录下了。
当我们输入/file../，在服务器上对应的就是/home/../，所以跳转到根目录，造成了目录穿越，如下图：
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1626763482134-4c4b2eb7-8d7c-49ef-8716-5ea99a66dade.png#align=left&display=inline&height=574&margin=%5Bobject%2Object%5D&name=image.png&originHeight=1148&originWidth=1812&size=173397&status=done&style=none&width=906)
