# JavaScript攻击

定义：简单来说，**JavaScript 攻击就是利用前端 JS 代码的设计缺陷，绕过限制、篡改数据或执行未授权操作**。它不是一种单一漏洞，而是一类**依赖前端逻辑、信任客户端输入**的安全问题统称。

## 1.低级防御：

低级没有防御，只有代码加载时的一些默认规则

1.generate_token();

（1）页面刚加载的时候，算一次 token。

2.低级的漏洞让你提交成功的英文单词，然后，同时，提交英文成功单词的token，两者一致就能绕过，但是因为这个generate_token();函数的原因，你刚刚点进这个页面，它函数就会用默认值先算一遍，等你后面输入成功的英文以后，它也不会再进行第二次计算，导致你一点提交，交的是默认值的tokyo，然后和成功的英文单词，两者对不上，所以也不成功

3.绕过：BP直接一个抓包，然后把tokyo改成成功英文的，把输入改成成功英文的，然后一提交就成功了

## 2.中极防御

1.生成token的方式和低级的不同，但是绕过方式一样

## 3.高级防御

1.生成token的方式和低级的不同，但是绕过方式一样

## 4.重新理一遍思路，

他这个漏洞其实想让你理解后端生成token的方式，嗯，但又不能真放到后端，要不然你就看不见了，啊，所以他就给你放到了前端，但是放到前端Token就失去了它的防御作用，就导致三个漏洞一模一样

## 5.token

1.定义：Token 就是一个 “临时通行证 / 暗号”

2.它的作用机理（流程）

**你输账号密码 → 发给服务器**

**服务器验证正确**

**服务器自己生成一串很长、随机、不可伪造的字符串 → 这就是 Token**

服务器把这个 Token **发给你**

你之后每次请求，都带上这个 Token

服务器检查 Token 是否有效、过期、被篡改

有效 → 放行；无效 → 让你重新登录

## 6.DVWA生成token

1.低级

```php
function generate_token() {
    // 1. 获取输入框里的文字
    var phrase = document.getElementById("phrase").value;

    // 2. 先 rot13，再 md5，塞进 token 框
    document.getElementById("token").value = md5(rot13(phrase));
}
```

先 rot13，再 md5，塞进 token 框

2.中级

```php
function do_elsesomething(e) {
    document.getElementById("token").value = 
        do_something(
            e + document.getElementById("phrase").value + "XX"
        );
}
```

  do_something  value + "XX":拼接：`XXsuccessXX`

 document.getElementById("token").value  反转：`XXsseccusXX`

3.高级

*high 和 medium 的漏洞原理完全一样，

只是 high 把代码 “化妆” 了，

加了 SHA256 库迷惑你，

把生成逻辑藏深了，

但本质还是：前端拼接 + 反转生成 token，可抓包伪造。**

（人话翻译就是和中级的一样，只不过加了一堆垃圾代码，让你看起来更高大上，然后其实一点用没有）
