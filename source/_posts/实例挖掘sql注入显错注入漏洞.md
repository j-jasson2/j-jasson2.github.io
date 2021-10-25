---
title: 花几分钟找个上市公司的SQL注入漏洞
tags: 
  - SQL注入
  - 显错注入
  - 漏洞利用
  - 实战渗透
  - 漏洞挖掘
categories: web安全
keywords: 'SQL注入,显错注入,漏洞利用,实战渗透,漏洞挖掘'
description: 花几分钟找个上市公司的SQL注入漏洞
cover: /images/pasted-36.png
date: 2020-03-02 10:45:41
---

之前发出了一篇显错注入的文章之后，很多读者反馈说难度太大，看不懂，那么今天，我进行了改进，为大家推送一篇漏洞实例，实战演示怎么挖掘sql显错注入漏洞。最后说一句，我们要做一名合法的白帽子，要学会 **点到为止** 。

## 中华人民共和国网络安全法
![upload successful](/images/pasted-41.png)

![upload successful](/images/pasted-42.png)

![upload successful](/images/pasted-43.png)
## 寻找漏洞
1.首先打开谷歌镜像站，这里不推荐搭梯子（vpn），因为这个违法且不安全，所以我们使用大佬搭的镜像站（百度搜索，找到一个能用的就行，这里推荐一个http://www.googlen.org/ ），搜索栏里输入： 
```
inurl:php?sid= 广州 有限公司
```
![upload successful](/images/pasted-33.png)

---

![upload successful](/images/pasted-34.png)

2.每打开一个网站，就在它的url地址后面加个单引号‘或者反斜杠\，如果页面报错了，那么恭喜你，这里有sql注入漏洞。经过尝试，在一个页面中，我加个单引号，页面产生错误了（ps:单引号在url栏里被url编码为%27）

![upload successful](/images/pasted-35.png)

## 显错注入具体流程
1.看到了warning，也就是说这个网站存在sql注入，先在%27后面加个空格，加个%23（%23通过url解码为#，起到注释的作用），新的url传参为：
```
?id=243' #
```

![upload successful](/images/pasted-36.png)

2.可以看到，闭合成功了，页面返回正常，这个时候，就是猜字段了，在单引号和#之间放入order by 数字，猜字段
当前传参为：
```
?id=243' order by 1 #
```

![upload successful](/images/pasted-37.png)

3.根据之前的教学推文中知道，接下来就是通过order by 判断当前数据库字段数了，order by 1页面正常，order by 2 页面正常……一直到order by 29的时候，页面又出现了错误，下面的文字内容没了，所以我们知道，字段数为28。当前url传参为：
```
?id=243' order by 29 #
```

![upload successful](/images/pasted-38.png)

4.字段知道了，接下来就是联合查询了，获取了显错点4，6，13。（ps:这里有个坑，243变成了-243，这是为了让联合查询前面的语句报错，这样就会输出后面的显错点了，跟limit 1,1 更换显示为第2行道理相同）
当前url传参为：
```
?id=-243' union select 1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28 #
```

![upload successful](/images/pasted-39.png)

5.有了显错点，已经可以说是找到漏洞了，我们偷偷看看它的数据库，将4，6，13换成之间推文里提到的函数，获取数据库库名，数据库当前用户名，数据库版本号。

![upload successful](/images/pasted-40.png)

6.这里我打码了一部分，数据库库名开头为ao ，数据库当前用户名为ao***@localhost，数据库当前版本为5.5.45。本次实例就到这里，点到为止嘛，查到库名已经可以去漏洞提交平台进行提交了。本次实例仅作为学习资料，若有出现打码遗漏等问题，也请大家不要对网站进行攻击。

