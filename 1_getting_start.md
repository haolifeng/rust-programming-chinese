# Getting Started

Let’s start your Rust journey! There’s a lot to learn, but every journey starts somewhere. In this chapter, we’ll discuss:

+ Installing Rust on Linux, macOS, and Windows
+ Writing a program that prints Hello, world!
+ Using cargo, Rust’s package manager and build system

让我们开始您的 Rust 之旅吧！ 有很多东西需要学习，但每一段旅程都有一个起点。 在本章中，我们将讨论：

+ 在 Linux、macOS 和 Windows 上安装 Rust
+ 编写一个打印 Hello, world! 的程序
+ 使用 Cargo、Rust 的包管理器和构建系统

## 1.1 安装
The first step is to install Rust. We’ll download Rust through rustup, a command line tool for managing Rust versions and associated tools. You’ll need an internet connection for the download.

第一步是安装 Rust。 我们将通过 rustup 下载 Rust，这是一个用于管理 Rust 版本和相关工具的命令行工具。 您需要连接互联网才能下载。
``````
Note: If you prefer not to use rustup for some reason, please see the Other Rust Installation Methods page for more options.
``````
The following steps install the latest stable version of the Rust compiler. Rust’s stability guarantees ensure that all the examples in the book that compile will continue to compile with newer Rust versions. The output might differ slightly between versions because Rust often improves error messages and warnings. In other words, any newer, stable version of Rust you install using these steps should work as expected with the content of this book.

以下步骤安装最新稳定版本的 Rust 编译器。 Rust 的稳定性保证确保书中所有编译的示例都将继续使用较新的 Rust 版本进行编译。 版本之间的输出可能略有不同，因为 Rust 通常会改进错误消息和警告。 换句话说，使用这些步骤安装的任何较新、稳定的 Rust 版本都应该按照本书的内容按预期工作。

``````
Command Line Notation
In this chapter and throughout the book, we’ll show some commands used in the terminal. Lines that you should enter in a terminal all start with $. You don’t need to type the $ character; it’s the command line prompt shown to indicate the start of each command. Lines that don’t start with $ typically show the output of the previous command. Additionally, PowerShell-specific examples will use > rather than $.
``````
### Installing rustup on Linux or macOS
If you’re using Linux or macOS, open a terminal and enter the following command:
``````
$ curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
``````
The command downloads a script and starts the installation of the rustup tool, which installs the latest stable version of Rust. You might be prompted for your password. If the install is successful, the following line will appear:
``````
Rust is installed now. Great!
``````
You will also need a linker, which is a program that Rust uses to join its compiled outputs into one file. It is likely you already have one. If you get linker errors, you should install a C compiler, which will typically include a linker. A C compiler is also useful because some common Rust packages depend on C code and will need a C compiler.

您还需要一个链接器，这是 Rust 用来将其编译输出连接到一个文件中的程序。 您很可能已经拥有一个。 如果出现链接器错误，则应安装 C 编译器，该编译器通常包含链接器。 C 编译器也很有用，因为一些常见的 Rust 包依赖于 C 代码，并且需要 C 编译器。

On macOS, you can get a C compiler by running:
``````
$ xcode-select --install
``````
Linux users should generally install GCC or Clang, according to their distribution’s documentation. For example, if you use Ubuntu, you can install the build-essential package.

Linux 用户通常应该根据其发行版的文档安装 GCC 或 Clang。 例如，如果您使用 Ubuntu，则可以安装 build-essential 包。

### Installing rustup on Windows
On Windows, go to https://www.rust-lang.org/tools/install and follow the instructions for installing Rust. At some point in the installation, you’ll receive a message explaining that you’ll also need the MSVC build tools for Visual Studio 2013 or later.

To acquire the build tools, you’ll need to install Visual Studio 2022. When asked which workloads to install, include:

+ “Desktop Development with C++”
+ The Windows 10 or 11 SDK
+ The English language pack component, along with any other language pack of your choosing

The rest of this book uses commands that work in both cmd.exe and PowerShell. If there are specific differences, we’ll explain which to use.

在 Windows 上，请访问 https://www.rust-lang.org/tools/install 并按照安装 Rust 的说明进行操作。 在安装过程中的某个时刻，您将收到一条消息，说明您还需要 Visual Studio 2013 或更高版本的 MSVC 构建工具。

