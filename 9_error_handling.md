# 9. 错误处理
Errors are a fact of life in software, so Rust has a number of features for handling situations in which something goes wrong. In many cases, Rust requires you to acknowledge the possibility of an error and take some action before your code will compile. This requirement makes your program more robust by ensuring that you’ll discover errors and handle them appropriately before you’ve deployed your code to production!

错误是软件中不可避免的事实，因此 Rust 有许多功能可以处理出现问题的情况。 在许多情况下，Rust 要求您承认错误的可能性并在代码编译之前采取一些操作。 此要求可确保您在将代码部署到生产环境之前发现错误并适当处理它们，从而使您的程序更加健壮！

Rust groups errors into two major categories: recoverable and unrecoverable errors. For a recoverable error, such as a file not found error, we most likely just want to report the problem to the user and retry the operation. Unrecoverable errors are always symptoms of bugs, like trying to access a location beyond the end of an array, and so we want to immediately stop the program.

Rust 将错误分为两大类：可恢复错误和不可恢复错误。 对于可恢复的错误，例如找不到文件错误，我们很可能只想向用户报告问题并重试操作。 不可恢复的错误始终是错误的症状，例如尝试访问超出数组末尾的位置，因此我们希望立即停止程序。

Most languages don’t distinguish between these two kinds of errors and handle both in the same way, using mechanisms such as exceptions. Rust doesn’t have exceptions. Instead, it has the type Result<T, E> for recoverable errors and the panic! macro that stops execution when the program encounters an unrecoverable error. This chapter covers calling panic! first and then talks about returning Result<T, E> values. Additionally, we’ll explore considerations when deciding whether to try to recover from an error or to stop execution.

大多数语言不区分这两种错误，并使用异常等机制以相同的方式处理这两种错误。 Rust 也不例外。 相反，它具有 Result<T, E> 类型来表示可恢复的错误和恐慌！ 当程序遇到不可恢复的错误时停止执行的宏。 本章介绍了调用恐慌！ 首先讨论返回 Result<T, E> 值。 此外，我们将探讨在决定是尝试从错误中恢复还是停止执行时的注意事项。

## 9.1 使用panic!处理不可恢复错误(Unrecoverable Errors with panic!)
Sometimes, bad things happen in your code, and there’s nothing you can do about it. In these cases, Rust has the panic! macro. There are two ways to cause a panic in practice: by taking an action that causes our code to panic (such as accessing an array past the end) or by explicitly calling the panic! macro. In both cases, we cause a panic in our program. By default, these panics will print a failure message, unwind, clean up the stack, and quit. Via an environment variable, you can also have Rust display the call stack when a panic occurs to make it easier to track down the source of the panic.

有时，您的代码中会发生不好的事情，而您对此无能为力。 在这些情况下，Rust 就会恐慌！ 宏。 在实践中，有两种方法会导致恐慌：采取导致代码恐慌的操作（例如访问超出末尾的数组）或显式调用恐慌！ 宏。 在这两种情况下，我们都会在程序中引起恐慌。 默认情况下，这些恐慌将打印一条失败消息、展开、清理堆栈并退出。 通过环境变量，您还可以让 Rust 在发生恐慌时显示调用堆栈，以便更轻松地追踪恐慌的来源。

Let’s try calling panic! in a simple program:
```
Filename: src/main.rs

fn main() {
    panic!("crash and burn");
}
```
When you run the program, you’ll see something like this:

```
$ cargo run
   Compiling panic v0.1.0 (file:///projects/panic)
    Finished dev [unoptimized + debuginfo] target(s) in 0.25s
     Running `target/debug/panic`
thread 'main' panicked at 'crash and burn', src/main.rs:2:5
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```
The call to panic! causes the error message contained in the last two lines. The first line shows our panic message and the place in our source code where the panic occurred: src/main.rs:2:5 indicates that it’s the second line, fifth character of our src/main.rs file.

呼吁恐慌！ 导致最后两行中包含错误消息。 第一行显示我们的恐慌消息以及源代码中发生恐慌的位置： src/main.rs:2:5 表示它是 src/main.rs 文件的第二行，第五个字符。

In this case, the line indicated is part of our code, and if we go to that line, we see the panic! macro call. In other cases, the panic! call might be in code that our code calls, and the filename and line number reported by the error message will be someone else’s code where the panic! macro is called, not the line of our code that eventually led to the panic! call. We can use the backtrace of the functions the panic! call came from to figure out the part of our code that is causing the problem. We’ll discuss backtraces in more detail next.

在这种情况下，指示的行是我们代码的一部分，如果我们转到该行，我们会看到恐慌！ 宏调用。 在其他情况下，恐慌！ call 可能在我们的代码调用的代码中，并且错误消息报告的文件名和行号将是别人的代码，其中恐慌！ 调用了宏，而不是最终导致恐慌的代码行！ 称呼。 我们可以使用函数的回溯来恐慌！ 调用是为了找出导致问题的代码部分。 接下来我们将更详细地讨论回溯。

### Using a panic! Backtrace
Let’s look at another example to see what it’s like when a panic! call comes from a library because of a bug in our code instead of from our code calling the macro directly. Listing 9-1 has some code that attempts to access an index in a vector beyond the range of valid indexes.

让我们看另一个例子，看看发生恐慌时是什么样子！ 由于代码中的错误，调用来自库，而不是直接调用宏的代码。 清单 9-1 有一些代码尝试访问向量中超出有效索引范围的索引。

Filename: src/main.rs
```
This code panics!
fn main() {
    let v = vec![1, 2, 3];

    v[99];
}
```
Listing 9-1: Attempting to access an element beyond the end of a vector, which will cause a call to panic!

Here, we’re attempting to access the 100th element of our vector (which is at index 99 because indexing starts at zero), but the vector has only 3 elements. In this situation, Rust will panic. Using [] is supposed to return an element, but if you pass an invalid index, there’s no element that Rust could return here that would be correct.

在这里，我们尝试访问向量的第 100 个元素（位于索引 99，因为索引从零开始），但向量只有 3 个元素。 在这种情况下，Rust 会恐慌。 使用 [] 应该返回一个元素，但是如果你传递一个无效的索引，Rust 就不会在这里返回正确的元素。

In C, attempting to read beyond the end of a data structure is undefined behavior. You might get whatever is at the location in memory that would correspond to that element in the data structure, even though the memory doesn’t belong to that structure. This is called a buffer overread and can lead to security vulnerabilities if an attacker is able to manipulate the index in such a way as to read data they shouldn’t be allowed to that is stored after the data structure.

在 C 中，尝试读取超出数据结构末尾的内容是未定义的行为。 您可能会得到内存中与数据结构中该元素相对应的位置的任何内容，即使内存不属于该结构。 这称为缓冲区过度读取，如果攻击者能够以某种方式操纵索引以读取他们不应该允许的存储在数据结构之后的数据，则可能会导致安全漏洞。

To protect your program from this sort of vulnerability, if you try to read an element at an index that doesn’t exist, Rust will stop execution and refuse to continue. Let’s try it and see:

为了保护您的程序免受此类漏洞的影响，如果您尝试读取不存在的索引处的元素，Rust 将停止执行并拒绝继续。 让我们尝试一下看看：

```
$ cargo run
   Compiling panic v0.1.0 (file:///projects/panic)
    Finished dev [unoptimized + debuginfo] target(s) in 0.27s
     Running `target/debug/panic`
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', src/main.rs:4:5
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```
This error points at line 4 of our main.rs where we attempt to access index 99. The next note line tells us that we can set the RUST_BACKTRACE environment variable to get a backtrace of exactly what happened to cause the error. A backtrace is a list of all the functions that have been called to get to this point. Backtraces in Rust work as they do in other languages: the key to reading the backtrace is to start from the top and read until you see files you wrote. That’s the spot where the problem originated. The lines above that spot are code that your code has called; the lines below are code that called your code. These before-and-after lines might include core Rust code, standard library code, or crates that you’re using. Let’s try getting a backtrace by setting the RUST_BACKTRACE environment variable to any value except 0. Listing 9-2 shows output similar to what you’ll see.

