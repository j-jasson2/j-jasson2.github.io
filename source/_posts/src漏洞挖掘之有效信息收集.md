---
title: src漏洞挖掘之有效信息收集
tags: 
  - SRC
  - 信息收集
  - 漏洞挖掘
categories: web安全
keywords: 'SRC,信息收集,漏洞挖掘'
description: src漏洞挖掘之有效信息收集
cover: /images/src_wajue.png
date: 2021-10-25 10:45:41

---

<meta name="referrer" content="no-referrer"/>

> 本文已发布到先知社区
>
> 作者：ajie

    
    说到信息收集，网上已经有许多文章进行描述了，那么从正常的子域名、端口、旁站、C段等进行信息收集的话，对于正常项目已经够用了，但是挖掘SRC的话，在诸多竞争对手的“帮助”下，大家收集到的信息都差不多，挖掘的漏洞也往往存在重复的情况。
      那么现在我就想分享一下平时自己进行SRC挖掘过程中，主要是如何进行入手的。以下均为小弟拙见，大佬勿喷。
## 0x01 确定目标
个人是非常讨厌无目标随便打的，有没有自己对应的SRC应急响应平台不说，还往往会因为一开始没有挖掘到漏洞而随意放弃，这样往往不能挖掘到深层次的漏洞。挖到的大多数是大家都可以简单挖到的漏洞，存在大概率重复可能。所以在真的想要花点时间在SRC漏洞挖掘上的话，建议先选好目标。
	那么目标怎么选呢，考虑到收益回报与付出的比例来看，建议是从专属SRC入手，特别在一些活动中，可以获取比平时更高的收益。
微信搜一搜：
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1634919905280-6a3474d5-e288-40e0-be14-a11ca5a37bcb.png#clientId=u713ba10f-a757-4&from=paste&height=660&id=u8b4a879d&margin=%5Bobject%2Object%5D&name=image.png&originHeight=1320&originWidth=1360&originalType=binary&ratio=1&size=333456&status=done&style=none&taskId=u74e51aee-8625-4c5e-b368-f880fa9654b&width=680)
百度搜一搜：
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1634919958148-21279f08-65f5-45e2-93cc-37781713440c.png#clientId=u713ba10f-a757-4&from=paste&height=668&id=ubea7849e&margin=%5Bobject%2Object%5D&name=image.png&originHeight=1336&originWidth=1784&originalType=binary&ratio=1&size=467822&status=done&style=none&taskId=u40ba20aa-3b6f-465a-b6ff-105b03a1cac&width=892)
现在有活动的src已经浮现水面了，那么我们就可与从中选择自己感兴趣的SRC。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1634963068419-ddbf4fb7-605b-46a5-aa91-139da2bd03c9.png#clientId=u713ba10f-a757-4&from=paste&height=706&id=u4d43608b&margin=%5Bobject%2Object%5D&name=image.png&originHeight=1412&originWidth=1726&originalType=binary&ratio=1&size=575771&status=done&style=none&taskId=u7150b32d-8964-4f4b-8100-3de307b4870&width=863)

