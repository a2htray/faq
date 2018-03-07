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


* _**Resource**：一个对象的实例，比如，一个动物`animal`
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

_No matter what you are building, no matter how much planning you do beforehand, your core application is going to change, your data relationships will change, attributes will invariably be added and removed from your Resources. This is just how software development works, and is especially true if your project is alive and used by many people (which is likely the case if you’re building an API)._

不管你的要构建什么样的应用，也不管事先你做了多少准备工作，你的核心应用总是会发生改变，你的数据关系也会发生改变，总是会有新增的数据属性，也会有删除的数据属性。这正是软件开发中需要做的工作，也特别适用于你的项目依旧在运行中，且有很多的用户正在使用你的服务(这也是需要构建API服务的案例)。

_Remember than an API is a published contract between a Server and a Consumer. If you make changes to the Servers API and these changes break backwards compatibility, you will break things for your Consumer and they will resent you for it. Do it enough, and they will leave. To ensure your application evolves AND you keep your Consumers happy, you need to occasionally introduce new versions of the API while still allowing old versions to be accessible._

请记住，API是服务器端与客户端之间的一个已经发布的协议。如果你对服务器API做了修改，然后这些修改破坏了向后兼容，那你也就是破坏了你和客户端之前的协议，你的用户也为因此而埋怨你。经常这样做的话，你的用户将不会在使用你的服务。为了确保你的服务能够正常升级，也为了让你的用户感觉到愉悦，你需要偶尔引入一些新版本的API，同时依旧允许旧版本的API还是可以被访问。

_As a side note, if you are simply ADDING new features to your API, such as new attributes on a Resource (which are not required and the Resource will function without), or if you are ADDING new Endpoints, you do not need to increment your API version number since these changes do not break backwards compatibility. You will want to update your API Documentation (your Contract), of course._

从另一个侧面说明，如果你对你的API只是简单地加了新的特性，比如一个资源的新的属性(这里的属性不是必选的，你的资源没有这个属性依旧可以正常使用)，再比如你新增了一个路由，你不需要对你的API版本进行更新，因为这些改变并没有破坏向后兼容性。当然，你还是需要更新你的API文档(你和客户端遵守的协议)。

_Over time you can deprecate old versions of the API. To deprecate a feature doesn’t mean to shut if off or diminish the quality of it, but to tell Consumers of your API that the older version will be removed on a specific date and that they should upgrade to a newer version._

过一段时间后，你可以弃用旧版本的API。为了弃用一个特性，并不是意味着直接移除这个特性，也不是降低这个特性的质量，而是告诉你的用户，这个旧版本的API在指定日期将会移除，需要升级到较新的版本。

_A good RESTful API design will keep track of the version in the URL. The other most common solution is to put a version number in a request header, but after working with many different Third Party Developers, I can tell you that adding headers is no where near as easy as adding a URL Segment._

一个良好的API设计需要在`URL`上保留版本信息。其中一个非常通用的解决方案就是将在请求头中加入版本信息，但在与多个第三方开发者合作之后，我可以告诉你增加一个`URL`片段(用于标识版本信息)比在请求头中增加更为简单。

## _Analytics_

## 分析

_Keep track of the version/endpoints of your API being used by Consumers. This can be as simple as incrementing an integer in a database each time a request is made. There are many reasons that keeping track of API Analytics is a good idea, for example, the most commonly used API calls should be made efficient._

对已经被用户使用的API，需要对它的版本/路由进行控制、追踪。这个非常容易实现，就好像每次发生一个请求，在数据库中进行自增。对API进行追踪分析是一个非常好的主意，也有很多其他的理由。比如，使用率最频繁的API应尽可能的高性能。

_For the purposes of building an API which Third Party Developers will love, the most important thing is that when you do deprecate a version of your API, you can actually contact developers using deprecated API features. This is the perfect way to remind them to upgrade before you kill the old API version._

为了构建出让第三方开发者喜爱的API，最重要的事在于当你的弃用一个版本的API时，你能马上通知正在使用该API的开发者。在移除一个旧版本前，让你的开发者更新API，这是一个非常完美的方法。

_The process of Third Party Developer notification can be automated, e.g. mail the developer every time 10,000 requests to a deprecated feature are made._

给第三方开发者发送通知的过程应该是自动的。比如，当用户对已经弃用的特性每发送10000次请求就已邮件的形式进行通知。


## _API Root URL_

## API根路径

_The root location of your API is important, believe it or not. When a developer (read as code archaeologist) inherits an old project using your API and needs to build new features, they may not know about your service at all. Perhaps all they know is a list of URLs which the Consumer calls out to. It’s important that the root entry point into your API is as simple as possible, as a long complex URL will appear daunting and can turn developers away._

