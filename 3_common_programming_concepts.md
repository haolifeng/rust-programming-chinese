# 3. 通用编程概念
This chapter covers concepts that appear in almost every programming language and how they work in Rust. Many programming languages have much in common at their core. None of the concepts presented in this chapter are unique to Rust, but we’ll discuss them in the context of Rust and explain the conventions around using these concepts.

Specifically, you’ll learn about variables, basic types, functions, comments, and control flow. These foundations will be in every Rust program, and learning them early will give you a strong core to start from.

本章涵盖了几乎所有编程语言中出现的概念以及它们在 Rust 中的工作原理。 许多编程语言的核心都有很多共同点。 本章中提出的概念都不是 Rust 所独有的，但我们将在 Rust 的背景下讨论它们，并解释使用这些概念的约定。

具体来说，您将了解变量、基本类型、函数、注释和控制流。 这些基础知识将存在于每个 Rust 程序中，尽早学习它们将为您提供强大的核心。

**关键字**  
The Rust language has a set of keywords that are reserved for use by the language only, much as in other languages. Keep in mind that you cannot use these words as names of variables or functions. Most of the keywords have special meanings, and you’ll be using them to do various tasks in your Rust programs; a few have no current functionality associated with them but have been reserved for functionality that might be added to Rust in the future. You can find a list of the keywords in Appendix A.

Rust 语言有一组保留仅供该语言使用的关键字，就像其他语言一样。 请记住，您不能使用这些单词作为变量或函数的名称。 大多数关键字都有特殊含义，您将使用它们在 Rust 程序中执行各种任务； 一些当前没有与之相关的功能，但已为将来可能添加到 Rust 的功能保留。 您可以在附录 A 中找到关键字列表。

## 3.1 变量和可变性

As mentioned in the “Storing Values with Variables” section, by default, variables are immutable. This is one of many nudges Rust gives you to write your code in a way that takes advantage of the safety and easy concurrency that Rust offers. However, you still have the option to make your variables mutable. Let’s explore how and why Rust encourages you to favor immutability and why sometimes you might want to opt out.

When a variable is immutable, once a value is bound to a name, you can’t change that value. To illustrate this, generate a new project called variables in your projects directory by using cargo new variables.

Then, in your new variables directory, open src/main.rs and replace its code with the following code, which won’t compile just yet:

Filename: src/main.rs

正如“用变量存储值”部分中提到的，默认情况下，变量是不可变的。 这是 Rust 为您提供的众多推动力之一，让您能够利用 Rust 提供的安全性和简单的并发性来编写代码。 但是，您仍然可以选择使变量可变。 让我们探讨 Rust 如何以及为什么鼓励您支持不变性，以及为什么有时您可能想要选择退出。

当变量是不可变的时，一旦将值绑定到名称，就无法更改该值。 为了说明这一点，请使用 Cargo new Variables 在项目目录中生成一个名为 Variables 的新项目。

然后，在新的变量目录中，打开 src/main.rs 并将其代码替换为以下代码，该代码目前还无法编译：

文件名：src/main.rs  
**不可编译**
```
fn main() {
    let x = 5;
    println!("The value of x is: {x}");
    x = 6;
    println!("The value of x is: {x}");
}
```
Save and run the program using cargo run. You should receive an error message regarding an immutability error, as shown in this output:  
使用cargo run 保存并运行程序。 您应该收到一条有关不变性错误的错误消息，如以下输出所示：
```
$ cargo run
   Compiling variables v0.1.0 (file:///projects/variables)
error[E0384]: cannot assign twice to immutable variable `x`
 --> src/main.rs:4:5
  |
2 |     let x = 5;
  |         -
  |         |
  |         first assignment to `x`
  |         help: consider making this binding mutable: `mut x`
3 |     println!("The value of x is: {x}");
4 |     x = 6;
  |     ^^^^^ cannot assign twice to immutable variable

For more information about this error, try `rustc --explain E0384`.
error: could not compile `variables` due to previous error

```
This example shows how the compiler helps you find errors in your programs. Compiler errors can be frustrating, but really they only mean your program isn’t safely doing what you want it to do yet; they do not mean that you’re not a good programmer! Experienced Rustaceans still get compiler errors.

You received the error message cannot assign twice to immutable variable `x` because you tried to assign a second value to the immutable x variable.

It’s important that we get compile-time errors when we attempt to change a value that’s designated as immutable because this very situation can lead to bugs. If one part of our code operates on the assumption that a value will never change and another part of our code changes that value, it’s possible that the first part of the code won’t do what it was designed to do. The cause of this kind of bug can be difficult to track down after the fact, especially when the second piece of code changes the value only sometimes. The Rust compiler guarantees that when you state that a value won’t change, it really won’t change, so you don’t have to keep track of it yourself. Your code is thus easier to reason through.

But mutability can be very useful, and can make code more convenient to write. Although variables are immutable by default, you can make them mutable by adding mut in front of the variable name as you did in Chapter 2. Adding mut also conveys intent to future readers of the code by indicating that other parts of the code will be changing this variable’s value.

For example, let’s change src/main.rs to the following:
Filename: src/main.rs

此示例显示编译器如何帮助您查找程序中的错误。 编译器错误可能会令人沮丧，但实际上它们仅意味着您的程序尚未安全地执行您希望它执行的操作； 它们并不意味着您不是一个好的程序员！ 经验丰富的 Rustaceans 仍然会遇到编译器错误。

您收到错误消息无法为不可变变量“x”分配两次，因为您尝试为不可变 x 变量分配第二个值。

当我们尝试更改指定为不可变的值时，我们会遇到编译时错误，这一点很重要，因为这种情况可能会导致错误。 如果我们的代码的一部分在假设值永远不会改变的情况下运行，而代码的另一部分更改了该值，则代码的第一部分可能不会执行其设计目的。 事后很难追查此类错误的原因，尤其是当第二段代码有时只更改值时。 Rust 编译器保证，当你声明一个值不会改变时，它确实不会改变，所以你不必自己跟踪它。 因此，您的代码更容易推理。

但可变性可能非常有用，并且可以使代码更方便编写。 尽管默认情况下变量是不可变的，但您可以通过在变量名称前面添加 mut 来使它们可变，就像在第 2 章中所做的那样。添加 mut 还可以通过指示代码的其他部分将发生更改来向未来的代码读者传达意图 该变量的值。

例如，我们将 src/main.rs 更改为以下内容：

Filename: src/main.rs
```
fn main() {
    let mut x = 5;
    println!("The value of x is: {x}");
    x = 6;
    println!("The value of x is: {x}");
}
```

When we run the program now, we get this:

```
$ cargo run
   Compiling variables v0.1.0 (file:///projects/variables)
    Finished dev [unoptimized + debuginfo] target(s) in 0.30s
     Running `target/debug/variables`
The value of x is: 5
The value of x is: 6

```
We’re allowed to change the value bound to x from 5 to 6 when mut is used. Ultimately, deciding whether to use mutability or not is up to you and depends on what you think is clearest in that particular situation.

当使用 mut 时，我们可以将绑定到 x 的值从 5 更改为 6。 最终，决定是否使用可变性取决于您，并且取决于您认为在特定情况下最清楚的内容。

### Constants常量
Like immutable variables, constants are values that are bound to a name and are not allowed to change, but there are a few differences between constants and variables.

First, you aren’t allowed to use mut with constants. Constants aren’t just immutable by default—they’re always immutable. You declare constants using the const keyword instead of the let keyword, and the type of the value must be annotated. We’ll cover types and type annotations in the next section, “Data Types”, so don’t worry about the details right now. Just know that you must always annotate the type.

Constants can be declared in any scope, including the global scope, which makes them useful for values that many parts of code need to know about.

The last difference is that constants may be set only to a constant expression, not the result of a value that could only be computed at runtime.

Here’s an example of a constant declaration:

与不可变变量一样，常量是绑定到名称且不允许更改的值，但常量和变量之间存在一些差异。

首先，不允许将 mut 与常量一起使用。 常量不仅在默认情况下是不可变的，而且始终是不可变的。 使用 const 关键字而不是 let 关键字声明常量，并且必须注释值的类型。 我们将在下一节“数据类型”中介绍类型和类型注释，所以现在不用担心细节。 只需知道您必须始终注释类型即可。

常量可以在任何范围内声明，包括全局范围，这使得它们对于代码的许多部分需要了解的值非常有用。

最后一个区别是常量只能设置为常量表达式，而不是只能在运行时计算的值的结果。

这是常量声明的示例：
```
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```
The constant’s name is THREE_HOURS_IN_SECONDS and its value is set to the result of multiplying 60 (the number of seconds in a minute) by 60 (the number of minutes in an hour) by 3 (the number of hours we want to count in this program). Rust’s naming convention for constants is to use all uppercase with underscores between words. The compiler is able to evaluate a limited set of operations at compile time, which lets us choose to write out this value in a way that’s easier to understand and verify, rather than setting this constant to the value 10,800. See the Rust Reference’s section on constant evaluation for more information on what operations can be used when declaring constants.

Constants are valid for the entire time a program runs, within the scope in which they were declared. This property makes constants useful for values in your application domain that multiple parts of the program might need to know about, such as the maximum number of points any player of a game is allowed to earn, or the speed of light.

