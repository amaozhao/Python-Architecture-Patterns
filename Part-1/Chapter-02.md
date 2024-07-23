# API 设计

在本章中，我们将讨论基本的应用程序编程接口 (API) 设计原则。我们将了解如何通过定义有用的抽象来开始我们的设计，这些抽象将为设计奠定基础。

然后，我们将介绍 RESTful 接口的原则，涵盖严格的学术定义和更实用的定义，以帮助进行设计。我们将研究设计方法和技术，以帮助创建基于标准实践的有用 API。我们还将花一些时间讨论身份验证，因为这是大多数 API 的关键元素。

我们将在本书中关注 RESTful 接口，因为它们是目前最常见的。在此之前，还有其他替代方案，包括 80 年代的远程过程调用 (RPC)，一种进行远程函数调用的方法，或 2000 年代初期的单对象访问协议 (SOAP)，它标准化了远程调用的格式。当前的 RESTful 接口更易于阅读，并且更强大地利用了已经建立的 HTTP 使用，尽管从本质上讲，它们可能会通过这些旧规范进行集成。

它们现在仍然可用，尽管主要在旧系统中。

我们将介绍如何为 API 创建版本控制系统，处理可能受到影响的不同用例。

我们将看到前端和后端之间的区别，以及它们的交互。尽管本章的主要目标是讨论 API 接口，但我们还将讨论 HTML 接口以了解它们之间的差异以及它们如何与其他 API 交互。

最后，我们将描述我们将在本书后面使用的示例的设计。

在本章中，我们将介绍以下主题：

- 抽象
- RESTful 接口
- 验证
- 版本控制 API
- 前端和后端
- HTML 接口
- 为示例设计 API

我们先来看看抽象。

## 抽象

API 允许我们在不完全了解所涉及的所有不同步骤的情况下使用一个软件。它提供了一个清晰的可执行操作菜单，使不一定了解操作复杂性的外部用户能够有效地执行这些操作。它提出了该过程的简化。

这些动作可以是纯功能性的，输出仅与输入相关；例如，一个数学函数，根据它们的轨道和质量计算行星和恒星的重心。

或者，它们可以处理状态，因为重复两次相同的动作可能会产生不同的效果；例如，检索系统中的时间。甚至一次调用也可能允许设置计算机的时区，随后两次调用检索时间可能会返回非常不同的结果。

在这两种情况下，API 都在定义抽象。在一次操作中检索系统的时间很简单，但这样做的细节可能并不那么容易。它可能涉及以某种方式读取一些跟踪时间的硬件。

不同的硬件可能会以不同的方式报告时间，但结果应始终以标准格式进行转换。需要应用时区和节省时间。所有这些复杂性都由模块的开发人员处理，该模块公开 API 并为任何用户提供清晰易懂的合同。 “调用此函数，将返回ISO格式的时间。”

> 虽然我们主要讨论 API，并且在整本书中我们将主要描述与在线服务相关的 API，但抽象的概念实际上可以应用于任何事物。管理用户的网页是一种抽象，因为它定义了“用户帐户”的概念和相关参数。另一个无处不在的例子是电子商务的“购物车”。创建清晰的心理形象是件好事，因为它有助于为用户创建更清晰、更一致的界面。

这当然是一个简单的例子，但是 API 可以在其接口下隐藏大量的复杂性。一个很好的例子是 curl 之类的程序。即使只是向 URL 发送 HTTP 请求并打印返回的标头，也有大量的复杂性与此相关：

```sh
$ curl -IL http://google.com
HTTP/1.1 301 Moved Permanently
Location: http://www.google.com/
Content-Type: text/html; charset=UTF-8
Date: Tue, 09 Mar 2021 20:39:09 GMT
Expires: Thu, 08 Apr 2021 20:39:09 GMT
Cache-Control: public, max-age=2592000
Server: gws
Content-Length: 219
X-XSS-Protection: 0
X-Frame-Options: SAMEORIGIN
HTTP/1.1 200 OK
Content-Type: text/html; charset=ISO-8859-1
P3P: CP="This is not a P3P policy! See g.co/p3phelp for more info."
Date: Tue, 09 Mar 2021 20:39:09 GMT
Server: gws
X-XSS-Protection: 0
X-Frame-Options: SAMEORIGIN
Transfer-Encoding: chunked
Expires: Tue, 09 Mar 2021 20:39:09 GMT
Cache-Control: private
Set-Cookie: NID=211=V-jsXV6z9PIpszplstSzABT9mOSk7wyucnPzeCz-TUSfOH9_F-07V6-fJ5t9L2eeS1WI-p2G_1_zKa2Tl6nztNH-ur0xF4yIk7iT5CxCTSDsjAaasn4c6mfp3fyYXMp7q1wA2qgmT_hlYScdeAMFkgXt1KaMFKIYmp0RGvpJ-jc; expires=Wed, 08-Sep-2021 20:39:09 GMT; path=/; domain=.google.com; HttpOnly
```

这会调用 www.google.com 并使用 -I 标志显示响应的标题。添加 -L 标志以自动重定向这里发生的任何请求。

与服务器建立远程连接需要许多不同的移动部件：

- DNS 访问将服务器地址 www.google.com 转换为实际的 IP 地址。
- 两台服务器之间的通信，涉及使用 TCP 协议生成持久连接并保证数据的接收。
- 重定向基于第一个请求的结果，因为服务器返回指向另一个 URL 的代码。这是由于使用了 -L 标志而完成的。
- 重定向指向一个 HTTPS URL，这需要在其之上添加一个验证和加密层。

这些步骤中的每一个还利用其他 API 来执行较小的操作，这可能涉及操作系统的功能，甚至调用远程服务器（例如 DNS）从那里获取数据。

> 在这里，curl 界面是从命令行使用的。虽然 API 丢弃的严格定义规定最终用户是人，但实际上并没有太大的变化。好的 API 应该很容易被人类用户测试。命令行界面也可以通过 bash 脚本或其他语言轻松实现自动化。

但是，从 curl 用户的角度来看，这并不是很重要。它被简化为带有几个标志的单个命令行可以执行定义明确的操作，而无需担心从 DNS 获取数据的格式或如何使用 SSL 加密请求。

### 使用正确的抽象

对于一个成功的界面来说，根本在于创建一系列抽象并将它们呈现给用户，以便他们可以执行操作。因此，在设计新 API 时，最重要的问题是决定哪些是最好的抽象。

当这个过程有机地发生时，抽象大多是在旅途中决定的。有一个最初的想法，被认为是对问题的理解，然后会被调整。

例如，通过向用户添加不同的标志来启动用户管理系统是很常见的。因此，用户有权执行操作 A，然后具有执行操作 B 的参数，依此类推。通过一次添加一个标志，即第十个标志，该过程变得非常混乱。

然后，可以使用新的抽象；角色和权限。某些类型的用户可以执行不同的操作，例如管理员角色。一个用户可以有一个角色，角色是描述相关权限的角色。

请注意，这简化了问题，因为它易于理解和管理。但是，从“一个单独的标志集合”转移到“多个角色”可能是一个复杂的过程。可能选项的数量有所减少。也许某些现有用户具有特殊的标志组合。所有这些都需要小心处理。

在设计新的 API 时，最好尝试明确地描述 API 用来阐明它们的固有抽象，至少在高层次上是这样。这还有一个好处，就是能够以 API 的用户的身份考虑这一点，看看事情是否加起来。

> 软件开发人员工作中最有用的观点之一是将自己从“内部视图”中分离出来，站在软件实际用户的位置上。这比听起来要困难，但它肯定是一项值得发展的技能。这将使你成为更好的设计师。不要害怕请朋友或同事来检测你设计中的盲点。

然而，每个抽象都有其局限性。

### 泄露抽象

当抽象从实现中泄漏细节，并且没有呈现完全不透明的图像时，它被称为泄漏抽象。

虽然一个好的 API 应该尽量避免这种情况，但有时它会发生。这可能是由服务于 API 的代码中的潜在错误引起的，或者有时直接来自代码在某些操作中的操作方式。

一个常见的情况是关系数据库。 SQL 从数据在数据库中的实际存储方式中抽象出搜索数据的过程。你可以使用复杂的查询进行搜索并获得结果，并且你无需了解数据的结构。但有时，你会发现某个特定查询很慢，而重新组织查询的参数对这种情况的发生方式有很大影响。这是一个泄漏的抽象。

> 这是很常见的，也是为什么有重要的工具可以帮助确定运行 SQL 查询时发生的事情的原因，这与实现非常分离。主要的是 EXPLAIN 命令。

操作系统是生成良好抽象且大多数时间不会泄漏的系统的很好示例。有很多例子。由于空间不足而无法读取或写入文件（现在比三年前更不常见的问题）；由于网络问题而中断与远程服务器的连接；或者由于打开文件描述符的数量达到限制而无法创建新连接。

在某种程度上，泄漏的抽象是不可避免的。它们是不生活在完美世界的结果。软件容易出错。理解并为此做好准备至关重要。

> “在某种程度上，所有重要的抽象都是有漏洞的。”
> -- Joel Spolsky 的泄漏抽象定律

在设计 API 时，出于以下几个原因考虑这一事实很重要：

- 在外部呈现清晰的错误和提示。一个好的设计将始终包含出现问题的案例，并尝试使用适当的错误代码或错误处理清楚地呈现它们。
- 处理可能来自内部依赖服务的错误。相关服务可能会失败或出现其他类型的问题。 API 应该在一定程度上对此进行抽象，如果可能的话从问题中恢复，如果没有，则优雅地失败，如果无法恢复，则返回正确的结果。

最好的设计是这样一种设计，它不仅可以按预期设计事物，还可以为意外问题做好准备，并确保可以对其进行分析和纠正。

### 资源和动作抽象

设计 API 时要考虑的一个非常有用的模式是生成一组可以执行操作的资源。此模式使用两种元素：资源和操作。