此错误指向 main.rs 的第 4 行，我们在其中尝试访问索引 99。下一条注释行告诉我们可以设置 RUST_BACKTRACE 环境变量来获取导致错误的确切原因的回溯。 回溯是为达到这一点而调用的所有函数的列表。 Rust 中的回溯就像在其他语言中一样工作：读取回溯的关键是从顶部开始读取，直到看到您编写的文件。 这就是问题的根源。 该点上方的行是您的代码调用的代码； 下面的行是调用您的代码的代码。 这些前后行可能包括核心 Rust 代码、标准库代码或您正在使用的 crate。 让我们尝试通过将 RUST_BACKTRACE 环境变量设置为除 0 之外的任何值来获取回溯。清单 9-2 显示的输出与您将看到的类似。


```
$ RUST_BACKTRACE=1 cargo run
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', src/main.rs:4:5
stack backtrace:
   0: rust_begin_unwind
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/std/src/panicking.rs:584:5
   1: core::panicking::panic_fmt
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/core/src/panicking.rs:142:14
   2: core::panicking::panic_bounds_check
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/core/src/panicking.rs:84:5
   3: <usize as core::slice::index::SliceIndex<[T]>>::index
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/core/src/slice/index.rs:242:10
   4: core::slice::index::<impl core::ops::index::Index<I> for [T]>::index
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/core/src/slice/index.rs:18:9
   5: <alloc::vec::Vec<T,A> as core::ops::index::Index<I>>::index
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/alloc/src/vec/mod.rs:2591:9
   6: panic::main
             at ./src/main.rs:4:5
   7: core::ops::function::FnOnce::call_once
             at /rustc/e092d0b6b43f2de967af0887873151bb1c0b18d3/library/core/src/ops/function.rs:248:5
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
```
Listing 9-2: The backtrace generated by a call to panic! displayed when the environment variable RUST_BACKTRACE is set

That’s a lot of output! The exact output you see might be different depending on your operating system and Rust version. In order to get backtraces with this information, debug symbols must be enabled. Debug symbols are enabled by default when using cargo build or cargo run without the --release flag, as we have here.

这输出量真大啊！ 您看到的确切输出可能会有所不同，具体取决于您的操作系统和 Rust 版本。 为了获得包含此信息的回溯，必须启用调试符号。 当使用没有 --release 标志的 Cargo build 或 Cargo Run 时，默认启用调试符号，就像我们在这里一样。

In the output in Listing 9-2, line 6 of the backtrace points to the line in our project that’s causing the problem: line 4 of src/main.rs. If we don’t want our program to panic, we should start our investigation at the location pointed to by the first line mentioning a file we wrote. In Listing 9-1, where we deliberately wrote code that would panic, the way to fix the panic is to not request an element beyond the range of the vector indexes. When your code panics in the future, you’ll need to figure out what action the code is taking with what values to cause the panic and what the code should do instead.

在清单 9-2 的输出中，回溯的第 6 行指向我们项目中导致问题的行：src/main.rs 的第 4 行。 如果我们不希望我们的程序出现恐慌，我们应该从第一行提到我们编写的文件所指向的位置开始我们的调查。 在清单 9-1 中，我们故意编写了会发生恐慌的代码，解决恐慌的方法是不请求超出向量索引范围的元素。 当您的代码将来发生恐慌时，您需要弄清楚代码正在采取什么操作以及哪些值会导致恐慌，以及代码应该做什么。

We’ll come back to panic! and when we should and should not use panic! to handle error conditions in the “To panic! or Not to panic!” section later in this chapter. Next, we’ll look at how to recover from an error using Result.

我们会再次陷入恐慌！ 以及什么时候我们应该和不应该使用恐慌！ 处理“惊慌！ 或者不要惊慌！” 本章稍后部分。 接下来，我们将了解如何使用 Result 从错误中恢复。

## 9.2 使用Result处理可恢复错误(Recoverable Errors with Result)

Most errors aren’t serious enough to require the program to stop entirely. Sometimes, when a function fails, it’s for a reason that you can easily interpret and respond to. For example, if you try to open a file and that operation fails because the file doesn’t exist, you might want to create the file instead of terminating the process.

大多数错误并没有严重到需要程序完全停止的程度。 有时，当某个功能失败时，其原因是您可以轻松解释和响应的。 例如，如果您尝试打开一个文件，但由于该文件不存在而导致该操作失败，您可能需要创建该文件而不是终止该进程。

Recall from “Handling Potential Failure with Result” in Chapter 2 that the Result enum is defined as having two variants, Ok and Err, as follows:

回想一下第 2 章中的“处理潜在的结果失败”，Result 枚举被定义为有两个变体，Ok 和 Err，如下所示：

```
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```
The T and E are generic type parameters: we’ll discuss generics in more detail in Chapter 10. What you need to know right now is that T represents the type of the value that will be returned in a success case within the Ok variant, and E represents the type of the error that will be returned in a failure case within the Err variant. Because Result has these generic type parameters, we can use the Result type and the functions defined on it in many different situations where the successful value and error value we want to return may differ.

T 和 E 是泛型类型参数：我们将在第 10 章中更详细地讨论泛型。您现在需要知道的是 T 表示 Ok 变体中成功案例中将返回的值的类型， E 表示 Err 变体中失败情况下将返回的错误类型。 由于 Result 具有这些通用类型参数，因此我们可以在许多不同的情况下使用 Result 类型及其定义的函数，在这些情况下，我们想要返回的成功值和错误值可能不同。

Let’s call a function that returns a Result value because the function could fail. In Listing 9-3 we try to open a file.

Filename: src/main.rs
```
use std::fs::File;

fn main() {
    let greeting_file_result = File::open("hello.txt");
}
```
Listing 9-3: Opening a file

The return type of File::open is a Result<T, E>. The generic parameter T has been filled in by the implementation of File::open with the type of the success value, std::fs::File, which is a file handle. The type of E used in the error value is std::io::Error. This return type means the call to File::open might succeed and return a file handle that we can read from or write to. The function call also might fail: for example, the file might not exist, or we might not have permission to access the file. The File::open function needs to have a way to tell us whether it succeeded or failed and at the same time give us either the file handle or error information. This information is exactly what the Result enum conveys.

File::open 的返回类型是 Result<T, E>。 通用参数 T 已由 File::open 的实现填充为成功值 std::fs::File 的类型，它是一个文件句柄。 错误值中使用的 E 类型是 std::io::Error。 此返回类型意味着对 File::open 的调用可能会成功并返回一个我们可以读取或写入的文件句柄。 函数调用也可能失败：例如，文件可能不存在，或者我们可能没有访问该文件的权限。 File::open 函数需要有一种方法来告诉我们它是成功还是失败，同时为我们提供文件句柄或错误信息。 此信息正是 Result 枚举所传达的信息。

In the case where File::open succeeds, the value in the variable greeting_file_result will be an instance of Ok that contains a file handle. In the case where it fails, the value in greeting_file_result will be an instance of Err that contains more information about the kind of error that happened.

如果 File::open 成功，变量greeting_file_result 中的值将是包含文件句柄的 Ok 实例。 如果失败，greeting_file_result 中的值将是 Err 的实例，其中包含有关所发生错误类型的更多信息。

We need to add to the code in Listing 9-3 to take different actions depending on the value File::open returns. Listing 9-4 shows one way to handle the Result using a basic tool, the match expression that we discussed in Chapter 6.

我们需要添加到清单 9-3 中的代码，以根据 File::open 返回的值采取不同的操作。 清单 9-4 显示了使用基本工具（我们在第 6 章中讨论的匹配表达式）处理结果的一种方法。

Filename: src/main.rs
```
use std::fs::File;

fn main() {
    let greeting_file_result = File::open("hello.txt");

    let greeting_file = match greeting_file_result {
        Ok(file) => file,
        Err(error) => panic!("Problem opening the file: {:?}", error),
    };
}
```
Listing 9-4: Using a match expression to handle the Result variants that might be returned

Note that, like the Option enum, the Result enum and its variants have been brought into scope by the prelude, so we don’t need to specify Result:: before the Ok and Err variants in the match arms.

请注意，与 Option 枚举一样，Result 枚举及其变体已被 prelude 纳入范围，因此我们不需要在匹配臂中的 Ok 和 Err 变体之前指定 Result:: 。

When the result is Ok, this code will return the inner file value out of the Ok variant, and we then assign that file handle value to the variable greeting_file. After the match, we can use the file handle for reading or writing.

