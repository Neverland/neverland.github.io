# 如何保证离开页面时成功发送请求

> 有些场景我们需要在用户跳转到其他页面或者提交表单时发送一些数据时发送HTTP请求去记录这些操作。

请看这个例子，在点击链接时向服务端发送一些数据：

``` html
<a href="/some-other-page" id="link">Go to Page</a>

<script>
document.getElementById('link').addEventListener('click', (e) => {
  fetch("/log", {
    method: "POST",
    headers: {
      "Content-Type": "application/json"
    },
    body: JSON.stringify({
      some: "data"
    })
  });
});
</script>
```

这里没有什么其他操作， 该链接可以正常打开（我没有使用e.preventDefault()）。但是在跳转前会触发一个POST请求，而且也不用等任何`response`返回。我的预期时他被成功发送到服务端。

乍看之下，期望是请求是同步的，然后我们会离开当前页面去目标页面。请求也成功被发送到服务端，但是事实证明真实情况并非如此。

## 浏览器并不保证成功发送HTTP请求

当浏览器中的页面终止时不能保证进程中的HTTP请求会成功（ [see more](https://developers.google.com/web/updates/2018/07/page-lifecycle-api) about the “terminated” and other states of a page’s lifecycle）。 请求的成功发送依赖一下几个条件：

 - 网络连接
 - 程序性能
 - 甚至是服务器的配置

因此，在这些场景发送数据并不靠谱，如果这些数据是业务以来的铭感数据，这样会存在很大的问题。

为了证明这点，我用上面的代码结合Express做了一个实验。点击链接时页面跳转到/other，在跳转前发送了一个POST请求。

我打开了浏览器的Network面板，限速选择`Slow 3G`，页面打开时我清空日志，一切都很稳！

![](https://i0.wp.com/css-tricks.com/wp-content/uploads/2022/02/initial-load-1.png?resize=1536%2C727&ssl=1)

但是一旦点击链接，页面跳转时，请求就会被取消。

![](https://i0.wp.com/css-tricks.com/wp-content/uploads/2022/02/request-failed-1.gif?resize=1000%2C472&ssl=1)

这种场景我们没信心能让服务端接受到请求，我们用代码`window.location`跳转页面时也是同样的情况：

```javascript
document.getElementById('link').addEventListener('click', (e) => {
+ e.preventDefault();

    // Request is queued, but cancelled as soon as navigation occurs.
    fetch("/log", {
        method: "POST",
        headers: {
            "Content-Type": "application/json"
        },
        body: JSON.stringify({
            some: 'data'
        }),
    });

+ window.location = e.target.href;
});
```

无论跳转啥时候发生，请求都有可能会被终止。


## 请求为啥被取消了？

根本原因在于默认情况下，`XHR`请求（通过fetch或XMLHttpRequest）是异步且非阻塞的。请求一旦进入队列，后续工作就会被交给后面浏览器级别的API。

由于性能问题而不喜欢请求占用主线程，这样也以为这页面被“终止”时，请求也有可能被取消的风险。从而无法保证后续工作都完成。[以下是谷歌对特殊生命周期的总结](https://developers.google.com/web/updates/2018/07/page-lifecycle-api#states)

> 一旦页面开始被浏览器卸载并从内存中清除，页面就处于终止状态。此时没有[new task](https://translate.google.com/website?sl=auto&tl=zh-CN&hl=zh-CN&client=webapp&u=https://html.spec.whatwg.org/multipage/webappapis.html%23queue-a-task)可以启动，那么正在进行的任务如果运行时间过长可能会被杀死。

人话：浏览器认为一个页面关闭时，就没必要为他队列中的后台进程浪费资源了。

## 我们有哪些选择呢？

避免这个问题是尽可能的延迟用户操作，确保应答返回。在过是通过XMLHttpRequest的[同步](https://translate.google.com/website?sl=auto&tl=zh-CN&hl=zh-CN&client=webapp&u=https://xhr.spec.whatwg.org/%23synchronous-flag)方式这种方式做到的。这样会阻塞进程并且导致性能问题（译者注：页面看起来假死了）这种骚操作肯定是不行的，事实上这种方式正在被淘汰[（Chrome v80+已经将其删除）](https://developers-google-com.translate.goog/web/updates/2019/12/chrome-80-deps-rems?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp)。

这种场景我们可以使用`Promise`等待`resolve`成功返回后，在安全的执行跳转行为。用之前的例子改造一下看起来是这样子：

```javascript
document.getElementById('link').addEventListener('click', async (e) => {
  e.preventDefault();

  // Wait for response to come back...
  await fetch("/log", {
    method: "POST",
    headers: {
      "Content-Type": "application/json"
    },
    body: JSON.stringify({
      some: 'data'
    }),
  });

  // ...and THEN navigate away.
   window.location = e.target.href;
});
```
这样符合预期，不过也有很多问题！

**首先，这样会延迟跳转的行为而导致用户体验不佳。** 收集分析数据肯定有益于业务（或潜在用户）。但是这样会影响存量用户，再说这样会强以来服务端，任何服务端的性能或者延迟都会影响用户。如果因为收集数据导致用户的有价值操作收到影响，那么肯定是双输。

**其次，这种方法也并不靠谱。因为某些终止行为并不能用代码的方式解决！** 。例如，`e.preventDefault()`在关闭浏览器选项卡的时候并不好使（译者注：强杀浏览器进程）,所以这种方式也只是收集了部分用户数据而不能完全信任它。

## 教浏览器完成外发请求

好在通过设置可以让浏览器保留未完成的`HTTP请求`而不伤害用户体验。


### 使用 Fetch 的keepalive配置

在使用`fetch()`把它的[keepalive](https://fetch.spec.whatwg.org/#request-keepalive-flag)选项设置为`true`，即使发起请求的页面被终止那么该请求也会保持存活状态。使用我们最初的例子改造一下结果如下：
```html
<a href="/some-other-page" id="link">Go to Page</a>

<script>
  document.getElementById('link').addEventListener('click', (e) => {
    fetch("/log", {
      method: "POST",
      headers: {
        "Content-Type": "application/json"
      },
      body: JSON.stringify({
        some: "data"
      }),
      keepalive: true
    });
  });
</script>
```
单击该链接并发生页面跳转时，不会取消请求：
![](https://i0.wp.com/css-tricks.com/wp-content/uploads/2022/02/request-succeeded.gif?resize=1000%2C472&ssl=1)

相反，请求的状态为(unknown)那是因为当前页面还没接收任何类型的响应。

像这样作为浏览器API的能力时，这个问题一行代码很容易解决。当你寻找更专业更简单的方式时，还有一个类似的方案。

### Navigator.sendBeacon()
Navigator.sendBeacon()函数专门用于发送单向请求[（信标）](https://w3c.github.io/beacon/#sec-processing-model)。

具体实现如下，发送一个POST带有字符串化的 JSON 和一个“text/plain” Content-Type：

```
navigator.sendBeacon('/log', JSON.stringify({
    some: "data"
}));

```
此 `API` 不允许您发送自定义`header`。因此，为了让我们以“application/json”的形式发送数据，我们需要做一些小改造并使用Blob：

```html
<a href="/some-other-page" id="link">Go to Page</a>

<script>
  document.getElementById('link').addEventListener('click', (e) => {
    const blob = new Blob([JSON.stringify({ some: "data" })], { type: 'application/json; charset=UTF-8' });
    navigator.sendBeacon('/log', blob));
  });
</script>
```
最终我们达到了目的，页面跳转并成功发送请求。但是更多的场景使用`fetch`可能更好，因为`Beacon`总是以低优先级发送。

为了演示，以下是同时`fetch()`使用`keepalive` 和 `sendBeacon()`时`netWork`选项卡中的日志：
![](https://i0.wp.com/css-tricks.com/wp-content/uploads/2022/02/request-priorities.png?resize=1536%2C727&ssl=1)

默认情况下`fetch()`获得**高**优先级，而`Beacon()`（上面称为“ping”类型）具有**最低**优先级。对于页面中不重要的请求，这是一件好事。见[Beacon规范](https://translate.google.com/website?sl=auto&tl=zh-CN&hl=zh-CN&client=webapp&u=https://www.w3.org/TR/beacon/)

> 该规范定义了一个接口，[…] 最大限度地减少与其他关键操作争资源，同时确保此类请求仍被处理并交付到目的地。

人话，sendBeacon()确保它的请求不会妨碍那些对您的应用程序和用户体验真正重要的请求。

### ping 属性
好消息是越来越多的浏览器开始支持[ping属性](https://css--tricks-com.translate.goog/the-ping-attribute-on-anchor-links/?_x_tr_sl=auto&_x_tr_tl=zh-CN&_x_tr_hl=zh-CN&_x_tr_pto=wapp)。该属性添加到链接时，会触发一个`POST`请求：
```html
<a href="http://localhost:3000/other" ping="http://localhost:3000/log">
  Go to Other Page
</a>
```
这些请求头将包含单击链接的页面 ( ping-from)，以及href该链接的值 ( ping-to)：
```
headers: {
    'ping-from': 'http://localhost:3000/',
    'ping-to': 'http://localhost:3000/other'
    'content-type': 'text/ping'
    // ...other headers
},
```
技术上它比较类似`Beacon()`但是它有更多的限制：

1. **它严格限制在链接上的使用，** 不能用于与其他交互相关的数据，例如按钮点击或表单提交。
2. **浏览器支持很好，** [但不完美](https://translate.google.com/website?sl=auto&tl=zh-CN&hl=zh-CN&client=webapp&u=https://caniuse.com/ping)， 此文撰写时Firefox还不支持。
2. **您无法随请求一起发送任何自定义数据。** 如上， 如前所述只能活得几个ping-*。

综上所述，ping可以用于发送简单的请求并且不想写代码，那么它是一个很好的工具。如果需要发送更多自定义内容，则不是最好的选择。

## 那么我该使用哪一个

使用`fetch keepalive`或`sendBeacon()`发送最后一秒请求适合的场景，需要考虑以下几点：
### 你可以这么使用fetch() + keepalive

- 需要随请求传递自定义headers。
- 想向服务端发送GET请求，而不是POST.
- 需要支持较旧的浏览器（如 IE）并且已经为`fetch`加了`polyfill`。

### 在以下情况下sendBeacon()是更好的选择：

- 发不需要太多自定义的简单服务请求。
- 更简洁、更优雅的 API。
- 确保的请求不会与应用程序中发送的其他高优先级请求竞争。

[原文:Reliably Send an HTTP Request as a User Leaves a Page](https://css-tricks.com/send-an-http-request-on-page-exit/)