Naming hardcoded values used throughout your program as constants is useful in conveying the meaning of that value to future maintainers of the code. It also helps to have only one place in your code you would need to change if the hardcoded value needed to be updated in the future.

该常量的名称为 THREE_HOURS_IN_SECONDS，其值设置为 60（一分钟的秒数）乘以 60（一小时的分钟数）乘以 3（我们要在此程序中计算的小时数） ）。 Rust 的常量命名约定是全部大写，单词之间使用下划线。 编译器能够在编译时评估一组有限的操作，这让我们可以选择以更容易理解和验证的方式写出该值，而不是将此常量设置为值 10,800。 有关声明常量时可以使用哪些操作的更多信息，请参阅 Rust 参考资料中有关常量求值的部分。

常量在程序运行的整个时间内（在声明它们的范围内）有效。 此属性使常量对于程序的多个部分可能需要了解的应用程序域中的值非常有用，例如允许游戏玩家获得的最大分数或光速。

将整个程序中使用的硬编码值命名为常量有助于将该值的含义传达给未来的代码维护者。 如果将来需要更新硬编码值，那么在代码中只需要更改一个位置也很有帮助。

### Shadowing(影没)

As you saw in the guessing game tutorial in Chapter 2, you can declare a new variable with the same name as a previous variable. Rustaceans say that the first variable is shadowed by the second, which means that the second variable is what the compiler will see when you use the name of the variable. In effect, the second variable overshadows the first, taking any uses of the variable name to itself until either it itself is shadowed or the scope ends. We can shadow a variable by using the same variable’s name and repeating the use of the let keyword as follows:

Filename: src/main.rs
正如您在第 2 章的猜谜游戏教程中看到的那样，您可以声明一个与先前变量同名的新变量。 Rustaceans 说第一个变量被第二个变量遮蔽，这意味着当您使用变量名称时编译器将看到第二个变量。 实际上，第二个变量覆盖第一个变量，将变量名称的任何使用都归为自身，直到它本身被覆盖或作用域结束。 我们可以通过使用相同的变量名称并重复使用 let 关键字来隐藏变量，如下所示：

文件名：src/main.rs
```
fn main() {
    let x = 5;

    let x = x + 1;

    {
        let x = x * 2;
        println!("The value of x in the inner scope is: {x}");
    }

    println!("The value of x is: {x}");
}
```

This program first binds x to a value of 5. Then it creates a new variable x by repeating let x =, taking the original value and adding 1 so the value of x is then 6. Then, within an inner scope created with the curly brackets, the third let statement also shadows x and creates a new variable, multiplying the previous value by 2 to give x a value of 12. When that scope is over, the inner shadowing ends and x returns to being 6. When we run this program, it will output the following:

该程序首先将 x 绑定到值 5。然后，它通过重复 let x = 创建一个新变量 x，取原始值并加 1，这样 x 的值就是 6。然后，在使用 curly 创建的内部作用域内 括号中，第三个 let 语句也遮蔽 x 并创建一个新变量，将先前的值乘以 2 得到 x 的值 12。当该范围结束时，内部遮蔽结束，x 返回到 6。当我们运行这个程序时 ，它将输出以下内容：

```
$ cargo run
   Compiling variables v0.1.0 (file:///projects/variables)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31s
     Running `target/debug/variables`
The value of x in the inner scope is: 12
The value of x is: 6

```
Shadowing is different from marking a variable as mut because we’ll get a compile-time error if we accidentally try to reassign to this variable without using the let keyword. By using let, we can perform a few transformations on a value but have the variable be immutable after those transformations have been completed.

The other difference between mut and shadowing is that because we’re effectively creating a new variable when we use the let keyword again, we can change the type of the value but reuse the same name. For example, say our program asks a user to show how many spaces they want between some text by inputting space characters, and then we want to store that input as a number:

隐藏与将变量标记为 mut 不同，因为如果我们不小心尝试在不使用 let 关键字的情况下重新分配给该变量，我们将收到编译时错误。 通过使用let，我们可以对一个值执行一些转换，但在这些转换完成后变量是不可变的。

mut 和 Shadowing 之间的另一个区别是，因为当我们再次使用 let 关键字时，我们实际上是在创建一个新变量，所以我们可以更改值的类型，但重复使用相同的名称。 例如，假设我们的程序要求用户通过输入空格字符来显示他们想要在某些文本之间有多少个空格，然后我们希望将该输入存储为数字：
```
    let spaces = "   ";
    let spaces = spaces.len();

```
The first spaces variable is a string type and the second spaces variable is a number type. Shadowing thus spares us from having to come up with different names, such as spaces_str and spaces_num; instead, we can reuse the simpler spaces name. However, if we try to use mut for this, as shown here, we’ll get a compile-time error:  
第一个空格变量是字符串类型，第二个空格变量是数字类型。 因此，阴影使我们不必想出不同的名称，例如spaces_str和spaces_num； 相反，我们可以重复使用更简单的空间名称。 但是，如果我们尝试使用 mut 来实现此目的，如下所示，我们将收到编译时错误：
```
    let mut spaces = "   ";
    spaces = spaces.len();

```
The error says we’re not allowed to mutate a variable’s type:  
这个错误告诉我们不能改变一个变量的类型:
```
$ cargo run
   Compiling variables v0.1.0 (file:///projects/variables)
error[E0308]: mismatched types
 --> src/main.rs:3:14
  |
2 |     let mut spaces = "   ";
  |                      ----- expected due to this value
3 |     spaces = spaces.len();
  |              ^^^^^^^^^^^^ expected `&str`, found `usize`

For more information about this error, try `rustc --explain E0308`.
error: could not compile `variables` due to previous error

```
Now that we’ve explored how variables work, let’s look at more data types they can have.  
现在我们已经探讨了变量的工作原理，让我们看看它们可以拥有的更多数据类型。

## 3.2 数据类型
Every value in Rust is of a certain data type, which tells Rust what kind of data is being specified so it knows how to work with that data. We’ll look at two data type subsets: scalar and compound.

Keep in mind that Rust is a statically typed language, which means that it must know the types of all variables at compile time. The compiler can usually infer what type we want to use based on the value and how we use it. In cases when many types are possible, such as when we converted a String to a numeric type using parse in the “Comparing the Guess to the Secret Number” section in Chapter 2, we must add a type annotation, like this:

```
let guess: u32 = "42".parse().expect("Not a number!");

```
If we don’t add the : u32 type annotation shown in the preceding code, Rust will display the following error, which means the compiler needs more information from us to know which type we want to use:

```
$ cargo build
   Compiling no_type_annotations v0.1.0 (file:///projects/no_type_annotations)
error[E0282]: type annotations needed
 --> src/main.rs:2:9
  |
2 |     let guess = "42".parse().expect("Not a number!");
  |         ^^^^^
  |
help: consider giving `guess` an explicit type
  |
2 |     let guess: _ = "42".parse().expect("Not a number!");
  |              +++

For more information about this error, try `rustc --explain E0282`.
error: could not compile `no_type_annotations` due to previous error

```

You’ll see different type annotations for other data types.

### 标量(Scalar Types)

A scalar type represents a single value. Rust has four primary scalar types: integers, floating-point numbers, Booleans, and characters. You may recognize these from other programming languages. Let’s jump into how they work in Rust.

#### Integer Types
An integer is a number without a fractional component. We used one integer type in Chapter 2, the u32 type. This type declaration indicates that the value it’s associated with should be an unsigned integer (signed integer types start with i instead of u) that takes up 32 bits of space. Table 3-1 shows the built-in integer types in Rust. We can use any of these variants to declare the type of an integer value.

整型是没有小数的部分的数字，
|长度|有符号|无符号|
|--|--|--|
|8-bit|i8|u8|
|16-bit|i16|u16|
|32-bit|i32|u32|
|64-bit|i64|u64|
|arch|isize|usize|

Each variant can be either signed or unsigned and has an explicit size. Signed and unsigned refer to whether it’s possible for the number to be negative—in other words, whether the number needs to have a sign with it (signed) or whether it will only ever be positive and can therefore be represented without a sign (unsigned). It’s like writing numbers on paper: when the sign matters, a number is shown with a plus sign or a minus sign; however, when it’s safe to assume the number is positive, it’s shown with no sign. Signed numbers are stored using two’s complement representation.

Each signed variant can store numbers from -(2n - 1) to 2n - 1 - 1 inclusive, where n is the number of bits that variant uses. So an i8 can store numbers from -(27) to 27 - 1, which equals -128 to 127. Unsigned variants can store numbers from 0 to 2n - 1, so a u8 can store numbers from 0 to 28 - 1, which equals 0 to 255.

Additionally, the isize and usize types depend on the architecture of the computer your program is running on, which is denoted in the table as “arch”: 64 bits if you’re on a 64-bit architecture and 32 bits if you’re on a 32-bit architecture.

You can write integer literals in any of the forms shown in Table 3-2. Note that number literals that can be multiple numeric types allow a type suffix, such as 57u8, to designate the type. Number literals can also use _ as a visual separator to make the number easier to read, such as 1_000, which will have the same value as if you had specified 1000.

