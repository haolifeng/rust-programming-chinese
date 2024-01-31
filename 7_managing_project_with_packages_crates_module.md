# 7. 使用包，箱和模块管理项目
As you write large programs, organizing your code will become increasingly important. By grouping related functionality and separating code with distinct features, you’ll clarify where to find code that implements a particular feature and where to go to change how a feature works.

The programs we’ve written so far have been in one module in one file. As a project grows, you should organize code by splitting it into multiple modules and then multiple files. A package can contain multiple binary crates and optionally one library crate. As a package grows, you can extract parts into separate crates that become external dependencies. This chapter covers all these techniques. For very large projects comprising a set of interrelated packages that evolve together, Cargo provides workspaces, which we’ll cover in the “Cargo Workspaces” section in Chapter 14.

We’ll also discuss encapsulating implementation details, which lets you reuse code at a higher level: once you’ve implemented an operation, other code can call your code via its public interface without having to know how the implementation works. The way you write code defines which parts are public for other code to use and which parts are private implementation details that you reserve the right to change. This is another way to limit the amount of detail you have to keep in your head.

A related concept is scope: the nested context in which code is written has a set of names that are defined as “in scope.” When reading, writing, and compiling code, programmers and compilers need to know whether a particular name at a particular spot refers to a variable, function, struct, enum, module, constant, or other item and what that item means. You can create scopes and change which names are in or out of scope. You can’t have two items with the same name in the same scope; tools are available to resolve name conflicts.

Rust has a number of features that allow you to manage your code’s organization, including which details are exposed, which details are private, and what names are in each scope in your programs. These features, sometimes collectively referred to as the module system, include:

当您编写大型程序时，组织代码将变得越来越重要。 通过对相关功能进行分组并将代码与不同的功能分开，您将弄清楚在哪里可以找到实现特定功能的代码以及在哪里可以更改功能的工作方式。

到目前为止，我们编写的程序都位于一个文件的一个模块中。 随着项目的增长，您应该通过将代码拆分为多个模块，然后拆分为多个文件来组织代码。 一个包可以包含多个二进制 crate，也可以包含一个库 crate。 随着包的增长，您可以将各个部分提取到单独的包中，这些包将成为外部依赖项。 本章涵盖了所有这些技术。 对于包含一组共同发展的相互关联的包的大型项目，Cargo 提供了工作空间，我们将在第 14 章的“Cargo 工作空间”部分中介绍这些工作空间。

我们还将讨论封装实现细节，这使您可以在更高级别重用代码：实现一个操作后，其他代码可以通过其公共接口调用您的代码，而无需了解实现的工作原理。 您编写代码的方式定义了哪些部分是可供其他代码使用的公共部分，以及哪些部分是您保留更改权利的私有实现细节。 这是限制您必须记住的细节数量的另一种方法。

一个相关的概念是范围：编写代码的嵌套上下文有一组定义为“范围内”的名称。 在读取、编写和编译代码时，程序员和编译器需要知道特定位置的特定名称是否引用变量、函数、结构、枚举、模块、常量或其他项目以及该项目的含义。 您可以创建范围并更改范围内或范围外的名称。 同一范围内不能有两个同名的项目； 可以使用工具来解决名称冲突。

Rust 具有许多功能，可让您管理代码的组织，包括公开哪些详细信息、哪些详细信息是私有的，以及程序中每个范围内的名称。 这些功能有时统称为模块系统，包括：

+ Packages: A Cargo feature that lets you build, test, and share crates
+ Crates: A tree of modules that produces a library or executable
+ Modules and use: Let you control the organization, scope, and privacy of paths
+ Paths: A way of naming an item, such as a struct, function, or module  

In this chapter, we’ll cover all these features, discuss how they interact, and explain how to use them to manage scope. By the end, you should have a solid understanding of the module system and be able to work with scopes like a pro!

+ Packages：Cargo 功能，可让您构建、测试和共享 crate
+ Crates：生成库或可执行文件的模块树
+ 模块和**use**：让您控制路径的组织、范围和隐私
+ 路径：命名项目（例如结构、函数或模块）的一种方式

在本章中，我们将介绍所有这些功能，讨论它们如何交互，并解释如何使用它们来管理范围。 最后，您应该对模块系统有深入的了解，并能够像专业人士一样使用存在空间(scopes)！以后存在空间(scopes),简称为空间。

## 7.1 Packages and Crates
The first parts of the module system we’ll cover are packages and crates.

A crate is the smallest amount of code that the Rust compiler considers at a time. Even if you run rustc rather than cargo and pass a single source code file (as we did all the way back in the “Writing and Running a Rust Program” section of Chapter 1), the compiler considers that file to be a crate. Crates can contain modules, and the modules may be defined in other files that get compiled with the crate, as we’ll see in the coming sections.

A crate can come in one of two forms: a binary crate or a library crate. Binary crates are programs you can compile to an executable that you can run, such as a command-line program or a server. Each must have a function called main that defines what happens when the executable runs. All the crates we’ve created so far have been binary crates.

Library crates don’t have a main function, and they don’t compile to an executable. Instead, they define functionality intended to be shared with multiple projects. For example, the rand crate we used in Chapter 2 provides functionality that generates random numbers. Most of the time when Rustaceans say “crate”, they mean library crate, and they use “crate” interchangeably with the general programming concept of a “library".

The crate root is a source file that the Rust compiler starts from and makes up the root module of your crate (we’ll explain modules in depth in the “Defining Modules to Control Scope and Privacy” section).

A package is a bundle of one or more crates that provides a set of functionality. A package contains a Cargo.toml file that describes how to build those crates. Cargo is actually a package that contains the binary crate for the command-line tool you’ve been using to build your code. The Cargo package also contains a library crate that the binary crate depends on. Other projects can depend on the Cargo library crate to use the same logic the Cargo command-line tool uses.

A package can contain as many binary crates as you like, but at most only one library crate. A package must contain at least one crate, whether that’s a library or binary crate.

Let’s walk through what happens when we create a package. First, we enter the command cargo new:

我们将介绍的模块系统的第一部分是Packages和Crates。

Crate是 Rust 编译器一次考虑的最小代码量。 即使您运行 rustc 而不是 Cargo 并传递单个源代码文件（正如我们在第 1 章的“编写和运行 Rust 程序”部分所做的那样），编译器也会认为该文件是一个 crate。 Crate可以包含模块，并且模块可以在与Crate一起编译的其他文件中定义，正如我们将在接下来的部分中看到的那样。

Crate可以采用以下两种形式之一：二进制Crate或库Crate。 二进制包是可以编译为可以运行的可执行文件的程序，例如命令行程序或服务器。 每个都必须有一个名为 main 的函数，该函数定义可执行文件运行时会发生什么。 到目前为止，我们创建的所有 crate 都是二进制 crate。

库包没有 main 函数，并且它们不会编译为可执行文件。 相反，它们定义了旨在与多个项目共享的功能。 例如，我们在第 2 章中使用的 rand crate 提供了生成随机数的功能。 大多数时候，Rustaceans 说“crate”时，他们指的是库 crate，并且他们将“crate”与“库”的一般编程概念互换使用。

crate 根是 Rust 编译器启动的源文件，并构成 crate 的根模块（我们将在“定义控制范围和隐私的模块”部分中深入解释模块）。

包是一个或多个提供一组功能的包的捆绑。 包中包含一个 Cargo.toml 文件，该文件描述了如何构建这些 crate。 Cargo 实际上是一个包，其中包含您用来构建代码的命令行工具的二进制包。 Cargo 包还包含二进制 crate 所依赖的库 crate。 其他项目可以依赖 Cargo 库 crate 来使用 Cargo 命令行工具使用的相同逻辑。

一个包可以包含任意多个二进制 crate，但最多只能包含一个库 crate。 一个包必须至少包含一个 crate，无论是库还是二进制 crate。

让我们来看看创建包时会发生什么。 首先，我们输入命令cargo new：
```
$ cargo new my-project
     Created binary (application) `my-project` package
$ ls my-project
Cargo.toml
src
$ ls my-project/src
main.rs

```
After we run cargo new, we use ls to see what Cargo creates. In the project directory, there’s a Cargo.toml file, giving us a package. There’s also a src directory that contains main.rs. Open Cargo.toml in your text editor, and note there’s no mention of src/main.rs. Cargo follows a convention that src/main.rs is the crate root of a binary crate with the same name as the package. Likewise, Cargo knows that if the package directory contains src/lib.rs, the package contains a library crate with the same name as the package, and src/lib.rs is its crate root. Cargo passes the crate root files to rustc to build the library or binary.

Here, we have a package that only contains src/main.rs, meaning it only contains a binary crate named my-project. If a package contains src/main.rs and src/lib.rs, it has two crates: a binary and a library, both with the same name as the package. A package can have multiple binary crates by placing files in the src/bin directory: each file will be a separate binary crate.


