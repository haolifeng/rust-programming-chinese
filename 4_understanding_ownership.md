# 4. 理解所有权
Ownership is Rust’s most unique feature and has deep implications for the rest of the language. It enables Rust to make memory safety guarantees without needing a garbage collector, so it’s important to understand how ownership works. In this chapter, we’ll talk about ownership as well as several related features: borrowing, slices, and how Rust lays data out in memory.

所有权是 Rust 最独特的功能，对语言的其他部分有着深远的影响。 它使 Rust 能够在不需要垃圾收集器的情况下保证内存安全，因此了解所有权的工作原理非常重要。 在本章中，我们将讨论所有权以及几个相关功能：借用、切片以及 Rust 如何在内存中布置数据。

## 4.1 什么是所有权
Ownership is a set of rules that govern how a Rust program manages memory. All programs have to manage the way they use a computer’s memory while running. Some languages have garbage collection that regularly looks for no-longer-used memory as the program runs; in other languages, the programmer must explicitly allocate and free the memory. Rust uses a third approach: memory is managed through a system of ownership with a set of rules that the compiler checks. If any of the rules are violated, the program won’t compile. None of the features of ownership will slow down your program while it’s running.

Because ownership is a new concept for many programmers, it does take some time to get used to. The good news is that the more experienced you become with Rust and the rules of the ownership system, the easier you’ll find it to naturally develop code that is safe and efficient. Keep at it!

When you understand ownership, you’ll have a solid foundation for understanding the features that make Rust unique. In this chapter, you’ll learn ownership by working through some examples that focus on a very common data structure: strings.

所有权是一组规则，用于控制 Rust 程序如何管理内存。 所有程序都必须管理它们在运行时使用计算机内存的方式。 有些语言具有垃圾收集功能，会在程序运行时定期查找不再使用的内存； 在其他语言中，程序员必须显式分配和释放内存。 Rust 使用第三种方法：通过所有权系统和编译器检查的一组规则来管理内存。 如果违反任何规则，程序将无法编译。 所有权的任何功能都不会减慢程序运行的速度。

因为所有权对于许多程序员来说是一个新概念，所以确实需要一些时间来适应。 好消息是，您对 Rust 和所有权系统规则的经验越丰富，您就越容易自然地开发出安全高效的代码。 坚持下去！

当您了解所有权时，您将为了解 Rust 独特的功能奠定坚实的基础。 在本章中，您将通过一些示例来学习所有权，这些示例重点关注非常常见的数据结构：字符串。

**The Stack and the Heap**

Many programming languages don’t require you to think about the stack and the heap very often. But in a systems programming language like Rust, whether a value is on the stack or the heap affects how the language behaves and why you have to make certain decisions. Parts of ownership will be described in relation to the stack and the heap later in this chapter, so here is a brief explanation in preparation.

Both the stack and the heap are parts of memory available to your code to use at runtime, but they are structured in different ways. The stack stores values in the order it gets them and removes the values in the opposite order. This is referred to as last in, first out. Think of a stack of plates: when you add more plates, you put them on top of the pile, and when you need a plate, you take one off the top. Adding or removing plates from the middle or bottom wouldn’t work as well! Adding data is called pushing onto the stack, and removing data is called popping off the stack. All data stored on the stack must have a known, fixed size. Data with an unknown size at compile time or a size that might change must be stored on the heap instead.

The heap is less organized: when you put data on the heap, you request a certain amount of space. The memory allocator finds an empty spot in the heap that is big enough, marks it as being in use, and returns a pointer, which is the address of that location. This process is called allocating on the heap and is sometimes abbreviated as just allocating (pushing values onto the stack is not considered allocating). Because the pointer to the heap is a known, fixed size, you can store the pointer on the stack, but when you want the actual data, you must follow the pointer. Think of being seated at a restaurant. When you enter, you state the number of people in your group, and the host finds an empty table that fits everyone and leads you there. If someone in your group comes late, they can ask where you’ve been seated to find you.

Pushing to the stack is faster than allocating on the heap because the allocator never has to search for a place to store new data; that location is always at the top of the stack. Comparatively, allocating space on the heap requires more work because the allocator must first find a big enough space to hold the data and then perform bookkeeping to prepare for the next allocation.

Accessing data in the heap is slower than accessing data on the stack because you have to follow a pointer to get there. Contemporary processors are faster if they jump around less in memory. Continuing the analogy, consider a server at a restaurant taking orders from many tables. It’s most efficient to get all the orders at one table before moving on to the next table. Taking an order from table A, then an order from table B, then one from A again, and then one from B again would be a much slower process. By the same token, a processor can do its job better if it works on data that’s close to other data (as it is on the stack) rather than farther away (as it can be on the heap).

When your code calls a function, the values passed into the function (including, potentially, pointers to data on the heap) and the function’s local variables get pushed onto the stack. When the function is over, those values get popped off the stack.

Keeping track of what parts of code are using what data on the heap, minimizing the amount of duplicate data on the heap, and cleaning up unused data on the heap so you don’t run out of space are all problems that ownership addresses. Once you understand ownership, you won’t need to think about the stack and the heap very often, but knowing that the main purpose of ownership is to manage heap data can help explain why it works the way it does.

栈和堆

许多编程语言并不要求您经常考虑堆栈和堆。 但在像 Rust 这样的系统编程语言中，值是在堆栈上还是在堆上会影响语言的行为方式以及为什么必须做出某些决定。 本章稍后将描述与堆栈和堆相关的所有权部分，因此这里是准备中的简要说明。

堆栈和堆都是可供代码在运行时使用的内存部分，但它们的结构方式不同。 堆栈按照获取值的顺序存储值，并按照相反的顺序删除值。 这称为后进先出。 想象一叠盘子：当你添加更多盘子时，你把它们放在一堆盘子的顶部，当你需要一个盘子时，你从上面拿一个。 从中间或底部添加或移除板也不起作用！ 添加数据称为压入堆栈，删除数据称为从堆栈弹出。 存储在堆栈上的所有数据都必须具有已知的固定大小。 编译时大小未知或大小可能更改的数据必须存储在堆上。

堆的组织性较差：当您将数据放入堆上时，您会请求一定量的空间。 内存分配器在堆中找到一个足够大的空位，将其标记为正在使用，并返回一个指针，即该位置的地址。 这个过程称为在堆上分配，有时缩写为分配（将值推入堆栈不被视为分配）。 因为指向堆的指针是已知的固定大小，所以您可以将指针存储在堆栈上，但是当您需要实际数据时，必须跟随指针。 想象一下坐在一家餐馆里。 当您进入时，请说出您的团体人数，然后主人会找到一张适合每个人的空桌子并带您前往那里。 如果您的团队中有人迟到，他们可以询问您坐在哪里以便找到您。

压入堆栈比在堆上分配更快，因为分配器永远不需要搜索存储新数据的位置； 该位置始终位于堆栈的顶部。 相比之下，在堆上分配空间需要更多的工作，因为分配器必须首先找到足够大的空间来容纳数据，然后进行簿记，为下一次分配做准备。

访问堆中的数据比访问堆栈中的数据慢，因为您必须遵循指针才能到达那里。 如果现代处理器在内存中的跳跃次数更少，那么它们的速度就会更快。 继续类比，考虑餐厅的服务员从许多桌子上点菜。 在转到下一张桌子之前，先在一张桌子上获得所有订单是最有效的。 从 A 表中获取订单，然后从 B 表中获取订单，然后再次从 A 中获取订单，然后再次从 B 中获取订单，这将是一个慢得多的过程。 出于同样的原因，如果处理器处理靠近其他数据（因为它在堆栈上）而不是较远的数据（因为它可以在堆上）的数据，那么它可以更好地完成工作。

当您的代码调用函数时，传递给函数的值（可能包括指向堆上数据的指针）和函数的局部变量被推送到堆栈上。 当函数结束时，这些值将从堆栈中弹出。

跟踪代码的哪些部分正在使用堆上的哪些数据，最大限度地减少堆上的重复数据量，以及清理堆上未使用的数据，这样就不会耗尽空间，这些都是所有权解决的问题。 一旦理解了所有权，您就不需要经常考虑堆栈和堆，但是知道所有权的主要目的是管理堆数据可以帮助解释为什么它会这样工作。

