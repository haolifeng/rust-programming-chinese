# 6. 枚举和模式匹配
In this chapter, we’ll look at enumerations, also referred to as enums. Enums allow you to define a type by enumerating its possible variants. First we’ll define and use an enum to show how an enum can encode meaning along with data. Next, we’ll explore a particularly useful enum, called Option, which expresses that a value can be either something or nothing. Then we’ll look at how pattern matching in the match expression makes it easy to run different code for different values of an enum. Finally, we’ll cover how the if let construct is another convenient and concise idiom available to handle enums in your code.

在本章中，我们将讨论枚举，也称为枚举。 枚举允许您通过枚举其可能的变体来定义类型。 首先，我们将定义并使用枚举来展示枚举如何将含义与数据一起编码。 接下来，我们将探索一个特别有用的枚举，称为 Option，它表示一个值可以是某物，也可以是无。 然后我们将了解匹配表达式中的模式匹配如何轻松地针对枚举的不同值运行不同的代码。 最后，我们将介绍 if let 构造如何成为另一种方便而简洁的习惯用法，可用于处理代码中的枚举。

## 6.1 Defining an Enum

Where structs give you a way of grouping together related fields and data, like a Rectangle with its width and height, enums give you a way of saying a value is one of a possible set of values. For example, we may want to say that Rectangle is one of a set of possible shapes that also includes Circle and Triangle. To do this, Rust allows us to encode these possibilities as an enum.

结构为您提供了一种将相关字段和数据分组在一起的方法，例如具有宽度和高度的矩形，枚举为您提供一种表示值是一组可能值之一的方法。例如，我们可能想说矩形是一组可能的形状之一，其中还包括圆形和三角形。为此，Rust允许我们将这些可能性编码为枚举。

Let’s look at a situation we might want to express in code and see why enums are useful and more appropriate than structs in this case. Say we need to work with IP addresses. Currently, two major standards are used for IP addresses: version four and version six. Because these are the only possibilities for an IP address that our program will come across, we can enumerate all possible variants, which is where enumeration gets its name.

Any IP address can be either a version four or a version six address, but not both at the same time. That property of IP addresses makes the enum data structure appropriate because an enum value can only be one of its variants. Both version four and version six addresses are still fundamentally IP addresses, so they should be treated as the same type when the code is handling situations that apply to any kind of IP address.

We can express this concept in code by defining an IpAddrKind enumeration and listing the possible kinds an IP address can be, V4 and V6. These are the variants of the enum:

让我们看一下我们可能想要用代码表达的情况，看看为什么枚举在这种情况下有用并且比结构更合适。 假设我们需要使用 IP 地址。 目前，IP 地址使用两个主要标准：第四版和第六版。 因为这些是我们的程序将遇到的 IP 地址的唯一可能性，所以我们可以枚举所有可能的变体，这就是枚举名称的由来。

任何 IP 地址都可以是版本 4 或版本 6 地址，但不能同时是两者。 IP 地址的这一属性使得枚举数据结构变得合适，因为枚举值只能是其变体之一。 版本四和版本六地址本质上仍然是 IP 地址，因此当代码处理适用于任何类型 IP 地址的情况时，应将它们视为同一类型。

我们可以通过定义 IpAddrKind 枚举并列出 IP 地址可能的类型（V4 和 V6）来用代码表达这个概念。 这些是枚举的变体：

``````
enum IpAddrKind {
    V4,
    V6,
}
``````
IpAddrKind is now a custom data type that we can use elsewhere in our code.

### Enum Values
We can create instances of each of the two variants of IpAddrKind like this:
``````
    let four = IpAddrKind::V4;
    let six = IpAddrKind::V6;
``````
Note that the variants of the enum are namespaced under its identifier, and we use a double colon to separate the two. This is useful because now both values IpAddrKind::V4 and IpAddrKind::V6 are of the same type: IpAddrKind. We can then, for instance, define a function that takes any IpAddrKind:

请注意，枚举的变体在其标识符下命名，我们使用双冒号来分隔两者。 这很有用，因为现在 IpAddrKind::V4 和 IpAddrKind::V6 的类型相同：IpAddrKind。 例如，我们可以定义一个接受任何 IpAddrKind 的函数：

``````
fn route(ip_kind: IpAddrKind) {}
``````
And we can call this function with either variant:
``````
    route(IpAddrKind::V4);
    route(IpAddrKind::V6);
``````
Using enums has even more advantages. Thinking more about our IP address type, at the moment we don’t have a way to store the actual IP address data; we only know what kind it is. Given that you just learned about structs in Chapter 5, you might be tempted to tackle this problem with structs as shown in Listing 6-1.

使用枚举还有更多优点。 进一步考虑我们的 IP 地址类型，目前我们没有办法存储实际的 IP 地址数据； 我们只知道它是什么类型。 鉴于您刚刚在第 5 章中了解了结构体，您可能会想使用结构体来解决这个问题，如清单 6-1 所示。

``````
    enum IpAddrKind {
        V4,
        V6,
    }

    struct IpAddr {
        kind: IpAddrKind,
        address: String,
    }

    let home = IpAddr {
        kind: IpAddrKind::V4,
        address: String::from("127.0.0.1"),
    };

    let loopback = IpAddr {
        kind: IpAddrKind::V6,
        address: String::from("::1"),
    };
``````
Listing 6-1: Storing the data and IpAddrKind variant of an IP address using a struct

Here, we’ve defined a struct IpAddr that has two fields: a kind field that is of type IpAddrKind (the enum we defined previously) and an address field of type String. We have two instances of this struct. The first is home, and it has the value IpAddrKind::V4 as its kind with associated address data of 127.0.0.1. The second instance is loopback. It has the other variant of IpAddrKind as its kind value, V6, and has address ::1 associated with it. We’ve used a struct to bundle the kind and address values together, so now the variant is associated with the value.

However, representing the same concept using just an enum is more concise: rather than an enum inside a struct, we can put data directly into each enum variant. This new definition of the IpAddr enum says that both V4 and V6 variants will have associated String values:

