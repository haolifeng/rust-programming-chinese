# 16. 并发(Fearleass Concurrency)
Handling concurrent programming safely and efficiently is another of Rust’s major goals. Concurrent programming, where different parts of a program execute independently, and parallel programming, where different parts of a program execute at the same time, are becoming increasingly important as more computers take advantage of their multiple processors. Historically, programming in these contexts has been difficult and error prone: Rust hopes to change that.

Initially, the Rust team thought that ensuring memory safety and preventing concurrency problems were two separate challenges to be solved with different methods. Over time, the team discovered that the ownership and type systems are a powerful set of tools to help manage memory safety and concurrency problems! By leveraging ownership and type checking, many concurrency errors are compile-time errors in Rust rather than runtime errors. Therefore, rather than making you spend lots of time trying to reproduce the exact circumstances under which a runtime concurrency bug occurs, incorrect code will refuse to compile and present an error explaining the problem. As a result, you can fix your code while you’re working on it rather than potentially after it has been shipped to production. We’ve nicknamed this aspect of Rust fearless concurrency. Fearless concurrency allows you to write code that is free of subtle bugs and is easy to refactor without introducing new bugs.

安全高效地处理并发编程是 Rust 的另一个主要目标。 随着越来越多的计算机利用其多个处理器，并发编程（程序的不同部分独立执行）和并行编程（程序的不同部分同时执行）变得越来越重要。 从历史上看，在这些环境中编程一直是困难且容易出错的：Rust 希望改变这一点。

最初，Rust 团队认为确保内存安全和防止并发问题是两个独立的挑战，需要用不同的方法来解决。 随着时间的推移，团队发现所有权和类型系统是一组强大的工具，可以帮助管理内存安全和并发问题！ 通过利用所有权和类型检查，许多并发错误是 Rust 中的编译时错误，而不是运行时错误。 因此，不正确的代码将拒绝编译并显示解释问题的错误，而不是让您花费大量时间尝试重现发生运行时并发错误的确切情况。 因此，您可以在处理代码时修复代码，而不是在将代码交付生产后修复。 我们给 Rust 无畏并发的这一方面起了个绰号。 无畏并发允许您编写没有细微错误的代码，并且易于重构而不会引入新的错误。

``````
Note: For simplicity’s sake, we’ll refer to many of the problems as concurrent rather than being more precise by saying concurrent and/or parallel. If this book were about concurrency and/or parallelism, we’d be more specific. For this chapter, please mentally substitute concurrent and/or parallel whenever we use concurrent.
注意：为了简单起见，我们将许多问题称为并发问题，而不是更精确地说并发和/或并行。 如果这本书是关于并发和/或并行性的，我们会更具体。 在本章中，每当我们使用并发时，请在心里用并发和/或并行代替。
``````

Many languages are dogmatic about the solutions they offer for handling concurrent problems. For example, Erlang has elegant functionality for message-passing concurrency but has only obscure ways to share state between threads. Supporting only a subset of possible solutions is a reasonable strategy for higher-level languages, because a higher-level language promises benefits from giving up some control to gain abstractions. However, lower-level languages are expected to provide the solution with the best performance in any given situation and have fewer abstractions over the hardware. Therefore, Rust offers a variety of tools for modeling problems in whatever way is appropriate for your situation and requirements.

许多语言对于它们为处理并发问题提供的解决方案都是教条的。 例如，Erlang 具有优雅的消息传递并发功能，但在线程之间共享状态的方式却很晦涩。 对于高级语言来说，仅支持可能解决方案的子集是一个合理的策略，因为高级语言承诺通过放弃某些控制来获得抽象而带来好处。 然而，较低级语言有望在任何给定情况下提供最佳性能的解决方案，并且对硬件的抽象较少。 因此，Rust 提供了各种工具来以适合您的情况和要求的方式对问题进行建模。

Here are the topics we’ll cover in this chapter:

+ How to create threads to run multiple pieces of code at the same time
+ Message-passing concurrency, where channels send messages between threads
+ Shared-state concurrency, where multiple threads have access to some piece of data
+ The Sync and Send traits, which extend Rust’s concurrency guarantees to user-defined types as well as types provided by the standard library

+ 如何创建线程同时运行多段代码
+ 消息传递并发，通道在线程之间发送消息
+ 共享状态并发，多个线程可以访问某些数据
+ Sync 和 Send 特性，将 Rust 的并发保证扩展到用户定义的类型以及标准库提供的类型

## 16.1 Using Threads to Run Code Simulataneously

In most current operating systems, an executed program’s code is run in a process, and the operating system will manage multiple processes at once. Within a program, you can also have independent parts that run simultaneously. The features that run these independent parts are called threads. For example, a web server could have multiple threads so that it could respond to more than one request at the same time.

Splitting the computation in your program into multiple threads to run multiple tasks at the same time can improve performance, but it also adds complexity. Because threads can run simultaneously, there’s no inherent guarantee about the order in which parts of your code on different threads will run. This can lead to problems, such as:

+ Race conditions, where threads are accessing data or resources in an inconsistent order
+ Deadlocks, where two threads are waiting for each other, preventing both threads from continuing
+ Bugs that happen only in certain situations and are hard to reproduce and fix reliably

Rust attempts to mitigate the negative effects of using threads, but programming in a multithreaded context still takes careful thought and requires a code structure that is different from that in programs running in a single thread.

Programming languages implement threads in a few different ways, and many operating systems provide an API the language can call for creating new threads. The Rust standard library uses a 1:1 model of thread implementation, whereby a program uses one operating system thread per one language thread. There are crates that implement other models of threading that make different tradeoffs to the 1:1 model.

在大多数当前的操作系统中，执行程序的代码在一个进程中运行，并且操作系统将同时管理多个进程。 在程序中，您还可以拥有同时运行的独立部分。 运行这些独立部分的功能称为线程。 例如，一台 Web 服务器可以有多个线程，以便它可以同时响应多个请求。

将程序中的计算拆分为多个线程以同时运行多个任务可以提高性能，但也会增加复杂性。 由于线程可以同时运行，因此对于不同线程上的代码部分的运行顺序没有固有的保证。 这可能会导致问题，例如：

+ 竞争条件，线程以不一致的顺序访问数据或资源
+ 死锁，两个线程互相等待，阻止两个线程继续
+ 仅在某些情况下发生且难以可靠地重现和修复的错误

Rust 试图减轻使用线程的负面影响，但在多线程上下文中编程仍然需要仔细考虑，并且需要与在单线程中运行的程序不同的代码结构。

编程语言以几种不同的方式实现线程，许多操作系统提供了一个 API，该语言可以调用该 API 来创建新线程。 Rust 标准库使用 1:1 的线程实现模型，即程序针对一种语言线程使用一个操作系统线程。 有些包实现了其他线程模型，对 1:1 模型进行了不同的权衡。

###  Creating a New Thread with spawn
To create a new thread, we call the thread::spawn function and pass it a closure (we talked about closures in Chapter 13) containing the code we want to run in the new thread. The example in Listing 16-1 prints some text from a main thread and other text from a new thread:

为了创建一个新线程，我们调用 thread::spawn 函数并向它传递一个闭包（我们在第 13 章中讨论过闭包），其中包含我们想要在新线程中运行的代码。 清单 16-1 中的示例从主线程打印一些文本，从新线程打印其他文本：

Filename: src/main.rs
``````
use std::thread;
use std::time::Duration;

fn main() {
    thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
}
``````
Listing 16-1: Creating a new thread to print one thing while the main thread prints something else

Note that when the main thread of a Rust program completes, all spawned threads are shut down, whether or not they have finished running. The output from this program might be a little different every time, but it will look similar to the following:

请注意，当 Rust 程序的主线程完成时，所有生成的线程都会关闭，无论它们是否已完成运行。 该程序的输出每次可能略有不同，但看起来类似于以下内容：