## 0x02 确认测试范围
前面说到确定测什么SRC，那么下面就要通过一些方法，获取这个SRC的测试范围，以免测偏。
### 1、公众号
从公众号推文入手，活动页面中可以发现测试范围
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1634963158654-f2016b05-9be3-4cd7-9bda-ea55a87defb1.png#clientId=u713ba10f-a757-4&from=paste&height=630&id=u077f22ee&margin=%5Bobject%2Object%5D&name=image.png&originHeight=1260&originWidth=1452&originalType=binary&ratio=1&size=190701&status=done&style=none&taskId=u56153bbe-9613-4ac9-96e0-549a84f1c4b&width=726)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1634963557707-c9d9fa67-1108-49c4-b38a-a74b34a3907f.png#clientId=u713ba10f-a757-4&from=paste&height=630&id=u1c25375f&margin=%5Bobject%2Object%5D&name=image.png&originHeight=1260&originWidth=1452&originalType=binary&ratio=1&size=285450&status=done&style=none&taskId=ue71919ac-0609-45cc-bc63-c3b9f4d2e1c&width=726)
### 2、应急响应官网
在应急响应官网，往往会有一些活动的公告，在里面可以获取到相应的测试范围。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1634963624546-d5d91332-8080-4b77-9972-77fc8abb52bb.png#clientId=u713ba10f-a757-4&from=paste&height=627&id=u128b3210&margin=%5Bobject%2Object%5D&name=image.png&originHeight=1254&originWidth=2206&originalType=binary&ratio=1&size=321049&status=done&style=none&taskId=u0aa25af0-c011-434c-92f5-f7e3a65d236&width=1103)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1634963709884-04818b7a-2686-4c31-b96b-270fd21af35b.png#clientId=u713ba10f-a757-4&from=paste&height=656&id=u3228ec56&margin=%5Bobject%2Object%5D&name=image.png&originHeight=1312&originWidth=2382&originalType=binary&ratio=1&size=592096&status=done&style=none&taskId=u83999522-9aaa-4b01-b710-a57b9ff4f7f&width=1191)
### 3、爱企查
从爱企查等商业查询平台获取公司所属域名
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1634964085716-15745fa8-785a-4209-b610-668ece5133a0.png#clientId=u713ba10f-a757-4&from=paste&height=484&id=u412ef52f&margin=%5Bobject%2Object%5D&name=image.png&originHeight=968&originWidth=2188&originalType=binary&ratio=1&size=786218&status=done&style=none&taskId=u55826600-0854-44f8-accb-54e3f12c347&width=1094)
搜索想要测试等SRC所属公司名称，在知识产权->网站备案中可以获取测试范围。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1634964167749-6d7913b9-a4be-41ce-a417-88f49ebd7673.png#clientId=u713ba10f-a757-4&from=paste&height=567&id=u96bf1fb3&margin=%5Bobject%2Object%5D&name=image.png&originHeight=1134&originWidth=1884&originalType=binary&ratio=1&size=298889&status=done&style=none&taskId=ua446ee4f-aa26-41c6-8e34-ba5e91db54e&width=942)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1634964215181-046d3df9-0dbb-44f2-ba89-59ecdee77f71.png#clientId=u713ba10f-a757-4&from=paste&height=665&id=uf3208897&margin=%5Bobject%2Object%5D&name=image.png&originHeight=1330&originWidth=1836&originalType=binary&ratio=1&size=280408&status=done&style=none&taskId=u8bb717b4-4748-4fbd-b919-6102ba1883b&width=918)
## 0x03 子域名(oneforall)
拿到域名之后，下一步我考虑使用oneforall扫描获取子域名，就像网上信息收集的文章一样，主域名的站点不是静态界面就是安全防护等级极强，不是随便就能够发现漏洞的，我们挖掘SRC也是要从子域名开始，从边缘资产或一般资产中发现漏洞。
工具下载：
```
https://github.com/shmilylty/OneForAll
```
具体用法如下：
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1634967168625-da7d57b5-19e1-4f9c-91a2-cd55e8072344.png#clientId=u713ba10f-a757-4&from=paste&height=442&id=u73c9b970&margin=%5Bobject%2Object%5D&name=image.png&originHeight=884&originWidth=1460&originalType=binary&ratio=1&size=385397&status=done&style=none&taskId=ub7b3972f-c412-4a4e-a3c0-224717feaa2&width=730)	
常用的获取子域名有2种选择，一种使用--target指定单个域名，一种使用--targets指定域名文件。
```
python3 oneforall.py --target example.com run
python3 oneforall.py --targets ./domains.txt run
```
其他获取子域名的工具还有layer子域名挖掘机、Sublist3r、证书透明度、在线工具等，这里就不一一阐述了，大体思路是一样等，获取子域，然后从中筛选边缘资产，安全防护低资产。
## 0x04 系统指纹探测
通过上面的方法，我们可以在/OneForAll-0.4.3/results/路径下获取以域名为名字的csv文件。里面放入到便是扫描到到所有子域名以及相应信息了。
下一步便是将收集到到域名全部进行一遍指纹探测，从中找出一些明显使用CMS、OA系统、shiro、Fastjson等的站点。下面介绍平时使用的2款工具：
### 1、Ehole
下载地址：
```
https://github.com/EdgeSecurityTeam/EHole
```
使用方法：
```
./Ehole-darwin -l url.txt   //URL地址需带上协议,每行一个
./Ehole-darwin -f 192.168.1.1/24  //支持单IP或IP段,fofa识别需要配置fofa密钥和邮箱
./Ehole-darwin -l url.txt -json export.json  //结果输出至export.json文件
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1634968178111-36292547-bea8-487c-a8cf-722c63427df3.png#clientId=u713ba10f-a757-4&from=paste&height=415&id=u3671b8ae&margin=%5Bobject%2Object%5D&name=image.png&originHeight=830&originWidth=1460&originalType=binary&ratio=1&size=320488&status=done&style=none&taskId=u181ebb87-7fc2-4806-81f9-6a3c42cf463&width=730)
### 2、Glass
下载地址：
```
https://github.com/s7ckTeam/Glass
```
使用方法：
```
python3 Glass.py -u http://www.examples.com  // 单url测试
python3 Glass.py -w domain.txt -o 1.txt  // url文件内
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1634968385558-f19e053b-b480-4bbb-8995-3cc50cb19ea1.png#clientId=u713ba10f-a757-4&from=paste&height=499&id=uc057208f&margin=%5Bobject%2Object%5D&name=image.png&originHeight=998&originWidth=1460&originalType=binary&ratio=1&size=357066&status=done&style=none&taskId=u2dd4f645-9400-4a16-90f9-567e1292a6c&width=730)
## 0x05 框架型站点漏洞测试
前面经过了子域名收集以及对收集到的子域名进行了指纹信息识别之后，那么对于框架型的站点，我们可以优先进行测试。
类似用友NC、通达OA、蓝凌OA等，可以通过尝试现有的Nday漏洞进行攻击。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1634971752296-14f4cc8b-a9ad-4f4a-a52d-b37290bf0889.png#clientId=u713ba10f-a757-4&from=paste&height=670&id=ue2eb2d79&margin=%5Bobject%2Object%5D&name=image.png&originHeight=1340&originWidth=1796&originalType=binary&ratio=1&size=633891&status=done&style=none&taskId=uc617d959-80c1-4194-ba8a-5a20f30043a&width=898)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1634971772511-ad2c33fa-0dcf-4e7b-8a36-ada77f0d208d.png#clientId=u713ba10f-a757-4&from=paste&height=660&id=uf7ff8327&margin=%5Bobject%2Object%5D&name=image.png&originHeight=1320&originWidth=1360&originalType=binary&ratio=1&size=311843&status=done&style=none&taskId=u10544bf0-4a12-4c42-bf41-7e9c5d45057&width=680)
## 0x06 非框架型站点漏洞测试
前面测试完框架型的站点了，之后就应该往正常网站，或者经过了二开未能直接检测出指纹的站点进行渗透了。那么对于这类站点，最经常遇到的便是登录框，在这里，我们便可以开始测试了。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1634972124912-1bf2fa5c-a1ac-40b0-b6b9-71fea6341075.png#clientId=u713ba10f-a757-4&from=paste&height=416&id=uc47928c9&margin=%5Bobject%2Object%5D&name=image.png&originHeight=832&originWidth=1322&originalType=binary&ratio=1&size=623185&status=done&style=none&taskId=ubc371a35-7ed8-4893-9488-84188fd50cf&width=661)
1、用户名枚举
抓包尝试是否用户名存在与不存在的情况，返回结果不同。
2、验证码
是否存在验证码，验证码是否可以抓包截断绕过，验证码是否可以为空。
3、暴力破解
下面是我收集的集中常见的用户名
```
1.弱口令用户名如admin,test,ceshi等
2.员工姓名全拼，员工姓名简拼
3.公司特征+员工工号/员工姓名
4.员工工号+姓名简拼
5.员工姓名全拼+员工工号
6.员工姓名全拼+重复次数，如zhangsan和zhangsan01
7.其他
```
关于暴力破解我要扯一句了，就是关于密码字典的问题。经常会听到某人说他的字典多么多么的大，有好几个G之类的，但是在我觉得，这很没有必要，有些密码是你跑几天都跑不出来的，就算字典确实够大，也没有必要这样跑，可能影响心情不说，大规模地暴力破解，很容易让人觉得你在拒绝服务攻击。
​