要获取构建工具，您需要安装 Visual Studio 2022。当被询问要安装哪些工作负载时，包括：

+ 使用 C++ 进行桌面开发
+ Windows 10 或 11 SDK
+ 英语语言包组件以及您选择的任何其他语言包

本书的其余部分使用可在 cmd.exe 和 PowerShell 中运行的命令。 如果存在具体差异，我们将解释使用哪个。

### 故障排除(Troubleshooting)
To check whether you have Rust installed correctly, open a shell and enter this line:
``````
$ rustc --version
``````
You should see the version number, commit hash, and commit date for the latest stable version that has been released, in the following format:
``````
rustc x.y.z (abcabcabc yyyy-mm-dd)
``````
If you see this information, you have installed Rust successfully! If you don’t see this information, check that Rust is in your %PATH% system variable as follows.

In Windows CMD, use:
``````
> echo %PATH%
``````
In PowerShell, use:
``````
> echo $env:Path
``````
In Linux and macOS, use:
``````
$ echo $PATH
``````
If that’s all correct and Rust still isn’t working, there are a number of places you can get help. Find out how to get in touch with other Rustaceans (a silly nickname we call ourselves) on the community page.

### Updating and Uninstalling
Once Rust is installed via rustup, updating to a newly released version is easy. From your shell, run the following update script:
``````
$ rustup update
``````
To uninstall Rust and rustup, run the following uninstall script from your shell:
``````
$ rustup self uninstall
``````
### Local Documentation
The installation of Rust also includes a local copy of the documentation so that you can read it offline. Run rustup doc to open the local documentation in your browser.

Any time a type or function is provided by the standard library and you’re not sure what it does or how to use it, use the application programming interface (API) documentation to find out!

Rust 的安装还包括文档的本地副本，以便您可以离线阅读。 运行 rustup doc 以在浏览器中打开本地文档。

每当标准库提供类型或函数并且您不确定它的作用或如何使用它时，请使用应用程序编程接口 (API) 文档来查找！

## 1.2 Hello, world
Now that you’ve installed Rust, it’s time to write your first Rust program. It’s traditional when learning a new language to write a little program that prints the text Hello, world! to the screen, so we’ll do the same here!

现在您已经安装了 Rust，是时候编写您的第一个 Rust 程序了。 学习一门新语言时，传统做法是编写一个小程序来打印文本“Hello, world!”。 到屏幕上，所以我们在这里也做同样的事情！

``````
Note: This book assumes basic familiarity with the command line. Rust makes no specific demands about your editing or tooling or where your code lives, so if you prefer to use an integrated development environment (IDE) instead of the command line, feel free to use your favorite IDE. Many IDEs now have some degree of Rust support; check the IDE’s documentation for details. The Rust team has been focusing on enabling great IDE support via rust-analyzer. See Appendix D for more details.
``````
### Creating a Project Directory
You’ll start by making a directory to store your Rust code. It doesn’t matter to Rust where your code lives, but for the exercises and projects in this book, we suggest making a projects directory in your home directory and keeping all your projects there.

Open a terminal and enter the following commands to make a projects directory and a directory for the “Hello, world!” project within the projects directory.

您将首先创建一个目录来存储 Rust 代码。 对于 Rust 来说，你的代码所在的位置并不重要，但对于本书中的练习和项目，我们建议在你的主目录中创建一个项目目录，并将所有项目保存在那里。

打开终端并输入以下命令来创建项目目录和“Hello, world!”目录 项目位于项目目录中。

For Linux, macOS, and PowerShell on Windows, enter this:
``````
$ mkdir ~/projects
$ cd ~/projects
$ mkdir hello_world
$ cd hello_world
``````
For Windows CMD, enter this:
``````
> mkdir "%USERPROFILE%\projects"
> cd /d "%USERPROFILE%\projects"
> mkdir hello_world
> cd hello_world
``````
### Writing and Running a Rust Program
Next, make a new source file and call it main.rs. Rust files always end with the .rs extension. If you’re using more than one word in your filename, the convention is to use an underscore to separate them. For example, use hello_world.rs rather than helloworld.rs.

接下来，创建一个新的源文件并将其命名为 main.rs。 Rust 文件始终以 .rs 扩展名结尾。 如果您的文件名中使用了多个单词，则惯例是使用下划线分隔它们。 例如，使用 hello_world.rs 而不是 helloworld.rs。

