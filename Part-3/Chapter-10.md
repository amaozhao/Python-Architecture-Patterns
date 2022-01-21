# 测试和TDD

无论开发人员多么优秀，他们编写的代码并不总是能正确执行。这是不可避免的，因为没有开发人员是完美的。但这也是因为预期的结果有时并不是沉浸在编码中的人会想到的结果。

设计很少按预期进行，并且在实施过程中总是会反复讨论，直到对其进行改进并使其正确为止。

> 每个人都有一个计划，直到他们被打到嘴里。 ——迈克·泰森

编写软件是出了名的困难，因为它具有极高的可塑性，但与此同时，我们可以使用软件来仔细检查代码是否在做它应该做的事情。

> 请注意，与任何其他代码一样，测试也可能有错误。

编写测试可以让你在代码新鲜时检测问题，并以一些理智的怀疑来验证预期结果是否是实际结果。我们将在本章中看到如何轻松编写测试，以及编写不同测试以捕获不同类型问题的不同策略。

我们将描述如何在 TDD 下工作，这是一种通过首先定义测试来工作的方法，以确保验证尽可能独立于实际代码实现。

我们还将展示如何使用常见的单元测试框架、标准的 ```unittest``` 模块以及更高级和更强大的 ```pytest``` 在 Python 中创建测试。

请注意，本章比其他章节要长一些，主要是因为需要展示示例代码。

在本章中，我们将介绍以下主题：

- 测试代码
- 不同级别的测试
- 测试理念
- 测试驱动开发
- Python中的单元测试简介
- 测试外部依赖
- 高级pytest

让我们从一些关于测试的基本概念开始。

## 测试代码

讨论代码测试时的第一个问题很简单：测试代码到底是什么意思？

虽然对此有多种答案，但从最广泛的意义上来说，答案可能是“在应用程序到达最终客户之前检查应用程序是否正常工作的任何程序”。从这个意义上说，任何正式或非正式的测试程序都将满足该定义。

> 最轻松的方法（有时出现在只有一两个开发人员的小型应用程序中）是不创建特定测试，而是进行非正式的“完整应用程序运行”，检查新实现的功能是否按预期工作。
>
> 这种方法可能适用于小型、简单的应用程序，但主要问题是确保旧功能保持稳定。

但是，对于足够大且足够复杂的高质量软件，我们需要更加小心地进行测试。因此，让我们尝试对测试提出一个更精确的定义：测试是任何记录在案的过程，最好是自动化的，它从已知的设置中检查应用程序的不同元素是否正常工作，然后才能到达最终客户。

如果我们检查与先前定义的差异，则有几个关键词。让我们检查它们中的每一个以查看不同的详细信息：

- 记录：与以前的版本相比，目标应该是记录测试。这使你可以在必要时精确地复制它们，并允许你比较它们以发现盲点。
    有多种方法可以记录测试，或者通过指定要运行的步骤列表和预期结果，或者通过创建运行测试的代码。主要思想是一个测试可以被分析，由不同的人运行多次，必要时可以改变，并且有一个清晰的设计和结果。

- 最好是自动化的：测试应该能够自动运行，尽可能少的人工干预。这使你可以触发持续集成技术来反复运行许多测试，从而创建一个能够尽早捕获意外错误的“安全网”。我们说“最好”是因为有些测试完全自动化是不可能的或非常昂贵的。无论如何，目标应该是使绝大多数测试自动化，让计算机完成繁重的工作并节省宝贵的人力时间。还有多种软件工具可让你运行测试，这会有所帮助。

- 从已知启动：为了能够单独运行测试，我们需要在运行测试之前知道系统的状态应该是什么。这确保了测试的结果不会产生可能干扰下一次测试的特定状态。在测试之前和之后，可能需要进行某些清理。
    与不担心初始或结束状态相比，这可以使批量运行测试变慢，但会为避免问题打下坚实的基础。

    > 作为一般规则，尤其是在自动化测试中，执行测试的顺序应该是无关紧要的，以避免交叉污染。这说起来容易做起来难，在某些情况下，测试的顺序会产生问题。例如，测试 A 创建一个测试 B 读取的条目。如果测试 B 单独运行，它将失败，因为它期望 A 创建的条目。这些情况应该修复，因为它们会使调试变得非常复杂。此外，能够独立运行测试允许它们被并行化。

- 应用程序的不同元素：大多数测试不应该针对整个应用程序，而应该针对它的较小部分。稍后我们将更多地讨论不同级别的测试，但测试应该具体说明它们要测试的内容并涵盖不同的元素，因为覆盖更多领域的测试成本会更高。

测试的一个关键要素是获得良好的投资回报。设计和运行测试需要时间，而这些时间需要好好利用。任何测试都需要维护，这应该是值得的。在整章中，我们将评论测试的这个重要方面。

> 有一种重要的测试我们没有包含在这个定义中，它被称为探索性测试。这些测试通常由 QA 工程师运行，他们在没有明确的先入为主的情况下使用最终应用程序，但会尝试先发制人地发现问题。如果应用程序具有面向客户的 UI，则这种测试风格对于检测设计阶段未检测到的不一致和问题非常有用。
>
> 例如，一个优秀的 QA 工程师会说 X 页面上的按钮颜色与 Y 页面上的按钮颜色不同，或者该按钮不够明显，无法执行某项操作，或者无法执行某项操作。某些操作有一个先决条件，新界面不明显或不可能。任何用户体验 (UX) 检查都可能属于这一类。
>
> 就其本质而言，这种测试不能“设计”或“记录”，因为它最终归结为解释和理解应用程序是否正确的良好眼光。一旦检测到问题，就可以记录下来以避免问题。
>
> 虽然这肯定有用且值得推荐，但这种测试风格更像是一门艺术而不是工程实践，我们不会详细讨论它。

这个一般定义有助于开始讨论，但我们可以更具体地了解在每次测试期间由多少系统正在测试中定义的不同测试。

## 不同级别的测试

正如我们之前所描述的，测试应该涵盖系统的不同元素。这意味着测试可以解决系统（或整个系统）的一小部分或大部分，试图减少其作用范围。

在测试系统的一小部分时，我们降低了测试的复杂性和范围。我们只需要调用系统的那一小部分，并且设置更容易开始。一般来说，要测试的元素越小，测试它就越快、越容易。

我们将定义三个不同级别或类型的测试，从小范围到大范围：

- 单元测试，用于仅检查部分服务的测试
- 集成测试，用于检查单个服务作为一个整体的测试
- 系统测试，用于检查多个服务协同工作的测试

名称实际上可以变化很大。在本书中，我们不会对定义非常严格，而是定义软限制并建议找到适合你特定项目的平衡点。不要害羞地为每个测试在适当的级别上做出决定并定义自己的命名法，并始终牢记创建测试需要付出多少努力才能确保它们总是值得的。

级别的定义可能有点模糊。例如，集成和单元测试可以并排定义，在这种情况下，它们之间的区别可能更具学术性。

让我们开始更详细地描述每个级别。

### 单元测试

最小的测试也是通常投入最多的测试，即单元测试。这种测试检查一小部分代码的行为，而不是整个系统。这个代码单元可以小到单个函数或测试单个 API 端点，等等。

> 正如我们上面所说，基于“单元”是什么以及它是否真的是一个单元，关于单元测试实际上应该有多大存在很多争论。例如，在某些情况下，如果测试涉及单个函数或类，人们只会将测试称为单元测试。

因为单元测试只检查一小部分功能，所以它可以很容易设置并快速运行。因此，进行新的单元测试是快速的，并且可以彻底测试系统，检查使整个系统按预期工作的单个小块。

单元测试的目的是深入检查服务的定义特性的行为。应该模拟任何外部请求或元素，这意味着它们被定义为测试的一部分。我们将在本章后面更详细地介绍单元测试，因为它们是 TDD 方法的关键元素。

### 集成测试

下一个级别是集成测试。这是检查一个服务或几个服务的整体行为。

集成测试的主要目标是确保同一服务内的不同服务或不同模块可以相互工作。在单元测试中，模拟外部请求，集成测试使用真实服务。

> 可能仍需要模拟外部 API。例如，为测试模拟外部支付提供商。但是，一般来说，集成测试应该使用尽可能多的真实服务，因为测试的重点是测试不同的服务是否一起工作。

需要注意的是，通常情况下，不同的服务将由不同的开发人员甚至不同的团队开发，并且他们对特定 API 的实现方式的理解可能存在差异，即使在定义明确的规范的情况下也是如此。

集成测试中的设置比单元测试更复杂，因为需要正确设置更多元素。这使得集成测试比单元测试更慢、更昂贵。

集成测试非常适合检查不同服务是否协同工作，但存在一些限制。

集成测试通常不像单元测试那样彻底，专注于检查基本功能并遵循愉快的道路。幸福路径是测试中的一个概念，意味着测试用例不应产生错误或异常。

预期的错误和异常通常在单元测试中进行测试，因为它们也是可能失败的元素。这并不意味着每一个集成测试都应该遵循一条快乐的道路。一些集成错误可能值得检查，但一般来说，一条快乐的路径会测试该功能的预期一般行为。它们将构成大部分集成测试。

### 系统测试

最后一级是系统级。系统测试检查所有不同的服务是否一起正常工作。

这种测试的一个要求是系统中实际上有多个服务。如果不是，它们与较低级别的测试没有什么不同。这些测试的主要目的是检查不同的服务是否可以合作，以及配置是否正确。