每个变体可以是有符号的或无符号的，并且具有明确的大小。 有符号和无符号是指数字是否可能为负数，换句话说，数字是否需要带有符号（有符号），或者它是否只能为正数，因此可以在没有符号的情况下表示（无符号） ）。 这就像在纸上写数字：当符号重要时，数字会显示加号或减号；当符号重要时，数字会显示加号或减号； 然而，当可以安全地假设该数字为正数时，它会不显示任何符号。 有符号数使用二进制补码表示形式存储。

每个有符号变体可以存储从 -(2n - 1) 到 2n - 1 - 1（含）的数字，其中 n 是变体使用的位数。 因此，i8 可以存储从 -(27) 到 27 - 1 的数字，这等于 -128 到 127。无符号变体可以存储从 0 到 2n - 1 的数字，因此 u8 可以存储从 0 到 28 - 1 的数字，这等于 0 到 255。

此外， isize 和 usize 类型取决于程序运行所在计算机的体系结构，在表中表示为“arch”：如果使用 64 位体系结构，则为 64 位；如果使用 64 位体系结构，则为 32 位 在 32 位架构上。

您可以采用表 3-2 中所示的任何形式编写整数文字。 请注意，可以是多个数字类型的数字文字允许使用类型后缀（例如 57u8）来指定类型。 数字文字还可以使用 _ 作为视觉分隔符，以使数字更易于阅读，例如 1_000，其值与指定的 1000 相同。
整型字面值
|数字字面值|例子|
|--|--|
|10进制Decimal|98_222|
|16进制Hex|0xff|
|8进制Octal|0o77|
|Binary二进制|0b111_000|
|Byte(u8 only)字节|b'A'|

So how do you know which type of integer to use? If you’re unsure, Rust’s defaults are generally good places to start: integer types default to i32. The primary situation in which you’d use isize or usize is when indexing some sort of collection.

**Integer Overflow**

Let’s say you have a variable of type u8 that can hold values between 0 and 255. If you try to change the variable to a value outside that range, such as 256, integer overflow will occur, which can result in one of two behaviors. When you’re compiling in debug mode, Rust includes checks for integer overflow that cause your program to panic at runtime if this behavior occurs. Rust uses the term panicking when a program exits with an error; we’ll discuss panics in more depth in the “Unrecoverable Errors with panic!” section in Chapter 9.

When you’re compiling in release mode with the --release flag, Rust does not include checks for integer overflow that cause panics. Instead, if overflow occurs, Rust performs two’s complement wrapping. In short, values greater than the maximum value the type can hold “wrap around” to the minimum of the values the type can hold. In the case of a u8, the value 256 becomes 0, the value 257 becomes 1, and so on. The program won’t panic, but the variable will have a value that probably isn’t what you were expecting it to have. Relying on integer overflow’s wrapping behavior is considered an error.

To explicitly handle the possibility of overflow, you can use these families of methods provided by the standard library for primitive numeric types:

Wrap in all modes with the wrapping_* methods, such as wrapping_add.
Return the None value if there is overflow with the checked_* methods.
Return the value and a boolean indicating whether there was overflow with the overflowing_* methods.
Saturate at the value’s minimum or maximum values with the saturating_* methods.

整数溢出
假设您有一个 u8 类型的变量，它可以保存 0 到 255 之间的值。如果您尝试将该变量更改为该范围之外的值（例如 256），则会发生整数溢出，这可能会导致以下两种行为之一。 当您在调试模式下进行编译时，Rust 会检查整数溢出，如果发生这种行为，则会导致程序在运行时出现恐慌。 当程序因错误退出时，Rust 使用术语“恐慌”。 我们将在“不可恢复的恐慌错误！”中更深入地讨论恐慌。 第 9 章中的部分。

当您使用 --release 标志在发布模式下进行编译时，Rust 不包括对导致恐慌的整数溢出的检查。 相反，如果发生溢出，Rust 会执行二进制补码换行。 简而言之，大于该类型可以容纳的最大值的值“环绕”到该类型可以容纳的最小值。 在 u8 的情况下，值 256 变为 0，值 257 变为 1，依此类推。 程序不会出现恐慌，但变量的值可能不是您期望的值。 依赖整数溢出的包装行为被视为错误。

要显式处理溢出的可能性，您可以使用标准库为原始数字类型提供的以下一系列方法：

使用wrapping_* 方法包裹所有模式，例如wrapping_add。
如果 check_* 方法存在溢出，则返回 None 值。
返回值和一个布尔值，指示 Overflowing_* 方法是否存在溢出。
使用 saturating_* 方法在值的最小值或最大值处饱和。

### Floating-Point Types
Rust also has two primitive types for floating-point numbers, which are numbers with decimal points. Rust’s floating-point types are f32 and f64, which are 32 bits and 64 bits in size, respectively. The default type is f64 because on modern CPUs, it’s roughly the same speed as f32 but is capable of more precision. All floating-point types are signed.

Here’s an example that shows floating-point numbers in action:

Filename: src/main.rs

Rust 还有两种浮点数的基本类型，即带小数点的数字。 Rust 的浮点类型是 f32 和 f64，其大小分别为 32 位和 64 位。 默认类型是 f64，因为在现代 CPU 上，它的速度与 f32 大致相同，但精度更高。 所有浮点类型都有符号。

下面是一个显示浮点数实际操作的示例：
```
fn main() {
    let x = 2.0; // f64

    let y: f32 = 3.0; // f32
}
```

Floating-point numbers are represented according to the IEEE-754 standard. The f32 type is a single-precision float, and f64 has double precision.

### Numeric Operations
Rust supports the basic mathematical operations you’d expect for all the number types: addition, subtraction, multiplication, division, and remainder. Integer division truncates toward zero to the nearest integer. The following code shows how you’d use each numeric operation in a let statement:

Filename: src/main.rs
```
fn main() {
    // addition
    let sum = 5 + 10;

    // subtraction
    let difference = 95.5 - 4.3;

    // multiplication
    let product = 4 * 30;

    // division
    let quotient = 56.7 / 32.2;
    let truncated = -5 / 3; // Results in -1

    // remainder
    let remainder = 43 % 5;
}
```

Each expression in these statements uses a mathematical operator and evaluates to a single value, which is then bound to a variable. Appendix B contains a list of all operators that Rust provides.

### The Boolean Type
As in most other programming languages, a Boolean type in Rust has two possible values: true and false. Booleans are one byte in size. The Boolean type in Rust is specified using bool. For example:

Filename: src/main.rs

```
fn main() {
    let t = true;

    let f: bool = false; // with explicit type annotation
}
```

The main way to use Boolean values is through conditionals, such as an if expression. We’ll cover how if expressions work in Rust in the “Control Flow” section.

### The Character Type
Rust’s char type is the language’s most primitive alphabetic type. Here are some examples of declaring char values:

Filename: src/main.rs

```
fn main() {
    let c = 'z';
    let z: char = 'ℤ'; // with explicit type annotation
    let heart_eyed_cat = '😻';
}
```

Note that we specify char literals with single quotes, as opposed to string literals, which use double quotes. Rust’s char type is four bytes in size and represents a Unicode Scalar Value, which means it can represent a lot more than just ASCII. Accented letters; Chinese, Japanese, and Korean characters; emoji; and zero-width spaces are all valid char values in Rust. Unicode Scalar Values range from U+0000 to U+D7FF and U+E000 to U+10FFFF inclusive. However, a “character” isn’t really a concept in Unicode, so your human intuition for what a “character” is may not match up with what a char is in Rust. We’ll discuss this topic in detail in “Storing UTF-8 Encoded Text with Strings” in Chapter 8.

请注意，我们使用单引号指定 char 文字，而不是使用双引号的字符串文字。 Rust 的 char 类型大小为 4 个字节，表示 Unicode 标量值，这意味着它可以表示的不仅仅是 ASCII。 重音字母； 中文、日文、韩文字符； 表情符号； 和零宽度空格都是 Rust 中的有效 char 值。 Unicode 标量值范围从 U+0000 到 U+D7FF 以及 U+E000 到 U+10FFFF（含）。 然而，“字符”并不是 Unicode 中真正的概念，因此你对“字符”的人类直觉可能与 Rust 中的 char 不相符。 我们将在第 8 章的“用字符串存储 UTF-8 编码文本”中详细讨论这个主题。

### Compound Types

Compound types can group multiple values into one type. Rust has two primitive compound types: tuples and arrays.

The Tuple Type
A tuple is a general way of grouping together a number of values with a variety of types into one compound type. Tuples have a fixed length: once declared, they cannot grow or shrink in size.

We create a tuple by writing a comma-separated list of values inside parentheses. Each position in the tuple has a type, and the types of the different values in the tuple don’t have to be the same. We’ve added optional type annotations in this example:

复合类型可以将多个值分组为一种类型。 Rust 有两种原始复合类型：元组和数组。

元组类型
元组是将具有多种类型的多个值组合在一起形成一个复合类型的通用方法。 元组有固定的长度：一旦声明，它们的大小就不能增长或缩小。

我们通过在括号内写入逗号分隔的值列表来创建一个元组。 元组中的每个位置都有一个类型，并且元组中不同值的类型不必相同。 我们在此示例中添加了可选的类型注释：