当结果为 Ok 时，此代码将从 Ok 变量中返回内部文件值，然后我们将该文件句柄值分配给变量greeting_file。 匹配后，我们可以使用文件句柄进行读取或写入。

The other arm of the match handles the case where we get an Err value from File::open. In this example, we’ve chosen to call the panic! macro. If there’s no file named hello.txt in our current directory and we run this code, we’ll see the following output from the panic! macro:

匹配的另一部分处理我们从 File::open 获取 Err 值的情况。 在这个例子中，我们选择调用恐慌！ 宏。 如果当前目录中没有名为 hello.txt 的文件，并且运行此代码，我们将看到恐慌的以下输出！ 宏：
```
$ cargo run
   Compiling error-handling v0.1.0 (file:///projects/error-handling)
    Finished dev [unoptimized + debuginfo] target(s) in 0.73s
     Running `target/debug/error-handling`
thread 'main' panicked at 'Problem opening the file: Os { code: 2, kind: NotFound, message: "No such file or directory" }', src/main.rs:8:23
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```
As usual, this output tells us exactly what has gone wrong.

### Matching on Different Errors
The code in Listing 9-4 will panic! no matter why File::open failed. However, we want to take different actions for different failure reasons: if File::open failed because the file doesn’t exist, we want to create the file and return the handle to the new file. If File::open failed for any other reason—for example, because we didn’t have permission to open the file—we still want the code to panic! in the same way as it did in Listing 9-4. For this we add an inner match expression, shown in Listing 9-5.

清单 9-4 中的代码将会出现混乱！ 无论 File::open 为何失败。 但是，我们希望针对不同的失败原因采取不同的操作：如果 File::open 由于文件不存在而失败，我们希望创建该文件并将句柄返回到新文件。 如果 File::open 由于任何其他原因失败（例如，因为我们没有打开文件的权限），我们仍然希望代码出现恐慌！ 与清单 9-4 中的方法相同。 为此，我们添加一个内部匹配表达式，如清单 9-5 所示。

Filename: src/main.rs
```
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let greeting_file_result = File::open("hello.txt");

    let greeting_file = match greeting_file_result {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {:?}", e),
            },
            other_error => {
                panic!("Problem opening the file: {:?}", other_error);
            }
        },
    };
}
```
Listing 9-5: Handling different kinds of errors in different ways

The type of the value that File::open returns inside the Err variant is io::Error, which is a struct provided by the standard library. This struct has a method kind that we can call to get an io::ErrorKind value. The enum io::ErrorKind is provided by the standard library and has variants representing the different kinds of errors that might result from an io operation. The variant we want to use is ErrorKind::NotFound, which indicates the file we’re trying to open doesn’t exist yet. So we match on greeting_file_result, but we also have an inner match on error.kind().

File::open 在 Err 变体中返回的值的类型是 io::Error，它是标准库提供的一个结构体。 这个结构体有一个方法 kind，我们可以调用它来获取 io::ErrorKind 值。 枚举 io::ErrorKind 由标准库提供，并具有表示 io 操作可能导致的不同类型错误的变体。 我们要使用的变体是 ErrorKind::NotFound，它表示我们尝试打开的文件尚不存在。 所以我们在greeting_file_result上进行匹配，但我们也在error.kind()上进行了内部匹配。

The condition we want to check in the inner match is whether the value returned by error.kind() is the NotFound variant of the ErrorKind enum. If it is, we try to create the file with File::create. However, because File::create could also fail, we need a second arm in the inner match expression. When the file can’t be created, a different error message is printed. The second arm of the outer match stays the same, so the program panics on any error besides the missing file error.

我们要在内部匹配中检查的条件是 error.kind() 返回的值是否是 ErrorKind 枚举的 NotFound 变体。 如果是，我们尝试使用 File::create 创建该文件。 但是，由于 File::create 也可能失败，因此我们需要在内部匹配表达式中添加第二个分支。 当无法创建文件时，会打印不同的错误消息。 外部匹配的第二个分支保持不变，因此除了丢失文件错误之外，程序会因任何错误而发生恐慌。

### 使用 match 与 Result<T, E> 的替代方法(Alternatives to Using match with Result<T, E>)
That’s a lot of match! The match expression is very useful but also very much a primitive. In Chapter 13, you’ll learn about closures, which are used with many of the methods defined on Result<T, E>. These methods can be more concise than using match when handling Result<T, E> values in your code.

这是很多match！ match表达式非常有用，但也非常原始。 在第 13 章中，您将学习闭包，它与 Result<T, E> 上定义的许多方法一起使用。 在处理代码中的 Result<T, E> 值时，这些方法比使用 match 更简洁。

For example, here’s another way to write the same logic as shown in Listing 9-5, this time using closures and the unwrap_or_else method:

例如，这是编写与清单 9-5 所示相同逻辑的另一种方法，这次使用闭包和 unwrap_or_else 方法：

```
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let greeting_file = File::open("hello.txt").unwrap_or_else(|error| {
        if error.kind() == ErrorKind::NotFound {
            File::create("hello.txt").unwrap_or_else(|error| {
                panic!("Problem creating the file: {:?}", error);
            })
        } else {
            panic!("Problem opening the file: {:?}", error);
        }
    });
}
```
Although this code has the same behavior as Listing 9-5, it doesn’t contain any match expressions and is cleaner to read. Come back to this example after you’ve read Chapter 13, and look up the unwrap_or_else method in the standard library documentation. Many more of these methods can clean up huge nested match expressions when you’re dealing with errors.

尽管此代码与清单 9-5 具有相同的行为，但它不包含任何匹配表达式，并且读起来更清晰。 读完第 13 章后，回到这个示例，并在标准库文档中查找 unwrap_or_else 方法。 当您处理错误时，更多这些方法可以清理巨大的嵌套匹配表达式。

### 快捷方式unwrap和expect(Shortcuts for Panic on Error: unwrap and expect)
Using match works well enough, but it can be a bit verbose and doesn’t always communicate intent well. The Result<T, E> type has many helper methods defined on it to do various, more specific tasks. The unwrap method is a shortcut method implemented just like the match expression we wrote in Listing 9-4. If the Result value is the Ok variant, unwrap will return the value inside the Ok. If the Result is the Err variant, unwrap will call the panic! macro for us. Here is an example of unwrap in action:

使用 match 效果很好，但它可能有点冗长，并且并不总是能很好地传达意图。 Result<T, E> 类型定义了许多辅助方法来执行各种更具体的任务。 unwrap 方法是一个快捷方法，其实现就像我们在清单 9-4 中编写的匹配表达式一样。 如果 Result 值是 Ok 变体，则 unwrap 将返回 Ok 内的值。 如果结果是 Err 变体，unwrap 将引发恐慌！ 对我们来说宏。 以下是展开操作的示例：

Filename: src/main.rs
```
use std::fs::File;

fn main() {
    let greeting_file = File::open("hello.txt").unwrap();
}
```
If we run this code without a hello.txt file, we’ll see an error message from the panic! call that the unwrap method makes:
```
thread 'main' panicked at 'called `Result::unwrap()` on an `Err` value: Os {
code: 2, kind: NotFound, message: "No such file or directory" }',
src/main.rs:4:49
```
Similarly, the expect method lets us also choose the panic! error message. Using expect instead of unwrap and providing good error messages can convey your intent and make tracking down the source of a panic easier. The syntax of expect looks like this:

Filename: src/main.rs
```
use std::fs::File;

fn main() {
    let greeting_file = File::open("hello.txt")
        .expect("hello.txt should be included in this project");
}
```
We use expect in the same way as unwrap: to return the file handle or call the panic! macro. The error message used by expect in its call to panic! will be the parameter that we pass to expect, rather than the default panic! message that unwrap uses. Here’s what it looks like:
```
thread 'main' panicked at 'hello.txt should be included in this project: Os {
code: 2, kind: NotFound, message: "No such file or directory" }',
src/main.rs:5:10
```
In production-quality code, most Rustaceans choose expect rather than unwrap and give more context about why the operation is expected to always succeed. That way, if your assumptions are ever proven wrong, you have more information to use in debugging.

### 传播错误（Propagating Errors）
When a function’s implementation calls something that might fail, instead of handling the error within the function itself, you can return the error to the calling code so that it can decide what to do. This is known as propagating the error and gives more control to the calling code, where there might be more information or logic that dictates how the error should be handled than what you have available in the context of your code.