其实我的话一般跑一跑弱口令就差不多了。
关于弱口令字典的问题，我也想说一嘴，你最好看看，你字典里面的admin、123456、password处在什么位置。记得之前玩CTF的时候，默认密码123456，但是那个师傅死活做不出来，后面一看，字典里面居然没有123456这个密码。。。
​

这里推荐一个字典，个人感觉还是挺好用的。当然更多的是需要自己不断更新。
```
https://github.com/fuzz-security/SuperWordlist
```
4、工具cupp和cewl
对于一些情况，密码不是直接使用弱口令，而是通过一些公司的特征+个人信息制作的，那么这个时候，我们的字典便不能直接使用了，需要在这之前加上一些特征，例如阿里SRC可能是a；百度SRC可能是bd等。
下面2款kali自带等工具，可以通过收集信息，生成好用的字典，方便渗透。说真的，在渗透测试过程中，弱口令，YYDS！
具体使用说明和工具介绍，可以查看文章：
[https://mp.weixin.qq.com/s/HOlPaJ4EMY7PfHh7p2d95A](https://mp.weixin.qq.com/s/HOlPaJ4EMY7PfHh7p2d95A)
5、自行注册
如果能够注册那就好办了，自己注册一下账户即可。
6、小总结
对于非框架的站点，登录接口一般是必不可少的，可能就在主页，也可能在某个路径下，藏着后台的登录接口，在尝试了多种方法成功登录之后，记得尝试里面是否存在未授权漏洞、越权等漏洞。
这里借用来自WS师傅的建议：可以直接扫描出来的洞，基本都被交完了，可以更多往逻辑漏洞方面找。登录后的漏洞重复率，比登录前的往往会低很多。
## 0x07 端口扫描
前面就是正常的渗透了，那么一个域名只是在80、443端口才有web服务吗？不可否认有些时候真的是，但是绝大多数情况下，类似8080、8443、8081、8089、7001等端口，往往会有惊喜哦～
端口扫描也算是老生常谈了，市面上也有很多介绍端口扫描的工具使用方法，这里也不细说了，就放出平时使用的命令吧。
```
sudo nmap -sS -Pn -n --open --min-hostgroup 4 --min-parallelism 1024 --host-timeout 30 -T4 -v  examples.com

sudo nmap -sS -Pn -n --open --min-hostgroup 4 --min-parallelism 1024 --host-timeout 30 -T4 -v -p 1-65535 examples.com
```
## 0x08 目录扫描dirsearch
目录扫描在渗透测试过程中我认为是必不可少的，一个站点在不同目录下的不同文件，往往可能有惊喜哦。
个人是喜欢使用dirserach这款工具，不仅高效、页面也好看。市面上还有例如御剑、御剑t00ls版等，也是不错的选择。
dirsearch下载地址：
```
https://github.com/maurosoria/dirsearch
```
具体使用方法可以查看github介绍，这里我一般是使用如下命令（因为担心线程太高所以通过-t参数设置为2）
```
python3 dirsearch.py -u www.xxx.com -e * -t 2
```
关键的地方是大家都可以下载这款工具，获取它自带的字典，那么路径的话，便是大家都能够搜得到的了，所以这里我推荐是可以适当整合一些师傅们发出来的路径字典到/dirsearch-0.4.2/db/dicc.txt中。例如我的话，是增加了springboot未授权的一些路径、swagger的路径以及一些例如vmvare-vcenter的漏洞路径。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1634974064316-4b6934a1-20a6-4491-bd58-90ce95839dc2.png#clientId=u713ba10f-a757-4&from=paste&height=418&id=ucf26188a&margin=%5Bobject%2Object%5D&name=image.png&originHeight=836&originWidth=1364&originalType=binary&ratio=1&size=132938&status=done&style=none&taskId=uec31186e-75ff-4ff7-bd63-c714e41f7df&width=682)
## 0x09 JS信息收集
在一个站点扫描了目录、尝试登录失败并且没有自己注册功能的情况下，我们还可以从JS文件入手，获取一些URL，也许某个URL便能够未授权访问获取敏感信息呢。
#### 1、JSFinder
工具下载：
```
https://github.com/Threezh1/JSFinder
```
JSFinder是一款用作快速在网站的js文件中提取URL，子域名的工具。个人觉得美中不足的地方便是不能对获取到到URL进行一些过滤，在某些情况下，JS文件中可以爬取非常多的URL，这其中可能大部分是页面空或者返回200但是页面显示404的。来自HZ师傅的建议，可以修改一下工具，基于当前的基础上，检测获取的URL是否可以访问，访问后的页面大小为多少，标题是什么。。。
思路放这了，找个时间改一改？
```
#检测URL状态码
#-----------------------
#! /usr/bin/env python
#coding=utf-8
import sys
import requests

url='xxxx'
request = requests.get(url)
httpStatusCode = request.status_code
if httpStatusCode == 200:
    xxxx
else:
		xxxx
```
```
#检测URL返回包大小
#-----------------------
import requests

def hum_convert(value):
    units = ["B", "KB", "MB", "GB", "TB", "PB"]
    size = 1024.0
    for i in range(len(units)):
        if (value / size) < 1:
            return "%.2f%s" % (value, units[i])
        value = value / size

r = requests.get('https://www.baidu.com')
r.status_code
r.headers

length = len(r.text)
print(hum_convert(length))
```
```
#获取网站标题
#-----------------------
#!/usr/bin/python
#coding=utf-8

urllib.request
import urllib.request
import re

url = urllib.request.urlopen('http://www.xxx.com')
html = url.read().decode('utf-8')

title=re.findall('<title>(.+)</title>',html)
print (title)
```
#### 2、JS文件
JS文件与HTML、CSS等文件统一作为前端文件，是可以通过浏览器访问到的，相对于HTML和CSS等文件的显示和美化作用，JS文件将会能够将页面的功能点进行升华。![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1634979609194-1625b496-f395-46ef-a57b-d40812e3ca83.png#clientId=u713ba10f-a757-4&from=paste&height=41&id=udf5e6bd9&margin=%5Bobject%2Object%5D&name=image.png&originHeight=82&originWidth=1052&originalType=binary&ratio=1&size=16086&status=done&style=none&taskId=u48d38e72-9bd0-43d2-ad50-bcc0ce56aec&width=526)
对于渗透测试来说，JS文件不仅仅能够找到一些URL、内网IP地址、手机号、调用的组件版本等信息，还存在一些接口，因为前端需要，所以一些接口将会在JS文件中直接或间接呈现。下面我将介绍如何发现这些隐藏的接口。
1、首先在某个页面中，鼠标右键，选择检查![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1634979847526-bc9f09c0-1a90-455d-ba9e-21f6a85077b2.png#clientId=u713ba10f-a757-4&from=paste&height=727&id=u06aaf698&margin=%5Bobject%2Object%5D&name=image.png&originHeight=1454&originWidth=2364&originalType=binary&ratio=1&size=884946&status=done&style=none&taskId=u680f6578-9ead-4a4d-a566-d93f3fb8e26&width=1182)
2、点击Application
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1634979884349-2daca2ef-6982-41b2-bb71-5c764816e908.png#clientId=u713ba10f-a757-4&from=paste&height=399&id=u951102d4&margin=%5Bobject%2Object%5D&name=image.png&originHeight=798&originWidth=1772&originalType=binary&ratio=1&size=290368&status=done&style=none&taskId=ud0a0670c-1fa3-4315-867c-25409dd85f8&width=886)
3、在Frames->top->Scripts中能够获取当前页面请求到的所有JS
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1634979911722-aedd9206-580a-476c-aafe-5e99d91a2d47.png#clientId=u713ba10f-a757-4&from=paste&height=604&id=u785851a2&margin=%5Bobject%2Object%5D&name=image.png&originHeight=1208&originWidth=1148&originalType=binary&ratio=1&size=133350&status=done&style=none&taskId=udc4effe1-a736-4b7d-884e-b10381b8c03&width=574)
4、火狐浏览器的话，则是在调试中
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1634980094740-b4afaf82-8f10-4f64-b368-da3faa3f5d93.png#clientId=u713ba10f-a757-4&from=paste&height=323&id=ua48ca828&margin=%5Bobject%2Object%5D&name=image.png&originHeight=646&originWidth=1646&originalType=binary&ratio=1&size=188824&status=done&style=none&taskId=ucae14475-c2e8-489d-98d3-a672350edf4&width=823)
5、如果你请求的JS文件内容都叠在了前几行的话，下面这个键可以帮你美化输出
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1634980174451-835cf97c-8c17-4d2a-a372-469073a78281.png#clientId=u713ba10f-a757-4&from=paste&height=269&id=u80fc4db5&margin=%5Bobject%2Object%5D&name=image.png&originHeight=850&originWidth=1420&originalType=binary&ratio=1&size=78688&status=done&style=none&taskId=ud815e24c-4503-40d1-a91c-ae3ec92a358&width=449)
6、在JS文件中，可以尤为注意带有api字眼的文件或内容，例如下面这里我发现了一个接口。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1634980516041-202ef7c0-0773-45fc-8ea0-8935ad385603.png#clientId=u713ba10f-a757-4&from=paste&height=427&id=udeb8bb8a&margin=%5Bobject%2Object%5D&name=image.png&originHeight=854&originWidth=1400&originalType=binary&ratio=1&size=199487&status=done&style=none&taskId=ue53b03c2-3b94-4dda-8b57-943ff687ffe&width=700)

## 0x10 小程序、APP

web端没有思路的时候，可以结合小程序、APP来进行渗透。小程序或APP的服务端其实可以在一定程度上与web应用的服务端相联系。也就是说，我们在小程序或者APP上，一样能够挖掘web端的漏洞如SQL注入、XSS等，并且相对来说，这类等服务端安全措施会相对没有那么完备，所以在web端确实没有思路的时候，可以迂回渗透，从小程序、APP中进行。

```
#小程序抓包、APP抓包参考链接：
https://mp.weixin.qq.com/s/xuoVxBsN-t5KcwuyGpR56g
https://mp.weixin.qq.com/s/45YF4tBaR-TUsHyF5RvEsw
https://mp.weixin.qq.com/s/M5xu_-_6fgp8q0KjpzvjLg
https://mp.weixin.qq.com/s/Mfkbxtrxv5AvY-n_bMU7ig
```

## 0x11 总结

​		以上就是我个人挖掘SRC的一些信息收集思路，挖掘SRC有的时候真的很看运气，也许别人对一个接口简单Fuzz，便出了一个注入，而我们花了几天，还是一直看到返回内容为404。所以有的时候真的可以换个站试试，也许就挖到高危甚至严重了～
​		作为一名SRC小白，以上内容均为小弟拙见，希望能够通过这篇文章，帮到更多的网络安全小白，没能帮上大佬们真的很抱歉～后续也会持续提高自己，将学到的更多的东西分享给大家。
