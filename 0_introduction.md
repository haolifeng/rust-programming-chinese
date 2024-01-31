# 介绍

Welcome to The Rust Programming Language, an introductory book about Rust. The Rust programming language helps you write faster, more reliable software. High-level ergonomics and low-level control are often at odds in programming language design; Rust challenges that conflict. Through balancing powerful technical capacity and a great developer experience, Rust gives you the option to control low-level details (such as memory usage) without all the hassle traditionally associated with such control.

欢迎阅读 Rust 编程语言，这是一本关于 Rust 的介绍性书籍。 Rust 编程语言可帮助您编写更快、更可靠的软件。 在编程语言设计中，高级人体工程学和低级控制常常是不一致的； Rust 挑战了这种冲突。 通过平衡强大的技术能力和出色的开发人员体验，Rust 为您提供了控制低级细节（例如内存使用）的选项，而无需传统上与此类控制相关的所有麻烦。

## Who Rust is For

Rust is ideal for many people for a variety of reasons. Let’s look at a few of the most important groups.

由于多种原因，Rust 对许多人来说是理想的选择。 让我们看看几个最重要的群体。
## Teams of Developers

Rust is proving to be a productive tool for collaborating among large teams of developers with varying levels of systems programming knowledge. Low-level code is prone to various subtle bugs, which in most other languages can be caught only through extensive testing and careful code review by experienced developers. In Rust, the compiler plays a gatekeeper role by refusing to compile code with these elusive bugs, including concurrency bugs. By working alongside the compiler, the team can spend their time focusing on the program’s logic rather than chasing down bugs.

Rust 被证明是一种高效的工具，可用于在具有不同系统编程知识水平的大型开发团队之间进行协作。 低级代码很容易出现各种细微的错误，而在大多数其他语言中，这些错误只能通过经验丰富的开发人员进行广泛的测试和仔细的代码审查来发现。 在 Rust 中，编译器扮演着看门人的角色，拒绝编译带有这些难以捉摸的错误（包括并发错误）的代码。 通过与编译器一起工作，团队可以将时间集中在程序的逻辑上，而不是追查错误。

Rust also brings contemporary developer tools to the systems programming world:

+ Cargo, the included dependency manager and build tool, makes adding, compiling, and managing dependencies painless and consistent across the Rust ecosystem.
+ The Rustfmt formatting tool ensures a consistent coding style across developers.
+ The Rust Language Server powers Integrated Development Environment (IDE) integration for code completion and inline error messages.

By using these and other tools in the Rust ecosystem, developers can be productive while writing systems-level code.

Rust 还为系统编程世界带来了当代的开发工具：

+ Cargo 是随附的依赖项管理器和构建工具，使添加、编译和管理依赖项变得轻松且在整个 Rust 生态系统中保持一致。
+ Rustfmt 格式化工具可确保开发人员之间保持一致的编码风格。
+ Rust 语言服务器支持集成开发环境 (IDE) 集成，以实现代码完成和内联错误消息。

通过使用 Rust 生态系统中的这些工具和其他工具，开发人员可以在编写系统级代码时提高工作效率。

## Students
Rust is for students and those who are interested in learning about systems concepts. Using Rust, many people have learned about topics like operating systems development. The community is very welcoming and happy to answer student questions. Through efforts such as this book, the Rust teams want to make systems concepts more accessible to more people, especially those new to programming.

Rust 适合学生和那些有兴趣学习系统概念的人。 使用 Rust，许多人了解了操作系统开发等主题。 社区非常欢迎并乐意回答学生的问题。 通过本书等努力，Rust 团队希望让更多人，尤其是那些刚接触编程的人更容易理解系统概念。

## companies
Hundreds of companies, large and small, use Rust in production for a variety of tasks, including command line tools, web services, DevOps tooling, embedded devices, audio and video analysis and transcoding, cryptocurrencies, bioinformatics, search engines, Internet of Things applications, machine learning, and even major parts of the Firefox web browser.

数百家大大小小的公司在生产中使用 Rust 来执行各种任务，包括命令行工具、Web 服务、DevOps 工具、嵌入式设备、音频和视频分析和转码、加密货币、生物信息学、搜索引擎、物联网应用程序 、机器学习，甚至是 Firefox 网络浏览器的主要部分。

## Open Source Developers
Rust 适合那些想要构建 Rust 编程语言、社区、开发工具和库的人。 我们很高兴您为 Rust 语言做出贡献。

## People Who Value Speed and Stability

Rust is for people who crave speed and stability in a language. By speed, we mean both how quickly Rust code can run and the speed at which Rust lets you write programs. The Rust compiler’s checks ensure stability through feature additions and refactoring. This is in contrast to the brittle legacy code in languages without these checks, which developers are often afraid to modify. By striving for zero-cost abstractions, higher-level features that compile to lower-level code as fast as code written manually, Rust endeavors to make safe code be fast code as well.

