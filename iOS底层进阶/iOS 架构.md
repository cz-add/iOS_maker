## 
初见

### **MVC**

尽管开发人员争论应该使用哪种体系结构，但 Apple 已经向我们提供了有关如何构建 iOS 应用程序的说明，即 MVC。

![输入图片说明](https://images.gitee.com/uploads/images/2021/0527/155454_d3e1a642_9027123.png "架构1.png")


View 是用户可以在屏幕上看到的部分。Model 是“数据”。Controller 是它们之间的中介。它从 Model 获取数据并在 View 上显示给用户，同时在 View 上处理用户操作并将其传输到 Model。

看起来很好。如果遵循要 Apple 指南的话，为什么不使用 MVC 呢？因为乍一看，MVC 真的很糟糕。您可能知道，ViewController 的大小和维护难度。因为除了视图和数据外，还有很多不同的逻辑，这显然应该由 Controller 完成。

> *Controller 负责管理其拥有的视图的视图层次结构。他们响应视图的加载，出现，消失等等操作。他们还倾向于处理我们想脱离模型的模型逻辑以及我们想脱离视图的业务逻辑。这导致我们遇到的第一个问题是 Massive View Controller。*

### **MVVM**

我们并不喜欢这样上面这种方式，因此开始寻找 MVC 替代方案。并且我们找到了它们。

![输入图片说明](https://images.gitee.com/uploads/images/2021/0527/155518_33b8c92b_9027123.png "架构2.png")


MVVM 添加了一个新层 ViewModel 来将代码与 Controller 分开。但是实际上，它并不能解决所有问题。ViewModel 应该真正包含什么？当ViewModel 也变得像 Controller 一样臃肿时，我该怎么办？社区也因此分裂为喜欢 MVVM 的人和不喜欢 MVVM 的人。

### **MVP**

解决此问题的另一种尝试是 MVP。它开始将 ViewController 视为 View，所有逻辑都交给新类 Presenter。但是它并没有流行起来，因为它看起来真的很奇怪。实际上，我们只是将所有问题从 ViewController 转移到 Presenter。

![输入图片说明](https://images.gitee.com/uploads/images/2021/0527/155529_63d81303_9027123.png "架构3.png")


### **VIPER**

然后，我们认为我们需要进一步分解并创建了 VIPER。现在，所有代码都进入视图，演示者，路由器，交互器或实体之一。

![输入图片说明](https://images.gitee.com/uploads/images/2021/0527/155539_5a230477_9027123.png "架构4.png")


在很短的时间内，VIPER 变得流行起来，但是后来我们知道它有问题。这种体系结构需要大量协议，类以及层之间的数据传递。但是由于某些原因，所有这些额外的工作并不能使我们的设计更好，更易读。

### **其他架构**

最后，我们无休止的去创建新架构。所有这些看起来都是个笑话。每个新架构看起来都比以前的架构更奇怪。吉尔赫姆·兰博（Guilherme Rambo）讲过一个笑话，很好地描述了这种情况的荒谬性。无论选择哪种架构，所有架构都是不好的。

但是正如我之前所说，这个问题有解决方案。我会告诉你我们应该使用哪种“模式”。您可能会感到惊讶，但实际上就是 MVC。我想要做的是从头开始，从原始资料中阅读 MVC，然后停止使用它。如果它还活着，也许还不算坏？

## 原始的 MVC

许多 iOS 开发人员抱怨 MVC。但是，如果我告诉您，我前面提到的所有 MVC 问题实际上都不存在的呢？诸如“Massive View Controller”，“模型就是数据”，“ ViewController做很多业务逻辑”等观察都是虚构的。他们都是出于对真正的 MVC 的误解而产生的。

人们对此有疑问的主要原因是由于 MVC 的过于简化。说真的，当您听到 MVC 时，您会怎么想？“一共有 3 个类：Model 是数据，View 是视图，Controller 在它们之间”。但是，MVC 并不是那么简单。

MVC是一项非常艰巨的工作的结果。它是由 Trygve Reenskaug 于 1979 年在施乐 PARC 的 Dynabook 项目上的提出的。Dynabook 是适用于所有年龄段儿童的个人计算机。这是一个真正的革命性项目。Dynabook 旨在使计算机易于使用，同时使用户能够管理复杂的应用程序。那时，图形界面的基础和“用户友好界面”的概念首先得到了发展。

这个项目进行了大约十年。Reenskaug 总结了这十年在 MVC 中积累的 GUI 应用程序开发的主要思想和解决方案。

并没有像“嘿，我们在10年内创建了一种通用模式，您应该用它来解决任何问题”。这是我们犯的根本错误。MVC 不是模式。这不是应用程序模块分解的方案。没有人可以为您提供具有一定数量的类的灵丹妙药解决方案，因为没人知道您的问题，应用程序的业务逻辑，域模型详细信息和主要目标。您应该自己设计应用程序。以下是 Martin Fowler 描述 MVC 误解的问题：

> *它通常被称为模式，但是我认为将其视为一种模式并不是非常有用，因为它包含了许多不同的想法。在不同地方阅读 MVC 的人不同，他们的想法也不同，并将其描述为 “MVC”。如果这不会引起足够的混乱，那么您会得到对 MVC 的误解，这种误解是通过层层传递而来的。*

**MVC 是一组架构思想和原则**。MVC 是正式尝试将具有图形用户界面的应用程序中的主要思想形式化的尝试之一。这些想法仍然有意义，不仅适用于 iOS 平台。您可以从 Trygve Reenskaug 的作品中了解有关 MVC 的信息。*The original MVC reports*1 与 *The Model-View-Controller. It's Past and Present*2。

MVC 的主要原则之一是将我们所有的代码划分为 Presentation 和 Domain Model。

Domain Model 是我们应用程序的核心。这是它的主要部分。它由几个业务对象组成，例如，诸如帐户，产品，交易等实体。这些对象相关的逻辑称为业务逻辑。例如，“如果用户帐户上的钱很少，请给他折扣”。MVC 中的模型意味着整个 Domain Model，而不仅仅是某个实体的一个哑模型（dumb model）。Domain Model 可以包含一个对象，也可以包含整个对象系统。这取决于业务逻辑的复杂程度。

Presentation 是用户可以看到并与之交互的内容。在 MVC 中，View 和 Controller 是 Presentation 的一部分。

马丁·福勒（Martin Fowler）将此原则称为*“Separated Presentation”*3。

> *MVC 的核心，也是对后来的框架最有影响力的想法，就是我所说的“分离表示”。分离演示的背后思想是在建模我们对现实世界的感知的领域对象和作为屏幕上看到的 GUI 元素的演示对象之间进行清晰的划分。领域对象应该完全独立并且可以在不引用 presentation 的情况下工作，它们还应该能够支持多个 presentation（可能同时支持）。这种方法也是 Unix 文化的重要组成部分，并且一直持续到今天，允许通过图形界面和命令行界面来操纵许多应用程序。*

因此，如果我们的 Presentation 与 Domain Model 松散耦合的，并且无需任何 Domain Model 的详细信息，同时 Domain Model 完全独立于Presentation，则我们应用的设计将是清晰，可重用和可维护的。

该方案取自 Reenskaug 的报告。

![输入图片说明](https://images.gitee.com/uploads/images/2021/0527/155552_1a8e0fd7_9027123.png "架构5.png")


其中的 Editor 是 Presentation 的最初表述。在此方案中，我们可以看到 MVC 不是 3 个部分。它更多地是关于按层而不是按类进行分解。重要的是，Presentation 应与 Domain Model 非常松散地耦合。理想情况下，它应该仅取决于所需的接口，以便任何 Domain Model 都可以实现此接口。该方案的 Facade 模式表明，Domain Model 中有一个类可以通过调用所需对象来实现此接口，因此 Presentation 不需要了解有关域模型中具体对象的任何知识。接口和外观帮助我们使 Presentation 和 Domain Model 之间的连接松散耦合。

但是 Domain Model 应该如何与 Presentation 通信？例如，如果某些数据在“Domain Model”中发生了更改，则应如何通知 Presentation？这是 MVC 的另一个原理。Domain Model 永远不应该依赖于Presentation，即使是通过接口也是如此。Domain Model 所能做的就是发送有关某个事件的通知，而不知道谁将处理此事件。可以通过观察者模式来完成。这将使我们完全独立于域模型。

Reenskaug 报告的另一种方案描述了 MVC 的第三项原则。

![输入图片说明](https://images.gitee.com/uploads/images/2021/0527/155603_ab630d33_9027123.png "架构6.png")


这是关于 Input 和 Output 的分离表示。最初，将 Presentation 分为负责向用户显示信息的层和负责从用户获取信息的层是一个很好的主意。稍后您将看到，该原理不适用于 iOS。但是您应该知道，在原始 MVC 中， Controller 和 View 都具有图形表示。

总而言之，原始 MVC 应该看起来像这样：

![输入图片说明](https://images.gitee.com/uploads/images/2021/0527/155613_c8d1aa4e_9027123.png "架构7.png")


## 这适用于iOS吗？

当然可以！如果我们将 MVC 视为一组原则，而不仅仅是一个“具有 3 种类的模式”，我们将永远不会知道 “Massive View Controller” 问题。让我们看看这些原理如何适用于iOS。

如前所述，**MVC 的核心是 Presentation 和 Domain Model 之间的强分离**。实际上，该原理已成为 GUI 应用程序设计中的主要原理之一。诸如 MVVM 或 MVP 之类的其他体系结构也基于这种分离。无论您针对哪个平台编写代码，使用哪种体系结构，都应始终进行这种分离。因此，这意味着该原则对 iOS 也很重要。

如何将视图划分为 View 和 Controller？通常，它也适用于 iOS，甚至包含 UIView 和 UIViewController 的 iOS SDK。但是我们应该知道，这种分离与原始 MVC 有一些区别。这并不奇怪，因为经过这么长的时间，用户界面也发生了变化。现在，我们不需要在输入和输出上划分图形元素。特别是在 iOS 上，每个 UIView 元素都能够显示信息并接收用户操作。因此，UIView 是一个类，具有图形表示形式，并负责与用户双向交流。UIViewController 是 UIView 的所有者。它“控制” View 及其生命周期，在 View 上处理用户操作，并在 View 上显示 Model 中的信息。

![输入图片说明](https://images.gitee.com/uploads/images/2021/0527/155625_64cd58b7_9027123.png "架构8.png")


原始 MVC 的这种变体具有不同的名称，稍后我们将看到它，但是无论如何，我们将其称为 MVC，因为保留了主要原理，并且仅仅是 MVC 的变体。此外，苹果公司本身称之为 MVC。

实际上，我们如何称呼它并不重要。重要的是要了解它是如何实现的。更确切地说，要意识到已经实现了 MVC。UIView 和 UIViewController 是已经在 iOS SDK 中实现的类。我的意思是，有些人拒绝 MVC，但仍使用 UIView 和 UIViewController。尽管这是主要问题，但它使 Apple MVC 与其他体系结构有所不同。这是我们如何处理用户交互的一种方式，而诸如 Interactor 或 Presenter 之类的其他类则不会更改这种方式。相反，MVC 在必要时根据问题涉及其他实体。尽管 Interactor 和 Presenter 都是不好的类的示例，但我们应该记住 MVC 并不是一种模式，可以根据需要提供许多类来解决问题。因此，如果您在用户面前使用 UIView 和 UIViewController，则不必介意创建其他哪些类，您可以使用 Apple MVC。

我们能不使用 UIView 和 UIViewController 吗？可以！许多工作在后台进行，因此我们可以轻松地通过我们的应用程序处理用户的所有通信。除了这两个类之外，还有很多其他东西：响应者链，UIEvent，UIView 层次结构，UIView 生命周期，Hit Testing，UIControls，UIGestureRecognizers 等。所有这些都是 Apple MVC。这意味着 MVC 不是我们的选择。如果您说自己不使用 MVC，然而事实并非如此！我们使用了 MVC，并且在 iOS 中不能使用任何替代方法。

使用 iOS SDK 进行战斗是不可能的，任何尝试都会使系统复杂化。但是，如果我们了解这一点，就还不错。一旦我们停止与 iOS SDK 的对抗，所有这些东西就会变得有用。SDK 开始帮助我们并从中受益。每个 UIViewController 都拥有一个根 UIView。我们可以在 interface builder 中绘制视图而无需任何代码，并将所有用户操作链接到UIViewController。UIViewController 还通过诸如viewDidLoad()，viewWillAppear() 等方法处理 View 的状态。我们应该使用所有这些功能。

iOS SDK 为我们提供了许多功能。许多开发人员抱怨 UIViewControllers 变胖了，但其中只有一小部分提到了 UIViewControllers 分解功能。因此，对于许多开发人员而言，它可能会让人感到惊讶。但是我们可以为 1 个页面创建多个 UIViewControllers。是的，如果一个屏幕上有多个逻辑上独立的组件，我们可以将其分为多个小 UIViewControllers。

• UIViewController 是表示层的一部分。如果您在此处编写业务逻辑，网络请求或其他与用户界面无关的内容，则不是 MVC。

• 如果需要，在表示层中创建其他类。IViewController 的存在并不会迫使您在此处编写所有代码。如果您有很多表示逻辑，请从 ViewController 中删除它。但是请确保确实需要新实体。

• 不要与 iOS SDK 抗争。它为我们提供了许多功能，如果我们开始使用它们，这些功能将带来巨大的好处。

## 我们需要MVC替代品吗？

好吧，答案很明显：我们不需要。您已经了解了什么是真正的 MVC，以及如何在 iOS 中使用它。此外，使用自己的体系结构与 iOS 平台抗衡几乎是不可能的。但是，让我们再次考虑一下我们在开始时描述的每种架构，您会发现它们在 iOS 环境中是多么的奇怪甚至荒谬。

### **MVP**

MVP 是其中最奇怪的一个。MVP 由 Mike Potel 于 1996 年推出，是对 MVC 的修改。在有关 MVP 的工作中，Potel 建议无需将小部件划分为“视图”和“控制器”。现代操作系统的用户界面已经在 View 类中提供了大多数 Controller 功能，因此 Controller 似乎有点多余。因此，删除了 Controller 并创建了一个新类 Presenter 作为 View 和 Model 之间的粘合剂。

等等，看起来像 Apple MVC 吗？也许它就是 Apple MVC？苹果原本想说是 MVP，却说成了 MVC？我不知道，因为这些术语之间有太多混淆。让我们看看 Martin Fowler 在有关 GUI 体系结构的文章中如何区分 MVC 和 MVP。

> *MVP使用 Supervising Controller 来操纵模型。小部件将用户手势传递给 Supervising Controller。小部件未分为视图和控制器。您可以将 presenters 看作是控制器，但无需最初处理用户手势。但是，还需要注意的是，presenters 通常是在表单级别，而不是在小部件级别 -– 这也许是更大的区别。*

![输入图片说明](https://images.gitee.com/uploads/images/2021/0527/155637_50752516_9027123.png "架构9.png")


现在，看看 MVC 和 MVP 的这些方案。并将它们与我们上面看到的 Apple MVC 方案进行比较。其中哪一个与 Apple MVC 更相似？是的，Apple MVC 看起来更像是 MVP，而不是原始的 MVC。我们如何称呼它并不重要。Apple MVC 无论如何都与它们两者不同。



最重要的是要了解我们已经拥有充当 UIView 持有者的 UIViewController。这意味着我们不需要具有 Presenter 或 Controller 角色的其他任何类。

因此，尝试创建一个新的 Presenter 类并将 UIViewController 视为一个视图是没有意义的。尽管我说过，除了 UIView 和 UIViewController 之外，Presentation 层中可能还有其他类，但是 Presenter 是这样做的一个不好的例子。这是与 iOS SDK 对抗的一个示例。无论我们是否希望将 UIViewController 视为 View 多少，它仍然是Controller（或者您可以将其称为 Presenter）。在 iOS 中，MVP 方案实际上如下所示：

![输入图片说明](https://images.gitee.com/uploads/images/2021/0527/155644_8e38853e_9027123.png "架构10.png")


我们真的需要这个新类吗？这看起来很奇怪，因为我们只是创建了具有完全相同角色的 UIViewController 的副本。如果没有给我们带来任何收益，我们为什么应该转移所有用户操作，将所有视图状态从 Controller 更改为 Presenter？它只会给我们带来额外的代码和复杂性。确实很难将每个动作委派给 Presenter。同样，不要与 iOS SDK 对抗，我们无法将 UIViewController 转换为 View。即使可以，也没有必要。

### **VIPER**

还记得我说过 MVP 是最奇怪的吗？不，VIPER 才是。因为，除了 MVP 的所有问题（它还会重复 Presentation 层中 MVP 的所有错误，包括复制 Presenter 以及将 UIViewController 转换为 View 的尝试失败），VIPER 还试图将我们的 Domain Model 划分为 Interactor，Service 和 Entity 类。

VIPER 是如何被创建的？是否有自己的历史记录，例如 MVC 或 MVP？是的，的确如此，但是历史并不那么光鲜。VIPER 于 2013 年创建，旨在解决 Apple MVC 问题。

> *由于许多应用程序逻辑不属于模型或视图，因此通常会在控制器中处理。这导致了一个称为 Massive View Controller 的问题，在该问题中，视图控制器最终会做太多事情。*

以上引用来自有关 VIPER 的原始帖子。这意味着 VIPER 的创建是为了解决不存在的 “Massive View Controller” 问题。它是基于 “MVC是具有3种类和巨大的UIViewController的模式”的错误思想而创建的。为了解决这个“问题”，VIPER 按 5 类进行了更多分解。但是实际上，您的“架构”有多少个字母并不重要。如果您仅将应用程序体系结构视为具有确切类的“模式”，则无论如何都会失败。类的数量是有限的，但是每次的逻辑可能会更宽，并且有朝一日我们的 Interactor 或另一个类将与 Massive View Controller 一样大。

而且，逻辑可能真的不同。但是在 VIPER 中，即使逻辑很小或非常具体，我们也总是创建 5 个类。问题确实有所不同，并且没有适合所有问题的方案。我们应该根据此特定逻辑单独进行分解。在 OOP 中，常见的任务是了解我们应该创建哪些实体，如何将它们彼此关联以及如何命名它们，从而以最清楚地描述代码。

克里斯·艾德霍夫（Chris Eidhof）在《App Architecture》一书中对 VIPER 问题和分解有很多想法。

> *虽然接口分解是一种管理代码大小的有效方法，但我们认为应该按需执行，而不是有条不紊地针对每个视图控制器执行。分解应该与所涉及的数据和任务的知识一起执行，以便可以实现最佳的抽象，从而可以最大程度地降低复杂性。*

Interactor 是否有这么好的抽象性？答案是否定的。“Interactor 是包含业务逻辑的类”。这有助于我们理解代码吗？它包含哪些业务逻辑？如果我有很多业务逻辑怎么办？我们应该创建并命名我们的实体，使其清晰明确，而不仅仅是通用的“Interactor”。

为所有问题创建相同的类，并且每次仅将代码添加到这些类中并不是一个好的设计。它甚至都不是 OOP，我认为这是具有 5 个文件的过程编程。

我认为，VIPER 是一个很大的错误。VIPER 证明我们还不了解 MVC。我的建议是忘记 VIPER，不要讨论它。

### **MVVM**

如果我们不使用 UIViewController 编写业务逻辑并使用分解将一个屏幕划分为多个 UIViewControllers，那么我们的 UIViewControllers 永远不会变得很大吗？好吧，这取决于它具有多少表示逻辑。

我们来看一个例子。

```
// Domain Model Object
struct Person {
    let name: String
    let gender: Gender
}
class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        // presentation logic
        if person.gender == .male {
            nameLabel.text = "Mr." + person.name
        } else {
            nameLabel.text = "Mrs. " + person.name
        }
    }
}
```

在上面的示例中，我们有一个表示逻辑，可以根据人的性别来格式化其姓名标题。这个逻辑应该在 UIViewController 中吗？如果存在很多复杂的表示逻辑怎么办？除了复杂性之外，还存在测试问题。测试 UIViewController 类并不容易。这也是开发人员创建自己的 Presenter 并将所有逻辑移至这个 NSObject 子类的另一个原因。但是我们已经看到了这种方法的问题。

我们可以在 Person 类中编写此逻辑吗？好了，在这种情况下，我们将根据 MVC 原理将表示和业务逻辑混合在一个不好的类中。很难理解为什么有此代码。我们看不到该代码是针对哪个具体视图编写的。最后，很难在不同的屏幕上重用此模型。如果在其他页面上以不同方式显示此信息（例如表情符号）怎么办？

现在，该再次重申 MVC 不是模式。是的，我们在 Presentation 层中有一些逻辑，MVC 不会强迫您在现有的类中编写此逻辑。我们可以创建一个新类并在那里封装具体逻辑。马丁·福勒（Martin Fowler）写了这个问题。他说，如果与 Domain Model 对象不同，我们可以在 Presentation 层中创建其他模型。他称其为“对象表示模型(Presentation Model)”。

```
struct PersonPresentation {
    let name: String
    init(person: Person) {
        if person.gender == .male {
            name = “Mr.” + person.name
        } else {
            name = “Mrs. “ + person.name
        }
    }
}
```

现在，在 UIViewController 中，我们仅将名称映射到标签。 您可能会说它是 MVVM。但事实并非如此。尽管我们将其称为 MVVM，但事实并非如此。因为实际上，MVVM 于 2005 年创建，是对 MVC 和 MVP 的修改。添加了 ViewModel 而不是 Controller 和 Presenter，它可以处理 View 操作。但是在 iOS 中，我们仍然没有摆脱 Controller。UIViewController 处理我们与用户交互的方式。我们要做的就是在 Presentation 层中创建一个额外的模型，这在 MVC 中是隐含的。我们只是使用了一个 Presentation Model。

但是同样，命名并不是一个大问题。当然，我们不会将所有 ViewModels 重命名为 PresentationModels。但是，我们应该了解 ViewModel 的真正含义（为避免混淆，现在将其称为 PresentationModel）。

PresentationModel 不是包含所有业务逻辑的类，很多开发人员都这么说。PresentationModel 不是将网络请求，数据库请求，缓存等组合在一起的外观。它只是 Presentation 层中的模型。使用 PresentationModel 并不意味着我们使用另一种架构。我们仍然使用 MVC，因为我们不会更改与用户交互的方式。

通常，PresentationModel 只是一种模式。是的，与 MVC 或原始 MVVM 不同，Presentation Model 是一种在确实需要时使用的模式。无需进行标准化，也无需无故在每个模块上创建 PresentationModel。

## 结论

MVC 不像具有 3 种类的方案那么简单。MVC 不是模式，而是一组架构思想和原则。

这些原则的核心是 Presentation 和 Domain Model 之间的强分离。MVC 中的模型表示整个域模型。UIViewController 是 Presentation 的一部分。这意味着 MVC 不允许我们创建一个哑实体并将所有业务逻辑移至 UIViewController。

这种分离已成为 GUI 应用程序设计中的主要分离之一，它们对 iOS 也很有用。但是表示层分离通常是特定于平台的。iOS SDK 已经完成了大量工作，因此我们可以轻松地通过我们的应用程序处理用户的所有交流。因此，MVC 不是我们的选择，我们无法更改与用户交互的工作方式。我们不应该与平台对抗，因为我们的设计会很复杂。但是，一旦我们停止与 iOS SDK 对抗，所有这些人员就会变得有用。

除了根据业务逻辑设计域模型外，我们还可以根据表示逻辑设计表示。MVC 不会强迫我们在 UIViewController 中编写所有代码。如果需要，我们可以在 Presentation 层中创建其他类。但是，如果您添加 ViewModel 或 Coordinator 或其他名称，则不要将其称为新架构。

请勿尝试将架构标准化为模式。根据特定的逻辑分别进行分解，以试图清楚地描述代码。

不要责怪 MVC。

#推荐
[iOS架构模式视频学习](https://www.bilibili.com/video/BV1wA411H7jB)

#最后
如果您觉得还不错，麻烦在文末 “点个赞” 或者 评论 “Mark”，谢谢您的支持