运行 Cargo new 后，我们使用 ls 来查看 Cargo 创建了什么。 在项目目录中，有一个 Cargo.toml 文件，为我们提供了一个包。 还有一个包含 main.rs 的 src 目录。 在文本编辑器中打开 Cargo.toml，请注意没有提及 src/main.rs。 Cargo 遵循一个约定，即 src/main.rs 是与包同名的二进制 crate 的 crate 根。 同样，Cargo 知道如果包目录包含 src/lib.rs，则该包包含一个与该包同名的库 crate，并且 src/lib.rs 是其 crate 根。 Cargo 将 crate 根文件传递给 rustc 以构建库或二进制文件。

在这里，我们有一个仅包含 src/main.rs 的包，这意味着它仅包含一个名为 my-project 的二进制包。 如果一个包包含 src/main.rs 和 src/lib.rs，它有两个 crate：一个二进制文件和一个库，两者与包同名。 通过将文件放置在 src/bin 目录中，一个包可以拥有多个二进制 crate：每个文件将是一个单独的二进制 crate。

## 7.2 定义模块来控制范围和隐私

In this section, we’ll talk about modules and other parts of the module system, namely paths that allow you to name items; the use keyword that brings a path into scope; and the pub keyword to make items public. We’ll also discuss the as keyword, external packages, and the glob operator.

First, we’re going to start with a list of rules for easy reference when you’re organizing your code in the future. Then we’ll explain each of the rules in detail.

在本节中，我们将讨论模块和模块系统的其他部分，即允许您命名项目的路径； use 关键字将路径引入范围； 和 pub 关键字使项目公开。 我们还将讨论 as 关键字、外部包和 glob 运算符。

首先，我们将从一系列规则开始，以便您将来组织代码时轻松参考。 然后我们将详细解释每条规则。

### 模块备忘单(Modules Cheat Sheet)
Here we provide a quick reference on how modules, paths, the use keyword, and the pub keyword work in the compiler, and how most developers organize their code. We’ll be going through examples of each of these rules throughout this chapter, but this is a great place to refer to as a reminder of how modules work.

+ Start from the crate root: When compiling a crate, the compiler first looks in the crate root file (usually src/lib.rs for a library crate or src/main.rs for a binary crate) for code to compile.
+ Declaring modules: In the crate root file, you can declare new modules; say, you declare a “garden” module with mod garden;. The compiler will look for the module’s code in these places:
   * Inline, within curly brackets that replace the semicolon following mod garden
   * In the file src/garden.rs
   * In the file src/garden/mod.rs
+ Declaring submodules: In any file other than the crate root, you can declare submodules. For example, you might declare mod vegetables; in src/garden.rs. The compiler will look for the submodule’s code within the directory named for the parent module in these places:
   * Inline, directly following mod vegetables, within curly brackets instead of the semicolon
   * In the file src/garden/vegetables.rs
   * In the file src/garden/vegetables/mod.rs
+ Paths to code in modules: Once a module is part of your crate, you can refer to code in that module from anywhere else in that same crate, as long as the privacy rules allow, using the path to the code. For example, an Asparagus type in the garden vegetables module would be found at crate::garden::vegetables::Asparagus.
+ Private vs public: Code within a module is private from its parent modules by default. To make a module public, declare it with pub mod instead of mod. To make items within a public module public as well, use pub before their declarations.
+ The use keyword: Within a scope, the use keyword creates shortcuts to items to reduce repetition of long paths. In any scope that can refer to crate::garden::vegetables::Asparagus, you can create a shortcut with use crate::garden::vegetables::Asparagus; and from then on you only need to write Asparagus to make use of that type in the scope.

Here we create a binary crate named backyard that illustrates these rules. The crate’s directory, also named backyard, contains these files and directories:

在这里，我们提供了有关模块、路径、use 关键字和 pub 关键字如何在编译器中工作以及大多数开发人员如何组织代码的快速参考。 我们将在本章中介绍每条规则的示例，但这是一个很好的地方，可以用来提醒模块如何工作。

+ 从 crate 根开始：编译 crate 时，编译器首先在 crate 根文件（对于库 crate 通常为 src/lib.rs，对于二进制 crate 则为 src/main.rs）查找要编译的代码。
+ 声明模块：在 crate 根文件中，您可以声明新模块； 比如说，你用 mod Garden; 声明了一个“garden”模块。 编译器将在这些位置查找模块的代码：
   * 在crate根文件中内联，在大括号内，替换 mod garden后面的分号
   * 在文件 src/garden.rs
   * 在文件 src/garden/mod.rs
+ 声明子模块：在除 crate 根之外的任何文件中，您都可以声明子模块。 例如，您可以声明 mod vegetables； 在 src/garden.rs 中。 编译器将在以下位置的父模块命名的目录中查找子模块的代码：
   * 在声明文件内联，直接跟随 mod vegetables，在大括号内,替换分号
   * 在文件 src/garden/vegetables.rs
   * 在文件 src/garden/vegetables/mod.rs
+ 模块中代码的路径：一旦模块成为您的 crate 的一部分，只要隐私规则允许，您就可以使用代码的路径从同一 crate 中的其他任何位置引用该模块中的代码。 例如，garden vegetable模块中的 Asparagus 类型可以在 crate::garden::vegetables::Asparagus 中找到。
+ 私有与公共：默认情况下，模块内的代码对于其父模块是私有的。 要使模块公开，请使用 pub mod 而不是 mod 声明它。 要使公共模块中的项目也成为公共的，请在其声明之前使用 pub 。
+ use 关键字：在某个范围内，use 关键字创建项目的快捷方式以减少长路径的重复。 在任何可以引用 crate::garden::vegetables::Asparagus 的作用域中，您可以使用 use crate::garden::vegetables::Asparagus; 创建快捷方式； 从那时起，您只需要编写 Asparagus 即可在作用域中使用该类型。
在这里，我们创建一个名为 backyard 的二进制crate来说明这些规则。 该crate的目录也称为 backyard，包含以下文件和目录：
```
backyard
├── Cargo.lock
├── Cargo.toml
└── src
    ├── garden
    │   └── vegetables.rs
    ├── garden.rs
    └── main.rs

```
The crate root file in this case is src/main.rs, and it contains:
在样例中的crate根文件是src/main.rs,它包括：
Filename: src/main.rs
```
use crate::garden::vegetables::Asparagus;

pub mod garden;

fn main() {
    let plant = Asparagus {};
    println!("I'm growing {:?}!", plant);
}
```
The **pub mod garden;** line tells the compiler to include the code it finds in src/garden.rs, which is:  
"pub mod garden;"告诉编译器去包含在src/garden.rs文件中发现的代码，它们是：  
Filename: src/garden.rs
```
pub mod vegetables;
```
Here, **pub mod vegetables;** means the code in src/garden/vegetables.rs is included too. That code is:  
这里， "pub mod vegetables;"意味着在src/garden/vegetables.rs文件中的代码也被包括。它们是：
```
#[derive(Debug)]
pub struct Asparagus {}
```
Now let’s get into the details of these rules and demonstrate them in action!

### 将相关代码分组到模块中

Modules let us organize code within a crate for readability and easy reuse. Modules also allow us to control the privacy of items, because code within a module is private by default. Private items are internal implementation details not available for outside use. We can choose to make modules and the items within them public, which exposes them to allow external code to use and depend on them.

As an example, let’s write a library crate that provides the functionality of a restaurant. We’ll define the signatures of functions but leave their bodies empty to concentrate on the organization of the code, rather than the implementation of a restaurant.

In the restaurant industry, some parts of a restaurant are referred to as front of house and others as back of house. Front of house is where customers are; this encompasses where the hosts seat customers, servers take orders and payment, and bartenders make drinks. Back of house is where the chefs and cooks work in the kitchen, dishwashers clean up, and managers do administrative work.

To structure our crate in this way, we can organize its functions into nested modules. Create a new library named restaurant by running cargo new restaurant --lib; then enter the code in Listing 7-1 into src/lib.rs to define some modules and function signatures. Here’s the front of house section:

Filename: src/lib.rs

模块让我们可以在Packages内组织代码，以提高可读性和易于重用。 模块还允许我们控制项目的隐私，因为模块内的代码默认是私有的。 私有项目是内部实现细节，不可供外部使用。 我们可以选择将模块及其中的项目公开，从而公开它们以允许外部代码使用和依赖它们。

作为示例，让我们编写一个提供餐厅功能的Crate。 我们将定义函数的签名，但将它们的主体留空，以专注于代码的组织，而不是餐厅的实现。

在餐饮业中，餐厅的某些部分称为前台，其他部分称为后台。 前台是顾客所在的地方； 这包括主人让顾客就座、服务员接受订单和付款以及调酒师调制饮料的地方。 后厨是厨师和厨师在厨房工作、洗碗机清洁以及经理进行行政工作的地方。

为了以这种方式构建我们的Crate，我们可以将其功能组织到嵌套模块中。 通过运行 Cargo new Restaurant --lib 创建一个名为 Restaurant 的新库； 然后将清单7-1中的代码输入到src/lib.rs中以定义一些模块和函数签名。 这是房子的前面部分：

文件名：src/lib.rs

```
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}

        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}

        fn serve_order() {}

        fn take_payment() {}
    }
}
```

清单 7-1：一个 front_of_house 模块，其中包含其他包含函数的模块  

We define a module with the mod keyword followed by the name of the module (in this case, front_of_house). The body of the module then goes inside curly brackets. Inside modules, we can place other modules, as in this case with the modules hosting and serving. Modules can also hold definitions for other items, such as structs, enums, constants, traits, and—as in Listing 7-1—functions.

