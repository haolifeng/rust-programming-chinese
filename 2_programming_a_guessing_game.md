# 2 猜谜游戏
Let’s jump into Rust by working through a hands-on project together! This chapter introduces you to a few common Rust concepts by showing you how to use them in a real program. You’ll learn about let, match, methods, associated functions, external crates, and more! In the following chapters, we’ll explore these ideas in more detail. In this chapter, you’ll just practice the fundamentals.

We’ll implement a classic beginner programming problem: a guessing game. Here’s how it works: the program will generate a random integer between 1 and 100. It will then prompt the player to enter a guess. After a guess is entered, the program will indicate whether the guess is too low or too high. If the guess is correct, the game will print a congratulatory message and exit.

让我们一起完成一个实践项目，开始学习 Rust！ 本章通过向您展示如何在实际程序中使用它们来向您介绍一些常见的 Rust 概念。 您将了解 let、match、方法、关联函数、外部包等等！ 在接下来的章节中，我们将更详细地探讨这些想法。 在本章中，您将只练习基础知识。

我们将实现一个经典的初学者编程问题：猜谜游戏。 它的工作原理如下：程序将生成一个 1 到 100 之间的随机整数。然后它会提示玩家输入猜测值。 输入猜测后，程序将指示猜测是否太低或太高。 如果猜测正确，游戏将打印一条祝贺消息并退出。

## Setting Up a New Project
To set up a new project, go to the projects directory that you created in Chapter 1 and make a new project using Cargo, like so:
``````
$ cargo new guessing_game
$ cd guessing_game
``````
The first command, cargo new, takes the name of the project (guessing_game) as the first argument. The second command changes to the new project’s directory.

Look at the generated Cargo.toml file:

Filename: Cargo.toml
``````
[package]
name = "guessing_game"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
``````
As you saw in Chapter 1, cargo new generates a “Hello, world!” program for you. Check out the src/main.rs file:

Filename: src/main.rs
``````
fn main() {
    println!("Hello, world!");
}
``````
Now let’s compile this “Hello, world!” program and run it in the same step using the cargo run command:
``````
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 1.50s
     Running `target/debug/guessing_game`
Hello, world!
``````
The run command comes in handy when you need to rapidly iterate on a project, as we’ll do in this game, quickly testing each iteration before moving on to the next one.

Reopen the src/main.rs file. You’ll be writing all the code in this file.

### Processing a Guess
The first part of the guessing game program will ask for user input, process that input, and check that the input is in the expected form. To start, we’ll allow the player to input a guess. Enter the code in Listing 2-1 into src/main.rs.

猜谜游戏程序的第一部分将要求用户输入，处理该输入，并检查输入是否为预期形式。 首先，我们将允许玩家输入猜测。 将清单 2-1 中的代码输入到 src/main.rs 中。

Filename: src/main.rs
``````
use std::io;

fn main() {
    println!("Guess the number!");

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin()
        .read_line(&mut guess)
        .expect("Failed to read line");

    println!("You guessed: {guess}");
}
``````
Listing 2-1: Code that gets a guess from the user and prints it

This code contains a lot of information, so let’s go over it line by line. To obtain user input and then print the result as output, we need to bring the io input/output library into scope. The io library comes from the standard library, known as std:

这段代码包含了很多信息，所以让我们逐行看一下。 为了获取用户输入然后将结果打印为输出，我们需要将 io 输入/输出库纳入范围。 io 库来自标准库，称为 std：
``````
use std::io;
``````
By default, Rust has a set of items defined in the standard library that it brings into the scope of every program. This set is called the prelude, and you can see everything in it in the standard library documentation.

If a type you want to use isn’t in the prelude, you have to bring that type into scope explicitly with a use statement. Using the std::io library provides you with a number of useful features, including the ability to accept user input.

默认情况下，Rust 在标准库中定义了一组项目，并将其引入每个程序的范围内。 这套称为前奏，你可以在标准库文档中看到其中的所有内容。

如果您要使用的类型不在前奏中，则必须使用 use 语句显式将该类型带入作用域。 使用 std::io 库为您提供了许多有用的功能，包括接受用户输入的能力。