当函数的实现调用可能失败的内容时，您可以将错误返回给调用代码，以便它决定要做什么，而不是在函数本身内处理错误。 这称为传播错误，并为调用代码提供更多控制，其中可能有比代码上下文中可用的信息或逻辑更多的信息或逻辑来指示应如何处理错误。

For example, Listing 9-6 shows a function that reads a username from a file. If the file doesn’t exist or can’t be read, this function will return those errors to the code that called the function.

例如，清单 9-6 显示了一个从文件中读取用户名的函数。 如果文件不存在或无法读取，该函数会将这些错误返回给调用该函数的代码。

Filename: src/main.rs
```
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let username_file_result = File::open("hello.txt");

    let mut username_file = match username_file_result {
        Ok(file) => file,
        Err(e) => return Err(e),
    };

    let mut username = String::new();

    match username_file.read_to_string(&mut username) {
        Ok(_) => Ok(username),
        Err(e) => Err(e),
    }
}
```
Listing 9-6: A function that returns errors to the calling code using match

This function can be written in a much shorter way, but we’re going to start by doing a lot of it manually in order to explore error handling; at the end, we’ll show the shorter way. Let’s look at the return type of the function first: Result<String, io::Error>. This means the function is returning a value of the type Result<T, E> where the generic parameter T has been filled in with the concrete type String, and the generic type E has been filled in with the concrete type io::Error.

这个函数可以用更短的方式编写，但我们将首先手动执行大量操作，以探索错误处理； 最后，我们将展示更短的方法。 我们先看一下函数的返回类型：Result<String, io::Error>。 这意味着该函数返回 Result<T, E> 类型的值，其中通用参数 T 已使用具体类型 String 填充，通用类型 E 已使用具体类型 io::Error 填充。

If this function succeeds without any problems, the code that calls this function will receive an Ok value that holds a String—the username that this function read from the file. If this function encounters any problems, the calling code will receive an Err value that holds an instance of io::Error that contains more information about what the problems were. We chose io::Error as the return type of this function because that happens to be the type of the error value returned from both of the operations we’re calling in this function’s body that might fail: the File::open function and the read_to_string method.

如果此函数成功且没有任何问题，则调用此函数的代码将收到一个 Ok 值，其中包含一个字符串，即此函数从文件中读取的用户名。 如果此函数遇到任何问题，调用代码将收到一个 Err 值，该值包含 io::Error 的实例，其中包含有关问题所在的更多信息。 我们选择 io::Error 作为该函数的返回类型，因为它恰好是我们在此函数体内调用的两个可能失败的操作返回的错误值的类型：File::open 函数和 read_to_string 方法。

The body of the function starts by calling the File::open function. Then we handle the Result value with a match similar to the match in Listing 9-4. If File::open succeeds, the file handle in the pattern variable file becomes the value in the mutable variable username_file and the function continues. In the Err case, instead of calling panic!, we use the return keyword to return early out of the function entirely and pass the error value from File::open, now in the pattern variable e, back to the calling code as this function’s error value.

函数体首先调用 File::open 函数。 然后我们使用类似于清单 9-4 中的匹配来处理 Result 值。 如果 File::open 成功，模式变量 file 中的文件句柄将成为可变变量 username_file 中的值，并且函数继续。 在 Err 情况下，我们不调用panic!，而是使用 return 关键字完全从函数中提前返回，并将错误值从 File::open（现在位于模式变量 e 中）传递回调用代码，作为该函数的 误差值。

So if we have a file handle in username_file, the function then creates a new String in variable username and calls the read_to_string method on the file handle in username_file to read the contents of the file into username. The read_to_string method also returns a Result because it might fail, even though File::open succeeded. So we need another match to handle that Result: if read_to_string succeeds, then our function has succeeded, and we return the username from the file that’s now in username wrapped in an Ok. If read_to_string fails, we return the error value in the same way that we returned the error value in the match that handled the return value of File::open. However, we don’t need to explicitly say return, because this is the last expression in the function.

The code that calls this code will then handle getting either an Ok value that contains a username or an Err value that contains an io::Error. It’s up to the calling code to decide what to do with those values. If the calling code gets an Err value, it could call panic! and crash the program, use a default username, or look up the username from somewhere other than a file, for example. We don’t have enough information on what the calling code is actually trying to do, so we propagate all the success or error information upward for it to handle appropriately.

This pattern of propagating errors is so common in Rust that Rust provides the question mark operator ? to make this easier.

因此，如果我们在 username_file 中有一个文件句柄，则该函数会在变量 username 中创建一个新字符串，并在 username_file 中的文件句柄上调用 read_to_string 方法，以将文件内容读取到 username 中。 read_to_string 方法还会返回 Result，因为即使 File::open 成功，它也可能会失败。 因此，我们需要另一个匹配来处理该结果：如果 read_to_string 成功，那么我们的函数就成功了，我们从文件中返回用户名，该文件现在位于用 Ok 包裹的 username 中。 如果 read_to_string 失败，我们将按照处理 File::open 返回值的匹配中返回错误值的方式返回错误值。 但是，我们不需要显式地说 return，因为这是函数中的最后一个表达式。

然后，调用此代码的代码将处理获取包含用户名的 Ok 值或包含 io::Error 的 Err 值。 由调用代码决定如何处理这些值。 如果调用代码获得 Err 值，它可能会调用恐慌！ 例如，使程序崩溃，使用默认用户名，或者从文件以外的其他地方查找用户名。 我们没有足够的信息来了解调用代码实际尝试执行的操作，因此我们向上传播所有成功或错误信息，以便其正确处理。

这种传播错误的模式在 Rust 中非常常见，以至于 Rust 提供了问号运算符 ? 让这变得更容易。

### A Shortcut for Propagating Errors: the ? Operator
Listing 9-7 shows an implementation of read_username_from_file that has the same functionality as in Listing 9-6, but this implementation uses the ? operator.

清单 9-7 显示了 read_username_from_file 的实现，其功能与清单 9-6 相同，但该实现使用 ? 操作员。

Filename: src/main.rs
```
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let mut username_file = File::open("hello.txt")?;
    let mut username = String::new();
    username_file.read_to_string(&mut username)?;
    Ok(username)
}
```
Listing 9-7: A function that returns errors to the calling code using the ? operator

The ? placed after a Result value is defined to work in almost the same way as the match expressions we defined to handle the Result values in Listing 9-6. If the value of the Result is an Ok, the value inside the Ok will get returned from this expression, and the program will continue. If the value is an Err, the Err will be returned from the whole function as if we had used the return keyword so the error value gets propagated to the calling code.

There is a difference between what the match expression from Listing 9-6 does and what the ? operator does: error values that have the ? operator called on them go through the from function, defined in the From trait in the standard library, which is used to convert values from one type into another. When the ? operator calls the from function, the error type received is converted into the error type defined in the return type of the current function. This is useful when a function returns one error type to represent all the ways a function might fail, even if parts might fail for many different reasons.

For example, we could change the read_username_from_file function in Listing 9-7 to return a custom error type named OurError that we define. If we also define impl From<io::Error> for OurError to construct an instance of OurError from an io::Error, then the ? operator calls in the body of read_username_from_file will call from and convert the error types without needing to add any more code to the function.

In the context of Listing 9-7, the ? at the end of the File::open call will return the value inside an Ok to the variable username_file. If an error occurs, the ? operator will return early out of the whole function and give any Err value to the calling code. The same thing applies to the ? at the end of the read_to_string call.

The ? operator eliminates a lot of boilerplate and makes this function’s implementation simpler. We could even shorten this code further by chaining method calls immediately after the ?, as shown in Listing 9-8.

这 ？ 放置在定义 Result 值之后，其工作方式与我们定义的用于处理清单 9-6 中的 Result 值的匹配表达式几乎相同。 如果 Result 的值为 Ok，则该表达式将返回 Ok 内的值，并且程序将继续。 如果该值是 Err，则 Err 将从整个函数返回，就像我们使用了 return 关键字一样，因此错误值会传播到调用代码。