By using modules, we can group related definitions together and name why they’re related. Programmers using this code can navigate the code based on the groups rather than having to read through all the definitions, making it easier to find the definitions relevant to them. Programmers adding new functionality to this code would know where to place the code to keep the program organized.

Earlier, we mentioned that src/main.rs and src/lib.rs are called crate roots. The reason for their name is that the contents of either of these two files form a module named crate at the root of the crate’s module structure, known as the module tree.

Listing 7-2 shows the module tree for the structure in Listing 7-1.



我们使用 mod 关键字定义一个模块，后跟模块名称（在本例中为 front_of_house）。 然后模块的主体进入大括号内。 在模块内部，我们可以放置其他模块，就像本例中的hosting和serving模块一样。 模块还可以保存其他项目的定义，例如结构、枚举、常量、特征以及（如清单 7-1 所示）函数。

通过使用模块，我们可以将相关的定义分组在一起并说出它们相关的原因。 使用此代码的程序员可以根据组导航代码，而不必阅读所有定义，从而更容易找到与其相关的定义。 向此代码添加新功能的程序员将知道将代码放置在哪里以保持程序的组织性。

之前，我们提到 src/main.rs 和 src/lib.rs 称为 crate 根。 它们的名称的原因是这两个文件中的任何一个的内容在 crate 模块结构的根部形成一个名为 crate 的模块，称为模块树。

清单 7-2 显示了清单 7-1 中结构的模块树。

```
crate
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment

```

清单 7-2：清单 7-1 中代码的模块树

This tree shows how some of the modules nest inside one another; for example, hosting nests inside front_of_house. The tree also shows that some modules are siblings to each other, meaning they’re defined in the same module; hosting and serving are siblings defined within front_of_house. If module A is contained inside module B, we say that module A is the child of module B and that module B is the parent of module A. Notice that the entire module tree is rooted under the implicit module named crate.  
该树显示了一些模块如何相互嵌套； 例如，hosting嵌套在 front_of_house。 该树还显示一些模块彼此是兄弟模块，这意味着它们是在同一模块中定义的； hosting和serving是 front_of_house 中定义的兄弟。 如果模块 A 包含在模块 B 中，我们就说模块 A 是模块 B 的子模块，而模块 B 是模块 A 的父模块。请注意，整个模块树的根都位于名为 crate 的隐式模块下。

The module tree might remind you of the filesystem’s directory tree on your computer; this is a very apt comparison! Just like directories in a filesystem, you use modules to organize your code. And just like files in a directory, we need a way to find our modules.


模块树可能会让您想起计算机上文件系统的目录树； 这是一个非常恰当的比较！ 就像文件系统中的目录一样，您可以使用模块来组织代码。 就像目录中的文件一样，我们需要一种方法来查找模块。

## 7.3 引用模块树中项目的路径

To show Rust where to find an item in a module tree, we use a path in the same way we use a path when navigating a filesystem. To call a function, we need to know its path.

A path can take two forms:

+ An absolute path is the full path starting from a crate root; for code from an external crate, the absolute path begins with the crate name, and for code from the current crate, it starts with the literal crate.
+ A relative path starts from the current module and uses self, super, or an identifier in the current module.

Both absolute and relative paths are followed by one or more identifiers separated by double colons (::).

Returning to Listing 7-1, say we want to call the add_to_waitlist function. This is the same as asking: what’s the path of the add_to_waitlist function? Listing 7-3 contains Listing 7-1 with some of the modules and functions removed.

We’ll show two ways to call the add_to_waitlist function from a new function eat_at_restaurant defined in the crate root. These paths are correct, but there’s another problem remaining that will prevent this example from compiling as-is. We’ll explain why in a bit.

The eat_at_restaurant function is part of our library crate’s public API, so we mark it with the pub keyword. In the “Exposing Paths with the pub Keyword” section, we’ll go into more detail about pub.

Filename: src/lib.rs

为了向 Rust 展示在模块树中的何处查找项目，我们使用路径，就像导航文件系统时使用路径一样。 要调用一个函数，我们需要知道它的路径。

路径可以采用两种形式：

+ 绝对路径是从 crate 根开始的完整路径； 对于来自外部 crate 的代码，绝对路径以 crate的crate_name开头，对于来自当前 crate 的代码，绝对路径以字符串"crate"开头。
+ 相对路径从当前模块开始，使用 self、super 或当前模块中的标识符。

绝对路径和相对路径都后跟一个或多个以双冒号 (::) 分隔的标识符。

返回清单 7-1，假设我们要调用 add_to_waitlist 函数。 这相当于问：add_to_waitlist 函数的路径是什么？ 清单 7-3 包含清单 7-1，其中删除了一些模块和函数。

我们将展示从 crate 根中定义的新函数 eat_at_restaurant 调用 add_to_waitlist 函数的两种方法。 这些路径是正确的，但还存在另一个问题，该问题将阻止该示例按原样编译。 我们稍后会解释原因。

eat_at_restaurant 函数是我们库 crate 的公共 API 的一部分，因此我们用 pub 关键字标记它。 在“使用 pub 关键字公开路径”部分中，我们将详细介绍 pub。

文件名：src/lib.rs

```
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // Absolute path
    crate::front_of_house::hosting::add_to_waitlist();

    // Relative path
    front_of_house::hosting::add_to_waitlist();
}
```

示例 7-3：使用绝对路径和相对路径调用 add_to_waitlist 函数

The first time we call the add_to_waitlist function in eat_at_restaurant, we use an absolute path. The add_to_waitlist function is defined in the same crate as eat_at_restaurant, which means we can use the crate keyword to start an absolute path. We then include each of the successive modules until we make our way to add_to_waitlist. You can imagine a filesystem with the same structure: we’d specify the path /front_of_house/hosting/add_to_waitlist to run the add_to_waitlist program; using the crate name to start from the crate root is like using / to start from the filesystem root in your shell.

The second time we call add_to_waitlist in eat_at_restaurant, we use a relative path. The path starts with front_of_house, the name of the module defined at the same level of the module tree as eat_at_restaurant. Here the filesystem equivalent would be using the path front_of_house/hosting/add_to_waitlist. Starting with a module name means that the path is relative.

Choosing whether to use a relative or absolute path is a decision you’ll make based on your project, and depends on whether you’re more likely to move item definition code separately from or together with the code that uses the item. For example, if we move the front_of_house module and the eat_at_restaurant function into a module named customer_experience, we’d need to update the absolute path to add_to_waitlist, but the relative path would still be valid. However, if we moved the eat_at_restaurant function separately into a module named dining, the absolute path to the add_to_waitlist call would stay the same, but the relative path would need to be updated. Our preference in general is to specify absolute paths because it’s more likely we’ll want to move code definitions and item calls independently of each other.

Let’s try to compile Listing 7-3 and find out why it won’t compile yet! The error we get is shown in Listing 7-4.



第一次调用 eat_at_restaurant 中的 add_to_waitlist 函数时，我们使用绝对路径。 add_to_waitlist 函数与 eat_at_restaurant 定义在同一个 crate 中，这意味着我们可以使用 crate 关键字来启动绝对路径。 然后，我们包含每个连续的模块，直到到达 add_to_waitlist。 你可以想象一个具有相同结构的文件系统：我们指定路径 /front_of_house/hosting/add_to_waitlist 来运行 add_to_waitlist 程序； 使用 crate 名称从 crate 根目录启动就像在 shell 中使用 / 从文件系统根目录启动一样。

第二次在 eat_at_restaurant 中调用 add_to_waitlist 时，我们使用相对路径。 该路径以 front_of_house 开头，它是在模块树中与 eat_at_restaurant 相同级别定义的模块名称。 这里的文件系统等效项将使用路径 front_of_house/hosting/add_to_waitlist。 以模块名称开头意味着路径是相对的。

选择是使用相对路径还是绝对路径是您根据项目做出的决定，并且取决于您是否更有可能将项目定义代码与使用该项目的代码分开移动或一起移动。 例如，如果我们将 front_of_house 模块和 eat_at_restaurant 函数移动到名为 customer_experience 的模块中，我们需要将绝对路径更新为 add_to_waitlist，但相对路径仍然有效。 但是，如果我们将 eat_at_restaurant 函数单独移动到名为dining 的模块中，则 add_to_waitlist 调用的绝对路径将保持不变，但相对路径需要更新。 一般来说，我们更喜欢指定绝对路径，因为我们更有可能希望彼此独立地移动代码定义和项目调用。

让我们尝试编译清单 7-3，看看为什么它还不能编译！ 我们得到的错误如清单 7-4 所示。

