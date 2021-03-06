# 跨域资源共享\(CORS\)

## 概念

**跨域资源共享**\(CORS\) 是一种机制：使用额外的HTTP头来告诉浏览器允许web应用访问不同源的资源。当web应用请求一个不同源 \(domain, protocol, or port\) 的资源时，会发起一个跨域 HTTP 请求。

> 跨域请求的示例，来自`https://domain-a.com`的前端js代码使用`XMLHttpRequest`发起`https://domain-b.com/data.json`的请求

出于安全原因，浏览器限制从脚本发起的跨源 HTTP 请求。比如，`XMLHttpRequest` and the [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) 遵循 [同源策略](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy)。除非响应里包含正确的CORS头

**跨域资源共享**通过让服务器添加新的HTTP headers来描述哪些源被允许访问资源。对那些可能对服务器数据产生副作用的 HTTP 请求方法，浏览器必须首先使用 [`OPTIONS`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/OPTIONS) 方法发起一个预检请求

> CORS请求失败会产生错误，但是为了安全，在JavaScript代码层面是无法获知到底具体是哪里出了问题。你只能查看浏览器的控制台以得知具体是哪里出现了错误。

## 简单请求

某些请求不会触发CORS preflight，称为简单请求。简单请求满足以下所有条件：

* 允许的methods
  * `GET`
  * `HEAD`
  * `POST`
* 允许手动设置的headers
  * [`Accept`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept)
  * [`Accept-Language`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Language)
  * [`Content-Language`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Language)
  * [`Content-Type`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Type)
  * `DPR`
  * `Downlink`
  * `Save-Data`
  * `Viewport-Width`
  * `Width`
* `Content-Type`允许的值
  * `application/x-www-form-urlencoded`
  * `multipart/form-data`
  * `text/plain`

### 示例

```javascript
const xhr = new XMLHttpRequest();
const url = 'https://bar.other/resources/public-data/';

xhr.open('GET', url);
xhr.onreadystatechange = someHandler;
xhr.send();
```

![Simple requests](https://mdn.mozillademos.org/files/17214/simple-req-updated.png)

请求报文/响应报文

```text
GET /resources/public-data/ HTTP/1.1
Host: bar.other
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:71.0) Gecko/20100101 Firefox/71.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Connection: keep-alive
Origin: https://foo.example
```

```text
HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 00:23:53 GMT
Server: Apache/2
Access-Control-Allow-Origin: *
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
Transfer-Encoding: chunked
Content-Type: application/xml

[…XML Data…]
```

> CORS preflight 跨域预检

## 预检请求

预检请求首先使用`OPTION`方法发起HTTP请求来确定实际请求是否安全。因为跨域请求可能对用户数据产生影响。

### 示例

```javascript
const xhr = new XMLHttpRequest();
xhr.open('POST', 'https://bar.other/resources/post-here/');
xhr.setRequestHeader('X-PINGOTHER', 'pingpong');
xhr.setRequestHeader('Content-Type', 'application/xml');
xhr.onreadystatechange = handler;
xhr.send('<person><name>Arun</name></person>');
```

![Preflighted requests](https://mdn.mozillademos.org/files/16753/preflight_correct.png)

> 注意：如下所述，实际 POST 请求不包括`Access-Control-Request-*`headers;它们仅在OPTIONS 请求时需要。

预检请求

```text
OPTIONS /resources/post-here/ HTTP/1.1
Host: bar.other
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:71.0) Gecko/20100101 Firefox/71.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Connection: keep-alive
Origin: http://foo.example
Access-Control-Request-Method: POST
Access-Control-Request-Headers: X-PINGOTHER, Content-Type


HTTP/1.1 204 No Content
Date: Mon, 01 Dec 2008 01:15:39 GMT
Server: Apache/2
Access-Control-Allow-Origin: https://foo.example
Access-Control-Allow-Methods: POST, GET, OPTIONS
Access-Control-Allow-Headers: X-PINGOTHER, Content-Type
Access-Control-Max-Age: 86400
Vary: Accept-Encoding, Origin
Keep-Alive: timeout=2, max=100
Connection: Keep-Alive
```

预检请求完成之后，发送实际请求

```text
POST /resources/post-here/ HTTP/1.1
Host: bar.other
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:71.0) Gecko/20100101 Firefox/71.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-us,en;q=0.5
Accept-Encoding: gzip,deflate
Connection: keep-alive
X-PINGOTHER: pingpong
Content-Type: text/xml; charset=UTF-8
Referer: https://foo.example/examples/preflightInvocation.html
Content-Length: 55
Origin: https://foo.example
Pragma: no-cache
Cache-Control: no-cache

<person><name>Arun</name></person>


HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 01:15:40 GMT
Server: Apache/2
Access-Control-Allow-Origin: https://foo.example
Vary: Accept-Encoding, Origin
Content-Encoding: gzip
Content-Length: 235
Keep-Alive: timeout=2, max=99
Connection: Keep-Alive
Content-Type: text/plain

[Some GZIP'd payload]
```

## 解决跨域

* JSONP: 利用<srcipt>标签通过Get去请求脚本，返回`callback(json)`形式的响应后，脚本执行便会触发callback函数
* 响应的header中设置`Access-Control-Allow-Origin`
* postMessage: 
```js
// 发送消息端
window.parent.postMessage('message', 'http://test.com')
// 接收消息端
var mc = new MessageChannel()
mc.addEventListener('message', event => {
  var origin = event.origin || event.originalEvent.origin
  if (origin === 'http://test.com') {
    console.log('验证通过')
  }
})
```
* document.domain: 设置为相同的二级域名，便可实现跨域。部分浏览器已废弃，不推荐

## 链接

[查看详情](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS)