不管你相信不相信，API服务的根路径是十分重要的。当一个开发者(把他当作代码的考古学家)得到了使用你API服务的旧项目，且他需要加入新的特性，他对你的API服务一点都不知情。也可能他们所有人都知道当前客户端所使用请求的API列表。你API的入口根路径应该尽可能的简单，这是非常重要的，因为一个较长且复杂的`URL`会让人望而却步，最终使用开发者不再使用而离开。

_Here are two common URL Roots:_

下面是两种常用的`URL`根路径：

* https://example.org/api/v1/*
* https://api.example.com/v1/*

如果你的应用是非常庞大的，或者你期盼你的应用将会变得庞大，使用子域名的形式搭建你的API服务(如api.)是一个不错的选择。顺着这个思路，可以让你的应用更加具有弹性伸缩。

_If you anticipate your API will never grow to be that large, or you want a much simpler application setup (e.g. you want to host the website AND API from the same framework), placing your API beneath a URL segment at the root of the domain (e.g. /api/) works as well._

如果你觉得你的API服务不会发展成那么大时，或者你只是需要一个足够简单的应用(比如，你的网站和API服务都是属于同一个框架)，在域名中加一个`URL`片段(比如，`/api/`)，也是可以非常好地工作的。

_It’s a good idea to have content at the root of your API. Hitting the root of GitHub’s API returns a listing of endpoints, for example. Personally, I’m a fan of having the root URL give information which a lost developer would find useful, e.g., how to get to the developer documentation for the API._

让你的API的根路径中包含内容是一个很好的主意。点击`GitHub`根路径的API，将会返回一个路由的列表。再比如，作为私人的，在根路径中加入一些帮助信息，让不知所措的开发者找到有用的信息，我是这种方法的支持者。

_Also, notice the HTTPS prefix. As a good RESTful API, you must host your API behind HTTPS._

同时，还需要注意到`HTTPS`前缀，作为一个良好的`RESTful API`，你必须让你的API在`HTTPS`协议中进行通信。


## _Endpoints_

## 路由

_An Endpoint is a URL within your API which points to a specific Resource or a Collection of Resources._

一个路由就一个`URL`路径，它标识着一个特定的资源，或者一个资源的集合。

_If you were building a fictional API to represent several different Zoo’s, each containing many Animals (with an animal belonging to exactly one Zoo), employees (who can work at multiple zoos) and keeping track of the species of each animal, you might have the following endpoints:_

如果你现在要为几个不同的动物园及其中的员工构建一个虚拟的`API`，每个动物园都包含许多动物(一个动物只属于一个动物园)，对不同种类的动物进行跟踪，你可能会设计出下面几个路由：

* https://api.example.com/v1/**zoos**
* https://api.example.com/v1/**animals**
* https://api.example.com/v1/**animal_types**
* https://api.example.com/v1/**employees**

_When referring to what each endpoint can do, you’ll want to list valid HTTP Verb and Endpoint combinations. For example, here’s a semi-comprehensive list of actions one can perform with our fictional API. Notice that I’ve preceded each endpoint with the HTTP Verb, as this is the same notation used within an HTTP Request header._

当要指出每一个路由具体能做什么的时候，你需要将`HTTP`动作与路由进行结合，并将其展示出来。比如，通过你虚拟的`API`，下面这些动作都是可以去执行的。注意，我在每个路由前都加上了`HTTP`的动作，这些动作和在`HTTP`请求中的动作是一样的。

* _GET /zoos: List all Zoos (ID and Name, not too much detail)_
* _POST /zoos: Create a new Zoo_
* _GET /zoos/ZID: Retrieve an entire Zoo object_
* _PUT /zoos/ZID: Update a Zoo (entire object)_
* _PATCH /zoos/ZID: Update a Zoo (partial object)_
* _DELETE /zoos/ZID: Delete a Zoo_
* _GET /zoos/ZID/animals: Retrieve a listing of Animals (ID and Name)._
* _GET /animals: List all Animals (ID and Name)._
* _POST /animals: Create a new Animal_
* _GET /animals/AID: Retrieve an Animal object_
* _PUT /animals/AID: Update an Animal (entire object)_
* _PATCH /animals/AID: Update an Animal (partial object)_
* _GET /animal_types: Retrieve a listing (ID and Name) of all Animal Types_
* _GET /animal_types/ATID: Retrieve an entire Animal Type object_
* _GET /employees: Retrieve an entire list of Employees_
* _GET /employees/EID: Retreive a specific Employee_
* _GET /zoos/ZID/employees: Retrieve a listing of Employees (ID and Name) who work at this Zoo_
* _POST /employees: Create a new Employee_
* _POST /zoos/ZID/employees: Hire an Employee at a specific Zoo_
* _DELETE /zoos/ZID/employees/EID: Fire an Employee from a specific Zoo_