资源是被引用的被动元素，而对资源执行操作。

例如，让我们定义一个非常简单的接口来玩一个简单的猜硬币游戏。这是一个由三个投掷硬币的三个猜测组成的游戏，如果这些猜测中至少有两个是正确的，则用户获胜。

资源和操作可能如下：

| 资源   | 动作 | 细节                                        |
| ------ | ---- | ------------------------------------------- |
| HEADS  | None | 掷硬币结果。                                |
| TAILS  | None | 掷硬币结果。                                |
| GAME   | 开始 | 开始一个新的游戏。                          |
|        | READ | 返回当前回合（1 到 3）和当前正确猜测。      |
| 掷硬币 | 掷   | 抛硬币。 如果 GUESS 尚未生成，则返回错误。  |
|        | 猜   | 接受 HEADS 或 TAILS 作为猜测。              |
|        | 结果 | 它返回 HEADS 或 TAILS 以及 GUESS 是否正确。 |

单个游戏的可能顺序可能是：

```
GAME START
> (GAME 1)
GAME 1 COIN_TOSS GUESS HEAD
GAME 1 COIN_TOSS TOSS
GAME 1 COIN_TOSS RESULT
> (TAILS, INCORRECT)
GAME 1 COIN_TOSS GUESS HEAD
GAME 1 COIN_TOSS TOSS
GAME 1 COIN_TOSS RESULT
> (HEAD, CORRECT)
GAME 1 READ
> (ROUND 2, 1 CORRECT, IN PROCESS)
GAME 1 COIN_TOSS GUESS HEAD
GAME 1 COIN_TOSS TOSS
GAME 1 COIN_TOSS RESULT
> (HEAD, CORRECT)
GAME 1 READ
> (ROUND 3, 2 CORRECT, YOU WIN)
```

请注意每个资源如何有自己的一组可以执行的操作。如果方便，可以重复操作，但这不是必需的。资源可以组合成分层表示（就像这里一样，其中 COIN_TOSS 依赖于更高的 GAME 资源）。操作可能需要可以是其他资源的参数。

但是，抽象是围绕具有一组一致的资源和操作进行组织的。这种明确组织 API 的方式很有用，因为它可以明确系统中的被动和主动。

> 面向对象编程 (OOP) 使用这些抽象，因为一切都是可以接收消息以执行某些操作的对象。另一方面，函数式编程并不完全适合这种结构，因为“动作”可以像资源一样工作。

这是一种常见的模式，它用于 RESTful 接口，我们将在接下来看到。

## RESTful 接口

如今，RESTful 接口非常普遍，而且有充分的理由。它们已成为服务于其他应用程序的 Web 服务的事实标准。

代表性状态转移 (REST) 于 2000 年在博士学位中定义。 Roy Fielding 的论文，它使用 HTTP 标准作为基础来创建软件架构风格的定义。

对于一个被认为是 RESTful 的系统，它应该遵循一定的规则：

- 客户端-服务器架构。它通过远程调用工作。
- 无状态。与特定请求相关的所有信息都应包含在请求本身中，使其独立于为请求提供服务的特定服务器。
- 可缓存性。响应的可缓存性应该很清楚，可以说它们是否可缓存。
- 分层系统。客户端无法判断它们是否连接到最终服务器或是否有中间服务器。
- 统一的界面，有四个先决条件：
    - 请求中的资源标识，意味着资源是明确表示的，并且它的表示是独立的
    - 通过表示进行资源操作，允许客户在获得表示时获得所有需要的信息来进行更改
    - 自我描述的消息，意味着消息本身是完整的
    - 超媒体作为应用程序状态的引擎，这意味着客户端可以使用引用的超链接浏览系统
- 按需编码。这是一个可选要求，通常不使用。服务器可以提交代码以帮助执行操作或改进客户端；例如，提交要在浏览器中执行的 JavaScript。

这是最正式的定义。如你所见，它不一定基于 HTTP 请求。为了更方便的使用，我们需要对可能性进行一些限制，并设置一个通用的框架。

### 更实用的定义

当人们通俗地谈论 RESTful 接口时，通常将其理解为基于 HTTP 资源使用 JSON 格式请求的接口。这与我们之前看到的定义完全兼容，但考虑了一些关键要素。

> 这些关键元素有时会被忽略，导致伪 RESTful 接口不具有相同的属性。

主要的一个是 URI（统一资源标识符）应该使用 CRUD（创建检索更新删除）方法描述清晰的资源，以及对它们执行的 HTTP 方法和操作。

> CRUD 接口有助于执行这些操作：创建（保存新条目）、检索（读取）、更新（覆盖）和删除条目。这些是任何持久存储系统的基本操作。

有两种 URI，无论是描述单个资源还是一组资源，如下表所示：

| 资源       | 例子     | 方法   | 描述                                                         |
| ---------- | -------- | ------ | ------------------------------------------------------------ |
| Collection | /books   | GET    | 列表操作。返回集合的所有可用元素，例如，所有书籍。           |
|            |          | POST   | 创建操作。创建集合的新元素。返回新创建的资源。               |
| Single     | /books/1 | GET    | 检索操作。从资源中返回数据，例如 ID 为 1 的书籍。            |
|            |          | PUT    | 设置（更新）操作。发送资源的新数据。如果它不存在，它将被创建。如果是这样，它将被覆盖。 |
|            |          | PATCH  | 部分更新操作。仅覆盖资源的部分值，例如，仅发送和写入用户对象的电子邮件。 |
|            |          | DELETE | 删除操作。它删除资源。                                       |

正如我们之前看到的，这种设计的关键元素是将所有东西都定义为资源。资源由它们的 URI 定义，其中包含资源的分层视图，例如：

```/books/1/cover``` 定义 ID 为 1 的图书封面图片的资源。

> 为简单起见，我们将使用整数 ID 来标识本章中的资源。在实际操作中，不建议这样做。它们根本没有任何意义，更糟糕的是，它们有时会泄露有关系统中元素数量或其内部顺序的信息。例如，竞争对手可以估计每周有多少新条目被添加。要脱离任何内部表示，请尝试始终在外部使用自然密钥（如果可用），例如书籍的 ISBN 号，或创建随机的通用唯一标识符 (UUID)。
>
> 顺序整数的另一个问题是，在高速率下，系统可能难以正确创建它们，因为不可能同时创建两个。这会限制系统的发展。

资源的大部分输入和输出将以 JSON 格式表示。例如，这可能是检索用户的请求和响应示例：

```
GET /books/1
HTTP/1.1 200 OK
Content-Type: application/json
{"name": "Frankenstein", "author": "Mary Shelley", "cover": "http://library.lbr/books/1/cover"}
```

响应采用 JSON 格式，如 ```Content-Type``` 中所指定。这使得自动解析和分析变得容易。请注意，```avatar```字段返回指向另一个资源的超链接。这使得界面可步行并减少了客户端事先需要的信息量。

> 这是设计 RESTful 接口时最容易被遗忘的属性之一。最好将完整的 URI 返回到资源而不是间接引用，例如无上下文 ID。
>
> 例如，在创建新资源时，在响应的 Location 标头中包含新的 URI。

要发送要覆盖的新值，应使用相同的格式。请注意，某些元素可能是只读的，例如封面，并且不是必需的：

```
PUT /books/1
Content-Type: application/json
{"name": "Frankenstein or The Modern Prometheus", "author": "Mary Shelley"}
HTTP/1.1 200 OK
Content-Type: application/json
{"name": "Frankenstein or The Modern Prometheus", "author": "Mary Shelley", "cover": "http://library.com/books/1/cover"}
```

输入和输出应该使用相同的表示，使客户端可以轻松地检索资源、修改它，然后重新提交它。

> 这真的很方便，并创建了在实现客户端时非常受欢迎的一致性级别。在测试时，尽量确保检索值并重新提交它是有效的并且不会产生问题。

当资源将直接由二进制内容表示时，它可以返回正确的格式，在 ```Content-Type``` 标头中指定。例如，检索头像资源可能会返回一个图像文件：

```
GET /books/1/cover
HTTP/1.1 200 OK
Content-Type: image/png
...
```

同样，在创建或更新新头像时，应以正确的格式发送。

> 虽然 RESTful 接口的初衷是利用多种格式，例如接受 XML 和 JSON，但这在实践中并不常见。总的来说，JSON 是当今最标准的格式。不过，某些系统可能会受益于使用多种格式。

另一个重要属性是确保某些操作是**幂等**的，而其他操作则不是。幂等动作可以重复多次，产生相同的结果，而重复**非幂等**动作会产生不同的结果。显然，动作应该是相同的。

一个明显的例子是创建一个新元素。如果我们为资源列表的新元素提交两个相同的 ```POST``` 创建，它将创建两个新元素。例如，提交具有相同名称和作者的两本书将创建两本相同的书籍。

> 这是假设资源的内容没有限制。如果是，则第二个请求将失败，无论如何都会产生与第一个请求不同的结果。

另一方面，两个 ```GET``` 请求将产生相同的结果。 ```PUT``` 或 ```DELETE``` 也是如此，因为它们将覆盖或“再次删除”资源。

唯一的非幂等请求是 ```POST``` 操作这一事实显着简化了在是否应该重试问题时处理问题的措施设计。幂等请求可以随时安全重试，从而简化网络问题等错误的处理。

### 标题和状态

有时会被忽略的 HTTP 协议的一个重要细节是不同的标头和状态代码。

标头包括有关请求或响应的元数据信息。其中一些是自动添加的，例如请求或响应正文的大小。需要考虑的一些有趣的标题如下：

