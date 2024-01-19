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