As you saw in Chapter 1, the main function is the entry point into the program:
``````
fn main() {
``````
The fn syntax declares a new function; the parentheses, (), indicate there are no parameters; and the curly bracket, {, starts the body of the function.

fn 语法声明一个新函数； 括号()表示没有参数； 大括号 { 开始函数体。

As you also learned in Chapter 1, println! is a macro that prints a string to the screen:
``````
    println!("Guess the number!");

    println!("Please input your guess.");
``````
This code is printing a prompt stating what the game is and requesting input from the user.

### Storing Values with Variables
Next, we’ll create a variable to store the user input, like this:
``````
    let mut guess = String::new();
``````
Now the program is getting interesting! There’s a lot going on in this little line. We use the let statement to create the variable. Here’s another example:
``````
let apples = 5;
``````
This line creates a new variable named apples and binds it to the value 5. In Rust, variables are immutable by default, meaning once we give the variable a value, the value won’t change. We’ll be discussing this concept in detail in the “Variables and Mutability” section in Chapter 3. To make a variable mutable, we add mut before the variable name:

这一行创建了一个名为 apples 的新变量，并将其绑定到值 5。在 Rust 中，变量默认是不可变的，这意味着一旦我们给变量赋予了值，该值就不会改变。 我们将在第 3 章的“变量和可变性”部分详细讨论这个概念。为了使变量可变，我们在变量名前添加 mut：

``````
let apples = 5; // immutable
let mut bananas = 5; // mutable
``````
Note: The // syntax starts a comment that continues until the end of the line. Rust ignores everything in comments. We’ll discuss comments in more detail in Chapter 3.

Returning to the guessing game program, you now know that let mut guess will introduce a mutable variable named guess. The equal sign (=) tells Rust we want to bind something to the variable now. On the right of the equal sign is the value that guess is bound to, which is the result of calling String::new, a function that returns a new instance of a String. String is a string type provided by the standard library that is a growable, UTF-8 encoded bit of text.

The :: syntax in the ::new line indicates that new is an associated function of the String type. An associated function is a function that’s implemented on a type, in this case String. This new function creates a new, empty string. You’ll find a new function on many types because it’s a common name for a function that makes a new value of some kind.

In full, the let mut guess = String::new(); line has created a mutable variable that is currently bound to a new, empty instance of a String. Whew!

注意： // 语法开始一个注释，一直持续到该行末尾。 Rust 会忽略注释中的所有内容。 我们将在第 3 章中更详细地讨论注释。

回到猜谜游戏程序，您现在知道 let mut Guess 将引入一个名为 Guess 的可变变量。 等号 (=) 告诉 Rust 我们现在想要将某些内容绑定到变量。 等号右侧是猜测所绑定的值，它是调用 String::new（一个返回 String 的新实例的函数）的结果。 String 是标准库提供的字符串类型，它是可增长的 UTF-8 编码文本位。

::new 行中的 :: 语法表明 new 是 String 类型的关联函数。 关联函数是在类型（本例中为 String）上实现的函数。 这个新函数创建一个新的空字符串。 您会在许多类型上找到新函数，因为它是产生某种新值的函数的通用名称。

完整来说，let mut Guess = String::new(); line 创建了一个可变变量，该变量当前绑定到一个新的空字符串实例。 哇！

### Receiving User Input
Recall that we included the input/output functionality from the standard library with use std::io; on the first line of the program. Now we’ll call the stdin function from the io module, which will allow us to handle user input:

回想一下，我们通过 use std::io; 包含了标准库中的输入/输出功能。 在程序的第一行。 现在我们将从 io 模块调用 stdin 函数，这将允许我们处理用户输入：

``````
    io::stdin()
        .read_line(&mut guess)
``````
If we hadn’t imported the io library with use std::io; at the beginning of the program, we could still use the function by writing this function call as std::io::stdin. The stdin function returns an instance of std::io::Stdin, which is a type that represents a handle to the standard input for your terminal.

Next, the line .read_line(&mut guess) calls the read_line method on the standard input handle to get input from the user. We’re also passing &mut guess as the argument to read_line to tell it what string to store the user input in. The full job of read_line is to take whatever the user types into standard input and append that into a string (without overwriting its contents), so we therefore pass that string as an argument. The string argument needs to be mutable so the method can change the string’s content.

The & indicates that this argument is a reference, which gives you a way to let multiple parts of your code access one piece of data without needing to copy that data into memory multiple times. References are a complex feature, and one of Rust’s major advantages is how safe and easy it is to use references. You don’t need to know a lot of those details to finish this program. For now, all you need to know is that, like variables, references are immutable by default. Hence, you need to write &mut guess rather than &guess to make it mutable. (Chapter 4 will explain references more thoroughly.)

如果我们没有使用 use std::io; 导入 io 库 在程序开始时，我们仍然可以通过将此函数调用编写为 std::io::stdin 来使用该函数。 stdin 函数返回 std::io::Stdin 的实例，它是表示终端标准输入句柄的类型。

接下来，行 .read_line(&mut Guess) 调用标准输入句柄上的 read_line 方法以获取用户的输入。 我们还将 &mut suggest 作为参数传递给 read_line，告诉它将用户输入存储在哪个字符串中。 read_line 的完整工作是将用户输入到标准输入中的所有内容并将其附加到字符串中（而不覆盖其内容） ），因此我们将该字符串作为参数传递。 字符串参数需要是可变的，以便该方法可以更改字符串的内容。

& 表示该参数是一个引用，它为您提供了一种让代码的多个部分访问一条数据的方法，而无需多次将该数据复制到内存中。 引用是一项复杂的功能，Rust 的主要优点之一是使用引用的安全性和易用性。 您不需要知道很多细节来完成这个程序。 现在，您需要知道的是，与变量一样，默认情况下引用是不可变的。 因此，您需要编写 &mut Guess 而不是 &Guess 来使其可变。 （第 4 章将更彻底地解释参考文献。）

### Handling Potential Failure with Result
We’re still working on this line of code. We’re now discussing a third line of text, but note that it’s still part of a single logical line of code. The next part is this method:
``````
        .expect("Failed to read line");
``````
We could have written this code as:
``````
io::stdin().read_line(&mut guess).expect("Failed to read line");
``````
However, one long line is difficult to read, so it’s best to divide it. It’s often wise to introduce a newline and other whitespace to help break up long lines when you call a method with the .method_name() syntax. Now let’s discuss what this line does.

As mentioned earlier, read_line puts whatever the user enters into the string we pass to it, but it also returns a Result value. Result is an enumeration, often called an enum, which is a type that can be in one of multiple possible states. We call each possible state a variant.

Chapter 6 will cover enums in more detail. The purpose of these Result types is to encode error-handling information.

Result’s variants are Ok and Err. The Ok variant indicates the operation was successful, and inside Ok is the successfully generated value. The Err variant means the operation failed, and Err contains information about how or why the operation failed.

Values of the Result type, like values of any type, have methods defined on them. An instance of Result has an expect method that you can call. If this instance of Result is an Err value, expect will cause the program to crash and display the message that you passed as an argument to expect. If the read_line method returns an Err, it would likely be the result of an error coming from the underlying operating system. If this instance of Result is an Ok value, expect will take the return value that Ok is holding and return just that value to you so you can use it. In this case, that value is the number of bytes in the user’s input.

然而，一长行很难阅读，所以最好将其分开。 当您使用 .method_name() 语法调用方法时，引入换行符和其他空格来帮助分解长行通常是明智的做法。 现在我们来讨论一下这条线的作用。

如前所述，read_line 将用户输入的任何内容放入我们传递给它的字符串中，但它也返回一个 Result 值。 结果是一个枚举，通常称为枚举，它是一种可以处于多种可能状态之一的类型。 我们将每种可能的状态称为变体。

第 6 章将更详细地介绍枚举。 这些结果类型的目的是对错误处理信息进行编码。

结果的变体是 Ok 和 Err。 Ok变量表示操作成功，Ok里面是成功生成的值。 Err 变体表示操作失败，Err 包含有关操作如何或为何失败的信息。

与任何类型的值一样，Result 类型的值也定义了方法。 Result 的实例有一个可以调用的 Expect 方法。 如果 Result 的此实例是 Err 值，expect 将导致程序崩溃并显示您作为参数传递给expect 的消息。 如果 read_line 方法返回 Err，则可能是来自底层操作系统的错误的结果。 如果 Result 的这个实例是 Ok 值，expect 将获取 Ok 所保存的返回值并将该值返回给您，以便您可以使用它。 在本例中，该值是用户输入中的字节数。

If you don’t call expect, the program will compile, but you’ll get a warning:
``````
$ cargo build
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
warning: unused `Result` that must be used
  --> src/main.rs:10:5
   |
10 |     io::stdin().read_line(&mut guess);
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
   |
   = note: this `Result` may be an `Err` variant, which should be handled
   = note: `#[warn(unused_must_use)]` on by default

warning: `guessing_game` (bin "guessing_game") generated 1 warning
    Finished dev [unoptimized + debuginfo] target(s) in 0.59s
``````
Rust warns that you haven’t used the Result value returned from read_line, indicating that the program hasn’t handled a possible error.

The right way to suppress the warning is to actually write error-handling code, but in our case we just want to crash this program when a problem occurs, so we can use expect. You’ll learn about recovering from errors in Chapter 9.

Rust 警告您尚未使用 read_line 返回的 Result 值，这表明程序尚未处理可能的错误。

抑制警告的正确方法是实际编写错误处理代码，但在我们的例子中，我们只想在出现问题时使该程序崩溃，因此我们可以使用expect。 您将在第 9 章中了解如何从错误中恢复。

### Printing Values with println! Placeholders
Aside from the closing curly bracket, there’s only one more line to discuss in the code so far:
``````
    println!("You guessed: {guess}");
``````
This line prints the string that now contains the user’s input. The {} set of curly brackets is a placeholder: think of {} as little crab pincers that hold a value in place. When printing the value of a variable, the variable name can go inside the curly brackets. When printing the result of evaluating an expression, place empty curly brackets in the format string, then follow the format string with a comma-separated list of expressions to print in each empty curly bracket placeholder in the same order. Printing a variable and the result of an expression in one call to println! would look like this:

此行打印现在包含用户输入的字符串。 大括号 {} 组是一个占位符：将 {} 想象成固定值的小螃蟹钳子。 打印变量值时，变量名称可以放在大括号内。 打印表达式求值结果时，将空大括号放在格式字符串中，然后在格式字符串后跟上以逗号分隔的表达式列表，以相同的顺序在每个空大括号占位符中打印。 在一次调用 println! 中打印变量和表达式的结果！ 看起来像这样：

``````
let x = 5;
let y = 10;

println!("x = {x} and y + 2 = {}", y + 2);
``````
This code would print x = 5 and y + 2 = 12.

### Testing the First Part
Let’s test the first part of the guessing game. Run it using cargo run:
``````
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 6.44s
     Running `target/debug/guessing_game`
Guess the number!
Please input your guess.
6
You guessed: 6
``````
At this point, the first part of the game is done: we’re getting input from the keyboard and then printing it.

## Generating a Secret Number
Next, we need to generate a secret number that the user will try to guess. The secret number should be different every time so the game is fun to play more than once. We’ll use a random number between 1 and 100 so the game isn’t too difficult. Rust doesn’t yet include random number functionality in its standard library. However, the Rust team does provide a rand crate with said functionality.

接下来，我们需要生成一个用户将尝试猜测的秘密数字。 每次的秘密数字应该不同，这样玩多次游戏就会很有趣。 我们将使用 1 到 100 之间的随机数，这样游戏就不会太困难。 Rust 的标准库中尚未包含随机数功能。 然而，Rust 团队确实提供了具有上述功能的 rand 箱。

### Using a Crate to Get More Functionality
Remember that a crate is a collection of Rust source code files. The project we’ve been building is a binary crate, which is an executable. The rand crate is a library crate, which contains code that is intended to be used in other programs and can’t be executed on its own.

Cargo’s coordination of external crates is where Cargo really shines. Before we can write code that uses rand, we need to modify the Cargo.toml file to include the rand crate as a dependency. Open that file now and add the following line to the bottom, beneath the [dependencies] section header that Cargo created for you. Be sure to specify rand exactly as we have here, with this version number, or the code examples in this tutorial may not work:

请记住，板条箱是 Rust 源代码文件的集合。 我们一直在构建的项目是一个二进制包，它是一个可执行文件。 rand crate 是一个库 crate，其中包含旨在在其他程序中使用且不能单独执行的代码。

Cargo 与外部板条箱的协调是 Cargo 真正的亮点。 在编写使用 rand 的代码之前，我们需要修改 Cargo.toml 文件以包含 rand 箱作为依赖项。 现在打开该文件，并将以下行添加到底部，即 Cargo 为您创建的 [dependencies] 部分标题下方。 请务必使用此版本号完全按照我们此处的方式指定 rand，否则本教程中的代码示例可能无法正常工作：

Filename: Cargo.toml
``````
[dependencies]
rand = "0.8.5"
``````
In the Cargo.toml file, everything that follows a header is part of that section that continues until another section starts. In [dependencies] you tell Cargo which external crates your project depends on and which versions of those crates you require. In this case, we specify the rand crate with the semantic version specifier 0.8.5. Cargo understands Semantic Versioning (sometimes called SemVer), which is a standard for writing version numbers. The specifier 0.8.5 is actually shorthand for ^0.8.5, which means any version that is at least 0.8.5 but below 0.9.0.

Cargo considers these versions to have public APIs compatible with version 0.8.5, and this specification ensures you’ll get the latest patch release that will still compile with the code in this chapter. Any version 0.9.0 or greater is not guaranteed to have the same API as what the following examples use.

在 Cargo.toml 文件中，标头后面的所有内容都是该部分的一部分，一直持续到另一个部分开始。 在 [依赖项] 中，您告诉 Cargo 您的项目依赖哪些外部 crate，以及您需要这些 crate 的哪些版本。 在本例中，我们使用语义版本说明符 0.8.5 指定 rand 箱。 Cargo 了解语义版本控制（有时称为 SemVer），这是编写版本号的标准。 说明符 0.8.5 实际上是 ^0.8.5 的简写，这意味着任何至少 0.8.5 但低于 0.9.0 的版本。

Cargo 认为这些版本具有与 0.8.5 版本兼容的公共 API，并且此规范确保您将获得仍将使用本章中的代码进行编译的最新补丁版本。 不保证任何 0.9.0 或更高版本具有与以下示例使用的 API 相同的 API。

Now, without changing any of the code, let’s build the project, as shown in Listing 2-2.
``````
$ cargo build
    Updating crates.io index
  Downloaded rand v0.8.5
  Downloaded libc v0.2.127
  Downloaded getrandom v0.2.7
  Downloaded cfg-if v1.0.0
  Downloaded ppv-lite86 v0.2.16
  Downloaded rand_chacha v0.3.1
  Downloaded rand_core v0.6.3
   Compiling libc v0.2.127
   Compiling getrandom v0.2.7
   Compiling cfg-if v1.0.0
   Compiling ppv-lite86 v0.2.16
   Compiling rand_core v0.6.3
   Compiling rand_chacha v0.3.1
   Compiling rand v0.8.5
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 2.53s
``````
Listing 2-2: The output from running cargo build after adding the rand crate as a dependency

You may see different version numbers (but they will all be compatible with the code, thanks to SemVer!) and different lines (depending on the operating system), and the lines may be in a different order.

When we include an external dependency, Cargo fetches the latest versions of everything that dependency needs from the registry, which is a copy of data from Crates.io. Crates.io is where people in the Rust ecosystem post their open source Rust projects for others to use.

After updating the registry, Cargo checks the [dependencies] section and downloads any crates listed that aren’t already downloaded. In this case, although we only listed rand as a dependency, Cargo also grabbed other crates that rand depends on to work. After downloading the crates, Rust compiles them and then compiles the project with the dependencies available.

If you immediately run cargo build again without making any changes, you won’t get any output aside from the Finished line. Cargo knows it has already downloaded and compiled the dependencies, and you haven’t changed anything about them in your Cargo.toml file. Cargo also knows that you haven’t changed anything about your code, so it doesn’t recompile that either. With nothing to do, it simply exits.

If you open the src/main.rs file, make a trivial change, and then save it and build again, you’ll only see two lines of output:

您可能会看到不同的版本号（但由于 SemVer，它们都与代码兼容！）和不同的行（取决于操作系统），并且这些行的顺序可能不同。

当我们包含外部依赖项时，Cargo 会从注册表中获取该依赖项所需的所有内容的最新版本，该注册表是来自 Crates.io 的数据副本。 Crates.io 是 Rust 生态系统中的人们发布开源 Rust 项目供其他人使用的地方。

更新注册表后，Cargo 检查 [依赖项] 部分并下载列出的所有尚未下载的包。 在本例中，虽然我们只将 rand 列为依赖项，但 Cargo 还抓取了 rand 工作所依赖的其他 crate。 下载包后，Rust 会编译它们，然后使用可用的依赖项编译项目。

如果您立即再次运行 Cargo build 而不进行任何更改，则除了 Finished 行之外，您将不会获得任何输出。 Cargo 知道它已经下载并编译了依赖项，并且您没有在 Cargo.toml 文件中更改它们的任何内容。 Cargo 还知道您没有对代码进行任何更改，因此它也不会重新编译。 无事可做，它就简单地退出了。

如果您打开 src/main.rs 文件，进行一些简单的更改，然后保存并再次构建，您将只会看到两行输出：

``````
$ cargo build
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 2.53 secs
``````
These lines show that Cargo only updates the build with your tiny change to the src/main.rs file. Your dependencies haven’t changed, so Cargo knows it can reuse what it has already downloaded and compiled for those.

这些行表明 Cargo 仅通过对 src/main.rs 文件进行微小更改来更新构建。 您的依赖项没有改变，因此 Cargo 知道它可以重用已经下载并编译的内容。

### Ensuring Reproducible Builds with the Cargo.lock File
Cargo has a mechanism that ensures you can rebuild the same artifact every time you or anyone else builds your code: Cargo will use only the versions of the dependencies you specified until you indicate otherwise. For example, say that next week version 0.8.6 of the rand crate comes out, and that version contains an important bug fix, but it also contains a regression that will break your code. To handle this, Rust creates the Cargo.lock file the first time you run cargo build, so we now have this in the guessing_game directory.

When you build a project for the first time, Cargo figures out all the versions of the dependencies that fit the criteria and then writes them to the Cargo.lock file. When you build your project in the future, Cargo will see that the Cargo.lock file exists and will use the versions specified there rather than doing all the work of figuring out versions again. This lets you have a reproducible build automatically. In other words, your project will remain at 0.8.5 until you explicitly upgrade, thanks to the Cargo.lock file. Because the Cargo.lock file is important for reproducible builds, it’s often checked into source control with the rest of the code in your project.

Cargo 具有一种机制，可确保您或其他任何人每次构建代码时都可以重建相同的工件：Cargo 将仅使用您指定的依赖项版本，直到您另有指示。 例如，假设下周 rand crate 的 0.8.6 版本将发布，该版本包含一个重要的错误修复，但它也包含一个会破坏您的代码的回归。 为了处理这个问题，Rust 在您第一次运行 Cargo build 时创建了 Cargo.lock 文件，因此我们现在将其放置在guessing_game 目录中。

当您第一次构建项目时，Cargo 会找出符合条件的所有依赖项版本，然后将它们写入 Cargo.lock 文件。 当您将来构建项目时，Cargo 将看到 Cargo.lock 文件存在，并将使用其中指定的版本，而不是再次执行确定版本的所有工作。 这可以让您自动获得可重复的构建。 换句话说，在您明确升级之前，您的项目将保持在 0.8.5，这要归功于 Cargo.lock 文件。 由于 Cargo.lock 文件对于可重现的构建非常重要，因此它通常会与项目中的其余代码一起检入源代码管理。

### Updating a Crate to Get a New Version
When you do want to update a crate, Cargo provides the command update, which will ignore the Cargo.lock file and figure out all the latest versions that fit your specifications in Cargo.toml. Cargo will then write those versions to the Cargo.lock file. Otherwise, by default, Cargo will only look for versions greater than 0.8.5 and less than 0.9.0. If the rand crate has released the two new versions 0.8.6 and 0.9.0, you would see the following if you ran cargo update:

当您确实想要更新 crate 时，Cargo 会提供命令 update，该命令将忽略 Cargo.lock 文件并找出符合您在 Cargo.toml 中的规范的所有最新版本。 然后 Cargo 会将这些版本写入 Cargo.lock 文件。 否则，默认情况下，Cargo 将仅查找大于 0.8.5 且小于 0.9.0 的版本。 如果 rand crate 已经发布了两个新版本 0.8.6 和 0.9.0，则运行 Cargo update 时您将看到以下内容：
``````
$ cargo update
    Updating crates.io index
    Updating rand v0.8.5 -> v0.8.6
``````
Cargo ignores the 0.9.0 release. At this point, you would also notice a change in your Cargo.lock file noting that the version of the rand crate you are now using is 0.8.6. To use rand version 0.9.0 or any version in the 0.9.x series, you’d have to update the Cargo.toml file to look like this instead:

Cargo 忽略 0.9.0 版本。 此时，您还会注意到 Cargo.lock 文件中的更改，指出您现在使用的 rand 箱的版本是 0.8.6。 要使用 rand 版本 0.9.0 或 0.9.x 系列中的任何版本，您必须将 Cargo.toml 文件更新为如下所示：

``````
[dependencies]
rand = "0.9.0"
``````
The next time you run cargo build, Cargo will update the registry of crates available and reevaluate your rand requirements according to the new version you have specified.

There’s a lot more to say about Cargo and its ecosystem, which we’ll discuss in Chapter 14, but for now, that’s all you need to know. Cargo makes it very easy to reuse libraries, so Rustaceans are able to write smaller projects that are assembled from a number of packages.

下次运行 Cargo build 时，Cargo 将更新可用 crate 的注册表，并根据您指定的新版本重新评估您的 rand 要求。

关于 Cargo 及其生态系统还有很多内容要说，我们将在第 14 章中讨论，但现在，这就是您需要了解的全部内容。 Cargo 使得重用库变得非常容易，因此 Rustaceans 能够编写由多个包组装而成的较小项目。

### Generating a Random Number
Let’s start using rand to generate a number to guess. The next step is to update src/main.rs, as shown in Listing 2-3.

Filename: src/main.rs
``````
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
``````
Listing 2-3: Adding code to generate a random number

First we add the line use rand::Rng;. The Rng trait defines methods that random number generators implement, and this trait must be in scope for us to use those methods. Chapter 10 will cover traits in detail.

Next, we’re adding two lines in the middle. In the first line, we call the rand::thread_rng function that gives us the particular random number generator we’re going to use: one that is local to the current thread of execution and is seeded by the operating system. Then we call the gen_range method on the random number generator. This method is defined by the Rng trait that we brought into scope with the use rand::Rng; statement. The gen_range method takes a range expression as an argument and generates a random number in the range. The kind of range expression we’re using here takes the form start..=end and is inclusive on the lower and upper bounds, so we need to specify 1..=100 to request a number between 1 and 100.

Note: You won’t just know which traits to use and which methods and functions to call from a crate, so each crate has documentation with instructions for using it. Another neat feature of Cargo is that running the cargo doc --open command will build documentation provided by all your dependencies locally and open it in your browser. If you’re interested in other functionality in the rand crate, for example, run cargo doc --open and click rand in the sidebar on the left.

The second new line prints the secret number. This is useful while we’re developing the program to be able to test it, but we’ll delete it from the final version. It’s not much of a game if the program prints the answer as soon as it starts!

首先我们添加行 use rand::Rng;。 Rng 特征定义了随机数生成器实现的方法，并且该特征必须在我们使用这些方法的范围内。 第 10 章将详细介绍特征。

接下来，我们在中间添加两行。 在第一行中，我们调用 rand::thread_rng 函数，它为我们提供了我们将要使用的特定随机数生成器：一个位于当前执行线程本地并由操作系统播种的随机数生成器。 然后我们调用随机数生成器上的 gen_range 方法。 该方法由我们通过 use rand::Rng; 引入范围的 Rng 特征定义。 陈述。 gen_range 方法采用范围表达式作为参数，并在该范围内生成一个随机数。 我们在这里使用的范围表达式采用 start..=end 的形式，并且包含下限和上限，因此我们需要指定 1..=100 来请求 1 到 100 之间的数字。

注意：您不仅知道要使用哪些特征以及要从包中调用哪些方法和函数，因此每个包都有包含使用说明的文档。 Cargo 的另一个巧妙功能是，运行 Cargo doc --open 命令将在本地构建所有依赖项提供的文档，并在浏览器中打开它。 例如，如果您对 rand 箱中的其他功能感兴趣，请运行 Cargo doc --open 并单击左侧边栏中的 rand。

第二个新行打印秘密号码。 这在我们开发程序以对其进行测试时很有用，但我们将从最终版本中删除它。 如果程序一开始就打印出答案，那就不算什么游戏了！

Try running the program a few times:
``````
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 2.53s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 7
Please input your guess.
4
You guessed: 4

$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.02s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 83
Please input your guess.
5
You guessed: 5
``````
You should get different random numbers, and they should all be numbers between 1 and 100. Great job!

## Comparing the Guess to the Secret Number
Now that we have user input and a random number, we can compare them. That step is shown in Listing 2-4. Note that this code won’t compile just yet, as we will explain.

Filename: src/main.rs
``````
This code does not compile!
use rand::Rng;
use std::cmp::Ordering;
use std::io;

fn main() {
    // --snip--

    println!("You guessed: {guess}");

    match guess.cmp(&secret_number) {
        Ordering::Less => println!("Too small!"),
        Ordering::Greater => println!("Too big!"),
        Ordering::Equal => println!("You win!"),
    }
}
``````
Listing 2-4: Handling the possible return values of comparing two numbers

First we add another use statement, bringing a type called std::cmp::Ordering into scope from the standard library. The Ordering type is another enum and has the variants Less, Greater, and Equal. These are the three outcomes that are possible when you compare two values.

Then we add five new lines at the bottom that use the Ordering type. The cmp method compares two values and can be called on anything that can be compared. It takes a reference to whatever you want to compare with: here it’s comparing guess to secret_number. Then it returns a variant of the Ordering enum we brought into scope with the use statement. We use a match expression to decide what to do next based on which variant of Ordering was returned from the call to cmp with the values in guess and secret_number.

A match expression is made up of arms. An arm consists of a pattern to match against, and the code that should be run if the value given to match fits that arm’s pattern. Rust takes the value given to match and looks through each arm’s pattern in turn. Patterns and the match construct are powerful Rust features: they let you express a variety of situations your code might encounter and they make sure you handle them all. These features will be covered in detail in Chapter 6 and Chapter 18, respectively.

Let’s walk through an example with the match expression we use here. Say that the user has guessed 50 and the randomly generated secret number this time is 38.

When the code compares 50 to 38, the cmp method will return Ordering::Greater because 50 is greater than 38. The match expression gets the Ordering::Greater value and starts checking each arm’s pattern. It looks at the first arm’s pattern, Ordering::Less, and sees that the value Ordering::Greater does not match Ordering::Less, so it ignores the code in that arm and moves to the next arm. The next arm’s pattern is Ordering::Greater, which does match Ordering::Greater! The associated code in that arm will execute and print Too big! to the screen. The match expression ends after the first successful match, so it won’t look at the last arm in this scenario.

首先，我们添加另一个 use 语句，将一个名为 std::cmp::Ordering 的类型从标准库引入范围。 Ordering 类型是另一个枚举，具有 Less、Greater 和 Equal 变体。 这是比较两个值时可能出现的三种结果。

然后我们在底部添加五个使用排序类型的新行。 cmp 方法比较两个值，并且可以在任何可以比较的对象上调用。 它需要引用您想要比较的任何内容：这里是将猜测值与秘密数字进行比较。 然后它返回我们通过 use 语句引入范围的 Ordering 枚举的变体。 我们使用匹配表达式来根据从调用 cmp 返回的 Ordering 的变体以及guess 和secret_number 中的值来决定下一步要做什么。

匹配表达式由手臂组成。 手臂由要匹配的模式组成，以及如果给定的匹配值适合该手臂的模式则应运行的代码。 Rust 获取给定的值进行匹配，并依次查看每个手臂的模式。 模式和匹配构造是强大的 Rust 功能：它们让您表达代码可能遇到的各种情况，并确保您能够处理所有情况。 这些功能将分别在第 6 章和第 18 章中详细介绍。

让我们看一下这里使用的匹配表达式的示例。 假设用户猜中了 50，这次随机生成的秘密数字是 38。

当代码比较 50 和 38 时，cmp 方法将返回 Ordering::Greater，因为 50 大于 38。匹配表达式获取 Ordering::Greater 值并开始检查每个臂的模式。 它查看第一个臂的模式 Ordering::Less，并发现值 Ordering::Greater 与 Ordering::Less 不匹配，因此它忽略该臂中的代码并移至下一个臂。 下一个手臂的模式是 Ordering::Greater，它与 Ordering::Greater 匹配！ 该手臂中的相关代码将执行并打印 Too big! 到屏幕上。 匹配表达式在第一次成功匹配后结束，因此在这种情况下它不会查看最后一个分支。

However, the code in Listing 2-4 won’t compile yet. Let’s try it:
``````
$ cargo build
   Compiling libc v0.2.86
   Compiling getrandom v0.2.2
   Compiling cfg-if v1.0.0
   Compiling ppv-lite86 v0.2.10
   Compiling rand_core v0.6.2
   Compiling rand_chacha v0.3.0
   Compiling rand v0.8.5
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
error[E0308]: mismatched types
  --> src/main.rs:22:21
   |
22 |     match guess.cmp(&secret_number) {
   |                 --- ^^^^^^^^^^^^^^ expected struct `String`, found integer
   |                 |
   |                 arguments to this function are incorrect
   |
   = note: expected reference `&String`
              found reference `&{integer}`
note: associated function defined here
  --> /rustc/d5a82bbd26e1ad8b7401f6a718a9c57c96905483/library/core/src/cmp.rs:783:8

For more information about this error, try `rustc --explain E0308`.
error: could not compile `guessing_game` due to previous error
``````
The core of the error states that there are mismatched types. Rust has a strong, static type system. However, it also has type inference. When we wrote let mut guess = String::new(), Rust was able to infer that guess should be a String and didn’t make us write the type. The secret_number, on the other hand, is a number type. A few of Rust’s number types can have a value between 1 and 100: i32, a 32-bit number; u32, an unsigned 32-bit number; i64, a 64-bit number; as well as others. Unless otherwise specified, Rust defaults to an i32, which is the type of secret_number unless you add type information elsewhere that would cause Rust to infer a different numerical type. The reason for the error is that Rust cannot compare a string and a number type.

Ultimately, we want to convert the String the program reads as input into a real number type so we can compare it numerically to the secret number. We do so by adding this line to the main function body:

错误的核心是类型不匹配。 Rust 拥有强大的静态类型系统。 然而，它也有类型推断。 当我们编写 let mut Guess = String::new() 时，Rust 能够推断 Guess 应该是一个 String 并且不会让我们编写类型。 另一方面，secret_number 是数字类型。 Rust 的一些数字类型的值可以在 1 到 100 之间： i32，一个 32 位数字； u32，无符号32位数字； i64，64位数字； 以及其他人。 除非另有说明，Rust 默认为 i32，这是 Secret_number 的类型，除非您在其他地方添加类型信息，导致 Rust 推断出不同的数字类型。 错误的原因是 Rust 无法比较字符串和数字类型。

最终，我们希望将程序读取作为输入的字符串转换为实数类型，以便我们可以将其与秘密数字进行数字比较。 我们通过将此行添加到主函数体来实现：

Filename: src/main.rs
``````
    // --snip--

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
``````
The line is:
``````
let guess: u32 = guess.trim().parse().expect("Please type a number!");
``````
We create a variable named guess. But wait, doesn’t the program already have a variable named guess? It does, but helpfully Rust allows us to shadow the previous value of guess with a new one. Shadowing lets us reuse the guess variable name rather than forcing us to create two unique variables, such as guess_str and guess, for example. We’ll cover this in more detail in Chapter 3, but for now, know that this feature is often used when you want to convert a value from one type to another type.

We bind this new variable to the expression guess.trim().parse(). The guess in the expression refers to the original guess variable that contained the input as a string. The trim method on a String instance will eliminate any whitespace at the beginning and end, which we must do to be able to compare the string to the u32, which can only contain numerical data. The user must press enter to satisfy read_line and input their guess, which adds a newline character to the string. For example, if the user types 5 and presses enter, guess looks like this: 5\n. The \n represents “newline.” (On Windows, pressing enter results in a carriage return and a newline, \r\n.) The trim method eliminates \n or \r\n, resulting in just 5.

The parse method on strings converts a string to another type. Here, we use it to convert from a string to a number. We need to tell Rust the exact number type we want by using let guess: u32. The colon (:) after guess tells Rust we’ll annotate the variable’s type. Rust has a few built-in number types; the u32 seen here is an unsigned, 32-bit integer. It’s a good default choice for a small positive number. You’ll learn about other number types in Chapter 3.

Additionally, the u32 annotation in this example program and the comparison with secret_number means Rust will infer that secret_number should be a u32 as well. So now the comparison will be between two values of the same type!

The parse method will only work on characters that can logically be converted into numbers and so can easily cause errors. If, for example, the string contained A👍%, there would be no way to convert that to a number. Because it might fail, the parse method returns a Result type, much as the read_line method does (discussed earlier in “Handling Potential Failure with Result”). We’ll treat this Result the same way by using the expect method again. If parse returns an Err Result variant because it couldn’t create a number from the string, the expect call will crash the game and print the message we give it. If parse can successfully convert the string to a number, it will return the Ok variant of Result, and expect will return the number that we want from the Ok value.

我们创建一个名为guess 的变量。 但是等等，程序不是已经有一个名为guess 的变量了吗？ 确实如此，但 Rust 允许我们用新的猜测值来掩盖之前的猜测值，这很有帮助。 遮蔽让我们可以重用猜测变量名称，而不是强迫我们创建两个唯一的变量，例如guess_str和guess。 我们将在第 3 章中更详细地介绍这一点，但现在，当您想要将值从一种类型转换为另一种类型时，通常会使用此功能。

我们将这个新变量绑定到表达式guess.trim().parse()。 表达式中的猜测是指包含字符串输入的原始猜测变量。 String 实例上的 trim 方法将消除开头和结尾的任何空格，我们必须这样做才能将字符串与只能包含数字数据的 u32 进行比较。 用户必须按 Enter 以满足 read_line 并输入他们的猜测，这会向字符串添加换行符。 例如，如果用户键入 5 并按 Enter 键，则猜测如下：5\n。 \n 代表“换行符”。 （在 Windows 上，按 Enter 键会产生回车符和换行符 \r\n。）trim 方法消除 \n 或 \r\n，结果仅为 5。

字符串的 parse 方法将字符串转换为另一种类型。 在这里，我们使用它从字符串转换为数字。 我们需要通过使用 let Guess: u32 告诉 Rust 我们想要的确切数字类型。 猜测后的冒号 (:) 告诉 Rust 我们将注释变量的类型。 Rust 有一些内置的数字类型； 这里看到的 u32 是一个无符号的 32 位整数。 对于较小的正数来说，这是一个很好的默认选择。 您将在第 3 章中了解其他数字类型。

此外，此示例程序中的 u32 注释以及与 Secret_number 的比较意味着 Rust 将推断 Secret_number 也应该是 u32。 所以现在比较将在相同类型的两个值之间进行！

parse 方法仅适用于逻辑上可以转换为数字的字符，因此很容易导致错误。 例如，如果字符串包含 A👍%，则无法将其转换为数字。 因为它可能会失败，所以 parse 方法返回 Result 类型，就像 read_line 方法所做的那样（前面在“使用结果处理潜在失败”中讨论过）。 我们将再次使用expect方法以同样的方式处理这个结果。 如果 parse 因为无法从字符串创建数字而返回 Err Result 变体，则 Expect 调用将使游戏崩溃并打印我们给它的消息。 如果parse可以成功地将字符串转换为数字，它将返回Result的Ok变体，而expect将从Ok值中返回我们想要的数字。

Let’s run the program now:
``````
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 0.43s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 58
Please input your guess.
  76
You guessed: 76
Too big!
``````
Nice! Even though spaces were added before the guess, the program still figured out that the user guessed 76. Run the program a few times to verify the different behavior with different kinds of input: guess the number correctly, guess a number that is too high, and guess a number that is too low.

We have most of the game working now, but the user can make only one guess. Let’s change that by adding a loop!

好的！ 即使在猜测之前添加了空格，程序仍然发现用户猜测了 76。运行程序几次以验证不同类型输入的不同行为：正确猜测数字，猜测数字太大， 并猜测一个太低的数字。

现在游戏的大部分功能都可以运行，但用户只能做出一种猜测。 让我们通过添加一个循环来改变它！

## Allowing Multiple Guesses with Looping
The loop keyword creates an infinite loop. We’ll add a loop to give users more chances at guessing the number:

Filename: src/main.rs
``````
    // --snip--

    println!("The secret number is: {secret_number}");

    loop {
        println!("Please input your guess.");

        // --snip--

        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal => println!("You win!"),
        }
    }
}
``````
As you can see, we’ve moved everything from the guess input prompt onward into a loop. Be sure to indent the lines inside the loop another four spaces each and run the program again. The program will now ask for another guess forever, which actually introduces a new problem. It doesn’t seem like the user can quit!

The user could always interrupt the program by using the keyboard shortcut ctrl-c. But there’s another way to escape this insatiable monster, as mentioned in the parse discussion in “Comparing the Guess to the Secret Number”: if the user enters a non-number answer, the program will crash. We can take advantage of that to allow the user to quit, as shown here:

正如您所看到的，我们已将猜测输入提示中的所有内容移至循环中。 确保将循环内的行每行缩进四个空格，然后再次运行程序。 程序现在将永远要求另一个猜测，这实际上引入了一个新问题。 用户似乎无法退出！

用户始终可以使用键盘快捷键 ctrl-c 来中断程序。 但还有另一种方法可以逃避这个贪得无厌的怪物，正如“比较猜测与秘密数字”中的解析讨论中提到的：如果用户输入非数字答案，程序就会崩溃。 我们可以利用这一点来允许用户退出，如下所示：

``````
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 1.50s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 59
Please input your guess.
45
You guessed: 45
Too small!
Please input your guess.
60
You guessed: 60
Too big!
Please input your guess.
59
You guessed: 59
You win!
Please input your guess.
quit
thread 'main' panicked at 'Please type a number!: ParseIntError { kind: InvalidDigit }', src/main.rs:28:47
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
``````
Typing quit will quit the game, but as you’ll notice, so will entering any other non-number input. This is suboptimal, to say the least; we want the game to also stop when the correct number is guessed.

输入 quit 将退出游戏，但您会注意到，输入任何其他非数字输入也会退出游戏。 至少可以说，这是次优的； 我们希望当猜到正确的数字时游戏也停止。

## Quitting After a Correct Guess
Let’s program the game to quit when the user wins by adding a break statement:

Filename: src/main.rs
``````
        // --snip--

        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal => {
                println!("You win!");
                break;
            }
        }
    }
}
``````
Adding the break line after You win! makes the program exit the loop when the user guesses the secret number correctly. Exiting the loop also means exiting the program, because the loop is the last part of main.