``````
hi number 1 from the main thread!
hi number 1 from the spawned thread!
hi number 2 from the main thread!
hi number 2 from the spawned thread!
hi number 3 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the main thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
``````
The calls to thread::sleep force a thread to stop its execution for a short duration, allowing a different thread to run. The threads will probably take turns, but that isn’t guaranteed: it depends on how your operating system schedules the threads. In this run, the main thread printed first, even though the print statement from the spawned thread appears first in the code. And even though we told the spawned thread to print until i is 9, it only got to 5 before the main thread shut down.

If you run this code and only see output from the main thread, or don’t see any overlap, try increasing the numbers in the ranges to create more opportunities for the operating system to switch between the threads.

对 thread::sleep 的调用会强制线程在短时间内停止执行，从而允许另一个线程运行。 线程可能会轮流运行，但这并不能保证：这取决于操作系统如何调度线程。 在此运行中，主线程首先打印，即使来自生成线程的打印语句首先出现在代码中。 即使我们告诉生成的线程打印直到 i 为 9，但在主线程关闭之前它只达到 5。

如果运行此代码并且只看到主线程的输出，或者没有看到任何重叠，请尝试增加范围中的数字，以便为操作系统在线程之间切换创造更多机会。


###  Waiting for All Threads to Finish Using join Handles
The code in Listing 16-1 not only stops the spawned thread prematurely most of the time due to the main thread ending, but because there is no guarantee on the order in which threads run, we also can’t guarantee that the spawned thread will get to run at all!

We can fix the problem of the spawned thread not running or ending prematurely by saving the return value of thread::spawn in a variable. The return type of thread::spawn is JoinHandle. A JoinHandle is an owned value that, when we call the join method on it, will wait for its thread to finish. Listing 16-2 shows how to use the JoinHandle of the thread we created in Listing 16-1 and call join to make sure the spawned thread finishes before main exits:

清单16-1中的代码不仅在大多数情况下会因主线程结束而提前停止生成的线程，而且由于无法保证线程运行的顺序，我们也无法保证生成的线程会 快跑吧！

我们可以通过将 thread::spawn 的返回值保存在变量中来解决生成的线程未运行或提前结束的问题。 thread::spawn的返回类型是JoinHandle。 JoinHandle 是一个拥有的值，当我们对其调用 join 方法时，它将等待其线程完成。 清单 16-2 显示了如何使用我们在清单 16-1 中创建的线程的 JoinHandle 并调用 join 以确保生成的线程在 main 退出之前完成：

Filename: src/main.rs
``````
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }

    handle.join().unwrap();
}
``````
Listing 16-2: Saving a JoinHandle from thread::spawn to guarantee the thread is run to completion

Calling join on the handle blocks the thread currently running until the thread represented by the handle terminates. Blocking a thread means that thread is prevented from performing work or exiting. Because we’ve put the call to join after the main thread’s for loop, running Listing 16-2 should produce output similar to this:

在句柄上调用 join 会阻塞当前正在运行的线程，直到句柄所代表的线程终止。 阻塞线程意味着阻止线程执行工作或退出。 因为我们将对 join 的调用放在主线程的 for 循环之后，所以运行清单 16-2 应该会产生与此类似的输出：

``````
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 1 from the spawned thread!
hi number 3 from the main thread!
hi number 2 from the spawned thread!
hi number 4 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
``````
The two threads continue alternating, but the main thread waits because of the call to handle.join() and does not end until the spawned thread is finished.

But let’s see what happens when we instead move handle.join() before the for loop in main, like this:

两个线程继续交替，但主线程因调用handle.join()而等待，直到生成的线程完成为止。

但是让我们看看当我们将 handle.join() 移到 main 中的 for 循环之前会发生什么，如下所示：


Filename: src/main.rs
``````
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    handle.join().unwrap();

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
}
``````
The main thread will wait for the spawned thread to finish and then run its for loop, so the output won’t be interleaved anymore, as shown here:
``````
hi number 1 from the spawned thread!
hi number 2 from the spawned thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 3 from the main thread!
hi number 4 from the main thread!
``````
Small details, such as where join is called, can affect whether or not your threads run at the same time.

### Using move Closures with Threads
We'll often use the move keyword with closures passed to thread::spawn because the closure will then take ownership of the values it uses from the environment, thus transferring ownership of those values from one thread to another. In the “Capturing References or Moving Ownership” section of Chapter 13, we discussed move in the context of closures. Now, we’ll concentrate more on the interaction between move and thread::spawn.

Notice in Listing 16-1 that the closure we pass to thread::spawn takes no arguments: we’re not using any data from the main thread in the spawned thread’s code. To use data from the main thread in the spawned thread, the spawned thread’s closure must capture the values it needs. Listing 16-3 shows an attempt to create a vector in the main thread and use it in the spawned thread. However, this won’t yet work, as you’ll see in a moment.

我们经常将 move 关键字与传递给 thread::spawn 的闭包一起使用，因为闭包随后将从环境中获取它使用的值的所有权，从而将这些值的所有权从一个线程转移到另一个线程。 在第 13 章的“捕获引用或移动所有权”部分中，我们讨论了闭包背景下的移动。 现在，我们将更多地关注 move 和 thread::spawn 之间的交互。

请注意，在清单 16-1 中，我们传递给 thread::spawn 的闭包不带任何参数：我们没有在生成线程的代码中使用来自主线程的任何数据。 要在生成的线程中使用来自主线程的数据，生成的线程的闭包必须捕获它所需的值。 清单 16-3 显示了在主线程中创建向量并在派生线程中使用它的尝试。 然而，这还行不通，稍后您就会看到。