### Ownership Rules
First, let’s take a look at the ownership rules. Keep these rules in mind as we work through the examples that illustrate them:

+ Each value in Rust has an owner.
+ There can only be one owner at a time.
+ When the owner goes out of scope, the value will be dropped.

首先，我们来看看所有权规则。 当我们通过示例来说明这些规则时，请记住这些规则：

+ Rust 中的每个值都有一个所有者。
+ 一次只能有一位所有者。
+ 当所有者超出范围时，该值将被删除。

### Variable Scope
Now that we’re past basic Rust syntax, we won’t include all the fn main() { code in examples, so if you’re following along, make sure to put the following examples inside a main function manually. As a result, our examples will be a bit more concise, letting us focus on the actual details rather than boilerplate code.

As a first example of ownership, we’ll look at the scope of some variables. A scope is the range within a program for which an item is valid. Take the following variable:

现在我们已经了解了基本的 Rust 语法，我们不会在示例中包含所有 fn main() { 代码，因此如果您正在遵循，请确保手动将以下示例放入 main 函数中。 因此，我们的示例将更加简洁，让我们专注于实际细节而不是样板代码。

作为所有权的第一个例子，我们将看看一些变量的范围。 范围是程序内某项有效的范围。 取以下变量：

```
let s = "hello";

```
The variable s refers to a string literal, where the value of the string is hardcoded into the text of our program. The variable is valid from the point at which it’s declared until the end of the current scope. Listing 4-1 shows a program with comments annotating where the variable s would be valid.

变量 s 指的是字符串文字，其中字符串的值被硬编码到我们程序的文本中。 该变量从声明之日起一直有效，直至当前作用域结束。 清单 4-1 显示了一个带有注释的程序，注释了变量 s 的有效位置。
```
    {                      // s is not valid here, it’s not yet declared
        let s = "hello";   // s is valid from this point forward

        // do stuff with s
    }                      // this scope is now over, and s is no longer valid
```
Listing 4-1: A variable and the scope in which it is valid

In other words, there are two important points in time here:

+ When s comes into scope, it is valid.
+ It remains valid until it goes out of scope.

At this point, the relationship between scopes and when variables are valid is similar to that in other programming languages. Now we’ll build on top of this understanding by introducing the String type.

换句话说，这里有两个重要的时间点：

+ 当s进入作用域时，它是有效的。
+ 它一直有效，直到超出范围。

此时，作用域和变量何时有效之间的关系与其他编程语言中的类似。 现在我们将通过介绍 String 类型来建立在这种理解的基础上。

### The String Type
To illustrate the rules of ownership, we need a data type that is more complex than those we covered in the “Data Types” section of Chapter 3. The types covered previously are of a known size, can be stored on the stack and popped off the stack when their scope is over, and can be quickly and trivially copied to make a new, independent instance if another part of code needs to use the same value in a different scope. But we want to look at data that is stored on the heap and explore how Rust knows when to clean up that data, and the String type is a great example.

We’ll concentrate on the parts of String that relate to ownership. These aspects also apply to other complex data types, whether they are provided by the standard library or created by you. We’ll discuss String in more depth in Chapter 8.

We’ve already seen string literals, where a string value is hardcoded into our program. String literals are convenient, but they aren’t suitable for every situation in which we may want to use text. One reason is that they’re immutable. Another is that not every string value can be known when we write our code: for example, what if we want to take user input and store it? For these situations, Rust has a second string type, String. This type manages data allocated on the heap and as such is able to store an amount of text that is unknown to us at compile time. You can create a String from a string literal using the from function, like so:

为了说明所有权规则，我们需要一个比第 3 章“数据类型”部分中介绍的数据类型更复杂的数据类型。前面介绍的类型具有已知的大小，可以存储在堆栈上并弹出 当它们的作用域结束时，堆栈可以被快速而简单地复制以创建一个新的独立实例，如果代码的另一部分需要在不同的作用域中使用相同的值。 但我们想要查看存储在堆上的数据并探索 Rust 如何知道何时清理这些数据，而 String 类型就是一个很好的例子。

我们将重点关注 String 中与所有权相关的部分。 这些方面也适用于其他复杂数据类型，无论它们是由标准库提供还是由您创建。 我们将在第 8 章中更深入地讨论字符串。

我们已经见过字符串文字，其中字符串值被硬编码到我们的程序中。 字符串文字很方便，但它们并不适合我们可能想要使用文本的所有情况。 原因之一是它们是不可变的。 另一个问题是，当我们编写代码时，并不是每个字符串值都是已知的：例如，如果我们想要获取用户输入并存储它怎么办？ 对于这些情况，Rust 有第二种字符串类型，String。 这种类型管理在堆上分配的数据，因此能够存储编译时我们未知的大量文本。 您可以使用 from 函数从字符串文字创建字符串，如下所示：
```
let s = String::from("hello");
```
The double colon :: operator allows us to namespace this particular from function under the String type rather than using some sort of name like string_from. We’ll discuss this syntax more in the “Method Syntax” section of Chapter 5, and when we talk about namespacing with modules in “Paths for Referring to an Item in the Module Tree” in Chapter 7.

双冒号 :: 运算符允许我们在 String 类型下命名这个特定的 from 函数，而不是使用诸如 string_from 之类的某种名称。 我们将在第 5 章的“方法语法”部分以及第 7 章的“引用模块树中项目的路径”中讨论模块的命名空间时详细讨论该语法。

This kind of string can be mutated:
```    
let mut s = String::from("hello");

    s.push_str(", world!"); // push_str() appends a literal to a String

    println!("{}", s); // This will print `hello, world!`
```

So, what’s the difference here? Why can String be mutated but literals cannot? The difference is in how these two types deal with memory.

### Memory and Allocation
In the case of a string literal, we know the contents at compile time, so the text is hardcoded directly into the final executable. This is why string literals are fast and efficient. But these properties only come from the string literal’s immutability. Unfortunately, we can’t put a blob of memory into the binary for each piece of text whose size is unknown at compile time and whose size might change while running the program.

With the String type, in order to support a mutable, growable piece of text, we need to allocate an amount of memory on the heap, unknown at compile time, to hold the contents. This means:

+ The memory must be requested from the memory allocator at runtime.
+ We need a way of returning this memory to the allocator when we’re done with our String.
That first part is done by us: when we call String::from, its implementation requests the memory it needs. This is pretty much universal in programming languages.

However, the second part is different. In languages with a garbage collector (GC), the GC keeps track of and cleans up memory that isn’t being used anymore, and we don’t need to think about it. In most languages without a GC, it’s our responsibility to identify when memory is no longer being used and to call code to explicitly free it, just as we did to request it. Doing this correctly has historically been a difficult programming problem. If we forget, we’ll waste memory. If we do it too early, we’ll have an invalid variable. If we do it twice, that’s a bug too. We need to pair exactly one allocate with exactly one free.

Rust takes a different path: the memory is automatically returned once the variable that owns it goes out of scope. Here’s a version of our scope example from Listing 4-1 using a String instead of a string literal:

对于字符串文字，我们在编译时知道内容，因此文本被直接硬编码到最终的可执行文件中。 这就是字符串文字快速且高效的原因。 但这些属性仅来自字符串文字的不变性。 不幸的是，我们无法为每一段文本放入一块内存到二进制文件中，这些文本的大小在编译时未知，并且在运行程序时其大小可能会发生变化。

对于 String 类型，为了支持可变、可增长的文本片段，我们需要在堆上分配一定量的内存（在编译时未知）来保存内容。 这意味着：

+ 必须在运行时向内存分配器请求内存。
+ 当我们使用完字符串后，我们需要一种将内存返回给分配器的方法。
第一部分是由我们完成的：当我们调用 String::from 时，它的实现会请求它所需的内存。 这在编程语言中几乎是通用的。

然而，第二部分则不同。 在带有垃圾收集器（GC）的语言中，GC 会跟踪并清理不再使用的内存，我们不需要考虑它。 在大多数没有 GC 的语言中，我们有责任识别内存何时不再被使用，并调用代码来显式释放它，就像我们请求它一样。 历史上，正确执行此操作一直是一个困难的编程问题。 如果我们忘记了，我们就会浪费记忆。 如果我们做得太早，我们就会得到一个无效的变量。 如果我们这样做两次，这也是一个错误。 我们需要将一个分配与一个空闲配对。

Rust 采用了不同的路径：一旦拥有内存的变量超出范围，内存就会自动返回。 下面是清单 4-1 中的作用域示例的一个版本，使用了字符串而不是字符串文字：
```
{
        let s = String::from("hello"); // s is valid from this point forward

        // do stuff with s
    }                                  // this scope is now over, and s is no
                                       // longer valid
```

There is a natural point at which we can return the memory our String needs to the allocator: when s goes out of scope. When a variable goes out of scope, Rust calls a special function for us. This function is called drop, and it’s where the author of String can put the code to return the memory. Rust calls drop automatically at the closing curly bracket.

Note: In C++, this pattern of deallocating resources at the end of an item’s lifetime is sometimes called Resource Acquisition Is Initialization (RAII). The drop function in Rust will be familiar to you if you’ve used RAII patterns.

This pattern has a profound impact on the way Rust code is written. It may seem simple right now, but the behavior of code can be unexpected in more complicated situations when we want to have multiple variables use the data we’ve allocated on the heap. Let’s explore some of those situations now.

有一个自然的点，我们可以将 String 所需的内存返回给分配器：当 s 超出范围时。 当变量超出范围时，Rust 会为我们调用一个特殊的函数。 这个函数称为 drop，String 的作者可以在其中放置返回内存的代码。 Rust 调用会自动落在右大括号处。

注意：在 C++ 中，这种在项目生命周期结束时释放资源的模式有时称为资源获取即初始化 (RAII)。 如果您使用过 RAII 模式，那么您会熟悉 Rust 中的 drop 函数。

这种模式对 Rust 代码的编写方式产生了深远的影响。 现在看起来可能很简单，但在更复杂的情况下，当我们想让多个变量使用我们在堆上分配的数据时，代码的行为可能会出乎意料。 现在让我们探讨其中的一些情况。

### Variables and Data Interacting with Move
Multiple variables can interact with the same data in different ways in Rust. Let’s look at an example using an integer in Listing 4-2.

在 Rust 中，多个变量可以以不同的方式与相同的数据交互。 让我们看一下清单 4-2 中使用整数的示例。
```
 let x = 5;
    let y = x;
```

Listing 4-2: Assigning the integer value of variable x to y

We can probably guess what this is doing: “bind the value 5 to x; then make a copy of the value in x and bind it to y.” We now have two variables, x and y, and both equal 5. This is indeed what is happening, because integers are simple values with a known, fixed size, and these two 5 values are pushed onto the stack.

我们大概可以猜到这是在做什么：“将值 5 绑定到 x； 然后复制 x 中的值并将其绑定到 y。” 现在我们有两个变量 x 和 y，并且都等于 5。这确实是正在发生的情况，因为整数是具有已知固定大小的简单值，并且这两个 5 值被压入堆栈。

Now let’s look at the String version:
```
    let s1 = String::from("hello");
    let s2 = s1;
```

This looks very similar, so we might assume that the way it works would be the same: that is, the second line would make a copy of the value in s1 and bind it to s2. But this isn’t quite what happens.

Take a look at Figure 4-1 to see what is happening to String under the covers. A String is made up of three parts, shown on the left: a pointer to the memory that holds the contents of the string, a length, and a capacity. This group of data is stored on the stack. On the right is the memory on the heap that holds the contents.

这看起来非常相似，因此我们可以假设它的工作方式是相同的：也就是说，第二行将复制 s1 中的值并将其绑定到 s2。 但实际情况并非如此。

看一下图 4-1，看看 String 到底发生了什么。 字符串由三部分组成，如左图所示：指向保存字符串内容的内存的指针、长度和容量。 这组数据存放在栈中。 右侧是堆上保存内容的内存。

"缺图片"

Figure 4-1: Representation in memory of a String holding the value "hello" bound to s1

The length is how much memory, in bytes, the contents of the String are currently using. The capacity is the total amount of memory, in bytes, that the String has received from the allocator. The difference between length and capacity matters, but not in this context, so for now, it’s fine to ignore the capacity.

When we assign s1 to s2, the String data is copied, meaning we copy the pointer, the length, and the capacity that are on the stack. We do not copy the data on the heap that the pointer refers to. In other words, the data representation in memory looks like Figure 4-2.

长度是字符串的内容当前使用的内存量（以字节为单位）。 容量是字符串从分配器接收的内存总量（以字节为单位）。 长度和容量之间的差异很重要，但在这种情况下并不重要，所以现在可以忽略容量。

当我们将 s1 分配给 s2 时，字符串数据被复制，这意味着我们复制堆栈上的指针、长度和容量。 我们不会复制指针引用的堆上的数据。 换句话说，内存中的数据表示如图 4-2 所示。

“缺图片”

Figure 4-2: Representation in memory of the variable s2 that has a copy of the pointer, length, and capacity of s1

The representation does not look like Figure 4-3, which is what memory would look like if Rust instead copied the heap data as well. If Rust did this, the operation s2 = s1 could be very expensive in terms of runtime performance if the data on the heap were large.

该表示形式与图 4-3 不同，如果 Rust 也复制堆数据，内存就会是什么样子。 如果 Rust 这样做，如果堆上的数据很大，则操作 s2 = s1 在运行时性能方面可能会非常昂贵。

“缺图片”

Figure 4-3: Another possibility for what s2 = s1 might do if Rust copied the heap data as well

Earlier, we said that when a variable goes out of scope, Rust automatically calls the drop function and cleans up the heap memory for that variable. But Figure 4-2 shows both data pointers pointing to the same location. This is a problem: when s2 and s1 go out of scope, they will both try to free the same memory. This is known as a double free error and is one of the memory safety bugs we mentioned previously. Freeing memory twice can lead to memory corruption, which can potentially lead to security vulnerabilities.

To ensure memory safety, after the line let s2 = s1;, Rust considers s1 as no longer valid. Therefore, Rust doesn’t need to free anything when s1 goes out of scope. Check out what happens when you try to use s1 after s2 is created; it won’t work:

之前，我们说过，当变量超出范围时，Rust 会自动调用 drop 函数并清理该变量的堆内存。 但图 4-2 显示两个数据指针都指向同一位置。 这是一个问题：当 s2 和 s1 超出范围时，它们都会尝试释放相同的内存。 这称为双重释放错误，是我们之前提到的内存安全错误之一。 两次释放内存可能会导致内存损坏，从而可能导致安全漏洞。

为了确保内存安全，在 let s2 = s1; 行之后，Rust 认为 s1 不再有效。 因此，当 s1 超出范围时，Rust 不需要释放任何东西。 看看在创建 s2 后尝试使用 s1 时会发生什么情况； 它不会工作：

This code does not compile!
```
    let s1 = String::from("hello");
    let s2 = s1;

    println!("{}, world!", s1);
 ```   
You’ll get an error like this because Rust prevents you from using the invalidated reference:
```

$ cargo run
   Compiling ownership v0.1.0 (file:///projects/ownership)
error[E0382]: borrow of moved value: `s1`
 --> src/main.rs:5:28
  |
2 |     let s1 = String::from("hello");
  |         -- move occurs because `s1` has type `String`, which does not implement the `Copy` trait
3 |     let s2 = s1;
  |              -- value moved here
4 |
5 |     println!("{}, world!", s1);
  |                            ^^ value borrowed here after move
  |
  = note: this error originates in the macro `$crate::format_args_nl` which comes from the expansion of the macro `println` (in Nightly builds, run with -Z macro-backtrace for more info)
help: consider cloning the value if the performance cost is acceptable
  |
3 |     let s2 = s1.clone();
  |                ++++++++
```
For more information about this error, try `rustc --explain E0382`.
error: could not compile `ownership` due to previous error
If you’ve heard the terms shallow copy and deep copy while working with other languages, the concept of copying the pointer, length, and capacity without copying the data probably sounds like making a shallow copy. But because Rust also invalidates the first variable, instead of being called a shallow copy, it’s known as a move. In this example, we would say that s1 was moved into s2. So, what actually happens is shown in Figure 4-4.

Figure 4-4: Representation in memory after s1 has been invalidated

That solves our problem! With only s2 valid, when it goes out of scope it alone will free the memory, and we’re done.

In addition, there’s a design choice that’s implied by this: Rust will never automatically create “deep” copies of your data. Therefore, any automatic copying can be assumed to be inexpensive in terms of runtime performance.

有关此错误的更多信息，请尝试“rustc --explain E0382”。
错误：由于之前的错误，无法编译“所有权”
如果您在使用其他语言时听说过浅表复制和深表复制这两个术语，那么复制指针、长度和容量而不复制数据的概念可能听起来像是浅表复制。 但因为 Rust 也会使第一个变量无效，所以它不被称为浅拷贝，而是被称为移动。 在此示例中，我们会说 s1 已移至 s2 中。 因此，实际发生的情况如图 4-4 所示。

图 4-4：s1 失效后内存中的表示

这解决了我们的问题！ 只有 s2 有效，当它超出范围时，它就会单独释放内存，我们就完成了。

此外，这还暗示着一个设计选择：Rust 永远不会自动创建数据的“深层”副本。 因此，就运行时性能而言，任何自动复制都可以被认为是廉价的。

### Variables and Data Interacting with Clone
If we do want to deeply copy the heap data of the String, not just the stack data, we can use a common method called clone. We’ll discuss method syntax in Chapter 5, but because methods are a common feature in many programming languages, you’ve probably seen them before.

如果我们确实想要深度复制String的堆数据，而不仅仅是堆栈数据，我们可以使用一个常见的方法，称为clone。 我们将在第 5 章中讨论方法语法，但由于方法是许多编程语言的常见功能，因此您以前可能已经见过它们。

Here’s an example of the clone method in action:
```
    let s1 = String::from("hello");
    let s2 = s1.clone();

    println!("s1 = {}, s2 = {}", s1, s2);
 ```   
This works just fine and explicitly produces the behavior shown in Figure 4-3, where the heap data does get copied.

When you see a call to clone, you know that some arbitrary code is being executed and that code may be expensive. It’s a visual indicator that something different is going on.

### Stack-Only Data: Copy
There’s another wrinkle we haven’t talked about yet. This code using integers—part of which was shown in Listing 4-2—works and is valid:
```


    let x = 5;
    let y = x;

    println!("x = {}, y = {}", x, y);
 ```   
But this code seems to contradict what we just learned: we don’t have a call to clone, but x is still valid and wasn’t moved into y.

The reason is that types such as integers that have a known size at compile time are stored entirely on the stack, so copies of the actual values are quick to make. That means there’s no reason we would want to prevent x from being valid after we create the variable y. In other words, there’s no difference between deep and shallow copying here, so calling clone wouldn’t do anything different from the usual shallow copying, and we can leave it out.

Rust has a special annotation called the Copy trait that we can place on types that are stored on the stack, as integers are (we’ll talk more about traits in Chapter 10). If a type implements the Copy trait, variables that use it do not move, but rather are trivially copied, making them still valid after assignment to another variable.

Rust won’t let us annotate a type with Copy if the type, or any of its parts, has implemented the Drop trait. If the type needs something special to happen when the value goes out of scope and we add the Copy annotation to that type, we’ll get a compile-time error. To learn about how to add the Copy annotation to your type to implement the trait, see “Derivable Traits” in Appendix C.

So, what types implement the Copy trait? You can check the documentation for the given type to be sure, but as a general rule, any group of simple scalar values can implement Copy, and nothing that requires allocation or is some form of resource can implement Copy. Here are some of the types that implement Copy:

+ All the integer types, such as u32.
+ The Boolean type, bool, with values true and false.
+ All the floating-point types, such as f64.
+ The character type, char.
+ Tuples, if they only contain types that also implement Copy. For example, (i32, i32) implements Copy, but (i32, String) does not.

但这段代码似乎与我们刚刚了解到的内容相矛盾：我们没有调用克隆，但 x 仍然有效并且没有移动到 y 中。

原因是在编译时具有已知大小的整数等类型完全存储在堆栈中，因此可以快速创建实际值的副本。 这意味着我们没有理由在创建变量 y 后阻止 x 有效。 换句话说，这里的深拷贝和浅拷贝没有区别，所以调用clone不会做任何与通常的浅拷贝不同的事情，我们可以省略它。

Rust 有一个称为 Copy 特征的特殊注释，我们可以将其放置在存储在堆栈中的类型上，就像整数一样（我们将在第 10 章中详细讨论特征）。 如果某个类型实现了 Copy 特征，则使用它的变量不会移动，而是会被简单地复制，从而使它们在分配给另一个变量后仍然有效。

如果类型或其任何部分实现了 Drop 特征，Rust 不会让我们用 Copy 注释类型。 如果当值超出范围时类型需要发生一些特殊的事情，并且我们向该类型添加 Copy 注释，我们将收到编译时错误。 要了解如何将 Copy 注释添加到您的类型以实现特征，请参阅附录 C 中的“可导出特征”。

那么，哪些类型实现了 Copy 特征呢？ 您可以检查给定类型的文档来确定，但作为一般规则，任何一组简单标量值都可以实现 Copy，并且任何需要分配或某种形式的资源都可以实现 Copy。 以下是一些实现 Copy 的类型：

+ 所有整数类型，例如 u32。
+ 布尔类型 bool，值为 true 和 false。
+ 所有浮点类型，例如 f64。
+ 字符类型，char。
+ 元组，如果它们仅包含也实现 Copy 的类型。 例如，(i32, i32) 实现 Copy，但 (i32, String) 则不实现。

### Ownership and Functions
The mechanics of passing a value to a function are similar to those when assigning a value to a variable. Passing a variable to a function will move or copy, just as assignment does. Listing 4-3 has an example with some annotations showing where variables go into and out of scope.

将值传递给函数的机制与将值分配给变量时的机制类似。 将变量传递给函数将会移动或复制，就像赋值一样。 清单 4-3 有一个示例，其中一些注释显示了变量进入和超出范围的位置。

Filename: src/main.rs

```
fn main() {
    let s = String::from("hello");  // s comes into scope

    takes_ownership(s);             // s's value moves into the function...
                                    // ... and so is no longer valid here

    let x = 5;                      // x comes into scope

    makes_copy(x);                  // x would move into the function,
                                    // but i32 is Copy, so it's okay to still
                                    // use x afterward

} // Here, x goes out of scope, then s. But because s's value was moved, nothing
  // special happens.

fn takes_ownership(some_string: String) { // some_string comes into scope
    println!("{}", some_string);
} // Here, some_string goes out of scope and `drop` is called. The backing
  // memory is freed.

fn makes_copy(some_integer: i32) { // some_integer comes into scope
    println!("{}", some_integer);
} // Here, some_integer goes out of scope. Nothing special happens.
```

Listing 4-3: Functions with ownership and scope annotated

If we tried to use s after the call to takes_ownership, Rust would throw a compile-time error. These static checks protect us from mistakes. Try adding code to main that uses s and x to see where you can use them and where the ownership rules prevent you from doing so.

如果我们在调用 gets_ownership 之后尝试使用 s，Rust 会抛出编译时错误。 这些静态检查可以保护我们免受错误的影响。 尝试将使用 s 和 x 的代码添加到 main 中，看看可以在哪里使用它们以及所有权规则在哪里禁止您这样做。

### Return Values and Scope
Returning values can also transfer ownership. Listing 4-4 shows an example of a function that returns some value, with similar annotations as those in Listing 4-3.

Filename: src/main.rs
```
fn main() {
    let s1 = gives_ownership();         // gives_ownership moves its return
                                        // value into s1

    let s2 = String::from("hello");     // s2 comes into scope

    let s3 = takes_and_gives_back(s2);  // s2 is moved into
                                        // takes_and_gives_back, which also
                                        // moves its return value into s3
} // Here, s3 goes out of scope and is dropped. s2 was moved, so nothing
  // happens. s1 goes out of scope and is dropped.

fn gives_ownership() -> String {             // gives_ownership will move its
                                             // return value into the function
                                             // that calls it

    let some_string = String::from("yours"); // some_string comes into scope

    some_string                              // some_string is returned and
                                             // moves out to the calling
                                             // function
}

// This function takes a String and returns one
fn takes_and_gives_back(a_string: String) -> String { // a_string comes into
                                                      // scope

    a_string  // a_string is returned and moves out to the calling function
}
```
Listing 4-4: Transferring ownership of return values

The ownership of a variable follows the same pattern every time: assigning a value to another variable moves it. When a variable that includes data on the heap goes out of scope, the value will be cleaned up by drop unless ownership of the data has been moved to another variable.

While this works, taking ownership and then returning ownership with every function is a bit tedious. What if we want to let a function use a value but not take ownership? It’s quite annoying that anything we pass in also needs to be passed back if we want to use it again, in addition to any data resulting from the body of the function that we might want to return as well.

Rust does let us return multiple values using a tuple, as shown in Listing 4-5.

变量的所有权每次都遵循相同的模式：将值分配给另一个变量会移动它。 当包含堆上数据的变量超出范围时，该值将被删除清理，除非数据的所有权已移至另一个变量。

虽然这可行，但获取所有权然后返回每个函数的所有权有点乏味。 如果我们想让一个函数使用一个值但不获取所有权怎么办？ 非常烦人的是，如果我们想再次使用我们传入的任何内容，除了我们可能想要返回的函数体产生的任何数据之外，还需要传回它。

Rust 确实允许我们使用元组返回多个值，如清单 4-5 所示。

Filename: src/main.rs

```
fn main() {
    let s1 = String::from("hello");

    let (s2, len) = calculate_length(s1);

    println!("The length of '{}' is {}.", s2, len);
}

fn calculate_length(s: String) -> (String, usize) {
    let length = s.len(); // len() returns the length of a String

    (s, length)
}
```
Listing 4-5: Returning ownership of parameters

But this is too much ceremony and a lot of work for a concept that should be common. Luckily for us, Rust has a feature for using a value without transferring ownership, called references.

但对于一个应该通用的概念来说，这太过仪式和大量工作。 对我们来说幸运的是，Rust 有一个使用值而不转移所有权的功能，称为引用。

## 4.2 引用和借用(References and Borrowing)


The issue with the tuple code in Listing 4-5 is that we have to return the String to the calling function so we can still use the String after the call to calculate_length, because the String was moved into calculate_length. Instead, we can provide a reference to the String value. A reference is like a pointer in that it’s an address we can follow to access the data stored at that address; that data is owned by some other variable. Unlike a pointer, a reference is guaranteed to point to a valid value of a particular type for the life of that reference.

Here is how you would define and use a calculate_length function that has a reference to an object as a parameter instead of taking ownership of the value:

清单 4-5 中元组代码的问题在于，我们必须将字符串返回给调用函数，这样我们在调用calculate_length之后仍然可以使用该字符串，因为该字符串已移至calculate_length中。 相反，我们可以提供对 String 值的引用。 引用就像一个指针，因为它是一个地址，我们可以遵循该地址来访问存储在该地址的数据； 该数据由其他一些变量拥有。 与指针不同，引用保证在该引用的生命周期内指向特定类型的有效值。

以下是定义和使用calculate_length函数的方法，该函数将对对象的引用作为参数，而不是获取值的所有权：

Filename: src/main.rs

```
fn main() {
    let s1 = String::from("hello");

    let len = calculate_length(&s1);

    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```
First, notice that all the tuple code in the variable declaration and the function return value is gone. Second, note that we pass &s1 into calculate_length and, in its definition, we take &String rather than String. These ampersands represent references, and they allow you to refer to some value without taking ownership of it. Figure 4-5 depicts this concept.

```
缺图

```
Note: The opposite of referencing by using & is dereferencing, which is accomplished with the dereference operator, *. We’ll see some uses of the dereference operator in Chapter 8 and discuss details of dereferencing in Chapter 15.
Let’s take a closer look at the function call here:

```
    let s1 = String::from("hello");

    let len = calculate_length(&s1);

```
The &s1 syntax lets us create a reference that refers to the value of s1 but does not own it. Because it does not own it, the value it points to will not be dropped when the reference stops being used.

Likewise, the signature of the function uses & to indicate that the type of the parameter s is a reference. Let’s add some explanatory annotations:
```
fn calculate_length(s: &String) -> usize { // s is a reference to a String
    s.len()
} // Here, s goes out of scope. But because it does not have ownership of what
  // it refers to, it is not dropped.
```
The scope in which the variable s is valid is the same as any function parameter’s scope, but the value pointed to by the reference is not dropped when s stops being used, because s doesn’t have ownership. When functions have references as parameters instead of the actual values, we won’t need to return the values in order to give back ownership, because we never had ownership.

We call the action of creating a reference borrowing. As in real life, if a person owns something, you can borrow it from them. When you’re done, you have to give it back. You don’t own it.

So, what happens if we try to modify something we’re borrowing? Try the code in Listing 4-6. Spoiler alert: it doesn’t work!

变量 s 的有效范围与任何函数参数的范围相同，但当 s 停止使用时，引用指向的值不会被删除，因为 s 没有所有权。 当函数将引用而不是实际值作为参数时，我们不需要返回值来归还所有权，因为我们从未拥有所有权。

我们将创建引用的操作称为借用。 就像在现实生活中一样，如果一个人拥有某样东西，你可以向他们借用。 当你完成后，你必须把它还给你。 你不拥有它。

那么，如果我们尝试修改借用的东西会发生什么？ 尝试清单 4-6 中的代码。 剧透警告：这不起作用！

Filename: src/main.rs
```
fn main() {
    let s = String::from("hello");

    change(&s);
}

fn change(some_string: &String) {
    some_string.push_str(", world");
}
```

Listing 4-6: Attempting to modify a borrowed value

Here’s the error:
```
$ cargo run
   Compiling ownership v0.1.0 (file:///projects/ownership)
error[E0596]: cannot borrow `*some_string` as mutable, as it is behind a `&` reference
 --> src/main.rs:8:5
  |
7 | fn change(some_string: &String) {
  |                        ------- help: consider changing this to be a mutable reference: `&mut String`
8 |     some_string.push_str(", world");
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ `some_string` is a `&` reference, so the data it refers to cannot be borrowed as mutable

For more information about this error, try `rustc --explain E0596`.
error: could not compile `ownership` due to previous error
```
Just as variables are immutable by default, so are references. We’re not allowed to modify something we have a reference to.

### Mutable References

We can fix the code from Listing 4-6 to allow us to modify a borrowed value with just a few small tweaks that use, instead, a mutable reference:

Filename: src/main.rs
```
fn main() {
    let mut s = String::from("hello");

    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```
First we change s to be mut. Then we create a mutable reference with &mut s where we call the change function, and update the function signature to accept a mutable reference with some_string: &mut String. This makes it very clear that the change function will mutate the value it borrows.

Mutable references have one big restriction: if you have a mutable reference to a value, you can have no other references to that value. This code that attempts to create two mutable references to s will fail:

首先我们将 s 更改为 mut。 然后，我们使用 &mut s 创建一个可变引用，在其中调用更改函数，并更新函数签名以接受使用 some_string: &mut String 的可变引用。 这非常清楚地表明，更改函数将改变它借用的值。

可变引用有一个很大的限制：如果您有一个对某个值的可变引用，则不能有对该值的其他引用。 尝试创建两个对 s 的可变引用的代码将失败：

Filename: src/main.rs
```
    let mut s = String::from("hello");

    let r1 = &mut s;
    let r2 = &mut s;

    println!("{}, {}", r1, r2);
```
Here’s the error:
```
$ cargo run
   Compiling ownership v0.1.0 (file:///projects/ownership)
error[E0499]: cannot borrow `s` as mutable more than once at a time
 --> src/main.rs:5:14
  |
4 |     let r1 = &mut s;
  |              ------ first mutable borrow occurs here
5 |     let r2 = &mut s;
  |              ^^^^^^ second mutable borrow occurs here
6 |
7 |     println!("{}, {}", r1, r2);
  |                        -- first borrow later used here

For more information about this error, try `rustc --explain E0499`.
error: could not compile `ownership` due to previous error
```
This error says that this code is invalid because we cannot borrow s as mutable more than once at a time. The first mutable borrow is in r1 and must last until it’s used in the println!, but between the creation of that mutable reference and its usage, we tried to create another mutable reference in r2 that borrows the same data as r1.

The restriction preventing multiple mutable references to the same data at the same time allows for mutation but in a very controlled fashion. It’s something that new Rustaceans struggle with because most languages let you mutate whenever you’d like. The benefit of having this restriction is that Rust can prevent data races at compile time. A data race is similar to a race condition and happens when these three behaviors occur:

+ Two or more pointers access the same data at the same time.
+ At least one of the pointers is being used to write to the data.
+ There’s no mechanism being used to synchronize access to the data.
Data races cause undefined behavior and can be difficult to diagnose and fix when you’re trying to track them down at runtime; Rust prevents this problem by refusing to compile code with data races!

As always, we can use curly brackets to create a new scope, allowing for multiple mutable references, just not simultaneous ones:

此错误表明此代码无效，因为我们一次不能多次借用 s 作为可变对象。 第一个可变借用位于 r1 中，并且必须持续到在 println! 中使用为止，但在该可变引用的创建和使用之间，我们尝试在 r2 中创建另一个可变引用，借用与 r1 相同的数据。

防止同时对同一数据进行多个可变引用的限制允许突变，但以非常受控的方式进行。 这是新 Rustaceans 所面临的难题，因为大多数语言都允许你随时进行变异。 具有此限制的好处是 Rust 可以防止编译时的数据竞争。 数据竞争与竞争条件类似，当发生以下三种行为时就会发生：

+ 两个或多个指针同时访问相同的数据。
+ 至少有一个指针用于写入数据。
+ 没有使用任何机制来同步对数据的访问。
数据争用会导致未定义的行为，并且当您尝试在运行时追踪数据争用时，可能很难诊断和修复； Rust 通过拒绝编译带有数据竞争的代码来防止这个问题！

与往常一样，我们可以使用大括号创建一个新范围，允许多个可变引用，但不能同时引用：

```
    let mut s = String::from("hello");

    {
        let r1 = &mut s;
    } // r1 goes out of scope here, so we can make a new reference with no problems.

    let r2 = &mut s;
```
Here’s the error:
```
$ cargo run
   Compiling ownership v0.1.0 (file:///projects/ownership)
error[E0502]: cannot borrow `s` as mutable because it is also borrowed as immutable
 --> src/main.rs:6:14
  |
4 |     let r1 = &s; // no problem
  |              -- immutable borrow occurs here
5 |     let r2 = &s; // no problem
6 |     let r3 = &mut s; // BIG PROBLEM
  |              ^^^^^^ mutable borrow occurs here
7 |
8 |     println!("{}, {}, and {}", r1, r2, r3);
  |                                -- immutable borrow later used here

For more information about this error, try `rustc --explain E0502`.
error: could not compile `ownership` due to previous error
```
Whew! We also cannot have a mutable reference while we have an immutable one to the same value.

Users of an immutable reference don’t expect the value to suddenly change out from under them! However, multiple immutable references are allowed because no one who is just reading the data has the ability to affect anyone else’s reading of the data.

Note that a reference’s scope starts from where it is introduced and continues through the last time that reference is used. For instance, this code will compile because the last usage of the immutable references, the println!, occurs before the mutable reference is introduced:

哇！ 当我们拥有相同值的不可变引用时，我们也不能拥有可变引用。

不可变引用的用户不会期望其值会突然发生变化！ 然而，多个不可变引用是允许的，因为仅仅读取数据的人没有能力影响其他人对数据的读取。

请注意，引用的范围从引入它的地方开始，一直持续到上次使用该引用时为止。 例如，此代码将进行编译，因为最后一次使用不可变引用 println! 发生在引入可变引用之前：

```
let mut s = String::from("hello");

    let r1 = &s; // no problem
    let r2 = &s; // no problem
    println!("{} and {}", r1, r2);
    // variables r1 and r2 will not be used after this point

    let r3 = &mut s; // no problem
    println!("{}", r3);
```
The scopes of the immutable references r1 and r2 end after the println! where they are last used, which is before the mutable reference r3 is created. These scopes don’t overlap, so this code is allowed: the compiler can tell that the reference is no longer being used at a point before the end of the scope.

Even though borrowing errors may be frustrating at times, remember that it’s the Rust compiler pointing out a potential bug early (at compile time rather than at runtime) and showing you exactly where the problem is. Then you don’t have to track down why your data isn’t what you thought it was.

不可变引用 r1 和 r2 的范围在 println! 之后结束。 它们最后一次使用的位置是在创建可变引用 r3 之前。 这些作用域不重叠，因此允许使用此代码：编译器可以判断在作用域结束之前的某个点不再使用该引用。

尽管借用错误有时可能会令人沮丧，但请记住，Rust 编译器会尽早指出潜在的错误（在编译时而不是运行时），并向您准确显示问题所在。 这样您就不必追查为什么您的数据与您想象的不同。

### Dangling References

In languages with pointers, it’s easy to erroneously create a dangling pointer—a pointer that references a location in memory that may have been given to someone else—by freeing some memory while preserving a pointer to that memory. In Rust, by contrast, the compiler guarantees that references will never be dangling references: if you have a reference to some data, the compiler will ensure that the data will not go out of scope before the reference to the data does.

在带有指针的语言中，通过释放一些内存同时保留指向该内存的指针，很容易错误地创建一个悬空指针（引用内存中可能已分配给其他人的位置的指针）。 相比之下，在 Rust 中，编译器保证引用永远不会是悬空引用：如果您引用了某些数据，编译器将确保数据不会在数据引用超出范围之前超出范围。

Let’s try to create a dangling reference to see how Rust prevents them with a compile-time error:

Filename: src/main.rs
```
fn main() {
    let reference_to_nothing = dangle();
}

fn dangle() -> &String {
    let s = String::from("hello");

    &s
}
```

Here’s the error:
```
$ cargo run
   Compiling ownership v0.1.0 (file:///projects/ownership)
error[E0106]: missing lifetime specifier
 --> src/main.rs:5:16
  |
5 | fn dangle() -> &String {
  |                ^ expected named lifetime parameter
  |
  = help: this function's return type contains a borrowed value, but there is no value for it to be borrowed from
help: consider using the `'static` lifetime
  |
5 | fn dangle() -> &'static String {
  |                 +++++++

For more information about this error, try `rustc --explain E0106`.
error: could not compile `ownership` due to previous error
```
This error message refers to a feature we haven’t covered yet: lifetimes. We’ll discuss lifetimes in detail in Chapter 10. But, if you disregard the parts about lifetimes, the message does contain the key to why this code is a problem:

此错误消息涉及我们尚未介绍的功能：生命周期。 我们将在第 10 章中详细讨论生命周期。但是，如果您忽略有关生命周期的部分，该消息确实包含了为什么此代码存在问题的关键：
```
this function's return type contains a borrowed value, but there is no value
for it to be borrowed from
```
Let’s take a closer look at exactly what’s happening at each stage of our dangle code:

Filename: src/main.rs
```
This code does not compile!
fn dangle() -> &String { // dangle returns a reference to a String

    let s = String::from("hello"); // s is a new String

    &s // we return a reference to the String, s
} // Here, s goes out of scope, and is dropped. Its memory goes away.
  // Danger!
```
Because s is created inside dangle, when the code of dangle is finished, s will be deallocated. But we tried to return a reference to it. That means this reference would be pointing to an invalid String. That’s no good! Rust won’t let us do this.

因为 s 是在 dangle 内部创建的，所以当 dangle 的代码完成时，s 将被释放。 但我们试图返回对它的引用。 这意味着该引用将指向无效的字符串。 这样可不行啊！ Rust 不会让我们这样做。

The solution here is to return the String directly:
```
fn no_dangle() -> String {
    let s = String::from("hello");

    s
}
```
This works without any problems. Ownership is moved out, and nothing is deallocated.

### The Rules of References
Let’s recap what we’ve discussed about references:

+ At any given time, you can have either one mutable reference or any number of immutable references.
+ References must always be valid.

Next, we’ll look at a different kind of reference: slices.

让我们回顾一下我们讨论过的关于参考文献的内容：

+ 在任何给定时间，您可以拥有一个可变引用或任意数量的不可变引用。
+ 参考文献必须始终有效。

接下来，我们将看看另一种不同的参考：切片。

## 4.3 切片类型(the Slice type)

Slices let you reference a contiguous sequence of elements in a collection rather than the whole collection. A slice is a kind of reference, so it does not have ownership.

Here’s a small programming problem: write a function that takes a string of words separated by spaces and returns the first word it finds in that string. If the function doesn’t find a space in the string, the whole string must be one word, so the entire string should be returned.

Let’s work through how we’d write the signature of this function without using slices, to understand the problem that slices will solve:

切片允许您引用集合中连续的元素序列，而不是整个集合。 切片是一种引用，因此它没有所有权。

这是一个小编程问题：编写一个函数，该函数接受由空格分隔的单词字符串，并返回在该字符串中找到的第一个单词。 如果函数在字符串中没有找到空格，则整个字符串必须是一个单词，因此应返回整个字符串。

让我们看看如何在不使用切片的情况下编写该函数的签名，以了解切片将解决的问题：
```
fn first_word(s: &String) -> ?
```
The first_word function has a &String as a parameter. We don’t want ownership, so this is fine. But what should we return? We don’t really have a way to talk about part of a string. However, we could return the index of the end of the word, indicated by a space. Let’s try that, as shown in Listing 4-7.

first_word 函数有一个 &String 作为参数。 我们不需要所有权，所以这很好。 但我们应该返回什么？ 我们确实没有办法谈论字符串的一部分。 但是，我们可以返回单词结尾的索引，用空格表示。 让我们尝试一下，如清单 4-7 所示。

Filename: src/main.rs
```
fn first_word(s: &String) -> usize {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return i;
        }
    }

    s.len()
}
```

Listing 4-7: The first_word function that returns a byte index value into the String parameter

Because we need to go through the String element by element and check whether a value is a space, we’ll convert our String to an array of bytes using the as_bytes method.
```
    let bytes = s.as_bytes();