Handling Invalid Input
To further refine the game’s behavior, rather than crashing the program when the user inputs a non-number, let’s make the game ignore a non-number so the user can continue guessing. We can do that by altering the line where guess is converted from a String to a u32, as shown in Listing 2-5.

获胜后添加断线！ 当用户正确猜出秘密数字时，使程序退出循环。 退出循环也意味着退出程序，因为循环是main的最后一部分。

处理无效输入
为了进一步完善游戏的行为，让游戏忽略非数字以便用户可以继续猜测，而不是在用户输入非数字时使程序崩溃。 我们可以通过更改将猜测从字符串转换为 u32 的行来实现这一点，如清单 2-5 所示。

Filename: src/main.rs
``````
        // --snip--

        io::stdin()
            .read_line(&mut guess)
            .expect("Failed to read line");

        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        println!("You guessed: {guess}");

        // --snip--
``````
Listing 2-5: Ignoring a non-number guess and asking for another guess instead of crashing the program

We switch from an expect call to a match expression to move from crashing on an error to handling the error. Remember that parse returns a Result type and Result is an enum that has the variants Ok and Err. We’re using a match expression here, as we did with the Ordering result of the cmp method.

If parse is able to successfully turn the string into a number, it will return an Ok value that contains the resultant number. That Ok value will match the first arm’s pattern, and the match expression will just return the num value that parse produced and put inside the Ok value. That number will end up right where we want it in the new guess variable we’re creating.