系统测试缓慢且难以实施。他们需要设置整个系统，并正确配置所有不同的服务。创建该环境可能很复杂。有时，实际执行任何系统测试的唯一方法是在实时环境中运行它们非常困难。

> 环境配置是这些测试检查的重要部分。这可能使它们在每个正在测试的环境（包括实时环境）上运行都很重要。

虽然这并不理想，但有时这是不可避免的，并且可以帮助提高部署后的信心，以确保新代码正常工作。在这种情况下，考虑到限制，只应运行最少量的测试，因为实时环境至关重要。要运行的测试还应该使用最大数量的常见功能和服务，以尽快检测到任何关键问题。这组测试有时称为验收测试或冒烟测试。它们可以手动运行，以确保一切看起来正确。

> 当然，冒烟测试不仅可以在实时环境中运行，而且可以作为确保其他环境正常工作的一种方式。

冒烟测试应该非常清晰、有据可查，并且经过精心设计以涵盖整个系统的最关键部分。理想情况下，它们也应该是只读的，因此它们在执行后不会留下无用的数据。

## 测试理念

与测试有关的所有事情的一个关键要素是另一个问题：为什么要测试？我们想用它来达到什么目的？

正如我们所见，测试是确保代码行为符合预期的一种方式。测试的目的是在代码发布并被真实用户使用之前检测可能的问题（有时称为缺陷）。

> 缺陷和错误之间存在细微差别。错误是一种缺陷，软件的行为方式与预期不同。例如，某些输入会产生意外错误。缺陷更普遍。缺陷可能是按钮不够可见，或者页面上的徽标不正确。一般来说，测试在检测错误方面比其他缺陷要好得多，但请记住我们所说的探索性测试。

未被检测到并被部署到实时系统中的缺陷修复起来非常昂贵。首先，它需要被检测。在具有大量活动的实时应用程序中，检测问题可能很困难（尽管我们将在第 16 章，持续架构中讨论它），但更糟糕的是，使用该应用程序的系统用户通常会检测到它。用户可能无法正确地反馈问题，因此问题仍然存在，从而产生问题或限制活动。检测用户可能会放弃系统，或者至少他们对系统的信心会降低。

任何声誉成本都会很糟糕，但也很难从用户那里提取足够的信息来确切地知道发生了什么以及如何解决它。这使得检测问题和解决问题之间的周期很长。

任何测试系统都会提高更早修复缺陷的能力。我们不仅可以创建一个特定的测试来模拟完全相同的问题，而且我们还可以创建一个定期执行测试的框架，以便有一个清晰的方法来检测和修复问题。

不同的测试级别对这个成本有不同的影响。一般来说，可以在单元测试级别检测到的任何问题都将更便宜地在那里修复，并且成本从那里增加。设计和运行单元测试比使用集成测试更容易和更快，并且集成测试比系统测试便宜。

不同的测试级别可以理解为捕获可能问题的不同层。如果出现，每一层都会捕获不同的问题。越接近过程的开始（编码时的设计和单元测试），创建一个能够检测和警告问题的密集网络就越便宜。解决问题的成本在过程开始时离受控环境越远，成本就会越高。

![](../images/10-1.png)

图 10.1：修复缺陷的成本越晚被发现

有些缺陷在单元测试级别是无法检测到的，例如不同部分的集成。这就是下一个级别发挥作用的地方。正如我们所见，最糟糕的情况是没有检测到问题，它会影响实时系统上的真实用户。

但是进行测试不仅是一次捕获问题的好方法。因为测试仍然可以保留，并在新代码更改时运行，所以它还在开发时创建了一个安全网，以确保创建新代码或修改代码不会影响旧功能。

> 根据持续集成实践，这是自动持续运行测试的最佳论据之一。开发人员可以专注于正在开发的功能，而持续集成工具将运行每个测试，并在某些测试出现问题时及早发出警报。先前引入的功能失败的问题称为回归。
>
> 回归问题很常见，因此拥有良好的测试覆盖率可以很好地防止它们未被检测到。可以引入涵盖先前功能的特定测试，以确保其按预期持续运行。这些是回归测试，有时会在我们检测到回归问题后添加。

拥有检查系统行为的良好测试的另一个好处是代码本身可以进行大量更改，因为知道行为将保持不变。可以进行这些更改以重组代码、清理代码并总体上改进代码。这些更改称为重构代码，即在不改变预期行为的情况下更改代码的编写方式。

现在，我们应该回答“什么是好的测试？”这个问题。正如我们所讨论的，编写测试不是免费的，需要付出努力，我们需要确保它是值得的。我们怎样才能创造出好的作品？

### 如何设计出色的测试

设计好的测试需要一定的心态。设计涵盖某些功能的代码的目标是使代码实现该功能，同时高效，编写清晰的代码，甚至可以描述为优雅。

测试的目的是确保功能符合预期的行为，并且所有可能出现的不同问题都会产生有意义的结果。

现在，为了能够真正对功能进行测试，心态应该是尽可能地强调代码。例如，让我们想象一个函数 divide(A, B)，它将两个整数相除在 -100 和 100 之间：A 在 B 之间。

在进行测试时，我们需要检查它的限制是什么，并尝试检查该函数是否以预期的行为正常执行。例如，可以创建以下测试：

| Action          | Expected behavior | Comments                   |
| --------------- | ----------------- | -------------------------- |
| divide(10, 2)   | return 5          | Basic case                 |
| divide(-20, 4)  | return -5         | 一个负整数和一个正整数相除 |
| divide(-10, -5) | return 2          | 将两个负整数相除           |
| divide(12, 2)   | return 5          | 不精确的划分               |
| divide(100, 50) | return 2          | A的最大值                  |
| divide(101, 50) | 产生输入错误      | A的值超过最大值            |
| divide(50, 100) | return 0          | B的最大值                  |
| divide(50, 101) | 产生输入错误      | B值超过最大值              |
| divide(10, 0)   | 产生异常          | 除以零                     |
| divide('10', 2) | 产生输入错误      | 参数 A 的格式无效          |
| divide(10, '2') | 产生输入错误      | 参数 B 的格式无效          |

请注意我们如何测试不同的可能性：

- 所有参数的通常行为都是正确的，并且除法工作正常。这包括正数和负数、精确除法和不精确除法。
- 最大值和最小值内的值：我们检查最大值是否命中并正确，并且正确检测下一个值。
- 除以零：对应该产生预定响应（异常）的功能的已知限制。
- 输入格式错误。

我们真的可以为简单的功能创建很多测试用例！请注意，所有这些情况都可以扩展。例如，我们可以添加```divide(-100, 50)``` 和```divide(100, -50)``` 的情况。在这些情况下，问题是相同的：这些测试是否增加了对问题的更好检测？

> 最好的测试是真正强调代码并确保它按预期工作的测试，非常努力地覆盖最困难的用例。让测试对被测代码提出难题是为实际操作准备代码的最佳方式。负载下的系统会看到各种组合，因此最好的准备是创建测试，尽可能努力地发现问题，以便能够在进入下一阶段之前解决它们。
>
> 这类似于足球训练，其中会进行一系列非常苛刻的练习，以确保受训者能够在比赛期间稍后进行表演。确保你的训练制度足够艰苦，可以为要求苛刻的比赛做好充分准备！

测试数量与不包含已由现有测试检查过的功能的测试之间的适当平衡（例如，创建一个大表，将数字划分为多个部分）可能在很大程度上取决于所测试的代码和组织中的实践.一些关键领域可能需要更彻底的测试，因为那里的失败可能更重要。

> 例如，任何外部 API 都应该小心地测试任何输入并对此采取真正的防御措施，因为外部用户可能会滥用外部 API。例如，测试在整数字段中输入字符串、添加无穷大或 NaN（非数字）值、超出有效负载限制、超出列表或页面的最大大小等时会发生什么。
>
> 相比之下，主要是内部的接口需要较少的测试，因为内部代码不太可能滥用 API。例如，如果除法函数只是内部的，则可能不需要测试输入格式是否不正确，只需检查限制是否得到遵守。

请注意，测试是独立于代码实现的。测试定义纯粹是从要测试的函数的外部视图完成的，不需要知道里面是什么。这称为黑盒测试。健康的测试套件总是以这种方法开始。

作为开发人员编写测试的关键能力是脱离代码本身的知识并独立进行测试。

> 测试可以如此独立，以至于它可能会使用独立的人来创建测试，就像 QA 团队执行测试一样。不幸的是，这不是单元测试的可行方法，单元测试很可能由编写代码本身的开发人员创建。

在某些情况下，这种外部方法是不够的。如果开发人员知道某些特定区域可能存在问题，那么最好用测试来补充它，以检查从外部角度看不明显的功能。

例如，基于某些输入计算结果的函数可能有一个内部点，其中算法更改为使用不同的模型进行计算。外部用户不需要知道此信息，但最好添加一些检查以确保转换正常工作。

与前面讨论的黑盒方法相比，这种测试称为白盒测试。

> 重要的是要记住，在测试套件中，白盒测试应该始终仅次于黑盒测试。主要目标是从外部角度测试功能。白盒测试可能是一个很好的补充，尤其是在某些方面，但它应该具有较低的优先级。
>
> 开发能够创建良好的黑盒测试的能力很重要，应该传递给团队。