| Header            | Type     | Details                                                      |
| ----------------- | -------- | ------------------------------------------------------------ |
| Authorization     | Standard | 用于验证请求的凭据。                                         |
| Content-Type      | Standard | 请求正文的类型，例如 application/json 或 text/html。         |
| Date              | Standard | 创建消息的时间。                                             |
| If-Modified-Since | Standard | 发件人此时拥有资源的副本。如果此后未更改，则可以返回 304 Not Modified 响应（正文为空）。这允许缓存数据并通过不返回重复信息来节省时间和带宽。这可以在 GET 请求中使用。 |
| X-Forwarded-From  | 事实标准 | 存储消息来源的 IP，以及它经过的不同代理。                    |
| Forwarded         | Standard | 与 X-Forwarded-From 相同。这是一个较新的标头，比 X-Forwarded-From 更不常见。 |

一个设计良好的 API 将使用标头来传达正确的信息，例如，正确设置 Content-Type 或在可能的情况下接受缓存参数。

> 可以在 https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers 找到完整的标头列表。

另一个重要的细节是充分利用可用的状态码。状态码提供了有关所发生情况的重要信息，并且为每种情况使用尽可能详细的信息将提供更好的界面。

一些常见的状态码如下：

| 状态码                  | 描述                                                         |
| ----------------------- | ------------------------------------------------------------ |
| 200 OK                  | 成功的资源访问或修改。它应该返回一个主体；如果没有，请使用 204 No Content。 |
| 201 Created             | 创建新资源的成功 POST 请求。                                 |
| 204 No Content          | 不返回正文的成功请求，例如成功的 DELETE 请求。               |
| 301 Moved Permanently   | 访问的资源现在永久位于不同的 URI 中。它应该返回带有新 URI 的 Location 标头。大多数库将自动跟进 GET 访问。例如，API 只能通过 HTTPS 访问，但它是通过 HTTP 访问的。 |
| 302 Found               | 访问的资源暂时位于不同的 URI 中。一个典型的例子是，如果经过身份验证，就会被重定向到登录页面。 |
| 304 Not Modified        | 缓存的资源仍然有效。身体应该是空的。此状态码仅在客户端请求缓存信息时返回，例如，使用 If-Modified-Since 标头。 |
| 400 Bad Request         | 请求中的一般错误。这是服务器说，“你这边出了点问题。”应在正文中添加更具描述性的消息。如果可以使用更具描述性的状态代码，则应该首选它。 |
| 401 Unauthorized        | 不允许该请求，因为该请求未正确验证。请求可能缺少用于身份验证的有效标头。 |
| 403 Forbidden           | 请求已通过身份验证，但无法访问此资源。这与 401 Unauthorized 状态不同，因为请求已经正确验证但没有访问权限。 |
| 404 Not Found           | 可能是最著名的状态码！找不到 URI 描述的资源。                |
| 405 Method Not Allowed  | 无法使用请求的方法；例如，不能删除资源。                     |
| 429 Too Many Requests   | 如果客户端可以执行的请求数量受到限制，则服务器应返回此状态代码。它应该在正文中返回一个描述或更多信息，理想情况下，一个 Retry-After 标头指示下一次重试的时间（以秒为单位）。 |
| 500 Server Error        | 服务器中的一般错误。仅当服务器中发生意外错误时才应使用此状态。 |
| 502 Bad Gateway         | 服务器正在将请求重定向到不同的服务器，并且通信不正确。 当某些后端服务不可用或配置不正确时，通常会出现此错误。 |
| 503 Service Unavailable | 服务器当前无法处理请求。 通常，这是一种临时情况，例如负载问题。 它可用于标记维护停机时间，但这通常很少见。 |
| 504 Gateway Timeout     | 与 502 Bad Gateway 类似，但在这种情况下，后端服务没有响应，引发超时。 |

一般情况下，一般情况下应保留非描述性错误代码，例如 ```400 Bad Request``` 和 ```500 Server Error```。但是，如果有更好的、更具描述性的状态代码，则应该使用它。

例如，覆盖参数的 ```PATCH``` 请求应返回 ```400 Bad Request``` 如果参数因任何原因不正确，但如果找不到资源 ```URI```，则应返回 ```404 Not Found```。

> 还有其他状态码。你可以在此处查看完整列表，包括每个列表的详细信息：https://httpstatuses.com/。

在任何错误中，请向用户提供一些额外的反馈并说明原因。通用描述符可以帮助处理意外情况并简化调试问题。

> 这对于 4XX 错误特别有用，因为它们将帮助 API 用户修复他们自己的错误并迭代地改进他们的集成。

例如，提到的 PATCH 可能会返回这个主体：

```json
{
    "message": "Field 'address' is unknown"
}
```

这将提供有关该问题的具体细节。其他选项包括返回错误代码、多条消息以防出现多个可能的错误，以及在正文中复制状态代码。

### 设计资源

RESTful API 中的可用操作仅限于 CRUD 操作。因此，资源是 API 的基本构建块。

将所有东西都变成资源有助于创建非常明确的 API，并有助于满足对 RESTful 接口的无状态要求。

无状态服务意味着完成请求所需的所有信息要么由调用者提供，要么从外部检索，通常从数据库中检索。这不包括保存信息的其他方式，例如将信息本地存储在同一服务器的硬盘驱动器中。这使得任何服务器都能够处理每个请求，这对于实现可伸缩性至关重要。

可以通过创建不同动作来激活的元素可以分成不同的资源。例如，模拟笔的界面可能需要以下元素：

- 打开和关闭笔。
- 写东西。只有打开的笔才能写字。

在某些 API（如面向对象的 API）中，这可能涉及创建一个笔对象并更改其状态：

```
pen = Pen()
pen.open()
pen.write("Something")
pen.close()
```

在 RESTful API 中，我们需要为笔及其状态创建不同的资源：

```sh
# Create a new pen with id 1
POST /pens
# Create a new open pen for pen 1
POST /pens/1/open
# Update the new open text for the open pen 1
PUT /pens/1/open/1/text
# Delete the open pen, closing the pen
DELETE /pens/1/open/1
```

这可能看起来有点麻烦，但是 RESTful API 的目标应该是比典型的面向对象的 API 更高级别。要么直接创建文本，要么创建笔然后文本，而无需执行打开/关闭操作。

> 请记住，RESTful API 是在远程调用的上下文中使用的。这意味着它们不能是低级别的，因为与本地 API 相比，每次调用都是一笔巨大的投资，因为每次调用的时间将是操作的明智部分。

另请注意，每个方面和步骤都已注册并具有自己的一组标识符并且是可寻址的。这比在 OOP 中可以找到的内部状态更明确。正如我们所见，我们希望它是无状态的，而对象是非常有状态的。

> 请记住，资源不需要直接转换为数据库对象。这是从存储到 API 的倒退思考。请记住，你不仅限于此，还可以编写从多个来源获取信息或不适合直接翻译的资源。我们将在下一章中看到示例。

如果来自更传统的 OOP 环境，仅处理资源可能需要进行某些调整，但它们是一个非常灵活的工具，可以分配多种执行操作的方式。

### 资源和参数

虽然一切都是资源，但有些元素作为与资源交互的参数更有意义。这在修改资源时很自然。需要提交任何更改以更新资源。但是，在其他情况下，可以出于其他原因修改某些资源。最常见的情况是搜索。

典型的搜索端点将定义搜索资源并检索其结果。但是，没有过滤参数的搜索并没有真正的用处，因此需要额外的参数来定义搜索，例如：

```
# Return every pen in the system
GET /pens/search
# Return only red pens
GET /pens/search?color=red
# Return only red pens, sorted by creation date
GET /pens/search?color=red&sort=creation_date
```

这些参数存储在查询参数中，查询参数是检索它们的自然扩展。

> 作为一般规则，只有 GET 请求应该有查询参数。其他类型的请求方法应该提供任何参数作为主体的一部分。

如果包含查询参数，GET 请求也很容易缓存。如果搜索为每个请求返回相同的值，假设这是一个幂等请求，那么包括查询参数在内的完整 URI 甚至可以从服务外部缓存。

按照惯例，存储 GET 请求的所有日志也将存储查询参数，而作为标头或在请求正文中发送的任何参数都不会被记录。这具有安全隐患，因为任何合理的参数（例如密码）都不应该作为查询参数发送。

有时，这就是创建通常是 GET 请求的 POST 操作的原因，但更喜欢在请求正文中设置参数而不是查询参数。虽然在 HTTP 协议中可以在 GET 请求中设置正文，但这绝对是非常不寻常的。

> 这方面的一个例子可能是通过电话号码、电子邮件或其他个人信息进行搜索，因此中间人代理可以拦截并了解它们。

使用 POST 请求的另一个原因是允许更大的参数空间，因为包括查询参数在内的完整 URL 通常限制为 2K 大小，而正文的大小限制要少得多。

### 分页

在 RESTful 接口中，任何返回合理数量元素的 LIST 请求都应该分页。

这意味着可以从请求中调整元素和页面的数量，只返回特定页面的元素。这限制了请求的范围并避免了非常慢的响应时间和浪费传输字节。

一个示例可能涉及使用参数页面和大小，例如：

```
# Return only first 10 elements
GET /pens/search?page=1&size=10
```

一个结构良好的响应将具有与此类似的格式：

```json
{
    "next": "http://pens.pns/pens/search?page=2&size=10",
    "previous": null,
    "result": [
        # elements
    ]
}
```

它包含一个带有结果列表的结果字段以及作为指向下一页和上一页的超链接的下一页和上一页字段，如果不可用，则值为 null。这使得浏览所有结果变得容易。

> 排序参数也可用于确保页面的一致性。

这种技术还允许并行检索多个页面，这可以加快信息的下载速度，执行几个小请求而不是一个大请求。然而，目标是提供足够的过滤器参数，通常不会返回太多信息，只能检索相关信息。

分页有一个问题，即集合中的数据可能会在多个请求之间发生变化，尤其是在检索许多页面时。问题如下：

```
# Obtain first page
GET /pens/search?page=1&size=10&sort=name
# Create a new resource that is added to the first page
POST /pens
# Obtain second page
GET /pens/search?page=2&size=10&sort=name
```