```
Next, we create an iterator over the array of bytes using the iter method:
```
    for (i, &item) in bytes.iter().enumerate() {
```
We’ll discuss iterators in more detail in Chapter 13. For now, know that iter is a method that returns each element in a collection and that enumerate wraps the result of iter and returns each element as part of a tuple instead. The first element of the tuple returned from enumerate is the index, and the second element is a reference to the element. This is a bit more convenient than calculating the index ourselves.

Because the enumerate method returns a tuple, we can use patterns to destructure that tuple. We’ll be discussing patterns more in Chapter 6. In the for loop, we specify a pattern that has i for the index in the tuple and &item for the single byte in the tuple. Because we get a reference to the element from .iter().enumerate(), we use & in the pattern.

Inside the for loop, we search for the byte that represents the space by using the byte literal syntax. If we find a space, we return the position. Otherwise, we return the length of the string by using s.len().

我们将在第 13 章中更详细地讨论迭代器。现在，我们知道 iter 是一个返回集合中每个元素的方法，并且 enumerate 包装了 iter 的结果并将每个元素作为元组的一部分返回。 enumerate 返回的元组的第一个元素是索引，第二个元素是对该元素的引用。 这比我们自己计算指数方便一点。

因为 enumerate 方法返回一个元组，所以我们可以使用模式来解构该元组。 我们将在第 6 章中更多地讨论模式。在 for 循环中，我们指定一个模式，其中 i 为元组中的索引，&item 为元组中的单个字节。 因为我们从 .iter().enumerate() 获得对元素的引用，所以我们在模式中使用 & 。

在 for 循环内，我们使用字节文字语法搜索表示空间的字节。 如果我们找到一个空间，我们就会返回该位置。 否则，我们使用 s.len() 返回字符串的长度。

```
        if item == b' ' {
            return i;
        }
    }

    s.len()