在这里，我们定义了一个 struct IpAddr，它有两个字段：一个 IpAddrKind 类型的 kind 字段（我们之前定义的枚举）和一个 String 类型的地址字段。 我们有这个结构的两个实例。 第一个是 home，它的类型为 IpAddrKind::V4，关联的地址数据为 127.0.0.1。 第二个实例是环回。 它具有 IpAddrKind 的另一个变体作为其种类值 V6，并具有与其关联的地址 ::1。 我们使用了一个结构体将种类和地址值捆绑在一起，因此现在变体与该值相关联。

然而，仅使用枚举来表示相同的概念更简洁：我们可以将数据直接放入每个枚举变体中，而不是结构中的枚举。 IpAddr 枚举的新定义表示 V4 和 V6 变体都将具有关联的字符串值：

``````
    enum IpAddr {
        V4(String),
        V6(String),
    }

    let home = IpAddr::V4(String::from("127.0.0.1"));

    let loopback = IpAddr::V6(String::from("::1"));
``````
We attach data to each variant of the enum directly, so there is no need for an extra struct. Here, it’s also easier to see another detail of how enums work: the name of each enum variant that we define also becomes a function that constructs an instance of the enum. That is, IpAddr::V4() is a function call that takes a String argument and returns an instance of the IpAddr type. We automatically get this constructor function defined as a result of defining the enum.

There’s another advantage to using an enum rather than a struct: each variant can have different types and amounts of associated data. Version four IP addresses will always have four numeric components that will have values between 0 and 255. If we wanted to store V4 addresses as four u8 values but still express V6 addresses as one String value, we wouldn’t be able to with a struct. Enums handle this case with ease:

我们直接将数据附加到枚举的每个变体，因此不需要额外的结构。 在这里，也更容易看到枚举如何工作的另一个细节：我们定义的每个枚举变体的名称也成为构造枚举实例的函数。 也就是说，IpAddr::V4() 是一个函数调用，它接受 String 参数并返回 IpAddr 类型的实例。 作为定义枚举的结果，我们自动定义了这个构造函数。

使用枚举而不是结构体还有另一个优点：每个变体可以具有不同类型和数量的关联数据。 版本四的 IP 地址将始终具有四个数字组件，其值介于 0 到 255 之间。如果我们想要将 V4 地址存储为四个 u8 值，但仍将 V6 地址表示为一个字符串值，我们将无法使用结构体 。 枚举可以轻松处理这种情况：

``````
    enum IpAddr {
        V4(u8, u8, u8, u8),
        V6(String),
    }

    let home = IpAddr::V4(127, 0, 0, 1);

    let loopback = IpAddr::V6(String::from("::1"));
``````
We’ve shown several different ways to define data structures to store version four and version six IP addresses. However, as it turns out, wanting to store IP addresses and encode which kind they are is so common that the standard library has a definition we can use! Let’s look at how the standard library defines IpAddr: it has the exact enum and variants that we’ve defined and used, but it embeds the address data inside the variants in the form of two different structs, which are defined differently for each variant:

我们展示了几种不同的方法来定义数据结构来存储第四版和第六版 IP 地址。 然而，事实证明，想要存储 IP 地址并对其类型进行编码非常常见，以至于标准库有一个我们可以使用的定义！ 让我们看看标准库如何定义 IpAddr：它具有我们定义和使用的精确枚举和变体，但它以两种不同结构的形式将地址数据嵌入到变体中，每个结构体的定义不同：

``````
struct Ipv4Addr {
    // --snip--
}

struct Ipv6Addr {
    // --snip--
}

enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}
``````
This code illustrates that you can put any kind of data inside an enum variant: strings, numeric types, or structs, for example. You can even include another enum! Also, standard library types are often not much more complicated than what you might come up with.

Note that even though the standard library contains a definition for IpAddr, we can still create and use our own definition without conflict because we haven’t brought the standard library’s definition into our scope. We’ll talk more about bringing types into scope in Chapter 7.

Let’s look at another example of an enum in Listing 6-2: this one has a wide variety of types embedded in its variants.

此代码说明您可以将任何类型的数据放入枚举变体中：例如字符串、数字类型或结构。 您甚至可以包含另一个枚举！ 此外，标准库类型通常并不比您可能想到的复杂多少。

请注意，即使标准库包含 IpAddr 的定义，我们仍然可以创建和使用我们自己的定义而不会发生冲突，因为我们尚未将标准库的定义纳入我们的范围。 我们将在第 7 章中详细讨论将类型纳入作用域。

让我们看一下清单 6-2 中枚举的另一个示例：这个枚举的变体中嵌入了多种类型。
``````
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
``````
Listing 6-2: A Message enum whose variants each store different amounts and types of values

This enum has four variants with different types:

+ Quit has no data associated with it at all.
+ Move has named fields, like a struct does.
+ Write includes a single String.
+ ChangeColor includes three i32 values.
Defining an enum with variants such as the ones in Listing 6-2 is similar to defining different kinds of struct definitions, except the enum doesn’t use the struct keyword and all the variants are grouped together under the Message type. The following structs could hold the same data that the preceding enum variants hold:

该枚举有四种不同类型的变体：

+ Quit 根本没有与之关联的数据。
+ Move 具有命名字段，就像结构一样。
+ Write 包含单个字符串。
+ ChangeColor 包括三个 i32 值。
使用诸如清单 6-2 中的变体定义枚举类似于定义不同类型的结构体定义，只不过枚举不使用 struct 关键字并且所有变体都在 Message 类型下分组在一起。 以下结构可以保存与前面的枚举变体保存的相同数据：

``````
struct QuitMessage; // unit struct
struct MoveMessage {
    x: i32,
    y: i32,
}
struct WriteMessage(String); // tuple struct
struct ChangeColorMessage(i32, i32, i32); // tuple struct
``````
But if we used the different structs, each of which has its own type, we couldn’t as easily define a function to take any of these kinds of messages as we could with the Message enum defined in Listing 6-2, which is a single type.