黑盒测试试图避免一个常见问题，即同一个开发人员同时编写代码和测试，然后检查代码中实现的功能的解释是否按预期工作，而不是检查它是否按预期工作一个外部端点。我们稍后会看一下 TDD，它试图通过在编写代码之前编写测试来确保创建测试时没有考虑到实现。

### 结构化测试

在结构方面，特别是对于单元测试，构建测试的一个好方法是使用 Arrange Act Assert (**AAA**) 模式。

这种模式意味着测试处于三个不同的阶段：

- Arrange：为测试准备环境。这包括所有设置，以在执行下一步之前使系统处于稳定状态。
- 执行：执行作为测试目标的行动。
- 断言：检查操作的结果是否是预期的结果。

测试的结构是这样的：

**GIVEN** (Arrange) 已知环境，**ACTION** (Act) 产生指定的 **RESULT** (Assert)

> 这种模式有时也称为 GIVEN, WHEN, THEN，因为每个步骤都可以用这些术语来描述。

请注意，此结构旨在使所有测试都是独立的，并且每个测试都测试一个事物。

> 一个常见的不同模式是将测试中的动作步骤分组，在单个测试中测试多个功能。例如，测试写入值是否正确，然后检查该值的搜索是否返回正确的值。这不会遵循 AAA 模式。相反，要遵循 AAA 模式，应该创建两个测试，第一个是验证写入是否正确，第二个是在执行搜索之前的排列步骤中创建值作为设置的一部分。

请注意，无论测试是通过代码执行还是手动运行，都可以使用此结构，尽管它们将更多地用于自动化测试。当手动运行它们时，Arrange 阶段可能需要很长时间才能为每个测试生成，导致花费大量时间。相反，手动测试通常按照我们上面描述的模式组合在一起，执行一系列 Act 和 Assert 并使用前一个阶段的输入作为下一个阶段的设置。这在要求以特定顺序运行测试时产生了依赖性，这对于单元测试套件来说并不是很好，但对于冒烟测试或其他安排步骤非常昂贵的环境来说可能会更好。

> 同样，如果要测试的代码是纯函数式的（意味着只有输入参数是决定其状态的参数，如上面的除法示例），则不需要排列步骤。

让我们看一个使用这种结构创建的代码示例。想象一下，我们有一个要测试的方法，称为 ```method_to_test```。该方法是名为 ```ClassToTest``` 的类的一部分。

```python
def test_example():
    # Arrange step
    # Create the instance of the class to test
    object_to_test = ClassToTest(paramA='some init param', 
                                 paramB='another init param')
    # Act step
    response = object_to_test.method_to_test(param='execution_param')
    # Assert step
    assert response == 'expected result'
```

每个步骤都定义得非常清楚。在这种情况下，第一个准备了我们要测试的类中的一个对象。请注意，我们可能需要添加一些参数或一些准备工作，以便对象处于已知起点，以便后续步骤按预期工作。

Act 步骤仅生成正在测试的操作。在这种情况下，使用适当的参数为准备好的对象调用 ```method_to_test``` 方法。

最后，Assert 步骤非常简单，只需检查响应是否为预期响应。

> 通常，Act 和 Assert 步骤的定义和编写都很简单。安排步骤通常是测试的大部分工作。

使用 **AAA** 模式进行测试的另一个常见模式是在排列步骤中创建用于测试的通用函数。例如，创建一个可能需要复杂设置的基本环境，然后拥有多个副本，其中 Act 和 Assert 步骤不同。这减少了代码的重复。

例如：

```python
def create_basic_environment():
    object_to_test = ClassToTest(paramA='some init param', 
                                 paramB='another init param')
    # This code may be much more complex and perhaps have
    # 100 more lines of code, because the basic environment
    # to test requires a lot of things to set up
    return object_to_test
def test_exampleA():
    # Arrange
    object_to_test = create_basic_environment()
    # Act
    response = object_to_test.method_to_test(param='execution_param')
    # Assert
    assert response == 'expected result B'
def test_exampleB():
    # Arrange
    object_to_test = create_basic_environment()
    # Act
    response = object_to_test.method_to_test(param='execution_param')
    # Assert
    assert response == 'expected result B'
```

稍后我们将看到如何构建多个非常相似的测试以避免重复，这在拥有大型测试套件时会出现问题。正如我们在上面看到的，拥有大型测试套件对于创建良好的测试覆盖率很重要。

> 测试中的重复在一定程度上是不可避免的，甚至在一定程度上是健康的。当由于发生更改而更改代码的某些部分的行为时，需要相应地更改测试以适应更改。此更改有助于权衡更改的大小并避免轻率地进行大更改，因为测试将提醒受影响的功能。
>
> 尽管如此，无意识的重复并不是很好，我们稍后会看到一些减少重复代码量的选项。

## 测试驱动开发

一种非常流行的编程技术是测试驱动开发或 TDD。 TDD 包括将测试置于开发体验的中心。

这建立在我们在本章前面介绍的一些想法的基础上，尽管以更一致的观点来研究它们。

开发软件的 TDD 流程如下：

1. 决定将新功能添加到代码中。
2. 编写了一个新测试来定义新功能。请注意，这是在代码之前完成的。
3. 运行测试套件以显示它失败了。
4. 然后将新功能添加到主代码中，重点是简单性。只应添加所需的功能，无需额外的细节。
5. 运行测试套件以显示新测试正在运行。这可能需要多次执行，直到代码准备好。
6. 新功能已准备就绪！现在可以重构代码以改进它，避免重复，重新排列元素，将其与以前存在的代码分组等。

对于任何新功能，循环可以重新开始。

如你所见，TDD 基于三个主要思想：

- 在编写代码之前编写测试：这可以防止创建与当前实现紧密耦合的测试的问题，从而迫使开发人员在开始编写之前考虑测试和功能。它还强制开发人员在编写功能之前检查测试是否确实失败，以确保稍后会检测到问题。这类似于我们之前在如何设计出色的测试部分中描述的黑盒测试方法。
- 不断运行测试：该过程的一个关键部分是运行整个测试套件以检查系统中的所有功能是否正确。每次创建新测试时，以及在编写功能时，都会一遍又一遍地完成。运行测试是 TDD 开发的重要组成部分。这可确保始终检查所有功能，并且代码始终按预期工作，因此可以快速解决任何错误或差异。
- 以非常小的增量工作：专注于手头的任务，因此每一步都会构建和扩展一个庞大的测试套件，并深入涵盖代码的全部功能。

这个大型测试套件创建了一个安全网，允许你经常执行代码重构，无论大小，从而不断改进代码。小增量意味着特定的小测试，在添加代码之前需要考虑。

> 这个想法的扩展是专注于只编写手头任务所需的代码，而不是更多。这有时被称为 **YAGNI** 原则（你不需要它）。该原则的目的是防止过度设计或为“未来可预见的请求”创建代码，在实践中，这些代码很可能永远不会实现，更糟糕的是，会使代码更难在其他方向上更改。鉴于软件开发是出了名的难以提前计划，重点应该放在保持小事上，不要让自己走得太远。

这三个想法在开发周期中不断相互作用，并将测试置于开发过程的中心，因此得名。

TDD 的另一个重要优势是，将重点放在测试上意味着从一开始就考虑如何测试代码，这有助于设计易于测试的代码。此外，减少要编写的代码量，专注于通过测试的严格要求，可以减少过度设计的可能性。创建小测试和增量工作的要求也倾向于生成模块化代码，这些小单元组合在一起但能够独立测试。

一般流程是不断处理新的失败测试，使它们通过然后重构，有时称为“红/绿/重构”模式：测试失败时为红色，所有测试都通过时为绿色。

重构是 TDD 过程的一个关键方面。强烈鼓励不断提高现有代码的质量。这种工作方式的最佳结果之一是生成了涵盖代码功能的每个细节的非常广泛的测试套件，这意味着可以在知道有一个坚实的基础来捕获由更改引入的任何问题的情况下重构代码代码和添加错误。

众所周知，通过重构来提高代码的可读性、可用性等在提高开发人员的士气和加快引入更改的速度方面具有良好的影响，因为代码保持良好状态。

> 一般来说，不仅在 TDD 中，留出时间清理旧代码并改进它对于保持良好的更改速度至关重要。陈旧的旧代码往往越来越难以使用，并且随着时间的推移，将需要更多的努力来更改它以进行更多更改。鼓励健康的习惯来关心代码的当前状态并留出时间进行维护改进，这对于任何软件系统的长期可持续性都至关重要。

TDD 的另一个重要方面是对快速测试的要求。由于测试总是按照 TDD 实践运行，因此总执行时间非常重要。应该仔细考虑每个测试所花费的时间，因为不断增长的测试套件会使其运行时间更长。

有一个通常的阈值会导致焦点丢失，因此运行时间超过 10 秒左右的测试将使它们不是“同一操作的一部分”，从而使开发人员冒着考虑其他事情的风险。

显然，在 10 秒内运行整个测试套件将非常困难，尤其是随着测试数量的增加。一个复杂应用程序的完整单元测试套件可以包含 10,000 个或更多测试！在现实生活中，有多种策略可以帮助缓解这一事实。

整个测试套件不需要一直运行。相反，任何测试运行器都应该允许你选择要运行的测试范围，从而允许你在开发功能时减少每次运行时要运行的测试数量。例如，这意味着只运行与同一模块相关的测试。在某些情况下，它甚至可能意味着运行单个测试以加快结果速度。