Rust 适合那些渴望语言速度和稳定性的人。 所谓速度，我们指的是 Rust 代码运行的速度以及 Rust 允许您编写程序的速度。 Rust 编译器的检查通过添加功能和重构来确保稳定性。 这与没有这些检查的语言中脆弱的遗留代码形成鲜明对比，开发人员通常不敢修改这些代码。 通过努力实现零成本抽象、与手动编写代码一样快地编译为较低级别代码的高级功能，Rust 致力于使安全代码也成为快速代码。

The Rust language hopes to support many other users as well; those mentioned here are merely some of the biggest stakeholders. Overall, Rust’s greatest ambition is to eliminate the trade-offs that programmers have accepted for decades by providing safety and productivity, speed and ergonomics. Give Rust a try and see if its choices work for you.



Rust 语言也希望支持许多其他用户； 这里提到的只是一些最大的利益相关者。 总的来说，Rust 最大的野心是通过提供安全性和生产力、速度和人体工程学来消除程序员几十年来所接受的权衡。 尝试一下 Rust，看看它的选择是否适合您。

## Who This Book is For
本书假设您已经用另一种编程语言编写了代码，但没有对使用哪种语言做出任何假设。 我们试图让来自各种编程背景的人都能广泛地访问这些材料。 我们不会花很多时间讨论什么是编程或如何思考它。 如果您对编程完全陌生，那么阅读一本专门介绍编程的书会更好。

## How to Use this Book

In general, this book assumes that you’re reading it in sequence from front to back. Later chapters build on concepts in earlier chapters, and earlier chapters might not delve into details on a particular topic but will revisit the topic in a later chapter.

You’ll find two kinds of chapters in this book: concept chapters and project chapters. In concept chapters, you’ll learn about an aspect of Rust. In project chapters, we’ll build small programs together, applying what you’ve learned so far. Chapters 2, 12, and 20 are project chapters; the rest are concept chapters.

Chapter 1 explains how to install Rust, how to write a “Hello, world!” program, and how to use Cargo, Rust’s package manager and build tool. Chapter 2 is a hands-on introduction to writing a program in Rust, having you build up a number guessing game. Here we cover concepts at a high level, and later chapters will provide additional detail. If you want to get your hands dirty right away, Chapter 2 is the place for that. Chapter 3 covers Rust features that are similar to those of other programming languages, and in Chapter 4 you’ll learn about Rust’s ownership system. If you’re a particularly meticulous learner who prefers to learn every detail before moving on to the next, you might want to skip Chapter 2 and go straight to Chapter 3, returning to Chapter 2 when you’d like to work on a project applying the details you’ve learned.

Chapter 5 discusses structs and methods, and Chapter 6 covers enums, match expressions, and the if let control flow construct. You’ll use structs and enums to make custom types in Rust.

In Chapter 7, you’ll learn about Rust’s module system and about privacy rules for organizing your code and its public Application Programming Interface (API). Chapter 8 discusses some common collection data structures that the standard library provides, such as vectors, strings, and hash maps. Chapter 9 explores Rust’s error-handling philosophy and techniques.

Chapter 10 digs into generics, traits, and lifetimes, which give you the power to define code that applies to multiple types. Chapter 11 is all about testing, which even with Rust’s safety guarantees is necessary to ensure your program’s logic is correct. In Chapter 12, we’ll build our own implementation of a subset of functionality from the grep command line tool that searches for text within files. For this, we’ll use many of the concepts we discussed in the previous chapters.

Chapter 13 explores closures and iterators: features of Rust that come from functional programming languages. In Chapter 14, we’ll examine Cargo in more depth and talk about best practices for sharing your libraries with others. Chapter 15 discusses smart pointers that the standard library provides and the traits that enable their functionality.

In Chapter 16, we’ll walk through different models of concurrent programming and talk about how Rust helps you to program in multiple threads fearlessly. Chapter 17 looks at how Rust idioms compare to object-oriented programming principles you might be familiar with.

Chapter 18 is a reference on patterns and pattern matching, which are powerful ways of expressing ideas throughout Rust programs. Chapter 19 contains a smorgasbord of advanced topics of interest, including unsafe Rust, macros, and more about lifetimes, traits, types, functions, and closures.

In Chapter 20, we’ll complete a project in which we’ll implement a low-level multithreaded web server!

Finally, some appendices contain useful information about the language in a more reference-like format. Appendix A covers Rust’s keywords, Appendix B covers Rust’s operators and symbols, Appendix C covers derivable traits provided by the standard library, Appendix D covers some useful development tools, and Appendix E explains Rust editions. In Appendix F, you can find translations of the book, and in Appendix G we’ll cover how Rust is made and what nightly Rust is.