Now open the main.rs file you just created and enter the code in Listing 1-1.

Filename: main.rs
``````
fn main() {
    println!("Hello, world!");
}
``````
Listing 1-1: A program that prints Hello, world!

Save the file and go back to your terminal window in the ~/projects/hello_world directory. On Linux or macOS, enter the following commands to compile and run the file:
``````
$ rustc main.rs
$ ./main
Hello, world!
``````
On Windows, enter the command .\main.exe instead of ./main:
```
> rustc main.rs
> .\main.exe
Hello, world!
```
Regardless of your operating system, the string Hello, world! should print to the terminal. If you don’t see this output, refer back to the “Troubleshooting” part of the Installation section for ways to get help.

If Hello, world! did print, congratulations! You’ve officially written a Rust program. That makes you a Rust programmer—welcome!

无论您使用什么操作系统，字符串 Hello, world! 应该打印到终端。 如果您没有看到此输出，请参阅安装部分的“故障排除”部分，了解获取帮助的方法。

如果你好，世界！ 打印出来了，恭喜！ 你已经正式编写了一个 Rust 程序。 这使您成为一名 Rust 程序员——欢迎！

### Rust 程序剖析
Let’s review this “Hello, world!” program in detail. Here’s the first piece of the puzzle:
``````
fn main() {

}
``````
These lines define a function named main. The main function is special: it is always the first code that runs in every executable Rust program. Here, the first line declares a function named main that has no parameters and returns nothing. If there were parameters, they would go inside the parentheses ().

The function body is wrapped in {}. Rust requires curly brackets around all function bodies. It’s good style to place the opening curly bracket on the same line as the function declaration, adding one space in between.

这些行定义了一个名为 main 的函数。 main 函数很特殊：它始终是每个可执行 Rust 程序中运行的第一个代码。 这里，第一行声明了一个名为 main 的函数，它没有参数，也不返回任何内容。 如果有参数，则将其放在括号 () 内。

函数体被包裹在{}中。 Rust 要求所有函数体都用大括号括起来。 将左大括号与函数声明放在同一行，并在中间添加一个空格是一种很好的风格。

``````
Note: If you want to stick to a standard style across Rust projects, you can use an automatic formatter tool called rustfmt to format your code in a particular style (more on rustfmt in Appendix D). The Rust team has included this tool with the standard Rust distribution, as rustc is, so it should already be installed on your computer!
``````
The body of the main function holds the following code:
``````
    println!("Hello, world!");
``````
This line does all the work in this little program: it prints text to the screen. There are four important details to notice here.

First, Rust style is to indent with four spaces, not a tab.

Second, println! calls a Rust macro. If it had called a function instead, it would be entered as println (without the !). We’ll discuss Rust macros in more detail in Chapter 19. For now, you just need to know that using a ! means that you’re calling a macro instead of a normal function and that macros don’t always follow the same rules as functions.

Third, you see the "Hello, world!" string. We pass this string as an argument to println!, and the string is printed to the screen.

Fourth, we end the line with a semicolon (;), which indicates that this expression is over and the next one is ready to begin. Most lines of Rust code end with a semicolon.

这一行完成了这个小程序中的所有工作：它将文本打印到屏幕上。 这里有四个重要细节需要注意。

首先，Rust 风格是用四个空格缩进，而不是制表符。

第二，打印！ 调用 Rust 宏。 如果它调用了一个函数，那么它将被输入为 println （不带！）。 我们将在第 19 章中更详细地讨论 Rust 宏。现在，你只需要知道使用 ! 意味着您正在调用宏而不是普通函数，并且宏并不总是遵循与函数相同的规则。

第三，您会看到“Hello, world!” 细绳。 我们将此字符串作为参数传递给 println!，并将该字符串打印到屏幕上。

第四，我们以分号 (;) 结束该行，这表明该表达式已结束，下一个表达式已准备好开始。 大多数 Rust 代码行都以分号结尾。

### Compiling and Running Are Separate Steps
You’ve just run a newly created program, so let’s examine each step in the process.

Before running a Rust program, you must compile it using the Rust compiler by entering the rustc command and passing it the name of your source file, like this:

您刚刚运行了一个新创建的程序，所以让我们检查一下该过程中的每个步骤。