```
$ cargo build
   Compiling restaurant v0.1.0 (file:///projects/restaurant)
error[E0603]: module `hosting` is private
 --> src/lib.rs:9:28
  |
9 |     crate::front_of_house::hosting::add_to_waitlist();
  |                            ^^^^^^^ private module
  |
note: the module `hosting` is defined here
 --> src/lib.rs:2:5
  |
2 |     mod hosting {
  |     ^^^^^^^^^^^

error[E0603]: module `hosting` is private
  --> src/lib.rs:12:21
   |
12 |     front_of_house::hosting::add_to_waitlist();
   |                     ^^^^^^^ private module
   |
note: the module `hosting` is defined here
  --> src/lib.rs:2:5
   |
2  |     mod hosting {
   |     ^^^^^^^^^^^

For more information about this error, try `rustc --explain E0603`.
error: could not compile `restaurant` due to 2 previous errors

```
清单 7-4：构建清单 7-3 中的代码时出现的编译器错误

The error messages say that module hosting is private. In other words, we have the correct paths for the hosting module and the add_to_waitlist function, but Rust won’t let us use them because it doesn’t have access to the private sections. In Rust, all items (functions, methods, structs, enums, modules, and constants) are private to parent modules by default. If you want to make an item like a function or struct private, you put it in a module.

Items in a parent module can’t use the private items inside child modules, but items in child modules can use the items in their ancestor modules. This is because child modules wrap and hide their implementation details, but the child modules can see the context in which they’re defined. To continue with our metaphor, think of the privacy rules as being like the back office of a restaurant: what goes on in there is private to restaurant customers, but office managers can see and do everything in the restaurant they operate.

Rust chose to have the module system function this way so that hiding inner implementation details is the default. That way, you know which parts of the inner code you can change without breaking outer code. However, Rust does give you the option to expose inner parts of child modules’ code to outer ancestor modules by using the pub keyword to make an item public.



错误消息表明模块hosting是私有的。 换句话说，我们拥有hostring模块和 add_to_waitlist 函数的正确路径，但 Rust 不允许我们使用它们，因为它无法访问私有部分。 在 Rust 中，默认情况下，所有项目（函数、方法、结构、枚举、模块和常量）都是父模块私有的。 如果您想将函数或结构等项目设为私有，则可以将其放入模块中。

父模块中的项目不能使用子模块中的私有项目，但子模块中的项目可以使用其祖先模块中的项目。 这是因为子模块包装并隐藏其实现细节，但子模块可以看到定义它们的上下文。 继续我们的比喻，将隐私规则想象成餐厅的后台：那里发生的事情对餐厅顾客来说是私人的，但办公室经理可以看到并执行他们经营的餐厅中的一切。

Rust 选择以这种方式拥有模块系统功能，因此隐藏内部实现细节是默认的。 这样，您就知道可以更改内部代码的哪些部分而不破坏外部代码。 然而，Rust 确实为您提供了通过使用 pub 关键字公开项目来将子模块代码的内部部分公开给外部祖先模块的选项。
### 使用 pub 关键字公开路径(Exposing Paths with the pub Keyword)

Let’s return to the error in Listing 7-4 that told us the hosting module is private. We want the eat_at_restaurant function in the parent module to have access to the add_to_waitlist function in the child module, so we mark the hosting module with the pub keyword, as shown in Listing 7-5.


让我们回到清单 7-4 中的错误，它告诉我们hosting模块是私有的。 我们希望父模块中的 eat_at_restaurant 函数能够访问子模块中的 add_to_waitlist 函数，因此我们使用 pub 关键字标记hosting模块，如清单 7-5 所示。

文件名：src/lib.rs
```
//error
mod front_of_house {
    pub mod hosting {
        fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // Absolute path
    crate::front_of_house::hosting::add_to_waitlist();

    // Relative path
    front_of_house::hosting::add_to_waitlist();
}
```
Listing 7-5: Declaring the hosting module as pub to use it from eat_at_restaurant

Unfortunately, the code in Listing 7-5 still results in an error, as shown in Listing 7-6.

不幸的是在listing 7-5中仍旧有一个错误，错误提示如listing 7-6.

```
$ cargo build
   Compiling restaurant v0.1.0 (file:///projects/restaurant)
error[E0603]: function `add_to_waitlist` is private
 --> src/lib.rs:9:37
  |
9 |     crate::front_of_house::hosting::add_to_waitlist();
  |                                     ^^^^^^^^^^^^^^^ private function
  |
note: the function `add_to_waitlist` is defined here
 --> src/lib.rs:3:9
  |
3 |         fn add_to_waitlist() {}
  |         ^^^^^^^^^^^^^^^^^^^^

error[E0603]: function `add_to_waitlist` is private
  --> src/lib.rs:12:30
   |
12 |     front_of_house::hosting::add_to_waitlist();
   |                              ^^^^^^^^^^^^^^^ private function
   |
note: the function `add_to_waitlist` is defined here
  --> src/lib.rs:3:9
   |
3  |         fn add_to_waitlist() {}
   |         ^^^^^^^^^^^^^^^^^^^^

For more information about this error, try `rustc --explain E0603`.
error: could not compile `restaurant` due to 2 previous errors

```
清单 7-6：构建清单 7-5 中的代码时出现的编译器错误


What happened? Adding the pub keyword in front of mod hosting makes the module public. With this change, if we can access front_of_house, we can access hosting. But the contents of hosting are still private; making the module public doesn’t make its contents public. The pub keyword on a module only lets code in its ancestor modules refer to it, not access its inner code. Because modules are containers, there’s not much we can do by only making the module public; we need to go further and choose to make one or more of the items within the module public as well.

The errors in Listing 7-6 say that the add_to_waitlist function is private. The privacy rules apply to structs, enums, functions, and methods as well as modules.

Let’s also make the add_to_waitlist function public by adding the pub keyword before its definition, as in Listing 7-7.

Filename: src/lib.rs



发生了什么？ 在 mod hosting前面添加 pub 关键字将使模块公开。 通过此更改，如果我们可以访问 front_of_house，我们就可以访问hosting。 但hosting的内容仍然是私密的； 公开模块并不会公开其内容。 模块上的 pub 关键字仅允许其祖先模块中的代码引用它，而不能访问其内部代码。 因为模块是容器，所以仅将模块公开我们无能为力； 我们需要更进一步，选择将模块中的一项或多项设为公开。

清单 7-6 中的错误表明 add_to_waitlist 函数是私有的。 隐私规则适用于结构、枚举、函数、方法以及模块。

我们还可以通过在定义之前添加 pub 关键字来将 add_to_waitlist 函数公开，如清单 7-7 所示。

文件名：src/lib.rs
```
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // Absolute path
    crate::front_of_house::hosting::add_to_waitlist();

    // Relative path
    front_of_house::hosting::add_to_waitlist();
}
```

示例 7-7：将 pub 关键字添加到 mod Hosting 和 fn add_to_waitlist 让我们可以从 eat_at_restaurant 调用该函数

Now the code will compile! To see why adding the pub keyword lets us use these paths in add_to_waitlist with respect to the privacy rules, let’s look at the absolute and the relative paths.

In the absolute path, we start with crate, the root of our crate’s module tree. The front_of_house module is defined in the crate root. While front_of_house isn’t public, because the eat_at_restaurant function is defined in the same module as front_of_house (that is, eat_at_restaurant and front_of_house are siblings), we can refer to front_of_house from eat_at_restaurant. Next is the hosting module marked with pub. We can access the parent module of hosting, so we can access hosting. Finally, the add_to_waitlist function is marked with pub and we can access its parent module, so this function call works!

In the relative path, the logic is the same as the absolute path except for the first step: rather than starting from the crate root, the path starts from front_of_house. The front_of_house module is defined within the same module as eat_at_restaurant, so the relative path starting from the module in which eat_at_restaurant is defined works. Then, because hosting and add_to_waitlist are marked with pub, the rest of the path works, and this function call is valid!

If you plan on sharing your library crate so other projects can use your code, your public API is your contract with users of your crate that determines how they can interact with your code. There are many considerations around managing changes to your public API to make it easier for people to depend on your crate. These considerations are out of the scope of this book; if you’re interested in this topic, see The Rust API Guidelines.


现在代码可以编译了！ 为了了解为什么添加 pub 关键字可以让我们在 add_to_waitlist 中使用这些路径来遵守隐私规则，让我们看看绝对路径和相对路径。

在绝对路径中，我们从 crate 开始，它是 crate 模块树的根。 front_of_house 模块在 crate 根中定义。 虽然 front_of_house 不是公开的，但由于 eat_at_restaurant 函数与 front_of_house 定义在同一模块中（即 eat_at_restaurant 和 front_of_house 是兄弟姐妹），因此我们可以从 eat_at_restaurant 引用 front_of_house。 接下来是标有 pub 的hostring模块。 我们可以访问hosting的父模块，这样就可以访问hosting了。 最后，add_to_waitlist函数被标记为pub，我们可以访问它的父模块，所以这个函数调用有效！

在相对路径中，除了第一步之外，逻辑与绝对路径相同：路径不是从 crate 根开始，而是从 front_of_house 开始。 front_of_house 模块与 eat_at_restaurant 定义在同一模块中，因此从定义 eat_at_restaurant 的模块开始的相对路径有效。 然后，因为hosting和add_to_waitlist都被标记为pub，所以路径的其余部分有效，并且这个函数调用是有效的！

如果您计划共享您的库包，以便其他项目可以使用您的代码，那么您的公共 API 就是您与您的包的用户之间的合同，决定了他们如何与您的代码进行交互。 为了让人们更容易依赖你的 crate，管理公共 API 的更改需要考虑很多因素。 这些考虑因素超出了本书的范围。 如果您对此主题感兴趣，请参阅 Rust API 指南。

