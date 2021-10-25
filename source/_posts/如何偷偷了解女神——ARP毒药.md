---
title: 如何偷偷了解女神——ARP毒药攻击
tags: 
  - ARP欺骗
  - 女神
  - mac地址
  - 实战演示
categories: ARP欺骗
keywords: 'ARP欺骗,女神,mac地址,实战演示'
description: 如何偷偷了解女神——ARP毒药攻击
cover: https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fwww.51wendang.com%2Fpic%2Fc300ace510021256dc6dba05%2F1-810-jpg_6-1080-0-0-1080.jpg&refer=http%3A%2F%2Fwww.51wendang.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1630915883&t=937aa034e18c55ac26f722976f6d62cd
date: 2020-05-28 17:33:34
---



最近学习了一种攻击——ARP毒药攻击。中间人攻击，也叫ARP欺骗，能够通过arp欺骗获取用户的数据，那么认真想想，这样能不能让自己更加了解女神呢，接下来就让我们来了解一下这个强大的攻击手法吧。

### 中华人民共和国网络安全法
文章开头，先学习一下中华人民共和国网络安全法，大家要做一名合法的白帽子，不要做一些违法乱纪的事情。	
https://www.cto.ac.cn/thread-106.htm

![upload successful](/images/pasted-5.png)

![upload successful](/images/pasted-6.png)

![upload successful](/images/pasted-7.png)

### 环境准备
1.攻击机：kali虚拟机 （ip:192.168.163.128）
![upload successful](/images/pasted-96.png)

2.被攻击机：windows sever 2008 （ip: 192.168.163.131）
![upload successful](/images/pasted-97.png)

3.网关：win10物理机 （ip: 192.168.163.2）


### 攻击原理

访问一个地址需要一种协议叫ARP协议，我们的主机通过ARP协议获取了对应ip地址的mac地址来进行访问。

情况是这样的，A访问某网站需要经过网关，这往往是直连的，但是第三方B，通过向A和网关发送数据包，修改了他们的mac地址指向了B，也就是说，A访问网关的数据包都会经过B再到达网关，达到了中间人攻击的效果。

![upload successful](/images/pasted-98.jpg)

### 攻击过程

1.首先打开攻击机kali和靶机Windows server 2008
![upload successful](/images/pasted-99.png)

![upload successful](/images/pasted-100.png)

2.在靶机命令行处通过arp –a命令获取了当前mac地址，网关192.168.163.2的mac地址为00-50-56-f3-04-25
![upload successful](/images/pasted-101.png)


3.攻击机机也查看一下，查看了当前mac地址为00:0c:29:bc:fd:f2
![upload successful](/images/pasted-102.png)


4.准备充分了，接下来在攻击机打开软件ettercap，通过-G参数打开图形化界面
![upload successful](/images/pasted-103.png)


5.刚打开的界面需要先选择收取信息的网卡，一般默认的eth0即可，然后打上勾勾进入软件。
![upload successful](/images/pasted-104.png)


6.进入软件后，额，因为是英文的，会有点烦，不过我们可以通过按钮来操作。
先点击那个放大镜按钮获取当前网段存在的主机。
![upload successful](/images/pasted-105.png)

然后点击那个展示按钮查看信息。
![upload successful](/images/pasted-106.png)


7.接下来就是选择目标了，将靶机192.168.163.131加入target 1；将网关192.168.163.2加入target2（这里也可以批量欺骗，就是target1可以加入多个进行欺骗）
![upload successful](/images/pasted-107.png)


8.选完了目标之后，可以进行攻击了。点击右上角那个地球一样的按钮，选择第一个ARP pois……；进去之后选择第一个sniff remote connections（嗅探远程连接） 打勾，点ok就可以了。
![upload successful](/images/pasted-108.png)

![upload successful](/images/pasted-109.png)


9.开始攻击之后在页面下面是可以看到已经建立了连接。
![upload successful](/images/pasted-110.png)

同样的，在靶机上也可以看到，网关的mac地址变了，变成了跟攻击机的地址一样了，这也就是说，已经欺骗成功了
![upload successful](/images/pasted-111.png)


10.这就算攻击成功了吗?当然没有，接下来就是当靶机进行网络访问，登录账号密码的时候，能够在攻击机上查看到账号密码。	
先使用靶机登录某界面。
![upload successful](/images/pasted-112.png)


11.为了能够直观点我将密码显示出来（在密码处右键检查，在弹出来的框处找到type="password"，修改成type="text"即可）。
![upload successful](/images/pasted-113.png)

12.点击了登录后，回到攻击机，已经获取了账号密码以及被我打码的网址。
![upload successful](/images/pasted-114.png)

### 总结

就这样，我们获取了靶机的账号密码，如果是想要女神的账号密码，只需要想办法将ip弄到手就可以了哦。当然这里不仅仅是获取账号密码，可以使用其他抓包软件，获取其他数据包。当然这里还是以学习为主，大家开开玩笑就好，不要真的去偷人家数据。

### ARP欺骗防御

arp欺骗当然也是可以防御的，对于我们客户端，可以将mac地址设置为静态的，只允许手动更改。对于服务器端，可以对于站点加个ssl加密，也就是http -- > https。

参考链接：
https://www.cnblogs.com/ichunqiu/p/5662832.html