清单 9-6 中的 match 表达式的作用与 ? 的作用是有区别的。 运算符的作用：具有 ? 的错误值 对它们调用的运算符会经过标准库中 From 特征中定义的 from 函数，该函数用于将值从一种类型转换为另一种类型。 当。。。的时候 ？ 运算符调用 from 函数，将接收到的错误类型转换为当前函数的返回类型中定义的错误类型。 当函数返回一种错误类型来表示函数可能失败的所有方式时（即使某些部分可能由于多种不同原因而失败），这非常有用。

例如，我们可以更改清单 9-7 中的 read_username_from_file 函数，以返回我们定义的名为 OurError 的自定义错误类型。 如果我们还为 OurError 定义 impl From<io::Error> 来从 io::Error 构造 OurError 的实例，那么 ? read_username_from_file 主体中的运算符调用将调用 from 并转换错误类型，而无需向函数添加任何更多代码。

在清单 9-7 的上下文中，? File::open 调用结束时会将 Ok 内的值返回给变量 username_file。 如果发生错误，则 ? 运算符将从整个函数中提前返回，并向调用代码提供任何 Err 值。 同样的事情也适用于？ 在 read_to_string 调用结束时。

这 ？ 运算符消除了大量样板代码，并使该函数的实现更简单。 我们甚至可以通过在 ? 之后立即链接方法调用来进一步缩短此代码，如清单 9-8 所示。

Filename: src/main.rs
```
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let mut username = String::new();

    File::open("hello.txt")?.read_to_string(&mut username)?;

    Ok(username)
}
```
Listing 9-8: Chaining method calls after the ? operator

We’ve moved the creation of the new String in username to the beginning of the function; that part hasn’t changed. Instead of creating a variable username_file, we’ve chained the call to read_to_string directly onto the result of File::open("hello.txt")?. We still have a ? at the end of the read_to_string call, and we still return an Ok value containing username when both File::open and read_to_string succeed rather than returning errors. The functionality is again the same as in Listing 9-6 and Listing 9-7; this is just a different, more ergonomic way to write it.

我们已将用户名中新字符串的创建移至函数的开头； 那部分没有改变。 我们没有创建变量 username_file，而是直接将对 read_to_string 的调用链接到 File::open("hello.txt")? 的结果上。 我们还有一个？ 在 read_to_string 调用结束时，当 File::open 和 read_to_string 都成功时，我们仍然返回包含用户名的 Ok 值，而不是返回错误。 功能与清单 9-6 和清单 9-7 相同； 这只是一种不同的、更符合人体工程学的书写方式。

Listing 9-9 shows a way to make this even shorter using fs::read_to_string.

Filename: src/main.rs
```
use std::fs;
use std::io;

fn read_username_from_file() -> Result<String, io::Error> {
    fs::read_to_string("hello.txt")
}
```
Listing 9-9: Using fs::read_to_string instead of opening and then reading the file

Reading a file into a string is a fairly common operation, so the standard library provides the convenient fs::read_to_string function that opens the file, creates a new String, reads the contents of the file, puts the contents into that String, and returns it. Of course, using fs::read_to_string doesn’t give us the opportunity to explain all the error handling, so we did it the longer way first.

将文件读入字符串是一个相当常见的操作，因此标准库提供了方便的 fs::read_to_string 函数，该函数打开文件，创建一个新的字符串，读取文件的内容，将内容放入该字符串中，然后返回 它。 当然，使用 fs::read_to_string 并不能让我们有机会解释所有错误处理，因此我们首先采用更长的方式。

### Where The ? Operator Can Be Used
The ? operator can only be used in functions whose return type is compatible with the value the ? is used on. This is because the ? operator is defined to perform an early return of a value out of the function, in the same manner as the match expression we defined in Listing 9-6. In Listing 9-6, the match was using a Result value, and the early return arm returned an Err(e) value. The return type of the function has to be a Result so that it’s compatible with this return.

In Listing 9-10, let’s look at the error we’ll get if we use the ? operator in a main function with a return type incompatible with the type of the value we use ? on:

这 ？ 运算符只能用在返回类型与 ? 值兼容的函数中。 用于. 这是因为？ 运算符被定义为执行从函数中提前返回值的操作，其方式与我们在清单 9-6 中定义的匹配表达式相同。 在清单 9-6 中，匹配使用 Result 值，早期返回臂返回 Err(e) 值。 函数的返回类型必须是 Result，以便与此返回兼容。

在清单 9-10 中，让我们看一下如果使用 ? 会出现的错误。 主函数中的运算符的返回类型与我们使用的值的类型不兼容？ 在：

Filename: src/main.rs
```
This code does not compile!
use std::fs::File;

fn main() {
    let greeting_file = File::open("hello.txt")?;
}
```
Listing 9-10: Attempting to use the ? in the main function that returns () won’t compile

This code opens a file, which might fail. The ? operator follows the Result value returned by File::open, but this main function has the return type of (), not Result. When we compile this code, we get the following error message:

此代码打开一个文件，这可能会失败。 这 ？ 运算符遵循 File::open 返回的 Result 值，但此 main 函数的返回类型为 ()，而不是 Result。 当我们编译这段代码时，我们收到以下错误消息：

```
$ cargo run
   Compiling error-handling v0.1.0 (file:///projects/error-handling)
error[E0277]: the `?` operator can only be used in a function that returns `Result` or `Option` (or another type that implements `FromResidual`)
 --> src/main.rs:4:48
  |
3 | fn main() {
  | --------- this function should return `Result` or `Option` to accept `?`
4 |     let greeting_file = File::open("hello.txt")?;
  |                                                ^ cannot use the `?` operator in a function that returns `()`
  |
  = help: the trait `FromResidual<Result<Infallible, std::io::Error>>` is not implemented for `()`

For more information about this error, try `rustc --explain E0277`.
error: could not compile `error-handling` due to previous error
```
This error points out that we’re only allowed to use the ? operator in a function that returns Result, Option, or another type that implements FromResidual.

To fix the error, you have two choices. One choice is to change the return type of your function to be compatible with the value you’re using the ? operator on as long as you have no restrictions preventing that. The other technique is to use a match or one of the Result<T, E> methods to handle the Result<T, E> in whatever way is appropriate.

The error message also mentioned that ? can be used with Option<T> values as well. As with using ? on Result, you can only use ? on Option in a function that returns an Option. The behavior of the ? operator when called on an Option<T> is similar to its behavior when called on a Result<T, E>: if the value is None, the None will be returned early from the function at that point. If the value is Some, the value inside the Some is the resulting value of the expression and the function continues. Listing 9-11 has an example of a function that finds the last character of the first line in the given text:

这个错误指出我们只允许使用 ? 返回 Result、Option 或实现 FromResidual 的其他类型的函数中的运算符。

要修复该错误，您有两种选择。 一种选择是更改函数的返回类型以与您正在使用 ? 的值兼容。 只要您没有任何限制，就可以使用运算符。 另一种技术是使用匹配或 Result<T, E> 方法之一以任何适当的方式处理 Result<T, E>。

错误消息还提到了？ 也可以与 Option<T> 值一起使用。 与使用 ? 在结果上，你只能使用 ? 返回 Option 的函数中的 Option。 的行为？ 在 Option<T> 上调用时的操作符与在 Result<T, E> 上调用时的行为类似：如果值为 None，则此时函数将提前返回 None。 如果值为 Some，则 Some 内的值是表达式的结果值，并且函数继续。 清单 9-11 有一个函数示例，该函数查找给定文本中第一行的最后一个字符：

```
fn last_char_of_first_line(text: &str) -> Option<char> {
    text.lines().next()?.chars().last()
}
```
Listing 9-11: Using the ? operator on an Option<T> value

This function returns Option<char> because it’s possible that there is a character there, but it’s also possible that there isn’t. This code takes the text string slice argument and calls the lines method on it, which returns an iterator over the lines in the string. Because this function wants to examine the first line, it calls next on the iterator to get the first value from the iterator. If text is the empty string, this call to next will return None, in which case we use ? to stop and return None from last_char_of_first_line. If text is not the empty string, next will return a Some value containing a string slice of the first line in text.

The ? extracts the string slice, and we can call chars on that string slice to get an iterator of its characters. We’re interested in the last character in this first line, so we call last to return the last item in the iterator. This is an Option because it’s possible that the first line is the empty string, for example if text starts with a blank line but has characters on other lines, as in "\nhi". However, if there is a last character on the first line, it will be returned in the Some variant. The ? operator in the middle gives us a concise way to express this logic, allowing us to implement the function in one line. If we couldn’t use the ? operator on Option, we’d have to implement this logic using more method calls or a match expression.