**Best Practices for Packages with a Binary and a Library**

We mentioned a package can contain both a src/main.rs binary crate root as well as a src/lib.rs library crate root, and both crates will have the package name by default. Typically, packages with this pattern of containing both a library and a binary crate will have just enough code in the binary crate to start an executable that calls code with the library crate. This lets other projects benefit from the most functionality that the package provides, because the library crate’s code can be shared.

The module tree should be defined in src/lib.rs. Then, any public items can be used in the binary crate by starting paths with the name of the package. The binary crate becomes a user of the library crate just like a completely external crate would use the library crate: it can only use the public API. This helps you design a good API; not only are you the author, you’re also a client!

In Chapter 12, we’ll demonstrate this organizational practice with a command-line program that will contain both a binary crate and a library crate.

**具有二进制文件和库的包的最佳实践**

我们提到一个包可以包含 src/main.rs 二进制 crate 根以及 src/lib.rs 库 crate 根，并且默认情况下这两个 crate 都将具有包名称。 通常，具有包含库和二进制 crate 的模式的包在二进制 crate 中将具有足够的代码来启动调用库 crate 代码的可执行文件。 这使得其他项目可以从该包提供的大部分功能中受益，因为库包的代码可以共享。

模块树应该在 src/lib.rs 中定义。 然后，通过以包的名称开始路径，可以在二进制包中使用任何公共项目。 二进制 crate 成为库 crate 的用户，就像完全外部的 crate 使用库 crate 一样：它只能使用公共 API。 这可以帮助你设计一个好的API； 您不仅是作者，也是客户！

在第 12 章中，我们将使用包含二进制 crate 和库 crate 的命令行程序来演示这种组织实践。

### Starting Relative Paths with super
We can construct relative paths that begin in the parent module, rather than the current module or the crate root, by using super at the start of the path. This is like starting a filesystem path with the .. syntax. Using super allows us to reference an item that we know is in the parent module, which can make rearranging the module tree easier when the module is closely related to the parent, but the parent might be moved elsewhere in the module tree someday.

Consider the code in Listing 7-8 that models the situation in which a chef fixes an incorrect order and personally brings it out to the customer. The function fix_incorrect_order defined in the back_of_house module calls the function deliver_order defined in the parent module by specifying the path to deliver_order starting with super:

Filename: src/lib.rs
我们可以通过在路径开头使用 super 来构造从父模块开始的相对路径，而不是从当前模块或 crate 根开始。 这就像使用 .. 语法启动文件系统路径。 使用 super 允许我们引用父模块中已知的项，当模块与父模块密切相关时，这可以使重新排列模块树变得更容易，但有一天父模块可能会被移动到模块树中的其他位置。

考虑清单 7-8 中的代码，该代码模拟了厨师修复错误订单并亲自将其呈现给顾客的情况。 back_of_house模块中定义的函数fix_in Correct_order通过指定以super开头的deliver_order的路径来调用父模块中定义的函数deliver_order：

文件名：src/lib.rs
```
fn deliver_order() {}

mod back_of_house {
    fn fix_incorrect_order() {
        cook_order();
        super::deliver_order();
    }

    fn cook_order() {}
}
```
示例 7-8：使用以 super 开头的相对路径调用函数


The fix_incorrect_order function is in the back_of_house module, so we can use super to go to the parent module of back_of_house, which in this case is crate, the root. From there, we look for deliver_order and find it. Success! We think the back_of_house module and the deliver_order function are likely to stay in the same relationship to each other and get moved together should we decide to reorganize the crate’s module tree. Therefore, we used super so we’ll have fewer places to update code in the future if this code gets moved to a different module.



fix_incorrect_order 函数位于 back_of_house 模块中，因此我们可以使用 super 转到 back_of_house 的父模块，在本例中是 crate，即根模块。 从那里，我们寻找 Deliver_order 并找到它。 成功！ 我们认为，如果我们决定重新组织crate的模块树，back_of_house 模块和 Deliver_order 函数可能会保持相同的相互关系，并一起移动。 因此，我们使用了 super，这样如果代码被移动到不同的模块，我们将来更新代码的地方就会更少。

### Making Structs and Enums Public
We can also use pub to designate structs and enums as public, but there are a few details extra to the usage of pub with structs and enums. If we use pub before a struct definition, we make the struct public, but the struct’s fields will still be private. We can make each field public or not on a case-by-case basis. In Listing 7-9, we’ve defined a public back_of_house::Breakfast struct with a public toast field but a private seasonal_fruit field. This models the case in a restaurant where the customer can pick the type of bread that comes with a meal, but the chef decides which fruit accompanies the meal based on what’s in season and in stock. The available fruit changes quickly, so customers can’t choose the fruit or even see which fruit they’ll get.

Filename: src/lib.rs
我们还可以使用 pub 将结构体和枚举指定为公共结构体和枚举，但是将 pub 与结构体和枚举一起使用还有一些额外的细节。 如果我们在结构体定义之前使用 pub，我们会将结构体设为公共，但结构体的字段仍然是私有的。 我们可以根据具体情况公开或不公开每个字段。 在清单 7-9 中，我们定义了一个公共 back_of_house::Breakfast 结构体，其中包含一个公共 toast 字段和一个私有seasonal_fruit 字段。 这个模型模拟了一家餐厅的情况，顾客可以选择餐食中搭配的面包类型，但厨师会根据时令和库存来决定搭配餐食的水果。 可用的水果变化很快，因此顾客无法选择水果，甚至无法看到他们会得到哪种水果。

文件名：src/lib.rs

```
mod back_of_house {
    pub struct Breakfast {
        pub toast: String,
        seasonal_fruit: String,
    }

    impl Breakfast {
        pub fn summer(toast: &str) -> Breakfast {
            Breakfast {
                toast: String::from(toast),
                seasonal_fruit: String::from("peaches"),
            }
        }
    }
}

pub fn eat_at_restaurant() {
    // Order a breakfast in the summer with Rye toast
    let mut meal = back_of_house::Breakfast::summer("Rye");
    // Change our mind about what bread we'd like
    meal.toast = String::from("Wheat");
    println!("I'd like {} toast please", meal.toast);

    // The next line won't compile if we uncomment it; we're not allowed
    // to see or modify the seasonal fruit that comes with the meal
    // meal.seasonal_fruit = String::from("blueberries");
}
```
示例 7-9：具有一些公共字段和一些私有字段的结构


Because the toast field in the back_of_house::Breakfast struct is public, in eat_at_restaurant we can write and read to the toast field using dot notation. Notice that we can’t use the seasonal_fruit field in eat_at_restaurant because seasonal_fruit is private. Try uncommenting the line modifying the seasonal_fruit field value to see what error you get!

Also, note that because back_of_house::Breakfast has a private field, the struct needs to provide a public associated function that constructs an instance of Breakfast (we’ve named it summer here). If Breakfast didn’t have such a function, we couldn’t create an instance of Breakfast in eat_at_restaurant because we couldn’t set the value of the private seasonal_fruit field in eat_at_restaurant.

In contrast, if we make an enum public, all of its variants are then public. We only need the pub before the enum keyword, as shown in Listing 7-10.


因为 back_of_house::Breakfast 结构中的 toast 字段是公共的，所以在 eat_at_restaurant 中我们可以使用点表示法写入和读取 toast 字段。 请注意，我们不能在 eat_at_restaurant 中使用seasonal_fruit 字段，因为seasonal_fruit 是私有的。 尝试取消注释修改seasonal_fruit字段值的行，看看会得到什么错误！

另外，请注意，因为 back_of_house::Breakfast 有一个私有字段，所以该结构需要提供一个公共关联函数来构造一个 Breakfast 实例（我们在这里将其命名为 Summer）。 如果Breakfast没有这样的函数，我们就无法在eat_at_restaurant中创建Breakfast的实例，因为我们无法在eat_at_restaurant中设置私有seasonal_fruit字段的值。

相反，如果我们公开一个枚举，那么它的所有变体都是公开的。 我们只需要在 enum 关键字之前加上 pub，如清单 7-10 所示。

文件名：src/lib.rs
```
mod back_of_house {
    pub enum Appetizer {
        Soup,
        Salad,
    }
}

pub fn eat_at_restaurant() {
    let order1 = back_of_house::Appetizer::Soup;
    let order2 = back_of_house::Appetizer::Salad;
}
```

示例 7-10：将枚举指定为 public 会使其所有变体公开
Because we made the Appetizer enum public, we can use the Soup and Salad variants in eat_at_restaurant.

Enums aren’t very useful unless their variants are public; it would be annoying to have to annotate all enum variants with pub in every case, so the default for enum variants is to be public. Structs are often useful without their fields being public, so struct fields follow the general rule of everything being private by default unless annotated with pub.

There’s one more situation involving pub that we haven’t covered, and that is our last module system feature: the use keyword. We’ll cover use by itself first, and then we’ll show how to combine pub and use.



因为我们公开了 Appetizer 枚举，所以我们可以在 eat_at_restaurant 中使用 Soup 和 Salad 变体。

