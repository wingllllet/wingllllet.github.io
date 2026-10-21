---
title: "Python基本库使用[更新中]"
date: 2024-02-16T20:51:43+08:00
categories: ["爬虫"]
tags: ["python"]
image: "https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/2402262226.jpg"
---

## requests

### 安装

```shell
pip3 install requests
```

如果有pycharm可在这个进行添加

![image-20240224111542944](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/image-20240224111542944.png)

### 实例

运行下面代码

```python
import requests

resp = requests.get("https://www.baidu.com")
print(type(resp))                    // 响应类型
print(resp.status_code)              // 状态码
print(type(resp.text))               // 响应体类型
print(resp.text[:100])               // 响应体源码
print(resp.cookies)                  // cookie内容
```

运行结果如下

```
<class 'requests.models.Response'>
200
<class 'str'>
<!DOCTYPE html>
<!--STATUS OK--><html> <head><meta http-equiv=content-type content=text/html;charse
<RequestsCookieJar[<Cookie BDORZ=27315 for .baidu.com/>]>
```

可以看见使用`requests.get`方法就相当于对该网址进行了`GET请求`。

`requests`作为强大的网站请求模块，它的请求方法也不止一种

```python
requests.get(url)
requests.post(url)
requests.put(url)
requests.delete(url)
requests.patch(url)
```

这里只是粗略的介绍，后面再详细介绍。

### GET请求

构造一个简单的get请求，链接是https\://www\.httpbin.org/get，这个网站可以判断客户端是不是进行了GET请求，并会返回响应包

```
import requests

resp = requests.get("https://www.httpbin.org/get")
print(resp.text)
```

运行结果

```json
{
  "args": {}, 
  "headers": {
    "Accept": "*/*", 
    "Accept-Encoding": "gzip, deflate", 
    "Host": "www.httpbin.org", 
    "User-Agent": "python-requests/2.28.1", 
    "X-Amzn-Trace-Id": "Root=1-65d964ae-5eed64e46ba780af34b5b853"
  }, 
  "origin": "194.99.79.246", 
  "url": "https://www.httpbin.org/get"
}
```

返回结果中有请求头、url、ip等信息，这就说明我们成功发起了GET请求

- **添加额外信息**

如果要在GET请求中添加额外信息，在正常请求时通常是写成 `http://www.temple.com/?name=winglet `所以在用模块进行请求时也可以写成这样。

但是在requests模块中我们也可以通过`params`参数进行添加，例

```python
import requests

data = {
    "name": "winglet",
    "age": "21"
}
resp = requests.get("http://www.httpbin.org/get", params=data)
print(resp.text)
```

运行结果

```json
{
  "args": {
    "age": "21", 
    "name": "winglet"
  }, 
  "headers": {
    "Accept": "*/*", 
    "Accept-Encoding": "gzip, deflate", 
    "Host": "www.httpbin.org", 
    "User-Agent": "python-requests/2.28.1", 
    "X-Amzn-Trace-Id": "Root=1-65d9ae9f-79d024cb23c6e21356632251"
  }, 
  "origin": "194.99.79.246", 
  "url": "http://www.httpbin.org/get?name=winglet&age=21"
}
```

返回结果与访问`http://www.httpbin.org/get?name=winglet&age=21`的返回结果一样

这里我们可以发现上面实验的返回结果明显是json类型，所以我们也可以转换它们的类型

```python
import requests

resp = requests.get("http://www.httpbin.org/get")
print(type(resp.text))
print(resp.json())
print(type(resp.json()))
```

运行结果

```json
<class 'str'>
{'args': {}, 'headers': {'Accept': '*/*', 'Accept-Encoding': 'gzip, deflate', 'Host': 'www.httpbin.org', 'User-Agent': 'python-requests/2.28.1', 'X-Amzn-Trace-Id': 'Root=1-65d9b0c4-7caba3c72eecb41d635c8987'}, 'origin': '194.99.79.246', 'url': 'http://www.httpbin.org/get'}
<class 'dict'>
```

- **抓取网页**

就以上面抓取结果来说，我们通常抓出来的就是这么一大串信息，但需要在这些信息里面提取有效信息，这里我们就以`https://ssr1.scrape.center/`为例

```python
import requests
import re

resp = requests.get("https://ssr1.scrape.center/")
pattern = re.compile("<h2.*?>(.*?)</h2>", re.S)
tiltles = re.findall(pattern, resp.text)
print(tiltles)
```

运行结果

```json
['霸王别姬 - Farewell My Concubine', '这个杀手不太冷 - Léon', '肖申克的救赎 - The Shawshank Redemption', '泰坦尼克号 - Titanic', '罗马假日 - Roman Holiday', '唐伯虎点秋香 - Flirting Scholar', '乱世佳人 - Gone with the Wind', '喜剧之王 - The King of Comedy', '楚门的世界 - The Truman Show', '狮子王 - The Lion King']
```

