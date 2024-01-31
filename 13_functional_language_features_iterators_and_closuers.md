# 13 函数式编程:迭代和闭包
Rust’s design has taken inspiration from many existing languages and techniques, and one significant influence is functional programming. Programming in a functional style often includes using functions as values by passing them in arguments, returning them from other functions, assigning them to variables for later execution, and so forth.

In this chapter, we won’t debate the issue of what functional programming is or isn’t but will instead discuss some features of Rust that are similar to features in many languages often referred to as functional.



More specifically, we’ll cover:

+ Closures, a function-like construct you can store in a variable
+ Iterators, a way of processing a series of elements
+ How to use closures and iterators to improve the I/O project in Chapter 12
+ The performance of closures and iterators (Spoiler alert: they’re faster than you might think!)

We’ve already covered some other Rust features, such as pattern matching and enums, that are also influenced by the functional style. Because mastering closures and iterators is an important part of writing idiomatic, fast Rust code, we’ll devote this entire chapter to them.

Rust 的设计从许多现有的语言和技术中汲取了灵感，其中一个重要的影响就是函数式编程。 函数式编程通常包括通过将函数作为值传递给参数、从其他函数返回它们、将它们分配给变量以供以后执行等等。

在本章中，我们不会争论函数式编程是什么或不是什么的问题，而是讨论 Rust 的一些特性，这些特性与许多语言中通常被称为函数式的特性相似。

更具体地说，我们将介绍：

+ 闭包，一种可以存储在变量中的类似函数的结构
+ 迭代器，一种处理一系列元素的方法
+ 第12章如何使用闭包和迭代器改进I/O项目
+ 闭包和迭代器的性能（剧透警告：它们比你想象的要快！）

我们已经介绍了一些其他 Rust 功能，例如模式匹配和枚举，它们也受到函数式风格的影响。 因为掌握闭包和迭代器是编写惯用的、快速的 Rust 代码的重要组成部分，所以我们将用整章来讨论它们。

## 13.1 闭包：可以使用环境的匿名函数

Rust’s closures are anonymous functions you can save in a variable or pass as arguments to other functions. You can create the closure in one place and then call the closure elsewhere to evaluate it in a different context. Unlike functions, closures can capture values from the scope in which they’re defined. We’ll demonstrate how these closure features allow for code reuse and behavior customization.

Rust的闭包是匿名函数，可以保存在变量中，也可以作为参数传递给其他函数。您可以在一个地方创建闭包，然后在其他地方调用闭包，以便在不同的上下文中对其进行评估。与函数不同，闭包可以从定义它们的范围中捕获值。我们将演示这些闭包特性如何允许代码重用和行为自定义。

### Capturing the Environment with Closures
We’ll first examine how we can use closures to capture values from the environment they’re defined in for later use. Here’s the scenario: Every so often, our t-shirt company gives away an exclusive, limited-edition shirt to someone on our mailing list as a promotion. People on the mailing list can optionally add their favorite color to their profile. If the person chosen for a free shirt has their favorite color set, they get that color shirt. If the person hasn’t specified a favorite color, they get whatever color the company currently has the most of.

There are many ways to implement this. For this example, we’re going to use an enum called ShirtColor that has the variants Red and Blue (limiting the number of colors available for simplicity). We represent the company’s inventory with an Inventory struct that has a field named shirts that contains a Vec<ShirtColor> representing the shirt colors currently in stock. The method giveaway defined on Inventory gets the optional shirt color preference of the free shirt winner, and returns the shirt color the person will get. This setup is shown in Listing 13-1:

我们将首先研究如何使用闭包从定义它们的环境中捕获值以供以后使用。 场景如下：我们的 T 恤公司经常向我们邮件列表中的某人赠送一件独家限量版衬衫作为促销。 邮件列表中的人们可以选择将他们喜欢的颜色添加到他们的个人资料中。 如果被选为免费衬衫的人有他们最喜欢的颜色套装，他们就会得到该颜色的衬衫。 如果此人没有指定最喜欢的颜色，他们会选择公司目前拥有最多的任何颜色。

有很多方法可以实现这一点。 在此示例中，我们将使用名为 ShirtColor 的枚举，它具有红色和蓝色变体（为简单起见，限制可用颜色的数量）。 我们使用 Inventory 结构来表示公司的库存，该结构有一个名为衬衫的字段，其中包含表示当前库存衬衫颜色的 Vec<ShirtColor>。 Inventory 上定义的赠品方法获取免费衬衫获胜者的可选衬衫颜色偏好，并返回该人将获得的衬衫颜色。 此设置如清单 13-1 所示：

Filename: src/main.rs
``````
#[derive(Debug, PartialEq, Copy, Clone)]
enum ShirtColor {
    Red,
    Blue,
}

struct Inventory {
    shirts: Vec<ShirtColor>,
}

impl Inventory {
    fn giveaway(&self, user_preference: Option<ShirtColor>) -> ShirtColor {
        user_preference.unwrap_or_else(|| self.most_stocked())
    }

    fn most_stocked(&self) -> ShirtColor {
        let mut num_red = 0;
        let mut num_blue = 0;

        for color in &self.shirts {
            match color {
                ShirtColor::Red => num_red += 1,
                ShirtColor::Blue => num_blue += 1,
            }
        }
        if num_red > num_blue {
            ShirtColor::Red
        } else {
            ShirtColor::Blue
        }
    }
}

fn main() {
    let store = Inventory {
        shirts: vec![ShirtColor::Blue, ShirtColor::Red, ShirtColor::Blue],
    };

    let user_pref1 = Some(ShirtColor::Red);
    let giveaway1 = store.giveaway(user_pref1);
    println!(
        "The user with preference {:?} gets {:?}",
        user_pref1, giveaway1
    );

    let user_pref2 = None;
    let giveaway2 = store.giveaway(user_pref2);
    println!(
        "The user with preference {:?} gets {:?}",
        user_pref2, giveaway2
    );
}
``````
Listing 13-1: Shirt company giveaway situation

The store defined in main has two blue shirts and one red shirt remaining to distribute for this limited-edition promotion. We call the giveaway method for a user with a preference for a red shirt and a user without any preference.

Again, this code could be implemented in many ways, and here, to focus on closures, we’ve stuck to concepts you’ve already learned except for the body of the giveaway method that uses a closure. In the giveaway method, we get the user preference as a parameter of type Option<ShirtColor> and call the unwrap_or_else method on user_preference. The unwrap_or_else method on Option<T> is defined by the standard library. It takes one argument: a closure without any arguments that returns a value T (the same type stored in the Some variant of the Option<T>, in this case ShirtColor). If the Option<T> is the Some variant, unwrap_or_else returns the value from within the Some. If the Option<T> is the None variant, unwrap_or_else calls the closure and returns the value returned by the closure.

We specify the closure expression || self.most_stocked() as the argument to unwrap_or_else. This is a closure that takes no parameters itself (if the closure had parameters, they would appear between the two vertical bars). The body of the closure calls self.most_stocked(). We’re defining the closure here, and the implementation of unwrap_or_else will evaluate the closure later if the result is needed.

main 中定义的商店还有两件蓝色衬衫和一件红色衬衫可供分发用于此限量版促销活动。 我们将偏爱红色衬衫的用户和没有任何偏爱的用户称为赠品方法。

同样，这段代码可以通过多种方式实现，在这里，为了关注闭包，我们坚持使用您已经学到的概念，除了使用闭包的赠品方法的主体之外。 在赠品方法中，我们将用户首选项作为 Option<ShirtColor> 类型的参数获取，并调用 user_preference 上的 unwrap_or_else 方法。 Option<T> 上的 unwrap_or_else 方法由标准库定义。 它需要一个参数：一个不带任何参数的闭包，返回一个值 T（与存储在 Option<T> 的 Some 变体中的类型相同，在本例中为 ShirtColor）。 如果 Option<T> 是 Some 变体，则 unwrap_or_else 从 Some 中返回值。 如果 Option<T> 是 None 变体，则 unwrap_or_else 调用闭包并返回闭包返回的值。

我们指定闭包表达式 || self.most_stocked() 作为 unwrap_or_else 的参数。 这是一个本身不带参数的闭包（如果闭包有参数，它们将出现在两个垂直条之间）。 闭包的主体调用 self.most_stocked()。 我们在这里定义闭包，如果需要结果，unwrap_or_else 的实现将在稍后评估闭包。