在运行 Rust 程序之前，您必须使用 Rust 编译器来编译它，方法是输入 rustc 命令并向其传递源文件的名称，如下所示：

``````
$ rustc main.rs
``````
If you have a C or C++ background, you’ll notice that this is similar to gcc or clang. After compiling successfully, Rust outputs a binary executable.

On Linux, macOS, and PowerShell on Windows, you can see the executable by entering the ls command in your shell:
``````
$ ls
main  main.rs
``````
On Linux and macOS, you’ll see two files. With PowerShell on Windows, you’ll see the same three files that you would see using CMD. With CMD on Windows, you would enter the following:
``````
> dir /B %= the /B option says to only show the file names =%
main.exe
main.pdb
main.rs
``````
This shows the source code file with the .rs extension, the executable file (main.exe on Windows, but main on all other platforms), and, when using Windows, a file containing debugging information with the .pdb extension. From here, you run the main or main.exe file, like this:
``````
$ ./main # or .\main.exe on Windows
``````
If your main.rs is your “Hello, world!” program, this line prints Hello, world! to your terminal.

If you’re more familiar with a dynamic language, such as Ruby, Python, or JavaScript, you might not be used to compiling and running a program as separate steps. Rust is an ahead-of-time compiled language, meaning you can compile a program and give the executable to someone else, and they can run it even without having Rust installed. If you give someone a .rb, .py, or .js file, they need to have a Ruby, Python, or JavaScript implementation installed (respectively). But in those languages, you only need one command to compile and run your program. Everything is a trade-off in language design.

Just compiling with rustc is fine for simple programs, but as your project grows, you’ll want to manage all the options and make it easy to share your code. Next, we’ll introduce you to the Cargo tool, which will help you write real-world Rust programs.

如果你的 main.rs 是你的“Hello, world!” 程序中，这一行打印 Hello, world! 到您的终端。

如果您更熟悉动态语言，例如 Ruby、Python 或 JavaScript，您可能不习惯将程序编译和运行作为单独的步骤。 Rust 是一种提前编译的语言，这意味着您可以编译程序并将可执行文件提供给其他人，即使没有安装 Rust，他们也可以运行它。 如果您向某人提供 .rb、.py 或 .js 文件，他们需要（分别）安装 Ruby、Python 或 JavaScript 实现。 但在这些语言中，您只需要一个命令来编译和运行您的程序。 一切都是语言设计的权衡。

对于简单的程序来说，使用 rustc 进行编译就足够了，但随着项目的增长，您将希望管理所有选项并轻松共享代码。 接下来，我们将向您介绍 Cargo 工具，它将帮助您编写真实的 Rust 程序。

## 1.3 Hello,Cargo!
Cargo is Rust’s build system and package manager. Most Rustaceans use this tool to manage their Rust projects because Cargo handles a lot of tasks for you, such as building your code, downloading the libraries your code depends on, and building those libraries. (We call the libraries that your code needs dependencies.)

The simplest Rust programs, like the one we’ve written so far, don’t have any dependencies. If we had built the “Hello, world!” project with Cargo, it would only use the part of Cargo that handles building your code. As you write more complex Rust programs, you’ll add dependencies, and if you start a project using Cargo, adding dependencies will be much easier to do.

Because the vast majority of Rust projects use Cargo, the rest of this book assumes that you’re using Cargo too. Cargo comes installed with Rust if you used the official installers discussed in the “Installation” section. If you installed Rust through some other means, check whether Cargo is installed by entering the following in your terminal:

Cargo 是 Rust 的构建系统和包管理器。 大多数 Rustaceans 使用这个工具来管理他们的 Rust 项目，因为 Cargo 会为您处理很多任务，例如构建代码、下载代码所依赖的库以及构建这些库。 （我们称您的代码需要依赖的库。）

最简单的 Rust 程序，就像我们迄今为止编写的程序一样，没有任何依赖项。 如果我们建造了“你好，世界！” 使用 Cargo 进行项目时，它只会使用 Cargo 中处理构建代码的部分。 当你编写更复杂的 Rust 程序时，你将添加依赖项，如果你使用 Cargo 启动一个项目，添加依赖项会更容易。