Filename: src/main.rs

```
fn main() {
    let tup: (i32, f64, u8) = (500, 6.4, 1);
}
```

This program first creates a tuple and binds it to the variable tup. It then uses a pattern with let to take tup and turn it into three separate variables, x, y, and z. This is called destructuring because it breaks the single tuple into three parts. Finally, the program prints the value of y, which is 6.4.

We can also access a tuple element directly by using a period (.) followed by the index of the value we want to access. For example:

该程序首先创建一个元组并将其绑定到变量 tup。 然后，它使用带有 let 的模式来获取 tup 并将其转换为三个单独的变量 x、y 和 z。 这称为解构，因为它将单个元组分解为三个部分。 最后，程序打印 y 的值，即 6.4。

我们还可以通过使用句点 (.) 后跟要访问的值的索引来直接访问元组元素。 例如：

Filename: src/main.rs

```
fn main() {
    let x: (i32, f64, u8) = (500, 6.4, 1);

    let five_hundred = x.0;

    let six_point_four = x.1;

    let one = x.2;
}
```
This program creates the tuple x and then accesses each element of the tuple using their respective indices. As with most programming languages, the first index in a tuple is 0.

The tuple without any values has a special name, unit. This value and its corresponding type are both written () and represent an empty value or an empty return type. Expressions implicitly return the unit value if they don’t return any other value.

该程序创建元组 x，然后使用各自的索引访问元组的每个元素。 与大多数编程语言一样，元组中的第一个索引是 0。

没有任何值的元组有一个特殊的名称：unit。 这个值和它对应的类型都写成()，表示一个空值或者一个空返回类型。 如果表达式不返回任何其他值，则隐式返回单位值。

### The Array Type
Another way to have a collection of multiple values is with an array. Unlike a tuple, every element of an array must have the same type. Unlike arrays in some other languages, arrays in Rust have a fixed length.

We write the values in an array as a comma-separated list inside square brackets:

拥有多个值的集合的另一种方法是使用数组。 与元组不同，数组的每个元素必须具有相同的类型。 与其他一些语言中的数组不同，Rust 中的数组具有固定长度。

我们将数组中的值写为方括号内的逗号分隔列表：

Filename: src/main.rs

```
fn main() {
    let a = [1, 2, 3, 4, 5];
}
```

Arrays are useful when you want your data allocated on the stack rather than the heap (we will discuss the stack and the heap more in Chapter 4) or when you want to ensure you always have a fixed number of elements. An array isn’t as flexible as the vector type, though. A vector is a similar collection type provided by the standard library that is allowed to grow or shrink in size. If you’re unsure whether to use an array or a vector, chances are you should use a vector. Chapter 8 discusses vectors in more detail.

However, arrays are more useful when you know the number of elements will not need to change. For example, if you were using the names of the month in a program, you would probably use an array rather than a vector because you know it will always contain 12 elements:

当您希望将数据分配在堆栈而不是堆上（我们将在第 4 章中详细讨论堆栈和堆）或者您希望确保始终拥有固定数量的元素时，数组非常有用。 不过，数组不如向量类型灵活。 矢量是标准库提供的类似集合类型，允许大小增长或缩小。 如果您不确定是使用数组还是向量，那么您很可能应该使用向量。 第 8 章更详细地讨论向量。

但是，当您知道元素数量不需要更改时，数组会更有用。 例如，如果您在程序中使用月份名称，您可能会使用数组而不是向量，因为您知道它始终包含 12 个元素：

```
let months = ["January", "February", "March", "April", "May", "June", "July",
              "August", "September", "October", "November", "December"];

```

You write an array’s type using square brackets with the type of each element, a semicolon, and then the number of elements in the array, like so:
```
let a: [i32; 5] = [1, 2, 3, 4, 5];

```
Here, i32 is the type of each element. After the semicolon, the number 5 indicates the array contains five elements.

You can also initialize an array to contain the same value for each element by specifying the initial value, followed by a semicolon, and then the length of the array in square brackets, as shown here:
```
let a = [3; 5];

```
#### Accessing Array Elements
An array is a single chunk of memory of a known, fixed size that can be allocated on the stack. You can access elements of an array using indexing, like this:

Filename: src/main.rs
```
fn main() {
    let a = [1, 2, 3, 4, 5];

    let first = a[0];
    let second = a[1];
}
```
In this example, the variable named first will get the value 1 because that is the value at index [0] in the array. The variable named second will get the value 2 from index [1] in the array.

#### Invalid Array Element Access
Let’s see what happens if you try to access an element of an array that is past the end of the array. Say you run this code, similar to the guessing game in Chapter 2, to get an array index from the user:

Filename: src/main.rs

```
use std::io;

fn main() {
    let a = [1, 2, 3, 4, 5];

    println!("Please enter an array index.");

    let mut index = String::new();

    io::stdin()
        .read_line(&mut index)
        .expect("Failed to read line");

    let index: usize = index
        .trim()
        .parse()
        .expect("Index entered was not a number");

    let element = a[index];

    println!("The value of the element at index {index} is: {element}");
}
```
This code compiles successfully. If you run this code using cargo run and enter 0, 1, 2, 3, or 4, the program will print out the corresponding value at that index in the array. If you instead enter a number past the end of the array, such as 10, you’ll see output like this:

```
thread 'main' panicked at 'index out of bounds: the len is 5 but the index is 10', src/main.rs:19:19
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace

```

The program resulted in a runtime error at the point of using an invalid value in the indexing operation. The program exited with an error message and didn’t execute the final println! statement. When you attempt to access an element using indexing, Rust will check that the index you’ve specified is less than the array length. If the index is greater than or equal to the length, Rust will panic. This check has to happen at runtime, especially in this case, because the compiler can’t possibly know what value a user will enter when they run the code later.

This is an example of Rust’s memory safety principles in action. In many low-level languages, this kind of check is not done, and when you provide an incorrect index, invalid memory can be accessed. Rust protects you against this kind of error by immediately exiting instead of allowing the memory access and continuing. Chapter 9 discusses more of Rust’s error handling and how you can write readable, safe code that neither panics nor allows invalid memory access.

该程序在索引操作中使用无效值时导致运行时错误。 程序退出并显示错误消息，并且没有执行最终的 println! 陈述。 当您尝试使用索引访问元素时，Rust 将检查您指定的索引是否小于数组长度。 如果索引大于或等于长度，Rust 就会出现恐慌。 此检查必须在运行时进行，尤其是在这种情况下，因为编译器不可能知道用户稍后运行代码时将输入什么值。

这是 Rust 内存安全原则实际应用的一个例子。 在许多低级语言中，没有进行这种检查，并且当您提供不正确的索引时，可以访问无效的内存。 Rust 通过立即退出而不是允许内存访问并继续来保护您免受此类错误的影响。 第 9 章详细讨论了 Rust 的错误处理，以及如何编写既不会发生恐慌也不会允许无效内存访问的可读、安全的代码。
## 3.3 函数

Functions are prevalent in Rust code. You’ve already seen one of the most important functions in the language: the main function, which is the entry point of many programs. You’ve also seen the fn keyword, which allows you to declare new functions.

Rust code uses snake case as the conventional style for function and variable names, in which all letters are lowercase and underscores separate words. Here’s a program that contains an example function definition:

函数在 Rust 代码中很常见。 您已经看到了该语言中最重要的函数之一：主函数，它是许多程序的入口点。 您还看到了 fn 关键字，它允许您声明新函数。

Rust 代码使用蛇形命名法作为函数和变量名称的常规样式，其中所有字母均为小写，并为单独的单词添加下划线。 这是一个包含示例函数定义的程序：

Filename: src/main.rs
```
fn main() {
    println!("Hello, world!");

    another_function();
}

fn another_function() {
    println!("Another function.");
}
```
We define a function in Rust by entering fn followed by a function name and a set of parentheses. The curly brackets tell the compiler where the function body begins and ends.

We can call any function we’ve defined by entering its name followed by a set of parentheses. Because another_function is defined in the program, it can be called from inside the main function. Note that we defined another_function after the main function in the source code; we could have defined it before as well. Rust doesn’t care where you define your functions, only that they’re defined somewhere in a scope that can be seen by the caller.

Let’s start a new binary project named functions to explore functions further. Place the another_function example in src/main.rs and run it. You should see the following output:

我们通过输入 fn 后跟函数名和一组括号来定义 Rust 中的函数。 大括号告诉编译器函数体的开始和结束位置。

我们可以通过输入其名称后跟一组括号来调用我们定义的任何函数。 因为another_function是在程序中定义的，所以可以从main函数内部调用它。 注意，我们在源代码中的main函数之后定义了another_function； 我们之前也可以定义它。 Rust 并不关心你在哪里定义函数，只关心它们是在调用者可以看到的作用域中的某个地方定义的。

让我们启动一个名为 Functions 的新二进制项目来进一步探索函数。 将 another_function 示例放入 src/main.rs 并运行它。 您应该看到以下输出：