在提取有效信息时就需要用到re模块，就是用正则表达式来匹配有效信息，这块后面再讲。

- **抓取二进制数据**

我们常见抓取的二进制数据就是图片了。像博客上传图片就是生成一个图片链接，所以我们直接访问其链接就能获取到这个图片的二进制内容，然后浏览器就会解析并将该图片展现出来。

现在随便抓取一个图片

```python
import requests

resp = requests.get("https://scrape.center/favicon.ico")
print(resp.text[:100])
print(resp.content[:100])     // byte类型
```

运行结果

![image-20240224213915508](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/image-20240224213915508.png)

存储这么一大串字符是个麻烦的事，所以我们要把它转换成图片，图片说白了就是一个二进制文件，把它转换成文件存储起来就可以了

```python
import requests

resp = requests.get("https://scrape.center/favicon.ico")
with open("1.ico", 'wb') as f:
    f.write(resp.content)
```

这样我们就在当前目录下生成了一个`1.ico`文件

- **添加请求头**

现在很多网站都有一些反爬机制，我们在之前的请求中是没有设置请求头的，有一些网站检测你的请求头时发现`user-agent`字段没有数据，就会认为该请求不是来自一个正常的浏览器，就会返回异常结果阻止爬取。

那么我们就需要自己去设置请求头，在`requests`模块中我们用`headers`参数来设置请求头内容

```python
import requests

headers = {
    "user-agent": "Mozilla / 5.0(Windows NT 10.0; Win64; x64) AppleWebKit / 537.36(KHTML, like Gecko) Chrome / 80.0.3987.122  Safari / 537.36"
}
resp = requests.get("https://scrape.center/favicon.ico", headers=headers)
```

通过此方法便可以传送请求头

### POST请求

检测是否进行了POST请求

```python
import requests

url = "https://www.httpbin.org/post"
resp = requests.post(url)
print(resp.text)xxxxxxxxxx import import requestsurl = "https://www.httpbin.org/post"resp = requests.post(url)print(resp.text)
```

运行结果

```json
{
  "args": {}, 
  "data": "", 
  "files": {}, 
  "form": {}, 
  "headers": {
    "Accept": "*/*", 
    "Accept-Encoding": "gzip, deflate", 
    "Content-Length": "0", 
    "Host": "www.httpbin.org", 
    "User-Agent": "python-requests/2.28.1", 
    "X-Amzn-Trace-Id": "Root=1-65db3cc9-36d6eb4a782f3e713ca51969"
  }, 
  "json": null, 
  "origin": "194.99.79.246", 
  "url": "https://www.httpbin.org/post"
}
```

有返回结果说明是POST请求。

如果要提交表单的话，需要使用data参数

```json
import requests

url = "https://www.httpbin.org/post"
data = {
    "name": "winglet"
}
resp = requests.post(url, data=data)
print(resp.text)
```

运行结果

```json
{
  "args": {}, 
  "data": "", 
  "files": {}, 
  "form": {
    "name": "winglet"
  }, 
  "headers": {
    "Accept": "*/*", 
    "Accept-Encoding": "gzip, deflate", 
    "Content-Length": "12", 
    "Content-Type": "application/x-www-form-urlencoded", 
    "Host": "www.httpbin.org", 
    "User-Agent": "python-requests/2.28.1", 
    "X-Amzn-Trace-Id": "Root=1-65dca076-47c67045141f47526b668509"
  }, 
  "json": null, 
  "origin": "194.99.79.246", 
  "url": "https://www.httpbin.org/post"
}
```

可以看见我们提交的表单数据就被放在了`form`参数中

### 响应

发送了请求后，自然能得到响应，我们上述的结果中只是展示了响应的一部分，之前我们讲到过，响应里面是有响应体，状态码，cookie等信息的。所以我们也可以通过其他方法函数将它们展示出来

```python
import requests

url = "https://ssr1.scrape.center"
resp = requests.get(url)
print(type(resp.status_code), resp.status_code)
print(type(resp.headers), resp.headers)
print(type(resp.cookies), resp.cookies)
print(type(resp.url), resp.url)
print(type(resp.history), resp.history)
```

运行结果

```json
<class 'int'> 200
<class 'requests.structures.CaseInsensitiveDict'> {'Date': 'Mon, 26 Feb 2024 14:36:24 GMT', 'Content-Type': 'text/html; charset=utf-8', 'X-Frame-Options': 'DENY', 'X-Content-Type-Options': 'nosniff', 'Expires': 'Mon, 26 Feb 2024 14:38:47 GMT', 'Strict-Transport-Security': 'max-age=15724800; includeSubDomains', 'Server': 'E0MID', 'X-Cache-Lookup': 'Cache Miss, Cache Miss, Cache Miss, Cache Miss', 'Cache-Control': 'max-age=600', 'Age': '0', 'Content-Length': '41667', 'X-NWS-LOG-UUID': '17190695892387289445', 'Connection': 'keep-alive'}
<class 'requests.cookies.RequestsCookieJar'> <RequestsCookieJar[]>
<class 'str'> https://ssr1.scrape.center/
<class 'list'> []
```