* GET /zoos: 列出所有的动物园 (ID and Name, 不需要太详细)
* POST /zoos: 创建一个新的动物园
* GET /zoos/ZID: 返回一个动物园的实例
* PUT /zoos/ZID: 更新动物园信息
* PATCH /zoos/ZID: 更新动物园信息 (使用部分信息)
* DELETE /zoos/ZID: 删除一个动物园
* GET /zoos/ZID/animals: 返回动物列表 (ID and Name).
* GET /animals: 返回所有的动物列表 (ID and Name).
* POST /animals: 创建一个新的动物
* GET /animals/AID: 返回一个动物的实例
* PUT /animals/AID: 更新一个动物信息
* PATCH /animals/AID: 更新一个动物信息 (使用部分信息)
* GET /animal_types: 返回所有的动物类型
* GET /animal_types/ATID: 返回一个动物类型的实例
* GET /employees: 返回员工列表
* GET /employees/EID: 返回特定的员工信息
* GET /zoos/ZID/employees: 指定动物园ID，返回该动物园中的员工列表
* POST /employees: 创建一个新的员工
* POST /zoos/ZID/employees: 指定动物园雇用一个员工
* DELETE /zoos/ZID/employees/EID: 解雇特定动物园中的特定员工

_In the above list, ZID means Zoo ID, AID means Animal ID, EID means Employee ID, and ATID means Animal Type ID. Having a key in your documentation for whatever convention you choose is a good idea._

在上述的列表中，`ZID`表示动物园的ID，`AID`表示动物的ID，`EID`表示员工的ID，`ATID`表示动物类型的ID。在你的文档中，对于你的选择的会话设置一个关键字是非常好的想法。

_I’ve left out the common API URL prefix in the above examples for brevity. While this can be fine during communications, in your actual API documentation, you should always display the full URL to each endpoint (e.g. GET http://api.example.com/v1/animal_type/ATID)._

为了简洁起见，对于每一个`API`都省略了前缀。当然这些问题可以在交流中协定好，在你真实的API文档中，你总是需要为每一个路由展示其全路径(如，GET http://api.example.com/v1/animal_type/ATID)。

_Notice how the relationships between data is displayed, specifically the many to many relationships between employees and zoos. By adding an additional URL segment, one can perform more specific interactions. Of course there is no HTTP verb for “FIRE”-ing an employee, but by performing a DELETE on an Employee located within a Zoo, we’re able to achieve the same effect._

注意，数据之间的关系要如何做展示，特别是例子中员工与动物园之间的多对多关系。通过加一个额外的`URL`片段，API可以执行特定的作用。当然，我们这没有一个明确的`HTTP`动作`FIRE`用于解雇一个员工，但通过在特定的动物园中删除一个员工的逻辑，我们有能力去实现相同的功能。

## _Filtering_

## 过滤

_When a Consumer makes a request for a listing of objects, it is important that you give them a list of every single object matching the requested criteria. This list could be massive. But, it is important that you don’t perform any arbitrary limitations of the data. It is these arbitrary limits which make it hard for a third party developer to know what is going on. If they request a certain Collection, and iterate over the results, and they never see more than 100 items, it is now their job to figure out where this limit is coming from. Is their ORM buggy and limiting items to 100? Is the network chopping up large packets?_

当用户发送一个请求，要求返回一些对象的列表，根据请求的要求，你需要给他们返回特定的列表，这是至关重要的。这个列表可以非常庞大。但其中，尤为重要的是，你不能对这些数据再强加限制。正是这些强加的限制会导致第三方的开发人员不知道如何使用。如果他们要请求一个特定的集合，而这个集合是从已有结果中重新生成的，那他们查看的条目数不会超过100条，接下来就是他们的工作，指出将出使用限制过滤出合适的数据。是他们的`ORM`有bug还是做了限制，不能超过100？还是因为网络问题，将一个大的数据包拆分了？

_Minimize the arbitrary limits imposed on Third Party Developers._

对第三方开发人员的限制尽可能最小化。

_It is important, however, that you do offer the ability for a Consumer to specify some sort of filtering/limitation of the results. The most important reason for this is that the network activity is minimal and the Consumer gets their results back as soon as possible. The second most important reason for this is the Consumer may be lazy, and if the Server can do filtering and pagination for them, all the better. The not-so-important reason (from the Consumers perspective), yet a great benefit for the Server, is that the request will be less resource heavy._