Note that you can use the ? operator on a Result in a function that returns Result, and you can use the ? operator on an Option in a function that returns Option, but you can’t mix and match. The ? operator won’t automatically convert a Result to an Option or vice versa; in those cases, you can use methods like the ok method on Result or the ok_or method on Option to do the conversion explicitly.

So far, all the main functions we’ve used return (). The main function is special because it’s the entry and exit point of executable programs, and there are restrictions on what its return type can be for the programs to behave as expected.

Luckily, main can also return a Result<(), E>. Listing 9-12 has the code from Listing 9-10 but we’ve changed the return type of main to be Result<(), Box<dyn Error>> and added a return value Ok(()) to the end. This code will now compile:

该函数返回 Option<char>，因为那里可能有一个字符，但也可能没有。 此代码采用文本字符串切片参数并对其调用lines方法，该方法返回字符串中各行的迭代器。 因为该函数想要检查第一行，所以它调用迭代器的 next 以从迭代器获取第一个值。 如果 text 是空字符串，则对 next 的调用将返回 None，在这种情况下我们使用 ? 停止并从last_char_of_first_line返回None。 如果 text 不是空字符串，则 next 将返回一个 Some 值，其中包含 text 第一行的字符串切片。

这 ？ 提取字符串切片，我们可以在该字符串切片上调用 chars 来获取其字符的迭代器。 我们对第一行的最后一个字符感兴趣，因此我们调用 last 来返回迭代器中的最后一项。 这是一个选项，因为第一行可能是空字符串，例如，如果文本以空行开头，但其他行上有字符，如“\nhi”。 但是，如果第一行有最后一个字符，它将在 Some 变体中返回。 这 ？ 中间的运算符为我们提供了一种简洁的方式来表达这个逻辑，使我们能够在一行内实现该功能。 如果我们不能使用 ? Option 上的运算符，我们必须使用更多方法调用或匹配表达式来实现此逻辑。

请注意，您可以使用 ? 在返回 Result 的函数中对 Result 进行运算符，您可以使用 ? 返回 Option 的函数中 Option 的运算符，但不能混合搭配。 这 ？ 运算符不会自动将结果转换为选项，反之亦然； 在这些情况下，您可以使用 Result 上的 ok 方法或 Option 上的 ok_or 方法等方法来显式执行转换。

到目前为止，我们使用的所有主要函数都是return()。 main 函数很特殊，因为它是可执行程序的入口点和出口点，并且其返回类型对于程序按预期运行有限制。

幸运的是，main 还可以返回 Result<(), E>。 清单 9-12 包含清单 9-10 中的代码，但我们将 main 的返回类型更改为 Result<(), Box<dyn Error>> 并在末尾添加了一个返回值 Ok(())。 该代码现在将编译：

```
use std::error::Error;
use std::fs::File;

fn main() -> Result<(), Box<dyn Error>> {
    let greeting_file = File::open("hello.txt")?;

    Ok(())
}
```
Listing 9-12: Changing main to return Result<(), E> allows the use of the ? operator on Result values

The Box<dyn Error> type is a trait object, which we’ll talk about in the “Using Trait Objects that Allow for Values of Different Types” section in Chapter 17. For now, you can read Box<dyn Error> to mean “any kind of error.” Using ? on a Result value in a main function with the error type Box<dyn Error> is allowed, because it allows any Err value to be returned early. Even though the body of this main function will only ever return errors of type std::io::Error, by specifying Box<dyn Error>, this signature will continue to be correct even if more code that returns other errors is added to the body of main.

When a main function returns a Result<(), E>, the executable will exit with a value of 0 if main returns Ok(()) and will exit with a nonzero value if main returns an Err value. Executables written in C return integers when they exit: programs that exit successfully return the integer 0, and programs that error return some integer other than 0. Rust also returns integers from executables to be compatible with this convention.

The main function may return any types that implement the std::process::Termination trait, which contains a function report that returns an ExitCode. Consult the standard library documentation for more information on implementing the Termination trait for your own types.

Now that we’ve discussed the details of calling panic! or returning Result, let’s return to the topic of how to decide which is appropriate to use in which cases.

Box<dyn Error> 类型是一个特征对象，我们将在第 17 章的“使用允许不同类型值的特征对象”部分中讨论它。现在，您可以将 Box<dyn Error> 理解为 “任何类型的错误。” 使用 ？ 允许在主函数中错误类型为 Box<dyn Error> 的 Result 值上执行操作，因为它允许提前返回任何 Err 值。 即使此主函数的主体只会返回 std::io::Error 类型的错误，但通过指定 Box<dyn Error>，即使将返回其他错误的更多代码添加到该函数中，此签名也将继续正确。 主体的主体。

当 main 函数返回 Result<(), E> 时，如果 main 返回 Ok(())，则可执行文件将以 0 值退出；如果 main 返回 Err 值，则可执行文件将以非零值退出。 用 C 编写的可执行文件在退出时返回整数：成功退出的程序返回整数 0，出错的程序返回 0 以外的某个整数。Rust 还从可执行文件返回整数以与此约定兼容。

main 函数可以返回实现 std::process::Termination 特征的任何类型，其中包含返回 ExitCode 的函数报告。 有关为您自己的类型实现终止特征的更多信息，请参阅标准库文档。

现在我们已经讨论了调用恐慌的细节！ 或者返回 Result，让我们回到如何决定哪种情况适合使用的主题。

## 9.3 ToPanic or Not to panic!

So how do you decide when you should call panic! and when you should return Result? When code panics, there’s no way to recover. You could call panic! for any error situation, whether there’s a possible way to recover or not, but then you’re making the decision that a situation is unrecoverable on behalf of the calling code. When you choose to return a Result value, you give the calling code options. The calling code could choose to attempt to recover in a way that’s appropriate for its situation, or it could decide that an Err value in this case is unrecoverable, so it can call panic! and turn your recoverable error into an unrecoverable one. Therefore, returning Result is a good default choice when you’re defining a function that might fail.

In situations such as examples, prototype code, and tests, it’s more appropriate to write code that panics instead of returning a Result. Let’s explore why, then discuss situations in which the compiler can’t tell that failure is impossible, but you as a human can. The chapter will conclude with some general guidelines on how to decide whether to panic in library code.

那么你如何决定何时应该调用恐慌！ 什么时候应该返回 Result？ 当代码发生恐慌时，没有办法恢复。 你可以称之为恐慌！ 对于任何错误情况，无论是否有可能的方法来恢复，但随后您将代表调用代码做出无法恢复的情况的决定。 当您选择返回结果值时，您为调用代码提供了选项。 调用代码可以选择尝试以适合其情况的方式进行恢复，或者它可以决定这种情况下的 Err 值是不可恢复的，因此它可以调用恐慌！ 并将您的可恢复错误变成不可恢复错误。 因此，当您定义可能失败的函数时，返回 Result 是一个不错的默认选择。

在示例、原型代码和测试等情况下，编写发生恐慌的代码比返回结果更合适。 让我们探讨一下原因，然后讨论编译器无法判断失败是不可能的情况，但你作为人类可以。 本章最后将提供一些关于如何决定是否在库代码中出现恐慌的一般准则。


### Examples, Prototype Code, and Tests
When you’re writing an example to illustrate some concept, also including robust error-handling code can make the example less clear. In examples, it’s understood that a call to a method like unwrap that could panic is meant as a placeholder for the way you’d want your application to handle errors, which can differ based on what the rest of your code is doing.

Similarly, the unwrap and expect methods are very handy when prototyping, before you’re ready to decide how to handle errors. They leave clear markers in your code for when you’re ready to make your program more robust.

If a method call fails in a test, you’d want the whole test to fail, even if that method isn’t the functionality under test. Because panic! is how a test is marked as a failure, calling unwrap or expect is exactly what should happen.

当您编写示例来说明某些概念时，还包含强大的错误处理代码可能会使示例不太清晰。 在示例中，可以理解的是，对像 unwrap 这样可能会发生恐慌的方法的调用意味着作为您希望应用程序处理错误的方式的占位符，这可能会根据其余代码正在执行的操作而有所不同。