基本信息便如此。

可以看见状态码是200，表示网站正常访问。不过在`requests`模块中，还存在内置函数来表示某种状态。比如`requests.codes.ok`就表示`200`，当然还有很多状态表示函数，不过个人感觉没必要去记，做个了解即可。

### 高级用法

- **文件上传**

直接实例展示

```python
import requests

file = {
    "file": open("1.ico", "rb")
}
resp = requests.post("http://httpbin.org/post", files=file)
print(resp.text)
```

运行结果

```json
{
  "args": {}, 
  "data": "", 
  "files": {
    "file": "data:application/octet-stream;base64,AAABAAEAI......"
  }, 
  "form": {}, 
  "headers": {
    "Accept": "*/*", 
    "Accept-Encoding": "gzip, deflate", 
    "Content-Length": "4427", 
    "Content-Type": "multipart/form-data; boundary=3f1d8575b6590aa87284daeaf1ebab4a", 
    "Host": "httpbin.org", 
    "User-Agent": "python-requests/2.28.1", 
    "X-Amzn-Trace-Id": "Root=1-65dca3f0-7485d19b3a37ce032285cfc5"
  }, 
  "json": null, 
  "origin": "194.99.79.246", 
  "url": "http://httpbin.org/post"
}
```

利用`files`我们就成功通过POST请求上传了一个文件。

- **Cookie设置**

我们先尝试获取cookie

```python
import requests

r = requests.get("https://www.baidu.com")
print(r.cookies)
for key, value in r.cookies.items():
    print(key + "=" + value)
```

运行结果

```json
<RequestsCookieJar[<Cookie BDORZ=27315 for .baidu.com/>]>
BDORZ=27315
```

那么现在就可以看到这就是我们截获到的`Cookie`了，还记得之前讲的`Cookie`就像是通行证，所以我们先登录一个网站然后用生成的`Cookie`来替换`headers`参数看看是否可以登录访问

首先登录我的126邮箱，然后找到它的Cookie

![image-20240227230422686](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/image-20240227230422686.png)

接着复制并添加到`headers`参数里面

```python
import requests

url = "https://mail.126.com/js6/main.jsp?sid=SFxzVLEEsGskZribnYEEzCuuiOescwUt&df=webmail126#module=welcome.WelcomeModule%7C%7B%7D"
headers = {
    "Cookie": 'nts_mail_user=w1121293707@126.com:-1:1; MAIL_SDID=874273342558449664; locale=; face=js6; MAIL_LOCATION=gz; NTES_SESS=VK5sD5z7nl9YqEkNzX0aaCCq7XSU27vS_b.8k1DWElVGxaw1xYbLveT2wf2_R0Vbz.rvYIKJwo.8mWXUnz.FbeYBS_lfIRdiurUwSQfcyK7DQjl0LfBZoaE8GME9deTNM_sKVZtSL9hkhZxq4YM7OfHAmhxcUUSXO8MRPD9q7V.ucDKWsdmD0H6KOpTc564bWXG2brXGvoOSfcNZqodIxStAW; S_INFO=1709045762|0|#3&80#|w1121293707@126.com; P_INFO=w1121293707@126.com|1709045762|0|mail126|00&99|sxi&1708854921&mail126#sxi&610100#10#0#0|&0|mail126|w1121293707@126.com; mail_idc=""; Coremail=7e63eb7de5574%SFxzVLEEsGskZribnYEEzCuuiOescwUt%wmsvr-41-106.mail.126.com; MAIL_ENTRY_INFO=1|0|mail126|webmail126|124.89.2.73|2fd8275d91a844e31dcc47f615d9df3b_v1||874273342558449664|1709045763|0; MAIL_ENTRY_CS=4013070295d8ed2148c0900b8bd1e7c6; mixmailTokens=1E1StjlbA-RsybFpsvpXmtOyfRGomvqnWPmwQQZz1N_0Co3MVdUHWlLjvlUgHo2flCGcsF73ZG4eh1rF9fGYegI2Gy0PzxGZLwKjGJMF13PjrCLVEI6eeWBnYFOpdp07vYMl1h49M23HAS4ZnL-eaQMVg; MAIL_MISC="w1121293707@126.com"; cm_last_info=dT13MTEyMTI5MzcwNyU0MDEyNi5jb20mZD1odHRwcyUzQSUyRiUyRm1haWwuMTI2LmNvbSUyRmpzNiUyRm1haW4uanNwJTNGc2lkJTNEU0Z4elZMRUVzR3NrWnJpYm5ZRUV6Q3V1aU9lc2N3VXQmcz1TRnh6VkxFRXNHc2tacmlibllFRXpDdXVpT2VzY3dVdCZoPWh0dHBzJTNBJTJGJTJGbWFpbC4xMjYuY29tJTJGanM2JTJGbWFpbi5qc3AlM0ZzaWQlM0RTRnh6VkxFRXNHc2tacmlibllFRXpDdXVpT2VzY3dVdCZ3PWh0dHBzJTNBJTJGJTJGbWFpbC4xMjYuY29tJmw9LTEmdD0tMSZhcz10cnVl; MAIL_SESS=VK5sD5z7nl9YqEkNzX0aaCCq7XSU27vS_b.8k1DWElVGxaw1xYbLveT2wf2_R0Vbz.rvYIKJwo.8mWXUnz.FbeYBS_lfIRdiurUwSQfcyK7DQjl0LfBZoaE8GME9deTNM_sKVZtSL9hkhZxq4YM7OfHAmhxcUUSXO8MRPD9q7V.ucDKWsdmD0H6KOpTc564bWXG2brXGvoOSfcNZqodIxStAW; MAIL_SINFO="1709045762|0|#3&80#|w1121293707@126.com"; MAIL_PINFO="w1121293707@126.com|1709045762|0|mail126|00&99|sxi&1708854921&mail126#sxi&610100#10#0#0|&0|mail126|w1121293707@126.com"; secu_info=1; mail_entry_sess=0859964f9866bb8c26b8fb64bff6baf1ee1cbbe3527f054da14400f1a9bf1cfaf8d12c4f89f9f4c1566860bbab761a5056fa151a418041eb884f4c80ef24a6d9dfb5dc3e2dd1cc19edc67c72bffc666073f8920566b6444b98eb0d8f14e5986b58173ad7ba751674d600f8ac802f320cc96a28d33200651300cdd96817b6713f71d5559e9d5c1b6b0eb9e063aee25969ed8278858b298bdc143de44d3296d56e5e52e0fd3ceb24a633d8a1756d4c8d6459358118b3a5745f2a91dad340c4d430; JSESSIONID=E357320184AF7ACB567AE3CEE389544C; Coremail.sid=SFxzVLEEsGskZribnYEEzCuuiOescwUt; mail_style=js6; mail_uid=w1121293707@126.com; mail_host=mail.126.com; stats_session_id=32bf5635-1d0e-4200-a853-3b279a557c3b'
}
resp = requests.get(url, headers=headers)
print(resp.text)
```

