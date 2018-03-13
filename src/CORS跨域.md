---
title: CORS跨域
date: 2017-03-25 15:41:42
tags: 
    - CORS
---

当访问其他域名或者同一域名不同端口上的资源时，就会产生跨域请求。而跨域请求确实是发起了，但是返回的结果却被浏览器拦截了，请求必须遵循的[同源策略](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy)。但是在工作中不可避免的会进行跨域访问。现在用的最多的就是Jsonp和cors但是jsonp只能发起get请求，而且携带数据量较小，如果要发起post请求，那么就要采用cors方案。
<!-- more -->

## 允许跨域场景

跨源资源共享标准( cross-origin sharing standard ) 允许以下场景可以发起跨站 HTTP 请求：

- 使用 XMLHttpRequest 或 Fetch发起跨站 HTTP 请求。
- Web 字体 (CSS 中通过 @font-face使用跨站字体资源)
- WebGL 贴图
- 使用drawImage 将 Images/video 画面绘制到canvas.
- 样式表（使用 CSSOM）
- Scripts (未处理的异常)

cors通过增加一些HTTP头，让服务器声明哪些源可以通过浏览器访问该服务器上的资源。其实也就是设置'Access-Control-Allow-Origin','\*'这个响应头，当然这里是允许所有来源，我们也可以具体指定。注意设置了credentials = true,就不能设置'Access-Control-Allow-Origin', '*',会报错。

## 预请求

一般发起的跨域请求都会先发送预请求，服务器允许后，浏览器才会以真正的HTTP请求方式发送请求，这是出于服务器的安全。那么如果同时满足一下几种情况是不会发送预请求（请求方式为OPTIONS，这是HTTP1.1中新增的方式）的。

### 请求方式必须是GET、POST、HEAD

### 请求头除了是用户代理自动设置（例如：Connection, User-Agent）和如下请求头

- Accept
- Accept-Language
- Content-Language
- Content-Type (but note the additional requirements below)
- DPR
- Downlink
- Save-Data
- Viewport-Width
- Width

### Content-type必须要是如下值

- application/x-www-form-urlencoded
- multipart/form-data
- text/plain

**注意上面三个条件必须是同时满足**

例如POST请求，但是如果Content-type:application/json那么就会发起预请求。

## 示例

域为http://foo.example向域为http://bar.other的发起请求

```js
var invocation = new XMLHttpRequest();
var url = 'http://bar.other/resources/public-data/';
   
function callOtherDomain() {
  if(invocation) {    
    invocation.open('GET', url, true);
    invocation.onreadystatechange = handler;
    invocation.send(); 
  }
}
```

![示意图](https://mdn.mozillademos.org/files/14293/simple_req.png)

```
GET /resources/public-data/ HTTP/1.1
Host: bar.other
User-Agent: Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10.5; en-US; rv:1.9.1b3pre) Gecko/20081130 Minefield/3.1b3pre
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Accept-Charset: ISO-8859-1,utf-8;q=0.7,*;q=0.7
Connection: keep-alive
Referer: http://foo.example/examples/access-control/simpleXSInvocation.html
Origin: http://foo.example


HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 00:23:53 GMT
Server: Apache/2.0.61 
Access-Control-Allow-Origin: *
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Transfer-Encoding: chunked
Content-Type: application/xml
```

我们可以看到响应头中设置了`Access-Control-Allow-Origin: *`，那么此时是能够正常拿到数据的。


## 携带cookies

一般而言，对于跨站请求，浏览器是不会发送凭证信息的。但如果将XMLHttpRequest的一个特殊标志位设置为true，浏览器就将允许该请求的发送。

```js
var invocation = new XMLHttpRequest();
var url = 'http://bar.other/resources/credentialed-content/';
    
function callOtherDomain(){
  if(invocation) {
    invocation.open('GET', url, true);
    invocation.withCredentials = true;
    invocation.onreadystatechange = handler;
    invocation.send(); 
  }
```

这里只是设置了一个标志位，并不是一个请求头，所以这是一个简单请求，是不会发送预请求的。但是在服务器端一定要设置`Access-Control-Allow-Credentials:true`,不然浏览器不会把服务器端的结果返回给发起请求的script脚本，浏览器会报错。

## 发起跨域请求时，服务器返回的部分HTTP响应头

- Access-Control-Allow-Origin    设置允许的客户端代理。**注意当设置值为\*时，就不能在客户端设置withCredentials为TRUE**
- Access-Control-Expose-Headers：设置浏览器可以的访问到的服务器头信息的白名单，如果设置，那么在浏览器端是可以看到服务器设置了哪些头信息。
> Access-Control-Expose-Headers: X-My-Custom-Header, X-Another-Custom-Header
- Access-Control-Max-Age：允许这个预请求可以缓存的时间，在设置时间内，将不再发送这个预请求
- Access-Control-Allow-Credentials：设置服务器响应数据是否可以被`credentials`设置为TRUE的客户端拿到，如果为FALSE，那么响应数据客户端script脚本是拿不到的。
- Access-Control-Allow-Methods：设置客户端发起请求的方式

## HTTP 请求头

- origin ：表明发起跨域请求的域
- Access-Control-Request-Method ：这个是在发送预请求时，告诉服务器，真实请求的请求方式。
- Access-Control-Request-Headers ：当发送预请求时，告诉服务器，发送真实请求时，所包含的请求头信息。

## 参考文章
[HTTP access control (CORS)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS)