> 当然，在某些时候，应该运行整个测试套件。 TDD 实际上与持续集成保持一致，因为它也基于运行测试，这一次是在代码签出到存储库后自动进行的。能够在本地运行一些测试以确保在代码提交到 repo 后在后台运行整个测试套件进行开发时，事情正常工作的组合很棒。

无论如何，由于在 TDD 中运行测试所花费的时间很重要，因此观察测试的持续时间很重要，并且生成可以快速运行的测试是能够以 TDD 方式工作的关键。这主要是通过创建覆盖一小部分代码的测试来实现的，因此可以控制设置时间。

> TDD 实践最适合单元测试。集成和系统测试可能需要与 TDD 工作所需的速度和紧密反馈循环不兼容的大型设置。
>
> 幸运的是，正如我们之前看到的，单元测试是大部分测试通常集中在大多数项目上的地方。

### 将 TDD 引入新团队

在组织中引入 TDD 实践可能很棘手，因为它们改变了执行非常基本的操作的方式，并且与通常的工作方式（编写代码后编写测试）有些背道而驰。

在考虑将 TDD 引入团队时，最好有一个倡导者可以充当团队其他成员的联络点，并解决通过创建测试可能出现的问题和问题。

TDD 在结对编程也很普遍的环境中非常流行，因此在培训其他开发人员并介绍实践的同时让某人主持会议是另一种可能性。

> 请记住，TDD 的关键要素是迫使开发人员在开始考虑实现之前首先考虑如何测试特定功能的心态。这种心态不是天生的，需要训练和练习。

将 TDD 技术应用于现有代码可能具有挑战性，因为在此配置中可能难以测试预先存在的代码，尤其是在开发人员不熟悉该实践的情况下。不过，TDD 非常适合新项目，因为新代码的测试套件将与代码同时创建。在现有项目中启动新模块的混合方法，因此大多数代码都是新的并且可以使用 TDD 技术进行设计，减少了处理遗留代码的问题。

如果你想看看 TDD 是否对新代码有效，试着从小处着手，使用一些小团队的小项目，以确保它不会造成太大的破坏，并且可以正确消化和应用这些原则。有些开发人员非常喜欢使用 TDD 原则，因为它符合他们的个性以及他们处理开发过程的方式。请记住，这不一定是每个人都会有的感受，从这些实践开始需要时间，也许不可能 100% 应用它们，因为之前的代码可能会限制它。

### 问题和局限

TDD 实践在业界非常流行并被广泛遵循，尽管它们有其局限性。一是大型测试运行时间过长的问题。在某些情况下，这些测试可能是不可避免的。

另一个是如果不从一开始就完全采用这种方法的困难，因为已经编写了部分代码，并且可能应该添加新的测试，这违反了在代码之前创建测试的规则。

另一个问题是设计新代码，而要实现的功能是流动的并且没有完全定义。这需要进行实验，例如，设计一个函数来返回与输入颜色形成对比的颜色，例如，根据用户可选择的主题呈现对比色。此功能可能需要检查以查看它是否“看起来正确”，这可能需要进行调整，而这是通过预配置的单元测试难以实现的。

不是 TDD 特有的问题，但需要注意的是要记住避免测试之间的依赖关系。任何测试套件都可能发生这种情况，但考虑到创建新测试的重点，如果团队从 TDD 实践开始，这可能是一个问题。可以通过要求测试以特定顺序运行来引入依赖关系，因为测试会污染环境。这通常不是故意的，而是在编写多个测试时无意中完成的。

> 一个典型的影响是，如果独立运行，一些测试会失败，因为在这种情况下它们的依赖关系不会运行。

无论如何，请记住，TDD 不一定是全有或全无，而是一组可以帮助你设计经过良好测试和高质量的代码的想法和实践。并非系统中的每个测试都需要使用 TDD 进行设计，但其中很多都可以。

### TDD 流程示例

假设我们需要创建一个函数：

- 对于小于 0 的值，返回零
- 对于大于 10 的值，返回 100
- 对于介于两者之间的值，它返回两个值的幂。请注意，对于边，它返回两个输入的幂（0 表示 0，100 表示 10）

为了以完整的 TDD 方式编写代码，我们从尽可能小的测试开始。让我们创建最小的骨架和第一个测试。

```python
def parameter_tdd(value):
    pass
assert parameter_tdd(5) == 25
```

我们运行测试，并得到一个测试失败的错误。现在，我们将使用纯 Python 代码，但在本章后面，我们将看到如何更有效地运行测试。

```python
$ python3 tdd_example.py
Traceback (most recent call last):
  File ".../tdd_example.py", line 6, in <module>
    assert parameter_tdd(5) == 25
AssertionError
```

用例的实现非常简单。

```python
def parameter_tdd(value):
    return 25
```

是的，我们实际上返回了一个硬编码的值，但这确实是通过第一个测试所需的全部内容。现在让我们运行测试，你将看不到任何错误。

```python
$ python3 tdd_example.py
```

但现在我们为下边缘添加测试。虽然这是两条线，但它们可以被视为相同的测试，因为它们正在检查边缘是否正确。

```python
assert parameter_tdd(-1) == 0
assert parameter_tdd(0) == 0
assert parameter_tdd(5) == 25
```

让我们再次运行测试。

```python
$ python3 tdd_example.py
Traceback (most recent call last):
  File ".../tdd_example.py", line 6, in <module>
    assert parameter_tdd(-1) == 0
AssertionError
```

我们需要添加代码来处理下边缘。

```python
def parameter_tdd(value):
    if value <= 0:
        return 0
    return 25
```

运行测试时，我们看到它正在正确运行测试。现在让我们添加参数来处理上边缘。

```python
assert parameter_tdd(-1) == 0
assert parameter_tdd(0) == 0
assert parameter_tdd(5) == 25
assert parameter_tdd(10) == 100
assert parameter_tdd(11) == 100
```

这会触发相应的错误。

```python
$ python3 tdd_example.py
Traceback (most recent call last):
  File "…/tdd_example.py", line 12, in <module>
    assert parameter_tdd(10) == 100
AssertionError
```

让我们添加更高的边缘。

```python
def parameter_tdd(value):
    if value <= 0:
        return 0
    if value >= 10:
        return 100
    return 25
```

这运行正确。我们不确定所有的代码都没有问题，而且我们真的想确保中间部分是正确的，所以我们添加了另一个测试。

```python
assert parameter_tdd(-1) == 0
assert parameter_tdd(0) == 0
assert parameter_tdd(5) == 25
assert parameter_tdd(7) == 49
assert parameter_tdd(10) == 100
assert parameter_tdd(11) == 100
```

啊哈！由于最初的硬编码，现在它显示一个错误。

```python
$ python3 tdd_example.py
Traceback (most recent call last):
  File "/…/tdd_example.py", line 15, in <module>
    assert parameter_tdd(7) == 49
AssertionError
```

所以让我们修复它。

```python
def parameter_tdd(value):
    if value <= 0:
        return 0
    if value >= 10:
        return 100
    return value ** 2
```

这将正确运行所有测试。现在，有了测试的安全网，我们认为我们可以稍微重构代码以清理它。

```python
def parameter_tdd(value):
    if value < 0:
        return 0
    if value < 10:
        return value ** 2
    return 100
```

我们可以在整个过程中运行测试，并确保代码是正确的。根据团队认为好的代码或更明确的代码，最终结果可能会有所不同，但我们有我们的测试套件，可以确保测试是一致的，并且行为是正确的。

这里的函数很小，但这显示了以 TDD 样式编写代码时的流程。

## Python中的单元测试简介

在 Python 中运行测试有多种方法。一个，正如我们在上面看到的，有点粗略，是执行带有多个断言的代码。一个常见的是标准库单元测试。

### Python unittest

unittest 是 Python 标准库中包含的一个模块。它基于创建一个测试类的概念，该类将几种测试方法分组。让我们编写一个新文件，其中包含以正确格式编写的测试，称为 test_unittest_example.py。

```python
import unittest
from tdd_example import parameter_tdd
class TestTDDExample(unittest.TestCase):
    def test_negative(self):
        self.assertEqual(parameter_tdd(-1), 0)
    def test_zero(self):
        self.assertEqual(parameter_tdd(0), 0)
    def test_five(self):
        self.assertEqual(parameter_tdd(5), 25)
    def test_seven(self):
        # Note this test is incorrect
        self.assertEqual(parameter_tdd(7), 0)
    def test_ten(self):
        self.assertEqual(parameter_tdd(10), 100)
    def test_eleven(self):
        self.assertEqual(parameter_tdd(11), 100)
if __name__ == '__main__':
    unittest.main()
```

让我们分析不同的元素。第一个是顶部的进口。

```python
import unittest
from tdd_example import parameter_tdd
```

我们导入 ```unittest``` 模块和要测试的函数。接下来是最重要的部分，它定义了测试。

```python
class TestTDDExample(unittest.TestCase):
    def test_negative(self):
        self.assertEqual(parameter_tdd(-1), 0)
```

```TestTDDExample``` 类对不同的测试进行分组。请注意，它继承自 ```unittest.TestCase```。然后，以 ```test_``` 开头的方法将产生独立的测试。在这里，我们将展示一个。在内部，它调用函数并将结果与 0 进行比较，使用 ```self.assertEqual``` 函数。

>请注意 test_seven 的定义不正确。我们这样做是为了在运行时产生错误。

最后，我们添加这段代码。

```python
if __name__ == '__main__':
    unittest.main()
```