除非枚举的变体是公开的，否则枚举并不是很有用。 在每种情况下都必须用 pub 注释所有枚举变体会很烦人，因此枚举变体的默认设置是公共的。 结构体在其字段不公开的情况下通常很有用，因此结构体字段遵循默认情况下所有内容都是私有的一般规则，除非用 pub 注释。

还有一种涉及 pub 的情况我们没有涉及，那就是我们的最后一个模块系统功能：use 关键字。 我们将首先介绍 use 本身，然后我们将展示如何将 pub 和 use 结合起来。

## 7.4 使用 use 关键字将路径纳入范围

Having to write out the paths to call functions can feel inconvenient and repetitive. In Listing 7-7, whether we chose the absolute or relative path to the add_to_waitlist function, every time we wanted to call add_to_waitlist we had to specify front_of_house and hosting too. Fortunately, there’s a way to simplify this process: we can create a shortcut to a path with the use keyword once, and then use the shorter name everywhere else in the scope.

In Listing 7-11, we bring the crate::front_of_house::hosting module into the scope of the eat_at_restaurant function so we only have to specify hosting::add_to_waitlist to call the add_to_waitlist function in eat_at_restaurant.

必须写出调用函数的路径可能会感觉不方便且重复。 在清单7-7中，无论我们选择add_to_waitlist函数的绝对路径还是相对路径，每次我们想要调用add_to_waitlist时，我们都必须指定front_of_house和hosting。 幸运的是，有一种方法可以简化这个过程：我们可以使用 use 关键字创建一个路径的快捷方式，然后在范围内的其他地方使用较短的名称。

在清单 7-11 中，我们将 crate::front_of_house::hosting 模块带入 eat_at_restaurant 函数的作用域中，因此我们只需指定hosting::add_to_waitlist 即可调用 eat_at_restaurant 中的 add_to_waitlist 函数。

Filename: src/lib.rs
```
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```
示例 7-11：通过 use 将模块纳入作用域


Adding use and a path in a scope is similar to creating a symbolic link in the filesystem. By adding use crate::front_of_house::hosting in the crate root, hosting is now a valid name in that scope, just as though the hosting module had been defined in the crate root. Paths brought into scope with use also check privacy, like any other paths.

Note that use only creates the shortcut for the particular scope in which the use occurs. Listing 7-12 moves the eat_at_restaurant function into a new child module named customer, which is then a different scope than the use statement, so the function body won’t compile:

在作用域中添加用途和路径类似于在文件系统中创建符号链接。 通过在 crate 根中添加 use crate::front_of_house::hosting，hosting 现在是该范围内的有效名称，就像hostring模块已在 crate 根中定义一样。 与任何其他路径一样，随着使用而进入范围的路径也会检查隐私。

请注意，use 仅为发生使用的特定范围创建快捷方式。 清单 7-12 将 eat_at_restaurant 函数移动到一个名为 customer 的新子模块中，该子模块与 use 语句的作用域不同，因此函数体将无法编译：

文件名：src/lib.rs
```
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting;

mod customer {
    pub fn eat_at_restaurant() {
        hosting::add_to_waitlist();
    }
}
```
Listing 7-12: A use statement only applies in the scope it’s in

The compiler error shows that the shortcut no longer applies within the customer module:
```
$ cargo build
   Compiling restaurant v0.1.0 (file:///projects/restaurant)
warning: unused import: `crate::front_of_house::hosting`
 --> src/lib.rs:7:5
  |
7 | use crate::front_of_house::hosting;
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  |
  = note: `#[warn(unused_imports)]` on by default

error[E0433]: failed to resolve: use of undeclared crate or module `hosting`
  --> src/lib.rs:11:9
   |
11 |         hosting::add_to_waitlist();
   |         ^^^^^^^ use of undeclared crate or module `hosting`

For more information about this error, try `rustc --explain E0433`.
warning: `restaurant` (lib) generated 1 warning
error: could not compile `restaurant` due to previous error; 1 warning emitted

```
Notice there’s also a warning that the use is no longer used in its scope! To fix this problem, move the use within the customer module too, or reference the shortcut in the parent module with super::hosting within the child customer module.

请注意，还有一个警告，表明该用途不再在其范围内使用！ 要解决此问题，请将使用移到客户模块中，或者在子客户模块中使用 super::hosting 引用父模块中的快捷方式。
### 创建惯用的使用路径(creating idiomatic use Paths)
In Listing 7-11, you might have wondered why we specified use crate::front_of_house::hosting and then called hosting::add_to_waitlist in eat_at_restaurant rather than specifying the use path all the way out to the add_to_waitlist function to achieve the same result, as in Listing 7-13.


在清单 7-11 中，您可能想知道为什么我们指定了 use crate::front_of_house::hosting，然后在 eat_at_restaurant 中调用hosting::add_to_waitlist，而不是一直指定 use 路径到 add_to_waitlist 函数来实现相同的结果， 如清单 7-13 所示。

文件名：src/lib.rs

```
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting::add_to_waitlist;

pub fn eat_at_restaurant() {
    add_to_waitlist();
}
```
示例 7-13：使用 use 将 add_to_waitlist 函数引入作用域，这是不惯用的


Although both Listing 7-11 and 7-13 accomplish the same task, Listing 7-11 is the idiomatic way to bring a function into scope with use. Bringing the function’s parent module into scope with use means we have to specify the parent module when calling the function. Specifying the parent module when calling the function makes it clear that the function isn’t locally defined while still minimizing repetition of the full path. The code in Listing 7-13 is unclear as to where add_to_waitlist is defined.

On the other hand, when bringing in structs, enums, and other items with use, it’s idiomatic to specify the full path. Listing 7-14 shows the idiomatic way to bring the standard library’s HashMap struct into the scope of a binary crate.


尽管清单 7-11 和 7-13 都完成了相同的任务，但清单 7-11 是将函数带入使用范围的惯用方法。 使用 use 将函数的父模块纳入作用域意味着我们在调用函数时必须指定父模块。 在调用函数时指定父模块可以清楚地表明该函数不是本地定义的，同时仍然最大限度地减少完整路径的重复。 清单 7-13 中的代码不清楚 add_to_waitlist 是在哪里定义的。

另一方面，当引入结构体、枚举和其他使用的项目时，指定完整路径是惯用的。 清单 7-14 显示了将标准库的 HashMap 结构引入二进制 crate 范围的惯用方法。

文件名：src/main.rs

```
use std::collections::HashMap;

fn main() {
    let mut map = HashMap::new();
    map.insert(1, 2);
}
```

示例 7-14：以惯用的方式将 HashMap 引入作用域
There’s no strong reason behind this idiom: it’s just the convention that has emerged, and folks have gotten used to reading and writing Rust code this way.

The exception to this idiom is if we’re bringing two items with the same name into scope with use statements, because Rust doesn’t allow that. Listing 7-15 shows how to bring two Result types into scope that have the same name but different parent modules and how to refer to them.

这个习惯用法背后没有什么强有力的理由：这只是已经出现的约定，人们已经习惯了以这种方式阅读和编写 Rust 代码。

这种习惯用法的例外是，如果我们使用 use 语句将两个同名的项带入作用域，因为 Rust 不允许这样做。 清单 7-15 显示了如何将两个具有相同名称但不同父模块的 Result 类型引入作用域，以及如何引用它们。

文件名：src/lib.rs
```
use std::fmt;
use std::io;

fn function1() -> fmt::Result {
    // --snip--
}

fn function2() -> io::Result<()> {
    // --snip--
}
```

示例 7-15：将两个同名类型放入同一作用域需要使用它们的父模块。
As you can see, using the parent modules distinguishes the two Result types. If instead we specified use std::fmt::Result and use std::io::Result, we’d have two Result types in the same scope and Rust wouldn’t know which one we meant when we used Result.


如您所见，使用父模块区分了两种结果类型。 相反，如果我们指定 use std::fmt::Result 和 use std::io::Result，我们就会在同一作用域中拥有两种 Result 类型，并且 Rust 不会知道当我们使用 Result 时我们指的是哪一种。

### 使用as关键字提供新的名称（Providing New Names with the as Keyword）
There’s another solution to the problem of bringing two types of the same name into the same scope with use: after the path, we can specify as and a new local name, or alias, for the type. Listing 7-16 shows another way to write the code in Listing 7-15 by renaming one of the two Result types using as.

对于使用 use 将两种同名类型引入同一作用域的问题，还有另一种解决方案：在路径之后，我们可以为该类型指定 as 和一个新的本地名称或别名。 清单 7-16 显示了编写清单 7-15 中的代码的另一种方法，即使用 as 重命名两个 Result 类型之一。

Filename: src/lib.rs

```
use std::fmt::Result;
use std::io::Result as IoResult;

fn function1() -> Result {
    // --snip--
    Ok(())
}