因为绝大多数 Rust 项目都使用 Cargo，所以本书的其余部分假设您也使用 Cargo。 如果您使用“安装”部分中讨论的官方安装程序，Cargo 会随 Rust 一起安装。 如果您通过其他方式安装了 Rust，请在终端中输入以下内容来检查 Cargo 是否已安装：

``````
$ cargo --version
``````
If you see a version number, you have it! If you see an error, such as command not found, look at the documentation for your method of installation to determine how to install Cargo separately.

如果您看到版本号，则说明您已拥有它！ 如果您看到错误，例如找不到命令，请查看安装方法的文档，以确定如何单独安装 Cargo。

### Creating a Project with Cargo
Let’s create a new project using Cargo and look at how it differs from our original “Hello, world!” project. Navigate back to your projects directory (or wherever you decided to store your code). Then, on any operating system, run the following:

让我们使用 Cargo 创建一个新项目，看看它与我们原来的“Hello, world!”有何不同。 项目。 导航回您的项目目录（或您决定存储代码的任何位置）。 然后，在任何操作系统上运行以下命令：

``````
$ cargo new hello_cargo
$ cd hello_cargo
``````
The first command creates a new directory and project called hello_cargo. We’ve named our project hello_cargo, and Cargo creates its files in a directory of the same name.

Go into the hello_cargo directory and list the files. You’ll see that Cargo has generated two files and one directory for us: a Cargo.toml file and a src directory with a main.rs file inside.

It has also initialized a new Git repository along with a .gitignore file. Git files won’t be generated if you run cargo new within an existing Git repository; you can override this behavior by using cargo new --vcs=git.

第一个命令创建一个名为 hello_cargo 的新目录和项目。 我们将项目命名为 hello_cargo，Cargo 在同名目录中创建其文件。

进入 hello_cargo 目录并列出文件。 你会看到 Cargo 为我们生成了两个文件和一个目录：一个 Cargo.toml 文件和一个 src 目录，里面有一个 main.rs 文件。

它还初始化了一个新的 Git 存储库以及一个 .gitignore 文件。 如果您在现有的 Git 存储库中运行 Cargo new，则不会生成 Git 文件； 您可以使用 Cargo new --vcs=git 覆盖此行为。

``````
Note: Git is a common version control system. You can change cargo new to use a different version control system or no version control system by using the --vcs flag. Run cargo new --help to see the available options.
``````

Open Cargo.toml in your text editor of choice. It should look similar to the code in Listing 1-2.

Filename: Cargo.toml
``````
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
``````
Listing 1-2: Contents of Cargo.toml generated by cargo new

This file is in the TOML (Tom’s Obvious, Minimal Language) format, which is Cargo’s configuration format.

The first line, [package], is a section heading that indicates that the following statements are configuring a package. As we add more information to this file, we’ll add other sections.

The next three lines set the configuration information Cargo needs to compile your program: the name, the version, and the edition of Rust to use. We’ll talk about the edition key in Appendix E.

The last line, [dependencies], is the start of a section for you to list any of your project’s dependencies. In Rust, packages of code are referred to as crates. We won’t need any other crates for this project, but we will in the first project in Chapter 2, so we’ll use this dependencies section then.

该文件采用 TOML（Tom’s Obvious，Minimal Language）格式，这是 Cargo 的配置格式。

第一行 [package] 是节标题，指示以下语句正在配置包。 当我们向此文件添加更多信息时，我们将添加其他部分。

接下来的三行设置 Cargo 编译程序所需的配置信息：名称、版本和要使用的 Rust 版本。 我们将在附录 E 中讨论版本密钥。

最后一行 [dependencies] 是一个部分的开始，用于列出项目的任何依赖项。 在 Rust 中，代码包被称为 crate。 这个项目不需要任何其他 crate，但我们会在第 2 章的第一个项目中使用，所以我们将使用这个依赖项部分。

Now open src/main.rs and take a look:

Filename: src/main.rs
``````
fn main() {
    println!("Hello, world!");
}
``````
Cargo has generated a “Hello, world!” program for you, just like the one we wrote in Listing 1-1! So far, the differences between our project and the project Cargo generated are that Cargo placed the code in the src directory and we have a Cargo.toml configuration file in the top directory.

Cargo expects your source files to live inside the src directory. The top-level project directory is just for README files, license information, configuration files, and anything else not related to your code. Using Cargo helps you organize your projects. There’s a place for everything, and everything is in its place.

