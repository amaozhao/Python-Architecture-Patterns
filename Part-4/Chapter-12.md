# 日志记录

监控和可观察性的基本要素之一是日志。日志允许我们检测正在运行的系统中发生的操作。该信息可用于分析系统的行为，尤其是可能出现的任何错误或错误，使我们能够深入了解实际发生的情况。

但是，正确使用日志看似困难。很容易收集太多或太少的信息，或者记录错误的信息。在本章中，我们将看到收集内容的一些关键要素，以及确保日志得到最佳使用的一般策略。

在本章中，我们将介绍以下主题：

- 日志基础
- 在 Python 中生成日志
- 通过日志检测问题
- 日志策略
- 开发时添加日志
- 日志限制

让我们从日志记录的基本原则开始。

## 日志基础

日志基本上是系统在运行时产生的消息。这些消息是由特定的代码在执行时产生的，使我们能够跟踪代码中发生的动作。

日志可以是完全通用的，例如“函数 X 被调用”，或者可以包含一些执行细节的上下文，例如“函数 X 被调用参数 Y”。

通常，日志以纯文本消息的形式生成。虽然还有其他选择，但纯纯文本非常容易处理，易于阅读，格式灵活，并且可以使用 ```grep``` 等纯文本工具进行搜索。这些工具通常非常快，大多数开发人员和系统管理员都知道如何使用它们。

除了主要的消息文本外，每个日志还包含一些关于哪个系统生成日志、创建日志的时间等的元数据。如果日志为文本格式，则通常附在行首。

> 标准且一致的日志格式可帮助你搜索、过滤和排序消息。确保你在不同系统中使用一致的格式。

另一个重要的元数据值是日志的严重性。这使我们能够根据它们的相对重要性对不同的日志进行分类。标准严重性级别按重要性从低到高依次为```DEBUG```, ```INFO```, ```WARNING```, 和 ```ERROR```。

> ```CRITICAL``` 级别使用较少，但它用于显示灾难性错误。

重要的是使用适当的严重性对日志进行分类并过滤掉不重要的消息以专注于更重要的消息。每个日志记录工具都可以配置为仅生成一个或多个严重级别的日志。

> 可以添加自定义日志级别而不是预定义的日志级别。这通常是一个坏主意，在大多数情况下应该避免，因为所有工具和工程师都很好地理解了日志级别。我们将在本章后面描述如何定义每个级别的策略以充分利用每个级别。

在服务请求的系统中，无论是作为请求-响应还是异步，大多数日志都将作为处理请求的一部分生成，这将产生几个指示请求正在做什么的日志。由于通常会同时处理多个请求，因此将混合生成日志。例如，考虑以下日志：

```
Sept 16 20:42:04.130 10.1.0.34 INFO web: REQUEST GET /login
Sept 16 20:42:04.170 10.1.0.37 INFO api: REQUEST GET /api/login
Sept 16 20:42:04.250 10.1.0.37 INFO api: REQUEST TIME 80 ms
Sept 16 20:42:04.270 10.1.0.37 INFO api: REQUEST STATUS 200
Sept 16 20:42:04.360 10.1.0.34 INFO web: REQUEST TIME 230 ms
Sept 16 20:42:04.370 10.1.0.34 INFO web: REQUEST STATUS 200
```

上述日志显示了两种不同的服务，由不同的 IP 地址（10.1.0.34 和 10.1.0.37）和两种不同的服务类型（web 和 api）表示。尽管这足以分隔请求，但最好创建一个请求 ID 以便能够按以下方式对请求进行分组：

```
Sept 16 20:42:04.130 10.1.0.34 INFO web: [4246953f8] REQUEST GET /login
Sept 16 20:42:04.170 10.1.0.37 INFO api: [fea9f04f3] REQUEST GET /api/login
Sept 16 20:42:04.250 10.1.0.37 INFO api: [fea9f04f3] REQUEST TIME 80 ms
Sept 16 20:42:04.270 10.1.0.37 INFO api: [fea9f04f3] REQUEST STATUS 200
Sept 16 20:42:04.360 10.1.0.34 INFO web: [4246953f8] REQUEST TIME 230 ms
Sept 16 20:42:04.370 10.1.0.34 INFO web: [4246953f8] REQUEST STATUS 200
```