第二页现在有一个重复的元素，它曾经在第一页上，但现在已经移到第二页，然后有一个元素没有返回。通常，新资源的不归还不是什么大问题，因为毕竟信息的检索在其创建之前就开始了。但是，同样的资源可以返回两次。

为了避免这种问题，默认情况下可以按创建日期或类似的方式对值进行排序。这样，任何新资源都将在分页结束时添加，并且将被一致地检索。

> 对于返回固有“新”元素的资源，如通知或类似元素，添加一个 updated_since 参数以仅检索自最近访问以来的新资源。这以实用的方式加快了访问速度，并仅检索相关信息。

创建一个灵活的分页系统可以增加任何 API 的实用性。确保你的分页定义在任何不同的资源中都是一致的。

### 设计 RESTful API 流程

开始设计 RESTful API 的最佳方式是清楚地说明资源，然后描述它们，包括以下详细信息：

- 描述：动作的描述
- 资源 URI：请注意，这可能为多个操作共享，由方法区分（例如，获取检索和删除删除）
- 适用的方法：用于此端点中定义的操作的 HTTP 方法
- （仅在相关时）输入正文：请求的输入正文
- 正文中的预期结果：结果
- 可能的预期错误：根据特定错误返回状态代码
- 描述：动作的描述
- （仅在相关时）输入查询参数：查询参数添加到 URI 以获得额外功能
- （仅在相关时）相关标头：任何受支持的标头
- （仅在相关时）异常返回状态码（200 和 201）：与错误不同，如果有一个状态码被认为是成功但不是通常的情况；例如，成功返回重定向

这足以创建一个其他工程师可以理解并允许他们在界面上工作的设计文档。

不过，最好先快速草拟不同的 URI 和方法，然后快速查看系统拥有的所有不同资源，而不涉及太多细节，例如正文描述或错误。这有助于检测 API 中缺少的资源缺口或其他类型的不一致。

例如，本章描述的 API 有以下动作：

```
GET    /pens
POST   /pens
POST   /pens/<pen_id>/open
PUT    /pens/<pen_id>/open/<open_pen_id>/text
DELETE /pens/<pen_id>/open/<open_pen_id>
GET    /pens/search
```

这里有几个细节可以调整和改进：

- 看起来我们忘记添加删除笔的动作，一旦创建
- 有几个 ```GET``` 操作可用于检索有关应添加的已创建资源的信息
- 在 ```PUT``` 动作中，感觉有点多余，不得不添加 /text

有了这个反馈，我们可以再次描述 API 如下（修改有一个箭头）：

```
GET    /pens
POST   /pens
GET    /pens/<pen_id> 
DELETE /pens/<pen_id> ←
POST   /pens/<pen_id>/open
GET    /pens/<pen_id>/open/<open_pen_id> ←
PUT    /pens/<pen_id>/open/<open_pen_id> ←
DELETE /pens/<pen_id>/open/<open_pen_id>
GET    /pens/search
```

请注意层次结构中的组织如何有助于仔细查看所有元素并找到乍一看可能并不明显的差距或关系。

在那之后，我们可以进入细节。我们可以使用本节开头描述的模板，或任何其他适合你的模板。例如，我们可以定义端点来创建新笔并在系统中读取笔：

#### 创建一支新笔：

- 描述：创建一个新的笔，指定颜色。

- 资源 URI：```/pen```

- 方法：POST

- 输入体：
    ```json
    {
        "name": <pen name>,
        "color": (black|blue|red)
    }
- 错误：
    ```
    400 Bad Request


正文中的错误，例如无法识别的颜色、重复的名称或错误的格式。

#### 检索现有的笔：

- 描述：检索现有的笔。

- 资源 URI：```/pens/\<pen id>```

- 方法：获取

- 返回正文：
  
    ```json
    {
        "name": <pen name>,
        "color": (black|blue|red)
    }
    
- 错误：
  
    ```
    404 Not Found 
    The pen ID is not found.

主要目标是这些小模板很有用且切中要害。随意按预期调整它们，不要担心对错误或细节过于完整。最重要的部分是它们很有用；例如，添加 405 Method Not Allowed 消息可能是多余的。

> API 也可以使用 Postman (www.postman.com) 等工具进行设计，Postman (www.postman.com) 是一个 API 平台，可用于设计或测试/调试现有 API。虽然很有用，但如果需要的话，最好能够在没有外部工具的情况下设计 API，因为它迫使你考虑设计而不是工具本身。我们还将看到如何使用 Open API，它更多地基于定义，而不是提供测试环境。

设计和定义 API 还可以使其在之后以标准方式构建以利用工具。

### 使用开放 API 规范