There is one more similarity between enums and structs: just as we’re able to define methods on structs using impl, we’re also able to define methods on enums. Here’s a method named call that we could define on our Message enum:

但是，如果我们使用不同的结构，每个结构都有自己的类型，我们就无法像清单 6-2 中定义的 Message 枚举那样轻松地定义一个函数来获取任何此类消息，该枚举是一个 单一类型。

枚举和结构之间还有一个相似之处：就像我们能够使用 impl 在结构上定义方法一样，我们也能够在枚举上定义方法。 这是一个名为 call 的方法，我们可以在 Message 枚举上定义它：

``````
    impl Message {
        fn call(&self) {
            // method body would be defined here
        }
    }

    let m = Message::Write(String::from("hello"));
    m.call();
``````
The body of the method would use self to get the value that we called the method on. In this example, we’ve created a variable m that has the value Message::Write(String::from("hello")), and that is what self will be in the body of the call method when m.call() runs.

Let’s look at another enum in the standard library that is very common and useful: Option.

方法的主体将使用 self 来获取我们调用该方法的值。 在此示例中，我们创建了一个变量 m，其值为 Message::Write(String::from("hello"))，这就是当 m.call( ）运行。

让我们看一下标准库中另一个非常常见且有用的枚举：Option。

### The Option Enum and Its Advantages Over Null Values
This section explores a case study of Option, which is another enum defined by the standard library. The Option type encodes the very common scenario in which a value could be something or it could be nothing.

For example, if you request the first item in a non-empty list, you would get a value. If you request the first item in an empty list, you would get nothing. Expressing this concept in terms of the type system means the compiler can check whether you’ve handled all the cases you should be handling; this functionality can prevent bugs that are extremely common in other programming languages.

Programming language design is often thought of in terms of which features you include, but the features you exclude are important too. Rust doesn’t have the null feature that many other languages have. Null is a value that means there is no value there. In languages with null, variables can always be in one of two states: null or not-null.

In his 2009 presentation “Null References: The Billion Dollar Mistake,” Tony Hoare, the inventor of null, has this to say:

本节探讨 Option 的案例研究，它是标准库定义的另一个枚举。 Option 类型对非常常见的场景进行编码，其中值可能是某物，也可能是什么。

例如，如果您请求非空列表中的第一项，您将获得一个值。 如果您请求空列表中的第一项，您将一无所获。 用类型系统来表达这个概念意味着编译器可以检查你是否已经处理了所有应该处理的情况； 此功能可以防止其他编程语言中极其常见的错误。

编程语言设计通常根据包含哪些功能来考虑，但排除哪些功能也很重要。 Rust 没有许多其他语言所具有的 null 功能。 Null 是一个值，意味着那里没有值。 在具有 null 的语言中，变量始终可以处于两种状态之一：null 或 not-null。

null 的发明者托尼·霍尔 (Tony Hoare) 在 2009 年的演讲“空引用：价值数十亿美元的错误”中说道：

``````
I call it my billion-dollar mistake. At that time, I was designing the first comprehensive type system for references in an object-oriented language. My goal was to ensure that all use of references should be absolutely safe, with checking performed automatically by the compiler. But I couldn’t resist the temptation to put in a null reference, simply because it was so easy to implement. This has led to innumerable errors, vulnerabilities, and system crashes, which have probably caused a billion dollars of pain and damage in the last forty years.
``````

The problem with null values is that if you try to use a null value as a not-null value, you’ll get an error of some kind. Because this null or not-null property is pervasive, it’s extremely easy to make this kind of error.

However, the concept that null is trying to express is still a useful one: a null is a value that is currently invalid or absent for some reason.

The problem isn’t really with the concept but with the particular implementation. As such, Rust does not have nulls, but it does have an enum that can encode the concept of a value being present or absent. This enum is Option<T>, and it is defined by the standard library as follows:

空值的问题在于，如果您尝试将空值用作非空值，则会收到某种错误。 由于 null 或非 null 属性非常普遍，因此很容易犯这种错误。

然而，null 试图表达的概念仍然是一个有用的概念：null 是当前无效或由于某种原因不存在的值。

问题实际上不在于概念，而在于特定的实现。 因此，Rust 没有空值，但它有一个枚举，可以对值存在或不存在的概念进行编码。 这个枚举是 Option<T>，它由标准库定义如下：
``````
enum Option<T> {
    None,
    Some(T),
}
``````
The Option<T> enum is so useful that it’s even included in the prelude; you don’t need to bring it into scope explicitly. Its variants are also included in the prelude: you can use Some and None directly without the Option:: prefix. The Option<T> enum is still just a regular enum, and Some(T) and None are still variants of type Option<T>.

The <T> syntax is a feature of Rust we haven’t talked about yet. It’s a generic type parameter, and we’ll cover generics in more detail in Chapter 10. For now, all you need to know is that <T> means that the Some variant of the Option enum can hold one piece of data of any type, and that each concrete type that gets used in place of T makes the overall Option<T> type a different type. Here are some examples of using Option values to hold number types and string types:

Option<T> 枚举非常有用，甚至包含在前奏中； 您不需要明确地将其纳入范围。 它的变体也包含在前奏中：您可以直接使用 Some 和 None ，而不需要 Option:: 前缀。 Option<T> 枚举仍然只是一个常规枚举，Some(T) 和 None 仍然是 Option<T> 类型的变体。

<T> 语法是 Rust 的一个特性，我们还没有讨论过。 它是一个泛型类型参数，我们将在第 10 章中更详细地介绍泛型。现在，您需要知道的是 <T> 意味着 Option 枚举的 Some 变体可以保存任何类型的一条数据 ，并且用于代替 T 的每个具体类型都会使整个 Option<T> 类型成为不同的类型。 以下是使用 Option 值保存数字类型和字符串类型的一些示例：
``````
    let some_number = Some(5);
    let some_char = Some('e');

    let absent_number: Option<i32> = None;
``````
The type of some_number is Option<i32>. The type of some_char is Option<char>, which is a different type. Rust can infer these types because we’ve specified a value inside the Some variant. For absent_number, Rust requires us to annotate the overall Option type: the compiler can’t infer the type that the corresponding Some variant will hold by looking only at a None value. Here, we tell Rust that we mean for absent_number to be of type Option<i32>.

When we have a Some value, we know that a value is present and the value is held within the Some. When we have a None value, in some sense it means the same thing as null: we don’t have a valid value. So why is having Option<T> any better than having null?

In short, because Option<T> and T (where T can be any type) are different types, the compiler won’t let us use an Option<T> value as if it were definitely a valid value. For example, this code won’t compile, because it’s trying to add an i8 to an Option<i8>:

some_number 的类型是 Option<i32>。 some_char 的类型是 Option<char>，这是一种不同的类型。 Rust 可以推断这些类型，因为我们在 Some 变量中指定了一个值。 对于absent_number，Rust 要求我们注释整个 Option 类型：编译器无法仅通过查看 None 值来推断相应 Some 变体将持有的类型。 在这里，我们告诉 Rust，我们的意思是absent_number 的类型为Option<i32>。

当我们有 Some 值时，我们知道存在一个值并且该值保存在 Some 中。 当我们有一个 None 值时，在某种意义上它与 null 意味着同样的事情：我们没有一个有效的值。 那么为什么 Option<T> 比 null 更好呢？

简而言之，因为 Option<T> 和 T（其中 T 可以是任何类型）是不同的类型，所以编译器不会让我们使用 Option<T> 值，就好像它绝对是一个有效值一样。 例如，此代码将无法编译，因为它试图将 i8 添加到 Option<i8>：
``````
This code does not compile!
    let x: i8 = 5;
    let y: Option<i8> = Some(5);

    let sum = x + y;
``````
If we run this code, we get an error message like this one:
``````
$ cargo run
   Compiling enums v0.1.0 (file:///projects/enums)
error[E0277]: cannot add `Option<i8>` to `i8`
 --> src/main.rs:5:17
  |
5 |     let sum = x + y;
  |                 ^ no implementation for `i8 + Option<i8>`
  |
  = help: the trait `Add<Option<i8>>` is not implemented for `i8`
  = help: the following other types implement trait `Add<Rhs>`:
            <&'a i8 as Add<i8>>
            <&i8 as Add<&i8>>
            <i8 as Add<&i8>>
            <i8 as Add>

For more information about this error, try `rustc --explain E0277`.
error: could not compile `enums` due to previous error
``````
Intense! In effect, this error message means that Rust doesn’t understand how to add an i8 and an Option<i8>, because they’re different types. When we have a value of a type like i8 in Rust, the compiler will ensure that we always have a valid value. We can proceed confidently without having to check for null before using that value. Only when we have an Option<i8> (or whatever type of value we’re working with) do we have to worry about possibly not having a value, and the compiler will make sure we handle that case before using the value.

In other words, you have to convert an Option<T> to a T before you can perform T operations with it. Generally, this helps catch one of the most common issues with null: assuming that something isn’t null when it actually is.

Eliminating the risk of incorrectly assuming a not-null value helps you to be more confident in your code. In order to have a value that can possibly be null, you must explicitly opt in by making the type of that value Option<T>. Then, when you use that value, you are required to explicitly handle the case when the value is null. Everywhere that a value has a type that isn’t an Option<T>, you can safely assume that the value isn’t null. This was a deliberate design decision for Rust to limit null’s pervasiveness and increase the safety of Rust code.

So how do you get the T value out of a Some variant when you have a value of type Option<T> so that you can use that value? The Option<T> enum has a large number of methods that are useful in a variety of situations; you can check them out in its documentation. Becoming familiar with the methods on Option<T> will be extremely useful in your journey with Rust.

In general, in order to use an Option<T> value, you want to have code that will handle each variant. You want some code that will run only when you have a Some(T) value, and this code is allowed to use the inner T. You want some other code to run only if you have a None value, and that code doesn’t have a T value available. The match expression is a control flow construct that does just this when used with enums: it will run different code depending on which variant of the enum it has, and that code can use the data inside the matching value.

激烈的！ 实际上，此错误消息意味着 Rust 不理解如何添加 i8 和 Option<i8>，因为它们是不同的类型。 当我们在 Rust 中拥有像 i8 这样的类型的值时，编译器将确保我们始终拥有有效的值。 我们可以放心地继续操作，而无需在使用该值之前检查 null 。 仅当我们有 Option<i8> （或我们正在使用的任何类型的值）时，我们才需要担心可能没有值，并且编译器将确保我们在使用该值之前处理这种情况。

换句话说，您必须先将 Option<T> 转换为 T，然后才能使用它执行 T 操作。 一般来说，这有助于捕获 null 最常见的问题之一：假设某些内容实际上不为 null。

消除错误假设非空值的风险可以帮助您对代码更有信心。 为了获得可能为 null 的值，您必须通过将该值的类型设置为 Option<T> 来显式选择加入。 然后，当您使用该值时，需要显式处理该值为 null 的情况。 只要值的类型不是 Option<T>，您就可以安全地假设该值不为 null。 这是 Rust 特意设计的决定，旨在限制 null 的普遍性并提高 Rust 代码的安全性。

那么，当您有 Option<T> 类型的值时，如何从 Some 变体中获取 T 值以便可以使用该值呢？ Option<T> 枚举具有大量在各种情况下有用的方法； 您可以在其文档中查看它们。 熟悉 Option<T> 上的方法对于您的 Rust 之旅非常有用。

一般来说，为了使用 Option<T> 值，您需要拥有能够处理每个变体的代码。 您希望某些代码仅当您具有 Some(T) 值时才运行，并且允许此代码使用内部 T。您希望某些其他代码仅在您具有 None 值时运行，并且该代码不 有可用的 T 值。 匹配表达式是一个控制流构造，与枚举一起使用时会执行此操作：它将根据其具有的枚举变体运行不同的代码，并且该代码可以使用匹配值内的数据。

## 6.2 使用match控制流结构

Rust has an extremely powerful control flow construct called match that allows you to compare a value against a series of patterns and then execute code based on which pattern matches. Patterns can be made up of literal values, variable names, wildcards, and many other things; Chapter 18 covers all the different kinds of patterns and what they do. The power of match comes from the expressiveness of the patterns and the fact that the compiler confirms that all possible cases are handled.

Think of a match expression as being like a coin-sorting machine: coins slide down a track with variously sized holes along it, and each coin falls through the first hole it encounters that it fits into. In the same way, values go through each pattern in a match, and at the first pattern the value “fits,” the value falls into the associated code block to be used during execution.

Speaking of coins, let’s use them as an example using match! We can write a function that takes an unknown US coin and, in a similar way as the counting machine, determines which coin it is and returns its value in cents, as shown in Listing 6-3.

Rust 有一个非常强大的控制流结构，称为 match，它允许您将一个值与一系列模式进行比较，然后根据模式匹配执行代码。 模式可以由文字值、变量名、通配符和许多其他内容组成； 第 18 章介绍了所有不同类型的模式及其用途。 匹配的力量来自模式的表达能力以及编译器确认所有可能的情况都已处理的事实。

将匹配表达式想象为硬币分类机：硬币沿着带有不同大小孔的轨道滑下，每枚硬币都会从它遇到的第一个适合的孔落下。 以同样的方式，值会遍历匹配中的每个模式，并且在第一个模式中，值“适合”，该值落入要在执行期间使用的关联代码块中。

说到硬币，让我们以它们为例，使用火柴！ 我们可以编写一个函数，接受一枚未知的美国硬币，并以与计数机类似的方式确定它是哪种硬币并返回其价值（以美分为单位），如清单 6-3 所示。
``````
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
``````
Listing 6-3: An enum and a match expression that has the variants of the enum as its patterns

Let’s break down the match in the value_in_cents function. First we list the match keyword followed by an expression, which in this case is the value coin. This seems very similar to a conditional expression used with if, but there’s a big difference: with if, the condition needs to evaluate to a Boolean value, but here it can be any type. The type of coin in this example is the Coin enum that we defined on the first line.

Next are the match arms. An arm has two parts: a pattern and some code. The first arm here has a pattern that is the value Coin::Penny and then the => operator that separates the pattern and the code to run. The code in this case is just the value 1. Each arm is separated from the next with a comma.

When the match expression executes, it compares the resultant value against the pattern of each arm, in order. If a pattern matches the value, the code associated with that pattern is executed. If that pattern doesn’t match the value, execution continues to the next arm, much as in a coin-sorting machine. We can have as many arms as we need: in Listing 6-3, our match has four arms.

The code associated with each arm is an expression, and the resultant value of the expression in the matching arm is the value that gets returned for the entire match expression.

We don’t typically use curly brackets if the match arm code is short, as it is in Listing 6-3 where each arm just returns a value. If you want to run multiple lines of code in a match arm, you must use curly brackets, and the comma following the arm is then optional. For example, the following code prints “Lucky penny!” every time the method is called with a Coin::Penny, but still returns the last value of the block, 1:

让我们分解一下 value_in_cents 函数中的匹配。 首先，我们列出匹配关键字，后跟一个表达式，在本例中是价值硬币。 这看起来与 if 中使用的条件表达式非常相似，但有一个很大的区别：在 if 中，条件需要计算为布尔值，但这里它可以是任何类型。 本例中的硬币类型是我们在第一行定义的 Coin 枚举。

接下来是火柴臂。 手臂有两部分：图案和代码。 这里的第一个分支有一个模式，即值 Coin::Penny，然后是分隔模式和要运行的代码的 => 运算符。 本例中的代码只是值 1。每个臂与下一个臂之间用逗号分隔。

当匹配表达式执行时，它会按顺序将结果值与每个臂的模式进行比较。 如果模式与值匹配，则执行与该模式关联的代码。 如果该模式与值不匹配，则继续执行下一个臂，就像硬币分类机一样。 我们可以根据需要拥有任意数量的手臂：在清单 6-3 中，我们的比赛有四个手臂。

与每个臂关联的代码是一个表达式，匹配臂中表达式的结果值是整个匹配表达式的返回值。

如果匹配臂代码很短，我们通常不会使用大括号，如清单 6-3 所示，每个臂只返回一个值。 如果要在匹配臂中运行多行代码，则必须使用大括号，并且臂后面的逗号是可选的。 例如，以下代码打印“Lucky penny!” 每次使用 Coin::Penny 调用该方法时，但仍返回该块的最后一个值 1：
``````
fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!");
            1
        }
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
``````
### Patterns That Bind to Values
Another useful feature of match arms is that they can bind to the parts of the values that match the pattern. This is how we can extract values out of enum variants.

As an example, let’s change one of our enum variants to hold data inside it. From 1999 through 2008, the United States minted quarters with different designs for each of the 50 states on one side. No other coins got state designs, so only quarters have this extra value. We can add this information to our enum by changing the Quarter variant to include a UsState value stored inside it, which we’ve done in Listing 6-4.

匹配臂的另一个有用的功能是它们可以绑定到与模式匹配的值的部分。 这就是我们从枚举变量中提取值的方法。

作为示例，让我们更改枚举变体之一以在其中保存数据。 从 1999 年到 2008 年，美国为一侧 50 个州铸造了不同设计的 25 美分硬币。 没有其他硬币有国家设计，因此只有 25 美分硬币具有这种额外价值。 我们可以通过更改 Quarter 变量以包含存储在其中的 UsState 值来将此信息添加到我们的枚举中，如清单 6-4 中所示。

``````
#[derive(Debug)] // so we can inspect the state in a minute
enum UsState {
    Alabama,
    Alaska,
    // --snip--
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}
``````
Listing 6-4: A Coin enum in which the Quarter variant also holds a UsState value

Let’s imagine that a friend is trying to collect all 50 state quarters. While we sort our loose change by coin type, we’ll also call out the name of the state associated with each quarter so that if it’s one our friend doesn’t have, they can add it to their collection.

In the match expression for this code, we add a variable called state to the pattern that matches values of the variant Coin::Quarter. When a Coin::Quarter matches, the state variable will bind to the value of that quarter’s state. Then we can use state in the code for that arm, like so:

让我们想象一下，一位朋友正在尝试收集全部 50 个州 25 美分。 当我们按硬币类型对零钱进行分类时，我们还会标出与每个季度相关的州名称，以便如果我们的朋友没有，他们可以将其添加到他们的收藏中。

在此代码的匹配表达式中，我们将一个名为 state 的变量添加到与变体 Coin::Quarter 的值匹配的模式中。 当 Coin::Quarter 匹配时，状态变量将绑定到该季度的状态值。 然后我们可以在该手臂的代码中使用状态，如下所示：

``````
fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        }
    }
}
``````
If we were to call value_in_cents(Coin::Quarter(UsState::Alaska)), coin would be Coin::Quarter(UsState::Alaska). When we compare that value with each of the match arms, none of them match until we reach Coin::Quarter(state). At that point, the binding for state will be the value UsState::Alaska. We can then use that binding in the println! expression, thus getting the inner state value out of the Coin enum variant for Quarter.