运行结果

![image-20240227230541983](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/image-20240227230541983.png)

这就表明我们通过设置Cookie的信息，成功模拟了登录状态。、

除了设置headers参数外，也可以设置cookies参数。在前面实例中看到了我们截获的cookie是`RequestsCookieJar对象`类型，所以我们可以新建一个`RequestsCookieJar对象` ，然后赋值给`cookies`参数

```python
import requests

url = "https://mail.126.com/js6/main.jsp?sid=SFxzVLEEsGskZribnYEEzCuuiOescwUt&df=webmail126#module=welcome.WelcomeModule%7C%7B%7D"
cookies = 'nts_mail_user=w1121293707@126.com:-1:1; MAIL_SDID=874273342558449664; locale=; face=js6; MAIL_LOCATION=gz; NTES_SESS=VK5sD5z7nl9YqEkNzX0aaCCq7XSU27vS_b.8k1DWElVGxaw1xYbLveT2wf2_R0Vbz.rvYIKJwo.8mWXUnz.FbeYBS_lfIRdiurUwSQfcyK7DQjl0LfBZoaE8GME9deTNM_sKVZtSL9hkhZxq4YM7OfHAmhxcUUSXO8MRPD9q7V.ucDKWsdmD0H6KOpTc564bWXG2brXGvoOSfcNZqodIxStAW; S_INFO=1709045762|0|#3&80#|w1121293707@126.com; P_INFO=w1121293707@126.com|1709045762|0|mail126|00&99|sxi&1708854921&mail126#sxi&610100#10#0#0|&0|mail126|w1121293707@126.com; mail_idc=""; Coremail=7e63eb7de5574%SFxzVLEEsGskZribnYEEzCuuiOescwUt%wmsvr-41-106.mail.126.com; MAIL_ENTRY_INFO=1|0|mail126|webmail126|124.89.2.73|2fd8275d91a844e31dcc47f615d9df3b_v1||874273342558449664|1709045763|0; MAIL_ENTRY_CS=4013070295d8ed2148c0900b8bd1e7c6; mixmailTokens=1E1StjlbA-RsybFpsvpXmtOyfRGomvqnWPmwQQZz1N_0Co3MVdUHWlLjvlUgHo2flCGcsF73ZG4eh1rF9fGYegI2Gy0PzxGZLwKjGJMF13PjrCLVEI6eeWBnYFOpdp07vYMl1h49M23HAS4ZnL-eaQMVg; MAIL_MISC="w1121293707@126.com"; cm_last_info=dT13MTEyMTI5MzcwNyU0MDEyNi5jb20mZD1odHRwcyUzQSUyRiUyRm1haWwuMTI2LmNvbSUyRmpzNiUyRm1haW4uanNwJTNGc2lkJTNEU0Z4elZMRUVzR3NrWnJpYm5ZRUV6Q3V1aU9lc2N3VXQmcz1TRnh6VkxFRXNHc2tacmlibllFRXpDdXVpT2VzY3dVdCZoPWh0dHBzJTNBJTJGJTJGbWFpbC4xMjYuY29tJTJGanM2JTJGbWFpbi5qc3AlM0ZzaWQlM0RTRnh6VkxFRXNHc2tacmlibllFRXpDdXVpT2VzY3dVdCZ3PWh0dHBzJTNBJTJGJTJGbWFpbC4xMjYuY29tJmw9LTEmdD0tMSZhcz10cnVl; MAIL_SESS=VK5sD5z7nl9YqEkNzX0aaCCq7XSU27vS_b.8k1DWElVGxaw1xYbLveT2wf2_R0Vbz.rvYIKJwo.8mWXUnz.FbeYBS_lfIRdiurUwSQfcyK7DQjl0LfBZoaE8GME9deTNM_sKVZtSL9hkhZxq4YM7OfHAmhxcUUSXO8MRPD9q7V.ucDKWsdmD0H6KOpTc564bWXG2brXGvoOSfcNZqodIxStAW; MAIL_SINFO="1709045762|0|#3&80#|w1121293707@126.com"; MAIL_PINFO="w1121293707@126.com|1709045762|0|mail126|00&99|sxi&1708854921&mail126#sxi&610100#10#0#0|&0|mail126|w1121293707@126.com"; secu_info=1; mail_entry_sess=0859964f9866bb8c26b8fb64bff6baf1ee1cbbe3527f054da14400f1a9bf1cfaf8d12c4f89f9f4c1566860bbab761a5056fa151a418041eb884f4c80ef24a6d9dfb5dc3e2dd1cc19edc67c72bffc666073f8920566b6444b98eb0d8f14e5986b58173ad7ba751674d600f8ac802f320cc96a28d33200651300cdd96817b6713f71d5559e9d5c1b6b0eb9e063aee25969ed8278858b298bdc143de44d3296d56e5e52e0fd3ceb24a633d8a1756d4c8d6459358118b3a5745f2a91dad340c4d430; JSESSIONID=E357320184AF7ACB567AE3CEE389544C; Coremail.sid=SFxzVLEEsGskZribnYEEzCuuiOescwUt; mail_style=js6; mail_uid=w1121293707@126.com; mail_host=mail.126.com; stats_session_id=32bf5635-1d0e-4200-a853-3b279a557c3b'
jar = requests.cookies.RequestsCookieJar()
for cookie in cookies.split(';'):
    key, value = cookie.split('=', 1)
    jar.set(key, value)
resp = requests.get(url, cookies=jar)
print(resp.text)
```