更结构化的替代方法是使用诸如 Open API (https://www.openapis.org/) 之类的工具。 Open API 是一种通过 YAML 或 JSON 文档定义 RESTful API 的规范。这允许此定义与其他工具交互以生成 API 的自动文档。

它允许定义可以重复的不同组件，作为输入和输出。这使得构建一致的可重用对象变得容易。还有一些方法可以相互继承或组合，从而创建一个丰富的界面。

详细描述整个 Open API 规范超出了本书的范围。大多数常见的 Web 框架都允许与之集成，自动生成 YAML 文件或我们稍后将看到的 Web 文档。它以前被称为 Swagger，它的网页 (https://swagger.io/) 有一个非常有用的编辑器和其他资源。

例如，这是一个描述上述两个端点的 YAML 文件。该文件在 GitHub 上可用：https://github.com/PacktPublishing/Python-Architecture-Patterns/blob/main/pen_example.yaml：

```yaml
openapi: 3.0.0
info:
  version: "1.0.0"
  title: "Swagger Pens"
paths:
  /pens:
    post:
      tags:
      - "pens"
      summary: "Add a new pen"
      requestBody:
        description: "Pen object that needs to be added to the store"
        required: true
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/Pen"
      responses:
        "201":
          description: "Created"
        "400":
          description: "Invalid input"
  /pens/{pen_id}:
    get:
      tags:
      - "pens"
      summary: "Retrieve an existing pen"
      parameters:
      - name: "pen_id"
        in: path
        description: "Pen ID"
        required: true
        schema:
          type: integer
          format: int64
      responses:
        "200":
          description: "OK"
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Pen"
        "404":
          description: "Not Found"
components:
  schemas:
    Pen:
      type: "object"
      properties:
        name:
          type: "string"
        color:
          type: "string"
          enum:
            - black
            - blue
            - red
```

在组件部分，Pen 对象被定义，然后在两个端点中使用。你可以看到 POST /pens 和 GET /pens/{pen_id} 这两个端点是如何定义的，并描述了预期的输入和输出，同时考虑了可能产生的不同错误。

Open API 最有趣的方面之一是能够自动生成包含所有信息的文档页面，以帮助任何可能的实现。生成的文档如下所示：

![](../images/2-1.png)

图 2.1：Swagger Pen 文档

如果 YAML 文件正确且完整地描述了你的界面，这将非常有用。在某些情况下，从 YAML 到 API 工作可能是有利的。这首先会生成 YAML 文件，并允许从那里开始双向工作，包括前端方向和后端方向。对于 API 优先的方法，它可能是有意义的。甚至可以用多种语言自动创建客户端和服务器的骨架，例如，Python Flask 或 Spring 中的服务器，以及 Java 或 Angular 中的客户端。

请记住，让实现与定义紧密匹配取决于你。这些骨架仍然需要足够的工作才能使它们正常工作。 Open API 会简化流程，但不会神奇地解决所有集成问题。

每个端点都包含更多信息，甚至可以在同一个文档中进行测试，从而显着帮助想要使用 API 的外部开发人员，如下图所示：

![](../images/2-2.png)

图 2.2：Swagger Pens 扩展文档

鉴于很容易确保服务器可以生成此自动文档，即使设计不是从 Open API YAML 文件开始的，生成它以创建自生成文档也是一个好主意。

## 验证

几乎任何 API 的一个关键部分是能够区分授权访问和未授权访问。能够正确记录用户是至关重要的，从安全的角度来看也是一个令人头疼的问题。

安全很难，所以最好依靠标准来简化操作。

> 正如我们之前所说，这些只是一般提示，但绝不是一套全面的安全实践。本书不关注安全性。请跟上安全问题和解决方案，因为这是一个不断发展的领域。

关于身份验证最重要的安全问题是在生产中始终使用 HTTPS 端点。这允许保护通道免受窃听并使通信保密。请注意，HTTP 网站仅意味着通信是私密的；你可能在和魔鬼说话。但这是允许你的 API 用户向你发送密码和其他敏感信息而不必担心外部用户会收到此信息的最低要求。

> 通常，大多数架构使用 HTTPS 直到请求到达数据中心或安全网络，然后在内部使用 HTTP。这允许检查内部流动的数据，但也可以保护通过 Internet 传输的数据。虽然现在不太重要，但它也提高了效率，因为 HTTPS 中的编码请求需要额外的处理能力。

HTTPS 端点对所有访问都有效，但其他详细信息取决于它们是 HTML 接口还是 RESTful 接口。

### 验证 HTML 接口

在 HTML 网页中，通常认证的流程如下：

1. 登录屏幕会呈现给用户。
2. 用户输入他们的登录名和密码并将它们发送到服务器。
3. 服务器验证密码。如果正确，它会返回一个带有会话 ID 的 cookie。
4. 浏览器接收响应并存储 cookie。
5. 所有新的请求都会发送 cookie。服务器将验证 cookie 并正确识别用户。
6. 用户可以注销，删除 cookie。如果这是明确完成的，则会向服务器发送删除会话 ID 的请求。通常，会话 ID 将有一个自我清理的到期时间。此到期可以在每次访问时自行更新或强制用户不时再次登录。

将 cookie 设置为 Secure、HttpOnly 和 SameSite 很重要。 Secure 确保 cookie 仅发送到 HTTPS 端点，而不是 HTTP 端点。 HttpOnly 使 cookie 无法被 JavaScript 访问，这使得通过恶意代码获取 cookie 变得更加困难。 cookie 将自动发送到设置它的主机。 SameSite 确保仅当源的来源是来自同一主机的页面时才发送 cookie。它可以设置为严格、宽松和无。 Lax 允许你从不同的站点导航到页面，从而发送 cookie，而 Strict 不允许这样做。

> 你可以在 Mozilla SameSite Cookie 页面获取更多信息：https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie/SameSite。

cookie 的错误使用可能是通过 XSS（跨站点脚本）攻击。受感染的脚本会读取该 cookie，然后伪造以用户身份验证的错误请求。

另一种重要的安全问题是跨站点请求伪造（CSRF）。在这种情况下，用户登录外部服务这一事实被利用了一个 URL，该 URL 将在不同的受感染网站中自动执行。

例如，在访问论坛时，会调用来自普通银行的 URL，并以图像的形式呈现。如果用户登录到该银行，将执行该操作。

SameSite 属性大大降低了 CSRF 的风险，但是如果旧浏览器不理解该属性，银行向用户呈现的操作应该呈现一个随机令牌，使用户发送带有 cookie 的经过身份验证的请求和一个有效的令牌。外部页面不知道有效的随机令牌，这使得这种利用变得更加困难。

cookie 包含的会话 ID 可以存储在数据库中，只是一个随机的唯一标识符，也可以是一个丰富的令牌。

随机标识符就是这样，一个随机数，存储了数据库中的相关信息，主要是谁在访问以及会话何时到期。每次访问时，都会向服务器查询此会话 ID 并检索相关信息。在具有许多访问权限的大型部署中，这可能会产生问题，因为它的可扩展性较差。所有工作人员都需要访问存储会话 ID 的数据库，这会造成瓶颈。

一种可能的解决方案是创建丰富的数据令牌。这是通过将所有必需的信息直接添加到 cookie 中来实现的；例如，直接存储用户 ID、有效期等。这避免了数据库访问，但使 cookie 可以伪造，因为所有信息都是公开的。为了解决这个问题，对 cookie 进行了签名。

签名证明数据来自受信任的登录服务器，并且可以由任何其他服务器独立验证。这更具可扩展性并避免了瓶颈。可选地，内容也可以被加密以避免被读取。

该系统的另一个优点是令牌的生成可以独立于通用系统。如果可以独立验证令牌，则登录服务器无需与通用服务器相同。

更重要的是，单个令牌签名者可以为多个服务颁发令牌。这是 SSO（单点登录）的基础：登录到身份验证提供商，然后在多个相关服务中使用相同的帐户。这在 Google、Facebook 或 GitHub 等常用服务中非常常见，以避免必须为某些网页创建特定登录。

这种具有令牌权限的操作模式是 OAuth 授权框架的基础。

### 验证 RESTful 接口

OAuth 已成为验证 API 访问权限的通用标准，尤其是 RESTful API。

> 身份验证和授权是有区别的，本质上，OAuth 是一个授权系统。认证是确定用户是谁，而授权是用户能够做的事情。 OAuth 使用范围的概念来返回用户的能力。

OAuth 的大多数实现，例如 OpenID Connect，还在返回令牌中包含用户信息以对用户进行身份验证，返回用户是谁。

它基于这样一种想法，即有一个授权人可以检查用户的身份并向他们提供一个令牌，该令牌包含允许用户登录的信息。服务将接收此令牌并将登录用户：

![](../images/2-3.png)

图 2.3：认证流程

目前最常见的版本是 OAuth 2.0，它在登录和流程方面提供了灵活性。请记住，OAuth 并不完全是一种协议，而是提供了可以针对特定用例进行调整的某些想法。

> 这意味着你可以通过不同的方式实现 OAuth，而且至关重要的是，不同的授权者会以不同的方式实现它。请在实施集成时仔细验证他们的文档。
>
> 通常，授权者使用基于 OAuth 的 OpenID Connect 协议。

访问 API 的系统是直接访问 API 的系统还是代表用户访问 API 的系统有一个重要区别。后者的一个例子可能是智能手机应用程序来访问 Twitter 等服务，或者需要访问为用户存储在 GitHub 中的数据的服务，例如代码分析工具。应用程序本身不是执行操作而是传输用户操作的应用程序。

此流程称为授权码授予。主要特点是身份验证提供程序将向用户显示登录页面并使用身份验证令牌重定向他们。

例如，这可能是授权代码授权的调用顺序：

```
GET https://myservice.com/login
    Return a page with a form to initiate the login with authorizer.com
Follow the flow in the external authorize until login, with something like.
POST https://authorizer.com/authorize
  grant_type=authorization_code
  redirect_uri=https://myservice.com/redirect
  user=myuser
  password=mypassword
    Return 302 Found to https://myservice.com/redirect?code=XXXXX
GET https://myservice.com/redirect?code=XXXXX
-> Login into the system and set proper cookie, 
   return 302 to https://myservice.com
```

如果访问 API 的系统直接来自最终用户，则可以改用 Client Credentials 授权类型流。在这种情况下，第一次调用将发送 ```client_id```（用户 ID）和 ```client_secret```（密码）以直接检索身份验证令牌。此令牌将在新调用中设置为标头，对请求进行身份验证。

请注意，这会跳过一个步骤，并且更容易自动化：

```
POST /token HTTP/1.1
  grant_type=authorization_code
  &client_id=XXXX
  &client_secret=YYYY
    Returns a JSON body with
    {
  "access_token":"ZZZZ",
  "token_type":"bearer",
  "expires_in":86400,
}
Make new requests setting the header
Authorization: "Bearer ZZZZ"
```

虽然 OAuth 允许你使用外部服务器来检索访问令牌，但这并不是严格要求的。它可以是与其他服务器相同的服务器。这对于最后一个流程很有用，在该流程中，使用 Facebook 或 Google 等外部提供商登录的能力没有那么有用。我们的示例系统将使用客户端凭据流。

### 自编码令牌
从授权服务器返回的令牌可以包含足够的信息，因此不需要与授权方进行外部检查。

> 正如我们所见，在令牌中包含用户信息对于确定用户是谁很重要。如果没有，我们将以能够完成工作的请求结束，但没有代表谁的信息。

为此，令牌通常以 JSON Web 令牌 (JWT) 进行编码。 JWT 是一种将 JSON 对象编码为 URL 安全字符序列的标准。

JWT 具有以下元素：

- 一个标头。这包含有关如何编码令牌的信息。
- 有效载荷。令牌的主体。此对象中的某些字段（称为声明）是标准的，但它也可以分配自定义声明。标准声明不是必需的，可以将发行者 (iss) 或令牌的到期时间等元素描述为 Unix 纪元 (exp)。
- 一个签名。这将验证令牌是由正确的来源生成的。这根据标头中的信息使用不同的算法。

一般来说，JWT 是经过编码的，但不是加密的。标准 JWT 库将对其部分进行解码并验证签名是否正确。

> 你可以在交互工具中测试不同的字段和系统：https://jwt.io/。

例如，要使用 pyjwt (https://pypi.org/project/PyJWT/) 生成令牌，如果之前未安装，则需要使用 pip 安装 PyJWT：

```sh
$ pip install PyJWT
```

然后，在打开 Python 解释器时，要使用带有用户 ID 和 HS256 算法的有效负载创建令牌以使用“秘密”秘密对其进行签名，请使用以下代码：

```python
>>> import jwt
>>> token = jwt.encode({"user_id": "1234"}, "secret", algorithm="HS256")
>>> token
'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoiMTIzNCJ9.vFn0prsLvRu00Kgy6M8s6S2Ddnuvz-FgtQ7nWz6NoC0'
```

然后可以解码 JWT 令牌并提取有效负载。如果 secret 不正确，则会产生错误：

```python
>>> jwt.decode(token,"secret", algorithms=['HS256'])
{'user_id': '1234'}
>>> jwt.decode(token,"badsecret", algorithms=['HS256'])
Traceback (most recent call last):
 …
  jwt.exceptions.InvalidSignatureError: Signature verification failed
```

要使用的算法存储在标头中，但出于安全原因，最好仅使用预期算法验证令牌而不依赖标头。过去，某些 JWT 实现和令牌伪造存在一些安全问题，你可以在此处阅读：https://www.chosenplaintext.ca/2015/03/31/jwt-algorithm-confusion.html。

> 然而，最有趣的算法不是对称算法，如 HS256，其中为编码和解码添加了相同的值，而是像 RSA-256 (RS256) 这样的公钥-私钥。这允许使用私钥对令牌进行编码并使用公钥进行验证。

这种模式很常见，因为公钥可以广泛分发，但只有拥有私钥的适当授权人才能成为令牌的来源。

包含可用于识别用户的有效负载信息允许仅使用有效负载中的信息对请求进行身份验证，一旦验证，如前所述。

## 版本控制 API

接口很少是完全从头开始创建的。它们不断被调整，添加新功能，并修复错误或不一致。为了更好地传达这些更改，创建某种版本控制来传输此信息很有用。

### 为什么要版本控制？

版本控制的主要优点是塑造关于何时包含哪些内容的对话。这可以是错误修复、新功能，甚至是新引入的错误。

如果我们知道当前发布的界面是 ```v1.2.3``` 版本，并且我们即将发布修复 bug X 的版本 ```v1.2.4```，我们可以更轻松地讨论它，并创建发布说明通知用户这一事实。

### 内部与外部版本控制

有两种版本可能会有点混乱。一个是内部版本，这对项目的开发人员来说是有意义的。这通常与软件的版本有关，通常需要一些版本控制的帮助，例如 Git。

这个版本非常详细，可以涵盖非常小的更改，包括小错误修复。它的目的是能够检测到软件之间的最小变化，以便检测错误或引入代码。

另一个是外部版本。外部版本是使用外部服务的人将能够感知的版本。虽然这可以像内部一样详细，但这通常对用户没有太大帮助，并且可能会提供令人困惑的信息。

> 这在很大程度上取决于系统的类型以及他们的预期用户是谁。技术含量高的用户会欣赏额外的细节，但比较随意的用户不会。

例如，内部版本可能会区分两个不同的错误修复，因为这对复制很有用。一个外部交流的版本可以将它们结合在“多个错误修复和改进”中。

另一个很好的例子是当界面发生巨大变化时。例如，网站外观的全新改造可以使用“版本 2 界面”，但这可能发生在多个内部新版本上，由内部测试或由选定组（例如，beta 测试人员）进行测试.最后，当“版本 2 界面”准备就绪时，就可以为所有用户激活了。

描述外部版本的一种方式是称其为“营销版本”。

> 请注意，这里我们避免使用术语“发布版本”，因为它可能会产生误导。此版本仅用于对外交流信息。

这个版本将更多地依赖于营销工作而不是技术实施。

### 语义版本控制

定义版本的常见模式是使用语义版本控制。语义版本控制描述了一种具有三个递增整数的方法，这些整数具有不同的含义，按不兼容的降序排列：

```vX.Y.Z```

X 被称为主要版本。主要版本的任何更改都将意味着向后不兼容的更改。

Y 是次要版本。微小的更改可能会添加新功能，但任何更改都将向后兼容。

Z 是补丁版本。它只会进行小的更改，例如错误修复和安全补丁，但不会更改界面本身。

> 开头的 ```v``` 是可选的，但有助于表明它是一个版本号。

这意味着设计用于 ```v1.2.15``` 的软件将适用于版本 ```v1.2.35``` 和 ```v1.3.5```，但不适用于版本 ```v2.1.3``` 或版本 ```v1.1.4```。它可能适用于 ```v1.2.14``` 版本，但它可能有一些稍后被纠正的错误。

有时，可以添加额外的细节来描述尚未准备好的接口，例如 ```v1.2.3-rc1```（候选发布版）或 ```v1.2.3-dev0```（开发版）。

> 通常，在软件准备好发布之前，主要编号设置为零（例如，```v0.1.3```），使版本 ```v1.0.0``` 成为第一个公开可用的版本。

这种语义版本控制非常容易理解，并提供了有关更改的良好信息。它被广泛使用，但在某些情况下存在一些问题：

- 对于没有明确向后兼容性的系统，严格采用主要版本可能很困难。这就是 Linux 内核停止使用正确语义版本控制的原因，因为它们永远不会更新主要版本，因为每个版本都需要向后兼容。在这种情况下，主要版本可能会被冻结多年，不再是有用的参考。在 Linux 内核中，版本 ```2.6.X``` 发生了这种情况，该版本保持了 8 年，直到 2011 年版本 ```3.0``` 发布，没有任何向后不兼容的更改。
- 语义版本控制需要对接口进行非常严格的定义。如果界面经常随着新功能而变化，就像在线服务通常发生的那样，次要版本会迅速增加，补丁版本几乎没有用处。

对于在线服务，两者结合只会使单个数字有用，这不是很好用。语义版本控制在需要多个 API 版本同时工作的情况下效果更好，例如：

- API 非常稳定并且很少更改，尽管有定期的安全更新。每隔几年，就会有一次重大更新。一个很好的例子是数据库，例如 MySQL。操作系统是另一个例子。
- API 属于一个软件库，可以被多个支持的环境使用；例如，一个 Python 库与 Python 2 的版本 v4 和 Python 3 的 v5 兼容。如果需要，这可以允许多个版本保持活动状态。

如果系统有效地同时运行一个版本，最好不要增加额外的努力来保持适当的语义版本控制，因为就所需的投资而言，这种努力不值得回报。

### 简单的版本控制

可以代替进行严格的语义版本控制，而可以进行简化的版本。这不会带有相同的含义，但它将是一个不断增加的计数器。这将有助于协调团队，尽管它不需要同样的承诺。

这与可以由编译器自动创建的内部版本号的想法相同，该版本号不断增加以区分一个版本并用作参考。但是，普通的内部版本号使用起来可能有点枯燥。

最好使用与语义版本控制类似的结构，因为每个人都可以理解；但不是将它与特定规则一起使用，而是比这更宽松：

- 通常，对于新版本，增加补丁版本。
- 如果补丁版本太高（换句话说，100、10 或其他任意数字），则增加次要版本并将补丁版本设置为零。
- 或者，如果项目有任何特殊的里程碑，由工作人员定义，请提前增加次要编号。
- 对主要版本号执行相同操作。

这将允许数字以一致的方式增加，而不必过多担心意义。

这种结构非常适合在线云服务之类的东西，从本质上讲，它们需要增加计数器，因为它们同时部署了一个版本。在这种情况下，版本最重要的用途是内部使用，不需要严格语义版本控制所需的维护。

## 前端和后端

划分不同服务的常用方法是谈论“前端”和“后端”。它们描述了软件的各个层，其中离最终用户更近的层是前端，而后面的层是后端。

传统上，前端是处理表示层的层，紧挨着用户，后端是数据访问层，服务于业务逻辑。在客户端-服务器架构中，客户端是前端，服务器是后端：

![](../images/2-4.png)

图 2.4：客户端-服务器架构

随着架构变得越来越复杂，这些术语变得有些多义，并且通常根据上下文来理解它们。虽然前端几乎总是被直接理解为用户界面，但后端可以应用于多个层，这意味着下一层支持正在讨论的任何系统。例如，在云应用程序中，Web 应用程序可能使用 MySQL 等数据库作为存储后端，或者使用 Redis 等内存存储作为缓存后端。

前端和后端的一般方法是完全不同的。

前端专注于用户体验，所以最重要的元素是可用性、令人愉悦的设计、响应能力等。其中很多需要关注“最终外观”以及如何使事情易于使用。前端代码在最终用户中执行，因此不同类型硬件之间的兼容性可能很重要。同时，它分配负载，因此从用户界面的角度来看，性能是最重要的。

后端更注重稳定性。在这里，硬件受到严格控制，但负载没有分布，因此在控制使用的总资源方面，性能很重要。修改后端也更容易，因为一旦更改它就会同时为所有用户更改它。但这风险更大，因为这里的问题可能会影响每个用户。这种环境更倾向于关注可靠的工程实践和可复制性。

> 全栈工程师一词通常用于描述能够轻松完成这两种工作的人。虽然这可以在某些方面发挥作用，但实际上很难找到一个同样舒适或倾向于长期同时处理这两个方面的人。
>
> 大多数工程师自然会倾向于某一方面，大多数公司会有不同的团队在这两个方面工作。在某种程度上，每件作品的个性特征都是不同的，前端工作需要更多的设计眼光，而后端用户则对稳定性和可靠性实践感到满意。

一般来说，前端常用的一些技术如下：

- HTML 和相关技术，例如 CSS
- 用于添加交互性的 JavaScript 和库或框架，例如 jQuery 或 React
- 设计工具

后端技术，因为它们受到更直接的控制，可以更加多样化，例如：

- 多种编程语言，可以是 Python、PHP、Ruby 等脚本语言，甚至是使用 Node.js 的 JavaScript，也可以是 Java 或 C# 等编译语言。它们甚至可以混合在一起，用不同的语言制作不同的元素。
- 数据库，可以是 MySQL 或 PostgreSQL 等关系数据库，也可以是 MongoDB、Riak 或 Cassandra 等非关系数据库。
- Web 服务器，例如 Nginx 或 Apache。
- 可扩展性和高可用性工具，例如负载均衡器。
- 基础设施和云技术，例如 AWS 服务。
- 容器相关技术，如 Docker 或 Kubernetes。

前端将利用后端定义的接口以用户友好的方式呈现动作。同一个后端可以有多个前端，一个典型的例子是不同平台的多个智能手机接口，但它们使用相同的 API 与后端通信。

请记住，前端和后端是概念上的划分，但不一定需要将它们划分为不同的进程或存储库。前端和后端一起存在的常见情况是 Web 框架，例如 Ruby on Rails 或 Django，你可以在其中定义前端 HTML 接口，同时定义处理数据访问和业务逻辑的后端控制器。在这种情况下，HTML 代码直接来自执行数据访问的同一进程。这个过程使用模型视图控制器结构分离关注点。

### MVC 结构

模型视图控制器或 MVC 是一种将程序逻辑分成三个不同组件的设计。

> MVC模式在图形用户界面的设计中很早就开始了，并且自 80 年代第一个完整的图形交互界面以来一直用于该领域。在 90 年代，它开始作为一种处理 Web 应用程序的方式被引入。

这种结构非常成功，因为它创建了清晰的概念分离：

- Model管理数据
- Controller 接受来自用户的输入并将其转换为对模型的操作
- View代表用户了解的信息

本质上，模型是系统的核心，因为它处理数据的操作。 Controller 代表输入，View 代表操作的输出。

![](../images/2-5.png)

图 2.5：模型视图控制器模式

MVC结构可以分不同层次考虑，可以看成是分形的。如果多个元素交互，它们可以有自己的 MVC 结构，并且系统的模型部分可以与提供信息的后端对话。

> MVC 模式可以以不同的方式实现。例如，Django 声称它是一个模型视图模板，因为控制器更像是框架本身。但是，这些都是与总体设计不矛盾的小细节。

模型可以说是三者中最重要的元素，因为它是其中的核心部分。它包含数据访问，还包含业务逻辑。丰富的模型组件用作从输入和输出中抽象应用程序逻辑的一种方式。

通常，控制器之间的一些障碍有点模糊。控制器可以处理不同的输入，从而产生对模型的不同调用。同时，输出可以在控制器中调整，然后再传递给视图。虽然始终难以强制执行清晰、严格的界限，但最好记住每个组件的主要目标是什么，以便提供清晰性。

## HTML 接口

虽然 API 的严格定义适用于旨在供其他程序访问的接口，但最好花点时间讨论如何创建成功的人机界面的基础知识。为此，我们将主要讨论 HTML 接口，旨在供最终用户在浏览器中使用。

> 我们将处理的大多数概念适用于其他类型的人机界面，例如 GUI 或移动应用程序。

HTML 技术与 RESTful 技术高度相关，因为它们是在互联网早期并行开发的。通常，它们在现代 Web 应用程序中交织在一起。

### 传统的 HTML 界面

传统 Web 界面的工作方式是通过 HTTP 请求，仅使用 ```GET``` 和 ```POST``` 方法。 ```GET``` 从服务器检索页面，而 ```POST``` 与一些将数据提交到服务器的表单配对。

> 这是一个先决条件，因为浏览器只实现了这些方法。虽然现在大多数现代浏览器都可以在请求中使用所有 HTTP 方法，但允许与旧浏览器兼容仍然是一个常见的要求。

虽然这肯定比所有可用选项更具限制性，但它可以很好地用于简单的网站界面。

例如，一个博客的阅读频率高于写入频率，因此读者使用大量的 ```GET``` 请求来获取信息，并且可能使用一些 ```POST``` 请求来发回一些评论。传统上删除或更改评论的需求很小，尽管它可以与使用 POST 的其他 URL 一起分配。

> 请注意，浏览器会在重试 ```POST``` 请求之前询问你，因为它们不是幂等的。

由于这些限制，HTML 接口与 RESTful 接口的工作方式不同，但它也可以通过考虑抽象和资源方法的设计来改进。

例如，博客的一些常见抽象如下：

- 每个帖子，带有相关评论
- 包含最新帖子的主页
- 可以返回包含特定单词或标签的帖子的搜索页面

这与资源中的接口非常相似，只有“评论”和“帖子”这两个资源会以 RESTful 的方式分开，在同一个概念中加入。

传统 HTML 界面的主要限制是每次更改都需要刷新整个页面。对于像博客这样的简单应用程序，这可以很好地工作，但更复杂的应用程序可能需要更动态的方法。

### 动态页面

为了向浏览器添加交互性，我们可以添加一些 JavaScript 代码，这些代码将执行操作以直接在浏览器表示上更改页面；例如，从下拉选择器中选择界面的颜色。

> 这称为操作文档对象模型 (DOM)，它包含由 HTML 和可能的 CSS 定义的文档表示。 JavaScript 可以访问此表示并通过编辑任何参数甚至添加或删除元素来更改它。

从 JavaScript 中，也可以完成独立的 HTTP 请求，因此我们可以使用它来进行特定的调用以检索可以添加的详细信息以改善用户体验。

例如，对于输入地址的表格，下拉菜单可以选择国家。一旦选择，对服务器的调用将检索适当的区域以合并输入。如果用户选择**美国**，所有州的列表将被检索并在下一个下拉列表中可用。如果用户选择**加拿大**，则将使用地区和省份列表：

![](../images/2-5-1.png)

图 2.5：通过适当的下拉菜单改善用户体验

另一个稍微颠倒界面的示例可能是使用邮政编码自动确定状态。

> 实际上有一个名为 https://zippopotam.us/ 的服务来检索此信息。它可以被调用，不仅可以返回状态，还可以返回更多信息，以 JSON 格式。

这些类型的调用称为异步 JavaScript 和 XML (AJAX)。虽然名称中提到了 XML，但这不是必需的，可以检索任何格式。目前，使用 JSON 甚至纯文本非常普遍。一种可能性是使用 HTML，因此页面的某个区域可以替换为来自服务器的片段：

![](../images/2-6.png)

图 2.6：使用 HTML 替换页面区域

原始 HTML 虽然有些不雅，但可能很有效，因此使用返回 JSON 的 RESTful API 来检索这些小元素的预期数据，然后通过 JavaScript 代码用它修改 DOM 是很常见的。鉴于此 API 的目标不是完全替换 HTML 接口，而是对其进行补充，因此此 RESTful API 很可能是不完整的。仅使用这些 RESTful 调用无法创建完整的体验。

其他应用程序直接创建 API 优先方法并从那里创建浏览器体验。

### 单页应用

单页应用程序背后的想法很简单。让我们打开一个 HTML 页面并动态更改其内容。如果需要任何新数据，将通过特定的（通常是 RESTful）API 进行访问。

这将人机界面（理解为负责向人显示信息的元素）与服务完全分离。该服务专门提供 RESTful API，无需担心数据的表示。

> 这种方法有时被称为 API 优先，因为它设计了一个从 API 到表示的系统，而不是反过来创建它，这是在有机服务中创建它的自然方式。

尽管有一些特定的框架和工具是为这一目标而设计的，例如 React 或 AngularJS，但这种方法存在两个主要挑战：

- 即使借助工具，在单个页面上创建成功的人机界面所需的技术技能也相当高。有效接口的任何重要表示都需要保持大量状态并处理多个调用。这很容易出现影响页面稳定性的错误。浏览器页面的传统方法适用于限制每个步骤范围的独立页面，这更容易处理。

    > 请记住，浏览器承载的界面期望可能难以避免或替换，例如，点击后退按钮。

- 需要提前设计和准备 API 可能会导致项目启动缓慢。即使双方并行发展，也需要更多的规划和前期承诺，这也有其挑战。
    这些问题确保了这种方法通常不适用于从头开始的新应用程序。但是，如果应用程序以另一种用户界面（如智能手机应用程序）启动，它可以利用现有的 REST API 生成复制功能的 HTML 界面。

这种方法的主要优点是将应用程序与用户界面分离。如果应用程序作为一个具有常规 HTML 界面的小项目开始开发，则风险在于任何其他用户界面都倾向于符合 HTML 界面。这很快就会增加大量技术债务并损害 API 的设计，因为所使用的抽象很可能来自现有接口，而不是最合适的接口。

整个 API-first 方法极大地分离了接口，因此创建新接口与现有 API 一样易于使用。对于需要多个界面的应用程序，例如 HTML 界面，以及适用于 iOS 和 Android 的不同智能手机应用程序，这可能是一个很好的解决方案。

在呈现完整界面方面，单页应用程序也可以是相当创新的。这可以创建丰富而复杂的界面，这些界面与可以理解为“网页”的界面不同，例如游戏或交互式应用程序。

### 混合方法

正如我们所见，全押单页应用程序可能非常具有挑战性。在某种程度上，它正在使用浏览器来覆盖它的使用。

这就是为什么通常设计不会走那么远并创建更传统的网络界面的原因。该接口仍可识别为 Web 应用程序，但严重依赖 JavaScript 使用 RESTful 接口获取信息。这可能是从传统 HTML 界面迁移到单页应用程序的自然步骤，但这也可能是一个有意识的决定。

这种方法结合了前两种。一方面，它仍然需要一个 HTML 界面，用于界面的一般做法，有清晰的页面可以导航。另一方面，它创建了一个 RESTful API 来填充大部分信息并使用 JavaScript 来利用这个 API。

> 这种方法类似于动态页面，但有一个重要区别，即旨在创建一个连贯的 API，无需完全针对 HTML 界面定制即可使用。这显着改变了方法。

在实践中，这往往会创建一个不太完整的 RESTful API，因为某些元素可能会直接添加到它的 HTML 部分。但是，与此同时，它允许将元素迭代迁移到 API 中，从某些元素开始，但随着时间的推移添加更多元素。这个阶段非常灵活。

## 设计 API 示例

正如我们在第一章示例概述中所描述的，我们需要为我们将在示例中处理的不同接口设置定义。请记住，该示例是一个微博应用程序，它允许用户编写自己的文本微博，以便其他人可以阅读。

示例中有两个主要接口：

- 允许用户使用浏览器与服务交互的 HTML 界面
- 一个 RESTful 接口，允许创建其他客户端，例如智能手机应用程序

在本章中，我们将描述第二个界面的设计。我们将从描述我们将使用的不同基本定义和资源开始：

- 用户：应用程序用户的表示。它将由用户名和密码定义，以便能够登录。
- 微博：用户发布的最多 255 个字符的小文本。微博可以选择发送给用户。它也有它被创造的时间。
- 收藏：来自用户的微博的显示。
- 关注者：一个用户可以关注另一个用户。
- 时间线：关注用户的微博的有序列表。
- 搜索：允许按用户或微帖子中包含的文本进行搜索。

我们可以以 RESTful 方式将这些元素定义为资源，就像本章前面介绍的方式一样，首先作为 URI 的快速描述：

```
POST   /api/token
DELETE /api/token
GET    /api/user/<username>
GET    /api/user/<username>/collection
POST   /api/user/<username>/collection
GET    /api/user/<username>/collection/<micropost_id>
PUT    /api/user/<username>/collection/<micropost_id>
PATCH  /api/user/<username>/collection/<micropost_id>
DELETE /api/user/<username>/collection/<micropost_id>
GET    /api/user/<username>/timeline
GET    /api/user/<username>/following
POST   /api/user/<username>/following
DELETE /api/user/<username>/following/<username>
GET    /api/user/<username>/followers
GET    /api/search
```

请注意，我们为 ```/token``` 添加了 ```POST``` 和 ```DELETE``` 资源来处理登录和注销。

一旦这个简短的设计完成，我们就可以充实每个端点的定义。

### 端点

我们将按照本章前面介绍的模板更详细地描述所有 API 端点。

#### 登录：

- 描述：使用正确的身份验证凭据，返回有效的访问令牌。令牌需要作为授权标头包含在请求中。
- 资源 URI：```/api/token```
- 方法：```POST```
- 请求正文：
    ```json
        {  
            "grant_type": "authorization_code"
            "client_id": <client id>,
            "client_secret": <client secret>
        }
    ```
- 返回正文：
    ```json
    {
        "access_token": <access token>,
        "token_type":"bearer",
        "expires_in":86400,
    }
    ```
- 错误：
    ```json
    400 Bad Request Incorrect body.
    400 Bad Request Bad credentials.
    ```

#### 登出：

- 说明：使承载令牌无效。如果成功，它将返回 ```204 No Content``` 错误。
- 资源 URI：```/api/token```
- 方法：```DELETE```
- 标头：```Authentication: Bearer: <token>```
- 错误：
    ```
    401 Unauthorized Trying to access this URI without being properly authenticated.
    ```
#### 检索用户：

- 说明：返回用户名资源。
- 资源 URI：```/api/users/<username>```
- 方法：```GET```
- 标头：``Authentication: Bearer: <token>``
- 查询参数：
    ```
    size Page size.
    page Page number.
    ```
- 返回正文：
    ```json
    {
        "username": <username>,
        "collection": /users/<username>/collection,
    }
    ```
- 错误：
    ```
    401 Unauthorized Trying to access this URI without being authenticated.
    404 Not Found Username does not exist.
    ```

#### 检索用户的收藏：

- 说明：以分页形式返回用户所有微博的集合。
- 资源 URI：```/api/users/<username>/collection```
- 方法：```GET```
- 标头：``Authentication: Bearer: <token>``
- 返回正文：
    ```json
    {
        "next": <next page or null>,
        "previous": <previous page or null>,
        "result": [
            {
                "id": <micropost id>,
                "href": <micropost url>,
                "user": <user url>,
                "text": <Micropost text>,
                "timestamp": <timestamp for micropost in ISO 8601>
            },
            ...
        ]
    }
    ```
- 错误：
    ```
    401 Unauthorized Trying to access this URI without being authenticated.
    404 Not Found Username does not exist.
    ```

#### 创建新的微博：

- 描述：创建一个新的微博。
- 资源 URI：```/api/users/<username>/collection```
- 方法：```POST```
- 标头：``Authentication: Bearer: <token>``
- 请求正文：
    ```json
    {
        "text": <Micropost text>,
        "referenced": <optional username of referenced user>
    }
    ```
- 错误：
    ```
    400 Bad Request Incorrect body.
    400 Bad Request Invalid text (for example, more than 255 characters).
    400 Bad Request Referenced user not found.
    401 Unauthorized Trying to access this URI without being authenticated.
    403 Forbidden Trying to create a micropost of a different user to the one logged in.
    ```

#### 检索微博：

- 描述：返回一个微博。
- 资源 URI：```/api/users/<username>/collection/<micropost_id>```
- 方法：```GET```
- 标头：``Authentication: Bearer: <token>``
- 返回正文：
    ```json
    {
        "id": <micropost id>,
        "href": <micropost url>,
        "user": <user url>,
        "text": <Micropost text>,
        "timestamp": <timestamp for micropost in ISO 8601>,
        "referenced": <optional username of referenced user>
    }
    ```
- 错误：
    ```
    401 Unauthorized Trying to access this URI without being authenticated.
    404 Not Found Username does not exist.
    404 Not Found Micropost ID does not exist.
    ```

#### 更新微博：

- 说明：更新微博的文字。
- 资源 URI：```/api/users/<username>/collection/<micropost_id>```
- 方法：```PUT```、```PATCH```
- 标头：```Authentication: Bearer: <token>```
- 请求正文：
    ```json
    {
        "text": <Micropost text>,
        "referenced": <optional username of referenced user>
    }
    ```
- 错误：
    ```
    400 Bad Request Incorrect body.
    400 Bad Request Invalid text (for example, more than 255 characters).
    400 Bad Request Referenced user not found.
    401 Unauthorized Trying to access this URI without being authenticated.
    403 Forbidden Trying to update a micropost of a different user to the one logged in.
    404 Not Found Username does not exist.
    404 Not Found Micropost ID does not exist.
    ```

#### 删除微博：

- 说明：删除一条微博。如果成功，它将返回 ```204 No Content``` 错误。
- 资源 URI：```/api/users/<username>/collection/<micropost_id>```
- 方法：```DELETE```
- 标头：```Authentication: Bearer: <token>```
- 错误：
    ```
    401 Unauthorized Trying to access this URI without being authenticated.
    403 Forbidden Trying to delete a micropost of a different user to the one logged in.
    404 Not Found Username does not exist.
    404 Not Found Micropost ID does not exist.
    ```

#### 检索用户的时间线：

- 说明：以分页形式返回用户时间线中所有微博的集合。微博将按时间戳顺序返回，最旧的先返回。
- 资源 URI：```/api/users/<username>/timeline```
- 方法：```GET```
- 标头：```Authentication: Bearer: <token>``
- 返回正文：
    ```json
    {
        "next": <next page or null>,
        "previous": <previous page or null>,
        "result": [
            {
                "id": <micropost id>,
                "href": <micropost url>,
                "user": <user url>,
                "text": <Micropost text>,
                "timestamp": <timestamp for micropost in ISO 8601>,
                "referenced": <optional username of referenced user>
            },
            ...
        ]
    }
    ```
- 错误：
    ```
    401 Unauthorized Trying to access this URI without being authenticated.
    404 Not Found Username does not exist.
    ```

#### 检索用户关注的用户：

- 说明：返回所选用户关注的所有用户的集合。
- 资源 URI：```/api/users/<username>/following```
- 方法：```GET```
- 标头：```Authentication: Bearer: <token>```
- 返回正文：
    ```json
    {
        "next": <next page or null>,
        "previous": <previous page or null>,
        "result": [
            {
                "username": <username>,
                "collection": /users/<username>/collection,
            },
            ...
        ]
    }
    ```
- 错误：
    ```
    401 Unauthorized Trying to access this URI without being authenticated.
    404 Not Found Username does not exist.

#### 关注用户：

- ：使选定的用户关注不同的用户。
- 资源 URI：```/api/users/<username>/following```
- 方法：```POST```
- 标头：```Authentication: Bearer: <token>```
- 请求正文：
    ```json
    {    
        "username": <username>
    }
    ```
- 错误：
    ```
    400 Bad Request The username to follow is incorrect or does not exist.
    400 Bad Request Bad body.
    401 Unauthorized Trying to access this URI without being authenticated.
    404 Not Found Username does not exist.
    ```

#### 停止关注用户：

- 描述：停止关注用户。如果成功，它将返回 ```204 No Content``` 错误。
- 资源 URI：```/api/users/<username>/following/<username>```
- 方法：```DELETE```
- 标头：```Authentication: Bearer: <token>```
- 错误：
    ```
    401 Unauthorized Trying to access this URI without being authenticated.
    403 Forbidden Trying to stop following a user who is not the authenticated one.
    404 Not Found Username to stop following does not exist.
    ```

#### 检索用户的关注者：

- 描述：以分页形式返回该用户的所有关注者。
- 资源 URI：```/api/users/<username>/followers```
- 方法：```GET```
- 标头：```Authentication: Bearer: <token>```
- 返回正文：
    ```json
    {
        "next": <next page or null>,
        "previous": <previous page or null>,
        "result": [
            {
                "username": <username>,
                "collection": /users/<username>/collection,
            },
            ...
        ]
    }
    ```
- 错误：
    ```
    401 Unauthorized Trying to access this URI without being authenticated.
    404 Not Found Username does not exist.
    ```

#### 搜索微博：

- 描述：以分页形式返回满足搜索查询的微博。
- 资源 URI：```/api/search```
- 方法：```GET```
- 标头：```Authentication: Bearer: <token>```
- 查询参数：
    ```
    username: Optional username to search. Partial matches will be returned.
    text: Mandatory text to search, with a minimum of three characters. Partial matches will be returned.
    ```
- 返回正文：
    ```json
    {
        "next": <next page or null>,
        "previous": <previous page or null>,
        "result": [
            {
                "id": <micropost id>,
                "href": <micropost url>,
                "user": <user url>,
                "text": <Micropost text>,
                "timestamp": <timestamp for micropost in ISO 8601>,
                "referenced": <optional username of referenced user>
            },
        ]
    }
    ```
- 错误：
    ```
    400 Bad Request No mandatory query parameters.
    400 Bad Request Incorrect value in query parameters.
    401 Unauthorized Trying to access this URI without being authenticated.
    ```

### 审查设计和实施

这种呈现和设计新 API 的两步方法使你能够快速查看设计是否有问题。然后，它可以迭代直到固定。下一步是从实现开始，正如我们将在接下来的章节中看到的那样。

## 概括

在本章中，我们描述了 API 设计的基础是如何创建一组有用的抽象，使用户无需关心内部细节即可执行操作。这导致描述如何使用资源和操作定义 API。

API 的这种定义已经发展到涵盖遵循某些属性的 RESTful 接口，这些属性使它们对 Web 服务器设计非常有趣。在设计 RESTful 接口以创建一致且完整的接口（包括 OpenAPI 工具）时，我们描述了一系列有用的标准和技术。我们浏览了身份验证详细信息，因为它是 API 的一个非常重要的元素。

> 请记住，在保护具有外部使用的 API 时应格外小心。我们介绍了一些一般性的想法和常见的策略，但请注意本书并不关注安全性。这是任何 API 设计的一个关键方面，应该谨慎进行。

我们介绍了版本控制背后的想法以及如何创建适合 API 特定用例的适当版本控制模式。我们还介绍了前端和后端之间的差异以及如何对其进行概括。我们还介绍了 MVC 模式，这是一种非常常见的软件结构方式。

我们描述了 HTML 接口的不同选项，以提供 Web 服务中不同接口的完整概述。我们讨论了如何构建 HTML 服务以及如何与其他 API 交互的不同选项。

最后，我们展示了示例的 RESTful 接口设计，同时回顾了一般设计和端点。

设计的另一个关键要素是数据结构。我们将在接下来介绍这一点。