如果我们调用 value_in_cents(Coin::Quarter(UsState::Alaska))，coin 将是 Coin::Quarter(UsState::Alaska)。 当我们将该值与每个匹配臂进行比较时，直到我们到达 Coin::Quarter(state) 为止，它们都不匹配。 此时，状态的绑定将是值 UsState::Alaska。 然后我们可以在 println! 中使用该绑定！ 表达式，从而从 Quarter 的 Coin 枚举变体中获取内部状态值。

### Matching with Option<T>
In the previous section, we wanted to get the inner T value out of the Some case when using Option<T>; we can also handle Option<T> using match, as we did with the Coin enum! Instead of comparing coins, we’ll compare the variants of Option<T>, but the way the match expression works remains the same.

Let’s say we want to write a function that takes an Option<i32> and, if there’s a value inside, adds 1 to that value. If there isn’t a value inside, the function should return the None value and not attempt to perform any operations.

在上一节中，我们希望在使用 Option<T> 时从 Some case 中获取内部 T 值； 我们还可以使用 match 来处理 Option<T>，就像我们对 Coin 枚举所做的那样！ 我们将比较 Option<T> 的变体，而不是比较硬币，但匹配表达式的工作方式保持不变。

假设我们要编写一个接受 Option<i32> 的函数，如果里面有值，则将该值加 1。 如果内部没有值，该函数应返回 None 值并且不尝试执行任何操作。