If parse is not able to turn the string into a number, it will return an Err value that contains more information about the error. The Err value does not match the Ok(num) pattern in the first match arm, but it does match the Err(_) pattern in the second arm. The underscore, _, is a catchall value; in this example, we’re saying we want to match all Err values, no matter what information they have inside them. So the program will execute the second arm’s code, continue, which tells the program to go to the next iteration of the loop and ask for another guess. So, effectively, the program ignores all errors that parse might encounter!

我们从期望调用切换到匹配表达式，以从因错误而崩溃转变为处理错误。 请记住，parse 返回 Result 类型，而 Result 是一个具有 Ok 和 Err 变体的枚举。 我们在这里使用匹配表达式，就像我们对 cmp 方法的排序结果所做的那样。

如果 parse 能够成功将字符串转换为数字，它将返回包含结果数字的 Ok 值。 该 Ok 值将与第一个臂的模式匹配，并且匹配表达式将仅返回解析生成的 num 值并将其放入 Ok 值中。 该数字最终将位于我们正在创建的新猜测变量中所需的位置。

如果 parse 无法将字符串转换为数字，它将返回一个 Err 值，其中包含有关错误的更多信息。 Err 值与第一个匹配臂中的 Ok(num) 模式不匹配，但与第二个匹配臂中的 Err(_) 模式匹配。 下划线 _ 是一个包罗万象的值； 在这个例子中，我们说我们想要匹配所有 Err 值，无论它们里面有什么信息。 因此，程序将执行第二个分支的代码 continue，它告诉程序进入循环的下一次迭代并要求进行另一个猜测。 因此，实际上，程序会忽略解析可能遇到的所有错误！