If you started a project that doesn’t use Cargo, as we did with the “Hello, world!” project, you can convert it to a project that does use Cargo. Move the project code into the src directory and create an appropriate Cargo.toml file.

Cargo 生成了“Hello, world!” 为您准备的程序，就像我们在清单 1-1 中编写的程序一样！ 到目前为止，我们的项目和 Cargo 生成的项目之间的区别在于 Cargo 将代码放在 src 目录中，并且我们在顶层目录中有一个 Cargo.toml 配置文件。

Cargo 希望您的源文件位于 src 目录中。 顶级项目目录仅用于 README 文件、许可证信息、配置文件以及与代码无关的任何其他内容。 使用 Cargo 可以帮助您组织项目。 一切都有一个地方，一切都在它的位置上。

如果您启动了一个不使用 Cargo 的项目，就像我们对“Hello, world!”所做的那样 项目，您可以将其转换为使用 Cargo 的项目。 将项目代码移至 src 目录并创建适当的 Cargo.toml 文件。

### Building and Running a Cargo Project
Now let’s look at what’s different when we build and run the “Hello, world!” program with Cargo! From your hello_cargo directory, build your project by entering the following command:
``````
$ cargo build
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 2.85 secs
``````
This command creates an executable file in target/debug/hello_cargo (or target\debug\hello_cargo.exe on Windows) rather than in your current directory. Because the default build is a debug build, Cargo puts the binary in a directory named debug. You can run the executable with this command:
``````
$ ./target/debug/hello_cargo # or .\target\debug\hello_cargo.exe on Windows
Hello, world!
``````
If all goes well, Hello, world! should print to the terminal. Running cargo build for the first time also causes Cargo to create a new file at the top level: Cargo.lock. This file keeps track of the exact versions of dependencies in your project. This project doesn’t have dependencies, so the file is a bit sparse. You won’t ever need to change this file manually; Cargo manages its contents for you.