This function is very easy to write, thanks to match, and will look like Listing 6-5.
``````
    fn plus_one(x: Option<i32>) -> Option<i32> {
        match x {
            None => None,
            Some(i) => Some(i + 1),
        }
    }

    let five = Some(5);
    let six = plus_one(five);
    let none = plus_one(None);
``````
Listing 6-5: A function that uses a match expression on an Option<i32>

Let’s examine the first execution of plus_one in more detail. When we call plus_one(five), the variable x in the body of plus_one will have the value Some(5). We then compare that against each match arm:
``````
            None => None,
``````
The Some(5) value doesn’t match the pattern None, so we continue to the next arm:
``````
            Some(i) => Some(i + 1),
``````
Does Some(5) match Some(i)? It does! We have the same variant. The i binds to the value contained in Some, so i takes the value 5. The code in the match arm is then executed, so we add 1 to the value of i and create a new Some value with our total 6 inside.

Now let’s consider the second call of plus_one in Listing 6-5, where x is None. We enter the match and compare to the first arm:
``````
            None => None,
``````
It matches! There’s no value to add to, so the program stops and returns the None value on the right side of =>. Because the first arm matched, no other arms are compared.

Combining match and enums is useful in many situations. You’ll see this pattern a lot in Rust code: match against an enum, bind a variable to the data inside, and then execute code based on it. It’s a bit tricky at first, but once you get used to it, you’ll wish you had it in all languages. It’s consistently a user favorite.