运行结果![image-20240227231829663](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/image-20240227231829663.png)

测试发现，依然可以正常登录

- **Session维持**

在利用`requests`库时，我们无论使用`GET`还是`POST`请求，都是当作开启了一个新的浏览器，然后去访问网站的。比如你先用`POST`请求输入了你的登录信息，之后你再使用`GET`请求还是会无法访问。

```python
import requests

requests.get("https://www.httpbin.org/cookies/set/number/123456789")
resp = requests.get("https://www.httpbin.org/cookies")
print(resp.text)
```

运行结果

```json
{
  "cookies": {}
}
```

所以一个解决办法就是像上面一节的例子，设置`cookie`参数即可，但这样比较麻烦。因为你需要先去浏览器上登录网站，然后再复制cookie头的内容。

那么如果不想重复复制粘贴cookie内容的话，我们可以利用session对象，只要维护好一个session对象，cookie也就不成问题了。

```python
import requests

s = requests.session()
s.get("https://www.httpbin.org/cookies/set/number/123456789")
resp = s.get("https://www.httpbin.org/cookies")
print(resp.text)
```

运行结果

```json
{
  "cookies": {
    "number": "123456789"
  }
}
```

- **SSL证书验证**

现在很多网站都用的https协议，但是有些网站没有安装HTTPS证书或者HTTPS证书不被ca机构认可，这就导致我们在访问的时候可能会出现SSL证书错误的提示。

比如我们现在访问`https://ssr2.scrape.center`

![image-20240228221757520](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/image-20240228221757520.png)

但是我们用`requests`库的话，就会无法访问

```python
import requests

resp = requests.get("https://ssr2.scrape.center")
print(resp.text)
```

运行结果如下

![image-20240228222432050](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/image-20240228222432050.png)

可以看见它抛出了SSLError错误

那么如果我们一定要访问该网站呢，可以使用`verify`参数，这个参数使用来控制证书的，默认为true，如果将它设置为false，那么在请求时就不会再验证证书。