Now everything in the program should work as expected. Let’s try it:
``````
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished dev [unoptimized + debuginfo] target(s) in 4.45s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 61
Please input your guess.
10
You guessed: 10
Too small!
Please input your guess.
99
You guessed: 99
Too big!
Please input your guess.
foo
Please input your guess.
61
You guessed: 61
You win!
``````
Awesome! With one tiny final tweak, we will finish the guessing game. Recall that the program is still printing the secret number. That worked well for testing, but it ruins the game. Let’s delete the println! that outputs the secret number. Listing 2-6 shows the final code.

惊人的！ 通过最后的一点小小的调整，我们将完成这个猜谜游戏。 回想一下，程序仍在打印秘密数字。 这对于测试来说效果很好，但它却破坏了游戏。 让我们删除 println 吧！ 输出秘密数字。 清单 2-6 显示了最终的代码。

Filename: src/main.rs
``````
use rand::Rng;
use std::cmp::Ordering;
use std::io;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1..=100);

    loop {
        println!("Please input your guess.");

        let mut guess = String::new();

        io::stdin()
            .read_line(&mut guess)
            .expect("Failed to read line");

        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        println!("You guessed: {guess}");

        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal => {
                println!("You win!");
                break;
            }
        }
    }
}
``````
Listing 2-6: Complete guessing game code

At this point, you’ve successfully built the guessing game. Congratulations!

## 2.2 Summary
This project was a hands-on way to introduce you to many new Rust concepts: let, match, functions, the use of external crates, and more. In the next few chapters, you’ll learn about these concepts in more detail. Chapter 3 covers concepts that most programming languages have, such as variables, data types, and functions, and shows how to use them in Rust. Chapter 4 explores ownership, a feature that makes Rust different from other languages. Chapter 5 discusses structs and method syntax, and Chapter 6 explains how enums work.

这个项目是向您介绍许多新的 Rust 概念的实践方式：let、match、函数、外部板条箱的使用等等。 在接下来的几章中，您将更详细地了解这些概念。 第 3 章涵盖了大多数编程语言所具有的概念，例如变量、数据类型和函数，并展示了如何在 Rust 中使用它们。 第 4 章探讨了所有权，这是 Rust 与其他语言不同的一个特性。 第 5 章讨论结构体和方法语法，第 6 章解释枚举的工作原理。