它匹配！ 没有可添加的值，因此程序停止并返回 => 右侧的 None 值。 由于第一个臂已匹配，因此不会比较其他臂。

组合匹配和枚举在许多情况下都很有用。 您会在 Rust 代码中经常看到这种模式：与枚举匹配，将变量绑定到内部数据，然后基于它执行代码。 一开始有点棘手，但一旦习惯了，您会希望拥有所有语言版本。 它一直是用户的最爱。

### Matches Are Exhaustive
There’s one other aspect of match we need to discuss: the arms’ patterns must cover all possibilities. Consider this version of our plus_one function, which has a bug and won’t compile:
``````
This code does not compile!
    fn plus_one(x: Option<i32>) -> Option<i32> {
        match x {
            Some(i) => Some(i + 1),
        }
    }
``````
We didn’t handle the None case, so this code will cause a bug. Luckily, it’s a bug Rust knows how to catch. If we try to compile this code, we’ll get this error:
``````
$ cargo run
   Compiling enums v0.1.0 (file:///projects/enums)
error[E0004]: non-exhaustive patterns: `None` not covered
 --> src/main.rs:3:15
  |
3 |         match x {
  |               ^ pattern `None` not covered
  |
note: `Option<i32>` defined here
 --> /rustc/d5a82bbd26e1ad8b7401f6a718a9c57c96905483/library/core/src/option.rs:518:1
  |
  = note: 
/rustc/d5a82bbd26e1ad8b7401f6a718a9c57c96905483/library/core/src/option.rs:522:5: not covered
  = note: the matched value is of type `Option<i32>`
help: ensure that all possible cases are being handled by adding a match arm with a wildcard pattern or an explicit pattern as shown
  |
4 ~             Some(i) => Some(i + 1),
5 ~             None => todo!(),
  |

For more information about this error, try `rustc --explain E0004`.
error: could not compile `enums` due to previous error
``````
Rust knows that we didn’t cover every possible case, and even knows which pattern we forgot! Matches in Rust are exhaustive: we must exhaust every last possibility in order for the code to be valid. Especially in the case of Option<T>, when Rust prevents us from forgetting to explicitly handle the None case, it protects us from assuming that we have a value when we might have null, thus making the billion-dollar mistake discussed earlier impossible.

Rust 知道我们没有涵盖所有可能的情况，甚至知道我们忘记了哪种模式！ Rust 中的匹配是详尽的：我们必须穷尽所有最后的可能性才能使代码有效。 特别是在 Option<T> 的情况下，当 Rust 防止我们忘记显式处理 None 情况时，它可以防止我们在可能为 null 的情况下假设我们有一个值，从而使之前讨论的数十亿美元的错误成为不可能。

### Catch-all Patterns and the _ Placeholder
Using enums, we can also take special actions for a few particular values, but for all other values take one default action. Imagine we’re implementing a game where, if you roll a 3 on a dice roll, your player doesn’t move, but instead gets a new fancy hat. If you roll a 7, your player loses a fancy hat. For all other values, your player moves that number of spaces on the game board. Here’s a match that implements that logic, with the result of the dice roll hardcoded rather than a random value, and all other logic represented by functions without bodies because actually implementing them is out of scope for this example:

使用枚举，我们还可以对一些特定值采取特殊操作，但对所有其他值采取一种默认操作。 想象一下，我们正在实现一个游戏，如果您掷骰子掷出 3，您的玩家不会移动，而是会获得一顶新的精美帽子。 如果你掷出 7，你的玩家就会失去一顶漂亮的帽子。 对于所有其他值，您的玩家将在游戏板上移动该数量的空格。 这是一个实现该逻辑的匹配，掷骰子的结果是硬编码的而不是随机值，所有其他逻辑都由没有主体的函数表示，因为实际实现它们超出了本示例的范围：

``````
    let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        other => move_player(other),
    }

    fn add_fancy_hat() {}
    fn remove_fancy_hat() {}
    fn move_player(num_spaces: u8) {}
``````
For the first two arms, the patterns are the literal values 3 and 7. For the last arm that covers every other possible value, the pattern is the variable we’ve chosen to name other. The code that runs for the other arm uses the variable by passing it to the move_player function.

This code compiles, even though we haven’t listed all the possible values a u8 can have, because the last pattern will match all values not specifically listed. This catch-all pattern meets the requirement that match must be exhaustive. Note that we have to put the catch-all arm last because the patterns are evaluated in order. If we put the catch-all arm earlier, the other arms would never run, so Rust will warn us if we add arms after a catch-all!

Rust also has a pattern we can use when we want a catch-all but don’t want to use the value in the catch-all pattern: _ is a special pattern that matches any value and does not bind to that value. This tells Rust we aren’t going to use the value, so Rust won’t warn us about an unused variable.

Let’s change the rules of the game: now, if you roll anything other than a 3 or a 7, you must roll again. We no longer need to use the catch-all value, so we can change our code to use _ instead of the variable named other:

对于前两个臂，模式是文字值 3 和 7。对于覆盖所有其他可能值的最后一个臂，模式是我们选择命名为 other 的变量。 为另一只手臂运行的代码通过将变量传递给 move_player 函数来使用该变量。

即使我们没有列出 u8 可以具有的所有可能值，此代码也会编译，因为最后一个模式将匹配所有未特别列出的值。 这种包罗万象的模式满足匹配必须详尽的要求。 请注意，我们必须将全能臂放在最后，因为模式是按顺序评估的。 如果我们提前放置 catch-all 手臂，其他手臂将永远不会运行，因此如果我们在 catch-all 之后添加手臂，Rust 会警告我们！

Rust 还有一个当我们想要包罗万象但又不想使用包罗万象模式中的值时可以使用的模式： _ 是一种特殊模式，可以匹配任何值但不绑定到该值。 这告诉 Rust 我们不会使用该值，因此 Rust 不会警告我们有关未使用的变量。

让我们改变游戏规则：现在，如果你掷出 3 或 7 以外的任何东西，你必须再次掷出。 我们不再需要使用 catch-all 值，因此我们可以更改代码以使用 _ 而不是名为 other 的变量：

``````
    let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        _ => reroll(),
    }

    fn add_fancy_hat() {}
    fn remove_fancy_hat() {}
    fn reroll() {}