如果我们运行文件，这会自动运行测试。所以，让我们运行文件：

```python
$ python3 test_unittest_example.py
...F..
======================================================================
FAIL: test_seven (__main__.TestTDDExample)
----------------------------------------------------------------------
Traceback (most recent call last):
  File ".../unittest_example.py", line 17, in test_seven
    self.assertEqual(parameter_tdd(7), 0)
AssertionError: 49 != 0
----------------------------------------------------------------------
Ran 6 tests in 0.001s
FAILED (failures=1)
```

如你所见，它已运行所有六个测试，并显示任何错误。在这里，我们可以清楚地看到问题所在。如果我们需要更多细节，我们可以使用 ```-v``` 来显示正在运行的每个测试：

```python
$ python3 test_unittest_example.py -v
test_eleven (__main__.TestTDDExample) ... ok
test_five (__main__.TestTDDExample) ... ok
test_negative (__main__.TestTDDExample) ... ok
test_seven (__main__.TestTDDExample) ... FAIL
test_ten (__main__.TestTDDExample) ... ok
test_zero (__main__.TestTDDExample) ... ok
======================================================================
FAIL: test_seven (__main__.TestTDDExample)
----------------------------------------------------------------------
Traceback (most recent call last):
  File ".../unittest_example.py", line 17, in test_seven
    self.assertEqual(parameter_tdd(7), 0)
AssertionError: 49 != 0
----------------------------------------------------------------------
Ran 6 tests in 0.001s
FAILED (failures=1)
```

你还可以使用 ```-k``` 选项运行单个测试或它们的组合，该选项搜索匹配的测试。

```python
$ python3 test_unittest_example.py -v -k test_ten
test_ten (__main__.TestTDDExample) ... ok
----------------------------------------------------------------------
Ran 1 test in 0.000s
OK
```

```unittest``` 非常流行并且可以接受很多选项，并且它几乎与 Python 中的每个框架都兼容。它在测试方式方面也非常灵活。例如，有多种方法可以比较值，如 ```assertNotEqual``` 和 ```assertGreater```。

> 有一个特定的断言函数工作方式不同，即 assertRaises，用于检测代码何时生成异常。我们稍后会在测试模拟外部调用时看看它。

它还具有 ```setUp``` 和 ```tearDown``` 方法，用于在执行类中的每个测试之前和之后执行代码。

> 请务必查看官方文档：https://docs.python.org/3/library/unittest.html。

虽然 ```unittest``` 可能是最流行的测试框架，但它并不是最强大的。让我们来看看它。

### pytest

Pytest 进一步简化了编写测试。关于 unittest 的一个常见抱怨是它迫使你设置许多不明显的 ```assertCompare``` 调用。它还需要构建测试，添加一些样板代码，如测试类。其他问题并不那么明显，但是在创建大型测试套件时，不同测试的设置可能会开始变得复杂。

> 一种常见的模式是创建从其他测试类继承的类。随着时间的推移，它可以长出自己的腿。

相反，```Pytest``` 简化了测试的运行和定义，并使用更易于阅读和识别的标准断言语句捕获所有相关信息。

> 在本节中，我们将以最简单的方式使用 ```pytest```。在本章的后面，我们将介绍更多有趣的案例。

请务必在你的环境中通过 ```pip``` 安装 ```pytest```。

```sh
$ pip3 install pytest
```

让我们看看如何在文件 test_pytest_example.py 中运行 unittest 中定义的测试。

```python
from tdd_example import parameter_tdd
def test_negative():
    assert parameter_tdd(-1) == 0
def test_zero():
    assert parameter_tdd(0) == 0
def test_five():
    assert parameter_tdd(5) == 25
def test_seven():
    # Note this test is deliberatly set to fail
    assert parameter_tdd(7) == 0
def test_ten():
    assert parameter_tdd(10) == 100
def test_eleven():
    assert parameter_tdd(11) == 100
```

如果将其与 ```test_unittest_example.py``` 中的等效代码进行比较，则代码明显更精简。使用 ```pytest``` 运行它时，它还会显示更详细的彩色信息。

```python
$ pytest test_unittest_example.py
================= test session starts =================
platform darwin -- Python 3.9.5, pytest-6.2.4, py-1.10.0, pluggy-0.13.1
collected 6 items
test_unittest_example.py ...F..                 [100%]
====================== FAILURES =======================
______________ TestTDDExample.test_seven ______________
self = <test_unittest_example.TestTDDExample testMethod=test_seven>
    def test_seven(self):
>       self.assertEqual(parameter_tdd(7), 0)
E       AssertionError: 49 != 0
test_unittest_example.py:17: AssertionError
=============== short test summary info ===============
FAILED test_unittest_example.py::TestTDDExample::test_seven
============= 1 failed, 5 passed in 0.10s =============
```

与 ```unittest``` 一样，我们可以使用 ```-v``` 查看更多信息并使用 ```-k``` 运行一系列测试。

```python
$ pytest -v test_unittest_example.py
========================= test session starts =========================
platform darwin -- Python 3.9.5, pytest-6.2.4, py-1.10.0, pluggy-0.13.1 -- /usr/local/opt/python@3.9/bin/python3.9
cachedir: .pytest_cache
collected 6 items
test_unittest_example.py::TestTDDExample::test_eleven PASSED      [16%]
test_unittest_example.py::TestTDDExample::test_five PASSED        [33%]
test_unittest_example.py::TestTDDExample::test_negative PASSED    [50%]
test_unittest_example.py::TestTDDExample::test_seven FAILED       [66%]
test_unittest_example.py::TestTDDExample::test_ten PASSED         [83%]
test_unittest_example.py::TestTDDExample::test_zero PASSED        [100%]
============================== FAILURES ===============================
______________________ TestTDDExample.test_seven ______________________
self = <test_unittest_example.TestTDDExample testMethod=test_seven>
    def test_seven(self):
>       self.assertEqual(parameter_tdd(7), 0)
E       AssertionError: 49 != 0
test_unittest_example.py:17: AssertionError
======================= short test summary info =======================
FAILED test_unittest_example.py::TestTDDExample::test_seven - AssertionErr...
===================== 1 failed, 5 passed in 0.08s =====================
$ pytest test_pytest_example.py -v -k test_ten
========================= test session starts =========================
platform darwin -- Python 3.9.5, pytest-6.2.4, py-1.10.0, pluggy-0.13.1 -- /usr/local/opt/python@3.9/bin/python3.9
cachedir: .pytest_cache
collected 6 items / 5 deselected / 1 selected
test_pytest_example.py::test_ten PASSED                           [100%]
=================== 1 passed, 5 deselected in 0.02s ===================
```

它与 ```unittest``` 定义的测试完全兼容，允许你组合两种样式或迁移它们。

```python
$ pytest test_unittest_example.py
========================= test session starts =========================
platform darwin -- Python 3.9.5, pytest-6.2.4, py-1.10.0, pluggy-0.13.1
collected 6 items
test_unittest_example.py ...F..                                   [100%]
============================== FAILURES ===============================
______________________ TestTDDExample.test_seven ______________________
self = <test_unittest_example.TestTDDExample testMethod=test_seven>
    def test_seven(self):
>       self.assertEqual(parameter_tdd(7), 0)
E       AssertionError: 49 != 0
test_unittest_example.py:17: AssertionError
======================= short test summary info =======================
FAILED test_unittest_example.py::TestTDDExample::test_seven - AssertionErr...
===================== 1 failed, 5 passed in 0.08s =====================
```

```pytest``` 的另一个重要功能是轻松自动发现以查找以 ```test_``` 开头并在所有测试中运行的文件。如果我们尝试一下，指向当前目录，我们可以看到它同时运行 ```test_unittest_example.py``` 和 ```test_pytest_example.py```。

```python
$ pytest .
========================= test session starts =========================
platform darwin -- Python 3.9.5, pytest-6.2.4, py-1.10.0, pluggy-0.13.1
collected 12 items
test_pytest_example.py ...F..                                    [50%]
test_unittest_example.py ...F..                                  [100%]
============================== FAILURES ===============================
_____________________________ test_seven ______________________________
    def test_seven():
        # Note this test is deliberatly set to fail
>       assert parameter_tdd(7) == 0
E       assert 49 == 0
E        +  where 49 = parameter_tdd(7)
test_pytest_example.py:18: AssertionError
______________________ TestTDDExample.test_seven ______________________
self = <test_unittest_example.TestTDDExample testMethod=test_seven>
    def test_seven(self):
>       self.assertEqual(parameter_tdd(7), 0)
E       AssertionError: 49 != 0
test_unittest_example.py:17: AssertionError
======================= short test summary info =======================
FAILED test_pytest_example.py::test_seven - assert 49 == 0
FAILED test_unittest_example.py::TestTDDExample::test_seven - AssertionErr...
==================== 2 failed, 10 passed in 0.23s =====================
```


我们将在本章中继续讨论 ```pytest``` 的更多特性，但首先，我们需要回到代码有依赖关系时如何定义测试。

## 测试外部依赖

在构建单元测试时，我们讨论了它如何基于在代码中隔离一个单元以独立测试它的概念。

这种隔离概念是关键，因为我们希望专注于代码的小部分以创建小而清晰的测试。创建小型测试也有助于保持测试速度。

在上面的示例中，我们测试了一个没有依赖关系的纯函数函数 ```parameter_tdd```。它没有使用任何外部库或任何其他功能。但不可避免地，在某些时候，你需要测试依赖于其他东西的东西。

