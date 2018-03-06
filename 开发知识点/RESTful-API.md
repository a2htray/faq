# RESTful-API 原文选段

## 原文[https://codeplanet.io/principles-good-restful-api-design/](https://codeplanet.io/principles-good-restful-api-design/)

> A good RESTful API design will keep track of the version in the URL. The other
most common solution is to put a version number in a request header, but after working
with many different Third Part Developers, I can tell you that adding headers is
no where near as easy as adding a URL Segment.

* API中需要加入API的版本信息
* 可以有URL中加入，如`/v2/...`
* 比较通用的方案是在请求头中加一个版本号

> The most important thing is that when you do deprecate a version of you API, you
actually contact the developers using the deprecated API feature.

* 当你要弃用一个版本的API后，必须要通知接口的使用者
* 可以使用邮件通知的方式告知开发者
* 比较的好方式，使用文档对API进行介绍及维护

> It's important that the root entry point into you API is as simple as possible,
as a long complex URL will apear daunting and turn developers away.

* API的URL尽可能简单明了
* 将API的接口放在子域名中，更为合理，如`api.domain.com/v2/...`

> It is a good idea to have content at the root of your API.

* 在最顶层的API中可以放置一些内容，用于展示平台有多少`endpoint`

> Having a key in your documentation for whatever conversation is a good idea.

* 在各别的API中加一个关键字

> If they request a certain collection, and iterate over the results, and they never
see more than 100 items.

* 作者的意思在于，不需要对返回的条目数进行限制，或者做一些过滤的操作
* 接上，也可以给第三方开发者排序、过滤等功能(# 鉴于网络的传输速度，用户可以需要更快的返回结果)
* **作者还是推崇对数据进行过滤及分页**

> Since those are GET request, filtering information should be passed via the URL.

* 对`GET`请求而言，需要将过滤信息放在`URL`中

> Make sure your white-list the columns for which the Comsumer can filter and sort by.
We don't want any database error being sent to Comsumers

* 设置可显示、可过滤、可排序的字符，不要将数据库的报错信息返回给用户。

> Some API creators recommend adding a `.json`,`.xml`, or `.html` file extension
to the URL(after the endpoint) for specifying the content type to be returned. although
I'm personally not a fan of this. I really like the Accept header(which is built into
the HTTP spec) and feel that is the appropriate thing to use.

* 对通用型的接口，不同返回数据的结构，建议使用在`request header`中指定返回数据结构。


> 这篇文章的内容实在太全面了，决定要翻译这篇文章。