```python
import requests

resp = requests.get("https://ssr2.scrape.center/", verify=False)
print(resp.status_code)
```

运行结果

```
D:\python\lib\site-packages\urllib3\connectionpool.py:1045: InsecureRequestWarning: Unverified HTTPS request is being made to host 'ssr2.scrape.center'. Adding certificate verification is strongly advised. See: https://urllib3.readthedocs.io/en/1.26.x/advanced-usage.html#ssl-warnings
  warnings.warn(
200
```

状态码为200，这就是表示我们已经访问成功了。

不过呢，上面还是有一段警告，它是建议我们给他指定证书。如果我们想要忽略它的话可以通过设置忽略警告的方式来屏蔽这个警告。

```python
import requests
from requests.packages import urllib3

urllib3.disable_warnings()
resp = requests.get("https://ssr2.scrape.center", verify=False)
print(resp.status_code)
```

运行结果

```
200
```

可以看到那段长长的警告不见了

还有两种方法可以忽略警告

捕获警告到日志的方式忽略警告：

```python
import requests
import logging

logging.captureWarnings(True)
resp = requests.get("https://ssr2.scrape.center", verify=False)
print(resp.status_code)
```

运行结果

```
200
```

或者我们可以直接指定一个本地证书作为客户端证书

```python
import requests

resp = requests.get("https://ssr2.scrape.center", cert=('/path/server.crt', '/path/server.key'))
print(resp.status_code)
```

运行结果

```
200
```

- **超时设置**

平常在上网的时候就可能会因为网络状况不好或者服务器网络响应太慢导致需要等待一段时间网站才能加载出来，甚至最后因为接收不到响应而报错。

所以我们可以通过设置超时时间，就是如果加载到一定时间还是没有响应，就直接跳过。

```python
import requests

resp = requests.get("http://www.httpbin.org", timeout=1)
print(resp.text)
```

但是这里的`timeout`参数是连接和读取的时间总和，如果想要分别设置连接和读取的时间，可以用元组类型

```python
import requests

resp = requests.get("http://www.httpbin.org", timeout=(5, 10))
print(resp.text)
```

- **身份认证 （OAUTH未）**

在访问某些网站的时候，通常会弹出登录弹窗，进行身份认证，比如访问`https://ssr3.scrape.center`时，就会出现一个登录弹窗

![image-20240229183300080](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/image-20240229183300080.png)

怎么解决这个问题呢，要知道`requests`模块是一个强大的库，它里面可以通过设置auth参数来通过身份认证。

```python
import requests
from requests.auth import HTTPBasicAuth

resp = requests.get("https://ssr3.scrape.center", auth=HTTPBasicAuth("admin", "admin"))
print(resp.status_code)
```

运行结果

```
200
```

除此之外，`requests`库还提供其他认证方式，比如`OAuth`认证

```python
import requests
from requests_oauthlib import OAuth1

url = "https://api.twitter.com/1.1/account/verify_credentials.json"
auth = OAuth1("YOUR_APP_KEY", "YOUR_APP_SECRET", "USER_OAUTH_TOKEN", "USER_OAUTH_TOKEN_SECRET")
resp = requests.get(url, auth=auth)
print(resp.status_code)
```

运行结果

```json
200
```

- **代理设置**

经常对某个网站进行爬虫时，前几次可以好好访问，但后面就会出现访问失败的情况。这就是网站的反爬技术，它们如果检测到某个IP在进行不正常访问时，就会判断其为爬虫，然后就会将该IP加入黑名单，这样就无法继续访问了。

如何解决这个问题呢，既然不允许一个IP多次访问的话，那么我们用多个IP依次去访问时，不就可以通过时间差使之IP在正常访问。

这时我们就要用到`proxies`参数。下面展示例子：

```python
import requests

proxies = {
    "http": "http://10.10.10.10:1080",
    "https": "http://10.10.10.10:1080"
}
requests.get("https://httpbin.org/get", proxies=proxies)
```

这个例子无法执行，如果要利用的话还是要去搜索有效的代理IP。

同时requests模块还支持`SOCK`协议

```
import requests

proxies = {
    "http": "sock5://10.10.10.10:1080",
    "https": "sock5://10.10.10.10:1080"
}
requests.get("https://httpbin.org/get", proxies=proxies)
```

- **Prepare Request**

我们知道使用get、post方法就可以实现向其他网站的请求动作，那么它们在requests模块的内部是怎么实现的呢？

在发送请求之前，requests模块构造一个Request对象，然后将url、header、data等信息添加进去，最终将这个Request对象发送出去，在请求成功后就会获取Response对象，接下来就是对Response对象进行解析。Request对象的类型就是Prepare Request

接下来我们尝试不使用get方法发送一条请求