``````
This example also meets the exhaustiveness requirement because we’re explicitly ignoring all other values in the last arm; we haven’t forgotten anything.

Finally, we’ll change the rules of the game one more time so that nothing else happens on your turn if you roll anything other than a 3 or a 7. We can express that by using the unit value (the empty tuple type we mentioned in “The Tuple Type” section) as the code that goes with the _ arm:

这个例子也满足详尽性要求，因为我们明确忽略了最后一个臂中的所有其他值； 我们没有忘记任何事情。

最后，我们将再次更改游戏规则，这样，如果您掷出 3 或 7 以外的任何内容，则轮到您时不会发生任何其他情况。我们可以使用单位值（我们提到的空元组类型）来表达这一点 在“元组类型”部分）作为与 _ 臂一起使用的代码：

``````
    let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        _ => (),
    }

    fn add_fancy_hat() {}
    fn remove_fancy_hat() {}
``````
Here, we’re telling Rust explicitly that we aren’t going to use any other value that doesn’t match a pattern in an earlier arm, and we don’t want to run any code in this case.

There’s more about patterns and matching that we’ll cover in Chapter 18. For now, we’re going to move on to the if let syntax, which can be useful in situations where the match expression is a bit wordy.

在这里，我们明确告诉 Rust，我们不会使用与早期分支中的模式不匹配的任何其他值，并且在这种情况下我们不想运行任何代码。