Running this code prints:
``````
$ cargo run
   Compiling shirt-company v0.1.0 (file:///projects/shirt-company)
    Finished dev [unoptimized + debuginfo] target(s) in 0.27s
     Running `target/debug/shirt-company`
The user with preference Some(Red) gets Red
The user with preference None gets Blue
``````
One interesting aspect here is that we’ve passed a closure that calls self.most_stocked() on the current Inventory instance. The standard library didn’t need to know anything about the Inventory or ShirtColor types we defined, or the logic we want to use in this scenario. The closure captures an immutable reference to the self Inventory instance and passes it with the code we specify to the unwrap_or_else method. Functions, on the other hand, are not able to capture their environment in this way.

这里一个有趣的方面是，我们传递了一个在当前 Inventory 实例上调用 self.most_stocked() 的闭包。 标准库不需要了解我们定义的 Inventory 或 ShirtColor 类型，或者我们想要在这种情况下使用的逻辑。 闭包捕获对 self Inventory 实例的不可变引用，并将其与我们指定的代码一起传递给 unwrap_or_else 方法。 另一方面，函数无法以这种方式捕获其环境。

### Closure Type Inference and Annotation
There are more differences between functions and closures. Closures don’t usually require you to annotate the types of the parameters or the return value like fn functions do. Type annotations are required on functions because the types are part of an explicit interface exposed to your users. Defining this interface rigidly is important for ensuring that everyone agrees on what types of values a function uses and returns. Closures, on the other hand, aren’t used in an exposed interface like this: they’re stored in variables and used without naming them and exposing them to users of our library.

Closures are typically short and relevant only within a narrow context rather than in any arbitrary scenario. Within these limited contexts, the compiler can infer the types of the parameters and the return type, similar to how it’s able to infer the types of most variables (there are rare cases where the compiler needs closure type annotations too).

As with variables, we can add type annotations if we want to increase explicitness and clarity at the cost of being more verbose than is strictly necessary. Annotating the types for a closure would look like the definition shown in Listing 13-2. In this example, we’re defining a closure and storing it in a variable rather than defining the closure in the spot we pass it as an argument as we did in Listing 13-1.

函数和闭包之间存在更多差异。 闭包通常不需要像 fn 函数那样注释参数的类型或返回值。 函数上需要类型注释，因为类型是向用户公开的显式接口的一部分。 严格定义此接口对于确保每个人都同意函数使用和返回的值类型非常重要。 另一方面，闭包不会在这样的公开接口中使用：它们存储在变量中并在不命名它们并将它们暴露给我们库的用户的情况下使用。

闭包通常很短，并且仅在狭窄的上下文中相关，而不是在任何任意场景中。 在这些有限的上下文中，编译器可以推断参数的类型和返回类型，类似于推断大多数变量的类型（在极少数情况下，编译器也需要闭包类型注释）。

与变量一样，如果我们想提高明确性和清晰度，则可以添加类型注释，但代价是比严格必要的更加冗长。 注释闭包的类型类似于清单 13-2 中所示的定义。 在这个例子中，我们定义了一个闭包并将其存储在一个变量中，而不是像清单 13-1 中那样在将其作为参数传递的地方定义闭包。

Filename: src/main.rs
``````
    let expensive_closure = |num: u32| -> u32 {
        println!("calculating slowly...");
        thread::sleep(Duration::from_secs(2));
        num
    };
``````
Listing 13-2: Adding optional type annotations of the parameter and return value types in the closure

With type annotations added, the syntax of closures looks more similar to the syntax of functions. Here we define a function that adds 1 to its parameter and a closure that has the same behavior, for comparison. We’ve added some spaces to line up the relevant parts. This illustrates how closure syntax is similar to function syntax except for the use of pipes and the amount of syntax that is optional:

添加了类型注释后，闭包的语法看起来与函数的语法更相似。在这里，我们定义了一个在参数上加1的函数和一个具有相同行为的闭包，以进行比较。我们增加了一些空间来排列相关部分。这说明了闭包语法与函数语法的相似之处，除了管道的使用和可选的语法数量：

``````
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
``````
The first line shows a function definition, and the second line shows a fully annotated closure definition. In the third line, we remove the type annotations from the closure definition. In the fourth line, we remove the brackets, which are optional because the closure body has only one expression. These are all valid definitions that will produce the same behavior when they’re called. The add_one_v3 and add_one_v4 lines require the closures to be evaluated to be able to compile because the types will be inferred from their usage. This is similar to let v = Vec::new(); needing either type annotations or values of some type to be inserted into the Vec for Rust to be able to infer the type.

For closure definitions, the compiler will infer one concrete type for each of their parameters and for their return value. For instance, Listing 13-3 shows the definition of a short closure that just returns the value it receives as a parameter. This closure isn’t very useful except for the purposes of this example. Note that we haven’t added any type annotations to the definition. Because there are no type annotations, we can call the closure with any type, which we’ve done here with String the first time. If we then try to call example_closure with an integer, we’ll get an error.

第一行显示函数定义，第二行显示完整注释的闭包定义。 在第三行中，我们从闭包定义中删除类型注释。 在第四行中，我们删除了括号，这是可选的，因为闭包体只有一个表达式。 这些都是有效的定义，在调用它们时会产生相同的行为。 add_one_v3 和 add_one_v4 行要求对闭包进行评估才能编译，因为类型将从其用法中推断出来。 这类似于 let v = Vec::new(); 需要将类型注释或某种类型的值插入到 Vec 中，以便 Rust 能够推断类型。

对于闭包定义，编译器将为每个参数及其返回值推断出一种具体类型。 例如，清单 13-3 显示了一个短闭包的定义，它只返回作为参数接收的值。 除了本示例的目的之外，这个闭包并不是很有用。 请注意，我们没有在定义中添加任何类型注释。 因为没有类型注释，所以我们可以使用任何类型调用闭包，这是我们第一次使用 String 完成的。 如果我们随后尝试使用整数调用 example_closure，我们将收到错误。

Filename: src/main.rs
``````
This code does not compile!
    let example_closure = |x| x;

    let s = example_closure(String::from("hello"));
    let n = example_closure(5);
``````
Listing 13-3: Attempting to call a closure whose types are inferred with two different types

The compiler gives us this error:
``````
$ cargo run
   Compiling closure-example v0.1.0 (file:///projects/closure-example)
error[E0308]: mismatched types
 --> src/main.rs:5:29
  |
5 |     let n = example_closure(5);
  |             --------------- ^- help: try using a conversion method: `.to_string()`
  |             |               |
  |             |               expected struct `String`, found integer
  |             arguments to this function are incorrect
  |
note: closure parameter defined here
 --> src/main.rs:2:28
  |
2 |     let example_closure = |x| x;
  |                            ^

For more information about this error, try `rustc --explain E0308`.
error: could not compile `closure-example` due to previous error
``````
The first time we call example_closure with the String value, the compiler infers the type of x and the return type of the closure to be String. Those types are then locked into the closure in example_closure, and we get a type error when we next try to use a different type with the same closure.

当我们第一次使用String值调用example_closure时，编译器推断x的类型和闭包的返回类型为String。然后，这些类型被锁定到example_closure中的闭包中，当我们下一次尝试将不同的类型与同一个闭包一起使用时，我们会得到一个类型错误。

### Capturing References or Moving Ownership
Closures can capture values from their environment in three ways, which directly map to the three ways a function can take a parameter: borrowing immutably, borrowing mutably, and taking ownership. The closure will decide which of these to use based on what the body of the function does with the captured values.

闭包可以通过三种方式从其环境中捕获值，这三种方式直接映射到函数获取参数的三种方式：不可变地借用、可变地借用和获取所有权。闭包将根据函数体对捕获的值所做的操作来决定使用其中的哪一个。

In Listing 13-4, we define a closure that captures an immutable reference to the vector named list because it only needs an immutable reference to print the value:

Filename: src/main.rs
``````
fn main() {
    let list = vec![1, 2, 3];
    println!("Before defining closure: {:?}", list);

    let only_borrows = || println!("From closure: {:?}", list);

    println!("Before calling closure: {:?}", list);
    only_borrows();
    println!("After calling closure: {:?}", list);
}
``````
Listing 13-4: Defining and calling a closure that captures an immutable reference

This example also illustrates that a variable can bind to a closure definition, and we can later call the closure by using the variable name and parentheses as if the variable name were a function name.

Because we can have multiple immutable references to list at the same time, list is still accessible from the code before the closure definition, after the closure definition but before the closure is called, and after the closure is called. This code compiles, runs, and prints:

这个例子还说明了变量可以绑定到闭包定义，我们稍后可以使用变量名和括号来调用闭包，就好像变量名是函数名一样。
因为我们可以同时有多个不可变的引用来列出列表，所以在闭包定义之前、在闭包定义之后但在调用闭包之前和调用闭包之后，仍然可以从代码中访问列表。此代码编译、运行和打印：

``````
$ cargo run
   Compiling closure-example v0.1.0 (file:///projects/closure-example)
    Finished dev [unoptimized + debuginfo] target(s) in 0.43s
     Running `target/debug/closure-example`
Before defining closure: [1, 2, 3]
Before calling closure: [1, 2, 3]
From closure: [1, 2, 3]
After calling closure: [1, 2, 3]
``````
Next, in Listing 13-5, we change the closure body so that it adds an element to the list vector. The closure now captures a mutable reference:

Filename: src/main.rs
``````
fn main() {
    let mut list = vec![1, 2, 3];
    println!("Before defining closure: {:?}", list);

    let mut borrows_mutably = || list.push(7);

    borrows_mutably();
    println!("After calling closure: {:?}", list);
}
``````
Listing 13-5: Defining and calling a closure that captures a mutable reference

This code compiles, runs, and prints:
``````
$ cargo run
   Compiling closure-example v0.1.0 (file:///projects/closure-example)
    Finished dev [unoptimized + debuginfo] target(s) in 0.43s
     Running `target/debug/closure-example`
Before defining closure: [1, 2, 3]
After calling closure: [1, 2, 3, 7]
``````
Note that there’s no longer a println! between the definition and the call of the borrows_mutably closure: when borrows_mutably is defined, it captures a mutable reference to list. We don’t use the closure again after the closure is called, so the mutable borrow ends. Between the closure definition and the closure call, an immutable borrow to print isn’t allowed because no other borrows are allowed when there’s a mutable borrow. Try adding a println! there to see what error message you get!

If you want to force the closure to take ownership of the values it uses in the environment even though the body of the closure doesn’t strictly need ownership, you can use the move keyword before the parameter list.

This technique is mostly useful when passing a closure to a new thread to move the data so that it’s owned by the new thread. We’ll discuss threads and why you would want to use them in detail in Chapter 16 when we talk about concurrency, but for now, let’s briefly explore spawning a new thread using a closure that needs the move keyword. Listing 13-6 shows Listing 13-4 modified to print the vector in a new thread rather than in the main thread:

请注意，不再有 println！ 借用可变闭包的定义和调用之间：当定义借用可变时，它捕获对列表的可变引用。 调用闭包后我们不再使用闭包，因此可变借用结束。 在闭包定义和闭包调用之间，不允许进行不可变借用打印，因为当存在可变借用时不允许其他借用。 尝试添加一个 println ！ 在那里查看您收到的错误消息！

如果您想强制闭包取得它在环境中使用的值的所有权，即使闭包主体并不严格需要所有权，您也可以在参数列表之前使用 move 关键字。

当将闭包传递给新线程以移动数据以使其归新线程所有时，此技术最有用。 我们将在第 16 章讨论并发时详细讨论线程以及为什么要详细使用它们，但现在，让我们简要探讨一下使用需要 move 关键字的闭包生成一个新线程。 清单 13-6 显示了修改后的清单 13-4，以在新线程而不是主线程中打印向量：

Filename: src/main.rs
``````
use std::thread;

fn main() {
    let list = vec![1, 2, 3];
    println!("Before defining closure: {:?}", list);

    thread::spawn(move || println!("From thread: {:?}", list))
        .join()
        .unwrap();
}
``````
Listing 13-6: Using move to force the closure for the thread to take ownership of list

We spawn a new thread, giving the thread a closure to run as an argument. The closure body prints out the list. In Listing 13-4, the closure only captured list using an immutable reference because that's the least amount of access to list needed to print it. In this example, even though the closure body still only needs an immutable reference, we need to specify that list should be moved into the closure by putting the move keyword at the beginning of the closure definition. The new thread might finish before the rest of the main thread finishes, or the main thread might finish first. If the main thread maintained ownership of list but ended before the new thread did and dropped list, the immutable reference in the thread would be invalid. Therefore, the compiler requires that list be moved into the closure given to the new thread so the reference will be valid. Try removing the move keyword or using list in the main thread after the closure is defined to see what compiler errors you get!

我们生成一个新线程，为该线程提供一个闭包作为参数运行。 闭包主体打印出列表。 在清单 13-4 中，闭包仅使用不可变引用捕获列表，因为这是打印列表所需的最少访问量。 在此示例中，即使闭包主体仍然只需要一个不可变引用，我们也需要通过将 move 关键字放在闭包定义的开头来指定应将列表移动到闭包中。 新线程可能会在主线程的其余部分完成之前完成，或者主线程可能会先完成。 如果主线程保持列表的所有权，但在新线程完成并删除列表之前结束，则线程中的不可变引用将无效。 因此，编译器要求将列表移动到给新线程的闭包中，以便引用有效。 尝试在定义闭包后删除 move 关键字或在主线程中使用 list ，看看会出现什么编译器错误！


### Moving Captured Values Out of Closures and the Fn Traits
Once a closure has captured a reference or captured ownership of a value from the environment where the closure is defined (thus affecting what, if anything, is moved into the closure), the code in the body of the closure defines what happens to the references or values when the closure is evaluated later (thus affecting what, if anything, is moved out of the closure). A closure body can do any of the following: move a captured value out of the closure, mutate the captured value, neither move nor mutate the value, or capture nothing from the environment to begin with.

一旦闭包从定义闭包的环境中捕获了引用或捕获了值的所有权（从而影响了将什么（如果有的话）移动到闭包中），则闭包主体中的代码将定义稍后评估闭包时引用或值会发生什么（从而影响了将什么从闭包中移出）。闭包主体可以执行以下任何操作：将捕获的值移出闭包，更改捕获的值，既不移动也不更改值，或者一开始就不从环境中捕获任何内容。

The way a closure captures and handles values from the environment affects which traits the closure implements, and traits are how functions and structs can specify what kinds of closures they can use. Closures will automatically implement one, two, or all three of these Fn traits, in an additive fashion, depending on how the closure’s body handles the values:

闭包从环境中捕获和处理值的方式会影响闭包实现哪些特征，而特征是函数和结构如何指定它们可以使用的闭包类型。闭包将以相加的方式自动实现这些Fn特征中的一个、两个或全部三个，具体取决于闭包主体如何处理这些值：

FnOnce applies to closures that can be called once. All closures implement at least this trait, because all closures can be called. A closure that moves captured values out of its body will only implement FnOnce and none of the other Fn traits, because it can only be called once.

FnOnce适用于可以调用一次的闭包。所有闭包都至少实现了这个特性，因为所有闭包都可以被调用。将捕获的值移出其体内的闭包只会实现FnOnce，而不会实现其他Fn特性，因为它只能调用一次。

FnMut applies to closures that don’t move captured values out of their body, but that might mutate the captured values. These closures can be called more than once.

FnMut适用于那些不会将捕获的值移出其主体，但可能会使捕获的值发生变异的闭包。这些闭包可以调用不止一次。

Fn applies to closures that don’t move captured values out of their body and that don’t mutate captured values, as well as closures that capture nothing from their environment. These closures can be called more than once without mutating their environment, which is important in cases such as calling a closure multiple times concurrently.

Fn适用于不会将捕获的值移出其主体且不会突变捕获的值的闭包，以及不会从其环境中捕获任何内容的闭包。这些闭包可以被多次调用，而不会改变其环境，这在同时多次调用闭包的情况下很重要。

Let’s look at the definition of the unwrap_or_else method on Option<T> that we used in Listing 13-1:


``````
impl<T> Option<T> {
    pub fn unwrap_or_else<F>(self, f: F) -> T
    where
        F: FnOnce() -> T
    {
        match self {
            Some(x) => x,
            None => f(),
        }
    }
}
``````
Recall that T is the generic type representing the type of the value in the Some variant of an Option. That type T is also the return type of the unwrap_or_else function: code that calls unwrap_or_else on an Option<String>, for example, will get a String.

Next, notice that the unwrap_or_else function has the additional generic type parameter F. The F type is the type of the parameter named f, which is the closure we provide when calling unwrap_or_else.

The trait bound specified on the generic type F is FnOnce() -> T, which means F must be able to be called once, take no arguments, and return a T. Using FnOnce in the trait bound expresses the constraint that unwrap_or_else is only going to call f at most one time. In the body of unwrap_or_else, we can see that if the Option is Some, f won’t be called. If the Option is None, f will be called once. Because all closures implement FnOnce, unwrap_or_else accepts the most different kinds of closures and is as flexible as it can be.

Note: Functions can implement all three of the Fn traits too. If what we want to do doesn’t require capturing a value from the environment, we can use the name of a function rather than a closure where we need something that implements one of the Fn traits. For example, on an Option<Vec<T>> value, we could call unwrap_or_else(Vec::new) to get a new, empty vector if the value is None.

Now let’s look at the standard library method sort_by_key defined on slices, to see how that differs from unwrap_or_else and why sort_by_key uses FnMut instead of FnOnce for the trait bound. The closure gets one argument in the form of a reference to the current item in the slice being considered, and returns a value of type K that can be ordered. This function is useful when you want to sort a slice by a particular attribute of each item. In Listing 13-7, we have a list of Rectangle instances and we use sort_by_key to order them by their width attribute from low to high:

回想一下，T 是泛型类型，表示 Option 的 Some 变体中值的类型。 该类型 T 也是 unwrap_or_else 函数的返回类型：例如，在 Option<String> 上调用 unwrap_or_else 的代码将获得一个 String。

接下来，请注意 unwrap_or_else 函数具有额外的泛型类型参数 F。F 类型是名为 f 的参数的类型，这是我们在调用 unwrap_or_else 时提供的闭包。

泛型类型 F 上指定的特征边界是 FnOnce() -> T，这意味着 F 必须能够被调用一次，不带参数，并返回 T。在特征边界中使用 FnOnce 表示 unwrap_or_else 仅是的约束 最多会调用 f 一次。 在unwrap_or_else的主体中，我们可以看到如果Option是Some，则不会调用f。 如果 Option 为 None，则 f 将被调用一次。 因为所有闭包都实现 FnOnce，所以 unwrap_or_else 接受最不同类型的闭包，并且尽可能灵活。

注意：函数也可以实现所有三个 Fn 特征。 如果我们想要做的事情不需要从环境中捕获值，那么我们可以在需要实现 Fn 特征之一的东西时使用函数的名称而不是闭包。 例如，对于 Option<Vec<T>> 值，如果该值为 None，我们可以调用 unwrap_or_else(Vec::new) 来获取新的空向量。

现在让我们看一下切片上定义的标准库方法 sort_by_key，看看它与 unwrap_or_else 有何不同，以及为什么 sort_by_key 使用 FnMut 而不是 FnOnce 作为特征绑定。 该闭包以对正在考虑的切片中当前项的引用的形式获取一个参数，并返回一个可排序的 K 类型值。 当您想要按每个项目的特定属性对切片进行排序时，此函数非常有用。 在清单 13-7 中，我们有一个 Rectangle 实例列表，我们使用 sort_by_key 按宽度属性从低到高对它们进行排序：

Filename: src/main.rs
``````
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let mut list = [
        Rectangle { width: 10, height: 1 },
        Rectangle { width: 3, height: 5 },
        Rectangle { width: 7, height: 12 },
    ];

    list.sort_by_key(|r| r.width);
    println!("{:#?}", list);
}
Listing 13-7: Using sort_by_key to order rectangles by width

This code prints:

$ cargo run
   Compiling rectangles v0.1.0 (file:///projects/rectangles)
    Finished dev [unoptimized + debuginfo] target(s) in 0.41s
     Running `target/debug/rectangles`
[
    Rectangle {
        width: 3,
        height: 5,
    },
    Rectangle {
        width: 7,
        height: 12,
    },
    Rectangle {
        width: 10,
        height: 1,
    },
]
``````
The reason sort_by_key is defined to take an FnMut closure is that it calls the closure multiple times: once for each item in the slice. The closure |r| r.width doesn’t capture, mutate, or move out anything from its environment, so it meets the trait bound requirements.

In contrast, Listing 13-8 shows an example of a closure that implements just the FnOnce trait, because it moves a value out of the environment. The compiler won’t let us use this closure with sort_by_key:

sort_by_key 被定义为采用 FnMut 闭包的原因是它多次调用闭包：对切片中的每个项目调用一次。 闭包 |r| r.width 不会捕获、变异或移出其环境中的任何内容，因此它满足特征限制要求。

相比之下，清单 13-8 显示了一个仅实现 FnOnce 特征的闭包示例，因为它将值移出了环境。 编译器不允许我们将这个闭包与 sort_by_key 一起使用：

Filename: src/main.rs
``````
This code does not compile!
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let mut list = [
        Rectangle { width: 10, height: 1 },
        Rectangle { width: 3, height: 5 },
        Rectangle { width: 7, height: 12 },
    ];

    let mut sort_operations = vec![];
    let value = String::from("by key called");

    list.sort_by_key(|r| {
        sort_operations.push(value);
        r.width
    });
    println!("{:#?}", list);
}
``````
Listing 13-8: Attempting to use an FnOnce closure with sort_by_key

This is a contrived, convoluted way (that doesn’t work) to try and count the number of times sort_by_key gets called when sorting list. This code attempts to do this counting by pushing value—a String from the closure’s environment—into the sort_operations vector. The closure captures value then moves value out of the closure by transferring ownership of value to the sort_operations vector. This closure can be called once; trying to call it a second time wouldn’t work because value would no longer be in the environment to be pushed into sort_operations again! Therefore, this closure only implements FnOnce. When we try to compile this code, we get this error that value can’t be moved out of the closure because the closure must implement FnMut:

这是一种人为的、复杂的方法（不起作用），用于尝试计算排序列表时调用 sort_by_key 的次数。 此代码尝试通过将值（来自闭包环境的字符串）推入 sort_operations 向量来完成此计数。 闭包捕获值，然后通过将值的所有权转移到 sort_operations 向量，将值移出闭包。 这个闭包可以被调用一次； 尝试第二次调用它是行不通的，因为值将不再存在于再次被推入 sort_operations 的环境中！ 因此，这个闭包只实现了FnOnce。 当我们尝试编译此代码时，我们收到以下错误：值无法移出闭包，因为闭包必须实现 FnMut：

``````
$ cargo run
   Compiling rectangles v0.1.0 (file:///projects/rectangles)
error[E0507]: cannot move out of `value`, a captured variable in an `FnMut` closure
  --> src/main.rs:18:30
   |
15 |     let value = String::from("by key called");
   |         ----- captured outer variable
16 |
17 |     list.sort_by_key(|r| {
   |                      --- captured by this `FnMut` closure
18 |         sort_operations.push(value);
   |                              ^^^^^ move occurs because `value` has type `String`, which does not implement the `Copy` trait

For more information about this error, try `rustc --explain E0507`.
error: could not compile `rectangles` due to previous error
``````
The error points to the line in the closure body that moves value out of the environment. To fix this, we need to change the closure body so that it doesn’t move values out of the environment. To count the number of times sort_by_key is called, keeping a counter in the environment and incrementing its value in the closure body is a more straightforward way to calculate that. The closure in Listing 13-9 works with sort_by_key because it is only capturing a mutable reference to the num_sort_operations counter and can therefore be called more than once:

该错误指向闭包主体中将值移出环境的行。 为了解决这个问题，我们需要更改闭包主体，以便它不会将值移出环境。 要计算 sort_by_key 被调用的次数，在环境中保留一个计数器并在闭包主体中递增其值是一种更直接的计算方法。 清单 13-9 中的闭包与 sort_by_key 一起使用，因为它仅捕获对 num_sort_operations 计数器的可变引用，因此可以多次调用：

Filename: src/main.rs
``````
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let mut list = [
        Rectangle { width: 10, height: 1 },
        Rectangle { width: 3, height: 5 },
        Rectangle { width: 7, height: 12 },
    ];

    let mut num_sort_operations = 0;
    list.sort_by_key(|r| {
        num_sort_operations += 1;
        r.width
    });
    println!("{:#?}, sorted in {num_sort_operations} operations", list);
}
``````
Listing 13-9: Using an FnMut closure with sort_by_key is allowed

The Fn traits are important when defining or using functions or types that make use of closures. In the next section, we’ll discuss iterators. Many iterator methods take closure arguments, so keep these closure details in mind as we continue!

The Fn traits are important when defining or using functions or types that make use of closures. In the next section, we’ll discuss iterators. Many iterator methods take closure arguments, so keep these closure details in mind as we continue!

## 13.2 Processing a Series of Items with Iterators

The iterator pattern allows you to perform some task on a sequence of items in turn. An iterator is responsible for the logic of iterating over each item and determining when the sequence has finished. When you use iterators, you don’t have to reimplement that logic yourself.

In Rust, iterators are lazy, meaning they have no effect until you call methods that consume the iterator to use it up. For example, the code in Listing 13-10 creates an iterator over the items in the vector v1 by calling the iter method defined on Vec<T>. This code by itself doesn’t do anything useful.

迭代器模式允许您依次对一系列项目执行某些任务。 迭代器负责迭代每个项目并确定序列何时完成的逻辑。 当您使用迭代器时，您不必自己重新实现该逻辑。

在 Rust 中，迭代器是惰性的，这意味着在您调用消耗迭代器的方法将其用完之前，它们不会产生任何效果。 例如，清单 13-10 中的代码通过调用 Vec<T> 上定义的 iter 方法在向量 v1 中的项上创建一个迭代器。 这段代码本身并没有做任何有用的事情。

``````
    let v1 = vec![1, 2, 3];

    let v1_iter = v1.iter();
``````
Listing 13-10: Creating an iterator

The iterator is stored in the v1_iter variable. Once we’ve created an iterator, we can use it in a variety of ways. In Listing 3-5 in Chapter 3, we iterated over an array using a for loop to execute some code on each of its items. Under the hood this implicitly created and then consumed an iterator, but we glossed over how exactly that works until now.

In the example in Listing 13-11, we separate the creation of the iterator from the use of the iterator in the for loop. When the for loop is called using the iterator in v1_iter, each element in the iterator is used in one iteration of the loop, which prints out each value.

迭代器存储在 v1_iter 变量中。 创建迭代器后，我们可以通过多种方式使用它。 在第 3 章的清单 3-5 中，我们使用 for 循环迭代一个数组，以对其每个项目执行一些代码。 在幕后，这隐式地创建然后消耗了一个迭代器，但到目前为止我们掩盖了它到底是如何工作的。

在清单 13-11 的示例中，我们将迭代器的创建与 for 循环中迭代器的使用分开。 当使用 v1_iter 中的迭代器调用 for 循环时，迭代器中的每个元素都会在循环的一次迭代中使用，从而打印出每个值。

``````
    let v1 = vec![1, 2, 3];

    let v1_iter = v1.iter();

    for val in v1_iter {
        println!("Got: {}", val);
    }
``````
Listing 13-11: Using an iterator in a for loop

In languages that don’t have iterators provided by their standard libraries, you would likely write this same functionality by starting a variable at index 0, using that variable to index into the vector to get a value, and incrementing the variable value in a loop until it reached the total number of items in the vector.

Iterators handle all that logic for you, cutting down on repetitive code you could potentially mess up. Iterators give you more flexibility to use the same logic with many different kinds of sequences, not just data structures you can index into, like vectors. Let’s examine how iterators do that.

在标准库没有提供迭代器的语言中，您可能会通过以下方式编写相同的功能：在索引 0 处启动变量，使用该变量索引到向量中以获取值，然后在循环中递增变量值 直到达到向量中的项目总数。

迭代器为您处理所有逻辑，减少可能会造成混乱的重复代码。 迭代器使您可以更灵活地对许多不同类型的序列使用相同的逻辑，而不仅仅是可以索引的数据结构，例如向量。 让我们看看迭代器是如何做到这一点的。

### The Iterator Trait and the next Method
All iterators implement a trait named Iterator that is defined in the standard library. The definition of the trait looks like this:
``````
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // methods with default implementations elided
}
``````
Notice this definition uses some new syntax: type Item and Self::Item, which are defining an associated type with this trait. We’ll talk about associated types in depth in Chapter 19. For now, all you need to know is that this code says implementing the Iterator trait requires that you also define an Item type, and this Item type is used in the return type of the next method. In other words, the Item type will be the type returned from the iterator.

The Iterator trait only requires implementors to define one method: the next method, which returns one item of the iterator at a time wrapped in Some and, when iteration is over, returns None.

We can call the next method on iterators directly; Listing 13-12 demonstrates what values are returned from repeated calls to next on the iterator created from the vector.

请注意，此定义使用了一些新语法：type Item 和 Self::Item，它们定义了与此特征相关联的类型。 我们将在第 19 章中深入讨论关联类型。现在，您需要知道的是这段代码表示实现 Iterator 特征要求您还定义一个 Item 类型，并且该 Item 类型用于返回类型 下一个方法。 换句话说，Item 类型将是从迭代器返回的类型。

Iterator 特征只需要实现者定义一个方法：next 方法，它一次返回迭代器的一项，包装在 Some 中，当迭代结束时，返回 None。

我们可以直接调用迭代器的 next 方法； 清单 13-12 演示了从向量创建的迭代器上重复调用 next 会返回哪些值。

Filename: src/lib.rs
``````
    #[test]
    fn iterator_demonstration() {
        let v1 = vec![1, 2, 3];

        let mut v1_iter = v1.iter();

        assert_eq!(v1_iter.next(), Some(&1));
        assert_eq!(v1_iter.next(), Some(&2));
        assert_eq!(v1_iter.next(), Some(&3));
        assert_eq!(v1_iter.next(), None);
    }
``````
Listing 13-12: Calling the next method on an iterator

Note that we needed to make v1_iter mutable: calling the next method on an iterator changes internal state that the iterator uses to keep track of where it is in the sequence. In other words, this code consumes, or uses up, the iterator. Each call to next eats up an item from the iterator. We didn’t need to make v1_iter mutable when we used a for loop because the loop took ownership of v1_iter and made it mutable behind the scenes.

Also note that the values we get from the calls to next are immutable references to the values in the vector. The iter method produces an iterator over immutable references. If we want to create an iterator that takes ownership of v1 and returns owned values, we can call into_iter instead of iter. Similarly, if we want to iterate over mutable references, we can call iter_mut instead of iter.

请注意，我们需要使 v1_iter 可变：在迭代器上调用 next 方法会更改迭代器用于跟踪其在序列中位置的内部状态。 换句话说，此代码消耗或用完迭代器。 每次调用 next 都会消耗迭代器中的一个项目。 当我们使用 for 循环时，我们不需要使 v1_iter 可变，因为循环获取了 v1_iter 的所有权并使其在幕后可变。

另请注意，我们从调用 next 获得的值是对向量中值的不可变引用。 iter 方法在不可变引用上生成一个迭代器。 如果我们想创建一个迭代器来获取 v1 的所有权并返回拥有的值，我们可以调用 into_iter 而不是 iter。 类似地，如果我们想迭代可变引用，我们可以调用 iter_mut 而不是 iter。

### Methods that Consume the Iterator
The Iterator trait has a number of different methods with default implementations provided by the standard library; you can find out about these methods by looking in the standard library API documentation for the Iterator trait. Some of these methods call the next method in their definition, which is why you’re required to implement the next method when implementing the Iterator trait.

Methods that call next are called consuming adaptors, because calling them uses up the iterator. One example is the sum method, which takes ownership of the iterator and iterates through the items by repeatedly calling next, thus consuming the iterator. As it iterates through, it adds each item to a running total and returns the total when iteration is complete. Listing 13-13 has a test illustrating a use of the sum method:

Iterator 特征有许多不同的方法，其默认实现由标准库提供； 您可以通过查看 Iterator 特征的标准库 API 文档来了解这些方法。 其中一些方法在其定义中调用 next 方法，这就是为什么在实现 Iterator 特征时需要实现 next 方法。

调用 next 的方法称为消耗适配器，因为调用它们会耗尽迭代器。 一个例子是 sum 方法，它获取迭代器的所有权，并通过重复调用 next 来迭代项目，从而消耗迭代器。 在迭代时，它将每个项目添加到运行总计中，并在迭代完成时返回总计。 清单 13-13 有一个测试说明了 sum 方法的使用：

Filename: src/lib.rs
``````
    #[test]
    fn iterator_sum() {
        let v1 = vec![1, 2, 3];

        let v1_iter = v1.iter();

        let total: i32 = v1_iter.sum();

        assert_eq!(total, 6);
    }
``````
Listing 13-13: Calling the sum method to get the total of all items in the iterator

We aren’t allowed to use v1_iter after the call to sum because sum takes ownership of the iterator we call it on.

### Methods that Produce Other Iterators
Iterator adaptors are methods defined on the Iterator trait that don’t consume the iterator. Instead, they produce different iterators by changing some aspect of the original iterator.

Listing 13-14 shows an example of calling the iterator adaptor method map, which takes a closure to call on each item as the items are iterated through. The map method returns a new iterator that produces the modified items. The closure here creates a new iterator in which each item from the vector will be incremented by 1:

迭代器适配器是在迭代器特征上定义的方法，不消耗迭代器。 相反，它们通过更改原始迭代器的某些方面来生成不同的迭代器。

清单 13-14 显示了调用迭代器适配器方法映射的示例，该方法使用一个闭包来在迭代项目时调用每个项目。 map 方法返回一个新的迭代器，该迭代器生成修改后的项。 这里的闭包创建了一个新的迭代器，其中向量中的每个项目都将增加 1：

Filename: src/main.rs
``````
    let v1: Vec<i32> = vec![1, 2, 3];

    v1.iter().map(|x| x + 1);
``````
Listing 13-14: Calling the iterator adaptor map to create a new iterator

However, this code produces a warning:
``````
$ cargo run
   Compiling iterators v0.1.0 (file:///projects/iterators)
warning: unused `Map` that must be used
 --> src/main.rs:4:5
  |
4 |     v1.iter().map(|x| x + 1);
  |     ^^^^^^^^^^^^^^^^^^^^^^^^
  |
  = note: iterators are lazy and do nothing unless consumed
  = note: `#[warn(unused_must_use)]` on by default

warning: `iterators` (bin "iterators") generated 1 warning
    Finished dev [unoptimized + debuginfo] target(s) in 0.47s
     Running `target/debug/iterators`
``````
The code in Listing 13-14 doesn’t do anything; the closure we’ve specified never gets called. The warning reminds us why: iterator adaptors are lazy, and we need to consume the iterator here.

To fix this warning and consume the iterator, we’ll use the collect method, which we used in Chapter 12 with env::args in Listing 12-1. This method consumes the iterator and collects the resulting values into a collection data type.

In Listing 13-15, we collect the results of iterating over the iterator that’s returned from the call to map into a vector. This vector will end up containing each item from the original vector incremented by 1.

清单 13-14 中的代码不执行任何操作； 我们指定的闭包永远不会被调用。 该警告提醒我们原因：迭代器适配器是惰性的，我们需要在这里使用迭代器。

为了修复这个警告并使用迭代器，我们将使用collect方法，我们在第12章中使用了该方法以及清单12-1中的env::args。 此方法使用迭代器并将结果值收集到集合数据类型中。

在清单 13-15 中，我们收集了对映射到向量的调用返回的迭代器进行迭代的结果。 该向量最终将包含原始向量中每一项加 1。

Filename: src/main.rs
``````
    let v1: Vec<i32> = vec![1, 2, 3];

    let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();

    assert_eq!(v2, vec![2, 3, 4]);
``````
Listing 13-15: Calling the map method to create a new iterator and then calling the collect method to consume the new iterator and create a vector

Because map takes a closure, we can specify any operation we want to perform on each item. This is a great example of how closures let you customize some behavior while reusing the iteration behavior that the Iterator trait provides.

You can chain multiple calls to iterator adaptors to perform complex actions in a readable way. But because all iterators are lazy, you have to call one of the consuming adaptor methods to get results from calls to iterator adaptors.

Using Closures that Capture Their Environment
Many iterator adapters take closures as arguments, and commonly the closures we’ll specify as arguments to iterator adapters will be closures that capture their environment.

For this example, we’ll use the filter method that takes a closure. The closure gets an item from the iterator and returns a bool. If the closure returns true, the value will be included in the iteration produced by filter. If the closure returns false, the value won’t be included.

In Listing 13-16, we use filter with a closure that captures the shoe_size variable from its environment to iterate over a collection of Shoe struct instances. It will return only shoes that are the specified size.

因为 map 采用闭包，所以我们可以指定要对每个项目执行的任何操作。 这是一个很好的例子，说明了闭包如何让您在重用 Iterator Trait 提供的迭代行为的同时自定义某些行为。

您可以链接对迭代器适配器的多个调用，以可读的方式执行复杂的操作。 但由于所有迭代器都是惰性的，因此您必须调用使用适配器方法之一才能从迭代器适配器的调用中获取结果。

使用捕获环境的闭包
许多迭代器适配器将闭包作为参数，通常我们指定为迭代器适配器参数的闭包将是捕获其环境的闭包。

对于这个例子，我们将使用带有闭包的过滤器方法。 闭包从迭代器获取一个项目并返回一个布尔值。 如果闭包返回 true，则该值将包含在过滤器生成的迭代中。 如果闭包返回 false，则不会包含该值。

在清单 13-16 中，我们使用带有闭包的过滤器，该闭包从其环境中捕获 Shoes_size 变量，以迭代 Shoe 结构体实例的集合。 它将仅返回指定尺寸的鞋子。

Filename: src/lib.rs
``````
#[derive(PartialEq, Debug)]
struct Shoe {
    size: u32,
    style: String,
}

fn shoes_in_size(shoes: Vec<Shoe>, shoe_size: u32) -> Vec<Shoe> {
    shoes.into_iter().filter(|s| s.size == shoe_size).collect()
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn filters_by_size() {
        let shoes = vec![
            Shoe {
                size: 10,
                style: String::from("sneaker"),
            },
            Shoe {
                size: 13,
                style: String::from("sandal"),
            },
            Shoe {
                size: 10,
                style: String::from("boot"),
            },
        ];

        let in_my_size = shoes_in_size(shoes, 10);

        assert_eq!(
            in_my_size,
            vec![
                Shoe {
                    size: 10,
                    style: String::from("sneaker")
                },
                Shoe {
                    size: 10,
                    style: String::from("boot")
                },
            ]
        );
    }
}
``````
Listing 13-16: Using the filter method with a closure that captures shoe_size

The shoes_in_size function takes ownership of a vector of shoes and a shoe size as parameters. It returns a vector containing only shoes of the specified size.

In the body of shoes_in_size, we call into_iter to create an iterator that takes ownership of the vector. Then we call filter to adapt that iterator into a new iterator that only contains elements for which the closure returns true.

The closure captures the shoe_size parameter from the environment and compares the value with each shoe’s size, keeping only shoes of the size specified. Finally, calling collect gathers the values returned by the adapted iterator into a vector that’s returned by the function.

The test shows that when we call shoes_in_size, we get back only shoes that have the same size as the value we specified.

Shoes_in_size 函数将鞋子向量的所有权和鞋子尺寸作为参数。 它返回一个仅包含指定尺寸鞋子的向量。

在shoes_in_size的主体中，我们调用into_iter来创建一个拥有向量所有权的迭代器。 然后我们调用 filter 将该迭代器调整为一个新的迭代器，该迭代器仅包含闭包返回 true 的元素。

该闭包从环境中捕获hoe_size参数，并将该值与每只鞋子的尺寸进行比较，仅保留指定尺寸的鞋子。 最后，调用collect将适应迭代器返回的值收集到函数返回的向量中。

测试表明，当我们调用 Shoes_in_size 时，我们只返回与我们指定的值相同尺寸的鞋子。

## 13.3 改进我们的I/O项目
With this new knowledge about iterators, we can improve the I/O project in Chapter 12 by using iterators to make places in the code clearer and more concise. Let’s look at how iterators can improve our implementation of the Config::build function and the search function.

有了关于迭代器的新知识，我们可以通过使用迭代器来改进第 12 章中的 I/O 项目，使代码中的位置更清晰、更简洁。 让我们看看迭代器如何改进 Config::build 函数和搜索函数的实现。

### Removing a clone Using an Iterator
In Listing 12-6, we added code that took a slice of String values and created an instance of the Config struct by indexing into the slice and cloning the values, allowing the Config struct to own those values. In Listing 13-17, we’ve reproduced the implementation of the Config::build function as it was in Listing 12-23:

在清单 12-6 中，我们添加了代码，该代码获取 String 值的切片，并通过索引该切片并克隆值来创建 Config 结构的实例，从而允许 Config 结构拥有这些值。 在清单 13-17 中，我们重现了 Config::build 函数的实现，如清单 12-23 所示：

Filename: src/lib.rs
``````
impl Config {
    pub fn build(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("not enough arguments");
        }

        let query = args[1].clone();
        let file_path = args[2].clone();

        let ignore_case = env::var("IGNORE_CASE").is_ok();

        Ok(Config {
            query,
            file_path,
            ignore_case,
        })
    }
}
``````
Listing 13-17: Reproduction of the Config::build function from Listing 12-23

At the time, we said not to worry about the inefficient clone calls because we would remove them in the future. Well, that time is now!

We needed clone here because we have a slice with String elements in the parameter args, but the build function doesn’t own args. To return ownership of a Config instance, we had to clone the values from the query and file_path fields of Config so the Config instance can own its values.

With our new knowledge about iterators, we can change the build function to take ownership of an iterator as its argument instead of borrowing a slice. We’ll use the iterator functionality instead of the code that checks the length of the slice and indexes into specific locations. This will clarify what the Config::build function is doing because the iterator will access the values.

Once Config::build takes ownership of the iterator and stops using indexing operations that borrow, we can move the String values from the iterator into Config rather than calling clone and making a new allocation.

当时，我们说不要担心低效的克隆调用，因为我们将来会删除它们。 嗯，就是现在了！

我们在这里需要克隆，因为我们在参数 args 中有一个包含 String 元素的切片，但构建函数不拥有 args。 要返回 Config 实例的所有权，我们必须从 Config 的 query 和 file_path 字段克隆值，以便 Config 实例可以拥有其值。

有了关于迭代器的新知识，我们可以更改构建函数以将迭代器的所有权作为其参数，而不是借用切片。 我们将使用迭代器功能，而不是检查切片长度和特定位置索引的代码。 这将澄清 Config::build 函数正在做什么，因为迭代器将访问这些值。

一旦 Config::build 获得迭代器的所有权并停止使用借用的索引操作，我们就可以将 String 值从迭代器移动到 Config 中，而不是调用克隆并进行新的分配。

### Using the Returned Iterator Directly
Open your I/O project’s src/main.rs file, which should look like this:

Filename: src/main.rs
``````
fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::build(&args).unwrap_or_else(|err| {
        eprintln!("Problem parsing arguments: {err}");
        process::exit(1);
    });

    // --snip--
}
``````
We’ll first change the start of the main function that we had in Listing 12-24 to the code in Listing 13-18, which this time uses an iterator. This won’t compile until we update Config::build as well.

Filename: src/main.rs
``````
This code does not compile!
fn main() {
    let config = Config::build(env::args()).unwrap_or_else(|err| {
        eprintln!("Problem parsing arguments: {err}");
        process::exit(1);
    });

    // --snip--
}
``````
Listing 13-18: Passing the return value of env::args to Config::build

The env::args function returns an iterator! Rather than collecting the iterator values into a vector and then passing a slice to Config::build, now we’re passing ownership of the iterator returned from env::args to Config::build directly.

Next, we need to update the definition of Config::build. In your I/O project’s src/lib.rs file, let’s change the signature of Config::build to look like Listing 13-19. This still won’t compile because we need to update the function body.

env::args 函数返回一个迭代器！ 现在我们不是将迭代器值收集到向量中，然后将切片传递给 Config::build，而是直接将从 env::args 返回的迭代器的所有权传递给 Config::build。

接下来，我们需要更新 Config::build 的定义。 在 I/O 项目的 src/lib.rs 文件中，我们将 Config::build 的签名更改为清单 13-19 所示。 这仍然无法编译，因为我们需要更新函数体。

Filename: src/lib.rs
``````
This code does not compile!
impl Config {
    pub fn build(
        mut args: impl Iterator<Item = String>,
    ) -> Result<Config, &'static str> {
        // --snip--
``````
Listing 13-19: Updating the signature of Config::build to expect an iterator

The standard library documentation for the env::args function shows that the type of the iterator it returns is std::env::Args, and that type implements the Iterator trait and returns String values.

We’ve updated the signature of the Config::build function so the parameter args has a generic type with the trait bounds impl Iterator<Item = String> instead of &[String]. This usage of the impl Trait syntax we discussed in the “Traits as Parameters” section of Chapter 10 means that args can be any type that implements the Iterator type and returns String items.

Because we’re taking ownership of args and we’ll be mutating args by iterating over it, we can add the mut keyword into the specification of the args parameter to make it mutable.

env::args 函数的标准库文档显示，它返回的迭代器的类型是 std::env::Args，并且该类型实现 Iterator 特征并返回 String 值。

我们更新了 Config::build 函数的签名，因此参数 args 具有带有特征边界 impl Iterator<Item = String> 而不是 &[String] 的泛型类型。 我们在第 10 章的“作为参数的特征”部分中讨论的 impl Trait 语法的用法意味着 args 可以是实现 Iterator 类型并返回 String 项的任何类型。

因为我们正在获取 args 的所有权，并且我们将通过迭代来改变 args，所以我们可以将 mut 关键字添加到 args 参数的规范中以使其可变。

### Using Iterator Trait Methods Instead of Indexing
Next, we’ll fix the body of Config::build. Because args implements the Iterator trait, we know we can call the next method on it! Listing 13-20 updates the code from Listing 12-23 to use the next method:

接下来，我们将修复 Config::build 的主体。 因为 args 实现了 Iterator 特性，所以我们知道我们可以调用它的 next 方法！ 清单 13-20 更新了清单 12-23 中的代码以使用 next 方法：

Filename: src/lib.rs
``````
impl Config {
    pub fn build(
        mut args: impl Iterator<Item = String>,
    ) -> Result<Config, &'static str> {
        args.next();

        let query = match args.next() {
            Some(arg) => arg,
            None => return Err("Didn't get a query string"),
        };

        let file_path = match args.next() {
            Some(arg) => arg,
            None => return Err("Didn't get a file path"),
        };

        let ignore_case = env::var("IGNORE_CASE").is_ok();

        Ok(Config {
            query,
            file_path,
            ignore_case,
        })
    }
}
``````
Listing 13-20: Changing the body of Config::build to use iterator methods

Remember that the first value in the return value of env::args is the name of the program. We want to ignore that and get to the next value, so first we call next and do nothing with the return value. Second, we call next to get the value we want to put in the query field of Config. If next returns a Some, we use a match to extract the value. If it returns None, it means not enough arguments were given and we return early with an Err value. We do the same thing for the file_path value.

请记住，env::args 返回值中的第一个值是程序的名称。 我们想忽略它并获取下一个值，因此首先我们调用 next 并对返回值不执行任何操作。 其次，我们调用 next 来获取我们想要放入 Config 的查询字段中的值。 如果 next 返回 Some，我们使用匹配来提取值。 如果它返回 None，则意味着没有给出足够的参数，我们会提前返回 Err 值。 我们对 file_path 值执行相同的操作。

### Making Code Clearer with Iterator Adaptors
We can also take advantage of iterators in the search function in our I/O project, which is reproduced here in Listing 13-21 as it was in Listing 12-19:

Filename: src/lib.rs
``````
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    let mut results = Vec::new();

    for line in contents.lines() {
        if line.contains(query) {
            results.push(line);
        }
    }

    results
}
``````
Listing 13-21: The implementation of the search function from Listing 12-19

We can write this code in a more concise way using iterator adaptor methods. Doing so also lets us avoid having a mutable intermediate results vector. The functional programming style prefers to minimize the amount of mutable state to make code clearer. Removing the mutable state might enable a future enhancement to make searching happen in parallel, because we wouldn’t have to manage concurrent access to the results vector. Listing 13-22 shows this change:

我们可以使用迭代器适配器方法以更简洁的方式编写此代码。 这样做还可以让我们避免出现可变的中间结果向量。 函数式编程风格更喜欢最小化可变状态的数量以使代码更清晰。 删除可变状态可能会启用未来的增强功能，使搜索并行发生，因为我们不必管理对结果向量的并发访问。 清单 13-22 显示了这一更改：

Filename: src/lib.rs
``````
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    contents
        .lines()
        .filter(|line| line.contains(query))
        .collect()
}
``````
Listing 13-22: Using iterator adaptor methods in the implementation of the search function

Recall that the purpose of the search function is to return all lines in contents that contain the query. Similar to the filter example in Listing 13-16, this code uses the filter adaptor to keep only the lines that line.contains(query) returns true for. We then collect the matching lines into another vector with collect. Much simpler! Feel free to make the same change to use iterator methods in the search_case_insensitive function as well.

回想一下，搜索函数的目的是返回内容中包含查询的所有行。 与清单 13-16 中的过滤器示例类似，此代码使用过滤器适配器仅保留 line.contains(query) 返回 true 的行。 然后我们使用collect将匹配的线收集到另一个向量中。 简单多了！ 您也可以随意进行相同的更改，以在 search_case_insensitive 函数中使用迭代器方法。

### Choosing Between Loops or Iterators
The next logical question is which style you should choose in your own code and why: the original implementation in Listing 13-21 or the version using iterators in Listing 13-22. Most Rust programmers prefer to use the iterator style. It’s a bit tougher to get the hang of at first, but once you get a feel for the various iterator adaptors and what they do, iterators can be easier to understand. Instead of fiddling with the various bits of looping and building new vectors, the code focuses on the high-level objective of the loop. This abstracts away some of the commonplace code so it’s easier to see the concepts that are unique to this code, such as the filtering condition each element in the iterator must pass.

But are the two implementations truly equivalent? The intuitive assumption might be that the more low-level loop will be faster. Let’s talk about performance.

下一个逻辑问题是您应该在自己的代码中选择哪种样式以及为什么：清单 13-21 中的原始实现或清单 13-22 中使用迭代器的版本。 大多数 Rust 程序员更喜欢使用迭代器风格。 一开始掌握窍门有点困难，但是一旦您了解了各种迭代器适配器及其用途，迭代器就会更容易理解。 该代码不是摆弄循环的各个部分和构建新向量，而是专注于循环的高级目标。 这抽象了一些常见的代码，因此更容易看到该代码特有的概念，例如迭代器中每个元素必须通过的过滤条件。

但这两种实现真的等效吗？ 直观的假设可能是越低级的循环会越快。 我们来谈谈性能。

## 13.4 性能比较：循环vs迭代
To determine whether to use loops or iterators, you need to know which implementation is faster: the version of the search function with an explicit for loop or the version with iterators.

We ran a benchmark by loading the entire contents of The Adventures of Sherlock Holmes by Sir Arthur Conan Doyle into a String and looking for the word the in the contents. Here are the results of the benchmark on the version of search using the for loop and the version using iterators:

要确定是使用循环还是迭代器，您需要知道哪种实现速度更快：带有显式 for 循环的搜索函数版本还是带有迭代器的版本。

我们通过将阿瑟·柯南道尔爵士所著的《夏洛克·福尔摩斯历险记》的全部内容加载到字符串中并在内容中查找单词 the 来运行基准测试。 以下是使用 for 循环的搜索版本和使用迭代器的版本的基准测试结果：

``````
test bench_search_for  ... bench:  19,620,300 ns/iter (+/- 915,700)
test bench_search_iter ... bench:  19,234,900 ns/iter (+/- 657,200)
``````
The iterator version was slightly faster! We won’t explain the benchmark code here, because the point is not to prove that the two versions are equivalent but to get a general sense of how these two implementations compare performance-wise.

For a more comprehensive benchmark, you should check using various texts of various sizes as the contents, different words and words of different lengths as the query, and all kinds of other variations. The point is this: iterators, although a high-level abstraction, get compiled down to roughly the same code as if you’d written the lower-level code yourself. Iterators are one of Rust’s zero-cost abstractions, by which we mean using the abstraction imposes no additional runtime overhead. This is analogous to how Bjarne Stroustrup, the original designer and implementor of C++, defines zero-overhead in “Foundations of C++” (2012):

In general, C++ implementations obey the zero-overhead principle: What you don’t use, you don’t pay for. And further: What you do use, you couldn’t hand code any better.

As another example, the following code is taken from an audio decoder. The decoding algorithm uses the linear prediction mathematical operation to estimate future values based on a linear function of the previous samples. This code uses an iterator chain to do some math on three variables in scope: a buffer slice of data, an array of 12 coefficients, and an amount by which to shift data in qlp_shift. We’ve declared the variables within this example but not given them any values; although this code doesn’t have much meaning outside of its context, it’s still a concise, real-world example of how Rust translates high-level ideas to low-level code.

迭代器版本稍微快一些！ 我们不会在这里解释基准测试代码，因为重点不是证明这两个版本是等效的，而是为了大致了解这两个实现在性能方面的比较。

为了获得更全面的基准，您应该使用不同大小的各种文本作为内容、不同的单词和不同长度的单词作为查询以及各种其他变体进行检查。 要点是：迭代器虽然是一种高级抽象，但它会被编译为大致相同的代码，就像您自己编写较低级代码一样。 迭代器是 Rust 的零成本抽象之一，我们的意思是使用该抽象不会带来额外的运行时开销。 这类似于 C++ 的原始设计者和实现者 Bjarne Stroustrup 在《C++ 基础》（2012 年）中定义零开销的方式：

一般来说，C++ 实现遵循零开销原则：不使用的东西，就不需要付费。 更进一步：你所使用的东西，你无法更好地手工编写代码。

作为另一个示例，以下代码取自音频解码器。 解码算法使用线性预测数学运算，根据先前样本的线性函数来估计未来值。 此代码使用迭代器链对范围内的三个变量进行一些数学运算：数据缓冲区切片、12 个系数的数组以及 qlp_shift 中数据的移位量。 我们在这个例子中声明了变量，但没有给它们任何值； 尽管这段代码在其上下文之外没有太多意义，但它仍然是 Rust 如何将高级思想转化为低级代码的简洁、真实的示例。

``````
let buffer: &mut [i32];
let coefficients: [i64; 12];
let qlp_shift: i16;

for i in 12..buffer.len() {
    let prediction = coefficients.iter()
                                 .zip(&buffer[i - 12..i])
                                 .map(|(&c, &s)| c * s as i64)
                                 .sum::<i64>() >> qlp_shift;
    let delta = buffer[i];
    buffer[i] = prediction as i32 + delta;
}
``````
To calculate the value of prediction, this code iterates through each of the 12 values in coefficients and uses the zip method to pair the coefficient values with the previous 12 values in buffer. Then, for each pair, we multiply the values together, sum all the results, and shift the bits in the sum qlp_shift bits to the right.

Calculations in applications like audio decoders often prioritize performance most highly. Here, we’re creating an iterator, using two adaptors, and then consuming the value. What assembly code would this Rust code compile to? Well, as of this writing, it compiles down to the same assembly you’d write by hand. There’s no loop at all corresponding to the iteration over the values in coefficients: Rust knows that there are 12 iterations, so it “unrolls” the loop. Unrolling is an optimization that removes the overhead of the loop controlling code and instead generates repetitive code for each iteration of the loop.

All of the coefficients get stored in registers, which means accessing the values is very fast. There are no bounds checks on the array access at runtime. All these optimizations that Rust is able to apply make the resulting code extremely efficient. Now that you know this, you can use iterators and closures without fear! They make code seem like it’s higher level but don’t impose a runtime performance penalty for doing so.

为了计算预测值，此代码迭代系数中的 12 个值中的每一个，并使用 zip 方法将系数值与缓冲区中的前 12 个值配对。 然后，对于每一对，我们将这些值相乘，对所有结果求和，并将总和 qlp_shift 位中的位向右移动。

音频解码器等应用中的计算通常最优先考虑性能。 在这里，我们使用两个适配器创建一个迭代器，然后使用该值。 该 Rust 代码会编译成什么汇编代码？ 好吧，在撰写本文时，它编译成与您手写的程序集相同。 根本不存在与系数值迭代相对应的循环：Rust 知道有 12 次迭代，因此它“展开”循环。 展开是一种优化，它消除了循环控制代码的开销，而是为循环的每次迭代生成重复的代码。

所有系数都存储在寄存器中，这意味着访问这些值非常快。 运行时对数组访问没有边界检查。 Rust 能够应用的所有这些优化使生成的代码极其高效。 现在您知道了这一点，您可以毫无恐惧地使用迭代器和闭包！ 它们使代码看起来更高级别，但不会因此而造成运行时性能损失。

## 13.5 Summary
Closures and iterators are Rust features inspired by functional programming language ideas. They contribute to Rust’s capability to clearly express high-level ideas at low-level performance. The implementations of closures and iterators are such that runtime performance is not affected. This is part of Rust’s goal to strive to provide zero-cost abstractions.

Now that we’ve improved the expressiveness of our I/O project, let’s look at some more features of cargo that will help us share the project with the world.

闭包和迭代器是受函数式编程语言思想启发的 Rust 功能。 它们有助于 Rust 以低级性能清晰表达高级思想的能力。 闭包和迭代器的实现不会影响运行时性能。 这是 Rust 努力提供零成本抽象的目标的一部分。

现在我们已经提高了 I/O 项目的表现力，让我们看看 Cargo 的更多功能，这些功能将帮助我们与世界分享该项目。