这种情况下的问题是其他组件是否应该成为测试的一部分？

这不是一个容易回答的问题。一些开发人员认为所有单元测试都应该纯粹针对单个函数或方法，因此，任何依赖项都不应该是测试的一部分。但是，在更实际的层面上，有时会有一些代码组成一个单元，与单独测试相比，它们更容易组合测试。

例如，考虑一个函数：

- 对于小于 0 的值，返回零。
- 对于大于 100 的值，返回 10。
- 对于介于两者之间的值，它返回值的平方根。请注意，对于边，它返回它们的平方根（0 表示 0，10 表示 100）。

这与之前的函数 ```parameter_tdd``` 非常相似，但这次我们需要外部库的帮助来生成数字的平方根。让我们看一下代码。

它分为两个文件。 ```dependent.py``` 包含函数的定义。

```python
import math
def parameter_dependent(value):
    if value < 0:
        return 0
    if value <= 100:
        return math.sqrt(value)
    return 10
```

该代码与 parameter_tdd 示例中的代码非常相似。模块 math.sqrt 返回一个数字的平方根。

测试在 test_dependent.py 中。

```python
from dependent import parameter_dependent
def test_negative():
    assert parameter_dependent(-1) == 0
def test_zero():
    assert parameter_dependent(0) == 0
def test_twenty_five():
    assert parameter_dependent(25) == 5
def test_hundred():
    assert parameter_dependent(100) == 10
def test_hundred_and_one():
    assert parameter_dependent(101) == 10
```

在这种情况下，我们完全使用外部库并在测试代码的同时对其进行测试。对于这个简单的示例，这是一个完全有效的选项，尽管在其他情况下可能并非如此。

> 该代码可在 GitHub 上的 https://github.com/PacktPublishing/Python-Architecture-Patterns/tree/main/chapter_10_testing_and_tdd 获得。

例如，外部依赖项可能是进行需要捕获的外部 HTTP 调用，以防止在运行测试时进行这些调用，并控制返回的值，或者其他应该单独测试的大块功能。

要将函数与其依赖项分离，有两种不同的方法。我们将使用 ```parameter_dependent``` 作为基线来展示它们。

> 同样，在这种情况下，测试在包含依赖项的情况下工作得非常好，因为它很简单并且不会产生像外部调用等副作用。

接下来我们将看到如何模拟外部调用。

### Mocking

模拟是一种在内部替换依赖项的做法，在测试本身的控制下用虚假调用替换它们。这样，我们可以为任何外部依赖引入一个已知的响应，而不是调用实际的代码。

> 在内部，模拟是使用所谓的猴子补丁实现的，这是用替代品动态替换现有库。虽然这可以在不同的编程语言中以不同的方式实现，但它在 Python 或 Ruby 等动态语言中尤其流行。 Monkey-patching 可以用于测试以外的其他目的，但应该小心使用，因为它可以改变库的行为并且对于调试来说非常令人不安。

为了能够模拟代码，在我们的测试代码中，我们需要准备模拟作为排列步骤的一部分。有不同的库来模拟调用，但最简单的是使用标准库中包含的 unittest.mock 库。

mock 最简单的用法是修补外部库：

```python
from unittest.mock import patch
from dependent import parameter_dependent
@patch('math.sqrt')
def test_twenty_five(mock_sqrt):
    mock_sqrt.return_value = 5
    assert parameter_dependent(25) == 5
    mock_sqrt.assert_called_once_with(25)
```

补丁装饰器拦截对已定义库 ```math.sqrt``` 的调用，并将其替换为传递给函数的模拟对象，此处称为 ```mock_sqrt```。

这个对象有点特别。它基本上允许任何调用，访问几乎任何方法或属性（预定义的除外），并不断返回一个模拟对象。这使得模拟对象变得非常灵活，可以适应它周围的任何代码。必要时，可以调用 ```.return_value``` 设置返回值，如第一行所示。

本质上，我们是说对 ```mock_sqrt``` 的调用将返回值 5。因此，我们正在准备外部调用的输出，以便我们可以控制它。

最后，我们检查我们是否调用了一次模拟 ```mock_sqrt```，输入 (25) 使用方法 ```assert_call_once_with```。

本质上，我们是：

- 准备模拟以替换 ```math.sqrt```
- 设置调用时返回的值
- 检查呼叫是否按预期工作
- 仔细检查是否使用正确的值调用了模拟

对于其他测试，比如我们可以检查mock没有被调用，说明外部依赖没有被调用。

```python
@patch('math.sqrt')
def test_hundred_and_one(mock_sqrt):
    assert parameter_dependent(101) == 10
    mock_sqrt.assert_not_called()
```

有多个```断言```函数可让你检测模拟的使用方式。一些例子：

- 根据模拟是否已被调用，被调用属性返回 True 或 False，允许你编写：
    ```assert mock_sqrt.called is True```
- ```call_count``` 属性返回模拟被调用的次数。
- ```assert_called_with()``` 方法检查它被调用的次数。如果最后一次调用不是以指定的方式产生的，它将引发异常。
- ```assert_any_call()``` 方法检查是否以指定的方式产生了任何调用。

有了这些信息，用于测试的完整文件 ```test_dependent_mocked_test.py``` 将是这样的。

```python
from unittest.mock import patch
from dependent import parameter_dependent
@patch('math.sqrt')
def test_negative(mock_sqrt):
    assert parameter_dependent(-1) == 0
    mock_sqrt.assert_not_called()
@patch('math.sqrt')
def test_zero(mock_sqrt):
    mock_sqrt.return_value = 0
    assert parameter_dependent(0) == 0
    mock_sqrt.assert_called_once_with(0)
@patch('math.sqrt')
def test_twenty_five(mock_sqrt):
    mock_sqrt.return_value = 5
    assert parameter_dependent(25) == 5
    mock_sqrt.assert_called_with(25)
@patch('math.sqrt')
def test_hundred(mock_sqrt):
    mock_sqrt.return_value = 10
    assert parameter_dependent(100) == 10
    mock_sqrt.assert_called_with(100)
@patch('math.sqrt')
def test_hundred_and_one(mock_sqrt):
    assert parameter_dependent(101) == 10
    mock_sqrt.assert_not_called()
```

如果 ```mock``` 需要返回不同的值，可以将 ```mock``` 的 ```side_effect``` 属性定义为列表或元组。 ```side_effect``` 类似于 ```return_value```，但有一些区别，我们将看到。

```python
@patch('math.sqrt')
def test_multiple_returns_mock(mock_sqrt):
    mock_sqrt.side_effect = (5, 10)
    assert parameter_dependent(25) == 5
    assert parameter_dependent(100) == 10
```

如果需要，```side_effect``` 也可用于产生异常。

```python
import pytest
from unittest.mock import patch
from dependent import parameter_dependent
@patch('math.sqrt')
def test_exception_raised_mock(mock_sqrt):
    mock_sqrt.side_effect = ValueError('Error on the external library')
    with pytest.raises(ValueError):
        parameter_dependent(25)
```

```with``` 部分断言在块中引发了预期的异常。如果不是，则显示错误。

> 在 ```unittest``` 中，可以使用类似的 ```with``` 块来检查引发的异常。
>
> ```python
> with self.assertRaises(ValueError):
>     parameter_dependent(25)
> ```

模拟并不是处理测试依赖关系的唯一方法。接下来我们将看到不同的方法。

### 依赖注入

虽然模拟在没有注意到原始代码的情况下替换了依赖项，但通过在外部修补它，依赖注入是一种在调用被测函数时使该依赖项显式化的技术，因此可以用测试替代品替换它。

从本质上讲，它是一种设计代码的方式，通过要求它们作为输入参数来明确依赖关系。

> 依赖注入虽然对测试很有用，但不仅仅针对于此。通过显式添加依赖项，它还减少了函数知道如何初始化特定依赖项的需要，而不是依赖于依赖项的接口。它在“初始化”依赖项（应该在外部处理）和“使用”它（这是依赖代码将执行的唯一部分）之间创建了一个分离。稍后当我们看到 OOP 示例时，这种区别会变得更加清晰。

让我们看看这如何改变被测代码。

```python
def parameter_dependent(value, sqrt_func):
    if value < 0:
        return 0
    if value <= 100:
        return sqrt_func(value)
    return 10
```

请注意 ```sqrt``` 函数现在如何成为输入参数。

例如，如果我们想在正常情况下使用 ```parameter_dependent``` 函数，我们将不得不产生依赖关系。

```python
import math
def test_good_dependency():
    assert parameter_dependent(25, math.sqrt) == 5
```

如果我们想要执行测试，我们可以通过将 ```math.sqrt``` 函数替换为特定函数，然后使用它来实现。例如：

```python
def test_twenty_five():
    def good_dependency(number):
        return 5
    assert parameter_dependent(25, good_dependency) == 5
```

例如，如果调用依赖项以确保在某些测试中不使用依赖项，我们也可能会引发错误。

```python
def test_negative():
    def bad_dependency(number):
        raise Exception('Function called')
    assert parameter_dependent(-1, bad_dependency) == 0
```

请注意这种方法比模拟更明确。本质上，要测试的代码变得完全正常，因为它没有外部依赖项。

### OOP 中的依赖注入

依赖注入也可以与 OOP 一起使用。在这种情况下，我们可以从这样的代码开始。