There is no wrong way to read this book: if you want to skip ahead, go for it! You might have to jump back to earlier chapters if you experience any confusion. But do whatever works for you.


An important part of the process of learning Rust is learning how to read the error messages the compiler displays: these will guide you toward working code. As such, we’ll provide many examples that don’t compile along with the error message the compiler will show you in each situation. Know that if you enter and run a random example, it may not compile! Make sure you read the surrounding text to see whether the example you’re trying to run is meant to error. Ferris will also help you distinguish code that isn’t meant to work:

一般来说，本书假设您从前到后按顺序阅读。 后面的章节建立在前面章节中的概念的基础上，前面的章节可能不会深入研究特定主题的细节，但会在后面的章节中重新讨论该主题。

本书中有两种章节：概念章节和项目章节。 在概念章节中，您将了解 Rust 的一个方面。 在项目章节中，我们将应用您迄今为止所学到的知识一起构建小程序。 第2章、第12章和第20章是项目章节； 其余的是概念章节。

第 1 章解释了如何安装 Rust，如何编写“Hello, world!” 程序，以及如何使用 Cargo、Rust 的包管理器和构建工具。 第 2 章是关于用 Rust 编写程序的实践介绍，让您构建一个猜数字游戏。 在这里，我们概括地介绍了概念，后面的章节将提供更多细节。 如果您想立即动手，请阅读第 2 章。 第 3 章介绍了与其他编程语言类似的 Rust 功能，在第 4 章中，您将了解 Rust 的所有权系统。 如果您是一个特别细心的学习者，喜欢在继续下一章之前学习每个细节，您可能想跳过第 2 章并直接进入第 3 章，当您想要从事应用程序的项目时返回第 2 章 你所学到的细节。

第 5 章讨论结构和方法，第 6 章介绍枚举、匹配表达式和 if let 控制流构造。 您将使用结构和枚举在 Rust 中创建自定义类型。

在第 7 章中，您将了解 Rust 的模块系统以及组织代码及其公共应用程序编程接口（API）的隐私规则。 第8章讨论标准库提供的一些常见的集合数据结构，例如向量、字符串和散列映射。 第 9 章探讨 Rust 的错误处理理念和技术。

第 10 章深入探讨了泛型、特征和生命周期，它们使您能够定义适用于多种类型的代码。 第 11 章是关于测试的，即使有 Rust 的安全保证，测试也是确保程序逻辑正确所必需的。 在第 12 章中，我们将构建我们自己的 grep 命令行工具的功能子集的实现，该工具在文件中搜索文本。 为此，我们将使用前面章节中讨论的许多概念。

第 13 章探讨了闭包和迭代器：来自函数式编程语言的 Rust 特性。 在第 14 章中，我们将更深入地研究 Cargo，并讨论与他人共享库的最佳实践。 第 15 章讨论标准库提供的智能指针以及实现其功能的特征。

在第 16 章中，我们将介绍并发编程的不同模型，并讨论 Rust 如何帮助您无所畏惧地在多线程中进行编程。 第 17 章着眼于 Rust 习惯用法与您可能熟悉的面向对象编程原理的比较。

第 18 章是关于模式和模式匹配的参考，它们是在 Rust 程序中表达想法的强大方式。 第 19 章包含了一系列令人感兴趣的高级主题，包括不安全的 Rust、宏以及更多关于生命周期、特征、类型、函数和闭包的内容。

在第 20 章中，我们将完成一个项目，在该项目中我们将实现一个低级多线程 Web 服务器！

最后，一些附录以更像参考的格式包含有关该语言的有用信息。 附录 A 涵盖 Rust 的关键字，附录 B 涵盖 Rust 的运算符和符号，附录 C 涵盖标准库提供的可派生特征，附录 D 涵盖一些有用的开发工具，附录 E 解释 Rust 版本。 在附录 F 中，您可以找到本书的翻译，在附录 G 中，我们将介绍 Rust 的制作方法以及夜间 Rust 是什么。

阅读这本书没有错误的方法：如果你想跳过，就直接跳过去吧！ 如果您遇到任何困惑，您可能必须跳回到前面的章节。 但做任何对你有用的事情。


学习 Rust 过程的一个重要部分是学习如何阅读编译器显示的错误消息：这些消息将指导您编写工作代码。 因此，我们将提供许多无法编译的示例以及编译器在每种情况下向您显示的错误消息。 请注意，如果您输入并运行随机示例，它可能无法编译！ 请务必阅读周围的文本，看看您尝试运行的示例是否会出错。 Ferris 还将帮助您区分不起作用的代码：