我们将在第 18 章中介绍更多关于模式和匹配的内容。现在，我们将继续讨论 if let 语法，它在匹配表达式有点冗长的情况下非常有用。

## 6.3 使用if let简化控制流
The if let syntax lets you combine if and let into a less verbose way to handle values that match one pattern while ignoring the rest. Consider the program in Listing 6-6 that matches on an Option<u8> value in the config_max variable but only wants to execute code if the value is the Some variant.

if let 语法允许您将 if 和 let 组合成一种不太冗长的方式来处理与一种模式匹配的值，同时忽略其余模式。 考虑清单 6-6 中的程序，它匹配 config_max 变量中的 Option<u8> 值，但只想在该值是 Some 变体时执行代码。

``````
    let config_max = Some(3u8);
    match config_max {
        Some(max) => println!("The maximum is configured to be {}", max),
        _ => (),
    }
``````
Listing 6-6: A match that only cares about executing code when the value is Some

If the value is Some, we print out the value in the Some variant by binding the value to the variable max in the pattern. We don’t want to do anything with the None value. To satisfy the match expression, we have to add _ => () after processing just one variant, which is annoying boilerplate code to add.

如果值为 Some，我们通过将该值绑定到模式中的变量 max 来打印 Some 变体中的值。 我们不想对 None 值做任何事情。 为了满足匹配表达式，我们必须在仅处理一个变体后添加 _ => ()，这是令人讨厌的样板代码。

Instead, we could write this in a shorter way using if let. The following code behaves the same as the match in Listing 6-6:
``````
    let config_max = Some(3u8);
    if let Some(max) = config_max {
        println!("The maximum is configured to be {}", max);
    }
``````
The syntax if let takes a pattern and an expression separated by an equal sign. It works the same way as a match, where the expression is given to the match and the pattern is its first arm. In this case, the pattern is Some(max), and the max binds to the value inside the Some. We can then use max in the body of the if let block in the same way we used max in the corresponding match arm. The code in the if let block isn’t run if the value doesn’t match the pattern.

Using if let means less typing, less indentation, and less boilerplate code. However, you lose the exhaustive checking that match enforces. Choosing between match and if let depends on what you’re doing in your particular situation and whether gaining conciseness is an appropriate trade-off for losing exhaustive checking.

In other words, you can think of if let as syntax sugar for a match that runs code when the value matches one pattern and then ignores all other values.

We can include an else with an if let. The block of code that goes with the else is the same as the block of code that would go with the _ case in the match expression that is equivalent to the if let and else. Recall the Coin enum definition in Listing 6-4, where the Quarter variant also held a UsState value. If we wanted to count all non-quarter coins we see while also announcing the state of the quarters, we could do that with a match expression, like this:

语法 if let 采用由等号分隔的模式和表达式。 它的工作方式与匹配相同，其中表达式被赋予匹配，模式是其第一个臂。 在本例中，模式是 Some(max)，并且 max 绑定到 Some 内的值。 然后，我们可以在 if let 块的主体中使用 max，就像我们在相应的匹配臂中使用 max 一样。 如果值与模式不匹配，则 if let 块中的代码不会运行。

使用 if let 意味着更少的打字、更少的缩进和更少的样板代码。 但是，您会失去匹配强制执行的详尽检查。 在 match 和 if let 之间进行选择取决于您在特定情况下所做的事情，以及获得简洁性是否是失去详尽检查的适当权衡。

换句话说，您可以将 if let 视为匹配的语法糖，当值与一种模式匹配时运行代码，然后忽略所有其他值。

我们可以在 if let 中包含 else。 与 else 一起使用的代码块与与匹配表达式中的 _ case 一起使用的代码块相同，相当于 if let 和 else。 回想一下清单 6-4 中的 Coin 枚举定义，其中 Quarter 变量还包含一个 UsState 值。 如果我们想计算我们看到的所有非 25 分硬币，同时宣布 25 分硬币的状态，我们可以使用匹配表达式来实现，如下所示：
``````
    let mut count = 0;
    match coin {
        Coin::Quarter(state) => println!("State quarter from {:?}!", state),
        _ => count += 1,
    }
``````
Or we could use an if let and else expression, like this:
``````
    let mut count = 0;
    if let Coin::Quarter(state) = coin {
        println!("State quarter from {:?}!", state);
    } else {
        count += 1;
    }
``````
If you have a situation in which your program has logic that is too verbose to express using a match, remember that if let is in your Rust toolbox as well.

如果您的程序的逻辑过于冗长而无法使用匹配来表达，请记住 if let 也在您的 Rust 工具箱中。
## 6.4 Summary

We’ve now covered how to use enums to create custom types that can be one of a set of enumerated values. We’ve shown how the standard library’s Option<T> type helps you use the type system to prevent errors. When enum values have data inside them, you can use match or if let to extract and use those values, depending on how many cases you need to handle.

Your Rust programs can now express concepts in your domain using structs and enums. Creating custom types to use in your API ensures type safety: the compiler will make certain your functions only get values of the type each function expects.

In order to provide a well-organized API to your users that is straightforward to use and only exposes exactly what your users will need, let’s now turn to Rust’s modules.

我们现在已经介绍了如何使用枚举来创建可以是一组枚举值之一的自定义类型。我们已经展示了标准库的Option<T>类型如何帮助您使用类型系统来防止错误。当枚举值中包含数据时，可以使用match或if let来提取和使用这些值，具体取决于需要处理的情况。
您的Rust程序现在可以使用structs和enum在您的域中表达概念。创建要在API中使用的自定义类型可确保类型安全：编译器将确保您的函数只获得每个函数所需类型的值。
为了向您的用户提供一个组织良好的API，它易于使用，并且只公开您的用户需要的东西，现在让我们转向Rust的模块。