Filename: src/main.rs
``````
This code does not compile!
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(|| {
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();
}
``````
Listing 16-3: Attempting to use a vector created by the main thread in another thread

The closure uses v, so it will capture v and make it part of the closure’s environment. Because thread::spawn runs this closure in a new thread, we should be able to access v inside that new thread. But when we compile this example, we get the following error:

``````
$ cargo run
   Compiling threads v0.1.0 (file:///projects/threads)
error[E0373]: closure may outlive the current function, but it borrows `v`, which is owned by the current function
 --> src/main.rs:6:32
  |
6 |     let handle = thread::spawn(|| {
  |                                ^^ may outlive borrowed value `v`
7 |         println!("Here's a vector: {:?}", v);
  |                                           - `v` is borrowed here
  |
note: function requires argument type to outlive `'static`
 --> src/main.rs:6:18
  |
6 |       let handle = thread::spawn(|| {
  |  __________________^
7 | |         println!("Here's a vector: {:?}", v);
8 | |     });
  | |______^
help: to force the closure to take ownership of `v` (and any other referenced variables), use the `move` keyword
  |
6 |     let handle = thread::spawn(move || {
  |                                ++++

For more information about this error, try `rustc --explain E0373`.
error: could not compile `threads` due to previous error
``````
Rust infers how to capture v, and because println! only needs a reference to v, the closure tries to borrow v. However, there’s a problem: Rust can’t tell how long the spawned thread will run, so it doesn’t know if the reference to v will always be valid.

Listing 16-4 provides a scenario that’s more likely to have a reference to v that won’t be valid:

Filename: src/main.rs
``````
This code does not compile!
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(|| {
        println!("Here's a vector: {:?}", v);
    });

    drop(v); // oh no!

    handle.join().unwrap();
}
``````
Listing 16-4: A thread with a closure that attempts to capture a reference to v from a main thread that drops v

If Rust allowed us to run this code, there’s a possibility the spawned thread would be immediately put in the background without running at all. The spawned thread has a reference to v inside, but the main thread immediately drops v, using the drop function we discussed in Chapter 15. Then, when the spawned thread starts to execute, v is no longer valid, so a reference to it is also invalid. Oh no!

如果 Rust 允许我们运行这段代码，那么生成的线程有可能会立即放入后台而不运行。 生成的线程内部有对 v 的引用，但主线程立即使用我们在第 15 章中讨论的 drop 函数删除 v。然后，当生成的线程开始执行时，v 不再有效，因此对它的引用是 也无效。 不好了！

To fix the compiler error in Listing 16-3, we can use the error message’s advice:
``````
help: to force the closure to take ownership of `v` (and any other referenced variables), use the `move` keyword
  |
6 |     let handle = thread::spawn(move || {
  |                                ++++
``````
By adding the move keyword before the closure, we force the closure to take ownership of the values it’s using rather than allowing Rust to infer that it should borrow the values. The modification to Listing 16-3 shown in Listing 16-5 will compile and run as we intend:

通过在闭包之前添加 move 关键字，我们强制闭包取得它正在使用的值的所有权，而不是让 Rust 推断它应该借用这些值。 清单 16-5 中所示的对清单 16-3 的修改将按照我们的预期进行编译和运行：

Filename: src/main.rs
``````
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(move || {
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();
}
``````
Listing 16-5: Using the move keyword to force a closure to take ownership of the values it uses

We might be tempted to try the same thing to fix the code in Listing 16-4 where the main thread called drop by using a move closure. However, this fix will not work because what Listing 16-4 is trying to do is disallowed for a different reason. If we added move to the closure, we would move v into the closure’s environment, and we could no longer call drop on it in the main thread. We would get this compiler error instead:

我们可能会尝试同样的方法来修复清单 16-4 中的代码，其中主线程通过使用移动闭包调用 drop。 然而，这个修复不会起作用，因为清单 16-4 试图做的事情由于不同的原因而被禁止。 如果我们将 move 添加到闭包中，我们会将 v 移动到闭包的环境中，并且我们无法再在主线程中对其调用 drop 。 我们会得到这个编译器错误：

``````
$ cargo run
   Compiling threads v0.1.0 (file:///projects/threads)
error[E0382]: use of moved value: `v`
  --> src/main.rs:10:10
   |
4  |     let v = vec![1, 2, 3];
   |         - move occurs because `v` has type `Vec<i32>`, which does not implement the `Copy` trait
5  |
6  |     let handle = thread::spawn(move || {
   |                                ------- value moved into closure here
7  |         println!("Here's a vector: {:?}", v);
   |                                           - variable moved due to use in closure
...
10 |     drop(v); // oh no!
   |          ^ value used here after move

For more information about this error, try `rustc --explain E0382`.
error: could not compile `threads` due to previous error
``````
Rust’s ownership rules have saved us again! We got an error from the code in Listing 16-3 because Rust was being conservative and only borrowing v for the thread, which meant the main thread could theoretically invalidate the spawned thread’s reference. By telling Rust to move ownership of v to the spawned thread, we’re guaranteeing Rust that the main thread won’t use v anymore. If we change Listing 16-4 in the same way, we’re then violating the ownership rules when we try to use v in the main thread. The move keyword overrides Rust’s conservative default of borrowing; it doesn’t let us violate the ownership rules.

Rust 的所有权规则再次拯救了我们！ 我们从清单 16-3 中的代码中得到了一个错误，因为 Rust 很保守，只为线程借用 v，这意味着主线程理论上可以使生成的线程的引用无效。 通过告诉 Rust 将 v 的所有权移至生成的线程，我们向 Rust 保证主线程不会再使用 v。 如果我们以同样的方式更改清单 16-4，那么当我们尝试在主线程中使用 v 时，我们就会违反所有权规则。 move 关键字会覆盖 Rust 保守的默认借用方式； 它不会让我们违反所有权规则。

With a basic understanding of threads and the thread API, let’s look at what we can do with threads.

## 16.2 Using Message Passing to Transfer Data Between Threads
One increasingly popular approach to ensuring safe concurrency is message passing, where threads or actors communicate by sending each other messages containing data. Here’s the idea in a slogan from the Go language documentation: “Do not communicate by sharing memory; instead, share memory by communicating.”

To accomplish message-sending concurrency, Rust's standard library provides an implementation of channels. A channel is a general programming concept by which data is sent from one thread to another.

You can imagine a channel in programming as being like a directional channel of water, such as a stream or a river. If you put something like a rubber duck into a river, it will travel downstream to the end of the waterway.

A channel has two halves: a transmitter and a receiver. The transmitter half is the upstream location where you put rubber ducks into the river, and the receiver half is where the rubber duck ends up downstream. One part of your code calls methods on the transmitter with the data you want to send, and another part checks the receiving end for arriving messages. A channel is said to be closed if either the transmitter or receiver half is dropped.

Here, we’ll work up to a program that has one thread to generate values and send them down a channel, and another thread that will receive the values and print them out. We’ll be sending simple values between threads using a channel to illustrate the feature. Once you’re familiar with the technique, you could use channels for any threads that need to communicate between each other, such as a chat system or a system where many threads perform parts of a calculation and send the parts to one thread that aggregates the results.

First, in Listing 16-6, we’ll create a channel but not do anything with it. Note that this won’t compile yet because Rust can’t tell what type of values we want to send over the channel.

确保安全并发的一种日益流行的方法是消息传递，其中线程或参与者通过向彼此发送包含数据的消息来进行通信。 这是 Go 语言文档中的一句口号的想法：“不要通过共享内存进行通信； 相反，通过沟通来共享记忆。”

为了实现消息发送并发，Rust 的标准库提供了通道的实现。 通道是一种通用编程概念，数据通过通道从一个线程发送到另一个线程。

您可以将编程中的通道想象为定向水道，例如小溪或河流。 如果你把橡皮鸭之类的东西放入河中，它会顺流而下，到达水道的尽头。

通道有两部分：发送器和接收器。 发射器的一半是您将橡皮鸭放入河中的上游位置，接收器的一半是橡皮鸭最终到达下游的位置。 代码的一部分使用要发送的数据调用发送器上的方法，另一部分检查接收端是否有到达的消息。 如果发送器或接收器一半掉线，则称通道已关闭。

在这里，我们将编写一个程序，该程序有一个线程生成值并将它们发送到通道，另一个线程将接收值并将它们打印出来。 我们将使用通道在线程之间发送简单的值来说明该功能。 一旦熟悉了这项技术，您就可以将通道用于任何需要相互通信的线程，例如聊天系统或许多线程执行部分计算并将这些部分发送到聚合计算的一个线程的系统。 结果。

首先，在清单 16-6 中，我们将创建一个通道，但不对其执行任何操作。 请注意，这还无法编译，因为 Rust 无法判断我们要通过通道发送什么类型的值。

Filename: src/main.rs
``````
This code does not compile!
use std::sync::mpsc;

fn main() {
    let (tx, rx) = mpsc::channel();
}
``````
Listing 16-6: Creating a channel and assigning the two halves to tx and rx

We create a new channel using the mpsc::channel function; mpsc stands for multiple producer, single consumer. In short, the way Rust’s standard library implements channels means a channel can have multiple sending ends that produce values but only one receiving end that consumes those values. Imagine multiple streams flowing together into one big river: everything sent down any of the streams will end up in one river at the end. We’ll start with a single producer for now, but we’ll add multiple producers when we get this example working.

The mpsc::channel function returns a tuple, the first element of which is the sending end--the transmitter--and the second element is the receiving end--the receiver. The abbreviations tx and rx are traditionally used in many fields for transmitter and receiver respectively, so we name our variables as such to indicate each end. We’re using a let statement with a pattern that destructures the tuples; we’ll discuss the use of patterns in let statements and destructuring in Chapter 18. For now, know that using a let statement this way is a convenient approach to extract the pieces of the tuple returned by mpsc::channel.

Let’s move the transmitting end into a spawned thread and have it send one string so the spawned thread is communicating with the main thread, as shown in Listing 16-7. This is like putting a rubber duck in the river upstream or sending a chat message from one thread to another.

我们使用 mpsc::channel 函数创建一个新通道； mpsc 代表多个生产者、单个消费者。 简而言之，Rust 标准库实现通道的方式意味着一个通道可以有多个产生值的发送端，但只有一个接收端消耗这些值。 想象一下，多条溪流汇聚成一条大河：任何一条溪流流下的所有东西最终都会流入一条河流。 我们现在将从单个生产者开始，但是当我们让这个示例运行时，我们将添加多个生产者。

mpsc::channel 函数返回一个元组，其中第一个元素是发送端——发送器——第二个元素是接收端——接收器。 缩写 tx 和 rx 传统上在许多领域中分别用于发送器和接收器，因此我们这样命名变量以指示每一端。 我们使用 let 语句和解构元组的模式； 我们将在第 18 章讨论 let 语句和解构中模式的使用。现在，我们知道以这种方式使用 let 语句是提取 mpsc::channel 返回的元组片段的便捷方法。

让我们将发送端移至衍生线程中，并让它发送一个字符串，以便衍生线程与主线程进行通信，如清单 16-7 所示。 这就像在上游河中放一只橡皮鸭或从一个线程向另一个线程发送聊天消息。


Filename: src/main.rs
``````
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
    });
}
``````
Listing 16-7: Moving tx to a spawned thread and sending “hi”

Again, we’re using thread::spawn to create a new thread and then using move to move tx into the closure so the spawned thread owns tx. The spawned thread needs to own the transmitter to be able to send messages through the channel. The transmitter has a send method that takes the value we want to send. The send method returns a Result<T, E> type, so if the receiver has already been dropped and there’s nowhere to send a value, the send operation will return an error. In this example, we’re calling unwrap to panic in case of an error. But in a real application, we would handle it properly: return to Chapter 9 to review strategies for proper error handling.

In Listing 16-8, we’ll get the value from the receiver in the main thread. This is like retrieving the rubber duck from the water at the end of the river or receiving a chat message.

同样，我们使用 thread::spawn 创建一个新线程，然后使用 move 将 tx 移动到闭包中，以便生成的线程拥有 tx。 生成的线程需要拥有发送器才能通过通道发送消息。 发送器有一个 send 方法，它接受我们想要发送的值。 send 方法返回 Result<T, E> 类型，因此如果接收方已被删除并且无处可发送值，则发送操作将返回错误。 在这个例子中，我们调用 unwrap 来避免发生错误。 但在实际的应用程序中，我们会正确处理它：返回第 9 章回顾正确的错误处理策略。

在清单 16-8 中，我们将从主线程中的接收器获取值。 这就像从河尾的水中取出橡皮鸭或收到聊天消息一样。

Filename: src/main.rs
``````
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
    });

    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
``````
Listing 16-8: Receiving the value “hi” in the main thread and printing it

The receiver has two useful methods: recv and try_recv. We’re using recv, short for receive, which will block the main thread’s execution and wait until a value is sent down the channel. Once a value is sent, recv will return it in a Result<T, E>. When the transmitter closes, recv will return an error to signal that no more values will be coming.

The try_recv method doesn’t block, but will instead return a Result<T, E> immediately: an Ok value holding a message if one is available and an Err value if there aren’t any messages this time. Using try_recv is useful if this thread has other work to do while waiting for messages: we could write a loop that calls try_recv every so often, handles a message if one is available, and otherwise does other work for a little while until checking again.

We’ve used recv in this example for simplicity; we don’t have any other work for the main thread to do other than wait for messages, so blocking the main thread is appropriate.

接收器有两个有用的方法：recv 和 try_recv。 我们使用recv（receive 的缩写），它将阻塞主线程的执行并等待，直到一个值沿着通道发送。 一旦发送了一个值，recv 就会在 Result<T, E> 中返回它。 当发送器关闭时，recv 将返回一个错误，表示不会有更多值到来。

try_recv 方法不会阻塞，而是立即返回一个 Result<T, E>：如果有消息可用，则返回一个包含消息的 Ok 值；如果这次没有任何消息，则返回一个 Err 值。 如果该线程在等待消息时有其他工作要做，则使用 try_recv 很有用：我们可以编写一个循环，经常调用 try_recv，如果消息可用则处理消息，否则执行其他工作一段时间，直到再次检查。

为了简单起见，我们在这个例子中使用了recv； 除了等待消息之外，主线程没有任何其他工作要做，因此阻塞主线程是合适的。

When we run the code in Listing 16-8, we’ll see the value printed from the main thread:
``````
Got: hi
``````
Perfect!

### Channels and Ownership Transference
The ownership rules play a vital role in message sending because they help you write safe, concurrent code. Preventing errors in concurrent programming is the advantage of thinking about ownership throughout your Rust programs. Let’s do an experiment to show how channels and ownership work together to prevent problems: we’ll try to use a val value in the spawned thread after we’ve sent it down the channel. Try compiling the code in Listing 16-9 to see why this code isn’t allowed:

所有权规则在消息发送中起着至关重要的作用，因为它们可以帮助您编写安全的并发代码。 防止并发编程中的错误是考虑整个 Rust 程序的所有权的优点。 让我们做一个实验来展示通道和所有权如何协同工作以防止出现问题：在将其发送到通道后，我们将尝试在生成的线程中使用 val 值。 尝试编译清单 16-9 中的代码，看看为什么不允许使用此代码：

Filename: src/main.rs
``````
This code does not compile!
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
        println!("val is {}", val);
    });

    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
``````
Listing 16-9: Attempting to use val after we’ve sent it down the channel

Here, we try to print val after we’ve sent it down the channel via tx.send. Allowing this would be a bad idea: once the value has been sent to another thread, that thread could modify or drop it before we try to use the value again. Potentially, the other thread’s modifications could cause errors or unexpected results due to inconsistent or nonexistent data. However, Rust gives us an error if we try to compile the code in Listing 16-9:

在这里，我们尝试在通过 tx.send 将 val 发送到通道后打印它。 允许这样做将是一个坏主意：一旦该值被发送到另一个线程，该线程就可以在我们尝试再次使用该值之前修改或删除它。 由于数据不一致或不存在，其他线程的修改可能会导致错误或意外结果。 然而，如果我们尝试编译清单 16-9 中的代码，Rust 会给我们一个错误：

``````
$ cargo run
   Compiling message-passing v0.1.0 (file:///projects/message-passing)
error[E0382]: borrow of moved value: `val`
  --> src/main.rs:10:31
   |
8  |         let val = String::from("hi");
   |             --- move occurs because `val` has type `String`, which does not implement the `Copy` trait
9  |         tx.send(val).unwrap();
   |                 --- value moved here
10 |         println!("val is {}", val);
   |                               ^^^ value borrowed here after move
   |
   = note: this error originates in the macro `$crate::format_args_nl` which comes from the expansion of the macro `println` (in Nightly builds, run with -Z macro-backtrace for more info)

For more information about this error, try `rustc --explain E0382`.
error: could not compile `message-passing` due to previous error
``````
Our concurrency mistake has caused a compile time error. The send function takes ownership of its parameter, and when the value is moved, the receiver takes ownership of it. This stops us from accidentally using the value again after sending it; the ownership system checks that everything is okay.

我们的并发错误导致了编译时错误。 发送函数取得其参数的所有权，当值移动时，接收者取得它的所有权。 这可以防止我们在发送后意外地再次使用该值； 所有权系统检查一切是否正常。

### Sending Multiple Values and Seeing the Receiver Waiting
The code in Listing 16-8 compiled and ran, but it didn’t clearly show us that two separate threads were talking to each other over the channel. In Listing 16-10 we’ve made some modifications that will prove the code in Listing 16-8 is running concurrently: the spawned thread will now send multiple messages and pause for a second between each message.

清单 16-8 中的代码已编译并运行，但它没有清楚地向我们显示两个单独的线程正在通过通道相互通信。 在清单 16-10 中，我们做了一些修改，以证明清单 16-8 中的代码是并发运行的：生成的线程现在将发送多条消息，并在每条消息之间暂停一秒钟。

Filename: src/main.rs
``````
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];

        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    for received in rx {
        println!("Got: {}", received);
    }
}
``````
Listing 16-10: Sending multiple messages and pausing between each

This time, the spawned thread has a vector of strings that we want to send to the main thread. We iterate over them, sending each individually, and pause between each by calling the thread::sleep function with a Duration value of 1 second.

In the main thread, we’re not calling the recv function explicitly anymore: instead, we’re treating rx as an iterator. For each value received, we’re printing it. When the channel is closed, iteration will end.

When running the code in Listing 16-10, you should see the following output with a 1-second pause in between each line:

这次，生成的线程有一个我们想要发送到主线程的字符串向量。 我们迭代它们，单独发送每个，并通过调用 thread::sleep 函数（持续时间值为 1 秒）在每个之间暂停。

在主线程中，我们不再显式调用 recv 函数：相反，我们将 rx 视为迭代器。 对于收到的每个值，我们都会打印它。 当通道关闭时，迭代将结束。

运行清单 16-10 中的代码时，您应该看到以下输出，每行之间有 1 秒的暂停：

``````
Got: hi
Got: from
Got: the
Got: thread
``````
Because we don’t have any code that pauses or delays in the for loop in the main thread, we can tell that the main thread is waiting to receive values from the spawned thread.

因为我们在主线程的 for 循环中没有任何暂停或延迟的代码，所以我们可以知道主线程正在等待从生成的线程接收值。

### Creating Multiple Producers by Cloning the Transmitter
Earlier we mentioned that mpsc was an acronym for multiple producer, single consumer. Let’s put mpsc to use and expand the code in Listing 16-10 to create multiple threads that all send values to the same receiver. We can do so by cloning the transmitter, as shown in Listing 16-11:

前面我们提到 mpsc 是多个生产者、单个消费者的缩写。 让我们使用 mpsc 并扩展清单 16-10 中的代码来创建多个线程，这些线程都将值发送到同一个接收器。 我们可以通过克隆发射器来实现这一点，如清单 16-11 所示：

Filename: src/main.rs
``````
    // --snip--

    let (tx, rx) = mpsc::channel();

    let tx1 = tx.clone();
    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];

        for val in vals {
            tx1.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    thread::spawn(move || {
        let vals = vec![
            String::from("more"),
            String::from("messages"),
            String::from("for"),
            String::from("you"),
        ];

        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    for received in rx {
        println!("Got: {}", received);
    }

    // --snip--
``````
Listing 16-11: Sending multiple messages from multiple producers

This time, before we create the first spawned thread, we call clone on the transmitter. This will give us a new transmitter we can pass to the first spawned thread. We pass the original transmitter to a second spawned thread. This gives us two threads, each sending different messages to the one receiver.

这次，在创建第一个衍生线程之前，我们在发送器上调用克隆。 这将为我们提供一个新的发送器，我们可以将其传递给第一个生成的线程。 我们将原始发送器传递给第二个生成的线程。 这给了我们两个线程，每个线程向一个接收者发送不同的消息。

When you run the code, your output should look something like this:
``````
Got: hi
Got: more
Got: from
Got: messages
Got: for
Got: the
Got: thread
Got: you
``````
You might see the values in another order, depending on your system. This is what makes concurrency interesting as well as difficult. If you experiment with thread::sleep, giving it various values in the different threads, each run will be more nondeterministic and create different output each time.

您可能会看到其他顺序的值，具体取决于您的系统。 这就是并发既有趣又困难的原因。 如果您尝试使用 thread::sleep，在不同的线程中为其赋予不同的值，则每次运行将更加不确定，并且每次都会创建不同的输出。

Now that we’ve looked at how channels work, let’s look at a different method of concurrency.
## 16.3 Shared-State Concurrency

Message passing is a fine way of handling concurrency, but it’s not the only one. Another method would be for multiple threads to access the same shared data. Consider this part of the slogan from the Go language documentation again: “do not communicate by sharing memory.”

What would communicating by sharing memory look like? In addition, why would message-passing enthusiasts caution not to use memory sharing?

In a way, channels in any programming language are similar to single ownership, because once you transfer a value down a channel, you should no longer use that value. Shared memory concurrency is like multiple ownership: multiple threads can access the same memory location at the same time. As you saw in Chapter 15, where smart pointers made multiple ownership possible, multiple ownership can add complexity because these different owners need managing. Rust’s type system and ownership rules greatly assist in getting this management correct. For an example, let’s look at mutexes, one of the more common concurrency primitives for shared memory.

Using Mutexes to Allow Access to Data from One Thread at a Time
Mutex is an abbreviation for mutual exclusion, as in, a mutex allows only one thread to access some data at any given time. To access the data in a mutex, a thread must first signal that it wants access by asking to acquire the mutex’s lock. The lock is a data structure that is part of the mutex that keeps track of who currently has exclusive access to the data. Therefore, the mutex is described as guarding the data it holds via the locking system.

Mutexes have a reputation for being difficult to use because you have to remember two rules:

消息传递是处理并发的一种很好的方法，但它不是唯一的方法。 另一种方法是让多个线程访问相同的共享数据。 再考虑一下 Go 语言文档中的这句口号：“不要通过共享内存进行通信。”

通过共享内存进行通信会是什么样子？ 此外，为什么消息传递爱好者会警告不要使用内存共享？

在某种程度上，任何编程语言中的通道都类似于单一所有权，因为一旦您将值转移到通道中，您就不应该再使用该值。 共享内存并发就像多重所有权：多个线程可以同时访问同一内存位置。 正如您在第 15 章中看到的，智能指针使多重所有权成为可能，多重所有权会增加复杂性，因为这些不同的所有者需要管理。 Rust 的类型系统和所有权规则极大地有助于正确管理。 举个例子，让我们看一下互斥体，它是共享内存的更常见的并发原语之一。

使用互斥体允许一次从一个线程访问数据
互斥体是互斥体的缩写，互斥体在任何给定时间只允许一个线程访问某些数据。 要访问互斥体中的数据，线程必须首先通过请求获取互斥体的锁来表明它想要访问。 锁是一种数据结构，是互斥体的一部分，用于跟踪当前谁对数据具有独占访问权。 因此，互斥体被描述为通过锁定系统保护其保存的数据。

互斥体因难以使用而闻名，因为您必须记住两条规则：

+ You must attempt to acquire the lock before using the data.
+ When you’re done with the data that the mutex guards, you must unlock the data so other threads can acquire the lock.

+ 在使用数据之前，您必须尝试获取锁。
+ 当您完成互斥锁保护的数据后，您必须解锁数据，以便其他线程可以获得锁。

For a real-world metaphor for a mutex, imagine a panel discussion at a conference with only one microphone. Before a panelist can speak, they have to ask or signal that they want to use the microphone. When they get the microphone, they can talk for as long as they want to and then hand the microphone to the next panelist who requests to speak. If a panelist forgets to hand the microphone off when they’re finished with it, no one else is able to speak. If management of the shared microphone goes wrong, the panel won’t work as planned!

Management of mutexes can be incredibly tricky to get right, which is why so many people are enthusiastic about channels. However, thanks to Rust’s type system and ownership rules, you can’t get locking and unlocking wrong.

对于互斥锁的现实世界比喻，想象一下在只有一个麦克风的会议上进行小组讨论。 在小组成员发言之前，他们必须询问或示意他们想要使用麦克风。 当他们拿到麦克风时，他们可以随意发言，然后将麦克风交给下一位要求发言的小组成员。 如果小组成员在结束后忘记交出麦克风，那么其他人就无法发言。 如果共享麦克风的管理出现问题，面板将无法按计划工作！

正确管理互斥体可能非常棘手，这就是为什么这么多人对通道充满热情的原因。 然而，由于 Rust 的类型系统和所有权规则，您不会出现错误的锁定和解锁。

### The API of Mutex<T>
As an example of how to use a mutex, let’s start by using a mutex in a single-threaded context, as shown in Listing 16-12:

Filename: src/main.rs
``````
use std::sync::Mutex;

fn main() {
    let m = Mutex::new(5);

    {
        let mut num = m.lock().unwrap();
        *num = 6;
    }

    println!("m = {:?}", m);
}
``````
Listing 16-12: Exploring the API of Mutex<T> in a single-threaded context for simplicity

As with many types, we create a Mutex<T> using the associated function new. To access the data inside the mutex, we use the lock method to acquire the lock. This call will block the current thread so it can’t do any work until it’s our turn to have the lock.

The call to lock would fail if another thread holding the lock panicked. In that case, no one would ever be able to get the lock, so we’ve chosen to unwrap and have this thread panic if we’re in that situation.

After we’ve acquired the lock, we can treat the return value, named num in this case, as a mutable reference to the data inside. The type system ensures that we acquire a lock before using the value in m. The type of m is Mutex<i32>, not i32, so we must call lock to be able to use the i32 value. We can’t forget; the type system won’t let us access the inner i32 otherwise.

As you might suspect, Mutex<T> is a smart pointer. More accurately, the call to lock returns a smart pointer called MutexGuard, wrapped in a LockResult that we handled with the call to unwrap. The MutexGuard smart pointer implements Deref to point at our inner data; the smart pointer also has a Drop implementation that releases the lock automatically when a MutexGuard goes out of scope, which happens at the end of the inner scope. As a result, we don’t risk forgetting to release the lock and blocking the mutex from being used by other threads, because the lock release happens automatically.

After dropping the lock, we can print the mutex value and see that we were able to change the inner i32 to 6.

与许多类型一样，我们使用关联的函数 new 创建一个 Mutex<T>。 为了访问互斥锁内的数据，我们使用lock方法来获取锁。 此调用将阻塞当前线程，因此在轮到我们获得锁之前它无法执行任何工作。

如果持有锁的另一个线程发生恐慌，则对 lock 的调用将会失败。 在这种情况下，没有人能够获得锁，因此如果我们处于这种情况，我们选择解包并使该线程发生恐慌。

获取锁后，我们可以将返回值（在本例中名为 num）视为对内部数据的可变引用。 类型系统确保我们在使用 m 中的值之前获取锁。 m 的类型是 Mutex<i32>，而不是 i32，因此我们必须调用 lock 才能使用 i32 值。 我们不能忘记； 否则类型系统不会让我们访问内部 i32。

正如您可能怀疑的那样，Mutex<T> 是一个智能指针。 更准确地说，对 lock 的调用返回一个名为 MutexGuard 的智能指针，该指针包装在我们通过调用 unwrap 处理的 LockResult 中。 MutexGuard 智能指针实现 Deref 来指向我们的内部数据； 智能指针还有一个 Drop 实现，当 MutexGuard 超出范围（发生在内部范围的末尾）时，该实现会自动释放锁。 因此，我们不会冒忘记释放锁并阻止其他线程使用互斥体的风险，因为锁释放会自动发生。

释放锁后，我们可以打印互斥锁值，并看到我们能够将内部 i32 更改为 6。

### Sharing a Mutex<T> Between Multiple Threads
Now, let’s try to share a value between multiple threads using Mutex<T>. We’ll spin up 10 threads and have them each increment a counter value by 1, so the counter goes from 0 to 10. The next example in Listing 16-13 will have a compiler error, and we’ll use that error to learn more about using Mutex<T> and how Rust helps us use it correctly.

现在，让我们尝试使用 Mutex<T> 在多个线程之间共享值。 我们将启动 10 个线程，并让它们各自将计数器值加 1，因此计数器从 0 变为 10。清单 16-13 中的下一个示例将出现编译器错误，我们将使用该错误来学习 更多关于使用 Mutex<T> 以及 Rust 如何帮助我们正确使用它的信息。

Filename: src/main.rs
``````
This code does not compile!
use std::sync::Mutex;
use std::thread;

fn main() {
    let counter = Mutex::new(0);
    let mut handles = vec![];

    for _ in 0..10 {
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
``````
Listing 16-13: Ten threads each increment a counter guarded by a Mutex<T>

We create a counter variable to hold an i32 inside a Mutex<T>, as we did in Listing 16-12. Next, we create 10 threads by iterating over a range of numbers. We use thread::spawn and give all the threads the same closure: one that moves the counter into the thread, acquires a lock on the Mutex<T> by calling the lock method, and then adds 1 to the value in the mutex. When a thread finishes running its closure, num will go out of scope and release the lock so another thread can acquire it.

In the main thread, we collect all the join handles. Then, as we did in Listing 16-2, we call join on each handle to make sure all the threads finish. At that point, the main thread will acquire the lock and print the result of this program.

我们创建一个计数器变量来将 i32 保存在 Mutex<T> 中，如清单 16-12 中所做的那样。 接下来，我们通过迭代一系列数字来创建 10 个线程。 我们使用 thread::spawn 并为所有线程提供相同的闭包：将计数器移动到线程中，通过调用 lock 方法获取 Mutex<T> 上的锁，然后将互斥体中的值加 1。 当一个线程完成运行其闭包时， num 将超出范围并释放锁，以便另一个线程可以获取它。

在主线程中，我们收集所有连接句柄。 然后，正如我们在清单 16-2 中所做的那样，我们对每个句柄调用 join 以确保所有线程都完成。 此时，主线程将获取锁并打印该程序的结果。

We hinted that this example wouldn’t compile. Now let’s find out why!
``````
$ cargo run
   Compiling shared-state v0.1.0 (file:///projects/shared-state)
error[E0382]: use of moved value: `counter`
  --> src/main.rs:9:36
   |
5  |     let counter = Mutex::new(0);
   |         ------- move occurs because `counter` has type `Mutex<i32>`, which does not implement the `Copy` trait
...
9  |         let handle = thread::spawn(move || {
   |                                    ^^^^^^^ value moved into closure here, in previous iteration of loop
10 |             let mut num = counter.lock().unwrap();
   |                           ------- use occurs due to use in closure

For more information about this error, try `rustc --explain E0382`.
error: could not compile `shared-state` due to previous error
``````
The error message states that the counter value was moved in the previous iteration of the loop. Rust is telling us that we can’t move the ownership of lock counter into multiple threads. Let’s fix the compiler error with a multiple-ownership method we discussed in Chapter 15.

错误消息指出计数器值在循环的上一次迭代中已移动。 Rust 告诉我们，我们不能将锁计数器的所有权转移到多个线程中。 让我们用第 15 章讨论的多重所有权方法来修复编译器错误。

### Multiple Ownership with Multiple Threads
In Chapter 15, we gave a value multiple owners by using the smart pointer Rc<T> to create a reference counted value. Let’s do the same here and see what happens. We’ll wrap the Mutex<T> in Rc<T> in Listing 16-14 and clone the Rc<T> before moving ownership to the thread.

在第 15 章中，我们通过使用智能指针 Rc<T> 创建引用计数值来赋予一个值多个所有者。 让我们在这里做同样的事情，看看会发生什么。 我们将在清单 16-14 中将 Mutex<T> 包装在 Rc<T> 中，并在将所有权移至线程之前克隆 Rc<T>。

Filename: src/main.rs
``````
This code does not compile!
use std::rc::Rc;
use std::sync::Mutex;
use std::thread;

fn main() {
    let counter = Rc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Rc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
``````
Listing 16-14: Attempting to use Rc<T> to allow multiple threads to own the Mutex<T>

Once again, we compile and get... different errors! The compiler is teaching us a lot.
``````
$ cargo run
   Compiling shared-state v0.1.0 (file:///projects/shared-state)
error[E0277]: `Rc<Mutex<i32>>` cannot be sent between threads safely
  --> src/main.rs:11:36
   |
11 |           let handle = thread::spawn(move || {
   |                        ------------- ^------
   |                        |             |
   |  ______________________|_____________within this `[closure@src/main.rs:11:36: 11:43]`
   | |                      |
   | |                      required by a bound introduced by this call
12 | |             let mut num = counter.lock().unwrap();
13 | |
14 | |             *num += 1;
15 | |         });
   | |_________^ `Rc<Mutex<i32>>` cannot be sent between threads safely
   |
   = help: within `[closure@src/main.rs:11:36: 11:43]`, the trait `Send` is not implemented for `Rc<Mutex<i32>>`
note: required because it's used within this closure
  --> src/main.rs:11:36
   |
11 |         let handle = thread::spawn(move || {
   |                                    ^^^^^^^
note: required by a bound in `spawn`
  --> /rustc/d5a82bbd26e1ad8b7401f6a718a9c57c96905483/library/std/src/thread/mod.rs:704:8
   |
   = note: required by this bound in `spawn`

For more information about this error, try `rustc --explain E0277`.
error: could not compile `shared-state` due to previous error
``````
Wow, that error message is very wordy! Here’s the important part to focus on: `Rc<Mutex<i32>>` cannot be sent between threads safely. The compiler is also telling us the reason why: the trait `Send` is not implemented for `Rc<Mutex<i32>>` . We’ll talk about Send in the next section: it’s one of the traits that ensures the types we use with threads are meant for use in concurrent situations.

Unfortunately, Rc<T> is not safe to share across threads. When Rc<T> manages the reference count, it adds to the count for each call to clone and subtracts from the count when each clone is dropped. But it doesn’t use any concurrency primitives to make sure that changes to the count can’t be interrupted by another thread. This could lead to wrong counts—subtle bugs that could in turn lead to memory leaks or a value being dropped before we’re done with it. What we need is a type exactly like Rc<T> but one that makes changes to the reference count in a thread-safe way.

哇，这个错误信息太啰嗦了！ 这是需要关注的重要部分：`Rc<Mutex<i32>>` 无法在线程之间安全地发送。 编译器还告诉我们原因：“Rc<Mutex<i32>>”没有实现“Send”特征。 我们将在下一节中讨论发送：它是确保我们在线程中使用的类型适用于并发情况的特征之一。

不幸的是，Rc<T> 不能安全地跨线程共享。 当 Rc<T> 管理引用计数时，它会添加每次调用克隆的计数，并在删除每个克隆时从计数中减去。 但它不使用任何并发原语来确保计数的更改不会被另一个线程中断。 这可能会导致错误的计数 - 微妙的错误可能会导致内存泄漏或在我们完成之前删除某个值。 我们需要的是一种与 Rc<T> 完全相同的类型，但它以线程安全的方式更改引用计数。

### Atomic Reference Counting with Arc<T>
Fortunately, Arc<T> is a type like Rc<T> that is safe to use in concurrent situations. The a stands for atomic, meaning it’s an atomically reference counted type. Atomics are an additional kind of concurrency primitive that we won’t cover in detail here: see the standard library documentation for std::sync::atomic for more details. At this point, you just need to know that atomics work like primitive types but are safe to share across threads.

You might then wonder why all primitive types aren’t atomic and why standard library types aren’t implemented to use Arc<T> by default. The reason is that thread safety comes with a performance penalty that you only want to pay when you really need to. If you’re just performing operations on values within a single thread, your code can run faster if it doesn’t have to enforce the guarantees atomics provide.

Let’s return to our example: Arc<T> and Rc<T> have the same API, so we fix our program by changing the use line, the call to new, and the call to clone. The code in Listing 16-15 will finally compile and run:

幸运的是，Arc<T> 是一种类似于 Rc<T> 的类型，可以在并发情况下安全使用。 a 代表原子，这意味着它是原子引用计数类型。 原子是另一种并发原语，我们在这里不会详细介绍：有关更多详细信息，请参阅 std::sync::atomic 的标准库文档。 此时，您只需要知道原子像原始类型一样工作，但可以安全地跨线程共享。

然后您可能想知道为什么所有基元类型都不是原子的，以及为什么标准库类型没有实现默认使用 Arc<T>。 原因是线程安全会带来性能损失，您只有在真正需要时才愿意付出代价。 如果您只是在单个线程中对值执行操作，那么如果不必强制执行原子提供的保证，您的代码可以运行得更快。

让我们回到我们的示例：Arc<T> 和 Rc<T> 具有相同的 API，因此我们通过更改 use 行、对 new 的调用以及对 Clone 的调用来修复程序。 清单16-15中的代码最终将编译并运行：

Filename: src/main.rs
``````
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
``````
Listing 16-15: Using an Arc<T> to wrap the Mutex<T> to be able to share ownership across multiple threads

This code will print the following:
``````
Result: 10
``````
We did it! We counted from 0 to 10, which may not seem very impressive, but it did teach us a lot about Mutex<T> and thread safety. You could also use this program’s structure to do more complicated operations than just incrementing a counter. Using this strategy, you can divide a calculation into independent parts, split those parts across threads, and then use a Mutex<T> to have each thread update the final result with its part.

Note that if you are doing simple numerical operations, there are types simpler than Mutex<T> types provided by the std::sync::atomic module of the standard library. These types provide safe, concurrent, atomic access to primitive types. We chose to use Mutex<T> with a primitive type for this example so we could concentrate on how Mutex<T> works.

我们做到了！ 我们从 0 数到 10，这可能看起来不太令人印象深刻，但它确实教会了我们很多关于 Mutex<T> 和线程安全的知识。 您还可以使用该程序的结构来执行更复杂的操作，而不仅仅是增加计数器。 使用此策略，您可以将计算划分为独立的部分，跨线程拆分这些部分，然后使用 Mutex<T> 让每个线程用其部分更新最终结果。

请注意，如果您正在进行简单的数值运算，则标准库的 std::sync::atomic 模块提供的类型比 Mutex<T> 类型更简单。 这些类型提供对原始类型的安全、并发、原子访问。 在本示例中，我们选择将 Mutex<T> 与原始类型一起使用，以便我们可以专注于 Mutex<T> 的工作原理。

### Similarities Between RefCell<T>/Rc<T> and Mutex<T>/Arc<T>
You might have noticed that counter is immutable but we could get a mutable reference to the value inside it; this means Mutex<T> provides interior mutability, as the Cell family does. In the same way we used RefCell<T> in Chapter 15 to allow us to mutate contents inside an Rc<T>, we use Mutex<T> to mutate contents inside an Arc<T>.

Another detail to note is that Rust can’t protect you from all kinds of logic errors when you use Mutex<T>. Recall in Chapter 15 that using Rc<T> came with the risk of creating reference cycles, where two Rc<T> values refer to each other, causing memory leaks. Similarly, Mutex<T> comes with the risk of creating deadlocks. These occur when an operation needs to lock two resources and two threads have each acquired one of the locks, causing them to wait for each other forever. If you’re interested in deadlocks, try creating a Rust program that has a deadlock; then research deadlock mitigation strategies for mutexes in any language and have a go at implementing them in Rust. The standard library API documentation for Mutex<T> and MutexGuard offers useful information.

We’ll round out this chapter by talking about the Send and Sync traits and how we can use them with custom types.

您可能已经注意到 counter 是不可变的，但我们可以获得对其内部值的可变引用； 这意味着 Mutex<T> 提供内部可变性，就像 Cell 系列一样。 就像我们在第 15 章中使用 RefCell<T> 来允许我们改变 Rc<T> 内的内容一样，我们使用 Mutex<T> 来改变 Arc<T> 内的内容。

另一个需要注意的细节是，当您使用 Mutex<T> 时，Rust 无法保护您免受各种逻辑错误的影响。 回想一下第 15 章，使用 Rc<T> 会带来创建引用循环的风险，其中两个 Rc<T> 值相互引用，从而导致内存泄漏。 同样，Mutex<T> 也存在产生死锁的风险。 当一个操作需要锁定两个资源并且两个线程各自获取其中一个锁，导致它们永远等待对方时，就会发生这种情况。 如果您对死锁感兴趣，请尝试创建一个存在死锁的 Rust 程序； 然后研究任何语言中互斥体的死锁缓解策略，并尝试在 Rust 中实现它们。 Mutex<T> 和 MutexGuard 的标准库 API 文档提供了有用的信息。

我们将通过讨论发送和同步特征以及如何将它们与自定义类型一起使用来结束本章。

## 16.4 Extendsible Concurrency with the Sync and Send Traints

Interestingly, the Rust language has very few concurrency features. Almost every concurrency feature we’ve talked about so far in this chapter has been part of the standard library, not the language. Your options for handling concurrency are not limited to the language or the standard library; you can write your own concurrency features or use those written by others.

However, two concurrency concepts are embedded in the language: the std::marker traits Sync and Send.

有趣的是，Rust 语言的并发特性非常少。 到目前为止，我们在本章中讨论的几乎所有并发特性都是标准库的一部分，而不是语言的一部分。 您处理并发的选项不仅限于语言或标准库； 您可以编写自己的并发功能或使用其他人编写的并发功能。

然而，该语言中嵌入了两个并发概念：std::marker 特性 Sync 和 Send。

## Allowing Transference of Ownership Between Threads with Send
The Send marker trait indicates that ownership of values of the type implementing Send can be transferred between threads. Almost every Rust type is Send, but there are some exceptions, including Rc<T>: this cannot be Send because if you cloned an Rc<T> value and tried to transfer ownership of the clone to another thread, both threads might update the reference count at the same time. For this reason, Rc<T> is implemented for use in single-threaded situations where you don’t want to pay the thread-safe performance penalty.

Therefore, Rust’s type system and trait bounds ensure that you can never accidentally send an Rc<T> value across threads unsafely. When we tried to do this in Listing 16-14, we got the error the trait Send is not implemented for Rc<Mutex<i32>>. When we switched to Arc<T>, which is Send, the code compiled.

Any type composed entirely of Send types is automatically marked as Send as well. Almost all primitive types are Send, aside from raw pointers, which we’ll discuss in Chapter 19.

Send 标记特征指示实现 Send 的类型的值的所有权可以在线程之间转移。 几乎所有 Rust 类型都是 Send，但也有一些例外，包括 Rc<T>：这不能是 Send，因为如果您克隆了 Rc<T> 值并尝试将克隆的所有权转移到另一个线程，则两个线程都可能会更新 同时进行引用计数。 因此，Rc<T> 被实现用于您不想付出线程安全性能损失的单线程情况。

因此，Rust 的类型系统和特征边界确保您永远不会意外地不安全地跨线程发送 Rc<T> 值。 当我们尝试在清单 16-14 中执行此操作时，我们得到了错误：Rc<Mutex<i32>> 的特征 Send 未实现。 当我们切换到 Arc<T>（即 Send）时，代码就会编译。

任何完全由发送类型组成的类型也会自动标记为发送。 除了我们将在第 19 章讨论的原始指针之外，几乎所有原始类型都是 Send 的。

## Allowing Access from Multiple Threads with Sync
The Sync marker trait indicates that it is safe for the type implementing Sync to be referenced from multiple threads. In other words, any type T is Sync if &T (an immutable reference to T) is Send, meaning the reference can be sent safely to another thread. Similar to Send, primitive types are Sync, and types composed entirely of types that are Sync are also Sync.

The smart pointer Rc<T> is also not Sync for the same reasons that it’s not Send. The RefCell<T> type (which we talked about in Chapter 15) and the family of related Cell<T> types are not Sync. The implementation of borrow checking that RefCell<T> does at runtime is not thread-safe. The smart pointer Mutex<T> is Sync and can be used to share access with multiple threads as you saw in the “Sharing a Mutex<T> Between Multiple Threads” section.

同步标记特征表明从多个线程引用实现同步的类型是安全的。 换句话说，如果 &T（对 T 的不可变引用）是 Send，则任何类型 T 都是 Sync，这意味着该引用可以安全地发送到另一个线程。 与 Send 类似，原始类型是 Sync，完全由 Sync 类型组成的类型也是 Sync。

智能指针 Rc<T> 也不是同步的，其原因与它不是发送的原因相同。 RefCell<T> 类型（我们在第 15 章中讨论过）和相关的 Cell<T> 类型系列不是同步的。 RefCell<T> 在运行时执行的借用检查的实现不是线程安全的。 智能指针 Mutex<T> 是同步的，可用于与多个线程共享访问，如您在“在多个线程之间共享 Mutex<T>”部分中看到的那样。

## Implementing Send and Sync Manually Is Unsafe
Because types that are made up of Send and Sync traits are automatically also Send and Sync, we don’t have to implement those traits manually. As marker traits, they don’t even have any methods to implement. They’re just useful for enforcing invariants related to concurrency.

Manually implementing these traits involves implementing unsafe Rust code. We’ll talk about using unsafe Rust code in Chapter 19; for now, the important information is that building new concurrent types not made up of Send and Sync parts requires careful thought to uphold the safety guarantees. “The Rustonomicon” has more information about these guarantees and how to uphold them.

因为由 Send 和 Sync 特征组成的类型也会自动发送和同步，所以我们不必手动实现这些特征。 作为标记特征，它们甚至没有任何方法可以实现。 它们只是用于强制执行与并发相关的不变量。

手动实现这些特征涉及实现不安全的 Rust 代码。 我们将在第 19 章讨论使用不安全的 Rust 代码； 目前，重要的信息是构建不由发送和同步部分组成的新并发类型需要仔细考虑以维护安全保证。 《The Rustonomicon》提供了有关这些保证以及如何维护这些保证的更多信息。

## 16.5 Summary
This isn’t the last you’ll see of concurrency in this book: the project in Chapter 20 will use the concepts in this chapter in a more realistic situation than the smaller examples discussed here.

As mentioned earlier, because very little of how Rust handles concurrency is part of the language, many concurrency solutions are implemented as crates. These evolve more quickly than the standard library, so be sure to search online for the current, state-of-the-art crates to use in multithreaded situations.

The Rust standard library provides channels for message passing and smart pointer types, such as Mutex<T> and Arc<T>, that are safe to use in concurrent contexts. The type system and the borrow checker ensure that the code using these solutions won’t end up with data races or invalid references. Once you get your code to compile, you can rest assured that it will happily run on multiple threads without the kinds of hard-to-track-down bugs common in other languages. Concurrent programming is no longer a concept to be afraid of: go forth and make your programs concurrent, fearlessly!

Next, we’ll talk about idiomatic ways to model problems and structure solutions as your Rust programs get bigger. In addition, we’ll discuss how Rust’s idioms relate to those you might be familiar with from object-oriented programming.

这并不是你在本书中最后一次看到并发：第 20 章中的项目将在比这里讨论的较小示例更现实的情况下使用本章中的概念。

如前所述，由于 Rust 处理并发的方式很少是语言的一部分，因此许多并发解决方案都是以 crate 的形式实现的。 它们的发展速度比标准库更快，因此请务必在线搜索当前最先进的 crate，以便在多线程情况下使用。

Rust 标准库提供了消息传递和智能指针类型的通道，例如 Mutex<T> 和 Arc<T>，它们可以在并发上下文中安全使用。 类型系统和借用检查器确保使用这些解决方案的代码不会出现数据争用或无效引用。 一旦你编译了你的代码，你就可以放心，它会很高兴地在多个线程上运行，而不会出现其他语言中常见的难以追踪的错误。 并发编程不再是一个令人害怕的概念：勇敢地让你的程序并发！

接下来，我们将讨论随着 Rust 程序变得越来越大，建模问题和构建解决方案的惯用方法。 此外，我们还将讨论 Rust 的习惯用法与您可能熟悉的面向对象编程的习惯用法之间的关系。