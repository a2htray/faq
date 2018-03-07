# _Principles of good RESTful API Design_

# 良好的RESTful API设计原则

_Good restful API design is hard! An API represents a contract between you and those who Consume your data. Breaking this contract will result in many angry emails, and a slew of sad users with mobile apps which no longer work. Documentation is half the battle, and it is very difficult to find programmer who also likes to write._

设计一个良好的RESTful API是如此的困难！一个API代表了你和使用你数据的用户之间的协议。破坏了这个协议，将会收到愤怒的使用者发来的邮件，使用一大群使用手机应用而应用无法工作的人感到沮丧。编写好的文档其实就已经成功一半了，当然，也很难发现喜欢写文档的开发人员。

_Building an API is one of the most important things you can do to increase the value of your service. By having an API, your service / core application has the potential to become a platform from which other services grow. Look at the current huge tech companies: Facebook, Twitter, Google, GitHub, Amazon, Netflix… None of them would be nearly as big as they are today if they hadn’t opened up their data via API. In fact, an entire industry exists with the sole purpose of consuming data provided by said platforms._

为了提高服务的价值，构建API服务是最重要的事件之一。正因为有API服务，伴随使用你服务的其他服务的成长，你的服务/核心应用才具有成为一个平台的潜力。看一看当今的科技巨头：Facebook、Twitter、Google、Github、Amazon、Netflix等等，如果他们通过API开放他们的数据，他们之中几乎不会有一家公司像他们现在公司的这么大的规模。实际上，整个行业中也存在着一个唯一的目标，那就是消费由平台所提供的数据。


> _The easier your API is to consume, the more people that will consume it._

> 早一点构建你的API服务供用户消费数据，会有更多的用户愿意消费你的数据。

_The principles of this document, if followed closely when designing your API, will ensure that Consumers of your API will be able to understand what is going on, and should drastically reduce the number of confused and/or angry emails you receive. I’ve organized everything into topics, which don’t necessarily need to be read in order._

对于文章中提到的设计原则，如果在设计API时紧紧遵循这些设计原则，可以保证那些使用你API的用户有能力理解接下来需要做什么，且你收到怀着困惑或生气的邮件也会减少。我会将所有有关API设计的内容组织到这次的主题中，你也不需要按文章的顺序来阅读，可进行选择性阅读有兴趣的内容。

## _RESTful API Design Definitions_

## RESTful API设计中的定义

_Here’s a few of the important terms I will use throughout the course of this document:_

接下来介绍一些重要的术语，在这些术语将会贯穿于这次的课程。

* _**Resource**: A single instance of an object. For example, an animal._
* _**Collection**: A collection of homogeneous objects. For example, animals._
* _**HTTP**: A protocol for communicating over a network._
* _**Consumer**: A client computer application capable of making HTTP requests._
* _**Third Party Developer**: A developer not a part of your project but who wishes to consume your data._
* _**Server**: An HTTP server/application accessible from a Consumer over a network._
* _**Endpoint**: An API URL on a Server which represents either a Resource or an entire Collection._
* _**Idempotent**: Side-effect free, can happen multiple times without penalty._
* _**URL Segment**: A slash-separated piece of information in the URL._


* **Collection**：两种类型对象的命令，比如动物集合`animals`
* **HTTP**：网络间通信的协议
* **Consumer**：可以发送HTTP请求的客户端应用
* **Third Party Developer**：不属于你项目的成员，但想要消费你数据的开发人员
* **Server**：通过网络，用户可以访问的HTTP服务或应用
* **Endpoint**：一个API的路径，该路径表示一个资源`Resource`或者一个集合`Collection`
* **Idempotent**：没有任何副作用，可安全地进行多次请求
* **URL Segment**：`URL`片段，使用斜线`/`分隔，在`URL`中表示特定的信息

## _Data Design and Abstraction_

## 数据设计及概要

_Planning how your API will look begins earlier than you’d think; first you need to decide how your data will be designed and how your core service / application will work. If you’re doing API First Development this should be easy. If you’re attaching an API to an existing project, you may need to provide more abstraction._

尽可能早地计划去设计你的API。首先，你需要决定你数据的结构设计和你的核心应用如何有效工作。如果你首次接手一个API项目，那会很容易。如果你是要一个已有的项目中新加一个API，你可以需要更多的信息去理解。

