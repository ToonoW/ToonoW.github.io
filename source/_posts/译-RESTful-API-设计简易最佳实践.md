---
title: 译-RESTful API 设计简易最佳实践
date: 2018-04-08 17:34:34
tags:
- RESTful
---

想着要设计出符合 RESTful 规范的接口，所以看了不少文章，看到这篇想要翻译以加深印象和方便查阅。[原文在此](https://blog.philipphauer.de/restful-api-design-best-practices/)。

  

# 正文

由于没有强制性的官方标准，设计 HTTP 和 RESTful API 会非常棘手。基本上，API 的实现方式有很多种，但其中一些已经在实践中证明和被广泛采用。这篇博客介绍了构建 HTTP 和 RESTful API 的最佳实践。我们会谈论到 URL 结构，HTTP 方法，创建和修改资源，设计关联，数据装载格式，分页，版本和其他更多内容。

![](https://ws4.sinaimg.cn/large/006tKfTcgy1fq5ec4sxhlj30ci064t8l.jpg)

## 2018 更新内容

我完全重写了这篇博客。我重读和拓展了现有的章节，并且增加新的章节：新的 HTTP 方法和状态码概述，PATCH，PUT 和 POST 准确语意，`data` 字段，设计关联，REST vs RPC 风格 API，版本演进，基于键值的分页，JSON:API，JSON:API 启发的装载格式。



## 每个资源使用两个 URL

一个资源集合的 URL，一个单个资源的 URL：

```
# 反映一个资源的集合的 URL
/employees
# 反应单个资源的 URL
/employees/56
```

## 一致使用复数名词

正例
```
/employees
/employees/21
```

反例
```
/employee
/employee/21
```

确实，这只是一个喜好问题，但是复数形式更为通用。此外，复数更加直观，特别是使用 GET 请求资源集合的 URL 的时候（GET /employee 返回多个 employees）。但是更重要的是：避免混合使用复数和单数形式，那样会让人困惑而且容易犯错。

## 使用名词而不是动词来表示资源

这将会使您的 API 简单而且 URL 数量少。不要这么做：
```
/getAllEmployees
/getAllExternalEmployees
/createEmployee
/updateEmployee
```

作为代替，我们可以使用合适的 HTTP 方法作用在较小的 URL 集合上，来表达我们想要的动作。请看下一节。

## HTTP 方法

### 使用 HTTP 方法操作你的资源

```
GET /employees
GET /employees?state=external
POST /employees
PUT /employees/56
```

使用 URL 可以指定你想要操作的资源，然后使用 HTTP 方法表达你想要如何操作这个资源。使用五个 HTTP 方法 GET、POST、PUT、PATCH 和 DELETE 你可以提供 CRUD 功能（创建，读取，更新，删除）和其他的功能。

* 读取：使用 GET 读取资源。
* 创建：使用 POST 或 PUT 创建新的资源
* 更新：使用 PUT 或 PATCH 更新现有的资源。
* 删除：使用 DELETE 删除现有的资源。

### 明白 HTTP 方法的语义

幂等：如果我们可以多次重复触发一个请求，得到的结果和仅触发一次请求是一样的，那么我们可以称这个 HTTP 方法是幂等的。

* GET
  * 幂等
  * 只读。GET 请求永远不会修改在服务器上的资源状态。它必须是无副作用的。
  * 于是，响应数据可以被安全地缓存。
  * 例如:
    * `GET /employees` - 列出所有的 employees
    * `GET /employees/1` - 展示 employee 1 的详细信息
* PUT
  * 幂等！
  * 可以被用作创建和更新
  * 一般用来更新（全部信息更新）。
    * 例如：`PUT /employees/1` -  更新 employee 1 （不常用：创建 employee 1）
  * 使用 PUT 创建，客户端需要知道完整的 URL （包括 ID）。但一般情况是服务端生成ID。所以 PUT 创建一般使用在只有一个元素需要被创建，而且 URL 是明确的时候。
    * 例如：`PUT /employees/1/avatar` - 创建或者更新 employee 1 的头像。每一个 employee 仅有一个头像。
  * 通常一个请求会包含所有信息。PUT 不是用于局部信息的更新（参看 PATCH）。
* POST
  * 不幂等！
  * 用于创建
  * 例如：`POST /employees` 创建一个新的 employee。新的 URL 会放在请求头 `Location` （例如：`Location: /employees/12`）中返回给客户端。多次 POST 请求 `/employees` 会创建多个不同的 employees（这就是 POST 不幂等的原因）。
* PATCH
  * 幂等
  * 用在局部更新
  * 例如：`PATCH /employees/1` - 用请求中装载的字段来更新 employee 1，employee 1 的其他字段不会被改动。
* DELETE
  * 幂等
  * 用于删除
  * 例如：`DELETE /employees/1`



### POST 请求资源集合 URL 创建新资源

下图客户端/服务器创建新资源的交互是怎样的？

![POST-for-Creation](https://ws3.sinaimg.cn/large/006tKfTcly1fq66xtc4q9j31320omjtw.jpg)

1. 客户端发送一个 POST 请求给资源集合 URL `/employees`。HTTP 请求体包含了新资源“Paul”的所有属性。
2. RESTful 的服务器为这个资源生成一个 ID，在服务器内部的数据模型创建一个 employee，并且发送响应数据给客户端。响应数据包括了状态码201（已创建）和一个 `Location` HTTP 请求头，这个 `Location` 中的 URL 标识了资源的准确地址。



### PUT 请求单个资源的 URL 去更新资源

![](https://ws1.sinaimg.cn/large/006tKfTcgy1fq674wuymej312u0o8di4.jpg)

1. 客户端发送一个 PUT 请求到单个资源的 URL `/employee/21`。PUT 方法 HTTP 请求体 employee 所有的字段内容，并且这些字段将会在服务端更新到数据模型上。
2. REST 服务更新 employee ID 为21 的 `name` 和 `status` 字段，并且会返回状态码200。



### 使用 PATCH 局部更新资源

PUT 不支持局部更新。不应该完全替换一个资源的内容。每一次发送所有字段（即使你只希望更新一个字段）可能会导致在并行更新时意外覆盖。另外，验证实现起来很困难，如果你同事要支持这两种操作情况：创建（一些字段不能够为 `null`）和更新（`null` 标记的字段不应该被更新）。所以别使用 PUT 来更新单个字段的内容。PUT 请求中缺少的字段应被视为 `null` 并清空数据库字段或出发验证错误。

作为替代，使用 PATCH 进行局部更新。只更新发送的字段。这样子请求装载的参数会更加直观，不同字段的并行更新不会覆盖不想管的字段，验证起来也会更加简单，`null` 的语意也会明确（对于 PUT 和 PATCH）和节省带宽。

例如，下面的 PATCH 请求只更新 `status` 字段，不更新 `name` 字段。

![](https://ws4.sinaimg.cn/large/006tNc79gy1fq8un7cserj315m0uktbz.jpg)

实现的注意事项：除了描述的“只发送你想更新的”（这也是 [JSON:API](http://jsonapi.org/format/#crud-updating) 推荐的），还有 [JSON-PATHCH](http://jsonapi.org/format/#crud-updating)。它是 PATCH 请求的有效荷载格式，并描述了应该在资源上执行的一些列更新。然而，很难实现而且很多情况下会过度使用。更详细的介绍，参看“[PUT vs PATCH vs JSON-PATCH](https://philsturgeon.uk/api/2016/05/03/put-vs-patch-vs-json-patch/)”。

## 封装数据到 `data` 字段里

`GET /employees` 返回的对象数组存在 `data` 字段中：

```
{
  "data": [
    { "id": 1, "name": "Larry" }
    , { "id": 2, "name": "Peter" }
  ]
}
```

`GET /employees/1` 返回单个对象存在 `data` 字段中：

```
{
  "data": { 
    "id": 1, 
    "name": "Larry"
  }
}
```

## 对可选参数和复杂参数使用查询字符串（？）

不要这么做：

```
GET /employees
GET /externalEmployees
GET /internalEmployees
GET /internalAndSeniorEmployees
```

保持你的 URL 简短。为你的资源选择一个基本的 URL。将复杂或者可选的参数放到查询字符串中去使用。

```
GET /employees?state=internal&title=senior
GET /employees?id=1,2
```

JSON:API 格式的过滤：

```
GET /employees?filter[state]=internal&filter[title]=senior
GET /employees?filter[id]=1,2
```

## 使用 HTTP 状态码

RESTful 的 Web 服务应该给客户端的请求返回的合适的 HTTP 状态码。

* `2xx` - 成功 - 所有任务都执行顺利。
* `4xx` - 客户端错误 - 可能客户端请求错误（例如，客户端发送了一个无效请求或者请求是非法的）
* `5xx` - 服务器错误 - 服务端处理失败（服务端在处理请求的时候发生了错误，例如数据库错误、依赖的服务不可用、程序报错或者是出现意料之外的状态）

我们可以考虑使用这些[可用的 HTTP 状态码](http://en.wikipedia.org/wiki/List_of_HTTP_status_codes)。但是请注意，使用它们有可能会让 API 的用户感到困惑，所以尽量缩小使用的 HTTP 状态码集合。以下列举了常用的状态码：

* **2xx: 成功**
  * 200 OK
  * 201 创建成功
* **3xx: 重定向**
  * 301 永久移动（告诉用户原有的页面永久移动了）
  * 304 未修改（告诉用户页面自上次访问后未曾修改，假如是告诉爬虫机器人就可以避免重复爬取，节省带宽）
* **4xx: 客户端错误**
  * 400 错误请求
  * 401 未验证
  * 403 禁止访问
  * 404 找不到
  * 410 已删除
* **5xx: 服务器错误**
  * 500 内部服务器错误

**不要过度使用 404**。要尽可能精确地描述。如果资源是可用的，但是不允许用户查看，你可以返回一个 403 禁止访问的状态码。如果资源曾经存在，但是现在已经被删除或者停用了，请使用 410 已删除状态码。

## 提供有用的错误信息

除了适当的状态码之外，你还应该在 HTTP 响应正文中提供有用的的详细错误说明。这是一个例子。

请求：

```
GET /employees?state=super
```

响应：

```
// 400 Bad Request
{
  "errors": [
    {
      "status": 400,
      "detail": "Invalid state. Valid values are 'internal' or 'external'",
      "code": 352,
      "links": {
        "about": "http://www.domain.com/rest/errorcode/352"
      }
    }
  ]
}
```

提出错误的有效荷载结构受 [JSON:API 标准](http://jsonapi.org/format/#errors)的启发。

## 提供导航访问 API 的链接（HATEOAS）

理想情况下，你不需要让客户端构建访问 REST API 的 URL。让我们分析下面的例子。

客户端想要访问员工的工资报表。因此他必须知道他可以将查询参数 `salaryStatements` 附加到员工 URL （例如，`/employees/21/salaryStatements`）来访问工资报表。这些字符串的连接容易出错，而且零散难以维护。假如你的 REST API 临时改变了访问工资报表的访问方式（例如，改用 “salary-statements” or “paySlips”）所有的客户端都会无法工作。

最好的方式是你的响应中带有接下来可能会访问的 URL。例如，请求 `GET /employees` 的响应内容可能看起来是这样的：

```
{
  "data": [
    {
      "id":1,
      "name":"Paul",
      "links": [
        {
          "salary": "http://www.domain.com/employees/1/salaryStatements"
        }
      ]
    }
  ]
}
```

如果客户端完全依赖响应数据中提供的链接去获取工资报表，那么就算我们更改工资报表的 API，客户端也不会受到影响，因为客户端始终拿到的是最新有效的 URL（只要你在更改 API 的时候同时更新响应内容中的链接）。

## 恰当地设计关联

现在假定每一个员工 `employee` 有一个经理 `manager` 和多个团队成员 `teamMembers`。基本上有三种常见的选择来设计 API 中的关系：链接、侧载（Sideloading）、嵌入。

它们都是有效的，需要根据具体的场景选择。一般来说，你应该根据客户端的访问模式和请求数量和请求有效负载的大小来设计关系。

### 链接

```json
{
  "data": [
    { 
      "id": 1, 
      "name": "Larry",
      "relationships": {
        "manager": "http://www.domain.com/employees/1/manager",
        "teamMembers": [ 
          "http://www.domain.com/employees/12",
          "http://www.domain.com/employees/13"
        ]
        //or "teamMembers": "http://www.domain.com/employees/1/teamMembers"
      }
    }
  ]
}
```

* 小负载。如果客户端不是每次都需要 `manager` 和 `teamManager`，这样会节省资源。
* 请求数量多。如果几乎每个客户端都需要 `manager` 和 `teamManager` ，客户端就必须要发起多次请求才拿到完整的数据，这样不但繁琐反而还浪费资源。而且这种情况会根据关系（`manager`, `teamMembers`）的数量成倍增长。
* 客户端必须将数据拼接在一起看能看到数据的全貌。

### 侧载（Sideloading）

我们可以利用外键引用的关系，将引用的对象也放到我们的响应数据中返回给客户端，放在专用字段 `included` 中。这种方法也被称为“复合文档”。

```json
{
  "data": [
    { 
      "id": 1, 
      "name": "Larry",
      "relationships": {
        "manager":  5 , 
        "teamMembers": [ 12, 13 ]
      }
    }
  ],
  "included": {
    "manager": {
      "id": 5, 
      "name": "Kevin"
    },
    "teamMembers": [
      { "id": 12, "name": "Albert" }
      , { "id": 13, "name": "Tom" }
    ]
  }
}
```

客户端也可以通过查询参数来控制侧载（Sideloading）的内容，像 `GET /employees?include=manager,teamMembers`。

* 我们只需要发起单个请求。
* 可控的负载数据大小。不重复（例如，可能你请求多个员工的数据，但是返回中只会出现一个经理的数据）。
* 客户端仍然需要将数据拼接在一起才能解决关系引用的问题，这可能非常麻烦。

### 内嵌

```json
{
  "data": [
    { 
      "id": 1, 
      "name": "Larry",
      "manager": {
        "id": 5, 
        "name": "Kevin"
      },
      "teamMembers": [
        { "id": 12, "name": "Albert" }
        , { "id": 13, "name": "Tom" }
      ]
    }
  ]
}
```

* 对客户端来说最方便。可以直接根据关系来获取到实际的数据。
* 如果有客户不需要的关系关联到的对象数据，那么会浪费资源。
* 增加负载数据的大小，并且有冗余数据。被关系引用的对象内容可能被嵌入多次。

未完待续...