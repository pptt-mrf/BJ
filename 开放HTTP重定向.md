# 开放HTTP重定向

## 1.定义

开放 HTTP 重定向（Open Redirect）是 Web 应用常见漏洞，指应用接收用户可控的 URL 参数并直接用于重定向，未做严格校验，导致

攻击者可诱导用户跳转到恶意站点，常作为钓鱼攻击、会话劫持的关键入口。

开放重定向 = 谁都能改跳转地址，网站还不拦着，直接放行。

开放 HTTP 重定向 = 不加验证的用户可控跳转

## 2.dvwa的低级防御

1.由于低级防御没有任何防御，所有直接跳转

```url
http://192.168.144.1/dvwa/vulnerabilities/open_redirect/source/low.php?redirect=http://www.baidu.com
```

可以成功

2.但是要找对跳转位置

```url
http://192.168.144.1/dvwa/vulnerabilities/open_redirect/source/info.php?redirect=http://www.baidu.com
```

跳转失败

为什么？

因为info.php?与low.php?不同：info.php是接口它的功能就不是跳转，它是用来接受功能的接口，所以你拼接跳转命令，info.php无法识别。

## 3.DVWA的中级防御

1.防御

```php
if (preg_match ("/http:\/\/|https:\/\//i", $_GET['redirect'])) 
```

过滤了协议头http:和https:

2.绕过

```php
http://192.168.144.1/dvwa/vulnerabilities/open_redirect/source/low.php?redirect=//www.baidu.com
```

如果没有明确指定协议，直接以 `//` 开头，则表示使用和当前页面相同的协议，便可以绕过了

#### 4.url的特殊规则，浏览器的语法。

##### 1.遇到 URL 以 `//` 开头 → 自动继承当前页面的 http 或 https

##### 2. `@` 用户名欺骗（超级常用）

plaintext

```url
http://合法网站@恶意网站.com
```

**浏览器会忽略 @ 前面的所有内容，只访问 @ 后面的域名！**

例子：

plaintext

```url
http://dvwa.com@evil.com
```

浏览器实际访问：**[evil.com](https://evil.com)**

后端正则如果判断 `contains(dvwa.com)`，就会被完美绕过。

##### 3. 数字 IP 格式（绕过域名白名单）

把域名转成纯数字：

plaintext

```url
http://111.13.101.208 → 百度
```

写成数字：

plaintext

```url
http://1869871824
```

后端不认识这串数字，但浏览器能正常访问。

##### 4. 斜线叠加 `///`

有些后端判断必须以 `/` 开头，但不判断数量：

plaintext

```url
///baidu.com
```

浏览器会自动解析成：

plaintext

```url
//baidu.com
```

直接跳走。

##### 5. 问号绕过后端判断

后端判断是否是 “内部路径”：

plaintext

```url
/baidu.com?http://dvwa.com
```

后端以为是内部路径

浏览器只看问号前面，跳去 baidu

##### 6. 反斜杠 `\`

浏览器会自动把 `\` 变成 `/`

plaintext

```url
\\baidu.com
```

变成

plaintext

```url
//baidu.com
```

##### 7. 子域欺骗

plaintext

```url
http://evil.com?dvwa.com
```

后端看到 [dvwa.com](https://dvwa.com) 以为安全

浏览器只看域名部分（只看？之前），访问 evil

##### 8. 编码绕过（双重 URL 编码）

后端解码一次，浏览器再解码一次

plaintext

```url
http:%252F%252Fbaidu.com
```

最终变成

plaintext

```url
http://baidu.com
```

## 4.DVWA的高级防御

1.防御:看URL中是否包含"info.php"

```php
<?php
// 检查URL中是否存在'redirect'参数，并且该参数不为空。
if (array_key_exists("redirect", $_GET) && $_GET['redirect'] != "") {
    // 检查'redirect'参数中是否包含"info.php"。
    if (strpos($_GET['redirect'], "info.php") !== false) {
        // 如果包含"info.php"，则进行重定向。
        header("location: " . $_GET['redirect']);
        exit; // 终止脚本执行
    } else {
        // 如果不包含"info.php"，返回HTTP 500状态码和错误信息。
        http_response_code(500);
        ?>
        <p>You can only redirect to the info page.</p>
        <?php
        exit; // 终止脚本执行
    }
}
// 如果'redirect'参数不存在或为空，则返回HTTP 500状态码并显示缺少重定向目标的错误信息。
http_response_code(500);
?>
<p>Missing redirect target.</p>
<?php
exit; // 终止脚本执行
?>

```

2.绕过：

```url
http://192.168.144.1/dvwa/vulnerabilities/open_redirect/source/low.php?redirect=http://www.baidu.com?id=info.php
```

因为浏览器的访问规则所以会直接跳转