```python
from requests import Request, Session

url = "https://httpbin.org/post"
data = {"name": "winglet"}
headers = {
    "User-Agent": "Mozilla / 5.0(Windows NT 10.0; Win64; x64) AppleWebKit / 537.36(KHTML, like Gecko) Chrome / 80.0.3987.122  Safari / 537.36"
}
s = Session()
req = Request('POST', url=url, data=data, headers=headers)
prepped = s.prepare_request(req)
r = s.send(prepped)
print(r.text)
```

运行结果如下

```json
{
  "args": {}, 
  "data": "", 
  "files": {}, 
  "form": {
    "name": "winglet"
  }, 
  "headers": {
    "Accept": "*/*", 
    "Accept-Encoding": "gzip, deflate", 
    "Content-Length": "12", 
    "Content-Type": "application/x-www-form-urlencoded", 
    "Host": "httpbin.org", 
    "User-Agent": "Mozilla / 5.0(Windows NT 10.0; Win64; x64) AppleWebKit / 537.36(KHTML, like Gecko) Chrome / 80.0.3987.122  Safari / 537.36", 
    "X-Amzn-Trace-Id": "Root=1-65f2cab2-1df53d590a48a2df49f2fa16"
  }, 
  "json": null, 
  "origin": "92.223.121.116", 
  "url": "https://httpbin.org/post"
}
```

可以看到，达到了和POST请求一样的效果。而这个Request对象已经成为一个独立的对象了，所以我们可以直接操作它。

## 正则表达式（re）

- **简单介绍**

现在给出一条数据

```
my phone number is 010-86432100
```

如果我们现在只想要电话号码的话，就需要编写一条正则表达式

![image-20240314200515989](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/image-20240314200515989.png)

当然这个只是一个比较简单的正则匹配，但效果大家都看见了，我们可以通过一条正则表达式，来匹配一段数据中我们关注的部分。

### match

这里先介绍第一个常用的匹配方法——match，我们只需要像它传入要匹配的字符串以及正则表达式，他就可以检测这个正则表达式是否与字符串相匹配。

```python
import re

content = "Hello 123 4567 world_This is a Regex Demo"
print(len(content))
result = re.match('^Hello\s\d\d\d\s\d{4}\s\w{10}', content)
print(result)
print(result.group())       // 输出匹配到的内容
print(result.span())        // 输出匹配的范围
```

运行结果

```json
41
<re.Match object; span=(0, 25), match='Hello 123 4567 world_This'>
Hello 123 4567 world_This
(0, 25)
```

- **匹配目标**

我们在上面实现了用match匹配内容，那么现在我们要的就是只取出字符串的一部分。

```python
import re

content = "Hello 1234567 World_This is a Regex Demo"
print(len(content))
result = re.match('^Hello\s+(\d+)\s+World', content)
print(result)
print(result.group())
print(result.group(1))  // 会输出完整的匹配结果
print(result.span())
```

运行结果

```json
40
<re.Match object; span=(0, 19), match='Hello 1234567 World'>
Hello 1234567 World
1234567
(0, 19)
```

- **通用匹配**

通过上面例子可以发现我们匹配时总是要写很多`\d \s`之类的，其实在正则表达式中式存在能全部匹配的表达式的。就是即将介绍的`.*`

```
.  可以匹配任意字符
*  可以匹配前面的字符无限次
但是当它们组合到一起时，就能够实现匹配任意字符了
```

我们现在稍微修改一下之前的代码

```python
import re

content = "Hello 1234567 World_This is a Regex Demo"
print(len(content))
result = re.match('^Hello.*Demo', content)
print(result)
print(result.group())
print(result.span())
```

运行结果

```json
40
<re.Match object; span=(0, 40), match='Hello 1234567 World_This is a Regex Demo'>
Hello 1234567 World_This is a Regex Demo
(0, 40)
```

- **贪婪与非贪婪**

我们有时在使用`.*`时常常得不到我们想要的结果。比如现在我们想要匹配中间的所有数字

```python
import re

content = "Hello 1234567 World_This is a Regex Demo"
result = re.match("^He.*(\d+).*Demo$", content)
print(result)
print(result.group(1))
```

运行结果

```json
<re.Match object; span=(0, 40), match='Hello 1234567 World_This is a Regex Demo'>
7
```

我们只匹配到了一个`7`数字，这显然不是我们想要的结果。

这里就要涉及贪婪和非贪婪匹配机制了。在贪婪匹配下,`.*`会尽可能匹配多的字符。以上面的表达式为例`.*`后面的是`\d+`，它至少会匹配一个数字。所以在我们没有指定几个数字的情况下，`.*`就会尽可能地匹配多的字符，这就导致`123456`都被匹配了，只留了`7`给`\d+`去匹配，所以我们最后得到的只有一个`7`，那么这时我们就要用到非贪婪模式了，它的写法就是`?`

```python
import re

content = "Hello 1234567 World_This is a Regex Demo"
result = re.match("^He.*?(\d+).*Demo$", content)
print(result)
print(result.group(1))
```