fn function2() -> IoResult<()> {
    // --snip--
    Ok(())
}
```

示例 7-16：当使用 as 关键字将类型引入作用域时重命名类型

In the second use statement, we chose the new name IoResult for the std::io::Result type, which won’t conflict with the Result from std::fmt that we’ve also brought into scope. Listing 7-15 and Listing 7-16 are considered idiomatic, so the choice is up to you!



在第二个 use 语句中，我们为 std::io::Result 类型选择了新名称 IoResult，这不会与我们也纳入范围的 std::fmt 的 Result 冲突。 清单 7-15 和清单 7-16 被认为是惯用的，因此选择取决于您！

### 使用pub use 重新导出名称（Re-exporting Names with pub use）
When we bring a name into scope with the use keyword, the name available in the new scope is private. To enable the code that calls our code to refer to that name as if it had been defined in that code’s scope, we can combine pub and use. This technique is called re-exporting because we’re bringing an item into scope but also making that item available for others to bring into their scope.

Listing 7-17 shows the code in Listing 7-11 with use in the root module changed to pub use.


当我们使用 use 关键字将名称引入作用域时，新作用域中可用的名称是私有的。 为了使调用我们代码的代码能够引用该名称，就好像它已在该代码的范围内定义一样，我们可以将 pub 和 use 结合起来。 这种技术称为重新导出，因为我们将某个项目纳入范围，同时也使该项目可供其他人纳入其范围。

清单 7-17 显示了清单 7-11 中的代码，其中 root 模块中的 use 更改为 pub use。

文件名：src/lib.rs

```
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

示例 7-17：通过 pub use 为新作用域中的任何代码提供可用的名称

Before this change, external code would have to call the add_to_waitlist function by using the path restaurant::front_of_house::hosting::add_to_waitlist(). Now that this pub use has re-exported the hosting module from the root module, external code can now use the path restaurant::hosting::add_to_waitlist() instead.

Re-exporting is useful when the internal structure of your code is different from how programmers calling your code would think about the domain. For example, in this restaurant metaphor, the people running the restaurant think about “front of house” and “back of house.” But customers visiting a restaurant probably won’t think about the parts of the restaurant in those terms. With pub use, we can write our code with one structure but expose a different structure. Doing so makes our library well organized for programmers working on the library and programmers calling the library. We’ll look at another example of pub use and how it affects your crate’s documentation in the “Exporting a Convenient Public API with pub use” section of Chapter 14.


在此更改之前，外部代码必须使用路径 Restaurant::front_of_house::hosting::add_to_waitlist() 调用 add_to_waitlist 函数。 现在，该 pub 使用已从根模块重新导出hostring模块，外部代码现在可以使用路径 Restaurant::hosting::add_to_waitlist() 代替。

当代码的内部结构与调用代码的程序员对域的看法不同时，重新导出非常有用。 例如，在这个餐厅比喻中，经营餐厅的人会想到“前台”和“后台”。 但光顾餐厅的顾客可能不会从这些方面考虑餐厅的各个部分。 通过 pub 使用，我们可以使用一种结构编写代码，但公开一种不同的结构。 这样做可以使我们的库组织得井井有条，方便在库上工作的程序员和调用库的程序员。 我们将在第 14 章的“使用 pub use 导出方便的公共 API”部分中查看 pub use 的另一个示例，以及它如何影响你的 crate 文档。

### Using External Packages
In Chapter 2, we programmed a guessing game project that used an external package called **rand** to get random numbers. To use rand in our project, we added this line to Cargo.toml:

在第二章，我们编写guessing游戏项目，使用了rand包创建一个随机数。为了在本项目中使用rand,我们在Cargo.toml添加一行：

Filename: Cargo.toml
```
rand = "0.8.5"

```
Adding rand as a dependency in Cargo.toml tells Cargo to download the rand package and any dependencies from crates.io and make rand available to our project.

Then, to bring rand definitions into the scope of our package, we added a use line starting with the name of the crate, rand, and listed the items we wanted to bring into scope. Recall that in the “Generating a Random Number” section in Chapter 2, we brought the Rng trait into scope and called the rand::thread_rng function:
在 Cargo.toml 中添加 rand 作为依赖项，告诉 Cargo 从 crates.io 下载 rand 包和任何依赖项，并使 rand 可用于我们的项目。

然后，为了将 rand 定义纳入我们的包的范围内，我们添加了一个以 crate 名称 rand 开头的 use 行，并列出了我们想要纳入范围的项目。 回想一下，在第 2 章的“生成随机数”部分中，我们将 Rng 特征引入作用域并调用 rand::thread_rng 函数：
```
use std::io;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1..=100);

    println!("The secret number is: {secret_number}");

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin()
        .read_line(&mut guess)
        .expect("Failed to read line");

    println!("You guessed: {guess}");
}
```

Members of the Rust community have made many packages available at crates.io, and pulling any of them into your package involves these same steps: listing them in your package’s Cargo.toml file and using use to bring items from their crates into scope.

Note that the standard std library is also a crate that’s external to our package. Because the standard library is shipped with the Rust language, we don’t need to change Cargo.toml to include std. But we do need to refer to it with use to bring items from there into our package’s scope. For example, with HashMap we would use this line:

Rust 社区的成员在 crates.io 上提供了许多包，将它们中的任何一个拉入您的包中都涉及以下相同的步骤：将它们列在包的 Cargo.toml 文件中，并使用 use 将项目从其 crate 引入范围。

请注意，标准 std 库也是我们包外部的一个Crate。 因为标准库是随 Rust 语言一起提供的，所以我们不需要更改 Cargo.toml 来包含 std。 但我们确实需要引用它并使用它来将项目从那里带入我们的包的范围。 例如，对于 HashMap，我们将使用这一行：

```
#![allow(unused)]
fn main() {
use std::collections::HashMap;
}
```
This is an absolute path starting with std, the name of the standard library crate.
### 使用嵌套路径清理大型使用列表(Using Nested Paths to Clean up Large use Lists)
If we’re using multiple items defined in the same crate or same module, listing each item on its own line can take up a lot of vertical space in our files. For example, these two use statements we had in the Guessing Game in Listing 2-4 bring items from std into scope:

Filename: src/main.rs  
如果我们使用在同一板条箱或同一模块中定义的多个项目，则将每个项目单独列出可能会占用文件中的大量垂直空间。 例如，清单 2-4 中的猜谜游戏中的这两个 use 语句将 std 中的项带入作用域：

文件名：src/main.rs

```
use rand::Rng;
// --snip--
use std::cmp::Ordering;
use std::io;
// --snip--

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1..=100);

    println!("The secret number is: {secret_number}");

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin()
        .read_line(&mut guess)
        .expect("Failed to read line");

    println!("You guessed: {guess}");

    match guess.cmp(&secret_number) {
        Ordering::Less => println!("Too small!"),
        Ordering::Greater => println!("Too big!"),
        Ordering::Equal => println!("You win!"),
    }
}
```
Instead, we can use nested paths to bring the same items into scope in one line. We do this by specifying the common part of the path, followed by two colons, and then curly brackets around a list of the parts of the paths that differ, as shown in Listing 7-18.
相反，我们可以使用嵌套路径将相同的项目放入一行中的范围内。 为此，我们指定路径的公共部分，后跟两个冒号，然后用大括号括住路径不同部分的列表，如清单 7-18 所示。
Filename: src/main.rs
```
use rand::Rng;
// --snip--
use std::{cmp::Ordering, io};
// --snip--

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1..=100);

    println!("The secret number is: {secret_number}");

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin()
        .read_line(&mut guess)
        .expect("Failed to read line");

    let guess: u32 = guess.trim().parse().expect("Please type a number!");

    println!("You guessed: {guess}");

    match guess.cmp(&secret_number) {
        Ordering::Less => println!("Too small!"),
        Ordering::Greater => println!("Too big!"),
        Ordering::Equal => println!("You win!"),
    }
}
```
Listing 7-18: Specifying a nested path to bring multiple items with the same prefix into scope

In bigger programs, bringing many items into scope from the same crate or module using nested paths can reduce the number of separate use statements needed by a lot!

We can use a nested path at any level in a path, which is useful when combining two use statements that share a subpath. For example, Listing 7-19 shows two use statements: one that brings std::io into scope and one that brings std::io::Write into scope.

Filename: src/lib.rs

示例 7-18：指定嵌套路径以将具有相同前缀的多个项目引入作用域

在较大的程序中，使用嵌套路径将同一包或模块中的许多项纳入范围可以大大减少所需的单独 use 语句的数量！

我们可以在路径中的任何级别使用嵌套路径，这在组合共享子路径的两个 use 语句时非常有用。 例如，清单 7-19 显示了两种 use 语句：一种将 std::io 引入作用域，另一种将 std::io::Write 引入作用域。

文件名：src/lib.rs
```
use std::io;
use std::io::Write;
```
Listing 7-19: Two use statements where one is a subpath of the other

The common part of these two paths is std::io, and that’s the complete first path. To merge these two paths into one use statement, we can use self in the nested path, as shown in Listing 7-20.

Filename: src/lib.rs
```
use std::io::{self, Write};
```
Listing 7-20: Combining the paths in Listing 7-19 into one use statement

This line brings std::io and std::io::Write into scope.

### 全局操作(The Glob Operator)
If we want to bring all public items defined in a path into scope, we can specify that path followed by the * glob operator:  
如果我们想要将路径中定义的所有公共项纳入作用域，我们可以指定该路径，后跟 * glob 运算符：

