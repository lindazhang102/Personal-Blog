## HTTP 缓存头提高应用性能

现代开发者有多种多样可用的方法和技术去提高应用性能和用户端体验。其中，被频繁忽略的技术之一就是 HTTP 缓存。
HTTP 缓存是现代各个浏览器广泛采用的规范，从而使得它在 web 应用上实现起来很简单。恰当的使用这些规范可以让你的应用获益匪浅，提高响应时间，降低服务器负载。然而，不正确的缓存却可以让用户看到过期的内容，很难调试问题。这篇文章讨论了 HTTP 缓存的详细内容以及在什么样的场景中使用基于策略的 HTTP 缓存头。

### 概述

当浏览器为了下次更快的检索请求资源而将 web 资源的副本进行本地存储时， HTTP 缓存就产生了。当你的应用提供资源的时候，它可以在响应中附加缓存头指定需要的缓存行为。

![image](https://github.com/lindazhang102/Personal-Blog/raw/master/image/cache0.jpg)

当一个条目被完全缓存时，浏览器可以选择完全不去联系服务器，而仅仅使用它的缓存副本：

![image](https://github.com/lindazhang102/Personal-Blog/raw/master/image/cache1.jpg)

举个例子，一旦你应用里的 CSS 样式表被浏览器下载，在用户会话期间就不用再次下载。这个适用于很多资产类型像 javascript 文件，图片以及不频繁改变的动态内容。在这些例子中，对用户浏览器来说本地缓存这些文件的副本是很有好处的，无论什么时候资源被请求都可以使用这些副本。使用 HTTP 缓存头的应用可以控制缓存行为，缓解服务端负载。
注意：考虑将用户端浏览器作为 HTTP 缓存头的主要客户是很直观的。然而，这些缓存头都可以被任何一个中间代理器以及存在于源服务器和用户端的缓存所得到和利用。
### HTTP 缓存头

这有两个基本的缓存头，Cache-Control 和 Expires。

##### Cache-Control

注意：如果没有 Cache-Control  头部设置，所有的缓存头都不会产生任何结果。

当Cache-Control 头有效的“开关”浏览器上的缓存时，它是最重要的头部设置。当这个头部存在并且设置了一个启动缓存的值时，只要该文件被指定了，浏览器就会缓存该文件。如果没有这个头部，浏览器将会在每一个后续的请求中重新请求这个文件。
public 资源不仅可以被用户端浏览器缓存，还可以被能够给其他众多用户提供服务的中间代理器缓存。

```
Cache-Control:public
```
private 资源会被中间代理器忽略，只能够被客户端缓存。
```
Cache-Control:private
```
Cache-Control 头部的值是一个复合值，既表明了资源是 public 还是 private ,还表明了它能够缓存的时间的最大值，在它被认为过时之前。max-age 设置了一个时间跨度，表示缓存这些资源多长时间。（以秒计算）
```
Cache-Control:public, max-age=31536000
```
当 Cache-Control 头部打开客户端缓存，设置一个资源的 max-age 时，Expires 头部被用来确定一个具体的时间点，到时间点时缓存的资源将会失效。

##### Expires

伴随着 Cache-Control 头部的设置，Expires 只是设置一个缓存资源失效的日期。浏览器将会从这个日期提前请求这个资源的新副本。在那个日期之前，浏览器使用的是本地缓存副本：

注意：如果 Expires 和 max-age 同时都被设置了，那么优先考虑 max-age .
```
Cache-Control:public
Expires: Mon, 25 Jun 2012 21:31:12 GMT
```
当 Cache-Control 和 Expires 告诉浏览器什么时候从网络中进行下一次资源检索时，一些额外的头部会详细说明如何从网络中检索资源。这些请求类型被称为条件请求。
### 条件请求

进行条件请求时，浏览器会询问服务器是否有资源的更新副本。浏览器会发送一些它拥有的缓存资源信息，然后服务器决定是返回更新后的内容还是浏览器的当前副本就是最新的。在后者的情况下，服务器将会返回 304 状态码（没有改动）。

![image](https://github.com/lindazhang102/Personal-Blog/raw/master/image/cache2.jpg)

尽管条件请求确实在网络上调用了一个请求，但没有修改的资源导致了一个空的响应体－－节省了资源传输到客户端的成本。在没有获取该资源的情况下，后端服务能够经常确定一个资源的最后修改日期，这本身就节省了并非微不足道的处理时间。

##### 基于时间

一个基于时间的条件请求确保只有当浏览器的副本被缓存后，请求资源发生了改变时，内容才会被传输。如果缓存副本时最新的，服务器将会返回 304 响应码。
为了使条件请求生效，该应用通过 Last-Modified 响应头指定资源的最后一次修改时间。
```
Cache-Control:public, max-age=31536000
Last-Modified: Mon, 03 Jan 2011 17:45:57 GMT
```
如果在这个日期上使用了 If-Modified-Since 请求头后，如果该资源没有变化，浏览器下次请求这个资源时只会请求这个资源的内容
```
If-Modified-Since: Mon, 03 Jan 2011 17:45:57 GMT
```
如果该资源在 Mon, 03 Jan 2011 17:45:57 GMT 后没有发生改变，服务器将会返回一个空的内容以及 304 响应码。

##### 基于内容

ETag （或实体标签）除了它的值是资源内容（例如，MD5 hash）的一个摘要以外，它和 Last-Modified 头部以类似的方式工作。它允许服务器去识别该资源的缓存内容和该资源最新版本是否有不同。
注意：当最后的修改时间难以确定的时候，这个标签此时是很有用的。
```
Cache-Control:public, max-age=31536000
ETag: “15f0fff99ed5aae4edffdd6496d7131f"
```
在后续的浏览器请求时，If-None-Match 请求头带着该资源的最后一个请求版本的 ETag 值被发送。
```
If-None-Match: “15f0fff99ed5aae4edffdd6496d7131f"
```
当带有If-None-Match 头时，如果当前版本有相同的 ETag 值，这表明它（译者注：也就是服务器里的当前资源版本）的值和浏览器缓存副本的值相同，然后返回 304 的 HTTP 状态。


### 可见性

大多数现代浏览器都有强劲的 请求／响应 的可视化和自查工具。在 Chrome 和 Safari 中，Web Inspector 中的 Network 项会展示响应和请求的头部信息。

Tip:  一个使用多种多样缓存的简单应用的代码可以在 [on GitHub](https://github.com/neilmiddleton/http_caching_demo) 上找到，可以通过运行 http://http-caching-demo.herokuapp.com/ 看到。
一个初始的请求 http://http-caching-demo.herokuapp.com/ 展示了该应用返回的默认头部组。（没有缓存指令）

![image](https://github.com/lindazhang102/Personal-Blog/raw/master/image/cache3.jpg)

通过添加 cached 查询参数， http://http-caching-demo.herokuapp.com/?cache=true ，该应用用 Cache-Control 和 Expires 头（这两个值未来都会被设为 30s）打开了缓存。

![image](https://github.com/lindazhang102/Personal-Blog/raw/master/image/cache4.jpg)

给请求添加 etag 参数， http://http-caching-demo.herokuapp.com/?etag=true ，导致这个简单的 app 要去指定 JSON 内容的 ETag 摘要。

![image](https://github.com/lindazhang102/Personal-Blog/raw/master/image/cache5.jpg)

在更深的检测中，基于 ETag 的条件请求正如所期待的那样执行。浏览器的初始请求可以在从服务器下载文件时被看到。

![image](https://github.com/lindazhang102/Personal-Blog/raw/master/image/cache6.jpg)

然而，在后续请求中你能够看到响应给浏览器的 ETag  检查，服务器返回了 304 HTTP 状态，这会导致浏览器使用它自己的缓存副本：

![image](https://github.com/lindazhang102/Personal-Blog/raw/master/image/cache7.jpg)

### 使用案例

##### 静态资源

在正常使用情况下，任何一个开发者的出发点应该被当作侵略式缓存策略加到应用中不会发生改变的文件上。一般情况下，这些文件包括服务于该应用的静态文件，比如图片，CSS 文件和 Javascript 文件。典型地，当这些文件在每个页面上被重新请求时，费一点点吹灰之力页面性能就能大幅度提升。
在这些例子中，你应该设置 Cache-Control 头，从发起请求的时间点算起，将 max-age 值设为一年。推荐  Expires 也被设置为类似的值。

Tip : 1 年＝ 31536000s
```
Cache-Control:public; max-age=31536000
Expires: Mon, 25 Jun 2013 21:31:12 GMT
```
通常来说，当更大的时间段不被 [RFC](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9) 支持时，该值比这个时间段还大就不是一个好主意，并且可能会被忽略。
##### 动态内容

动态内容更加的细致入微。对于每一个资源，开发者都必须评估它可以被缓存的多么严重，以及什么样的暗示可能会提供给用户过时的内容。这有两个例子，博客 RSS 订阅（它不会每隔几个小时就改变），JSON 包，它会驱动用户的 Twitter 时间轴（每隔几秒就会更新）。在这些案例中缓存资源是合理的，只要你认为这不会给用户端带来问题。

##### 私有内容

私有内容（也就是被认为敏感的，并且受保密措施影响的）甚至要求更多的评估。你作为开发者不仅需要确定一个资源的可缓存性，而且还要考虑中间缓存器（比如网络代理）缓存不受用户控制的文件时带来的影响。如果存在疑问，那么这些文件一个都不缓存是一个安全的选择。
客户端缓存仍然是可取的，你可以请求只被私有缓存的资源（也就是只存在于用户端浏览器的缓存）
```
Cache-Control:private, max-age=31536000
```
### 缓存预防

高安全性或多样性的资源经常要求不能缓存。举个例子，涉及到一个购物车结算流程的任何东西。不幸的是，仅仅删除缓存头是无效的，因为很多现代浏览器都是基于它们自己的内部算法缓存内容的。在这种情况下，明确告诉浏览器不要缓存内容是很必要的。
Cache-Control 头除了指定 public 和 private 外，还可以指定 no-cache 和 no-store, 它们会告诉浏览器在任何情况下都不要去缓存资源。

注意：这两个值都是需要的（笔者注：这当然是为了兼容性喽\~_~），因为 IE 使用 no-cache，Firefox 使用 no-store。
```
Cache-Control:no-cache, no-store
```
### 实现

一旦理解了 HTTP 缓存背后的这些概念，下一步就是在你的应用中去实现它们。大多数现代 web 框架让它成为了一个琐碎的任务。

Language/framework	 | Tutorials
---|---
Ruby/Rails | * [HTTP Caching in Ruby with Rails](https://devcenter.heroku.com/articles/http-caching-ruby-rails)* [Using Rack::Cache with Memcached](https://devcenter.heroku.com/articles/rack-cache-memcached-rails31) to provide an HTTP cache store
Java | [HTTP Header Caching in JAX-RS](https://devcenter.heroku.com/articles/jax-rs-http-caching)

相关阅读：

[Application Architecture](https://devcenter.heroku.com/categories/application-architecture)

[HTTP Caching in Ruby with Rails](https://devcenter.heroku.com/articles/http-caching-ruby-rails)

[Using Rack::Cache with Memcached in Rails 3.1+ (Including Rails 4)](https://devcenter.heroku.com/articles/rack-cache-memcached-rails31)

[HTTP Caching in Java with JAX-RS](https://devcenter.heroku.com/articles/jax-rs-http-caching)

原文链接：

[https://devcenter.heroku.com/articles/increasing-application-performance-with-http-cache-headers](https://devcenter.heroku.com/articles/increasing-application-performance-with-http-cache-headers)