```python
class Writer:
    def __init__(self):
        self.path = settings.WRITER_PATH

    def write(self, filename, data):
        with open(self.path + filename, 'w') as fp:
            fp.write(data)


class Model:
    def __init__(self, data):
        self.data = data
        self.filename = settings.MODEL_FILE
        self.writer = Writer()

    def save(self):
        self.writer.write(self.filename, self.data)
```

正如我们所看到的，设置类存储了数据存储位置所需的不同元素。模型接收一些数据，然后将其保存。运行中的代码将需要最少的初始化。

```python
    model = Model('test')
    model.save()
```

模型接收一些数据，然后将其保存。运行中的代码需要最少的初始化，但同时它不是显式的。

要使用依赖注入原则，代码将需要以这种方式编写：

```python
class WriterInjection:
    def __init__(self, path):
        self.path = path

    def write(self, filename, data):
        with open(self.path + filename, 'w') as fp:
            fp.write(data)


class ModelInjection:
    def __init__(self, data, filename, writer):
        self.data = data
        self.filename = filename
        self.writer = writer

    def save(self):
        self.writer.write(self.filename, self.data)
```

在这种情况下，每个作为依赖项的值都是显式提供的。在代码的定义中，```settings```模块不存在于任何地方，而是在类被实例化时指定。代码现在需要直接定义配置。

```python
    writer = WriterInjection('./')
    model = ModelInjection('test', 'model_injection.txt', writer)
    model.save()
```

我们可以比较如何测试这两种情况，如文件 ```test_dependency_injection_test.py``` 所示。正如我们之前看到的，第一个测试是模拟 ```Writer``` 类的 ```write``` 方法，以断言它已被正确调用。

```python
@patch('class_injection.Writer.write')
def test_model(mock_write):
    model = Model('test_model')
    model.save()
    mock_write.assert_called_with('model.txt', 'test_model')
```

与此相比，依赖注入示例不需要通过猴子修补进行模拟。它只是创建自己的 ```Writer``` 来模拟界面。

```python
def test_modelinjection():
    EXPECTED_DATA = 'test_modelinjection'
    EXPECTED_FILENAME = 'model_injection.txt'
    class MockWriter:
        def write(self, filename, data):
            self.filename = filename
            self.data = data
    writer = MockWriter()
    model = ModelInjection(EXPECTED_DATA, EXPECTED_FILENAME,
                           writer)
    model.save()
    assert writer.data == EXPECTED_DATA
    assert writer.filename == EXPECTED_FILENAME
```


第二种风格更冗长，但它显示了以这种方式编写代码时的一些差异：

- 不需要monkey-patching模拟。猴子补丁可能非常脆弱，因为它会干预不应该暴露的内部代码。虽然在测试这种干扰时与对常规代码运行进行测试不同，但它仍然可能是混乱的并且会产生意想不到的影响，特别是如果内部代码以某种不可预见的方式发生变化。

    > 请记住，在某些时候，模拟可能会涉及与二级依赖关系相关的内容，这可能会开始产生奇怪或复杂的效果，需要你花时间处理这种额外的复杂性。

- 编写代码的方式本身就不同。正如我们所见，使用依赖注入生成的代码更加模块化，并且由更小的元素组成。这往往会创建更小、更可组合的模块，这些模块可以一起使用，并且具有更少的未知依赖项，因为它们总是显式的。

- 不过要小心，因为这需要一定的纪律和思维框架才能产生真正松散耦合的模块。如果在设计接口时不考虑这一点，则生成的代码将被人为地划分，从而导致不同模块之间的代码紧密耦合。发展这门学科需要一定的培训；不要指望它会自然而然地出现在所有开发人员身上。

- 代码有时可能更难调试，因为配置将与其余代码分开，有时难以理解代码的流程。复杂性可以在类的交互中产生，这可能更难以理解和测试。通常，以这种风格开发代码的前期工作量也会更大。

依赖注入是某些软件圈和编程语言中非常流行的技术。在动态性较低的语言中模拟比 Python 更困难，而且不同的编程语言对于如何构建代码有自己的一套想法。例如，依赖注入在 Java 中非常流行，其中有特定的工具可以以这种方式工作。

## 高级pytest

虽然我们已经描述了 pytest 的基本功能，但就它为帮助生成测试代码提供的可能性数量而言，我们几乎没有触及表面。

> Pytest 是一个大而全面的工具。值得学习如何使用它。在这里，我们只会触及表面。请务必查看 https://docs.pytest.org/ 上的官方文档。

在不详尽的情况下，我们将看到该工具的一些有用的可能性。

### 分组测试

有时将测试组合在一起是很有用的，这样它们就可以与特定的事物相关联，比如模块，或者统一运行它们。将测试组合在一起的最简单方法是将它们连接到一个类中。

例如，回到之前的测试示例，我们可以将测试构造成两个类，正如我们在 ```test_group_classes.py``` 中看到的那样。

```python
from tdd_example import parameter_tdd


class TestEdgesCases():
    def test_negative(self):
        assert parameter_tdd(-1) == 0
    def test_zero(self):
        assert parameter_tdd(0) == 0
    def test_ten(self):
        assert parameter_tdd(10) == 100
    def test_eleven(self):
        assert parameter_tdd(11) == 100
        
        
class TestRegularCases():
    def test_five(self):
        assert parameter_tdd(5) == 25
    def test_seven(self):
        assert parameter_tdd(7) == 49
```

这是划分测试并允许你独立运行它们的简单方法：

```sh
$ pytest -v test_group_classes.py
======================== test session starts =========================
platform darwin -- Python 3.9.5, pytest-6.2.4, py-1.10.0, pluggy-0.13.1 -- /usr/local/opt/python@3.9/bin/python3.9
collected 6 items
test_group_classes.py::TestEdgesCases::test_negative PASSED      [16%]
test_group_classes.py::TestEdgesCases::test_zero PASSED          [33%]
test_group_classes.py::TestEdgesCases::test_ten PASSED           [50%]
test_group_classes.py::TestEdgesCases::test_eleven PASSED        [66%]
test_group_classes.py::TestRegularCases::test_five PASSED        [83%]
test_group_classes.py::TestRegularCases::test_seven PASSED       [100%]
========================= 6 passed in 0.02s ==========================
$ pytest -k TestRegularCases -v test_group_classes.py
========================= test session starts ========================
platform darwin -- Python 3.9.5, pytest-6.2.4, py-1.10.0, pluggy-0.13.1 -- /usr/local/opt/python@3.9/bin/python3.9
collected 6 items / 4 deselected / 2 selected
test_group_classes.py::TestRegularCases::test_five PASSED        [50%]
test_group_classes.py::TestRegularCases::test_seven PASSED       [100%]
================== 2 passed, 4 deselected in 0.02s ===================
$ pytest -v test_group_classes.py::TestRegularCases
========================= test session starts ========================
platform darwin -- Python 3.9.5, pytest-6.2.4, py-1.10.0, pluggy-0.13.1 -- /usr/local/opt/python@3.9/bin/python3.9
cachedir: .pytest_cache
rootdir: /Users/jaime/Dropbox/Packt/architecture_book/chapter_09_testing_and_tdd/advanced_pytest
plugins: celery-4.4.7
collected 2 items
test_group_classes.py::TestRegularCases::test_five PASSED        [50%]
test_group_classes.py::TestRegularCases::test_seven PASSED       [100%]
========================== 2 passed in 0.02s =========================
```

另一种可能性是使用标记。标记是可以通过测试中的装饰器添加的指标，例如在 ```test_markers.py``` 中。

```python
import pytest
from tdd_example import parameter_tdd


@pytest.mark.edge
def test_negative():
    assert parameter_tdd(-1) == 0
    
@pytest.mark.edge
def test_zero():
    assert parameter_tdd(0) == 0
    
def test_five():
    assert parameter_tdd(5) == 25
    
def test_seven():
    assert parameter_tdd(7) == 49
    
@pytest.mark.edge
def test_ten():
    assert parameter_tdd(10) == 100
    
@pytest.mark.edge
def test_eleven():
    assert parameter_tdd(11) == 100
```

看到我们在所有检查值边缘的测试上定义了一个装饰器```@pytest.mark.edge```。

如果我们执行测试，我们可以使用参数 -m 只运行带有特定标签的测试。

```sh
$ pytest -m edge -v test_markers.py
========================= test session starts ========================
platform darwin -- Python 3.9.5, pytest-6.2.4, py-1.10.0, pluggy-0.13.1 -- /usr/local/opt/python@3.9/bin/python3.9
collected 6 items / 2 deselected / 4 selected
test_markers.py::test_negative PASSED                            [25%]
test_markers.py::test_zero PASSED                                [50%]
test_markers.py::test_ten PASSED                                 [75%]
test_markers.py::test_eleven PASSED                              [100%]
========================== warnings summary ==========================
test_markers.py:5
  test_markers.py:5: PytestUnknownMarkWarning: Unknown pytest.mark.edge - is this a typo?  You can register custom marks to avoid this warning - for details, see https://docs.pytest.org/en/stable/mark.html
    @pytest.mark.edge
test_markers.py:10
...
-- Docs: https://docs.pytest.org/en/stable/warnings.html
============ 4 passed, 2 deselected, 4 warnings in 0.02s =============
```

如果标记边缘未注册，则会产生警告 ```PytestUnknownMarkWarning: Unknown pytest.mark.edge``` 。