We just built a project with cargo build and ran it with ./target/debug/hello_cargo, but we can also use cargo run to compile the code and then run the resultant executable all in one command:
``````
$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/hello_cargo`
Hello, world!
``````
Using cargo run is more convenient than having to remember to run cargo build and then use the whole path to the binary, so most developers use cargo run.

Notice that this time we didn’t see output indicating that Cargo was compiling hello_cargo. Cargo figured out that the files hadn’t changed, so it didn’t rebuild but just ran the binary. If you had modified your source code, Cargo would have rebuilt the project before running it, and you would have seen this output:

使用cargo run比必须记住运行cargo build然后使用二进制文件的整个路径更方便，因此大多数开发人员使用cargo run。

请注意，这次我们没有看到表明 Cargo 正在编译 hello_cargo 的输出。 Cargo 发现文件没有改变，所以它没有重建，只是运行二进制文件。 如果您修改了源代码，Cargo 将在运行之前重新构建项目，并且您将看到以下输出：

``````
$ cargo run
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.33 secs
     Running `target/debug/hello_cargo`
Hello, world!
``````
Cargo also provides a command called cargo check. This command quickly checks your code to make sure it compiles but doesn’t produce an executable:
``````
$ cargo check
   Checking hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32 secs
``````
Why would you not want an executable? Often, cargo check is much faster than cargo build because it skips the step of producing an executable. If you’re continually checking your work while writing the code, using cargo check will speed up the process of letting you know if your project is still compiling! As such, many Rustaceans run cargo check periodically as they write their program to make sure it compiles. Then they run cargo build when they’re ready to use the executable.

Let’s recap what we’ve learned so far about Cargo:

+ We can create a project using cargo new.
+ We can build a project using cargo build.
+ We can build and run a project in one step using cargo run.
+ We can build a project without producing a binary to check for errors using cargo check.
+ Instead of saving the result of the build in the same directory as our code, Cargo stores it in the target/debug directory.

An additional advantage of using Cargo is that the commands are the same no matter which operating system you’re working on. So, at this point, we’ll no longer provide specific instructions for Linux and macOS versus Windows.

为什么你不想要一个可执行文件？ 通常，货物检查比货物构建快得多，因为它跳过了生成可执行文件的步骤。 如果您在编写代码时不断检查您的工作，那么使用货物检查将加快让您知道项目是否仍在编译的过程！ 因此，许多 Rustaceans 在编写程序时定期运行货物检查以确保其编译。 然后，当他们准备好使用可执行文件时，他们会运行Cargo build。

让我们回顾一下到目前为止我们所了解的有关 Cargo 的知识：

+ 我们可以使用cargo new 创建一个项目。
+ 我们可以使用cargo build 来构建一个项目。
+我们可以使用cargo run一步构建并运行一个项目。
+ 我们可以构建一个项目，而无需生成二进制文件来使用货物检查来检查错误。
+ Cargo 没有将构建结果保存在与我们的代码相同的目录中，而是将其存储在 target/debug 目录中。

使用 Cargo 的另一个优点是，无论您使用哪种操作系统，命令都是相同的。 因此，目前我们将不再提供针对 Linux 和 macOS 与 Windows 的具体说明。

### Building for Release
When your project is finally ready for release, you can use cargo build --release to compile it with optimizations. This command will create an executable in target/release instead of target/debug. The optimizations make your Rust code run faster, but turning them on lengthens the time it takes for your program to compile. This is why there are two different profiles: one for development, when you want to rebuild quickly and often, and another for building the final program you’ll give to a user that won’t be rebuilt repeatedly and that will run as fast as possible. If you’re benchmarking your code’s running time, be sure to run cargo build --release and benchmark with the executable in target/release.

当您的项目最终准备好发布时，您可以使用 Cargo build --release 对其进行优化编译。 此命令将在 target/release 而不是 target/debug 中创建可执行文件。 这些优化使您的 Rust 代码运行得更快，但打开它们会延长程序编译所需的时间。 这就是为什么有两种不同的配置文件：一种用于开发，当您想要快速且频繁地重建时，另一种用于构建最终程序，您将提供给用户，该程序不会重复重建并且运行速度与 可能的。 如果您正在对代码的运行时间进行基准测试，请务必运行 Cargo build --release 并使用 target/release 中的可执行文件进行基准测试。

### Cargo as Convention
With simple projects, Cargo doesn’t provide a lot of value over just using rustc, but it will prove its worth as your programs become more intricate. Once programs grow to multiple files or need a dependency, it’s much easier to let Cargo coordinate the build.

Even though the hello_cargo project is simple, it now uses much of the real tooling you’ll use in the rest of your Rust career. In fact, to work on any existing projects, you can use the following commands to check out the code using Git, change to that project’s directory, and build:

对于简单的项目，Cargo 并不能提供比仅使用 rustc 更大的价值，但随着你的程序变得更加复杂，它会证明它的价值。 一旦程序增长到多个文件或需要依赖项，让 Cargo 协调构建就会容易得多。

尽管 hello_cargo 项目很简单，但它现在使用了您在 Rust 职业生涯的其余部分中将使用的大部分真实工具。 事实上，要处理任何现有项目，您可以使用以下命令通过 Git 检查代码，更改到该项目的目录，然后构建：

``````
$ git clone example.org/someproject
$ cd someproject
$ cargo build
``````
For more information about Cargo, check out its documentation <https://doc.rust-lang.org/cargo/>.


## 1.4 总结

You’re already off to a great start on your Rust journey! In this chapter, you’ve learned how to:

+ Install the latest stable version of Rust using rustup
+ Update to a newer Rust version
+ Open locally installed documentation
+ Write and run a “Hello, world!” program using rustc directly
+ Create and run a new project using the conventions of Cargo

This is a great time to build a more substantial program to get used to reading and writing Rust code. So, in Chapter 2, we’ll build a guessing game program. If you would rather start by learning how common programming concepts work in Rust, see Chapter 3 and then return to Chapter 2.

您的 Rust 之旅已经有了一个良好的开端！ 在本章中，您学习了如何：

+ 使用 rustup 安装最新稳定版本的 Rust
+ 更新到较新的 Rust 版本
+ 打开本地安装的文档
+ 编写并运行“Hello, world!” 直接使用rustc进行编程
+ 使用 Cargo 约定创建并运行一个新项目

现在是构建一个更充实的程序以习惯阅读和编写 Rust 代码的好时机。 因此，在第 2 章中，我们将构建一个猜谜游戏程序。 如果您想从学习 Rust 中常见编程概念的工作原理开始，请参阅第 3 章，然后返回第 2 章。