```
$ cargo run
   Compiling functions v0.1.0 (file:///projects/functions)
    Finished dev [unoptimized + debuginfo] target(s) in 0.28s
     Running `target/debug/functions`
Hello, world!
Another function.

```
The lines execute in the order in which they appear in the main function. First the “Hello, world!” message prints, and then another_function is called and its message is printed.

### Parameters
We can define functions to have parameters, which are special variables that are part of a function’s signature. When a function has parameters, you can provide it with concrete values for those parameters. Technically, the concrete values are called arguments, but in casual conversation, people tend to use the words parameter and argument interchangeably for either the variables in a function’s definition or the concrete values passed in when you call a function.

我们可以定义具有参数的函数，这些参数是作为函数签名一部分的特殊变量。 当函数有参数时，您可以为其提供这些参数的具体值。 从技术上讲，具体值称为参数，但在日常对话中，人们倾向于交替使用“参数”和“参数”这两个词来表示函数定义中的变量或调用函数时传入的具体值。

In this version of another_function we add a parameter:

Filename: src/main.rs

```
fn main() {
    another_function(5);
}

fn another_function(x: i32) {
    println!("The value of x is: {x}");
}
```
Try running this program; you should get the following output:

```
$ cargo run
   Compiling functions v0.1.0 (file:///projects/functions)
    Finished dev [unoptimized + debuginfo] target(s) in 1.21s
     Running `target/debug/functions`
The value of x is: 5

```
The declaration of another_function has one parameter named x. The type of x is specified as i32. When we pass 5 in to another_function, the println! macro puts 5 where the pair of curly brackets containing x was in the format string.

In function signatures, you must declare the type of each parameter. This is a deliberate decision in Rust’s design: requiring type annotations in function definitions means the compiler almost never needs you to use them elsewhere in the code to figure out what type you mean. The compiler is also able to give more helpful error messages if it knows what types the function expects.

another_function 的声明有一个名为 x 的参数。 x 的类型指定为 i32。 当我们将 5 传递给 another_function 时， println! 宏将 5 放入格式字符串中包含 x 的大括号对中。

在函数签名中，必须声明每个参数的类型。 这是 Rust 设计中经过深思熟虑的决定：在函数定义中要求类型注释意味着编译器几乎不需要您在代码中的其他地方使用它们来弄清楚您所指的类型。 如果编译器知道函数需要什么类型，它还能够给出更有用的错误消息。

When defining multiple parameters, separate the parameter declarations with commas, like this:

Filename: src/main.rs
```
fn main() {
    print_labeled_measurement(5, 'h');
}

fn print_labeled_measurement(value: i32, unit_label: char) {
    println!("The measurement is: {value}{unit_label}");
}
```
This example creates a function named print_labeled_measurement with two parameters. The first parameter is named value and is an i32. The second is named unit_label and is type char. The function then prints text containing both the value and the unit_label.

Let’s try running this code. Replace the program currently in your functions project’s src/main.rs file with the preceding example and run it using cargo run:

此示例创建一个名为 print_labeled_measurement 的函数，该函数具有两个参数。 第一个参数名为 value，是一个 i32。 第二个名为unit_label，类型为char。 然后该函数打印包含值和单位标签的文本。

让我们尝试运行这段代码。 将函数项目 src/main.rs 文件中当前的程序替换为前面的示例，并使用 Cargo run 运行它：

```
$ cargo run
   Compiling functions v0.1.0 (file:///projects/functions)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31s
     Running `target/debug/functions`
The measurement is: 5h

```
Because we called the function with 5 as the value for value and 'h' as the value for unit_label, the program output contains those values.

### Statements and Expressions
Function bodies are made up of a series of statements optionally ending in an expression. So far, the functions we’ve covered haven’t included an ending expression, but you have seen an expression as part of a statement. Because Rust is an expression-based language, this is an important distinction to understand. Other languages don’t have the same distinctions, so let’s look at what statements and expressions are and how their differences affect the bodies of functions.

+ Statements are instructions that perform some action and do not return a value.
+ Expressions evaluate to a resultant value. Let’s look at some examples.

We’ve actually already used statements and expressions. Creating a variable and assigning a value to it with the let keyword is a statement. In Listing 3-1, let y = 6; is a statement.

函数体由一系列可选地以表达式结尾的语句组成。 到目前为止，我们介绍的函数尚未包含结束表达式，但您已经看到表达式是语句的一部分。 因为 Rust 是一种基于表达式的语言，所以这是一个需要理解的重要区别。 其他语言没有相同的区别，所以让我们看看什么是语句和表达式以及它们的差异如何影响函数体。

+ 语句是执行某些操作但不返回值的指令。
+ 表达式计算结果值。 让我们看一些例子。

我们实际上已经使用过语句和表达式。 创建变量并使用 let 关键字为其赋值是一条语句。 在清单 3-1 中，令 y = 6； 是一个声明。

Filename: src/main.rs
```
fn main() {
    let y = 6;
}
```
Listing 3-1: A main function declaration containing one statement

Function definitions are also statements; the entire preceding example is a statement in itself.

Statements do not return values. Therefore, you can’t assign a let statement to another variable, as the following code tries to do; you’ll get an error:

函数定义也是语句； 前面的整个例子本身就是一个陈述。

语句不返回值。 因此，您不能将 let 语句分配给另一个变量，如以下代码尝试执行的操作； 你会得到一个错误：

Filename: src/main.rs
```
fn main() {
    let x = (let y = 6);
}
```
When you run this program, the error you’ll get looks like this:

```
$ cargo run
   Compiling functions v0.1.0 (file:///projects/functions)
error: expected expression, found `let` statement
 --> src/main.rs:2:14
  |
2 |     let x = (let y = 6);
  |              ^^^

error: expected expression, found statement (`let`)
 --> src/main.rs:2:14
  |
2 |     let x = (let y = 6);
  |              ^^^^^^^^^
  |
  = note: variable declaration using `let` is a statement

error[E0658]: `let` expressions in this position are unstable
 --> src/main.rs:2:14
  |
2 |     let x = (let y = 6);
  |              ^^^^^^^^^
  |
  = note: see issue #53667 <https://github.com/rust-lang/rust/issues/53667> for more information

warning: unnecessary parentheses around assigned value
 --> src/main.rs:2:13
  |
2 |     let x = (let y = 6);
  |             ^         ^
  |
  = note: `#[warn(unused_parens)]` on by default
help: remove these parentheses
  |
2 -     let x = (let y = 6);
2 +     let x = let y = 6;
  |

For more information about this error, try `rustc --explain E0658`.
warning: `functions` (bin "functions") generated 1 warning
error: could not compile `functions` due to 3 previous errors; 1 warning emitted

```
The let y = 6 statement does not return a value, so there isn’t anything for x to bind to. This is different from what happens in other languages, such as C and Ruby, where the assignment returns the value of the assignment. In those languages, you can write x = y = 6 and have both x and y have the value 6; that is not the case in Rust.

Expressions evaluate to a value and make up most of the rest of the code that you’ll write in Rust. Consider a math operation, such as 5 + 6, which is an expression that evaluates to the value 11. Expressions can be part of statements: in Listing 3-1, the 6 in the statement let y = 6; is an expression that evaluates to the value 6. Calling a function is an expression. Calling a macro is an expression. A new scope block created with curly brackets is an expression, for example:

let y = 6 语句不返回值，因此 x 没有任何可绑定的内容。 这与其他语言（例如 C 和 Ruby）中发生的情况不同，在其他语言中，赋值返回赋值的值。 在这些语言中，您可以写成 x = y = 6 并且 x 和 y 的值为 6； Rust 中的情况并非如此。

表达式求值并构成您将用 Rust 编写的其余代码的大部分。 考虑一个数学运算，例如 5 + 6，它是一个计算结果为 11 的表达式。表达式可以是语句的一部分：在清单 3-1 中，语句中的 6 let y = 6; 是一个计算结果为 6 的表达式。调用函数是一个表达式。 调用宏是一个表达式。 使用大括号创建的新作用域块是一个表达式，例如：

Filename: src/main.rs

```
fn main() {
    let y = {
        let x = 3;
        x + 1
    };

    println!("The value of y is: {y}");
}
```

This expression:
```
{
    let x = 3;
    x + 1
}
```
is a block that, in this case, evaluates to 4. That value gets bound to y as part of the let statement. Note that the x + 1 line doesn’t have a semicolon at the end, which is unlike most of the lines you’ve seen so far. Expressions do not include ending semicolons. If you add a semicolon to the end of an expression, you turn it into a statement, and it will then not return a value. Keep this in mind as you explore function return values and expressions next.

是一个块，在本例中，其计算结果为 4。该值作为 let 语句的一部分绑定到 y。 请注意，x + 1 行末尾没有分号，这与您目前看到的大多数行不同。 表达式不包含结束分号。 如果在表达式末尾添加分号，则将其转换为语句，并且它不会返回值。 当您接下来探索函数返回值和表达式时，请记住这一点。

### Functions with Return Values

Functions can return values to the code that calls them. We don’t name return values, but we must declare their type after an arrow (->). In Rust, the return value of the function is synonymous with the value of the final expression in the block of the body of a function. You can return early from a function by using the return keyword and specifying a value, but most functions return the last expression implicitly. Here’s an example of a function that returns a value:

函数可以将值返回给调用它们的代码。 我们不命名返回值，但必须在箭头 (->) 之后声明它们的类型。 在 Rust 中，函数的返回值与函数体块中最终表达式的值同义。 您可以通过使用 return 关键字并指定一个值来提前从函数返回，但大多数函数都会隐式返回最后一个表达式。 下面是一个返回值的函数示例：

Filename: src/main.rs

```
fn five() -> i32 {
    5
}