> 在微服务环境中，请求将从一个服务流向另一个服务，因此创建一个跨服务共享的请求 ID 是一个好主意，这样可以理解完整的跨服务流。为此，需要由第一个服务创建请求 ID，然后将其传输到下一个服务，通常作为 HTTP 请求中的标头。

正如我们在第 5 章“十二因素应用程序方法论”中看到的，在十二因素应用程序方法论中，日志应该被视为事件流。这意味着应用程序本身不应该关心日志的存储和处理。相反，应将日志定向到标准输出。从那里，在开发应用程序时，开发人员可以在应用程序运行时提取信息。

在生产环境中，应捕获```stdout```，以便其他工具可以使用它，然后路由，将任何不同的源附加到单个流中，然后存储或索引以供以后查阅。这些工具应该在生产环境中配置，而不是在应用程序本身中。

这种重新路由的可能工具包括 Fluentd (https://github.com/fluent/fluentd) 之类的替代方法，甚至是直接到 logger Linux 命令以创建系统日志然后将这些日志发送到配置的 rsyslog (https ://www.rsyslog.com/) 可以转发和聚合它们的服务器。

无论我们如何收集日志，典型的系统都会产生大量日志，并且需要将它们存储在某个地方。虽然每个单独的日志都很小，但聚合数千个日志会占用大量空间。任何日志系统都应该配置为有一个关于它应该接受多少数据以避免无限增长的策略。一般来说，基于时间的保留策略（例如保留过去 15 天的日志）是最好的方法，因为它很容易理解。在你需要能够查看多远的过去和系统使用的空间量之间找到平衡很重要。

启用任何新的日志服务（无论是本地的还是基于云的）时，请务必检查保留策略，以确保它与你定义的保留期兼容。你将无法分析时间窗口之前发生的任何事情。仔细检查日志创建速率是否符合预期，并且空间消耗没有使你可以收集日志的有效时间窗口变小。你不想在跟踪错误时发现意外超出配额。

生成日志条目很容易，我们将在下一节中看到，用 Python 生成日志。

## 在 Python 中生成日志

Python 包含一个标准模块来生成日志。该模块易于使用，配置非常灵活，但如果你不了解它的操作方式，可能会造成混淆。

创建日志的基本程序如下所示。这在 GitHub 上作为 basic_logging.py 可用，网址为 https://github.com/PacktPublishing/Python-Architecture-Patterns/tree/main/chapter_12_logging：

```python
import logging
# Generate two logs with different severity levels
logging.warning('This is a warning message')
logging.info('This is an info message')
```

```.warning``` 和 ```.info``` 方法创建带有相应严重性消息的日志。消息是一个文本字符串。

执行时显示如下：

```sh
$ python3 basic_logging.py
WARNING:root:This is a warning message
```

默认情况下，日志被路由到```stdout```，这是我们想要的，但它被配置为不显示 ```INFO``` 日志。日志的格式也是默认的，不包括时间戳。

要添加所有这些信息，我们需要了解 Python 中用于登录的三个基本元素：

- 格式化程序，描述完整日志的显示方式，附加时间戳或严重性等元数据。
- 一个处理程序，它决定如何传播日志。如上所述，它通过格式化程序设置日志的格式。
- 一个记录器，它产生日志。它有一个或多个描述日志如何传播的处理程序。

有了这些信息，我们可以配置日志来指定我们想要的所有细节：

```python
import sys
import logging
# Define the format
FORMAT = '%(asctime)s.%(msecs)dZ:APP:%(name)s:%(levelname)s:%(message)s'
formatter = logging.Formatter(FORMAT, datefmt="%Y-%m-%dT%H:%M:%S")
# Create a handler that sends the logs to stdout
handler = logging.StreamHandler(stream=sys.stdout)
handler.setFormatter(formatter)
# Create a logger with name 'mylogger', adding the handler and setting
# the level to INFO
logger = logging.getLogger('mylogger')
logger.addHandler(handler)
logger.setLevel(logging.INFO)
# Generate three logs
logger.warning('This is a warning message')
logger.info('This is an info message')
logger.debug('This is a debug message, not to be displayed')
```

我们按照之前看到的相同顺序定义这三个元素。首先是格式化程序，然后是设置格式化程序的处理程序，最后是添加处理程序的记录器。

格式化程序具有以下格式：

```python
FORMAT = '%(asctime)s.%(msecs)dZ:APP:%(name)s:%(levelname)s:%(message)s'
formatter = logging.Formatter(FORMAT, datefmt="%Y-%m-%dT%H:%M:%S")
```

```FORMAT``` 由 Python ```%``` 格式组成，这是一种描述字符串的旧方式。大多数元素被描述为 ```%(name)s```，其中最后的 ```s``` 字符表示字符串格式。以下是每个元素的说明：

- ```asctime``` 以人类可读的格式设置时间戳。我们在 datefmt 参数中描述它以遵循 ISO 8601 格式。我们还添加下一个毫秒和一个 Z 以获得完整 ISO 8601 形式的时间戳。 %(msecs)d 以 d 结尾表示我们将值打印为整数。这是为了将值限制为毫秒，并且不显示任何额外的分辨率，它可以作为小数值使用。
- ```name``` 是记录器的名称，我们稍后会描述。我们还添加了 APP 以区分不同的应用程序。
- ```levelname``` 是日志的严重性，例如 INFO、WARNING 或 ERROR。
- ```message```最后日志消息。

一旦我们定义了格式化程序，我们就可以移动到处理程序：

```python
handler = logging.StreamHandler(stream=sys.stdout)
handler.setFormatter(formatter)
```

处理程序是一个 ```StreamHandler```，我们将流的目标设置为 ```sys.stdout```，它是 Python 定义的指向 ```stdout``` 的变量。

> 有更多可用的处理程序，例如 FileHandler 将日志发送到文件，SysLogHandler 将日志发送到 syslog 目标，甚至更高级的情况，例如 TimeRotatingFileHandler，它根据时间轮换日志，这意味着它存储最后定义的时间，以及归档旧版本。你可以在 https://docs.python.org/3/howto/logging.html#useful-handlers 的文档中查看所有可用处理程序的更多信息。

一旦定义了```handler```，我们就可以创建```logger```：

```python
logger = logging.getLogger('mylogger')
logger.addHandler(handler)
logger.setLevel(logging.INFO)
```

首先要做的是为记录器创建一个名称，在这里我们定义为 mylogger。这允许我们将应用程序的日志划分为多个小节。我们使用 ```.addHandler``` 附加处理程序。

最后，我们使用 ```.setLevel``` 方法将要记录的级别定义为 ```INFO```。这将显示所有级别为 ```INFO``` 及更高级别的日志，而那些级别较低的日志不会显示。

如果我们运行该文件，我们会看到整个配置组合在一起：

```python
$ python3 configured_logging.py
2021-09-18T23:15:24.563Z:APP:mylogger:WARNING:This is a warning message
2021-09-18T23:15:24.563Z:APP:mylogger:INFO:This is an info message
```

我们可以看到：

- 时间以 ISO 8601 格式定义为 ```2021-09-18T23:15:24.563Z```。这是 ```asctime``` 和 ```msec``` 参数的组合。
- ```APP``` 和 ```mylogger``` 参数允许我们按应用程序和子模块进行过滤。
- 显示严重性。请注意，没有显示一条 ```DEBUG``` 消息，因为配置的最低级别是 ```INFO```。

Python 中的日志记录模块能够进行高级别的配置。查看官方文档以获取更多信息，网址为 https://docs.python.org/3/library/logging.html。

## 通过日志检测问题

对于正在运行的系统中的任何问题，都可能发生两种错误：预期错误和意外错误。在本节中，我们将看到它们在日志方面的差异以及我们如何处理它们。

### 检测预期错误

预期错误是通过在代码中创建错误日志明确检测到的错误。例如，当访问的 URL 返回不同于 ```200 OK``` 的状态码时，以下代码会生成 ```ERROR``` 日志：

```python
import logging
import requests
URL = 'https://httpbin.org/status/500'
response = requests.get(URL)
status_code = response.status_code
if status_code != 200:
    logging.error(f'Error accessing {URL} status code {status_code}')
```

此代码在执行时会触发错误日志：

```sh
$ python3 expected_error.py
ERROR:root:Error accessing https://httpbin.org/status/500 status code 500
```

这是访问外部 URL 并验证它是否已被正确访问的常见模式。生成日志的块可以执行一些补救或重试等等。

> 在这里，我们使用 ```https://httpbin.org``` 服务，这是一个简单的 HTTP 请求和响应服务，可用于测试代码。特别是 ```https://httpbin.org/status/<code>``` 端点返回指定的状态码，容易产生错误。

这是预期错误的示例。我们提前计划了一些我们不想发生的事情，但我们知道它有可能发生。通过提前计划，代码已准备好处理错误并充分捕获它。

在这种情况下，我们可以足够清楚地描述情况，并提供上下文来了解正在发生的事情。问题是显而易见的，即使解决方案可能不是。

这些类型的错误相对容易处理，因为它们描述了可预见的问题。

例如，站点可能不可用，可能存在身份验证问题，或者基本 URL 配置错误。

> 请记住，在某些情况下，代码可以处理某种情况而不会失败，但它仍然被视为错误。例如，你可能想检测是否有人仍在使用旧的身份验证系统。这种在检测到不推荐的操作时添加错误或警告日志的方法可以让你采取措施来纠正这种情况。

此类错误的其他示例包括与数据库的连接以及以不推荐的格式存储的数据。

### 捕获意外错误

但预期的错误并不是唯一可能发生的错误。不幸的是，任何正在运行的系统都会以各种意想不到的行为让你大吃一惊，这些行为会以创造性的方式破坏代码。 Python 中的意外错误通常是由代码中的某个点引发的异常产生的，而该异常不会被捕获。

例如，假设在对某些代码进行小改动时，我们引入了一个错字：

```python
import logging
import requests
URL = 'https://httpbin.org/status/500'
logging.info(f'GET {URL}')
response = requests.ge(URL)
status_code = response.status_code
if status_code != 200:
    logging.error(f'Error accessing {URL} status code {status_code}')
```

请注意，在第 8 行中，我们引入了一个错字：

```python
response = requests.ge(URL)
```

正确的 ```.get``` 调用已替换为 ```.ge```。当我们运行它时，它会产生以下错误：

```python
$ python3 unexpected_error.py
Traceback (most recent call last):
  File "./unexpected_error.py", line 8, in <module>
    response = requests.ge(URL)
AttributeError: module 'requests' has no attribute 'ge'
```

默认情况下，在 Python 中，它将在```stdout```中显示错误和堆栈跟踪。当代码作为 Web 服务器的一部分执行时，有时足以将这些消息作为 ```ERROR``` 日志发送，具体取决于配置的设置方式。

> 任何 Web 服务器都会捕获这些消息并将其正确路由到日志并生成正确的 500 状态代码，表明出现了意外错误。服务器仍可用于下一个请求。

如果你需要创建一个需要无限运行并防止任何意外错误的脚本，请务必使用 try..except 块，因为它是通用的，因此将捕获和处理任何可能的异常。

> 任何使用特定 except 块正确捕获的 Python 异常都可以被视为预期错误。其中一些可能需要生成 ERROR 消息，但其他可能不需要此类信息即可捕获和处理。

例如，让我们调整代码以每隔几秒发出一次请求。该代码可在 GitHub 上的 https://github.com/PacktPublishing/Python-Architecture-Patterns/tree/main/chapter_12_logging 获得：

```python
import logging
import requests
from time import sleep
logger = logging.getLogger()
logger.setLevel(logging.INFO)
while True:
    
    try:
        sleep(3)
        logging.info('--- New request ---')
    
        URL = 'https://httpbin.org/status/500'
        logging.info(f'GET {URL}')
        response = requests.ge(URL)
        scode = response.status_code
        if scode != 200:
            logger.error(f'Error accessing {URL} status code {scode}')
    except Exception as err:
        logger.exception(f'ERROR {err}')
```

关键元素是以下无限循环：

```python
while True:
    try:
        code
    except Exception as err:
        logger.exception(f'ERROR {err}')
```

```try..except``` 块在循环内部，所以即使有错误，循环也不会中断。如果有任何错误，除了```except Exception```，无论异常是什么。

> 这有时被称为 ```Pokemon``` 异常处理，如“Gotta catch 'em all”。这应该仅限于一种“最后的安全网”。一般来说，对要捕获的异常不精确是个坏主意，因为你可以通过错误处理来隐藏错误。错误永远不应该悄无声息地过去。

为了确保不仅记录了错误，而且记录了完整的堆栈跟踪，我们使用 .exception 而不是 .error 来记录它。这会将信息扩展到单个文本消息，同时以 ERROR 严重性记录它。

当我们运行命令时，我们会得到这些日志。请务必按 Ctrl + C 停止它：

```sh
$ python3 protected_errors.py
INFO:root:--- New request ---
INFO:root:GET https://httpbin.org/status/500
ERROR:root:ERROR module 'requests' has no attribute 'ge'
Traceback (most recent call last):
  File "./protected_errors.py", line 18, in <module>
    response = requests.ge(URL)
AttributeError: module 'requests' has no attribute 'ge'
INFO:root:--- New request ---
INFO:root:GET https://httpbin.org/status/500
ERROR:root:ERROR module 'requests' has no attribute 'ge'
Traceback (most recent call last):
  File "./protected_errors.py", line 18, in <module>
    response = requests.ge(URL)
AttributeError: module 'requests' has no attribute 'ge'
^C
...
KeyboardInterrupt
```

如你所见，日志包括 Traceback，它允许我们通过添加有关异常产生位置的信息来检测特定问题。

任何意外错误都应记录为 ERROR。理想情况下，还应该分析它们并更改代码以修复它们或至少将它们转换为预期的错误。有时由于其他紧迫问题或问题发生率较低，这是不可行的，但应实施一些策略以确保在处理错误时保持一致性。

> 处理意外错误的好工具是 Sentry (https://sentry.io/)。该工具为许多常见平台上的每个错误创建一个触发器，包括 Python Django、Ruby on Rails、Node、JavaScript、C#、iOS 和 Android。它汇总了检测到的错误，并允许我们更有策略地处理它们，这在仅访问日志时有时会很困难。

有时，意外错误会向自己提供有关问题所在的足够信息，这可能与网络问题或数据库问题等外部问题有关。解决方案可能位于服务本身的范围之外。

## 日志策略

处理日志时的一个常见问题是为每个单独的服务确定适当的严重性。此消息是```WARNING```还是```ERROR```？是否应将此语句添加为 ```INFO``` 消息？

大多数日志严重性描述都有定义，例如程序显示潜在的有害情况或应用程序突出显示请求的进度。这些是模糊的定义，很难在现实生活中采取行动。与其使用这些模糊的定义，不如尝试定义与遇到问题时应采取的任何后续行动相关的每个级别。这有助于向开发人员阐明在找到给定错误日志时应该做什么。例如：“每次发生这种情况时，我都希望得到通知吗？”

下表显示了不同严重级别的一些示例以及可以采取的措施：

| 日志级别 | 采取的行动                                   | Comments                                                     |
| -------- | -------------------------------------------- | ------------------------------------------------------------ |
| DEBUG    | None.                                        | 未追踪。仅在开发时有用。                                     |
| INFO     | None.                                        | INFO 日志显示有关应用程序中操作流程的一般信息，以帮助跟踪系统。 |
| WARNING  | 跟踪日志数量。提高水平的警报。               | WARNING 日志跟踪自动修复的错误，例如重试连接到外部服务或数据库中可修复的格式错误。突然增加可能需要调查。 |
| ERROR    | 跟踪日志数量。提高水平的警报。查看所有错误。 | ERROR 日志跟踪无法恢复的错误。突然增加可能需要立即采取行动。所有这些都应定期审查以修复常见事件并减轻它们，也许将它们移动到警告级别。 |
| CRITICAL | 即时响应。                                   | CRITICAL 日志指示应用程序中的灾难性故障。单个表示系统根本不工作并且无法恢复。 |

这对如何应对设定了明确的期望。请注意，这是一个示例，你可能需要进行调整和调整以使其适应特定组织的需求。

不同严重性的层次结构非常清晰，在我们的示例中，可以接受会生成一定数量的 ERROR 日志。为了开发团队的理智，并非所有事情都需要立即修复，而是应该执行一定的顺序和优先级。

> 在生产环境中，ERROR 日志通常会被分类为“我们注定要失败”到“meh”。开发团队应积极修复“meh”日志或停止记录问题以消除监控工具的噪音。如果不值得检查，这可能包括降低日志级别。你需要尽可能少的 ERROR 日志，因此所有这些日志都是有意义的。
>
> 请记住，错误日志将包含意外错误，这些错误通常需要修复才能完全解决问题，或者显式捕获它并在不重要的情况下降低其严重性。
>
> 随着应用程序的增长，这种跟进肯定是一个挑战，因为 ERROR 日志的数量将显着增加。它需要时间花在主动维护上。如果不认真对待这一点，并且经常将其用于其他任务，它将在中期内损害应用程序的可靠性。

```WARNING``` 日志表明某些事情可能不像预期的那样顺利，但事情是在控制之中的，除非这种日志的数量突然增加。 ```INFO``` 日志只是在出现问题时提供上下文，否则可以忽略。

> 一个常见的错误是在输入参数不正确的操作中生成 ERROR 日志，例如在 Web 请求中返回 400 BAD REQUEST 状态代码时。一些开发人员会争辩说，客户发送格式错误的请求是一个错误。但是，如果请求被正确检测并返回，则开发团队不应该做任何事情。一切照旧，唯一的行动可能是向请求者返回有意义的消息，以便他们可以修复他们的请求。
>
> 如果此行为在某些关键请求中持续存在，例如重复发送错误密码，则可以创建警告日志。当应用程序按预期运行时，创建 ERROR 日志是没有意义的。
>
> 根据经验，在 Web 应用程序中，仅当状态代码是 50X 变体之一（如 500、502 和 503）时才应创建 ERROR 日志。请记住，40X 错误意味着发件人有问题，而 50X 意味着应用程序有问题，你的团队有责任修复它。

通过整个团队对日志级别的通用和共享定义，所有工程师将对错误严重性有共同的理解，这将有助于制定有意义的行动来改进代码。

留出时间来调整和调整任何定义。你还可能必须处理在定义之前创建的日志，这可能需要工作。遗留系统中最大的挑战之一是创建一个适当的日志系统来对问题进行分类，因为它们可能会非常嘈杂，从而难以区分真正的问题和烦恼，甚至是非问题。

## 开发时添加日志

任何测试运行程序都会在运行测试时捕获日志并将其显示为跟踪的一部分。

> 我们在第 10 章，测试和 TDD 中介绍过的 pytest 将显示日志作为测试失败结果的一部分。

这是在功能仍处于开发阶段时检查是否正在生成预期日志的好机会，特别是如果它是在 TDD 流程中完成的，其中失败的测试和错误作为流程的一部分定期产生，正如我们在第 10 章，测试和 TDD。任何检查错误的测试还应添加相应的日志，并在开发功能时检查它们是否正在生成。

> 你可以使用 pytest-catchlog (https://pypi.org/project/pytest-catchlog/) 之类的工具显式向测试添加检查以验证是否正在生成日志。
>
> 但是，通常情况下，我们只是稍加注意，并在使用 TDD 实践的同时将检查实践作为测试失败的初始检查的一部分。但是，请确保开发人员了解为什么在开发过程中记录日志有助于养成习惯。

在开发过程中，```DEBUG```日志可用于添加有关代码流的额外信息，这对于生产来说是过多的。在开发中，这些额外的信息可以帮助填补```INFO```日志之间的空白，帮助开发者巩固添加日志的习惯。如果在测试期间发现它对跟踪生产中的问题很有用，则可以将 ```DEBUG``` 日志提升为 ```INFO```。

此外，在特殊情况下，可以在受控情况下在生产中启用 ```DEBUG``` 日志，以跟踪某些难以理解的问题。请注意，这对生成的日志数量有很大影响，这可能会导致存储问题。在这里要非常小心。

> 请注意 ```INFO``` 和更高严重性日志中显示的消息。在显示的信息方面，请避免使用敏感数据，例如密码、密钥、信用卡号和个人信息。
>
> 密切关注生产中的任何大小限制以及生成日志的速度。在生成新功能、请求数量增加或系统中工作人员数量增加的情况下，系统可能会遇到日志爆炸。这三种情况都是在系统增长时产生的。

仔细检查日志是否被正确捕获并在不同环境中可用总是一个好主意。确保正确捕获日志的所有配置可能需要一些时间，因此最好提前执行此操作。这涉及捕获生产中的意外错误和其他日志，并检查所有管道是否正确完成。另一种方法是在遇到真正的问题后才发现它不能正常工作。

## 日志限制

日志对于了解正在运行的系统中发生的事情非常有用，但它们具有某些重要的限制：

- 日志与它们的消息一样好。一个好的、描述性的消息对于使日志有用是至关重要的。以批判的眼光查看日志消息，并在需要时进行更正，这对于节省生产问题的宝贵时间非常重要。

- 有适当数量的日志。太多的日志可能会混淆流程，而太少的日志可能无法包含足够的信息来让我们理解问题。大量日志也会产生存储问题。

- 日志应该作为问题上下文的指示，但可能不会查明它。试图生成完全解释错误的特定日志将是一项不可能完成的任务。相反，专注于显示操作的一般流程和周围环境，以便可以在本地复制和调试它。例如，对于请求，请确保记录请求及其参数，以便可以复制情况。

- 日志允许我们跟踪单个实例的执行。当使用请求 ID 或类似名称将日志分组在一起时，可以按执行对日志进行分组，从而使我们能够跟踪请求或任务的流程。但是，日志不直接显示汇总信息。日志回答了“这个任务发生了什么？”这个问题，而不是“系统中发生了什么？”对于这类信息，最好使用指标。

    > 有一些工具可用于根据日志创建指标。我们将在第 13 章“度量”中详细讨论度量。

- 日志只能追溯工作。当检测到任务中的问题时，日志只能显示预先准备好的信息。这就是为什么批判性地分析和提炼信息、删除无用的日志并添加具有相关上下文信息的其他日志以帮助重现问题的重要原因。

日志是一个很棒的工具，但需要对其进行维护，以确保它们可用于检测错误和问题，并使我们能够尽可能有效地采取行动。

## 概括

在本章中，我们首先介绍了日志的基本元素。我们定义了日志如何包含消息以及时间戳等一些元数据，并考虑了不同的严重性级别。我们还描述了定义请求 ID 以对与同一任务相关的日志进行分组的需要。我们还讨论了如何在十二因素应用程序方法中将日志发送到标准输出，以将日志生成与处理过程分离并将它们路由到正确的目的地，以允许收集系统中的所有日志。

然后，我们展示了如何使用标准日志记录模块在 Python 中生成日志，描述了记录器、处理程序和格式化程序的三个关键元素。接下来，我们展示了系统中可能产生的两种不同错误：预期的，理解为尽可能预见并处理的错误；和意想不到的，意思是那些没有预见到并且发生在我们控制之外的事情。然后，我们针对这些进行了不同的策略和案例。

我们描述了不同的严重性以及如何为检测到特定严重性的日志时应采取的措施生成策略，而不是根据“它们的严重程度”对日志进行分类，这最终会产生模糊的指导方针，而不是非常有用。

我们讨论了几个习惯，通过将它们包含在 TDD 工作流的开发中来改进有用性日志。这允许开发人员在编写测试和产生错误时考虑日志中显示的信息，这是确保生成的日志正常工作的绝佳机会。

最后，我们讨论了日志的局限性以及如何处理它们。

在下一章中，我们将研究如何使用聚合信息通过使用度量来找出系统的一般状态。