_Occasionally, a Collection can represent a database table, and a Resource can represent a row within that table. However, this is not the usual case. In fact, your API should abstract away as much of your data and business logic as possible. It is very important that you don’t overwhelm Third-Party Developers with any complex application data, if you do they won’t want to use your API._

有时候，一个`Collection`可以表示数据库中的一张表里的数据，一个`Resource`可以表示表中的一行数据。然而，这不是一种常用的案例。实际上，你的API需要尽可以将你的数据和业务逻辑抽象出来。对于使用复杂数据的第三方开发者，你不应该强制他们使用你的数据格式，这是非常重要的。如果你没将进行数据抽象，第三方开发者将不愿意使用你的API。

_There are also many parts of your service which you SHOULD NOT expose via API at all. A common example is that many APIs will not allow third parties to create users._

你服务中当然也存在一部分的功能不应用暴露在API中。一个通用的例子，许多API服务中是不允许第三方开发者去创建用户。

## Verbs

## HTTP动作

_Surely you know about GET and POST requests. These are the two most commonly requests used when your browser visits different webpages. The term POST is so popular that it has even invaded common language, where people who know nothing about how the Internet works do know they can “post” something on a friends Facebook wall._

请确保你是有已经过`GET`和`POST`请求的。当你使用浏览器访问不同的网页时，这是两种最常使用的请求。这个术语`POST`也变得非常流行，都已经影响到我们常用的语言之中。对于那些不知道因特网如何工作的人，也知道使用`Facebook`的朋友圈`post`一些内容。

_There are four and a half very important HTTP verbs that you need to know about. I say “and a half”, because the PATCH verb is very similar to the PUT verb, and two two are often combined by many an API developer. Here are the verbs, and next to them are their associated database call (I’m assuming most people reading this know more about writing to a database than designing an API)._

你需要知道且比较重要的`HTTP`动作有四个半。我说还有`半个`，这是因为`PATCH`与`PUT`十分相似，且经常被开发人员结合使用。下面是这些动作，接下来是这些动作有关于数据库的调用(假设，绝大部分阅读到这篇文章的人，其数据库编程懂得知识比如何设计一个API更多)。

* _**GET** (SELECT): Retrieve a specific Resource from the Server, or a listing of Resources._
* _**POST** (CREATE): Create a new Resource on the Server._
* _**PUT** (UPDATE): Update a Resource on the Server, providing the entire Resource._
* _**PATCH** (UPDATE): Update a Resource on the Server, providing only changed attributes._
* _**DELETE** (DELETE): Remove a Resource from the Server._

* **GET** (SELECT)：从服务中得到特定的资源`Resource`，或者资源的集合
* **POST** (CREATE)：在服务中创建新的资源
* **PUT** (UPDATE)：提供资源的所有信息，在服务器中更新一个资源
* **PATCH** (UPDATE)：只提供需要更新的属性，在服务器中更新一个资源
* **DELETE** (DELETE)：在服务中移除一个资源

_Here are two lesser known HTTP verbs:_

有两个较少人知的`HTTP`动作：

* _**HEAD** – Retrieve meta data about a Resource, such as a hash of the data or when it was last updated._
* _**OPTIONS** – Retrieve information about what the Consumer is allowed to do with the Resource._

* **HEAD**：得到关于资源的元数据，比如一个数据的哈希值，或者最近更新的时间
* **OPTIONS**：得到对于特定资源，用户可以允许操作的信息

_A good RESTful API will make use of the four and a half HTTP verbs for allowing third parties to interact with its data, and will never include actions / verbs as URL segments._

一个良好的`RESTful API`会利用这四个半的`HTTP`动作，允许第三方和`API`的数据做交互，并且不会有`URL`片段中包含具体动作的信息。

_Typically, GET requests can be cached (and often are!) Browsers, for example, will cache GET requests (depending on cache headers), and will go as far as prompt the user if they attempt to POST for a second time. A HEAD request is basically a GET without the response body, and can be cached as well._

很典型的一个例子，`GET`请求可以被浏览器进行缓存(常常都是这样的)，比如，浏览器缓存一个`GET`请求(根据它的`cache`请求头)，会持续到浏览器第二次尝试发送一个`POST`请求。`HEAD`请求是一个基本的`GET`请求，它没有响应体，同样可以被缓存。

## _Versioning_

## 版本