```
We now have a way to find out the index of the end of the first word in the string, but there’s a problem. We’re returning a usize on its own, but it’s only a meaningful number in the context of the &String. In other words, because it’s a separate value from the String, there’s no guarantee that it will still be valid in the future. Consider the program in Listing 4-8 that uses the first_word function from Listing 4-7.

我们现在有办法找出字符串中第一个单词末尾的索引，但是有一个问题。 我们自己返回一个 usize，但它只是 &String 上下文中的一个有意义的数字。 换句话说，因为它是一个独立于字符串的值，所以不能保证它在将来仍然有效。 考虑清单 4-8 中的程序，它使用清单 4-7 中的first_word 函数。

Filename: src/main.rs
```
fn main() {
    let mut s = String::from("hello world");

    let word = first_word(&s); // word will get the value 5

    s.clear(); // this empties the String, making it equal to ""

    // word still has the value 5 here, but there's no more string that
    // we could meaningfully use the value 5 with. word is now totally invalid!
}
```
Listing 4-8: Storing the result from calling the first_word function and then changing the String contents

This program compiles without any errors and would also do so if we used word after calling s.clear(). Because word isn’t connected to the state of s at all, word still contains the value 5. We could use that value 5 with the variable s to try to extract the first word out, but this would be a bug because the contents of s have changed since we saved 5 in word.

Having to worry about the index in word getting out of sync with the data in s is tedious and error prone! Managing these indices is even more brittle if we write a second_word function. Its signature would have to look like this:

该程序编译时没有任何错误，如果我们在调用 s.clear() 之后使用 word 也会出现这种情况。 因为 word 根本没有连接到 s 的状态，所以 word 仍然包含值 5。我们可以将值 5 与变量 s 一起使用来尝试提取第一个单词，但这将是一个错误，因为 自从我们在 word 中保存了 5 以来，s 已经发生了变化。

不得不担心 word 中的索引与 s 中的数据不同步是乏味且容易出错的！ 如果我们编写 secondary_word 函数，管理这些索引会更加脆弱。 它的签名必须如下所示：

```
fn second_word(s: &String) -> (usize, usize) {
```
Now we’re tracking a starting and an ending index, and we have even more values that were calculated from data in a particular state but aren’t tied to that state at all. We have three unrelated variables floating around that need to be kept in sync.

现在我们正在跟踪起始索引和结束索引，并且我们有更多的值是根据特定状态的数据计算出来的，但根本与该状态无关。 我们有三个不相关的变量需要保持同步。

Luckily, Rust has a solution to this problem: string slices.

### String Slices

A string slice is a reference to part of a String, and it looks like this:
```
    let s = String::from("hello world");

    let hello = &s[0..5];
    let world = &s[6..11];
```

Rather than a reference to the entire String, hello is a reference to a portion of the String, specified in the extra [0..5] bit. We create slices using a range within brackets by specifying [starting_index..ending_index], where starting_index is the first position in the slice and ending_index is one more than the last position in the slice. Internally, the slice data structure stores the starting position and the length of the slice, which corresponds to ending_index minus starting_index. So, in the case of let world = &s[6..11];, world would be a slice that contains a pointer to the byte at index 6 of s with a length value of 5.

hello 不是对整个字符串的引用，而是对字符串的一部分的引用，在额外的 [0..5] 位中指定。 我们通过指定 [starting_index..ending_index] 使用括号内的范围创建切片，其中starting_index 是切片中的第一个位置，ending_index 比切片中的最后一个位置大一个。 在内部，切片数据结构存储切片的起始位置和长度，对应于ending_index减去starting_index。 因此，在 let world = &s[6..11]; 的情况下，world 将是一个切片，其中包含指向 s 索引 6 处的字节的指针，长度值为 5。

Figure 4-6 shows this in a diagram.

Figure 4-6: String slice referring to part of a String

With Rust’s .. range syntax, if you want to start at index 0, you can drop the value before the two periods. In other words, these are equal:
```
let s = String::from("hello");

let slice = &s[0..2];
let slice = &s[..2];
```
By the same token, if your slice includes the last byte of the String, you can drop the trailing number. That means these are equal:
```
let s = String::from("hello");

let len = s.len();

let slice = &s[3..len];
let slice = &s[3..];
```
You can also drop both values to take a slice of the entire string. So these are equal:

```
let s = String::from("hello");

let len = s.len();

let slice = &s[0..len];
let slice = &s[..];

```
Note: String slice range indices must occur at valid UTF-8 character boundaries. If you attempt to create a string slice in the middle of a multibyte character, your program will exit with an error. For the purposes of introducing string slices, we are assuming ASCII only in this section; a more thorough discussion of UTF-8 handling is in the “Storing UTF-8 Encoded Text with Strings” section of Chapter 8.

注意：字符串切片范围索引必须出现在有效的 UTF-8 字符边界处。 如果您尝试在多字节字符的中间创建字符串切片，您的程序将错误退出。 为了介绍字符串切片，我们在本节中仅假设 ASCII； 第 8 章的“用字符串存储 UTF-8 编码文本”部分对 UTF-8 处理进行了更全面的讨论。

With all this information in mind, let’s rewrite first_word to return a slice. The type that signifies “string slice” is written as &str:

Filename: src/main.rs
```
fn first_word(s: &String) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}
```
We get the index for the end of the word the same way we did in Listing 4-7, by looking for the first occurrence of a space. When we find a space, we return a string slice using the start of the string and the index of the space as the starting and ending indices.

Now when we call first_word, we get back a single value that is tied to the underlying data. The value is made up of a reference to the starting point of the slice and the number of elements in the slice.

我们以与清单 4-7 中相同的方式获取单词结尾的索引，即查找第一次出现的空格。 当我们找到一个空格时，我们使用字符串的开头和空格的索引作为开始和结束索引返回一个字符串切片。

现在，当我们调用first_word时，我们会返回一个与底层数据相关的单个值。 该值由对切片起始点的引用和切片中元素的数量组成。

Returning a slice would also work for a second_word function:
```
fn second_word(s: &String) -> &str {
```
We now have a straightforward API that’s much harder to mess up because the compiler will ensure the references into the String remain valid. Remember the bug in the program in Listing 4-8, when we got the index to the end of the first word but then cleared the string so our index was invalid? That code was logically incorrect but didn’t show any immediate errors. The problems would show up later if we kept trying to use the first word index with an emptied string. Slices make this bug impossible and let us know we have a problem with our code much sooner. Using the slice version of first_word will throw a compile-time error:

我们现在有了一个简单的 API，更难搞乱，因为编译器将确保对 String 的引用保持有效。 还记得清单 4-8 中程序中的错误吗？当时我们获得了第一个单词末尾的索引，但随后清除了字符串，因此我们的索引无效了？ 该代码在逻辑上是不正确的，但没有显示任何直接错误。 如果我们继续尝试将第一个单词索引与空字符串一起使用，问题稍后就会出现。 切片使这个错误不可能发生，并让我们更快地知道我们的代码有问题。 使用first_word 的切片版本将引发编译时错误：

Filename: src/main.rs
```
fn main() {
    let mut s = String::from("hello world");

    let word = first_word(&s);

    s.clear(); // error!

    println!("the first word is: {}", word);
}
```
Here’s the compiler error:
```
$ cargo run
   Compiling ownership v0.1.0 (file:///projects/ownership)
error[E0502]: cannot borrow `s` as mutable because it is also borrowed as immutable
  --> src/main.rs:18:5
   |
16 |     let word = first_word(&s);
   |                           -- immutable borrow occurs here
17 |
18 |     s.clear(); // error!
   |     ^^^^^^^^^ mutable borrow occurs here
19 |
20 |     println!("the first word is: {}", word);
   |                                       ---- immutable borrow later used here



For more information about this error, try `rustc --explain E0502`.
error: could not compile `ownership` due to previous error
```
Recall from the borrowing rules that if we have an immutable reference to something, we cannot also take a mutable reference. Because clear needs to truncate the String, it needs to get a mutable reference. The println! after the call to clear uses the reference in word, so the immutable reference must still be active at that point. Rust disallows the mutable reference in clear and the immutable reference in word from existing at the same time, and compilation fails. Not only has Rust made our API easier to use, but it has also eliminated an entire class of errors at compile time!

回想一下借用规则，如果我们对某个东西有一个不可变的引用，我们就不能同时使用一个可变的引用。 因为clear需要截断String，所以需要获取可变引用。 打印！ 在调用clear之后使用word中的引用，因此不可变引用此时必须仍然处于活动状态。 Rust不允许clear中的可变引用和word中的不可变引用同时存在，并且编译失败。 Rust 不仅使我们的 API 更易于使用，而且还消除了编译时的一整类错误！

### String Literals as Slices
Recall that we talked about string literals being stored inside the binary. Now that we know about slices, we can properly understand string literals:
```
let s = "Hello, world!";
```
The type of s here is &str: it’s a slice pointing to that specific point of the binary. This is also why string literals are immutable; &str is an immutable reference.

### String Slices as Parameters
Knowing that you can take slices of literals and String values leads us to one more improvement on first_word, and that’s its signature:
```
fn first_word(s: &String) -> &str {
```
A more experienced Rustacean would write the signature shown in Listing 4-9 instead because it allows us to use the same function on both &String values and &str values.
```
fn first_word(s: &str) -> &str {
```
Listing 4-9: Improving the first_word function by using a string slice for the type of the s parameter

If we have a string slice, we can pass that directly. If we have a String, we can pass a slice of the String or a reference to the String. This flexibility takes advantage of deref coercions, a feature we will cover in “Implicit Deref Coercions with Functions and Methods” section of Chapter 15.

如果我们有一个字符串切片，我们可以直接传递它。 如果我们有一个字符串，我们可以传递字符串的切片或对字符串的引用。 这种灵活性利用了 deref 强制转换，我们将在第 15 章的“使用函数和方法进行隐式 Deref 强制转换”部分介绍该功能。

Defining a function to take a string slice instead of a reference to a String makes our API more general and useful without losing any functionality:

Filename: src/main.rs
```
fn main() {
    let my_string = String::from("hello world");

    // `first_word` works on slices of `String`s, whether partial or whole
    let word = first_word(&my_string[0..6]);
    let word = first_word(&my_string[..]);
    // `first_word` also works on references to `String`s, which are equivalent
    // to whole slices of `String`s
    let word = first_word(&my_string);

    let my_string_literal = "hello world";

    // `first_word` works on slices of string literals, whether partial or whole
    let word = first_word(&my_string_literal[0..6]);
    let word = first_word(&my_string_literal[..]);

    // Because string literals *are* string slices already,
    // this works too, without the slice syntax!
    let word = first_word(my_string_literal);
}
```
### Other Slices
String slices, as you might imagine, are specific to strings. But there’s a more general slice type too. Consider this array:
```
let a = [1, 2, 3, 4, 5];
```
Just as we might want to refer to part of a string, we might want to refer to part of an array. We’d do so like this:
```
let a = [1, 2, 3, 4, 5];

let slice = &a[1..3];

assert_eq!(slice, &[2, 3]);
```
This slice has the type &[i32]. It works the same way as string slices do, by storing a reference to the first element and a length. You’ll use this kind of slice for all sorts of other collections. We’ll discuss these collections in detail when we talk about vectors in Chapter 8.

该切片的类型为 &[i32]。 它的工作方式与字符串切片相同，通过存储对第一个元素的引用和长度。 您将把这种切片用于各种其他集合。 当我们在第 8 章讨论向量时，我们将详细讨论这些集合。

## 4.4 Summary
The concepts of ownership, borrowing, and slices ensure memory safety in Rust programs at compile time. The Rust language gives you control over your memory usage in the same way as other systems programming languages, but having the owner of data automatically clean up that data when the owner goes out of scope means you don’t have to write and debug extra code to get this control.

Ownership affects how lots of other parts of Rust work, so we’ll talk about these concepts further throughout the rest of the book. Let’s move on to Chapter 5 and look at grouping pieces of data together in a struct.

所有权、借用和切片的概念确保 Rust 程序在编译时的内存安全。 Rust 语言让您能够以与其他系统编程语言相同的方式控制内存使用情况，但是当数据所有者超出范围时，让数据所有者自动清理该数据意味着您无需编写和调试额外的代码 来获得这个控制权。

所有权会影响 Rust 许多其他部分的工作方式，因此我们将在本书的其余部分进一步讨论这些概念。 让我们继续第 5 章，看看如何将数据片段分组到一个结构体中。