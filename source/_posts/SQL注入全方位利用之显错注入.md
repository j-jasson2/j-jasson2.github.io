---
title: SQL注入全方位利用之显错注入
tags: 
  - web安全
  - SQL注入
  - 显错注入
categories: web安全
keywords: '显错注入,漏洞利用,SQL注入,靶场演示'
description: SQL注入全方位利用之显错注入
cover: https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fimg.shangdixinxi.com%2Fup%2Finfo%2F202006%2F20200604170532028792.png&refer=http%3A%2F%2Fimg.shangdixinxi.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1630916300&t=5efb5b64bc17ac6be7c880a5fa3db56b
date: 2020-03-08 22:21:29
---



显错注入：顾名思义，通过显错点进行注入。那么什么是显错点呢，就是数据库中的查询结果，通过联合查询的方式，叠加替换显示了我们输入的数据。接下来只需要替换数据为我们要查询的SQL语句，就可以得到我们想要的结果。

#### 中华人民共和国网络安全法

![upload successful](/images/pasted-5.png)

![upload successful](/images/pasted-6.png)

![upload successful](/images/pasted-7.png)

#### 漏洞之王SQL注入
在这里先简单介绍一下漏洞之王SQL注入：
SQL注入即是指web应用程序对用户输入数据的合法性没有判断或过滤不严，攻击者可以在web应用程序中事先定义好的查询语句的结尾上添加额外的SQL语句，在管理员不知情的情况下实现非法操作，以此来实现欺骗数据库服务器执行非授权的任意查询，从而进一步得到相应的数据信息。
作为OWASP榜首的漏洞，SQL注入一直是网络上占比重最大的漏洞，而要实现这一漏洞，需要有两个关键条件：
1.	用户可以控制输入。
2.	用户输入的数据被当作代码拼接到了数据库语句当中。

#### 显错注入具体流程
1.这里我们先打开靶场Sqli-Labs-less1。
![upload successful](/images/pasted-45.png)

2.提示知道了需要在GET传参处传入ID，那么就传入id=1试试。可以看到多了用户名和密码，那么这个页面就是很标准的get传参显示数据的页面了。
![upload successful](/images/pasted-46.png)

3.我们日常进行渗透测试的时候，对于这种id传参或者cid传参或者其他传参，最开始也是最喜欢的，便是传个单引号试试看，这里看到页面直接报错了。（ps:单引号经过了url编码变成了%27）单引号报错那就说明我们传入的1'造成了效果，看到这种情况，我们可以直接认定，这里存在sql注入，后面会教一种简单的方法，可以直接注入，这里先不细说。
![upload successful](/images/pasted-47.png)

4.报错了报错了，为了大家能够更加直观的了解，那么我们直接作弊，看看源码吧。有点乱，这里我们没有php基础了读者可能有点慌，不要紧哦，我们只需要知道我标的那句sql查询语句即可。
```
$sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1"
```
这里大概解释一下，这里是通过查询users表中的id字段中的第一行数据，可以看到，$id处就是我们输入的数据。好的回到之前查询报错处，因为我传入了id=1' 被放到查询语句中，语句经过拼接变成：**SELECT * FROM users WHERE id='1'' LIMIT 0,1** 
这里发现了吗，我传入的单引号跟前面的单引号闭合了，可以这样理解，后面那个单引号就像一只单身狗，找不到伙伴，所以生气报错了。
![upload successful](/images/pasted-48.png)

5.那我只需要将后面的单引号丢掉，就可以页面正常显示了，这里使用#注释后面的语句（ps:#经过url编码变成了%23），可以看到，页面直接显示正常了。新的sql查询语句为:
```
SELECT * FROM users WHERE id='1' #' LIMIT 0,1
```
#后面的东西被注释了，已经没有用了，那么我们就成功跳出了单引号，在单引号和#之间，我们就可以放入自己的语句进行执行。
![upload successful](/images/pasted-49.png)

6.因为这里是显错注入嘛，下一步就要通过order by查询数据库当前的字段数，具体传参为：?id=1' order by 1#。order by 1页面正常，order by 2页面正常，order by 3页面正常，order by 4页面不正常。这就说明了当前表中的字段为3。
![upload successful](/images/pasted-50.png)

![upload successful](/images/pasted-51.png)

7.下一步就是联合查询了，传参为 **?id=1' union select 1,2,3 limit 1,1#** ，这里我通过刚刚知道了字段数为3，所以将语句换成了union select 1,2,3 ，这样的语句在页面是可以正常执行的，但是返回的数据还是dump和dump，这不是我要的显错点，所以我加上了limit 1，1 ，取出第二段数据，成功看到了2和3，这里的2，3就是我们想要的显错点了。
![upload successful](/images/pasted-52.png)

![upload successful](/images/pasted-53.png)

8.嘿嘿，看到2和3了，接下来就好办了，替换2和3为执行语句，就可以获得对应的信息。这里学习3个基础函数：database() 查询当前数据库库名 , user() 查询当前登录的用户名, version() 查询当前数据库版本号。

---
我把2换成了database()，这里成功查看到了库名security
![upload successful](/images/pasted-54.png)

9.这里普及一下，在mysql数据库中有一个information_schema表，这是系统自带的表，里面放入了我们从建库开始的所有操作，例如建表，加用户。这里给出一条查询语句：
```
union select 1,table_name,3 from information_schema.tables where 
table_schema=database() limit 1,1# 
```
这条语句通过查询了系统自带库获取了里面的表名emails
![upload successful](/images/pasted-55.png)

10.接下来查询这个表里面的字段，语句为：
```
union select 1,2,column_name from information_schema.columns where table_name=表名 and table_schema=database() limit 1,1 
```
这里有个坑，表名在语句中是以字符串的形式存在的，所以需要加单引号。
这里的传参为：
```
?id=1' union select 1,2,column_name from information_schema.columns where table_name='emails' and table_schema=database() limit 1,1#
```
看到第一个字段为id
![upload successful](/images/pasted-56.png)

```
?id=1' union select 1,2,column_name from information_schema.columns where table_name='emails' and table_schema=database() limit 2,1#
```
看到第二个字段为email_id
![upload successful](/images/pasted-57.png)

11.现在有表名emails，有字段名email_id，那就可以直接查询里面的数据了。将传参换成:
```
?id=1' union select 1,id,email_id from emails limit 1,1#
```

![upload successful](/images/pasted-58.png)

12.看到了吧，id为1的人的email为Dumb@dhakkan.com。这就是显错注入的整个过程。因为说着说着，好像内容有点多，后面的内容就放到以后的推文吧，这里说一下前面说的那个“简单”的方法，在发现报错之后，放上这条语句，可以直接查询到数据库库名哦。
```
and updatexml(1,concat(0x7e,(select database()),0x7e),1)%23
```

![upload successful](/images/pasted-59.png)

13.报错注入具体详情请看另一篇文章哦！