```
#![allow(unused)]
fn main() {
use std::collections::*;
}
```
This use statement brings all public items defined in std::collections into the current scope. Be careful when using the glob operator! Glob can make it harder to tell what names are in scope and where a name used in your program was defined.

The glob operator is often used when testing to bring everything under test into the tests module; we’ll talk about that in the “How to Write Tests” section in Chapter 11. The glob operator is also sometimes used as part of the prelude pattern: see the standard library documentation for more information on that pattern.

此 use 语句将 std::collections 中定义的所有公共项带入当前范围。 使用 glob 运算符时要小心！ Glob 会使判断哪些名称在范围内以及程序中使用的名称的定义位置变得更加困难。

测试时经常使用 glob 运算符，将所有被测试的内容放入测试模块中； 我们将在第 11 章的“如何编写测试”部分中讨论这一点。 glob 运算符有时也用作前奏模式的一部分：有关该模式的更多信息，请参阅标准库文档。

## 7.5 将模块分成不同的文件
So far, all the examples in this chapter defined multiple modules in one file. When modules get large, you might want to move their definitions to a separate file to make the code easier to navigate.

For example, let’s start from the code in Listing 7-17 that had multiple restaurant modules. We’ll extract modules into files instead of having all the modules defined in the crate root file. In this case, the crate root file is src/lib.rs, but this procedure also works with binary crates whose crate root file is src/main.rs.

First, we’ll extract the front_of_house module to its own file. Remove the code inside the curly brackets for the front_of_house module, leaving only the mod front_of_house; declaration, so that src/lib.rs contains the code shown in Listing 7-21. Note that this won’t compile until we create the src/front_of_house.rs file in Listing 7-22.

Filename: src/lib.rs

到目前为止，本章中的所有示例都在一个文件中定义了多个模块。 当模块变大时，您可能希望将其定义移动到单独的文件中，以使代码更易于导航。

例如，让我们从清单 7-17 中具有多个餐厅模块的代码开始。 我们将把模块提取到文件中，而不是在crate根文件中定义所有模块。 在本例中，crate 根文件为 src/lib.rs，但此过程也适用于 crate 根文件为 src/main.rs 的二进制 crate。

首先，我们将 front_of_house 模块提取到它自己的文件中。 删除front_of_house模块大括号内的代码，只留下mod front_of_house； 声明，以便 src/lib.rs 包含清单 7-21 所示的代码。 请注意，直到我们创建清单 7-22 中的 src/front_of_house.rs 文件后，它才会编译。

文件名：src/lib.rs

```
mod front_of_house;

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```
示例 7-21：声明 front_of_house 模块，其主体位于 src/front_of_house.rs 中


Next, place the code that was in the curly brackets into a new file named src/front_of_house.rs, as shown in Listing 7-22. The compiler knows to look in this file because it came across the module declaration in the crate root with the name front_of_house.


接下来，将大括号中的代码放入名为 src/front_of_house.rs 的新文件中，如清单 7-22 所示。 编译器知道要查找此文件，因为它在 crate 根中遇到了名为 front_of_house 的模块声明。

文件名：src/front_of_house.rs
```
pub mod hosting {
    pub fn add_to_waitlist() {}
}
```

示例 7-22：src/front_of_house.rs 中 front_of_house 模块内的定义


Note that you only need to load a file using a mod declaration once in your module tree. Once the compiler knows the file is part of the project (and knows where in the module tree the code resides because of where you’ve put the mod statement), other files in your project should refer to the loaded file’s code using a path to where it was declared, as covered in the “Paths for Referring to an Item in the Module Tree” section. In other words, mod is not an “include” operation that you may have seen in other programming languages.

Next, we’ll extract the hosting module to its own file. The process is a bit different because hosting is a child module of front_of_house, not of the root module. We’ll place the file for hosting in a new directory that will be named for its ancestors in the module tree, in this case src/front_of_house/.

To start moving hosting, we change src/front_of_house.rs to contain only the declaration of the hosting module:

请注意，您只需在模块树中使用 mod 声明加载一次文件。 一旦编译器知道该文件是项目的一部分（并且知道代码在模块树中的位置，因为您放置了 mod 语句），项目中的其他文件应该使用以下路径引用加载的文件的代码： 声明它的位置，如“引用模块树中的项目的路径”部分所述。 换句话说，mod 不是您在其他编程语言中看到的“包含”操作。

接下来，我们将把hosting模块提取到它自己的文件中。 该过程有点不同，因为hostring是 front_of_house 的子模块，而不是根模块的子模块。 我们将把要hosting的文件放置在一个新目录中，该目录将以其在模块树中的祖先命名，在本例中为 src/front_of_house/。

要开始移动hosting，我们更改 src/front_of_house.rs 以仅包含hosting模块的声明：

文件名：src/front_of_house.rs
```
pub mod hosting;
```
Then we create a src/front_of_house directory and a file hosting.rs to contain the definitions made in the hosting module:

Filename: src/front_of_house/hosting.rs

```
pub fn add_to_waitlist() {}
```
If we instead put hosting.rs in the src directory, the compiler would expect the hosting.rs code to be in a hosting module declared in the crate root, and not declared as a child of the front_of_house module. The compiler’s rules for which files to check for which modules’ code means the directories and files more closely match the module tree.

如果我们将hosting.rs放在src目录中，编译器会期望hosting.rs代码位于crate根中声明的hosting模块中，而不是声明为front_of_house模块的子模块。 编译器关于哪些文件检查哪些模块代码的规则意味着目录和文件与模块树更加匹配。

------------------------------------------------------------
**Alternate File Paths**

**So far we’ve covered the most idiomatic file paths the Rust compiler uses, but Rust also supports an older style of file path. For a module named front_of_house declared in the crate root, the compiler will look for the module’s code in:

+ src/front_of_house.rs (what we covered)
+ src/front_of_house/mod.rs (older style, still supported path)

For a module named hosting that is a submodule of front_of_house, the compiler will look for the module’s code in:

+ src/front_of_house/hosting.rs (what we covered)
+ src/front_of_house/hosting/mod.rs (older style, still supported path)

If you use both styles for the same module, you’ll get a compiler error. Using a mix of both styles for different modules in the same project is allowed, but might be confusing for people navigating your project.

The main downside to the style that uses files named mod.rs is that your project can end up with many files named mod.rs, which can get confusing when you have them open in your editor at the same time.

到目前为止，我们已经介绍了 Rust 编译器使用的最惯用的文件路径，但 Rust 还支持旧样式的文件路径。 对于在 crate 根中声明的名为 front_of_house 的模块，编译器将在以下位置查找该模块的代码：

+ src/front_of_house.rs（我们涵盖的内容）
+ src/front_of_house/mod.rs（较旧的样式，仍然受支持的路径）
对于名为 Hosting 的模块，它是 front_of_house 的子模块，编译器将在以下位置查找该模块的代码：

+ src/front_of_house/hosting.rs（我们介绍的内容）
+ src/front_of_house/hosting/mod.rs（较旧的样式，仍然受支持的路径）
如果您对同一模块使用两种样式，则会出现编译器错误。 允许在同一项目中的不同模块中混合使用两种样式，但可能会让浏览项目的人感到困惑。

使用名为 mod.rs 的文件的样式的主要缺点是，您的项目最终可能会包含许多名为 mod.rs 的文件，当您同时在编辑器中打开它们时，这可能会造成混乱。**

--------------------------------------------------------

We’ve moved each module’s code to a separate file, and the module tree remains the same. The function calls in eat_at_restaurant will work without any modification, even though the definitions live in different files. This technique lets you move modules to new files as they grow in size.

Note that the pub use crate::front_of_house::hosting statement in src/lib.rs also hasn’t changed, nor does use have any impact on what files are compiled as part of the crate. The mod keyword declares modules, and Rust looks in a file with the same name as the module for the code that goes into that module.

我们已将每个模块的代码移至单独的文件中，并且模块树保持不变。 eat_at_restaurant 中的函数调用无需任何修改即可工作，即使定义位于不同的文件中。 此技术允许您在模块大小增长时将模块移动到新文件中。

请注意，src/lib.rs 中的 pub use crate::front_of_house::hosting 语句也没有更改，use 对哪些文件被编译为 crate 的一部分也没有任何影响。 mod 关键字声明模块，Rust 在与模块同名的文件中查找进入该模块的代码。

## Summary
Rust lets you split a package into multiple crates and a crate into modules so you can refer to items defined in one module from another module. You can do this by specifying absolute or relative paths. These paths can be brought into scope with a use statement so you can use a shorter path for multiple uses of the item in that scope. Module code is private by default, but you can make definitions public by adding the pub keyword.

In the next chapter, we’ll look at some collection data structures in the standard library that you can use in your neatly organized code.

Rust 允许您将一个包拆分为多个crate，并将一个crate拆分为模块，以便您可以从另一个模块引用一个模块中定义的项。 您可以通过指定绝对或相对路径来完成此操作。 可以使用 use 语句将这些路径引入范围，以便您可以使用较短的路径在该范围内多次使用该项目。 模块代码默认是私有的，但您可以通过添加 pub 关键字将定义公开。

在下一章中，我们将介绍标准库中的一些集合数据结构，您可以在组织整齐的代码中使用它们。