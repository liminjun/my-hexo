---
title: 前后端分离下如何完成Web登录
date: 2021-08-08 12:18:04
categories: ["前端"]
tags: ["登录","前端"]
---
### 1 Web登录涉及到知识点
#### 1.1 HTTP无状态性
>HTTP是无状态的，一次请求结束，连接断开，下次服务器再收到请求，它就不知道这个请求是哪个用户发过来的。当然它知道是哪个客户端地址发过来的，但是对于我们的应用来说，我们是靠用户来管理，而不是靠客户端。所以对我们的应用而言，它是需要有状态管理的，以便服务端能够准确的知道http请求是哪个用户发起的，从而判断该用户是否有权限继续这个请求。这个过程就是常说的**会话管理**。

#### 1.2 登录流程
登录的基本流程
![web-login-flow](https://images2018.cnblogs.com/blog/59618/201808/59618-20180816092611781-493556035.png)

### 2 同域登录
目前大多数Web应用采用前后端分离方式进行开发。所以前端网站或应用都属于SPA（Single Page Application）。如果前端，后台API部署在同域下，不存在跨域的情况，登录方式相对简单。

#### 2.1 基于Session登录
服务器端使用Session技术，浏览器端使用Cookie技术。

![session-based-login](https://images2015.cnblogs.com/blog/459873/201611/459873-20161115231400951-1095594983.png)

1. 服务端session是用户第一次访问应用时，服务器就会创建的对象，代表用户的一次会话过程，可以用来存放数据。服务器为每一个session都分配一个唯一的sessionid，以保证每个用户都有一个不同的session对象。

2. 服务器在创建完session后，会把sessionid通过cookie返回给用户所在的浏览器，这样当用户第二次及以后向服务器发送请求的时候，就会通过cookie把sessionid传回给服务器，以便服务器能够根据sessionid找到与该用户对应的session对象。

3. session通常有失效时间的设定，比如2个小时。当失效时间到，服务器会销毁之前的session，并创建新的session返回给用户。但是只要用户在失效时间内，有发送新的请求给服务器，通常服务器都会把他对应的session的失效时间根据当前的请求时间再延长2个小时。

4. session在一开始并不具备会话管理的作用。它只有在用户登录认证成功之后，并且往sesssion对象里面放入了用户登录成功的凭证，才能用来管理会话。管理会话的逻辑也很简单，只要拿到用户的session对象，看它里面有没有登录成功的凭证，就能判断这个用户是否已经登录。当用户主动退出的时候，会把它的session对象里的登录凭证清掉。所以在用户登录前或退出后或者session对象失效时，肯定都是拿不到需要的登录凭证的。

#### 2.2 基于Token登录
![token-based-login](https://images2015.cnblogs.com/blog/459873/201611/459873-20161120210044154-648255641.png)

1. 用户在浏览器中输入用户和密码，后台服务器通过加密或者其他逻辑，生成一个Token。
2. 前端获取到Token，存储到cookie或者localStorage中，在接下来的请求中，将token通过url参数或者HTTP Header头部传入到服务器
3. 服务器获取token值，通过查找数据库判断当前token是否有效

基于Token登录，而且可以用于第三方单点登录的OAuth2.0更适合。可以参考网址：[理解OAuth 2.0](http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)

token传入示例：
![by-cookie](https://images2018.cnblogs.com/blog/59618/201808/59618-20180818023625525-463599259.png)

![by-http-header](https://images2018.cnblogs.com/blog/59618/201808/59618-20180818023643013-2141566824.png)



### 3 Cookie的传输
简单地说，cookie 就是浏览器储存在用户电脑上的一小段文本文件。cookie 是纯文本格式，不包含任何可执行的代码。一个 Web 页面或服务器告知浏览器按照一定规范来储存这些信息，**并在随后的请求中将这些信息发送至服务器**，Web 服务器就可以使用这些信息来识别不同的用户。大多数需要登录的网站在用户验证成功之后都会设置一个 cookie，只要这个 cookie 存在并可以，用户就可以自由浏览这个网站的任意页面。再次说明，cookie 只包含数据，就其本身而言并不有害。

**同域情况下，Cookie会在随后的请求中携带**

### 4 跨域登录
**跨越定义** :由于浏览器同源策略，凡是发送请求的url的协议(http和https)、域名（www.example.com,about.example.com）、端口(8010和8020)三者之间任意一个与当前页面地址不同则视为跨域。

#### 4.1 解决同源策略

基于Session和Token登录都要解决。

[浏览器的同源策略](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy)

如果使用同域的方法，那么浏览器会抛出如下错误。demo示例，前端运行在`http://localhost:8010/login.html`，后台运行在`http://localhost:8020/api/login.php`

![Access-Control-Allow-Origin](https://images2018.cnblogs.com/blog/59618/201808/59618-20180818025037687-514830877.png)

需要在服务器端设置`Header`，以PHP为例：
```
header('Access-Control-Allow-Origin: http://localhost:8010');
```
设置完成之后，可以发送请求了，登录成功之后跳转到`home.html`还是显示未登录，会跳转到`login.html`页面。效果如下图：

#### 4.2 解决请求带上Cookie信息
基于Session登录才需要，因为相关信息是通过Cookie传入，如果是通过url传入，也不需要解决这个。基于Token，后续请求携带token都是通过header里面的字段，所以也不需要解决这个。

跨越情况下，浏览器此时不会默认在后续请求里面携带上`Cookie`信息，这个时候前后端都需要设置。以`jQuery`和`PHP`为列。

前端`jQuery`代码
Ajax请求中药设置`xhrFields`
```
xhrFields: {
        withCredentials: true
    }
```
完整代码如下：
```
$.ajax({
    url: "http://localhost:8020/api/login.php",
    type: "POST",
    data: {
        username: $("#username").val(),
        password: $("#password").val()
    },
    dataType: "json",
    xhrFields: {
        withCredentials: true
    }
}).done(function (response) {
    debugger;
    $("#log").html(response.message);
    window.location.href = "home.html";
}).fail(function (jqXHR, textStatus) {
    console.log("Request failed: " + textStatus);
});
```
后端`php`代码
```
/*需要设置这一行，接收传入Credentials字段*/
header('Access-Control-Allow-Credentials: true');  
header('Access-Control-Allow-Origin: http://localhost:8010');
```

Demo代码下载：https://share.weiyun.com/5Nm46eU

### 参考链接：
[HTTP无状态协议和cookie、session原理](https://segmentfault.com/a/1190000009518499)

[HTTP cookies 详解](http://bubkoo.com/2014/04/21/http-cookies-explained/)
[3种web会话管理的方式](http://www.cnblogs.com/lyzg/p/6067766.html)

[你会做WEB上的用户登录功能吗？](https://coolshell.cn/articles/5353.html)

[登录工程：现代Web应用中的身份验证技术](http://insights.thoughtworkers.org/web-app-authentication/)

[angular和jquery的 withCredentials用法](https://www.jianshu.com/p/e613a00510dd)