> 请注意，GitHub 代码包含 ```pytest.ini``` 代码。如果存在 ```pytest.ini``` 文件，例如，如果你克隆整个 repo，你将不会看到警告。

这对于查找拼写错误非常有用，例如不小心写了 ```egde``` 或类似内容。为避免此警告，你需要添加一个带有标记定义的``` pytest.ini ```配置文件，如下所示。

```ini
[pytest]
markers = edge: tests related to edges in intervals
```

现在，运行测试不会显示任何警告。

```sh
$ pytest -m edge -v test_markers.py
========================= test session starts =========================
platform darwin -- Python 3.9.5, pytest-6.2.4, py-1.10.0, pluggy-0.13.1 -- /usr/local/opt/python@3.9/bin/python3.9
cachedir: .pytest_cache
rootdir: /Users/jaime/Dropbox/Packt/architecture_book/chapter_09_testing_and_tdd/advanced_pytest, configfile: pytest.ini
plugins: celery-4.4.7
collected 6 items / 2 deselected / 4 selected
test_markers.py::test_negative PASSED                            [25%]
test_markers.py::test_zero PASSED                                [50%]
test_markers.py::test_ten PASSED                                 [75%]
test_markers.py::test_eleven PASSED                              [100%]
=================== 4 passed, 2 deselected in 0.02s ===================
```

请注意，标记可以在整个测试套件中使用，包括多个文件。这允许制作标记来识别测试中的常见模式，例如，创建一个快速测试套件，其中包含最重要的测试以使用标记基础运行。

还有一些带有一些内置功能的预定义标记。最常见的是skip（将跳过测试）和xfail（将反转测试，意味着它预计它会失败）。

### 使用夹具

使用夹具是在 pytest 中设置测试的首选方式。夹具本质上是为设置测试而创建的上下文。

夹具用作测试功能的输入，因此可以设置它们并为要创建的测试创建特定环境。

例如，让我们看一个计算字符串中字符出现次数的简单函数。

```python
def count_characters(char_to_count, string_to_count):
    number = 0
    for char in string_to_count:
        if char == char_to_count:
            number += 1
    return number
```

这是一个非常简单的循环，它遍历字符串并计算匹配的字符。

> 这等效于对字符串使用函数 .count()，但包含它是为了呈现一个工作函数。以后可以重构！

覆盖功能的常规测试如下。

```python
def test_counting():
    assert count_characters('a', 'Barbara Ann') == 3
```

很简单。现在让我们看看我们如何定义一个夹具来定义一个设置，以防我们想要复制它。

```python
import pytest
@pytest.fixture()
def prepare_string():
    # Setup the values to return
    prepared_string = 'Ba, ba, ba, Barbara Ann'
    # Return the value
    yield prepared_string
    # Teardown any value
    del prepared_string
```

首先，```fixture``` 用 ```pytest.fixture``` 装饰以将其标记为这样。一个夹具分为三个步骤：

- 设置：在这里，我们简单地定义了一个字符串，但这可能是最大的部分，其中准备了值。
- 返回值：如果我们使用yield功能，我们将能够进入下一步；如果没有，夹具将在这里完成。
- 拆除和清理值：这里，我们只是简单地删除变量作为示例，尽管这将在稍后自动发生。
    
    > 稍后，我们将看到更复杂的夹具。在这里，我们只是介绍这个概念。

以这种方式定义夹具将允许我们在不同的测试功能中轻松重用它，只需使用名称作为输入参数。

```python
def test_counting_fixture(prepare_string):
    assert count_characters('a', prepare_string) == 6


def test_counting_fixture2(prepare_string):
    assert count_characters('r', prepare_string) == 2
```

请注意 ```prepare_string``` 参数如何自动提供我们使用 ```yield``` 定义的值。如果我们运行测试，我们可以看到效果。更重要的是，我们可以使用参数 ```--setup-show``` 来查看设置并拆除所有固定装置。

```sh
$ pytest -v test_fixtures.py -k counting_fixture --setup-show
======================== test session starts ========================
platform darwin -- Python 3.9.5, pytest-6.2.4, py-1.10.0, pluggy-0.13.1 -- /usr/local/opt/python@3.9/bin/python3.9
plugins: celery-4.4.7
collected 3 items / 1 deselected / 2 selected
test_fixtures.py::test_counting_fixture
        SETUP    F prepare_string
        test_fixtures.py::test_counting_fixture (fixtures used: prepare_string)PASSED
        TEARDOWN F prepare_string
test_fixtures.py::test_counting_fixture2
        SETUP    F prepare_string
        test_fixtures.py::test_counting_fixture2 (fixtures used: prepare_string)PASSED
        TEARDOWN F prepare_string
=================== 2 passed, 1 deselected in 0.02s ===================
```

这个夹具非常简单，没有做任何定义字符串无法完成的事情，但夹具可用于连接到数据库或准备文件，考虑到它们可以在最后清理它们。

例如，将同一个示例稍微复杂一点，而不是从字符串计数，它应该从文件计数，因此该函数需要打开一个文件，读取它并计算字符数。功能将是这样的。

```python
def count_characters_from_file(char_to_count, file_to_count):
    '''
    Open a file and count the characters in the text contained
    in the file
    '''
    number = 0
    with open(file_to_count) as fp:
        for line in fp:
            for char in line:
                if char == char_to_count:
                    number += 1
    return number
```

然后，```fixture``` 应该创建一个文件，将其返回，然后将其作为拆卸的一部分删除。让我们来看看它。

```python
import os
import time
import pytest


@pytest.fixture()
def prepare_file():
    data = [
        'Ba, ba, ba, Barbara Ann',
        'Ba, ba, ba, Barbara Ann',
        'Barbara Ann',
        'take my hand',
    ]
    filename = f'./test_file_{time.time()}.txt'
    # Setup the values to return
    with open(filename, 'w') as fp:
        for line in data:
            fp.write(line)
    # Return the value
    yield filename
    # Delete the file as teardown
    os.remove(filename)
```

请注意，在文件名中，我们定义了在生成时间戳时添加时间戳的名称。这意味着将由该夹具生成的每个文件都是唯一的。

```python
filename = f'./test_file_{time.time()}.txt'
```

然后创建文件并写入数据。

```python
with open(filename, 'w') as fp:
    for line in data:
        fp.write(line)
```

文件名，正如我们所见，是唯一的，被生成了。最后，文件在拆解中被删除。

这些测试与之前的测试类似，因为大部分复杂性都存储在夹具中。

```python
def test_counting_fixture(prepare_file):
    assert count_characters_from_file('a', prepare_file) == 17
    
    
def test_counting_fixture2(prepare_file):
    assert count_characters_from_file('r', prepare_file) == 6
```

运行它时，我们看到它按预期工作，并且我们可以检查拆卸步骤是否在每次测试后删除了测试文件。

```sh
$ pytest -v test_fixtures2.py
========================= test session starts =========================
platform darwin -- Python 3.9.5, pytest-6.2.4, py-1.10.0, pluggy-0.13.1 -- /usr/local/opt/python@3.9/bin/python3.9
collected 2 items
test_fixtures2.py::test_counting_fixture PASSED                  [50%]
test_fixtures2.py::test_counting_fixture2 PASSED                 [100%]
========================== 2 passed in 0.02s ==========================
```

夹具不需要在同一个文件中定义。它们也可以存储在一个名为 conftest.py 的特殊文件中，该文件将由 pytest 在所有测试中自动共享。

> 夹具也可以组合，它们可以设置为自动使用，并且已经有内置的夹具可以处理时间数据和目录或捕获输出。 PyPI 中还有很多有用的固定装置插件，可作为第三方模块安装，涵盖连接数据库或与其他外部资源交互等功能。请务必检查 Pytest 文档并在实现自己的夹具之前进行搜索，以查看是否可以利用现有模块：https://docs.pytest.org/en/latest/explanation/fixtures.html#about-fixtures。

在本章中，我们只涉及 pytest 的可能性的皮毛。这是一个很棒的工具，我鼓励你学习。有效地运行测试并以最好的方式设计它们将带来巨大的回报。测试是项目的关键部分，也是开发人员花费大部分时间的开发阶段之一。

## 概括

在本章中，我们通过测试的原因和方式来描述如何需要一个好的测试策略来生产高质量的软件并在代码被客户使用后防止出现问题。

我们首先描述了测试背后的一般原则，如何制作比成本更有价值的测试，以及确保这一点的不同级别的测试。我们看到了三个主要级别的测试，我们称之为单元测试（单个组件的一部分）、系统测试（整个系统）和中间的集成测试（整个组件或几个组件，但不是全部）。

我们继续描述了不同的策略以确保我们的测试是出色的，以及如何使用 Arrange-Act-Assert 模式来构建它们，以便于编写和在编写后理解它们。

后来，我们详细描述了测试驱动开发背后的原则，这是一种将测试置于开发中心的技术，它要求在代码之前编写测试，以小增量工作，并一遍又一遍地运行测试以创建一个好的防止意外行为的测试套件。我们还分析了以 TDD 方式工作的限制和注意事项，并提供了流程的示例。

我们继续介绍在 Python 中创建单元测试的方法，既使用标准 unittest 模块，又介绍了更强大的 pytest。我们还介绍了 pytest 的高级用法部分，以展示这个出色的第三方模块的功能。

我们描述了如何测试外部依赖关系，这在编写单元测试以隔离功能时至关重要。我们还描述了如何模拟依赖关系以及如何在依赖注入原则下工作。