运行结果

```json
<re.Match object; span=(0, 40), match='Hello 1234567 World_This is a Regex Demo'>
1234567
```

可以看见我们只是在`.*`后面加了一个`?`，就可以使它只匹配到`1`之前。在非贪婪模式下，`.*?`会尽可能匹配少的字符，所以当它匹配到`1`前面的空格符后，就会把数字全交给`\d+`去进行匹配，自己就歇着去了。`.*?`经常会被用在中间进行匹配。

要注意的一点就是`.*?`如果放在字符串的结尾，就有可能匹配不到任何内容了。

```python
import re

content = "http://weibo.com/comment/kEraCN"
result1 = re.match("http.*?comment/(.*?)", content)
result2 = re.match("http.*?comment/(.*)", content)
print(result1.group())
print(result2.group())
```

运行结果

```
http://weibo.com/comment/
http://weibo.com/comment/kEraCN
```

- **修饰符**

先举个例子

```python
import re

content = '''Hello 1234567 World_this
is a Winglet Demo
'''
result = re.match('^He.*?(\d+).*?Demo$', content)
print(result.group(1))
```

运行结果

```
Traceback (most recent call last):
  File "D:\pyproject\爬虫\测试.py", line 7, in <module>
    print(result.group(1))
AttributeError: 'NoneType' object has no attribute 'group'
```

这里因为字符串中出现了换行符导致无法匹配，但是只要我们在后面加上一个`re.S`，就可以解决这个错误。

```python
import re

content = '''Hello 1234567 World_this
is a Winglet Demo
'''
result = re.match('^He.*?(\d+).*?Demo$', content, re.S)
print(result.group())
```

运行结果

```json
Hello 1234567 World_this
is a Winglet Demo
```

除此之外还有一些别的修饰符

| 修饰符 | 描述                                                    |
| ------ | ------------------------------------------------------- |
| re.I   | 使匹配对大小写不敏感                                    |
| re.M   | 多行匹配，影响 ^ 和 $                                   |
| re.L   | 实现本地化识别 (locale-aware) 匹配                      |
| re.S   | 使匹配内容包括换行符在内的所有字符                      |
| re.U   | 根据Unicode字符集解析字符。这个标志会影响\w、\W、\b和\B |
| re.X   | 该标志能给予你更灵活的格式，以便将正则变大时            |

常用的就是`re.S`和`re.I`

- **转义匹配**

举个例子

```python
import re

content = "(百度) www.baidu.com"
result = re.match('\(百度\) www\.baidu\.com', content)
print(result)
```

运行结果

```json
<re.Match object; span=(0, 18), match='(百度) www.baidu.com'>
```

可以看见我们如果想要匹配一些符号的话就可以在前面加上`\`，进行字符转义

### search

`match`方法是从字符串的第一个字符开始匹配的，所以如果一旦第一个字符就匹配失败了，那么这整个字符串就无法匹配。而我们一般都是会匹配中间部分重要的信息，所以这里就要介绍到一个新的方法`search`

这个方法会先扫描整个字符串，然后会从第一个匹配到的字符开始匹配

例子如下

```python
import re

content = "Extra string Hello 1234567 World_This is a Regex Demo Extra       stings"
result = re.search('Hello.*?(\d+).*?Demo', content)
print(result.group())
```

运行如下

```json
Hello 1234567 World_This is a Regex Demo
```

## 操作文件

### OS

#### os.walk()

```python
os.walk(top, topdown=True, onerror=None, followlinks=False)
```

**参数说明：**

- `top`: 要遍历的根目录路径
- `topdown`: `True` 时自上而下遍历，`False` 时自下而上
- `onerror`: 错误处理函数
- `followlinks`: 是否跟随符号链接

**返回值：** 生成器，每次迭代返回一个元组 `(dirpath, dirnames, filenames)`

**基本用法**

```python
import os

# 遍历当前目录
for root, dirs, files in os.walk('.'):
    print(f"当前目录: {root}")   # str
    print(f"子目录: {dirs}")	 # list
    print(f"文件: {files}")	 # list
```

## 程序打包

### pyinstaller

#### 安装

```shell
pip install pyinstaller
```

#### 使用方式

```shell
pyinstaller --onefile --windowed --name "WingletTools" winglet工具集.py 
```

- `--onefile`：生成单个exe文件
- `--windowed`：禁止后台控制台窗口（适合GUI程序）
- `--name`：指定exe名称

打包成功后，在 `dist` 文件夹中找到 `winglet工具集.exe`。`demo.py` 会自动被包含进去，无需额外操作。

```shell
pyinstaller --onefile --windowed --icon=my.ico --add-data "demo.py;." --name "winglet工具集" winglet工具集.py
```

- `--icon`：设置exe图标
- `--add-data "demo.py;."`：手动把demo.py打包到根目录（一般不需要，因为导入会自动处理）