同样，在您准备好决定如何处理错误之前，unwrap 和 Expect 方法在原型设计时非常方便。 它们会在您的代码中留下清晰的标记，以便您准备好让您的程序更加健壮。

如果测试中的方法调用失败，您会希望整个测试失败，即使该方法不是正在测试的功能。 因为恐慌！ 是测试被标记为失败的方式，调用 unwrap 或 Expect 正是应该发生的事情。

### Cases in Which You Have More Information Than the Compiler
It would also be appropriate to call unwrap or expect when you have some other logic that ensures the Result will have an Ok value, but the logic isn’t something the compiler understands. You’ll still have a Result value that you need to handle: whatever operation you’re calling still has the possibility of failing in general, even though it’s logically impossible in your particular situation. If you can ensure by manually inspecting the code that you’ll never have an Err variant, it’s perfectly acceptable to call unwrap, and even better to document the reason you think you’ll never have an Err variant in the expect text. Here’s an example:

当您有一些其他逻辑确保 Result 将具有 Ok 值时，调用 unwrap 或 Expect 也是合适的，但编译器无法理解这些逻辑。 您仍然需要处理一个 Result 值：无论您调用什么操作，通常仍然有可能失败，即使在您的特定情况下逻辑上这是不可能的。 如果您可以通过手动检查代码来确保永远不会出现 Err 变体，那么调用 unwrap 是完全可以接受的，甚至更好地在期望文本中记录您认为永远不会出现 Err 变体的原因。 这是一个例子：

```
    use std::net::IpAddr;

    let home: IpAddr = "127.0.0.1"
        .parse()
        .expect("Hardcoded IP address should be valid");
```
We’re creating an IpAddr instance by parsing a hardcoded string. We can see that 127.0.0.1 is a valid IP address, so it’s acceptable to use expect here. However, having a hardcoded, valid string doesn’t change the return type of the parse method: we still get a Result value, and the compiler will still make us handle the Result as if the Err variant is a possibility because the compiler isn’t smart enough to see that this string is always a valid IP address. If the IP address string came from a user rather than being hardcoded into the program and therefore did have a possibility of failure, we’d definitely want to handle the Result in a more robust way instead. Mentioning the assumption that this IP address is hardcoded will prompt us to change expect to better error handling code if in the future, we need to get the IP address from some other source instead.

我们通过解析硬编码字符串来创建 IpAddr 实例。 我们可以看到127.0.0.1是一个有效的IP地址，所以这里使用expect是可以接受的。 然而，拥有一个硬编码的有效字符串不会改变解析方法的返回类型：我们仍然得到一个 Result 值，并且编译器仍然会让我们处理 Result，就好像 Err 变体是可能的，因为编译器不' 足够聪明，可以看出该字符串始终是有效的 IP 地址。 如果 IP 地址字符串来自用户而不是硬编码到程序中，因此确实有失败的可能性，我们肯定希望以更稳健的方式处理结果。 如果将来我们需要从其他来源获取 IP 地址，提及此 IP 地址是硬编码的假设将促使我们更改期望以更好的错误处理代码。

### Guidelines for Error Handling
It’s advisable to have your code panic when it’s possible that your code could end up in a bad state. In this context, a bad state is when some assumption, guarantee, contract, or invariant has been broken, such as when invalid values, contradictory values, or missing values are passed to your code—plus one or more of the following:

+ The bad state is something that is unexpected, as opposed to something that will likely happen occasionally, like a user entering data in the wrong format.
+ Your code after this point needs to rely on not being in this bad state, rather than checking for the problem at every step.
+ There’s not a good way to encode this information in the types you use. We’ll work through an example of what we mean in the “Encoding States and Behavior as Types” section of Chapter 17.

If someone calls your code and passes in values that don’t make sense, it’s best to return an error if you can so the user of the library can decide what they want to do in that case. However, in cases where continuing could be insecure or harmful, the best choice might be to call panic! and alert the person using your library to the bug in their code so they can fix it during development. Similarly, panic! is often appropriate if you’re calling external code that is out of your control and it returns an invalid state that you have no way of fixing.

However, when failure is expected, it’s more appropriate to return a Result than to make a panic! call. Examples include a parser being given malformed data or an HTTP request returning a status that indicates you have hit a rate limit. In these cases, returning a Result indicates that failure is an expected possibility that the calling code must decide how to handle.

When your code performs an operation that could put a user at risk if it’s called using invalid values, your code should verify the values are valid first and panic if the values aren’t valid. This is mostly for safety reasons: attempting to operate on invalid data can expose your code to vulnerabilities. This is the main reason the standard library will call panic! if you attempt an out-of-bounds memory access: trying to access memory that doesn’t belong to the current data structure is a common security problem. Functions often have contracts: their behavior is only guaranteed if the inputs meet particular requirements. Panicking when the contract is violated makes sense because a contract violation always indicates a caller-side bug and it’s not a kind of error you want the calling code to have to explicitly handle. In fact, there’s no reasonable way for calling code to recover; the calling programmers need to fix the code. Contracts for a function, especially when a violation will cause a panic, should be explained in the API documentation for the function.

However, having lots of error checks in all of your functions would be verbose and annoying. Fortunately, you can use Rust’s type system (and thus the type checking done by the compiler) to do many of the checks for you. If your function has a particular type as a parameter, you can proceed with your code’s logic knowing that the compiler has already ensured you have a valid value. For example, if you have a type rather than an Option, your program expects to have something rather than nothing. Your code then doesn’t have to handle two cases for the Some and None variants: it will only have one case for definitely having a value. Code trying to pass nothing to your function won’t even compile, so your function doesn’t have to check for that case at runtime. Another example is using an unsigned integer type such as u32, which ensures the parameter is never negative.

当您的代码可能最终处于错误状态时，建议让您的代码出现恐慌。 在这种情况下，坏状态是指某些假设、保证、契约或不变量被破坏，例如当无效值、矛盾值或缺失值传递到代码时，加上以下一项或多项：

+ 不良状态是指意外的情况，而不是偶尔发生的情况，例如用户以错误的格式输入数据。
+ 此时您的代码需要依赖于不处于这种糟糕的状态，而不是在每一步都检查问题。
+ 没有一个好的方法可以用您使用的类型来编码这些信息。 我们将通过一个例子来解释第 17 章“将状态和行为编码为类型”部分的含义。

如果有人调用您的代码并传入没有意义的值，最好返回一个错误（如果可以的话），以便库的用户可以决定在这种情况下他们想要做什么。 然而，如果继续可能不安全或有害，最好的选择可能是恐慌！ 并提醒使用您库的人注意其代码中的错误，以便他们可以在开发过程中修复它。 同样，恐慌！ 如果您调用不受您控制的外部代码并且它返回您无法修复的无效状态，则通常是合适的。

然而，当预计会失败时，返回 Result 比恐慌更合适！ 称呼。 示例包括向解析器提供格式错误的数据或返回指示已达到速率限制的状态的 HTTP 请求。 在这些情况下，返回 Result 表明失败是一种预期的可能性，调用代码必须决定如何处理。

当您的代码执行一项操作（如果使用无效值调用该操作可能会使用户面临风险）时，您的代码应首先验证这些值是否有效，如果这些值无效，则发生恐慌。 这主要是出于安全原因：尝试对无效数据进行操作可能会使您的代码面临漏洞。 这是标准库会调用panic的主要原因！ 如果尝试越界内存访问：尝试访问不属于当前数据结构的内存是一个常见的安全问题。 函数通常有契约：只有当输入满足特定要求时，它们的行为才能得到保证。 当违反合同时出现恐慌是有道理的，因为违反合同总是表明调用方存在错误，并且这不是您希望调用代码必须显式处理的错误。 事实上，没有合理的方法来调用代码进行恢复； 调用程序员需要修复代码。 函数的合约，尤其是当违规会导致恐慌时，应该在函数的 API 文档中进行解释。

然而，在所有函数中进行大量错误检查将是冗长且烦人的。 幸运的是，您可以使用 Rust 的类型系统（以及编译器完成的类型检查）来为您完成许多检查。 如果您的函数具有特定类型作为参数，您可以继续执行代码逻辑，因为编译器已经确保您具有有效值。 例如，如果您有一个类型而不是一个选项，那么您的程序期望有一些东西而不是没有。 这样，您的代码就不必处理 Some 和 None 变体的两种情况：它只会有一种肯定有值的情况。 尝试向函数传递任何内容的代码甚至无法编译，因此您的函数不必在运行时检查这种情况。 另一个示例是使用无符号整数类型（例如 u32），这可确保参数永远不会为负数。

