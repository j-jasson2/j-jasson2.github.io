---
title: Nginx中间件漏洞之add_header被覆盖
tags: 
  - Nginx
  - 中间件漏洞
  - web安全
categoryries: web安全
keywords: 'web安全,Nginx,中间件漏洞'
description: Nginx中间件漏洞之add_header被覆盖
cover: https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fupload-images.jianshu.io%2Fupload_images%2F3778244-65c62b2e2775b20a.png%3FimageMogr2%2Fauto-orient%2Fstrip%257CimageView2%2F2%2Fw%2F1240&refer=http%3A%2F%2Fupload-images.jianshu.io&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1630941831&t=6d3fe55cc7cb43085f42e676981de631
date: 2021-08-07 09:45:41
---

<meta name="referrer" content="no-referrer"/>

# 0x01 复现环境

本地搭建docker+vulhub
```shell
cd vulhub-master 
cd nginx/insecure-configuration 
docker-compose up -d
```
访问[http://127.0.0.1:8082/](http://127.0.0.1:8082/)显示如下界面即搭建成功
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1626765408455-657778d3-0c6a-4334-b7fc-4997dc0ddbb8.png#align=left&display=inline&height=337&margin=%5Bobject%2Object%5D&name=image.png&originHeight=674&originWidth=1478&size=88008&status=done&style=none&width=739)

# 0x02 漏洞复现
首先访问/test1，这里发现设置了Content-Security-Policy
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1626766283341-d1e4fb11-642c-4fbb-91d4-92ff04c4c833.png#align=left&display=inline&height=459&margin=%5Bobject%2Object%5D&name=image.png&originHeight=918&originWidth=1820&size=213630&status=done&style=none&width=910)
访问/test2，Content-Security-Policy被X-Content-Type-Options: nosniff替换了，即安全配置失效。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1626766358395-e1afd4f5-f850-4c6b-9e2f-a332224d77f2.png#align=left&display=inline&height=458&margin=%5Bobject%2Object%5D&name=image.png&originHeight=916&originWidth=1800&size=209111&status=done&style=none&width=900)
这种情况网上说是可以导致XSS漏洞的，我这里失败了，具体图片如下：
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1626766461360-0db2ae04-19ef-4307-a494-f1d1f8856075.png#align=left&display=inline&height=303&margin=%5Bobject%2Object%5D&name=image.png&originHeight=606&originWidth=1996&size=101702&status=done&style=none&width=998)


# 0x03 漏洞分析
先看具体配置情况：
```nginx
add_header Content-Security-Policy "default-src 'self'"; //CSP安全头部
add_header X-Frame-Options DENY;

location = /test1 {
    rewrite ^(.*)$ /xss.html break;
} 

location = /test2 {
    add_header X-Content-Type-Options nosniff;
    rewrite ^(.*)$ /xss.html break;
}
```
首先可以看到在全局设置了CSP安全头部
接下来看到在/test2配置了一个**X-Content-Type-Options nosniff;**


这里因为子块配置了**X-Content-Type-Options**，所以父块的配置失效，所以存在漏洞。