fn main() {
    let x = five();

    println!("The value of x is: {x}");
}
```
There are no function calls, macros, or even let statements in the five function—just the number 5 by itself. That’s a perfectly valid function in Rust. Note that the function’s return type is specified too, as -> i32. Try running this code; the output should look like this:

五个函数中没有函数调用、宏，甚至没有 let 语句，只有数字 5 本身。 这是 Rust 中一个完全有效的函数。 请注意，函数的返回类型也被指定为 -> i32。 尝试运行这段代码； 输出应如下所示：

```
$ cargo run
   Compiling functions v0.1.0 (file:///projects/functions)
    Finished dev [unoptimized + debuginfo] target(s) in 0.30s
     Running `target/debug/functions`
The value of x is: 5

```
The 5 in five is the function’s return value, which is why the return type is i32. Let’s examine this in more detail. There are two important bits: first, the line let x = five(); shows that we’re using the return value of a function to initialize a variable. Because the function five returns a 5, that line is the same as the following:

五分之五是函数的返回值，这就是返回类型为 i32 的原因。 让我们更详细地研究一下这一点。 有两个重要的位：首先，行 let x = Five(); 表明我们正在使用函数的返回值来初始化变量。 因为函数 5 返回 5，所以该行与以下内容相同：

```
let xt = 5;
```
Second, the five function has no parameters and defines the type of the return value, but the body of the function is a lonely 5 with no semicolon because it’s an expression whose value we want to return.

其次，五个函数没有参数并定义了返回值的类型，但函数体是一个没有分号的孤独的 5，因为它是一个我们想要返回其值的表达式。

Let’s look at another example:

Filename: src/main.rs

```
fn main() {
    let x = plus_one(5);

    println!("The value of x is: {x}");
}

fn plus_one(x: i32) -> i32 {
    x + 1
}
```
Running this code will print The value of x is: 6. But if we place a semicolon at the end of the line containing x + 1, changing it from an expression to a statement, we’ll get an error:

Filename: src/main.rs

```
fn main() {
    let x = plus_one(5);

    println!("The value of x is: {x}");
}

fn plus_one(x: i32) -> i32 {
    x + 1;
}
```
Compiling this code produces an error, as follows:
```
$ cargo run
   Compiling functions v0.1.0 (file:///projects/functions)
error[E0308]: mismatched types
 --> src/main.rs:7:24
  |
7 | fn plus_one(x: i32) -> i32 {
  |    --------            ^^^ expected `i32`, found `()`
  |    |
  |    implicitly returns `()` as its body has no tail or `return` expression
8 |     x + 1;
  |          - help: remove this semicolon to return this value

For more information about this error, try `rustc --explain E0308`.
error: could not compile `functions` due to previous error

```
The main error message, mismatched types, reveals the core issue with this code. The definition of the function plus_one says that it will return an i32, but statements don’t evaluate to a value, which is expressed by (), the unit type. Therefore, nothing is returned, which contradicts the function definition and results in an error. In this output, Rust provides a message to possibly help rectify this issue: it suggests removing the semicolon, which would fix the error.

主要错误消息（类型不匹配）揭示了此代码的核心问题。 函数 plus_one 的定义表明它将返回一个 i32，但语句不会求值，该值由单位类型 () 表示。 因此，不会返回任何内容，这与函数定义相矛盾并导致错误。 在此输出中，Rust 提供了一条消息，可能有助于纠正此问题：它建议删除分号，这将修复错误。

## 3.4 注释

All programmers strive to make their code easy to understand, but sometimes extra explanation is warranted. In these cases, programmers leave comments in their source code that the compiler will ignore but people reading the source code may find useful.

Here’s a simple comment:

所有程序员都努力使他们的代码易于理解，但有时需要额外的解释。 在这些情况下，程序员会在源代码中留下注释，编译器会忽略这些注释，但阅读源代码的人可能会发现有用。

这是一个简单的评论：
```
// hello, world
```
In Rust, the idiomatic comment style starts a comment with two slashes, and the comment continues until the end of the line. For comments that extend beyond a single line, you’ll need to include // on each line, like this:  
在 Rust 中，惯用的注释风格以两个斜杠开始注释，并且注释一直持续到行尾。 对于超出单行的注释，您需要在每一行中包含//，如下所示：
```
// So we’re doing something complicated here, long enough that we need
// multiple lines of comments to do it! Whew! Hopefully, this comment will
// explain what’s going on.

```
Comments can also be placed at the end of lines containing code:

Filename: src/main.rs
```
fn main() {
    let lucky_number = 7; // I’m feeling lucky today
}
```
But you’ll more often see them used in this format, with the comment on a separate line above the code it’s annotating:  
但您会更经常看到它们以这种格式使用，注释位于其注释的代码上方的单独行上：

Filename: src/main.rs

```
fn main() {
    // I’m feeling lucky today
    let lucky_number = 7;
}
```
Rust also has another kind of comment, documentation comments, which we’ll discuss in the “Publishing a Crate to Crates.io” section of Chapter 14.  
在14章我们讨论另一种注释方式，文档注释。

## 3.5 控制流
The ability to run some code depending on whether a condition is true and to run some code repeatedly while a condition is true are basic building blocks in most programming languages. The most common constructs that let you control the flow of execution of Rust code are if expressions and loops.

根据条件是否为真来运行某些代码以及在条件为真时重复运行某些代码的能力是大多数编程语言中的基本构建块。 让您控制 Rust 代码执行流程的最常见结构是 if 表达式和循环。

### if Expressions
An if expression allows you to branch your code depending on conditions. You provide a condition and then state, “If this condition is met, run this block of code. If the condition is not met, do not run this block of code.”

Create a new project called branches in your projects directory to explore the if expression. In the src/main.rs file, input the following:

if 表达式允许您根据条件分支代码。 您提供一个条件，然后声明：“如果满足此条件，则运行此代码块。 如果不满足条件，则不要运行该代码块。”

在项目目录中创建一个名为 Branches 的新项目来探索 if 表达式。 在 src/main.rs 文件中，输入以下内容：

Filename: src/main.rs
```
fn main() {
    let number = 3;

    if number < 5 {
        println!("condition was true");
    } else {
        println!("condition was false");
    }
}
```
All if expressions start with the keyword if, followed by a condition. In this case, the condition checks whether or not the variable number has a value less than 5. We place the block of code to execute if the condition is true immediately after the condition inside curly brackets. Blocks of code associated with the conditions in if expressions are sometimes called arms, just like the arms in match expressions that we discussed in the “Comparing the Guess to the Secret Number” section of Chapter 2.

Optionally, we can also include an else expression, which we chose to do here, to give the program an alternative block of code to execute should the condition evaluate to false. If you don’t provide an else expression and the condition is false, the program will just skip the if block and move on to the next bit of code.

所有 if 表达式都以关键字 if 开头，后跟条件。 在本例中，条件检查变量 number 的值是否小于 5。如果条件为真，我们将要执行的代码块放在大括号内的条件之后。 与 if 表达式中的条件相关的代码块有时称为 Arm，就像我们在第 2 章的“比较猜测与秘密数字”部分中讨论的 match 表达式中的 Arm。

或者，我们还可以包含一个 else 表达式，我们在这里选择这样做，以便在条件评估为 false 时为程序提供一个要执行的替代代码块。 如果您不提供 else 表达式并且条件为 false，则程序将跳过 if 块并继续执行下一段代码。

Try running this code; you should see the following output:

```
$ cargo run
   Compiling branches v0.1.0 (file:///projects/branches)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31s
     Running `target/debug/branches`
condition was true

```
Let’s try changing the value of number to a value that makes the condition false to see what happens:
```
let number = 7;

```
Run the program again, and look at the output:

```
$ cargo run
   Compiling branches v0.1.0 (file:///projects/branches)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31s
     Running `target/debug/branches`
condition was false

```
It’s also worth noting that the condition in this code must be a bool. If the condition isn’t a bool, we’ll get an error. For example, try running the following code:

Filename: src/main.rs

```
fn main() {
    let number = 3;

    if number {
        println!("number was three");
    }
}
```
The if condition evaluates to a value of 3 this time, and Rust throws an error:
```
$ cargo run
   Compiling branches v0.1.0 (file:///projects/branches)
error[E0308]: mismatched types
 --> src/main.rs:4:8
  |
4 |     if number {
  |        ^^^^^^ expected `bool`, found integer

For more information about this error, try `rustc --explain E0308`.
error: could not compile `branches` due to previous error

```
The error indicates that Rust expected a bool but got an integer. Unlike languages such as Ruby and JavaScript, Rust will not automatically try to convert non-Boolean types to a Boolean. You must be explicit and always provide if with a Boolean as its condition. If we want the if code block to run only when a number is not equal to 0, for example, we can change the if expression to the following:

该错误表明 Rust 期望是布尔值，但得到的是整数。 与 Ruby 和 JavaScript 等语言不同，Rust 不会自动尝试将非布尔类型转换为布尔类型。 您必须明确并始终提供以布尔值作为条件的 if。 例如，如果我们希望 if 代码块仅在数字不等于 0 时运行，我们可以将 if 表达式更改为以下内容：

Filename: src/main.rs
```
fn main() {
    let number = 3;

    if number != 0 {
        println!("number was something other than zero");
    }
}
```
Running this code will print number was something other than zero.

### Handling Multiple Conditions with else if
You can use multiple conditions by combining if and else in an else if expression. For example:

Filename: src/main.rs

```
fn main() {
    let number = 6;

    if number % 4 == 0 {
        println!("number is divisible by 4");
    } else if number % 3 == 0 {
        println!("number is divisible by 3");
    } else if number % 2 == 0 {
        println!("number is divisible by 2");
    } else {
        println!("number is not divisible by 4, 3, or 2");
    }
}
```
This program has four possible paths it can take. After running it, you should see the following output:
```
$ cargo run
   Compiling branches v0.1.0 (file:///projects/branches)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31s
     Running `target/debug/branches`
number is divisible by 3
```

When this program executes, it checks each if expression in turn and executes the first body for which the condition evaluates to true. Note that even though 6 is divisible by 2, we don’t see the output number is divisible by 2, nor do we see the number is not divisible by 4, 3, or 2 text from the else block. That’s because Rust only executes the block for the first true condition, and once it finds one, it doesn’t even check the rest.

Using too many else if expressions can clutter your code, so if you have more than one, you might want to refactor your code. Chapter 6 describes a powerful Rust branching construct called match for these cases.

当该程序执行时，它依次检查每个 if 表达式并执行条件评估为 true 的第一个主体。 请注意，即使 6 可以被 2 整除，我们也看不到输出数字可以被 2 整除，也没有看到该数字不能被 else 块中的 4、3 或 2 整除。 这是因为 Rust 只执行第一个 true 条件的块，一旦找到一个，它甚至不会检查其余的。

使用太多 else if 表达式会使您的代码变得混乱，因此如果您有多个 else if 表达式，您可能需要重构您的代码。 第 6 章描述了一个强大的 Rust 分支结构，称为 match，适用于这些情况。

### Using if in a let Statement

Because if is an expression, we can use it on the right side of a let statement to assign the outcome to a variable, as in Listing 3-2.

因为 if 是一个表达式，所以我们可以在 let 语句的右侧使用它来将结果分配给变量，如清单 3-2 所示。

Filename: src/main.rs
```
fn main() {
    let condition = true;
    let number = if condition { 5 } else { 6 };

    println!("The value of number is: {number}");
}
```
Listing 3-2: Assigning the result of an if expression to a variable

The number variable will be bound to a value based on the outcome of the if expression. Run this code to see what happens:

```
$ cargo run
   Compiling branches v0.1.0 (file:///projects/branches)
    Finished dev [unoptimized + debuginfo] target(s) in 0.30s
     Running `target/debug/branches`
The value of number is: 5

```

Remember that blocks of code evaluate to the last expression in them, and numbers by themselves are also expressions. In this case, the value of the whole if expression depends on which block of code executes. This means the values that have the potential to be results from each arm of the if must be the same type; in Listing 3-2, the results of both the if arm and the else arm were i32 integers. If the types are mismatched, as in the following example, we’ll get an error:

请记住，代码块的计算结果是其中的最后一个表达式，数字本身也是表达式。 在这种情况下，整个 if 表达式的值取决于执行哪个代码块。 这意味着 if 的每个分支可能产生的值必须是相同的类型； 在清单 3-2 中，if 臂和 else 臂的结果都是 i32 整数。 如果类型不匹配，如下例所示，我们将收到错误：

Filename: src/main.rs

```
fn main() {
    let condition = true;

    let number = if condition { 5 } else { "six" };

    println!("The value of number is: {number}");
}
```

When we try to compile this code, we’ll get an error. The if and else arms have value types that are incompatible, and Rust indicates exactly where to find the problem in the program:

当我们尝试编译这段代码时，我们会得到一个错误。 if 和 else 分支具有不兼容的值类型，Rust 准确指出了在程序中查找问题的位置：

```
$ cargo run
   Compiling branches v0.1.0 (file:///projects/branches)
error[E0308]: `if` and `else` have incompatible types
 --> src/main.rs:4:44
  |
4 |     let number = if condition { 5 } else { "six" };
  |                                 -          ^^^^^ expected integer, found `&str`
  |                                 |
  |                                 expected because of this

For more information about this error, try `rustc --explain E0308`.
error: could not compile `branches` due to previous error

```
The expression in the if block evaluates to an integer, and the expression in the else block evaluates to a string. This won’t work because variables must have a single type, and Rust needs to know at compile time what type the number variable is, definitively. Knowing the type of number lets the compiler verify the type is valid everywhere we use number. Rust wouldn’t be able to do that if the type of number was only determined at runtime; the compiler would be more complex and would make fewer guarantees about the code if it had to keep track of multiple hypothetical types for any variable.

if 块中的表达式计算结果为整数，else 块中的表达式计算结果为字符串。 这是行不通的，因为变量必须具有单一类型，并且 Rust 需要在编译时明确知道数字变量是什么类型。 了解数字的类型可以让编译器验证该类型在我们使用数字的任何地方都有效。 如果数字类型仅在运行时确定，Rust 将无法做到这一点； 如果编译器必须跟踪任何变量的多个假设类型，编译器将会更加复杂，并且对代码的保证也会更少。

### Repetition with Loops
It’s often useful to execute a block of code more than once. For this task, Rust provides several loops, which will run through the code inside the loop body to the end and then start immediately back at the beginning. To experiment with loops, let’s make a new project called loops.

多次执行一段代码通常很有用。 对于这个任务，Rust 提供了几个循环，这些循环将运行循环体内的代码直到结束，然后立即从头开始。 为了试验循环，让我们创建一个名为 Loops 的新项目。

Rust has three kinds of loops: loop, while, and for. Let’s try each one.

#### Repeating Code with loop
The loop keyword tells Rust to execute a block of code over and over again forever or until you explicitly tell it to stop.

As an example, change the src/main.rs file in your loops directory to look like this:

Filename: src/main.rs
```
fn main() {
    loop {
        println!("again!");
    }
}
```
When we run this program, we’ll see again! printed over and over continuously until we stop the program manually. Most terminals support the keyboard shortcut ctrl-c to interrupt a program that is stuck in a continual loop. Give it a try:

```
$ cargo run
   Compiling loops v0.1.0 (file:///projects/loops)
    Finished dev [unoptimized + debuginfo] target(s) in 0.29s
     Running `target/debug/loops`
again!
again!
again!
again!
^Cagain!

```
The symbol ^C represents where you pressed ctrl-c. You may or may not see the word again! printed after the ^C, depending on where the code was in the loop when it received the interrupt signal.

Fortunately, Rust also provides a way to break out of a loop using code. You can place the break keyword within the loop to tell the program when to stop executing the loop. Recall that we did this in the guessing game in the “Quitting After a Correct Guess” section of Chapter 2 to exit the program when the user won the game by guessing the correct number.

We also used continue in the guessing game, which in a loop tells the program to skip over any remaining code in this iteration of the loop and go to the next iteration.

符号 ^C 代表您按下 ctrl-c 的位置。 您可能会也可能不会再次看到这个词！ 在 ^C 之后打印，具体取决于收到中断信号时代码在循环中的位置。

幸运的是，Rust 还提供了一种使用代码打破循环的方法。 您可以在循环中放置break关键字来告诉程序何时停止执行循环。 回想一下，我们在第 2 章“猜对后退出”部分的猜谜游戏中这样做了，当用户通过猜对数字赢得游戏时退出程序。

我们还在猜谜游戏中使用了 continue，它在循环中告诉程序跳过本次循环迭代中的任何剩余代码并进入下一次迭代。

#### Returning Values from Loops

One of the uses of a loop is to retry an operation you know might fail, such as checking whether a thread has completed its job. You might also need to pass the result of that operation out of the loop to the rest of your code. To do this, you can add the value you want returned after the break expression you use to stop the loop; that value will be returned out of the loop so you can use it, as shown here:

循环的用途之一是重试您知道可能会失败的操作，例如检查线程是否已完成其作业。 您可能还需要将该操作的结果从循环传递到代码的其余部分。 为此，您可以在用于停止循环的break表达式之后添加您想要返回的值； 该值将从循环中返回，以便您可以使用它，如下所示：

```
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;
        }
    };

    println!("The result is {result}");
}
```
Before the loop, we declare a variable named counter and initialize it to 0. Then we declare a variable named result to hold the value returned from the loop. On every iteration of the loop, we add 1 to the counter variable, and then check whether the counter is equal to 10. When it is, we use the break keyword with the value counter * 2. After the loop, we use a semicolon to end the statement that assigns the value to result. Finally, we print the value in result, which in this case is 20.

在循环之前，我们声明一个名为 counter 的变量并将其初始化为 0。然后我们声明一个名为 result 的变量来保存循环返回的值。 在循环的每次迭代中，我们将 counter 变量加 1，然后检查 counter 是否等于 10。如果等于，我们使用带有值 counter * 2 的break关键字。循环后，我们使用分号 结束将值赋给结果的语句。 最后，我们打印 result 中的值，在本例中为 20。

#### Loop Labels to Disambiguate Between Multiple Loops
If you have loops within loops, break and continue apply to the innermost loop at that point. You can optionally specify a loop label on a loop that you can then use with break or continue to specify that those keywords apply to the labeled loop instead of the innermost loop. Loop labels must begin with a single quote. Here’s an example with two nested loops:

如果循环内有循环，则 Break 并 continue 应用于该点的最内层循环。 您可以选择在循环上指定循环标签，然后将其与break 一起使用，或继续指定这些关键字应用于带标签的循环而不是最内层循环。 循环标签必须以单引号开头。 这是一个包含两个嵌套循环的示例：

```
fn main() {
    let mut count = 0;
    'counting_up: loop {
        println!("count = {count}");
        let mut remaining = 10;

        loop {
            println!("remaining = {remaining}");
            if remaining == 9 {
                break;
            }
            if count == 2 {
                break 'counting_up;
            }
            remaining -= 1;
        }

        count += 1;
    }
    println!("End count = {count}");
}
```
The outer loop has the label 'counting_up, and it will count up from 0 to 2. The inner loop without a label counts down from 10 to 9. The first break that doesn’t specify a label will exit the inner loop only. The break 'counting_up; statement will exit the outer loop. This code prints:

外层循环有标签“counting_up”，它将从 0 到 2 递增计数。没有标签的内层循环从 10 到 9 递减计数。第一个不指定标签的中断将仅退出内层循环。 中断'counting_up; 语句将退出外循环。 此代码打印：

```
$ cargo run
   Compiling loops v0.1.0 (file:///projects/loops)
    Finished dev [unoptimized + debuginfo] target(s) in 0.58s
     Running `target/debug/loops`
count = 0
remaining = 10
remaining = 9
count = 1
remaining = 10
remaining = 9
count = 2
remaining = 10
End count = 2

```

#### Conditional Loops with while

A program will often need to evaluate a condition within a loop. While the condition is true, the loop runs. When the condition ceases to be true, the program calls break, stopping the loop. It’s possible to implement behavior like this using a combination of loop, if, else, and break; you could try that now in a program, if you’d like. However, this pattern is so common that Rust has a built-in language construct for it, called a while loop. In Listing 3-3, we use while to loop the program three times, counting down each time, and then, after the loop, print a message and exit.

程序通常需要评估循环内的条件。 当条件为真时，循环运行。 当条件不再为真时，程序调用break，停止循环。 可以使用loop、if、else 和break 的组合来实现这样的行为； 如果您愿意，您现在可以在程序中尝试一下。 然而，这种模式非常常见，以至于 Rust 有一个内置的语言构造，称为 while 循环。 在清单 3-3 中，我们使用 while 循环程序三次，每次倒计时，然后在循环结束后打印一条消息并退出。

Filename: src/main.rs

```
fn main() {
    let mut number = 3;

    while number != 0 {
        println!("{number}!");

        number -= 1;
    }

    println!("LIFTOFF!!!");
}
```
Listing 3-3: Using a while loop to run code while a condition holds true

This construct eliminates a lot of nesting that would be necessary if you used loop, if, else, and break, and it’s clearer. While a condition evaluates to true, the code runs; otherwise, it exits the loop.

这种结构消除了使用循环、if、else 和 break 时所必需的大量嵌套，而且更加清晰。 当条件评估为真时，代码运行； 否则，它退出循环。

####  Looping Through a Collection with for
You can choose to use the while construct to loop over the elements of a collection, such as an array. For example, the loop in Listing 3-4 prints each element in the array a.

您可以选择使用 while 构造来循环集合（例如数组）的元素。 例如，清单 3-4 中的循环打印数组 a 中的每个元素。

Filename: src/main.rs

```
fn main() {
    let a = [10, 20, 30, 40, 50];
    let mut index = 0;

    while index < 5 {
        println!("the value is: {}", a[index]);

        index += 1;
    }
}
```
Listing 3-4: Looping through each element of a collection using a while loop

Here, the code counts up through the elements in the array. It starts at index 0, and then loops until it reaches the final index in the array (that is, when index < 5 is no longer true). Running this code will print every element in the array:

此处，代码对数组中的元素进行向上计数。 它从索引 0 开始，然后循环直到到达数组中的最终索引（即，当索引 < 5 不再为 true 时）。 运行此代码将打印数组中的每个元素：

```
$ cargo run
   Compiling loops v0.1.0 (file:///projects/loops)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32s
     Running `target/debug/loops`
the value is: 10
the value is: 20
the value is: 30
the value is: 40
the value is: 50

```
All five array values appear in the terminal, as expected. Even though index will reach a value of 5 at some point, the loop stops executing before trying to fetch a sixth value from the array.

However, this approach is error prone; we could cause the program to panic if the index value or test condition is incorrect. For example, if you changed the definition of the a array to have four elements but forgot to update the condition to while index < 4, the code would panic. It’s also slow, because the compiler adds runtime code to perform the conditional check of whether the index is within the bounds of the array on every iteration through the loop.

As a more concise alternative, you can use a for loop and execute some code for each item in a collection. A for loop looks like the code in Listing 3-5.

正如预期的那样，所有五个数组值都出现在终端中。 即使索引在某个时刻将达到值 5，循环也会在尝试从数组中获取第六个值之前停止执行。

然而，这种方法很容易出错； 如果索引值或测试条件不正确，我们可能会导致程序出现恐慌。 例如，如果您将 a 数组的定义更改为包含四个元素，但忘记将条件更新为 while index < 4，则代码将会出现混乱。 它也很慢，因为编译器添加了运行时代码来在循环的每次迭代中执行索引是否在数组范围内的条件检查。

作为更简洁的替代方案，您可以使用 for 循环并为集合中的每个项目执行一些代码。 for 循环类似于清单 3-5 中的代码。

Filename: src/main.rs

```
fn main() {
    let a = [10, 20, 30, 40, 50];

    for element in a {
        println!("the value is: {element}");
    }
}
```
Listing 3-5: Looping through each element of a collection using a for loop

When we run this code, we’ll see the same output as in Listing 3-4. More importantly, we’ve now increased the safety of the code and eliminated the chance of bugs that might result from going beyond the end of the array or not going far enough and missing some items.

Using the for loop, you wouldn’t need to remember to change any other code if you changed the number of values in the array, as you would with the method used in Listing 3-4.

The safety and conciseness of for loops make them the most commonly used loop construct in Rust. Even in situations in which you want to run some code a certain number of times, as in the countdown example that used a while loop in Listing 3-3, most Rustaceans would use a for loop. The way to do that would be to use a Range, provided by the standard library, which generates all numbers in sequence starting from one number and ending before another number.

Here’s what the countdown would look like using a for loop and another method we’ve not yet talked about, rev, to reverse the range:

当我们运行这段代码时，我们将看到与清单 3-4 相同的输出。 更重要的是，我们现在提高了代码的安全性，并消除了由于超出数组末尾或不够远而丢失某些项目而可能导致错误的可能性。

使用 for 循环，如果您更改了数组中的值的数量，则无需记住更改任何其他代码，就像使用清单 3-4 中使用的方法一样。

for 循环的安全性和简洁性使其成为 Rust 中最常用的循环结构。 即使在您想要运行某些代码一定次数的情况下（如清单 3-3 中使用 while 循环的倒计时示例），大多数 Rustaceans 也会使用 for 循环。 实现这一点的方法是使用标准库提供的 Range，它按顺序生成从一个数字开始到另一个数字之前结束的所有数字。

这是使用 for 循环和我们尚未讨论的另一种方法 rev 来反转范围的倒计时：

Filename: src/main.rs

```
fn main() {
    for number in (1..4).rev() {
        println!("{number}!");
    }
    println!("LIFTOFF!!!");
}
```
This code is a bit nicer, isn’t it?

## 3.6 总结
You made it! This was a sizable chapter: you learned about variables, scalar and compound data types, functions, comments, if expressions, and loops! To practice with the concepts discussed in this chapter, try building programs to do the following:

+ Convert temperatures between Fahrenheit and Celsius.
+ Generate the nth Fibonacci number.
+ Print the lyrics to the Christmas carol “The Twelve Days of Christmas,” taking advantage of the repetition in the song.

你做到了！ 这是一个相当大的章节：您了解了变量、标量和复合数据类型、函数、注释、if 表达式和循环！ 要练习本章讨论的概念，请尝试构建程序来执行以下操作：

+ 在华氏度和摄氏度之间转换温度。
+ 生成第 n 个斐波那契数。
+ 利用歌曲中的重复内容打印圣诞颂歌“圣诞节的十二天”的歌词。

When you’re ready to move on, we’ll talk about a concept in Rust that doesn’t commonly exist in other programming languages: ownership.

当您准备好继续前进时，我们将讨论 Rust 中的一个在其他编程语言中不常见的概念：所有权。