### Creating Custom Types for Validation
Let’s take the idea of using Rust’s type system to ensure we have a valid value one step further and look at creating a custom type for validation. Recall the guessing game in Chapter 2 in which our code asked the user to guess a number between 1 and 100. We never validated that the user’s guess was between those numbers before checking it against our secret number; we only validated that the guess was positive. In this case, the consequences were not very dire: our output of “Too high” or “Too low” would still be correct. But it would be a useful enhancement to guide the user toward valid guesses and have different behavior when a user guesses a number that’s out of range versus when a user types, for example, letters instead.

One way to do this would be to parse the guess as an i32 instead of only a u32 to allow potentially negative numbers, and then add a check for the number being in range, like so:

让我们进一步考虑使用 Rust 的类型系统来确保我们有一个有效的值，并考虑创建一个用于验证的自定义类型。 回想一下第 2 章中的猜谜游戏，其中我们的代码要求用户猜测 1 到 100 之间的数字。在与我们的秘密数字进行检查之前，我们从未验证过用户的猜测是否在这些数字之间；我们从未验证过用户的猜测是否在这些数字之间。 我们只是验证了猜测是肯定的。 在这种情况下，后果并不是非常可怕：我们的“太高”或“太低”输出仍然是正确的。 但这将是一个有用的增强功能，可以引导用户进行有效的猜测，并且当用户猜测超出范围的数字时与用户键入字母（例如，字母）时具有不同的行为。

一种方法是将猜测解析为 i32 而不是仅解析 u32 以允许潜在的负数，然后添加对范围内数字的检查，如下所示：

```
    loop {
        // --snip--

        let guess: i32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        if guess < 1 || guess > 100 {
            println!("The secret number will be between 1 and 100.");
            continue;
        }

        match guess.cmp(&secret_number) {
            // --snip--
    }
```
The if expression checks whether our value is out of range, tells the user about the problem, and calls continue to start the next iteration of the loop and ask for another guess. After the if expression, we can proceed with the comparisons between guess and the secret number knowing that guess is between 1 and 100.

However, this is not an ideal solution: if it was absolutely critical that the program only operated on values between 1 and 100, and it had many functions with this requirement, having a check like this in every function would be tedious (and might impact performance).

Instead, we can make a new type and put the validations in a function to create an instance of the type rather than repeating the validations everywhere. That way, it’s safe for functions to use the new type in their signatures and confidently use the values they receive. Listing 9-13 shows one way to define a Guess type that will only create an instance of Guess if the new function receives a value between 1 and 100.

if 表达式检查我们的值是否超出范围，告诉用户问题，然后调用 continue 开始循环的下一次迭代并要求再次猜测。 在 if 表达式之后，我们可以继续比较猜测值和秘密数字，知道猜测值在 1 到 100 之间。

然而，这不是一个理想的解决方案：如果程序仅对 1 到 100 之间的值进行操作是绝对关键的，并且它有许多满足此要求的函数，那么在每个函数中进行这样的检查将是乏味的（并且可能会影响 表现）。

相反，我们可以创建一个新类型并将验证放入函数中以创建该类型的实例，而不是在各处重复验证。 这样，函数就可以安全地在其签名中使用新类型并自信地使用它们收到的值。 清单 9-13 显示了定义 Guess 类型的一种方法，该方法仅在新函数接收到 1 到 100 之间的值时才会创建 Guess 实例。

```
pub struct Guess {
    value: i32,
}

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess value must be between 1 and 100, got {}.", value);
        }

        Guess { value }
    }

    pub fn value(&self) -> i32 {
        self.value
    }
}
```
Listing 9-13: A Guess type that will only continue with values between 1 and 100

First, we define a struct named Guess that has a field named value that holds an i32. This is where the number will be stored.

Then we implement an associated function named new on Guess that creates instances of Guess values. The new function is defined to have one parameter named value of type i32 and to return a Guess. The code in the body of the new function tests value to make sure it’s between 1 and 100. If value doesn’t pass this test, we make a panic! call, which will alert the programmer who is writing the calling code that they have a bug they need to fix, because creating a Guess with a value outside this range would violate the contract that Guess::new is relying on. The conditions in which Guess::new might panic should be discussed in its public-facing API documentation; we’ll cover documentation conventions indicating the possibility of a panic! in the API documentation that you create in Chapter 14. If value does pass the test, we create a new Guess with its value field set to the value parameter and return the Guess.

Next, we implement a method named value that borrows self, doesn’t have any other parameters, and returns an i32. This kind of method is sometimes called a getter, because its purpose is to get some data from its fields and return it. This public method is necessary because the value field of the Guess struct is private. It’s important that the value field be private so code using the Guess struct is not allowed to set value directly: code outside the module must use the Guess::new function to create an instance of Guess, thereby ensuring there’s no way for a Guess to have a value that hasn’t been checked by the conditions in the Guess::new function.

A function that has a parameter or returns only numbers between 1 and 100 could then declare in its signature that it takes or returns a Guess rather than an i32 and wouldn’t need to do any additional checks in its body.

首先，我们定义一个名为 Guess 的结构，它有一个名为 value 的字段，该字段保存 i32。 这是号码将被存储的地方。

然后我们在 Guess 上实现一个名为 new 的关联函数，用于创建 Guess 值的实例。 新函数被定义为具有一个名为 i32 类型的 value 的参数并返回 Guess。 新函数主体中的代码测试 value 以确保它在 1 到 100 之间。如果 value 没有通过此测试，我们会发生恐慌！ 调用，这将提醒正在编写调用代码的程序员，他们有一个需要修复的错误，因为创建具有超出此范围的值的 Guess 将违反 Guess::new 所依赖的约定。 Guess::new 可能出现恐慌的情况应该在其面向公众的 API 文档中讨论； 我们将介绍表明恐慌可能性的文档约定！ 在第 14 章创建的 API 文档中。如果 value 确实通过了测试，我们将创建一个新的 Guess，并将其 value 字段设置为 value 参数并返回 Guess。

接下来，我们实现一个名为 value 的方法，该方法借用 self，没有任何其他参数，并返回 i32。 这种方法有时称为 getter，因为它的目的是从其字段中获取一些数据并返回它。 这个公共方法是必要的，因为 Guess 结构的值字段是私有的。 值字段必须是私有的，这一点很重要，因此使用 Guess 结构的代码不允许直接设置值：模块外部的代码必须使用 Guess::new 函数来创建 Guess 的实例，从而确保 Guess 无法直接设置值。 具有未经 Guess::new 函数中的条件检查的值。

具有参数或仅返回 1 到 100 之间数字的函数可以在其签名中声明它接受或返回 Guess 而不是 i32，并且不需要在其主体中进行任何额外的检查。

## 9.4 总结
Rust’s error handling features are designed to help you write more robust code. The panic! macro signals that your program is in a state it can’t handle and lets you tell the process to stop instead of trying to proceed with invalid or incorrect values. The Result enum uses Rust’s type system to indicate that operations might fail in a way that your code could recover from. You can use Result to tell code that calls your code that it needs to handle potential success or failure as well. Using panic! and Result in the appropriate situations will make your code more reliable in the face of inevitable problems.

Now that you’ve seen useful ways that the standard library uses generics with the Option and Result enums, we’ll talk about how generics work and how you can use them in your code.

Rust 的错误处理功能旨在帮助您编写更健壮的代码。 恐慌！ 宏表示您的程序处于无法处理的状态，并让您告诉进程停止，而不是尝试使用无效或不正确的值继续。 Result 枚举使用 Rust 的类型系统来指示操作可能会失败，而您的代码可以从中恢复。 您可以使用 Result 告诉调用您的代码的代码，它也需要处理潜在的成功或失败。 使用恐慌！ 在适当的情况下结果将使您的代码在面对不可避免的问题时更加可靠。

现在您已经了解了标准库通过 Option 和 Result 枚举使用泛型的有用方法，我们将讨论泛型的工作原理以及如何在代码中使用它们。
