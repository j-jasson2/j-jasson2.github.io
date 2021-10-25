---
title: 被忽视的漏洞-CSRF跨站请求伪造
tags: 
  - web安全
  - CSRF
  - Cookie
categories: web安全
keywords: 'web安全,CSRF,Cookie'
description: 从CSRF漏洞学习cookie对于web安全的危害
cover: https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fwww.pianshen.com%2Fimages%2F744%2F5338f05ea641c337635df38c44dcd4f0.png&refer=http%3A%2F%2Fwww.pianshen.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1630747376&t=8d4387e192b14d4412c6f21e854951f5
date: 2020-11-09 16:17:41
---

<meta name="referrer" content="no-referrer"/>

# 0x01 前言

几天前，有位师傅联系我询问CSRF的事，最近也刚好在学习CSRF，就弄出一篇文章出来吧。CSRF在我眼里，真的是被大家低估了的漏洞，配合好社会工程学的话，我觉得在现在大家对于sql注入，xss，文件上传的防护都很重视的情况下，CSRF能够造成意想不到的效果。
CSRF简介
CSRF（**跨站请求伪造**），是一种挟制用户在当前已登录的Web应用程序上执行非本意的操作的攻击方法。跟[跨网站脚本](https://baike.baidu.com/item/%E8%B7%A8%E7%BD%91%E7%AB%99%E8%84%9A%E6%9C%AC)（XSS）相比，**XSS** 利用的是用户对指定网站的信任，CSRF 利用的是网站对用户网页浏览器的信任。

# 0x02 Cookie

这里需要了解一个概念，cookie，一串字符串，是网站辨别用户的标识，也可以这样理解，拿到了某用户在某网站的cookie，我就可以登录他在那个网站的账号。
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317575665-8a8af458-2a41-42ba-ab39-dc1b0ccb3997.png#align=left&display=inline&height=193&margin=%5Bobject%20Object%5D&originHeight=822&originWidth=1764&status=done&style=none&width=415)
这里扯一嘴，偷取cookie是XSS做的事，CSRF的作用是借用cookie，并不能获取cookie。

# 0x03 CSRF原理

接下来进行CSRF的讲解，我是这样理解CSRF的，它通过构造一个poc，让已经登录某网站的用户访问，这个poc便会以该用户的cookie来操作网站，对于那个受害者来说，他点击了我的链接，浏览器就偷偷进行了一些操作。

# 0x04 危害

攻击者盗用了你的身份，以你的名义发送恶意请求。CSRF能够做的事情包括：以你名义发送邮件，发消息，盗取你的账号，甚至于购买商品，虚拟货币转账......造成的问题包括：个人隐私泄露以及财产安全。

# 0x05 检测CSRF漏洞

检测CSRF漏洞是一项比较繁琐的工作，最简单的方法就是抓取一个正常请求的数据包，去掉Referer字段后再重新提交，如果该提交还有效，那么基本上可以确定存在CSRF漏洞。当然也可以像我这样，看看有没有Token字段，没有就先认为存在。（Token是一种防御CSRF的机制，也是目前使用相对较好的防御方法，会在后面进行解释）

通过CSRF 修改受害者的个人信息
1.首先打开靶场
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317576172-fdae19b7-7517-4745-bbd1-d3108537ec73.png#align=left&display=inline&height=195&margin=%5Bobject%20Object%5D&originHeight=902&originWidth=1920&status=done&style=none&width=416)
2.注册一个新用户
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317576618-3c455c98-0ccd-4585-935a-026da67cd7aa.png#align=left&display=inline&height=195&margin=%5Bobject%20Object%5D&originHeight=902&originWidth=1920&status=done&style=none&width=416)
3.这是注册成功后的界面，可以看到有个用户管理
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317576852-50d8a2b8-4766-4030-acd6-4f25e300c97f.png#align=left&display=inline&height=183&margin=%5Bobject%20Object%5D&originHeight=735&originWidth=1669&status=done&style=none&width=416)

4.点击我的个人资料，对于我的账号进行一些修改
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317577219-a78ec569-bb05-4d57-a4bb-f211c4f3a77c.png#align=left&display=inline&height=128&margin=%5Bobject%20Object%5D&originHeight=288&originWidth=931&status=done&style=none&width=415)
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317577777-d9f0f503-a28b-4f2f-822b-42caf10fe7e9.png#align=left&display=inline&height=233&margin=%5Bobject%20Object%5D&originHeight=755&originWidth=1342&status=done&style=none&width=415)
5.通过burp抓取这个修改包，做成CSRF的poc
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317578303-f5676351-cfb0-459b-a596-d8dc8e1ec799.png#align=left&display=inline&height=210&margin=%5Bobject%20Object%5D&originHeight=863&originWidth=1704&status=done&style=none&width=415)
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317578849-a5f906aa-3d34-4a14-9378-c5b320e8a8b7.png#align=left&display=inline&height=375&margin=%5Bobject%20Object%5D&originHeight=787&originWidth=871&status=done&style=none&width=415)
6.这里我将代码存在CSRF.html
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317579367-678ec02a-3533-4bd7-b396-bece13463417.png#align=left&display=inline&height=147&margin=%5Bobject%20Object%5D&originHeight=421&originWidth=1191&status=done&style=none&width=416)
7.接下来，我们需要社工，诱骗登录了该网站的受害者点击我们的链接，这里因为是演示，就直接拿我的管理员账号来试试了。首先看看我的admin1的账号信息，这里都是admin1
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317579884-76ddacfe-cb5a-4ead-93bc-a6aa3f7e701d.png#align=left&display=inline&height=216&margin=%5Bobject%20Object%5D&originHeight=835&originWidth=1601&status=done&style=none&width=415)
8.登录了用户后，(也可以关闭该页面，cookie在登录之后是有一定的存在时间的)，在这个浏览器，访问poc的链接
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317580294-17a7df2a-59fa-4d4b-b163-26f164792b43.png#align=left&display=inline&height=139&margin=%5Bobject%20Object%5D&originHeight=278&originWidth=829&status=done&style=none&width=415)
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317580878-91f8b916-affe-45c8-a056-d423965a1fae.png#align=left&display=inline&height=111&margin=%5Bobject%20Object%5D&originHeight=189&originWidth=705&status=done&style=none&width=415)
9.点击按钮之后，回去看我的账号信息，已经变成了刚刚我做成的poc的内容
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317582113-1a84b1d7-d28f-4b7f-abe8-85c8ad0c180e.png#align=left&display=inline&height=234&margin=%5Bobject%20Object%5D&originHeight=727&originWidth=1295&status=done&style=none&width=416)

这就是简单的CSRF的功能，看着好像危害不大的样子，那么接下来让大家看看，通过CSRF来getshell。

# 0x06 通过CSRF来getshell

## 1、什么是shell？
shell就是一个脚本文件，我们通过这个脚本文件来管理或者控制服务器的文件、数据库等信息。
## 2、如何getshell？
目的是将脚本文件上传到服务器，并让服务器解析。脚本文件可以是asp、php、jsp等。

## 3、流程

1.首先打开我本机上的某cms的后台
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317582636-ffccb573-3a21-4b5a-88c3-6d6162562111.png#align=left&display=inline&height=231&margin=%5Bobject%20Object%5D&originHeight=696&originWidth=1256&status=done&style=none&width=416)


2.通过上传一句话木马构造CSRFpoc
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317583155-5333caa9-9546-4baa-9017-ffeca88fda3c.png#align=left&display=inline&height=177&margin=%5Bobject%20Object%5D&originHeight=671&originWidth=1580&status=done&style=none&width=416)
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317583689-637dfc12-9278-4ddc-8a7a-afecfe13d07b.png#align=left&display=inline&height=298&margin=%5Bobject%20Object%5D&originHeight=675&originWidth=940&status=done&style=none&width=415)

3.将poc保存为html文件，这里我进行了一些配置，设置页面为打开直接自动点击。
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317584018-ca75c982-737a-41a2-8a22-c05aed0a2f4e.png#align=left&display=inline&height=175&margin=%5Bobject%20Object%5D&originHeight=703&originWidth=1920&status=done&style=none&width=477)
4.到靶机处，登录它的网站，可以看到，这个网站的ip为靶机的，这个网站与我构造poc  的网站不是同一个。
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317584541-63e6ebf9-7b18-43b8-8134-b5d40d4b90cb.png#align=left&display=inline&height=245&margin=%5Bobject%20Object%5D&originHeight=749&originWidth=1270&status=done&style=none&width=415)
5.接下来，通过靶机访问构造好的html
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317585031-bbcac406-020f-4caa-a996-7ce4857b9d46.png#align=left&display=inline&height=137&margin=%5Bobject%20Object%5D&originHeight=384&originWidth=1164&status=done&style=none&width=416)
6.点击之后，一闪而过了保存文件的界面之后跳出了文件管理器，看到IP知道文件已经保存成功了。
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317585542-404664fc-7061-4812-a95f-8ffff4562569.png#align=left&display=inline&height=184&margin=%5Bobject%20Object%5D&originHeight=568&originWidth=1280&status=done&style=none&width=415)
还是验证一下吧，回去之前那个页面刷新后发现确实保存成功了
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317586014-06a09b2f-9033-4715-a5d2-20a7dc0a4e97.png#align=left&display=inline&height=248&margin=%5Bobject%20Object%5D&originHeight=761&originWidth=1273&status=done&style=none&width=415)
7.现在也就是说我已经将木马文件放到对方的服务器上了，接下来回到本机，连接菜刀，拿下服务器
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317586564-9db664a5-45ff-497f-a00b-7024fd2d4a17.png#align=left&display=inline&height=186&margin=%5Bobject%20Object%5D&originHeight=725&originWidth=1625&status=done&style=none&width=416)
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317587075-2f880b21-12a0-4fc7-91e5-9803851f01ff.png#align=left&display=inline&height=186&margin=%5Bobject%20Object%5D&originHeight=725&originWidth=1625&status=done&style=none&width=416)

# 0x07 防御CSRF
1.验证 HTTP Referer 字段
根据 HTTP 协议，在 HTTP 头中有一个字段叫 Referer，它记录了该 HTTP 请求的来源地址。这种验证方法通过查看referer来验证CSRF攻击的数据包是否是用户自己的操作，当然因为可以伪造referer，所以现在比较没有那么流行。
2.Token机制
可以在 HTTP 请求中以参数的形式加入一个随机产生的 token，并在服务器端建立一个拦截器来验证这个 token，如果请求中没有 token 或者 token 内容不正确，则认为可能是 CSRF 攻击而拒绝该请求。一般为了预防CSRF或者其他攻击，都能够在抓包处发现存在token字段，有的是get传参，有的是post传参。token 可以在用户登陆后产生并放于 session 之中，然后在每次请求时把 token 从 session 中拿出，与请求中的 token 进行比对，如果token一样，则执行操作，token不一样就不予执行。
3.在 HTTP 头中自定义属性并验证

# 0x08 总结思考
CSRF漏洞因为需要受害者点击才能够触发，所以经常不被重视，但是正是这种情况下，CSRF能够造成的危害会比想象中要大。嗯嗯，社会工程学就是高端的欺骗，具体可以百度学习一下。本篇文章就到这里了，对于实验中的环境，工具，有需要的可以添加我的微信获取。
