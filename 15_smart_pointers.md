# 15 智能指针
A pointer is a general concept for a variable that contains an address in memory. This address refers to, or “points at,” some other data. The most common kind of pointer in Rust is a reference, which you learned about in Chapter 4. References are indicated by the & symbol and borrow the value they point to. They don’t have any special capabilities other than referring to data, and have no overhead.

指针是包含内存中地址的变量的一般概念。此地址指的是或“指向”其他一些数据。Rust中最常见的指针类型是引用，您在第4章中了解到了这一点。引用由&符号表示，并借用它们所指向的值。除了引用数据之外，它们没有任何特殊功能，也没有开销。

Smart pointers, on the other hand, are data structures that act like a pointer but also have additional metadata and capabilities. The concept of smart pointers isn’t unique to Rust: smart pointers originated in C++ and exist in other languages as well. Rust has a variety of smart pointers defined in the standard library that provide functionality beyond that provided by references. To explore the general concept, we’ll look at a couple of different examples of smart pointers, including a reference counting smart pointer type. This pointer enables you to allow data to have multiple owners by keeping track of the number of owners and, when no owners remain, cleaning up the data.

另一方面，智能指针是一种数据结构，其作用类似于指针，但也具有额外的元数据和功能。智能指针的概念并不是Rust独有的：智能指针起源于C++，也存在于其他语言中。Rust在标准库中定义了各种智能指针，它们提供的功能超出了引用所提供的功能。为了探索一般概念，我们将看几个不同的智能指针示例，包括引用计数智能指针类型。此指针使您能够通过跟踪所有者的数量，并在没有所有者的情况下清理数据，从而允许数据具有多个所有者。

Rust, with its concept of ownership and borrowing, has an additional difference between references and smart pointers: while references only borrow data, in many cases, smart pointers own the data they point to.

Rust的所有权和借用概念在引用和智能指针之间还有一个额外的区别：虽然引用只借用数据，但在许多情况下，智能指针拥有它们所指向的数据。

Though we didn’t call them as such at the time, we’ve already encountered a few smart pointers in this book, including String and Vec<T> in Chapter 8. Both these types count as smart pointers because they own some memory and allow you to manipulate it. They also have metadata and extra capabilities or guarantees. String, for example, stores its capacity as metadata and has the extra ability to ensure its data will always be valid UTF-8.

虽然我们当时没有这样称呼它们，但在这本书中我们已经遇到了一些智能指针，包括第8章中的String和Vec<t>。这两种类型都算作智能指针，因为它们拥有一些内存并允许您对其进行操作。它们还具有元数据和额外的功能或保证。例如，字符串将其容量存储为元数据，并具有确保其数据始终为有效UTF-8的额外功能。

Smart pointers are usually implemented using structs. Unlike an ordinary struct, smart pointers implement the Deref and Drop traits. The Deref trait allows an instance of the smart pointer struct to behave like a reference so you can write your code to work with either references or smart pointers. The Drop trait allows you to customize the code that’s run when an instance of the smart pointer goes out of scope. In this chapter, we’ll discuss both traits and demonstrate why they’re important to smart pointers.

智能指针通常使用结构来实现。与普通结构不同，智能指针实现Deref和Drop特性。Deref特性允许智能指针结构的实例表现得像引用，因此您可以编写代码来使用引用或智能指针。Drop特性允许您自定义智能指针实例超出范围时运行的代码。在本章中，我们将讨论这两个特征，并说明为什么它们对智能指针很重要。


Given that the smart pointer pattern is a general design pattern used frequently in Rust, this chapter won’t cover every existing smart pointer. Many libraries have their own smart pointers, and you can even write your own. We’ll cover the most common smart pointers in the standard library:

考虑到智能指针模式是Rust中经常使用的通用设计模式，本章不会涵盖所有现有的智能指针。许多库都有自己的智能指针，你甚至可以自己编写。我们将介绍标准库中最常见的智能指针：

+ Box<T> for allocating values on the heap
+ Rc<T>, a reference counting type that enables multiple ownership
+ Ref<T> and RefMut<T>, accessed through RefCell<T>, a type that enforces the borrowing rules at runtime instead of compile time

+ Box<T> 用于在堆上分配值
+ Rc<T>，一种支持多重所有权的引用计数类型
+ Ref<T> 和 RefMut<T>，通过 RefCell<T> 访问，这是一种在运行时而不是编译时强制执行借用规则的类型

In addition, we’ll cover the interior mutability pattern where an immutable type exposes an API for mutating an interior value. We’ll also discuss reference cycles: how they can leak memory and how to prevent them.

此外，我们还将介绍内部可变性模式，其中不可变类型公开用于改变内部值的 API。 我们还将讨论引用循环：它们如何泄漏内存以及如何防止它们。

Let’s dive in!

## 15.1 Using Box<T> to Point to Data on the Heap
The most straightforward smart pointer is a box, whose type is written Box<T>. Boxes allow you to store data on the heap rather than the stack. What remains on the stack is the pointer to the heap data. Refer to Chapter 4 to review the difference between the stack and the heap.

最直接的智能指针是一个框，其类型写为box<T>。框允许您将数据存储在堆上，而不是堆栈上。堆栈上剩下的是指向堆数据的指针。请参阅第4章，查看堆栈和堆之间的区别。

Boxes don’t have performance overhead, other than storing their data on the heap instead of on the stack. But they don’t have many extra capabilities either. You’ll use them most often in these situations:

除了将数据存储在堆上而不是堆栈上之外，盒子没有性能开销。但它们也没有太多额外的功能。在这种情况下，你最常使用它们:

+ When you have a type whose size can’t be known at compile time and you want to use a value of that type in a context that requires an exact size

+ When you have a large amount of data and you want to transfer ownership but ensure the data won’t be copied when you do so
+ When you want to own a value and you care only that it’s a type that implements a particular trait rather than being of a specific type

+当您有一个类型的大小在编译时未知，并且您希望在需要确切大小的上下文中使用该类型的值时
+当您有大量数据，并且希望转移所有权，但要确保这样做时不会复制数据时
+当你想拥有一个值，而你只关心它是实现特定trait的类型，而不是特定类型时

We’ll demonstrate the first situation in the “Enabling Recursive Types with Boxes” section. In the second case, transferring ownership of a large amount of data can take a long time because the data is copied around on the stack. To improve performance in this situation, we can store the large amount of data on the heap in a box. Then, only the small amount of pointer data is copied around on the stack, while the data it references stays in one place on the heap. The third case is known as a trait object, and Chapter 17 devotes an entire section, “Using Trait Objects That Allow for Values of Different Types,” just to that topic. So what you learn here you’ll apply again in Chapter 17!

我们将在“使用Boxes启用递归类型”一节中演示第一种情况。在第二种情况下，转移大量数据的所有权可能需要很长时间，因为数据是在堆栈上复制的。为了在这种情况下提高性能，我们可以将堆上的大量数据存储在一个盒子中。然后，只有少量的指针数据在堆栈上复制，而它引用的数据则留在堆上的一个位置。第三种情况被称为特质对象，第17章专门用了一整节“使用允许不同类型值的特质对象”来讨论这个主题。所以，你在这里学到的东西，将在第17章中再次申请！

### Using a Box<T> to Store Data on the Heap
Before we discuss the heap storage use case for Box<T>, we’ll cover the syntax and how to interact with values stored within a Box<T>.

在讨论Box<T>的堆存储用例之前，我们将介绍语法以及如何与存储在Box<T>中的值交互。

Listing 15-1 shows how to use a box to store an i32 value on the heap:

Filename: src/main.rs
``````
fn main() {
    let b = Box::new(5);
    println!("b = {}", b);
}
``````
Listing 15-1: Storing an i32 value on the heap using a box

We define the variable b to have the value of a Box that points to the value 5, which is allocated on the heap. This program will print b = 5; in this case, we can access the data in the box similar to how we would if this data were on the stack. Just like any owned value, when a box goes out of scope, as b does at the end of main, it will be deallocated. The deallocation happens both for the box (stored on the stack) and the data it points to (stored on the heap).

我们将变量b定义为具有指向值5的Box的值，该值是在堆上分配的。此程序将打印b=5；在这种情况下，我们可以访问框中的数据，类似于如果这些数据在堆栈上的情况。就像任何拥有的值一样，当一个框超出范围时，就像b在main末尾所做的那样，它将被释放。盒子（存储在堆栈上）和它所指向的数据（存储在堆上）都会发生解除分配。

Putting a single value on the heap isn’t very useful, so you won’t use boxes by themselves in this way very often. Having values like a single i32 on the stack, where they’re stored by default, is more appropriate in the majority of situations. Let’s look at a case where boxes allow us to define types that we wouldn’t be allowed to if we didn’t have boxes.

### Enabling Recursive Types with Boxes
A value of recursive type can have another value of the same type as part of itself. Recursive types pose an issue because at compile time Rust needs to know how much space a type takes up. However, the nesting of values of recursive types could theoretically continue infinitely, so Rust can’t know how much space the value needs. Because boxes have a known size, we can enable recursive types by inserting a box in the recursive type definition.

在堆上放一个值不是很有用，所以您不会经常以这种方式单独使用框。在大多数情况下，将值（如单个i32）存储在堆栈上（默认情况下存储在堆栈中）更合适。让我们来看一个案例，在这个案例中，一个box允许我们定义如果没有box就不允许定义的类型。

As an example of a recursive type, let’s explore the cons list. This is a data type commonly found in functional programming languages. The cons list type we’ll define is straightforward except for the recursion; therefore, the concepts in the example we’ll work with will be useful any time you get into more complex situations involving recursive types.

作为递归类型的一个例子，让我们研究一下cons列表。这是一种常见于函数式编程语言中的数据类型。除了递归之外，我们将定义的cons列表类型非常简单；因此，当您遇到涉及递归类型的更复杂的情况时，我们将使用的示例中的概念将非常有用。

#### More Information About the Cons List
A cons list is a data structure that comes from the Lisp programming language and its dialects and is made up of nested pairs, and is the Lisp version of a linked list. Its name comes from the cons function (short for “construct function”) in Lisp that constructs a new pair from its two arguments. By calling cons on a pair consisting of a value and another pair, we can construct cons lists made up of recursive pairs.

For example, here’s a pseudocode representation of a cons list containing the list 1, 2, 3 with each pair in parentheses:

cons 列表是一种来自 Lisp 编程语言及其方言的数据结构，由嵌套对组成，是链表的 Lisp 版本。 它的名字来自 Lisp 中的 cons 函数（“构造函数”的缩写），该函数根据两个参数构造一个新的对。 通过在由一个值和另一对组成的对上调用 cons，我们可以构造由递归对组成的 cons 列表。

例如，下面是 cons 列表的伪代码表示，其中包含列表 1、2、3，每对都放在括号中：

``````
(1, (2, (3, Nil)))
``````
Each item in a cons list contains two elements: the value of the current item and the next item. The last item in the list contains only a value called Nil without a next item. A cons list is produced by recursively calling the cons function. The canonical name to denote the base case of the recursion is Nil. Note that this is not the same as the “null” or “nil” concept in Chapter 6, which is an invalid or absent value.

The cons list isn’t a commonly used data structure in Rust. Most of the time when you have a list of items in Rust, Vec<T> is a better choice to use. Other, more complex recursive data types are useful in various situations, but by starting with the cons list in this chapter, we can explore how boxes let us define a recursive data type without much distraction.

Listing 15-2 contains an enum definition for a cons list. Note that this code won’t compile yet because the List type doesn’t have a known size, which we’ll demonstrate.

cons 列表中的每个项目都包含两个元素：当前项目的值和下一个项目的值。 列表中的最后一项仅包含一个名为 Nil 的值，没有下一项。 cons 列表是通过递归调用 cons 函数生成的。 表示递归基本情况的规范名称是 Nil。 请注意，这与第 6 章中的“null”或“nil”概念不同，后者是无效或不存在的值。

cons 列表不是 Rust 中常用的数据结构。 大多数时候，当您有 Rust 中的项目列表时，Vec<T> 是更好的选择。 其他更复杂的递归数据类型在各种情况下都很有用，但是从本章中的缺点列表开始，我们可以探索框如何让我们定义递归数据类型而不会造成太多干扰。

清单 15-2 包含 cons 列表的枚举定义。 请注意，此代码尚未编译，因为 List 类型没有已知的大小，我们将对此进行演示。

Filename: src/main.rs
``````
This code does not compile!
enum List {
    Cons(i32, List),
    Nil,
}
``````
Listing 15-2: The first attempt at defining an enum to represent a cons list data structure of i32 values
``````
Note: We’re implementing a cons list that holds only i32 values for the purposes of this example. We could have implemented it using generics, as we discussed in Chapter 10, to define a cons list type that could store values of any type.

注意：出于本示例的目的，我们正在实现一个仅包含 i32 值的 cons 列表。 我们可以使用泛型来实现它，正如我们在第 10 章中讨论的那样，定义一个可以存储任何类型值的 cons 列表类型。

``````

Using the List type to store the list 1, 2, 3 would look like the code in Listing 15-3:

Filename: src/main.rs
``````
This code does not compile!
use crate::List::{Cons, Nil};

fn main() {
    let list = Cons(1, Cons(2, Cons(3, Nil)));
}
``````
Listing 15-3: Using the List enum to store the list 1, 2, 3

The first Cons value holds 1 and another List value. This List value is another Cons value that holds 2 and another List value. This List value is one more Cons value that holds 3 and a List value, which is finally Nil, the non-recursive variant that signals the end of the list.

第一个 Cons 值包含 1 和另一个 List 值。 该列表值是另一个包含 2 和另一个列表值的 Cons 值。 这个 List 值又是一个包含 3 的 Cons 值和一个 List 值，最后是 Nil，这是一种表示列表结束的非递归变体。

If we try to compile the code in Listing 15-3, we get the error shown in Listing 15-4:
``````
$ cargo run
   Compiling cons-list v0.1.0 (file:///projects/cons-list)
error[E0072]: recursive type `List` has infinite size
 --> src/main.rs:1:1
  |
1 | enum List {
  | ^^^^^^^^^
2 |     Cons(i32, List),
  |               ---- recursive without indirection
  |
help: insert some indirection (e.g., a `Box`, `Rc`, or `&`) to break the cycle
  |
2 |     Cons(i32, Box<List>),
  |               ++++    +

For more information about this error, try `rustc --explain E0072`.
error: could not compile `cons-list` due to previous error
``````
Listing 15-4: The error we get when attempting to define a recursive enum

The error shows this type “has infinite size.” The reason is that we’ve defined List with a variant that is recursive: it holds another value of itself directly. As a result, Rust can’t figure out how much space it needs to store a List value. Let’s break down why we get this error. First, we’ll look at how Rust decides how much space it needs to store a value of a non-recursive type.

该错误表明该类型“具有无限大小”。 原因是我们用递归的变体定义了 List：它直接保存自身的另一个值。 因此，Rust 无法计算出存储 List 值需要多少空间。 让我们来分析一下为什么会出现这个错误。 首先，我们来看看 Rust 如何决定存储非递归类型的值需要多少空间。

#### Computing the Size of a Non-Recursive Type
Recall the Message enum we defined in Listing 6-2 when we discussed enum definitions in Chapter 6:
``````
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
``````
To determine how much space to allocate for a Message value, Rust goes through each of the variants to see which variant needs the most space. Rust sees that Message::Quit doesn’t need any space, Message::Move needs enough space to store two i32 values, and so forth. Because only one variant will be used, the most space a Message value will need is the space it would take to store the largest of its variants.

Contrast this with what happens when Rust tries to determine how much space a recursive type like the List enum in Listing 15-2 needs. The compiler starts by looking at the Cons variant, which holds a value of type i32 and a value of type List. Therefore, Cons needs an amount of space equal to the size of an i32 plus the size of a List. To figure out how much memory the List type needs, the compiler looks at the variants, starting with the Cons variant. The Cons variant holds a value of type i32 and a value of type List, and this process continues infinitely, as shown in Figure 15-1.

为了确定为 Message 值分配多少空间，Rust 会遍历每个变体以查看哪个变体需要最多空间。 Rust 认为 Message::Quit 不需要任何空间，Message::Move 需要足够的空间来存储两个 i32 值，依此类推。 由于仅使用一种变体，因此 Message 值所需的最大空间就是存储其最大变体所需的空间。

将此与 Rust 尝试确定递归类型（如清单 15-2 中的 List 枚举）需要多少空间时发生的情况进行对比。 编译器首先查看 Cons 变体，它包含一个 i32 类型的值和一个 List 类型的值。 因此，Cons 需要的空间量等于 i32 的大小加上 List 的大小。 为了弄清楚 List 类型需要多少内存，编译器会从 Cons 变体开始查看变体。 Cons 变量保存一个 i32 类型的值和一个 List 类型的值，并且这个过程无限地继续下去，如图 15-1 所示。

An infinite Cons list
Figure 15-1: An infinite List consisting of infinite Cons variants

### Using Box<T> to Get a Recursive Type with a Known Size
Because Rust can’t figure out how much space to allocate for recursively defined types, the compiler gives an error with this helpful suggestion:

因为 Rust 无法计算出要为递归定义的类型分配多少空间，所以编译器会给出一个错误并给出以下有用的建议：

``````
help: insert some indirection (e.g., a `Box`, `Rc`, or `&`) to make `List` representable
  |
2 |     Cons(i32, Box<List>),
  |               ++++    +
``````
In this suggestion, “indirection” means that instead of storing a value directly, we should change the data structure to store the value indirectly by storing a pointer to the value instead.

Because a Box<T> is a pointer, Rust always knows how much space a Box<T> needs: a pointer’s size doesn’t change based on the amount of data it’s pointing to. This means we can put a Box<T> inside the Cons variant instead of another List value directly. The Box<T> will point to the next List value that will be on the heap rather than inside the Cons variant. Conceptually, we still have a list, created with lists holding other lists, but this implementation is now more like placing the items next to one another rather than inside one another.

We can change the definition of the List enum in Listing 15-2 and the usage of the List in Listing 15-3 to the code in Listing 15-5, which will compile:

在这个建议中，“间接”意味着我们不应该直接存储值，而是应该改变数据结构，通过存储指向该值的指针来间接存储该值。

因为 Box<T> 是一个指针，Rust 总是知道 Box<T> 需要多少空间：指针的大小不会根据它指向的数据量而改变。 这意味着我们可以将 Box<T> 放入 Cons 变体中，而不是直接放置另一个 List 值。 Box<T> 将指向堆上而不是 Cons 变量内部的下一个 List 值。 从概念上讲，我们仍然有一个列表，是用包含其他列表的列表创建的，但这种实现现在更像是将项目彼此相邻放置，而不是彼此放在一起。

我们可以将清单 15-2 中 List 枚举的定义和清单 15-3 中 List 的用法更改为清单 15-5 中的代码，它将编译：

Filename: src/main.rs
``````
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use crate::List::{Cons, Nil};

fn main() {
    let list = Cons(1, Box::new(Cons(2, Box::new(Cons(3, Box::new(Nil))))));
}
``````
Listing 15-5: Definition of List that uses Box<T> in order to have a known size

The Cons variant needs the size of an i32 plus the space to store the box’s pointer data. The Nil variant stores no values, so it needs less space than the Cons variant. We now know that any List value will take up the size of an i32 plus the size of a box’s pointer data. By using a box, we’ve broken the infinite, recursive chain, so the compiler can figure out the size it needs to store a List value. Figure 15-2 shows what the Cons variant looks like now.

Cons 变体需要 i32 的大小加上存储框指针数据的空间。 Nil 变体不存储任何值，因此它比 Cons 变体需要更少的空间。 我们现在知道任何 List 值都会占用 i32 的大小加上框指针数据的大小。 通过使用盒子，我们打破了无限的递归链，因此编译器可以计算出存储列表值所需的大小。 图 15-2 显示了 Cons 变体现在的样子。

A finite Cons list
Figure 15-2: A List that is not infinitely sized because Cons holds a Box

Boxes provide only the indirection and heap allocation; they don’t have any other special capabilities, like those we’ll see with the other smart pointer types. They also don’t have the performance overhead that these special capabilities incur, so they can be useful in cases like the cons list where the indirection is the only feature we need. We’ll look at more use cases for boxes in Chapter 17, too.

The Box<T> type is a smart pointer because it implements the Deref trait, which allows Box<T> values to be treated like references. When a Box<T> value goes out of scope, the heap data that the box is pointing to is cleaned up as well because of the Drop trait implementation. These two traits will be even more important to the functionality provided by the other smart pointer types we’ll discuss in the rest of this chapter. Let’s explore these two traits in more detail.

盒子仅提供间接和堆分配； 它们没有任何其他特殊功能，就像我们将在其他智能指针类型中看到的那样。 它们也没有这些特殊功能所产生的性能开销，因此它们在像 cons 列表这样的情况下非常有用，其中间接是我们唯一需要的功能。 我们还将在第 17 章中讨论更多盒子的用例。

Box<T> 类型是一个智能指针，因为它实现了 Deref 特征，该特征允许将 Box<T> 值视为引用。 当 Box<T> 值超出范围时，由于 Drop 特征实现，该框指向的堆数据也会被清除。 这两个特征对于我们将在本章其余部分讨论的其他智能指针类型提供的功能甚至更为重要。 让我们更详细地探讨这两个特征。

## 15.2 Treating Smart Pointers Like Reqular References with the Deref Trait

Implementing the Deref trait allows you to customize the behavior of the dereference operator * (not to be confused with the multiplication or glob operator). By implementing Deref in such a way that a smart pointer can be treated like a regular reference, you can write code that operates on references and use that code with smart pointers too.

Let’s first look at how the dereference operator works with regular references. Then we’ll try to define a custom type that behaves like Box<T>, and see why the dereference operator doesn’t work like a reference on our newly defined type. We’ll explore how implementing the Deref trait makes it possible for smart pointers to work in ways similar to references. Then we’ll look at Rust’s deref coercion feature and how it lets us work with either references or smart pointers.

Note: there’s one big difference between the MyBox<T> type we’re about to build and the real Box<T>: our version will not store its data on the heap. We are focusing this example on Deref, so where the data is actually stored is less important than the pointer-like behavior.

实现 Deref 特征允许您自定义取消引用运算符 * 的行为（不要与乘法或 glob 运算符混淆）。 通过以智能指针被视为常规引用的方式实现 Deref，您可以编写对引用进行操作的代码，并将该代码与智能指针一起使用。

让我们首先看看取消引用运算符如何与常规引用一起使用。 然后，我们将尝试定义一个行为类似于 Box<T> 的自定义类型，并了解为什么取消引用运算符不像我们新定义的类型上的引用那样工作。 我们将探讨如何实现 Deref 特征使智能指针能够以类似于引用的方式工作。 然后我们将了解 Rust 的 deref 强制转换功能以及它如何让我们使用引用或智能指针。

注意：我们要构建的 MyBox<T> 类型和真正的 Box<T> 之间有一个很大的区别：我们的版本不会将其数据存储在堆上。 我们将这个示例重点放在 Deref 上，因此数据实际存储的位置不如类似指针的行为重要。

### Following the Pointer to the Value

A regular reference is a type of pointer, and one way to think of a pointer is as an arrow to a value stored somewhere else. In Listing 15-6, we create a reference to an i32 value and then use the dereference operator to follow the reference to the value:

常规引用是一种指针，将指针视为指向存储在其他位置的值的箭头。 在清单 15-6 中，我们创建了对 i32 值的引用，然后使用取消引用运算符来跟踪对该值的引用：

Filename: src/main.rs
``````
fn main() {
    let x = 5;
    let y = &x;

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
``````
Listing 15-6: Using the dereference operator to follow a reference to an i32 value

The variable x holds an i32 value 5. We set y equal to a reference to x. We can assert that x is equal to 5. However, if we want to make an assertion about the value in y, we have to use *y to follow the reference to the value it’s pointing to (hence dereference) so the compiler can compare the actual value. Once we dereference y, we have access to the integer value y is pointing to that we can compare with 5.

变量 x 保存 i32 值 5。我们将 y 设置为等于对 x 的引用。 我们可以断言 x 等于 5。但是，如果我们想对 y 中的值进行断言，则必须使用 *y 来跟踪对其所指向的值的引用（因此取消引用），以便编译器可以比较 实际值。 一旦我们取消引用 y，我们就可以访问 y 所指向的整数值，我们可以将其与 5 进行比较。

If we tried to write assert_eq!(5, y); instead, we would get this compilation error:
``````
$ cargo run
   Compiling deref-example v0.1.0 (file:///projects/deref-example)
error[E0277]: can't compare `{integer}` with `&{integer}`
 --> src/main.rs:6:5
  |
6 |     assert_eq!(5, y);
  |     ^^^^^^^^^^^^^^^^ no implementation for `{integer} == &{integer}`
  |
  = help: the trait `PartialEq<&{integer}>` is not implemented for `{integer}`
  = help: the following other types implement trait `PartialEq<Rhs>`:
            f32
            f64
            i128
            i16
            i32
            i64
            i8
            isize
          and 6 others
  = note: this error originates in the macro `assert_eq` (in Nightly builds, run with -Z macro-backtrace for more info)

For more information about this error, try `rustc --explain E0277`.
error: could not compile `deref-example` due to previous error
``````
Comparing a number and a reference to a number isn’t allowed because they’re different types. We must use the dereference operator to follow the reference to the value it’s pointing to.

不允许比较数字和数字引用，因为它们是不同的类型。 我们必须使用解引用运算符来跟踪对其所指向的值的引用。

### Using Box<T> Like a Reference
We can rewrite the code in Listing 15-6 to use a Box<T> instead of a reference; the dereference operator used on the Box<T> in Listing 15-7 functions in the same way as the dereference operator used on the reference in Listing 15-6:

我们可以重写清单 15-6 中的代码以使用 Box<T> 而不是引用； 清单 15-7 中 Box<T> 上使用的解引用运算符的功能与清单 15-6 中引用上使用的解引用运算符的功能相同：

Filename: src/main.rs
``````
fn main() {
    let x = 5;
    let y = Box::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
``````
Listing 15-7: Using the dereference operator on a Box<i32>

The main difference between Listing 15-7 and Listing 15-6 is that here we set y to be an instance of a Box<T> pointing to a copied value of x rather than a reference pointing to the value of x. In the last assertion, we can use the dereference operator to follow the pointer of the Box<T> in the same way that we did when y was a reference. Next, we’ll explore what is special about Box<T> that enables us to use the dereference operator by defining our own type.

清单 15-7 和清单 15-6 之间的主要区别在于，这里我们将 y 设置为 Box<T> 的实例，指向 x 的复制值，而不是指向 x 值的引用。 在最后一个断言中，我们可以使用取消引用运算符来跟踪 Box<T> 的指针，就像 y 是引用时所做的那样。 接下来，我们将探讨 Box<T> 的特殊之处，它使我们能够通过定义自己的类型来使用解引用运算符。

### Defining Our Own Smart Pointer
Let’s build a smart pointer similar to the Box<T> type provided by the standard library to experience how smart pointers behave differently from references by default. Then we’ll look at how to add the ability to use the dereference operator.

The Box<T> type is ultimately defined as a tuple struct with one element, so Listing 15-8 defines a MyBox<T> type in the same way. We’ll also define a new function to match the new function defined on Box<T>.

让我们构建一个类似于标准库提供的 Box<T> 类型的智能指针，以体验默认情况下智能指针与引用的行为有何不同。 然后我们将了解如何添加使用取消引用运算符的功能。

Box<T> 类型最终被定义为具有一个元素的元组结构，因此清单 15-8 以相同的方式定义了 MyBox<T> 类型。 我们还将定义一个新函数来匹配 Box<T> 上定义的新函数。


Filename: src/main.rs
``````
struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}
``````
Listing 15-8: Defining a MyBox<T> type

We define a struct named MyBox and declare a generic parameter T, because we want our type to hold values of any type. The MyBox type is a tuple struct with one element of type T. The MyBox::new function takes one parameter of type T and returns a MyBox instance that holds the value passed in.

Let’s try adding the main function in Listing 15-7 to Listing 15-8 and changing it to use the MyBox<T> type we’ve defined instead of Box<T>. The code in Listing 15-9 won’t compile because Rust doesn’t know how to dereference MyBox.

我们定义一个名为 MyBox 的结构体并声明一个泛型参数 T，因为我们希望我们的类型能够保存任何类型的值。 MyBox 类型是一种元组结构，其中有一个类型为 T 的元素。MyBox::new 函数采用一个类型为 T 的参数，并返回一个保存传入值的 MyBox 实例。

让我们尝试将清单 15-7 中的 main 函数添加到清单 15-8 中，并将其更改为使用我们定义的 MyBox<T> 类型，而不是 Box<T>。 清单 15-9 中的代码无法编译，因为 Rust 不知道如何取消引用 MyBox。

Filename: src/main.rs
``````
This code does not compile!
fn main() {
    let x = 5;
    let y = MyBox::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
``````
Listing 15-9: Attempting to use MyBox<T> in the same way we used references and Box<T>

Here’s the resulting compilation error:
``````
$ cargo run
   Compiling deref-example v0.1.0 (file:///projects/deref-example)
error[E0614]: type `MyBox<{integer}>` cannot be dereferenced
  --> src/main.rs:14:19
   |
14 |     assert_eq!(5, *y);
   |                   ^^

For more information about this error, try `rustc --explain E0614`.
error: could not compile `deref-example` due to previous error
``````
Our MyBox<T> type can’t be dereferenced because we haven’t implemented that ability on our type. To enable dereferencing with the * operator, we implement the Deref trait.

我们的 MyBox<T> 类型无法取消引用，因为我们尚未在我们的类型上实现该功能。 为了使用 * 运算符取消引用，我们实现了 Deref 特征。

### Treating a Type Like a Reference by Implementing the Deref Trait
As discussed in the “Implementing a Trait on a Type” section of Chapter 10, to implement a trait, we need to provide implementations for the trait’s required methods. The Deref trait, provided by the standard library, requires us to implement one method named deref that borrows self and returns a reference to the inner data. Listing 15-10 contains an implementation of Deref to add to the definition of MyBox:

正如第 10 章“在类型上实现 Trait”部分所讨论的，为了实现 Trait，我们需要为 Trait 所需的方法提供实现。 标准库提供的 Deref 特征要求我们实现一种名为 deref 的方法，该方法借用 self 并返回对内部数据的引用。 清单 15-10 包含添加到 MyBox 定义中的 Deref 实现：

Filename: src/main.rs
``````
use std::ops::Deref;

impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &Self::Target {
        &self.0
    }
}
``````
Listing 15-10: Implementing Deref on MyBox<T>

The type Target = T; syntax defines an associated type for the Deref trait to use. Associated types are a slightly different way of declaring a generic parameter, but you don’t need to worry about them for now; we’ll cover them in more detail in Chapter 19.

We fill in the body of the deref method with &self.0 so deref returns a reference to the value we want to access with the * operator; recall from the “Using Tuple Structs without Named Fields to Create Different Types” section of Chapter 5 that .0 accesses the first value in a tuple struct. The main function in Listing 15-9 that calls * on the MyBox<T> value now compiles, and the assertions pass!

Without the Deref trait, the compiler can only dereference & references. The deref method gives the compiler the ability to take a value of any type that implements Deref and call the deref method to get a & reference that it knows how to dereference.

类型目标=T； 语法定义了要使用的 Deref 特征的关联类型。 关联类型是声明泛型参数的一种稍微不同的方式，但您现在不需要担心它们； 我们将在第 19 章中更详细地介绍它们。

我们用 &self.0 填充 deref 方法的主体，因此 deref 返回我们想要使用 * 运算符访问的值的引用； 回想一下第 5 章的“使用不带命名字段的元组结构创建不同类型”部分，.0 访问元组结构中的第一个值。 清单 15-9 中对 MyBox<T> 值调用 * 的主函数现在可以编译，并且断言通过！

如果没有 Deref 特征，编译器只能取消引用和引用。 deref 方法使编译器能够获取实现 Deref 的任何类型的值，并调用 deref 方法来获取它知道如何取消引用的 & 引用。

When we entered *y in Listing 15-9, behind the scenes Rust actually ran this code:
``````
*(y.deref())
``````
Rust substitutes the * operator with a call to the deref method and then a plain dereference so we don’t have to think about whether or not we need to call the deref method. This Rust feature lets us write code that functions identically whether we have a regular reference or a type that implements Deref.

The reason the deref method returns a reference to a value, and that the plain dereference outside the parentheses in *(y.deref()) is still necessary, is to do with the ownership system. If the deref method returned the value directly instead of a reference to the value, the value would be moved out of self. We don’t want to take ownership of the inner value inside MyBox<T> in this case or in most cases where we use the dereference operator.

Note that the * operator is replaced with a call to the deref method and then a call to the * operator just once, each time we use a * in our code. Because the substitution of the * operator does not recurse infinitely, we end up with data of type i32, which matches the 5 in assert_eq! in Listing 15-9.

Rust 将 * 运算符替换为调用 deref 方法，然后进行简单的取消引用，因此我们不必考虑是否需要调用 deref 方法。 这个 Rust 功能让我们可以编写功能相同的代码，无论我们有常规引用还是实现 Deref 的类型。

deref 方法返回对值的引用，并且 *(y.deref()) 中括号外的普通取消引用仍然是必要的，这与所有权系统有关。 如果 deref 方法直接返回值而不是对该值的引用，则该值将从 self 中移出。 在这种情况下或在大多数使用解引用运算符的情况下，我们不想获得 MyBox<T> 内部值的所有权。

请注意，每次我们在代码中使用 * 时，* 运算符都会被替换为对 deref 方法的调用，然后仅调用一次 * 运算符。 因为 * 运算符的替换不会无限递归，所以我们最终得到 i32 类型的数据，它与assert_eq! 中的 5 匹配！ 如清单 15-9 所示。

### Implicit Deref Coercions with Functions and Methods
Deref coercion converts a reference to a type that implements the Deref trait into a reference to another type. For example, deref coercion can convert &String to &str because String implements the Deref trait such that it returns &str. Deref coercion is a convenience Rust performs on arguments to functions and methods, and works only on types that implement the Deref trait. It happens automatically when we pass a reference to a particular type’s value as an argument to a function or method that doesn’t match the parameter type in the function or method definition. A sequence of calls to the deref method converts the type we provided into the type the parameter needs.

Deref coercion was added to Rust so that programmers writing function and method calls don’t need to add as many explicit references and dereferences with & and *. The deref coercion feature also lets us write more code that can work for either references or smart pointers.

To see deref coercion in action, let’s use the MyBox<T> type we defined in Listing 15-8 as well as the implementation of Deref that we added in Listing 15-10. Listing 15-11 shows the definition of a function that has a string slice parameter:

Deref 强制将对实现 Deref 特征的类型的引用转换为对另一种类型的引用。 例如，deref 强制转换可以将 &String 转换为 &str，因为 String 实现了 Deref 特征，因此它返回 &str。 Deref 强制是 Rust 对函数和方法的参数执行的一种便利方法，并且仅适用于实现 Deref 特征的类型。 当我们将对特定类型值的引用作为参数传递给与函数或方法定义中的参数类型不匹配的函数或方法时，它会自动发生。 对 deref 方法的一系列调用会将我们提供的类型转换为参数所需的类型。

Rust 中添加了 Deref 强制，以便编写函数和方法调用的程序员不需要使用 & 和 * 添加尽可能多的显式引用和取消引用。 deref 强制转换功能还允许我们编写更多可用于引用或智能指针的代码。

要查看 deref 强制转换的实际效果，让我们使用清单 15-8 中定义的 MyBox<T> 类型以及清单 15-10 中添加的 Deref 实现。 清单 15-11 显示了具有字符串切片参数的函数的定义：

Filename: src/main.rs
``````
fn hello(name: &str) {
    println!("Hello, {name}!");
}
``````
Listing 15-11: A hello function that has the parameter name of type &str

We can call the hello function with a string slice as an argument, such as hello("Rust"); for example. Deref coercion makes it possible to call hello with a reference to a value of type MyBox<String>, as shown in Listing 15-12:

我们可以使用字符串切片作为参数来调用 hello 函数，例如 hello("Rust"); 例如。 Deref 强制使得可以通过引用 MyBox<String> 类型的值来调用 hello，如清单 15-12 所示：

Filename: src/main.rs
``````
fn main() {
    let m = MyBox::new(String::from("Rust"));
    hello(&m);
}
``````
Listing 15-12: Calling hello with a reference to a MyBox<String> value, which works because of deref coercion

Here we’re calling the hello function with the argument &m, which is a reference to a MyBox<String> value. Because we implemented the Deref trait on MyBox<T> in Listing 15-10, Rust can turn &MyBox<String> into &String by calling deref. The standard library provides an implementation of Deref on String that returns a string slice, and this is in the API documentation for Deref. Rust calls deref again to turn the &String into &str, which matches the hello function’s definition.

If Rust didn’t implement deref coercion, we would have to write the code in Listing 15-13 instead of the code in Listing 15-12 to call hello with a value of type &MyBox<String>.

这里我们使用参数 &m 调用 hello 函数，它是对 MyBox<String> 值的引用。 因为我们在清单 15-10 中的 MyBox<T> 上实现了 Deref 特征，所以 Rust 可以通过调用 deref 将 &MyBox<String> 转换为 &String。 标准库提供了 String 上的 Deref 实现，它返回字符串切片，这位于 Deref 的 API 文档中。 Rust 再次调用 deref 将 &String 转换为 &str，这与 hello 函数的定义匹配。

如果 Rust 没有实现 deref 强制转换，我们就必须编写清单 15-13 中的代码而不是清单 15-12 中的代码来使用 &MyBox<String> 类型的值调用 hello。

Filename: src/main.rs
``````
fn main() {
    let m = MyBox::new(String::from("Rust"));
    hello(&(*m)[..]);
}
``````
Listing 15-13: The code we would have to write if Rust didn’t have deref coercion

The (*m) dereferences the MyBox<String> into a String. Then the & and [..] take a string slice of the String that is equal to the whole string to match the signature of hello. This code without deref coercions is harder to read, write, and understand with all of these symbols involved. Deref coercion allows Rust to handle these conversions for us automatically.

When the Deref trait is defined for the types involved, Rust will analyze the types and use Deref::deref as many times as necessary to get a reference to match the parameter’s type. The number of times that Deref::deref needs to be inserted is resolved at compile time, so there is no runtime penalty for taking advantage of deref coercion!

(*m) 将 MyBox<String> 取消引用为字符串。 然后 & 和 [..] 取 String 的一个字符串切片，该切片等于整个字符串，以匹配 hello 的签名。 由于涉及所有这些符号，没有解引用强制的代码更难以阅读、编写和理解。 Deref 强制允许 Rust 自动为我们处理这些转换。

当为涉及的类型定义 Deref 特征时，Rust 将分析类型并根据需要多次使用 Deref::deref 来获取与参数类型匹配的引用。 Deref::deref 需要插入的次数在编译时确定，因此利用 deref 强制不会产生运行时损失！

### How Deref Coercion Interacts with Mutability
Similar to how you use the Deref trait to override the * operator on immutable references, you can use the DerefMut trait to override the * operator on mutable references.

Rust does deref coercion when it finds types and trait implementations in three cases:

与使用 Deref 特征覆盖不可变引用上的 * 运算符类似，您可以使用 DerefMut 特征覆盖可变引用上的 * 运算符。

Rust 在三种情况下找到类型和特征实现时会进行 deref 强制转换：

+ From &T to &U when T: Deref<Target=U>
+ From &mut T to &mut U when T: DerefMut<Target=U>
+ From &mut T to &U when T: Deref<Target=U>

+ 当 T 时从 &T 到 &U：Deref<Target=U>
+ 当 T 时从 &mut T 到 &mut U：DerefMut<Target=U>
+ 当 T 时从 &mut T 到 &U：Deref<Target=U>

The first two cases are the same as each other except that the second implements mutability. The first case states that if you have a &T, and T implements Deref to some type U, you can get a &U transparently. The second case states that the same deref coercion happens for mutable references.

The third case is trickier: Rust will also coerce a mutable reference to an immutable one. But the reverse is not possible: immutable references will never coerce to mutable references. Because of the borrowing rules, if you have a mutable reference, that mutable reference must be the only reference to that data (otherwise, the program wouldn’t compile). Converting one mutable reference to one immutable reference will never break the borrowing rules. Converting an immutable reference to a mutable reference would require that the initial immutable reference is the only immutable reference to that data, but the borrowing rules don’t guarantee that. Therefore, Rust can’t make the assumption that converting an immutable reference to a mutable reference is possible.

前两种情况彼此相同，只是第二种情况实现了可变性。 第一种情况表明，如果您有一个 &T，并且 T 对某种类型 U 实现了 Deref，您可以透明地获得 &U。 第二种情况表明，可变引用也会发生相同的 deref 强制转换。

第三种情况比较棘手：Rust 还会将可变引用强制指向不可变引用。 但反过来是不可能的：不可变引用永远不会强制可变引用。 由于借用规则，如果您有可变引用，则该可变引用必须是对该数据的唯一引用（否则，程序将无法编译）。 将一个可变引用转换为一个不可变引用永远不会违反借用规则。 将不可变引用转换为可变引用将要求初始不可变引用是对该数据的唯一不可变引用，但借用规则不能保证这一点。 因此，Rust 不能假设将不可变引用转换为可变引用是可能的。

## 15.3 Running Code one Cleanup with the Drop Trait
The second trait important to the smart pointer pattern is Drop, which lets you customize what happens when a value is about to go out of scope. You can provide an implementation for the Drop trait on any type, and that code can be used to release resources like files or network connections.

We’re introducing Drop in the context of smart pointers because the functionality of the Drop trait is almost always used when implementing a smart pointer. For example, when a Box<T> is dropped it will deallocate the space on the heap that the box points to.

In some languages, for some types, the programmer must call code to free memory or resources every time they finish using an instance of those types. Examples include file handles, sockets, or locks. If they forget, the system might become overloaded and crash. In Rust, you can specify that a particular bit of code be run whenever a value goes out of scope, and the compiler will insert this code automatically. As a result, you don’t need to be careful about placing cleanup code everywhere in a program that an instance of a particular type is finished with—you still won’t leak resources!

You specify the code to run when a value goes out of scope by implementing the Drop trait. The Drop trait requires you to implement one method named drop that takes a mutable reference to self. To see when Rust calls drop, let’s implement drop with println! statements for now.

Listing 15-14 shows a CustomSmartPointer struct whose only custom functionality is that it will print Dropping CustomSmartPointer! when the instance goes out of scope, to show when Rust runs the drop function.

对于智能指针模式重要的第二个特征是 Drop，它允许您自定义当值即将超出范围时会发生的情况。 您可以为任何类型提供 Drop 特征的实现，并且该代码可用于释放文件或网络连接等资源。

我们在智能指针的上下文中引入 Drop，因为在实现智能指针时几乎总是使用 Drop 特征的功能。 例如，当删除 Box<T> 时，它将释放该框指向的堆上的空间。

在某些语言中，对于某些类型，程序员每次使用完这些类型的实例时都必须调用代码来释放内存或资源。 示例包括文件句柄、套接字或锁。 如果他们忘记了，系统可能会过载并崩溃。 在 Rust 中，您可以指定每当值超出范围时运行特定的代码位，编译器将自动插入此代码。 因此，您无需小心地将清理代码放置在特定类型实例完成的程序中的任何位置 - 您仍然不会泄漏资源！

您可以通过实现 Drop 特征来指定当值超出范围时要运行的代码。 Drop 特征要求您实现一个名为 drop 的方法，该方法采用对 self 的可变引用。 要查看 Rust 何时调用 drop，让我们用 println 实现 drop！ 暂时发表声明。

清单 15-14 显示了一个 CustomSmartPointer 结构，其唯一的自定义功能是它将打印 Dropping CustomSmartPointer! 当实例超出范围时，显示 Rust 何时运行 drop 函数。

Filename: src/main.rs
``````
struct CustomSmartPointer {
    data: String,
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!("Dropping CustomSmartPointer with data `{}`!", self.data);
    }
}

fn main() {
    let c = CustomSmartPointer {
        data: String::from("my stuff"),
    };
    let d = CustomSmartPointer {
        data: String::from("other stuff"),
    };
    println!("CustomSmartPointers created.");
}
``````
Listing 15-14: A CustomSmartPointer struct that implements the Drop trait where we would put our cleanup code

The Drop trait is included in the prelude, so we don’t need to bring it into scope. We implement the Drop trait on CustomSmartPointer and provide an implementation for the drop method that calls println!. The body of the drop function is where you would place any logic that you wanted to run when an instance of your type goes out of scope. We’re printing some text here to demonstrate visually when Rust will call drop.

In main, we create two instances of CustomSmartPointer and then print CustomSmartPointers created. At the end of main, our instances of CustomSmartPointer will go out of scope, and Rust will call the code we put in the drop method, printing our final message. Note that we didn’t need to call the drop method explicitly.

Drop 特征包含在前奏中，因此我们不需要将其纳入范围。 我们在 CustomSmartPointer 上实现 Drop 特征，并提供调用 println! 的 drop 方法的实现。 drop 函数的主体是您可以放置当您的类型的实例超出范围时要运行的任何逻辑的位置。 我们在这里打印一些文本来直观地演示 Rust 何时调用 drop。

在 main 中，我们创建了两个 CustomSmartPointer 实例，然后打印创建的 CustomSmartPointers。 在 main 的末尾，我们的 CustomSmartPointer 实例将超出范围，Rust 将调用我们放入 drop 方法中的代码，打印我们的最终消息。 请注意，我们不需要显式调用 drop 方法。

When we run this program, we’ll see the following output:
``````
$ cargo run
   Compiling drop-example v0.1.0 (file:///projects/drop-example)
    Finished dev [unoptimized + debuginfo] target(s) in 0.60s
     Running `target/debug/drop-example`
CustomSmartPointers created.
Dropping CustomSmartPointer with data `other stuff`!
Dropping CustomSmartPointer with data `my stuff`!
``````
Rust automatically called drop for us when our instances went out of scope, calling the code we specified. Variables are dropped in the reverse order of their creation, so d was dropped before c. This example’s purpose is to give you a visual guide to how the drop method works; usually you would specify the cleanup code that your type needs to run rather than a print message.

当我们的实例超出范围时，Rust 会自动为我们调用 drop，调用我们指定的代码。 变量按照其创建时的相反顺序被删除，因此 d 在 c 之前被删除。 此示例的目的是为您提供 drop 方法如何工作的直观指南； 通常，您会指定您的类型需要运行的清理代码，而不是打印消息。

### Dropping a Value Early with std::mem::drop
Unfortunately, it’s not straightforward to disable the automatic drop functionality. Disabling drop isn’t usually necessary; the whole point of the Drop trait is that it’s taken care of automatically. Occasionally, however, you might want to clean up a value early. One example is when using smart pointers that manage locks: you might want to force the drop method that releases the lock so that other code in the same scope can acquire the lock. Rust doesn’t let you call the Drop trait’s drop method manually; instead you have to call the std::mem::drop function provided by the standard library if you want to force a value to be dropped before the end of its scope.

If we try to call the Drop trait’s drop method manually by modifying the main function from Listing 15-14, as shown in Listing 15-15, we’ll get a compiler error:

不幸的是，禁用自动删除功能并不简单。 通常不需要禁用 drop； Drop 特征的全部要点在于它是自动处理的。 然而，有时您可能希望尽早清理某个值。 一个例子是使用管理锁的智能指针时：您可能希望强制释放锁的 drop 方法，以便同一范围内的其他代码可以获得锁。 Rust 不允许您手动调用 Drop 特征的 drop 方法； 相反，如果您想强制在其范围结束之前删除某个值，则必须调用标准库提供的 std::mem::drop 函数。

如果我们尝试通过修改清单 15-14 中的 main 函数来手动调用 Drop 特征的 drop 方法，如清单 15-15 所示，我们将得到一个编译器错误：

Filename: src/main.rs
``````
This code does not compile!
fn main() {
    let c = CustomSmartPointer {
        data: String::from("some data"),
    };
    println!("CustomSmartPointer created.");
    c.drop();
    println!("CustomSmartPointer dropped before the end of main.");
}
``````
Listing 15-15: Attempting to call the drop method from the Drop trait manually to clean up early

When we try to compile this code, we’ll get this error:
``````
$ cargo run
   Compiling drop-example v0.1.0 (file:///projects/drop-example)
error[E0040]: explicit use of destructor method
  --> src/main.rs:16:7
   |
16 |     c.drop();
   |     --^^^^--
   |     | |
   |     | explicit destructor calls not allowed
   |     help: consider using `drop` function: `drop(c)`

For more information about this error, try `rustc --explain E0040`.
error: could not compile `drop-example` due to previous error
``````
This error message states that we’re not allowed to explicitly call drop. The error message uses the term destructor, which is the general programming term for a function that cleans up an instance. A destructor is analogous to a constructor, which creates an instance. The drop function in Rust is one particular destructor.

Rust doesn’t let us call drop explicitly because Rust would still automatically call drop on the value at the end of main. This would cause a double free error because Rust would be trying to clean up the same value twice.

We can’t disable the automatic insertion of drop when a value goes out of scope, and we can’t call the drop method explicitly. So, if we need to force a value to be cleaned up early, we use the std::mem::drop function.

The std::mem::drop function is different from the drop method in the Drop trait. We call it by passing as an argument the value we want to force drop. The function is in the prelude, so we can modify main in Listing 15-15 to call the drop function, as shown in Listing 15-16:

此错误消息表明我们不允许显式调用 drop。 该错误消息使用术语析构函数，这是清理实例的函数的通用编程术语。 析构函数类似于构造函数，它创建一个实例。 Rust 中的 drop 函数是一种特殊的析构函数。

Rust 不允许我们显式调用 drop，因为 Rust 仍然会自动对 main 末尾的值调用 drop。 这会导致双重释放错误，因为 Rust 会尝试两次清理相同的值。

当值超出范围时，我们无法禁用 drop 的自动插入，并且无法显式调用 drop 方法。 因此，如果我们需要强制尽早清除某个值，我们可以使用 std::mem::drop 函数。

std::mem::drop 函数与 Drop 特征中的 drop 方法不同。 我们通过将要强制删除的值作为参数传递来调用它。 该函数位于prelude中，因此我们可以修改清单15-15中的main来调用drop函数，如清单15-16所示：

Filename: src/main.rs
``````
fn main() {
    let c = CustomSmartPointer {
        data: String::from("some data"),
    };
    println!("CustomSmartPointer created.");
    drop(c);
    println!("CustomSmartPointer dropped before the end of main.");
}
``````
Listing 15-16: Calling std::mem::drop to explicitly drop a value before it goes out of scope

Running this code will print the following:
``````
$ cargo run
   Compiling drop-example v0.1.0 (file:///projects/drop-example)
    Finished dev [unoptimized + debuginfo] target(s) in 0.73s
     Running `target/debug/drop-example`
CustomSmartPointer created.
Dropping CustomSmartPointer with data `some data`!
CustomSmartPointer dropped before the end of main.
``````
The text Dropping CustomSmartPointer with data `some data`! is printed between the CustomSmartPointer created. and CustomSmartPointer dropped before the end of main. text, showing that the drop method code is called to drop c at that point.

You can use code specified in a Drop trait implementation in many ways to make cleanup convenient and safe: for instance, you could use it to create your own memory allocator! With the Drop trait and Rust’s ownership system, you don’t have to remember to clean up because Rust does it automatically.

You also don’t have to worry about problems resulting from accidentally cleaning up values still in use: the ownership system that makes sure references are always valid also ensures that drop gets called only once when the value is no longer being used.

Now that we’ve examined Box<T> and some of the characteristics of smart pointers, let’s look at a few other smart pointers defined in the standard library.

文本 Droping CustomSmartPointer with data `some data`! 在创建的 CustomSmartPointer 之间打印。 和 CustomSmartPointer 在 main 结束之前掉落。 文本，显示在该点调用 drop 方法代码来删除 c。

您可以通过多种方式使用 Drop 特征实现中指定的代码，以使清理方便且安全：例如，您可以使用它来创建自己的内存分配器！ 有了 Drop 特征和 Rust 的所有权系统，你不必记得清理，因为 Rust 会自动完成。

您也不必担心因意外清理仍在使用的值而导致的问题：确保引用始终有效的所有权系统还确保当不再使用该值时仅调用一次 drop 。

现在我们已经研究了 Box<T> 和智能指针的一些特性，让我们看看标准库中定义的其他一些智能指针。

## 15.4 Rc<T>, the Reference Counted Smart Pointer
In the majority of cases, ownership is clear: you know exactly which variable owns a given value. However, there are cases when a single value might have multiple owners. For example, in graph data structures, multiple edges might point to the same node, and that node is conceptually owned by all of the edges that point to it. A node shouldn’t be cleaned up unless it doesn’t have any edges pointing to it and so has no owners.

You have to enable multiple ownership explicitly by using the Rust type Rc<T>, which is an abbreviation for reference counting. The Rc<T> type keeps track of the number of references to a value to determine whether or not the value is still in use. If there are zero references to a value, the value can be cleaned up without any references becoming invalid.

Imagine Rc<T> as a TV in a family room. When one person enters to watch TV, they turn it on. Others can come into the room and watch the TV. When the last person leaves the room, they turn off the TV because it’s no longer being used. If someone turns off the TV while others are still watching it, there would be uproar from the remaining TV watchers!

We use the Rc<T> type when we want to allocate some data on the heap for multiple parts of our program to read and we can’t determine at compile time which part will finish using the data last. If we knew which part would finish last, we could just make that part the data’s owner, and the normal ownership rules enforced at compile time would take effect.

Note that Rc<T> is only for use in single-threaded scenarios. When we discuss concurrency in Chapter 16, we’ll cover how to do reference counting in multithreaded programs.

在大多数情况下，所有权是明确的：您确切地知道哪个变量拥有给定值。 但是，在某些情况下，单个值可能有多个所有者。 例如，在图数据结构中，多个边可能指向同一个节点，并且该节点在概念上由指向它的所有边所拥有。 一个节点不应该被清理，除非它没有任何边缘指向它，因此没有所有者。

您必须使用 Rust 类型 Rc<T>（引用计数的缩写）显式启用多重所有权。 Rc<T> 类型跟踪对某个值的引用次数，以确定该值是否仍在使用中。 如果某个值的引用为零，则可以清除该值，而不会导致任何引用无效。

将 Rc<T> 想象为家庭房间中的电视。 当一个人进来看电视时，他们就会打开电视。 其他人可以进入房间看电视。 当最后一个人离开房间时，他们会关掉电视，因为电视不再被使用。 如果有人在其他人还在看电视的时候关掉电视，剩下的电视观众就会哗然！

当我们想要在堆上分配一些数据供程序的多个部分读取并且我们无法在编译时确定哪个部分最后使用这些数据时，我们使用 Rc<T> 类型。 如果我们知道哪个部分将最后完成，我们可以使该部分成为数据的所有者，并且在编译时强制执行的正常所有权规则将生效。

请注意，Rc<T> 仅适用于单线程场景。 当我们在第 16 章讨论并发时，我们将介绍如何在多线程程序中进行引用计数。

### Using Rc<T> to Share Data
Let’s return to our cons list example in Listing 15-5. Recall that we defined it using Box<T>. This time, we’ll create two lists that both share ownership of a third list. Conceptually, this looks similar to Figure 15-3:

Two lists that share ownership of a third list
Figure 15-3: Two lists, b and c, sharing ownership of a third list, a

We’ll create list a that contains 5 and then 10. Then we’ll make two more lists: b that starts with 3 and c that starts with 4. Both b and c lists will then continue on to the first a list containing 5 and 10. In other words, both lists will share the first list containing 5 and 10.

Trying to implement this scenario using our definition of List with Box<T> won’t work, as shown in Listing 15-17:

让我们回到清单 15-5 中的 cons 列表示例。 回想一下，我们使用 Box<T> 定义了它。 这次，我们将创建两个列表，它们共享第三个列表的所有权。 从概念上讲，这与图 15-3 类似：

两个列表共享第三个列表的所有权
图 15-3：两个列表 b 和 c，共享第三个列表 a 的所有权

我们将创建包含 5 和 10 的列表 a。然后我们将再创建两个列表：以 3 开头的 b 和以 4 开头的 c。然后 b 和 c 列表将继续到第一个包含 5 的 a 列表 和 10。换句话说，两个列表将共享第一个包含 5 和 10 的列表。

尝试使用我们的 List 定义和 Box<T> 来实现这个场景是行不通的，如清单 15-17 所示：

Filename: src/main.rs
``````
This code does not compile!
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use crate::List::{Cons, Nil};

fn main() {
    let a = Cons(5, Box::new(Cons(10, Box::new(Nil))));
    let b = Cons(3, Box::new(a));
    let c = Cons(4, Box::new(a));
}
``````
Listing 15-17: Demonstrating we’re not allowed to have two lists using Box<T> that try to share ownership of a third list

When we compile this code, we get this error:
``````
$ cargo run
   Compiling cons-list v0.1.0 (file:///projects/cons-list)
error[E0382]: use of moved value: `a`
  --> src/main.rs:11:30
   |
9  |     let a = Cons(5, Box::new(Cons(10, Box::new(Nil))));
   |         - move occurs because `a` has type `List`, which does not implement the `Copy` trait
10 |     let b = Cons(3, Box::new(a));
   |                              - value moved here
11 |     let c = Cons(4, Box::new(a));
   |                              ^ value used here after move

For more information about this error, try `rustc --explain E0382`.
error: could not compile `cons-list` due to previous error
``````
The Cons variants own the data they hold, so when we create the b list, a is moved into b and b owns a. Then, when we try to use a again when creating c, we’re not allowed to because a has been moved.

We could change the definition of Cons to hold references instead, but then we would have to specify lifetime parameters. By specifying lifetime parameters, we would be specifying that every element in the list will live at least as long as the entire list. This is the case for the elements and lists in Listing 15-17, but not in every scenario.

Instead, we’ll change our definition of List to use Rc<T> in place of Box<T>, as shown in Listing 15-18. Each Cons variant will now hold a value and an Rc<T> pointing to a List. When we create b, instead of taking ownership of a, we’ll clone the Rc<List> that a is holding, thereby increasing the number of references from one to two and letting a and b share ownership of the data in that Rc<List>. We’ll also clone a when creating c, increasing the number of references from two to three. Every time we call Rc::clone, the reference count to the data within the Rc<List> will increase, and the data won’t be cleaned up unless there are zero references to it.

Cons 变体拥有它们所保存的数据，因此当我们创建 b 列表时，a 被移动到 b 中，并且 b 拥有 a。 然后，当我们在创建 c 时尝试再次使用 a 时，我们将不允许这样做，因为 a 已被移动。

我们可以更改 Cons 的定义来保存引用，但随后我们必须指定生命周期参数。 通过指定生命周期参数，我们将指定列表中的每个元素的生存时间至少与整个列表一样长。 清单 15-17 中的元素和列表就是这种情况，但并非在所有情况下都是如此。

相反，我们将更改 List 的定义，使用 Rc<T> 代替 Box<T>，如清单 15-18 所示。 每个 Cons 变体现在将保存一个值和一个指向列表的 Rc<T> 。 当我们创建 b 时，我们将克隆 a 所持有的 Rc<List>，而不是获取 a 的所有权，从而将引用数量从 1 个增加到 2 个，并让 a 和 b 共享该 Rc< 中数据的所有权 列表>。 我们还将在创建 c 时克隆 a，将引用数量从两个增加到三个。 每次我们调用 Rc::clone 时，对 Rc<List> 中数据的引用计数都会增加，并且除非对它的引用为零，否则数据不会被清理。

Filename: src/main.rs
``````
enum List {
    Cons(i32, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::rc::Rc;

fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    let b = Cons(3, Rc::clone(&a));
    let c = Cons(4, Rc::clone(&a));
}
``````
Listing 15-18: A definition of List that uses Rc<T>

We need to add a use statement to bring Rc<T> into scope because it’s not in the prelude. In main, we create the list holding 5 and 10 and store it in a new Rc<List> in a. Then when we create b and c, we call the Rc::clone function and pass a reference to the Rc<List> in a as an argument.

We could have called a.clone() rather than Rc::clone(&a), but Rust’s convention is to use Rc::clone in this case. The implementation of Rc::clone doesn’t make a deep copy of all the data like most types’ implementations of clone do. The call to Rc::clone only increments the reference count, which doesn’t take much time. Deep copies of data can take a lot of time. By using Rc::clone for reference counting, we can visually distinguish between the deep-copy kinds of clones and the kinds of clones that increase the reference count. When looking for performance problems in the code, we only need to consider the deep-copy clones and can disregard calls to Rc::clone.

我们需要添加一条 use 语句将 Rc<T> 纳入作用域，因为它不在前奏中。 在 main 中，我们创建包含 5 和 10 的列表并将其存储在 a 中的新 Rc<List> 中。 然后，当我们创建 b 和 c 时，我们调用 Rc::clone 函数并将对 a 中的 Rc<List> 的引用作为参数传递。

我们可以调用 a.clone() 而不是 Rc::clone(&a)，但 Rust 的约定是在这种情况下使用 Rc::clone。 Rc::clone 的实现并不像大多数类型的克隆实现那样对所有数据进行深层复制。 对 Rc::clone 的调用仅增加引用计数，这不会花费太多时间。 数据的深层复制可能会花费大量时间。 通过使用Rc::clone进行引用计数，我们可以直观地区分深拷贝类型的克隆和增加引用计数的克隆类型。 在查找代码中的性能问题时，我们只需要考虑深拷贝克隆，可以忽略对 Rc::clone 的调用。

### Cloning an Rc<T> Increases the Reference Count
Let’s change our working example in Listing 15-18 so we can see the reference counts changing as we create and drop references to the Rc<List> in a.

In Listing 15-19, we’ll change main so it has an inner scope around list c; then we can see how the reference count changes when c goes out of scope.

让我们更改清单 15-18 中的工作示例，以便在创建和删除对 a 中的 Rc<List> 的引用时看到引用计数发生变化。

在清单 15-19 中，我们将更改 main，使其具有围绕列表 c 的内部作用域； 然后我们可以看到当 c 超出范围时引用计数如何变化。

Filename: src/main.rs
``````
fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    println!("count after creating a = {}", Rc::strong_count(&a));
    let b = Cons(3, Rc::clone(&a));
    println!("count after creating b = {}", Rc::strong_count(&a));
    {
        let c = Cons(4, Rc::clone(&a));
        println!("count after creating c = {}", Rc::strong_count(&a));
    }
    println!("count after c goes out of scope = {}", Rc::strong_count(&a));
}
``````
Listing 15-19: Printing the reference count

At each point in the program where the reference count changes, we print the reference count, which we get by calling the Rc::strong_count function. This function is named strong_count rather than count because the Rc<T> type also has a weak_count; we’ll see what weak_count is used for in the “Preventing Reference Cycles: Turning an Rc<T> into a Weak<T>” section.

在程序中引用计数发生变化的每个点，我们都会打印引用计数，这是通过调用 Rc::strong_count 函数获得的。 这个函数被命名为strong_count而不是count，因为Rc<T>类型也有一个weak_count； 我们将在“防止引用循环：将 Rc<T> 转换为 Weak<T>”部分中了解weak_count 的用途。

This code prints the following:
``````
$ cargo run
   Compiling cons-list v0.1.0 (file:///projects/cons-list)
    Finished dev [unoptimized + debuginfo] target(s) in 0.45s
     Running `target/debug/cons-list`
count after creating a = 1
count after creating b = 2
count after creating c = 3
count after c goes out of scope = 2
``````
We can see that the Rc<List> in a has an initial reference count of 1; then each time we call clone, the count goes up by 1. When c goes out of scope, the count goes down by 1. We don’t have to call a function to decrease the reference count like we have to call Rc::clone to increase the reference count: the implementation of the Drop trait decreases the reference count automatically when an Rc<T> value goes out of scope.

What we can’t see in this example is that when b and then a go out of scope at the end of main, the count is then 0, and the Rc<List> is cleaned up completely. Using Rc<T> allows a single value to have multiple owners, and the count ensures that the value remains valid as long as any of the owners still exist.

Via immutable references, Rc<T> allows you to share data between multiple parts of your program for reading only. If Rc<T> allowed you to have multiple mutable references too, you might violate one of the borrowing rules discussed in Chapter 4: multiple mutable borrows to the same place can cause data races and inconsistencies. But being able to mutate data is very useful! In the next section, we’ll discuss the interior mutability pattern and the RefCell<T> type that you can use in conjunction with an Rc<T> to work with this immutability restriction.

我们可以看到a中的Rc<List>初始引用计数为1； 那么每次我们调用clone时，计数就会增加1。当c超出范围时，计数就会减少1。我们不必像调用Rc:那样调用函数来减少引用计数： 克隆以增加引用计数：当 Rc<T> 值超出范围时，Drop 特征的实现会自动减少引用计数。

在这个例子中我们看不到的是，当 b 和 a 在 main 末尾超出范围时，计数为 0，并且 Rc<List> 被完全清理。 使用 Rc<T> 允许单个值拥有多个所有者，并且计数可确保只要任何所有者仍然存在，该值就保持有效。

通过不可变引用，Rc<T> 允许您在程序的多个部分之间共享数据以进行只读。 如果 Rc<T> 也允许您拥有多个可变引用，则可能会违反第 4 章中讨论的借用规则之一：对同一位置的多个可变借用可能会导致数据争用和不一致。 但能够改变数据是非常有用的！ 在下一节中，我们将讨论内部可变性模式和 RefCell<T> 类型，您可以将其与 Rc<T> 结合使用来处理此不变性限制。

## 15.4 RefCell<T> and the Interiour Mutability Pattern
Interior mutability is a design pattern in Rust that allows you to mutate data even when there are immutable references to that data; normally, this action is disallowed by the borrowing rules. To mutate data, the pattern uses unsafe code inside a data structure to bend Rust’s usual rules that govern mutation and borrowing. Unsafe code indicates to the compiler that we’re checking the rules manually instead of relying on the compiler to check them for us; we will discuss unsafe code more in Chapter 19.

We can use types that use the interior mutability pattern only when we can ensure that the borrowing rules will be followed at runtime, even though the compiler can’t guarantee that. The unsafe code involved is then wrapped in a safe API, and the outer type is still immutable.

Let’s explore this concept by looking at the RefCell<T> type that follows the interior mutability pattern.

Enforcing Borrowing Rules at Runtime with RefCell<T>
Unlike Rc<T>, the RefCell<T> type represents single ownership over the data it holds. So, what makes RefCell<T> different from a type like Box<T>? Recall the borrowing rules you learned in Chapter 4:

+ At any given time, you can have either (but not both) one mutable reference or any number of immutable references.
+ References must always be valid.

With references and Box<T>, the borrowing rules’ invariants are enforced at compile time. With RefCell<T>, these invariants are enforced at runtime. With references, if you break these rules, you’ll get a compiler error. With RefCell<T>, if you break these rules, your program will panic and exit.

The advantages of checking the borrowing rules at compile time are that errors will be caught sooner in the development process, and there is no impact on runtime performance because all the analysis is completed beforehand. For those reasons, checking the borrowing rules at compile time is the best choice in the majority of cases, which is why this is Rust’s default.

The advantage of checking the borrowing rules at runtime instead is that certain memory-safe scenarios are then allowed, where they would’ve been disallowed by the compile-time checks. Static analysis, like the Rust compiler, is inherently conservative. Some properties of code are impossible to detect by analyzing the code: the most famous example is the Halting Problem, which is beyond the scope of this book but is an interesting topic to research.

Because some analysis is impossible, if the Rust compiler can’t be sure the code complies with the ownership rules, it might reject a correct program; in this way, it’s conservative. If Rust accepted an incorrect program, users wouldn’t be able to trust in the guarantees Rust makes. However, if Rust rejects a correct program, the programmer will be inconvenienced, but nothing catastrophic can occur. The RefCell<T> type is useful when you’re sure your code follows the borrowing rules but the compiler is unable to understand and guarantee that.

Similar to Rc<T>, RefCell<T> is only for use in single-threaded scenarios and will give you a compile-time error if you try using it in a multithreaded context. We’ll talk about how to get the functionality of RefCell<T> in a multithreaded program in Chapter 16.

Here is a recap of the reasons to choose Box<T>, Rc<T>, or RefCell<T>:

内部可变性是 Rust 中的一种设计模式，即使存在对该数据的不可变引用，也允许您改变数据； 通常，借贷规则不允许这种行为。 为了改变数据，该模式在数据结构内使用不安全的代码来改变 Rust 管理突变和借用的通常规则。 不安全代码向编译器表明我们正在手动检查规则，而不是依赖编译器为我们检查规则； 我们将在第 19 章详细讨论不安全代码。

仅当我们可以确保在运行时遵循借用规则时，我们才能使用使用内部可变性模式的类型，即使编译器无法保证这一点。 然后，涉及的不安全代码被包装在安全的 API 中，并且外部类型仍然是不可变的。

让我们通过查看遵循内部可变性模式的 RefCell<T> 类型来探索这个概念。

使用 RefCell<T> 在运行时强制执行借用规则
与 Rc<T> 不同，RefCell<T> 类型表示对其所保存的数据的单一所有权。 那么，RefCell<T> 与 Box<T> 等类型有何不同？ 回想一下您在第 4 章中学到的借用规则：

+ 在任何给定时间，您可以拥有（但不能同时拥有）一个可变引用或任意数量的不可变引用。
+ 参考文献必须始终有效。

通过引用和 Box<T>，借用规则的不变量在编译时强制执行。 使用 RefCell<T>，这些不变量在运行时强制执行。 对于引用，如果违反这些规则，则会出现编译器错误。 对于 RefCell<T>，如果您违反这些规则，您的程序将出现恐慌并退出。

在编译时检查借用规则的优点是在开发过程中可以更快地捕获错误，并且由于所有分析都已提前完成，因此不会影响运行时性能。 出于这些原因，在大多数情况下，在编译时检查借用规则是最佳选择，这就是为什么这是 Rust 的默认设置。

相反，在运行时检查借用规则的优点是允许某些内存安全的场景，而编译时检查将不允许这些场景。 静态分析与 Rust 编译器一样，本质上是保守的。 代码的某些属性无法通过分析代码来检测：最著名的例子是停止问题，这超出了本书的范围，但却是一个有趣的研究主题。

因为有些分析是不可能的，如果 Rust 编译器不能确定代码符合所有权规则，它可能会拒绝正确的程序； 这样看来，它是保守的。 如果 Rust 接受了不正确的程序，用户将无法信任 Rust 所做的保证。 然而，如果 Rust 拒绝正确的程序，程序员会感到不便，但不会发生灾难性的情况。 当您确定代码遵循借用规则但编译器无法理解和保证这一点时，RefCell<T> 类型非常有用。

与 Rc<T> 类似，RefCell<T> 仅适用于单线程场景，如果您尝试在多线程上下文中使用它，则会出现编译时错误。 我们将在第 16 章讨论如何在多线程程序中获取 RefCell<T> 的功能。

以下是选择 Box<T>、Rc<T> 或 RefCell<T> 的原因的回顾：

+ Rc<T> enables multiple owners of the same data; Box<T> and RefCell<T> have single owners.
+ Box<T> allows immutable or mutable borrows checked at compile time; Rc<T> allows only immutable borrows checked at compile time; RefCell<T> allows immutable or mutable borrows checked at runtime.
+ Because RefCell<T> allows mutable borrows checked at runtime, you can mutate the value inside the RefCell<T> even when the RefCell<T> is immutable.

Mutating the value inside an immutable value is the interior mutability pattern. Let’s look at a situation in which interior mutability is useful and examine how it’s possible.

改变不可变值内部的值是内部可变性模式。 让我们看一下内部可变性有用的情况，并研究它是如何实现的。

### Interior Mutability: A Mutable Borrow to an Immutable Value
A consequence of the borrowing rules is that when you have an immutable value, you can’t borrow it mutably. For example, this code won’t compile:
``````
This code does not compile!
fn main() {
    let x = 5;
    let y = &mut x;
}
``````
If you tried to compile this code, you’d get the following error:
``````
$ cargo run
   Compiling borrowing v0.1.0 (file:///projects/borrowing)
error[E0596]: cannot borrow `x` as mutable, as it is not declared as mutable
 --> src/main.rs:3:13
  |
2 |     let x = 5;
  |         - help: consider changing this to be mutable: `mut x`
3 |     let y = &mut x;
  |             ^^^^^^ cannot borrow as mutable

For more information about this error, try `rustc --explain E0596`.
error: could not compile `borrowing` due to previous error
``````
However, there are situations in which it would be useful for a value to mutate itself in its methods but appear immutable to other code. Code outside the value’s methods would not be able to mutate the value. Using RefCell<T> is one way to get the ability to have interior mutability, but RefCell<T> doesn’t get around the borrowing rules completely: the borrow checker in the compiler allows this interior mutability, and the borrowing rules are checked at runtime instead. If you violate the rules, you’ll get a panic! instead of a compiler error.

Let’s work through a practical example where we can use RefCell<T> to mutate an immutable value and see why that is useful.

然而，在某些情况下，一个值在其方法中改变自身是有用的，但对于其他代码来说似乎是不可变的。 值方法之外的代码将无法改变该值。 使用 RefCell<T> 是获得内部可变性能力的一种方法，但 RefCell<T> 并不能完全绕过借用规则：编译器中的借用检查器允许这种内部可变性，并且借用规则在 而是运行时。 如果你违反了规则，你会感到恐慌！ 而不是编译器错误。

让我们看一个实际的例子，我们可以使用 RefCell<T> 来改变一个不可变的值，看看为什么它有用。

### A Use Case for Interior Mutability: Mock Objects
Sometimes during testing a programmer will use a type in place of another type, in order to observe particular behavior and assert it’s implemented correctly. This placeholder type is called a test double. Think of it in the sense of a “stunt double” in filmmaking, where a person steps in and substitutes for an actor to do a particular tricky scene. Test doubles stand in for other types when we’re running tests. Mock objects are specific types of test doubles that record what happens during a test so you can assert that the correct actions took place.

Rust doesn’t have objects in the same sense as other languages have objects, and Rust doesn’t have mock object functionality built into the standard library as some other languages do. However, you can definitely create a struct that will serve the same purposes as a mock object.

Here’s the scenario we’ll test: we’ll create a library that tracks a value against a maximum value and sends messages based on how close to the maximum value the current value is. This library could be used to keep track of a user’s quota for the number of API calls they’re allowed to make, for example.

Our library will only provide the functionality of tracking how close to the maximum a value is and what the messages should be at what times. Applications that use our library will be expected to provide the mechanism for sending the messages: the application could put a message in the application, send an email, send a text message, or something else. The library doesn’t need to know that detail. All it needs is something that implements a trait we’ll provide called Messenger. Listing 15-20 shows the library code:

有时，在测试过程中，程序员会使用一种类型代替另一种类型，以便观察特定行为并断言其实现正确。 这种占位符类型称为测试替身。 可以把它想象成电影制作中的“替身”，一个人介入并替代演员来完成一个特定的棘手场景。 当我们运行测试时，测试替身代表其他类型。 模拟对象是特定类型的测试替身，用于记录测试期间发生的情况，以便您可以断言发生了正确的操作。

Rust 不具有与其他语言具有对象相同意义上的对象，并且 Rust 没有像其他语言那样将模拟对象功能内置到标准库中。 但是，您绝对可以创建一个与模拟对象具有相同用途的结构。

这是我们要测试的场景：我们将创建一个库，该库根据最大值跟踪一个值，并根据当前值与最大值的接近程度发送消息。 例如，该库可用于跟踪用户允许进行的 API 调用数量的配额。

我们的库将仅提供跟踪值与最大值的接近程度以及消息应该在什么时间发出的功能。 使用我们的库的应用程序预计将提供发送消息的机制：应用程序可以在应用程序中放置消息、发送电子邮件、发送文本消息或其他内容。 图书馆不需要知道这个细节。 它所需要的只是实现我们将提供的称为 Messenger 的特征。 清单 15-20 显示了库代码：

Filename: src/lib.rs
``````
pub trait Messenger {
    fn send(&self, msg: &str);
}

pub struct LimitTracker<'a, T: Messenger> {
    messenger: &'a T,
    value: usize,
    max: usize,
}

impl<'a, T> LimitTracker<'a, T>
where
    T: Messenger,
{
    pub fn new(messenger: &'a T, max: usize) -> LimitTracker<'a, T> {
        LimitTracker {
            messenger,
            value: 0,
            max,
        }
    }

    pub fn set_value(&mut self, value: usize) {
        self.value = value;

        let percentage_of_max = self.value as f64 / self.max as f64;

        if percentage_of_max >= 1.0 {
            self.messenger.send("Error: You are over your quota!");
        } else if percentage_of_max >= 0.9 {
            self.messenger
                .send("Urgent warning: You've used up over 90% of your quota!");
        } else if percentage_of_max >= 0.75 {
            self.messenger
                .send("Warning: You've used up over 75% of your quota!");
        }
    }
}
``````
Listing 15-20: A library to keep track of how close a value is to a maximum value and warn when the value is at certain levels

One important part of this code is that the Messenger trait has one method called send that takes an immutable reference to self and the text of the message. This trait is the interface our mock object needs to implement so that the mock can be used in the same way a real object is. The other important part is that we want to test the behavior of the set_value method on the LimitTracker. We can change what we pass in for the value parameter, but set_value doesn’t return anything for us to make assertions on. We want to be able to say that if we create a LimitTracker with something that implements the Messenger trait and a particular value for max, when we pass different numbers for value, the messenger is told to send the appropriate messages.

We need a mock object that, instead of sending an email or text message when we call send, will only keep track of the messages it’s told to send. We can create a new instance of the mock object, create a LimitTracker that uses the mock object, call the set_value method on LimitTracker, and then check that the mock object has the messages we expect. Listing 15-21 shows an attempt to implement a mock object to do just that, but the borrow checker won’t allow it:

此代码的一个重要部分是 Messenger 特征有一种名为 send 的方法，该方法采用对 self 和消息文本的不可变引用。 这个特征是我们的模拟对象需要实现的接口，以便模拟可以像真实对象一样使用。 另一个重要部分是我们要测试 LimitTracker 上 set_value 方法的行为。 我们可以更改为 value 参数传递的内容，但 set_value 不会返回任何内容供我们进行断言。 我们希望能够说，如果我们使用实现 Messenger 特征和 max 的特定值的东西创建一个 LimitTracker，那么当我们传递不同的数字作为值时，信使会被告知发送适当的消息。

我们需要一个模拟对象，当我们调用 send 时，它不会发送电子邮件或短信，而只会跟踪它被告知发送的消息。 我们可以创建一个新的模拟对象实例，创建一个使用该模拟对象的LimitTracker，调用LimitTracker上的set_value方法，然后检查该模拟对象是否具有我们期望的消息。 清单 15-21 展示了尝试实现一个模拟对象来做到这一点，但借用检查器不允许这样做：

Filename: src/lib.rs
``````
This code does not compile!
#[cfg(test)]
mod tests {
    use super::*;

    struct MockMessenger {
        sent_messages: Vec<String>,
    }

    impl MockMessenger {
        fn new() -> MockMessenger {
            MockMessenger {
                sent_messages: vec![],
            }
        }
    }

    impl Messenger for MockMessenger {
        fn send(&self, message: &str) {
            self.sent_messages.push(String::from(message));
        }
    }

    #[test]
    fn it_sends_an_over_75_percent_warning_message() {
        let mock_messenger = MockMessenger::new();
        let mut limit_tracker = LimitTracker::new(&mock_messenger, 100);

        limit_tracker.set_value(80);

        assert_eq!(mock_messenger.sent_messages.len(), 1);
    }
}
``````
Listing 15-21: An attempt to implement a MockMessenger that isn’t allowed by the borrow checker

This test code defines a MockMessenger struct that has a sent_messages field with a Vec of String values to keep track of the messages it’s told to send. We also define an associated function new to make it convenient to create new MockMessenger values that start with an empty list of messages. We then implement the Messenger trait for MockMessenger so we can give a MockMessenger to a LimitTracker. In the definition of the send method, we take the message passed in as a parameter and store it in the MockMessenger list of sent_messages.

In the test, we’re testing what happens when the LimitTracker is told to set value to something that is more than 75 percent of the max value. First, we create a new MockMessenger, which will start with an empty list of messages. Then we create a new LimitTracker and give it a reference to the new MockMessenger and a max value of 100. We call the set_value method on the LimitTracker with a value of 80, which is more than 75 percent of 100. Then we assert that the list of messages that the MockMessenger is keeping track of should now have one message in it.

此测试代码定义了一个 MockMessenger 结构体，该结构体具有一个 sent_messages 字段，其中包含一个 String 值的 Vec，用于跟踪被告知要发送的消息。 我们还定义了一个关联函数 new ，以便于创建以空消息列表开头的新 MockMessenger 值。 然后，我们为 MockMessenger 实现 Messenger 特征，这样我们就可以将 MockMessenger 提供给 LimitTracker。 在send方法的定义中，我们将传入的消息作为参数，存储在MockMessenger的sent_messages列表中。

在测试中，我们正在测试当 LimitTracker 被告知将值设置为大于最大值 75% 时会发生什么。 首先，我们创建一个新的 MockMessenger，它将以空消息列表开始。 然后我们创建一个新的 LimitTracker 并给它一个新的 MockMessenger 的引用和最大值 100。我们调用 LimitTracker 上的 set_value 方法，值为 80，超过 100 的 75%。然后我们断言 MockMessenger 跟踪的消息列表现在应该包含一条消息。

However, there’s one problem with this test, as shown here:
``````
$ cargo test
   Compiling limit-tracker v0.1.0 (file:///projects/limit-tracker)
error[E0596]: cannot borrow `self.sent_messages` as mutable, as it is behind a `&` reference
  --> src/lib.rs:58:13
   |
2  |     fn send(&self, msg: &str);
   |             ----- help: consider changing that to be a mutable reference: `&mut self`
...
58 |             self.sent_messages.push(String::from(message));
   |             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ `self` is a `&` reference, so the data it refers to cannot be borrowed as mutable

For more information about this error, try `rustc --explain E0596`.
error: could not compile `limit-tracker` due to previous error
warning: build failed, waiting for other jobs to finish...
``````
We can’t modify the MockMessenger to keep track of the messages, because the send method takes an immutable reference to self. We also can’t take the suggestion from the error text to use &mut self instead, because then the signature of send wouldn’t match the signature in the Messenger trait definition (feel free to try and see what error message you get).

This is a situation in which interior mutability can help! We’ll store the sent_messages within a RefCell<T>, and then the send method will be able to modify sent_messages to store the messages we’ve seen. Listing 15-22 shows what that looks like:

我们无法修改 MockMessenger 来跟踪消息，因为 send 方法采用对 self 的不可变引用。 我们也不能从错误文本中获取建议来使用 &mut self 来代替，因为这样 send 的签名将与 Messenger 特征定义中的签名不匹配（请随意尝试看看您会收到什么错误消息）。

在这种情况下，内部可变性可以提供帮助！ 我们将 send_messages 存储在 RefCell<T> 中，然后 send 方法将能够修改 sent_messages 以存储我们看到的消息。 清单 15-22 显示了它的样子：
Filename: src/lib.rs
``````
#[cfg(test)]
mod tests {
    use super::*;
    use std::cell::RefCell;

    struct MockMessenger {
        sent_messages: RefCell<Vec<String>>,
    }

    impl MockMessenger {
        fn new() -> MockMessenger {
            MockMessenger {
                sent_messages: RefCell::new(vec![]),
            }
        }
    }

    impl Messenger for MockMessenger {
        fn send(&self, message: &str) {
            self.sent_messages.borrow_mut().push(String::from(message));
        }
    }

    #[test]
    fn it_sends_an_over_75_percent_warning_message() {
        // --snip--

        assert_eq!(mock_messenger.sent_messages.borrow().len(), 1);
    }
}
``````
Listing 15-22: Using RefCell<T> to mutate an inner value while the outer value is considered immutable

The sent_messages field is now of type RefCell<Vec<String>> instead of Vec<String>. In the new function, we create a new RefCell<Vec<String>> instance around the empty vector.

For the implementation of the send method, the first parameter is still an immutable borrow of self, which matches the trait definition. We call borrow_mut on the RefCell<Vec<String>> in self.sent_messages to get a mutable reference to the value inside the RefCell<Vec<String>>, which is the vector. Then we can call push on the mutable reference to the vector to keep track of the messages sent during the test.

The last change we have to make is in the assertion: to see how many items are in the inner vector, we call borrow on the RefCell<Vec<String>> to get an immutable reference to the vector.

Now that you’ve seen how to use RefCell<T>, let’s dig into how it works!

sent_messages 字段现在的类型为 RefCell<Vec<String>> 而不是 Vec<String>。 在新函数中，我们围绕空向量创建一个新的 RefCell<Vec<String>> 实例。

对于 send 方法的实现，第一个参数仍然是 self 的不可变借用，它与特征定义匹配。 我们在 self.sent_messages 中对 RefCell<Vec<String>> 调用borrow_mut，以获取对 RefCell<Vec<String>> 内的值（即向量）的可变引用。 然后我们可以对向量的可变引用调用 Push 来跟踪测试期间发送的消息。

我们必须做的最后一个更改是在断言中：要查看内部向量中有多少项，我们对 RefCell<Vec<String>> 调用借用以获取对该向量的不可变引用。

现在您已经了解了如何使用 RefCell<T>，让我们深入了解它是如何工作的！

### Keeping Track of Borrows at Runtime with RefCell<T>
When creating immutable and mutable references, we use the & and &mut syntax, respectively. With RefCell<T>, we use the borrow and borrow_mut methods, which are part of the safe API that belongs to RefCell<T>. The borrow method returns the smart pointer type Ref<T>, and borrow_mut returns the smart pointer type RefMut<T>. Both types implement Deref, so we can treat them like regular references.

The RefCell<T> keeps track of how many Ref<T> and RefMut<T> smart pointers are currently active. Every time we call borrow, the RefCell<T> increases its count of how many immutable borrows are active. When a Ref<T> value goes out of scope, the count of immutable borrows goes down by one. Just like the compile-time borrowing rules, RefCell<T> lets us have many immutable borrows or one mutable borrow at any point in time.

If we try to violate these rules, rather than getting a compiler error as we would with references, the implementation of RefCell<T> will panic at runtime. Listing 15-23 shows a modification of the implementation of send in Listing 15-22. We’re deliberately trying to create two mutable borrows active for the same scope to illustrate that RefCell<T> prevents us from doing this at runtime.

创建不可变和可变引用时，我们分别使用 & 和 &mut 语法。 对于RefCell<T>，我们使用borrow 和borrow_mut 方法，它们是属于RefCell<T> 的安全API 的一部分。 borrow 方法返回智能指针类型 Ref<T>，borrow_mut 返回智能指针类型 RefMut<T>。 这两种类型都实现了 Deref，因此我们可以将它们视为常规引用。

RefCell<T> 跟踪当前有多少 Ref<T> 和 RefMut<T> 智能指针处于活动状态。 每次我们调用借用时，RefCell<T> 都会增加其活动的不可变借用数量的计数。 当 Ref<T> 值超出范围时，不可变借用的计数就会减一。 就像编译时借用规则一样，RefCell<T> 让我们可以在任何时间点拥有许多不可变借用或一个可变借用。

如果我们尝试违反这些规则，RefCell<T> 的实现将在运行时出现恐慌，而不是像引用引用那样得到编译器错误。 清单 15-23 显示了清单 15-22 中 send 实现的修改。 我们故意尝试为同一范围创建两个活动的可变借用，以说明 RefCell<T> 阻止我们在运行时执行此操作。

Filename: src/lib.rs
``````
This code panics!
    impl Messenger for MockMessenger {
        fn send(&self, message: &str) {
            let mut one_borrow = self.sent_messages.borrow_mut();
            let mut two_borrow = self.sent_messages.borrow_mut();

            one_borrow.push(String::from(message));
            two_borrow.push(String::from(message));
        }
    }
``````
Listing 15-23: Creating two mutable references in the same scope to see that RefCell<T> will panic

We create a variable one_borrow for the RefMut<T> smart pointer returned from borrow_mut. Then we create another mutable borrow in the same way in the variable two_borrow. This makes two mutable references in the same scope, which isn’t allowed. When we run the tests for our library, the code in Listing 15-23 will compile without any errors, but the test will fail:

我们为从borrow_mut返回的RefMut<T>智能指针创建一个变量one_borrow。 然后我们以相同的方式在变量two_borrow中创建另一个可变借用。 这使得两个可变引用处于同一范围内，这是不允许的。 当我们对我们的库运行测试时，清单 15-23 中的代码将编译没有任何错误，但测试将失败：

``````
$ cargo test
   Compiling limit-tracker v0.1.0 (file:///projects/limit-tracker)
    Finished test [unoptimized + debuginfo] target(s) in 0.91s
     Running unittests src/lib.rs (target/debug/deps/limit_tracker-e599811fa246dbde)

running 1 test
test tests::it_sends_an_over_75_percent_warning_message ... FAILED

failures:

---- tests::it_sends_an_over_75_percent_warning_message stdout ----
thread 'tests::it_sends_an_over_75_percent_warning_message' panicked at 'already borrowed: BorrowMutError', src/lib.rs:60:53
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::it_sends_an_over_75_percent_warning_message

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass `--lib`
``````
Notice that the code panicked with the message already borrowed: BorrowMutError. This is how RefCell<T> handles violations of the borrowing rules at runtime.

Choosing to catch borrowing errors at runtime rather than compile time, as we’ve done here, means you’d potentially be finding mistakes in your code later in the development process: possibly not until your code was deployed to production. Also, your code would incur a small runtime performance penalty as a result of keeping track of the borrows at runtime rather than compile time. However, using RefCell<T> makes it possible to write a mock object that can modify itself to keep track of the messages it has seen while you’re using it in a context where only immutable values are allowed. You can use RefCell<T> despite its trade-offs to get more functionality than regular references provide.

请注意，代码因已借用的消息而发生恐慌：BorrowMutError。 这就是 RefCell<T> 在运行时处理违反借用规则的方式。

选择在运行时而不是编译时捕获借用错误（正如我们在这里所做的那样），意味着您可能会在开发过程的后期发现代码中的错误：可能直到您的代码部署到生产环境后才会发现。 此外，由于在运行时而不是编译时跟踪借用，您的代码会产生较小的运行时性能损失。 但是，使用 RefCell<T> 可以编写一个模拟对象，该对象可以修改自身以跟踪在仅允许不可变值的上下文中使用它时所看到的消息。 尽管需要权衡，但您可以使用 RefCell<T> 来获得比常规引用提供的更多功能。

### Having Multiple Owners of Mutable Data by Combining Rc<T> and RefCell<T>
A common way to use RefCell<T> is in combination with Rc<T>. Recall that Rc<T> lets you have multiple owners of some data, but it only gives immutable access to that data. If you have an Rc<T> that holds a RefCell<T>, you can get a value that can have multiple owners and that you can mutate!

For example, recall the cons list example in Listing 15-18 where we used Rc<T> to allow multiple lists to share ownership of another list. Because Rc<T> holds only immutable values, we can’t change any of the values in the list once we’ve created them. Let’s add in RefCell<T> to gain the ability to change the values in the lists. Listing 15-24 shows that by using a RefCell<T> in the Cons definition, we can modify the value stored in all the lists:

使用 RefCell<T> 的常见方法是与 Rc<T> 结合使用。 回想一下，Rc<T> 允许您拥有某些数据的多个所有者，但它只提供对该数据的不可变访问。 如果您有一个包含 RefCell<T> 的 Rc<T>，您可以获得一个可以有多个所有者并且可以变异的值！

例如，回想一下清单 15-18 中的 cons 列表示例，其中我们使用 Rc<T> 来允许多个列表共享另一个列表的所有权。 因为 Rc<T> 仅保存不可变值，所以一旦创建了列表中的任何值，我们就无法更改它们。 让我们添加 RefCell<T> 以获得更改列表中的值的能力。 清单 15-24 显示，通过在 Cons 定义中使用 RefCell<T>，我们可以修改存储在所有列表中的值：

Filename: src/main.rs
``````
#[derive(Debug)]
enum List {
    Cons(Rc<RefCell<i32>>, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::cell::RefCell;
use std::rc::Rc;

fn main() {
    let value = Rc::new(RefCell::new(5));

    let a = Rc::new(Cons(Rc::clone(&value), Rc::new(Nil)));

    let b = Cons(Rc::new(RefCell::new(3)), Rc::clone(&a));
    let c = Cons(Rc::new(RefCell::new(4)), Rc::clone(&a));

    *value.borrow_mut() += 10;

    println!("a after = {:?}", a);
    println!("b after = {:?}", b);
    println!("c after = {:?}", c);
}
``````
Listing 15-24: Using Rc<RefCell<i32>> to create a List that we can mutate

We create a value that is an instance of Rc<RefCell<i32>> and store it in a variable named value so we can access it directly later. Then we create a List in a with a Cons variant that holds value. We need to clone value so both a and value have ownership of the inner 5 value rather than transferring ownership from value to a or having a borrow from value.

We wrap the list a in an Rc<T> so when we create lists b and c, they can both refer to a, which is what we did in Listing 15-18.

After we’ve created the lists in a, b, and c, we want to add 10 to the value in value. We do this by calling borrow_mut on value, which uses the automatic dereferencing feature we discussed in Chapter 5 (see the section “Where’s the -> Operator?”) to dereference the Rc<T> to the inner RefCell<T> value. The borrow_mut method returns a RefMut<T> smart pointer, and we use the dereference operator on it and change the inner value.

我们创建一个作为 Rc<RefCell<i32>> 实例的值，并将其存储在名为 value 的变量中，以便稍后可以直接访问它。 然后我们在 a 中创建一个 List，其中包含保存值的 Cons 变体。 我们需要克隆 value，以便 a 和 value 都拥有内部 5 值的所有权，而不是将所有权从 value 转移到 a 或从 value 借用。

我们将列表 a 包装在 Rc<T> 中，因此当我们创建列表 b 和 c 时，它们都可以引用 a，这就是我们在清单 15-18 中所做的。

在创建了 a、b 和 c 中的列表后，我们要将 value 中的值加 10。 我们通过对 value 调用borrow_mut 来实现这一点，它使用我们在第 5 章中讨论的自动解引用功能（请参阅“-> 运算符在哪里？”一节）来解引用 Rc<T> 到内部 RefCell<T> 值。 borrow_mut 方法返回一个 RefMut<T> 智能指针，我们对其使用解引用运算符并更改内部值。

When we print a, b, and c, we can see that they all have the modified value of 15 rather than 5:
``````
$ cargo run
   Compiling cons-list v0.1.0 (file:///projects/cons-list)
    Finished dev [unoptimized + debuginfo] target(s) in 0.63s
     Running `target/debug/cons-list`
a after = Cons(RefCell { value: 15 }, Nil)
b after = Cons(RefCell { value: 3 }, Cons(RefCell { value: 15 }, Nil))
c after = Cons(RefCell { value: 4 }, Cons(RefCell { value: 15 }, Nil))
``````
This technique is pretty neat! By using RefCell<T>, we have an outwardly immutable List value. But we can use the methods on RefCell<T> that provide access to its interior mutability so we can modify our data when we need to. The runtime checks of the borrowing rules protect us from data races, and it’s sometimes worth trading a bit of speed for this flexibility in our data structures. Note that RefCell<T> does not work for multithreaded code! Mutex<T> is the thread-safe version of RefCell<T> and we’ll discuss Mutex<T> in Chapter 16.

这个技术非常巧妙！ 通过使用 RefCell<T>，我们有一个表面上不可变的 List 值。 但是我们可以使用 RefCell<T> 上的方法来访问其内部可变性，以便我们可以在需要时修改数据。 借用规则的运行时检查可以保护我们免受数据竞争的影响，有时为了数据结构的灵活性而牺牲一点速度是值得的。 请注意，RefCell<T> 不适用于多线程代码！ Mutex<T> 是 RefCell<T> 的线程安全版本，我们将在第 16 章讨论 Mutex<T>。

## 15.6 Reference Cycles Can Leak Memory

Rust’s memory safety guarantees make it difficult, but not impossible, to accidentally create memory that is never cleaned up (known as a memory leak). Preventing memory leaks entirely is not one of Rust’s guarantees, meaning memory leaks are memory safe in Rust. We can see that Rust allows memory leaks by using Rc<T> and RefCell<T>: it’s possible to create references where items refer to each other in a cycle. This creates memory leaks because the reference count of each item in the cycle will never reach 0, and the values will never be dropped.

Rust 的内存安全保证使得意外创建从未清理过的内存（称为内存泄漏）变得困难，但并非不可能。 完全防止内存泄漏并不是 Rust 的保证之一，这意味着内存泄漏在 Rust 中是内存安全的。 我们可以看到，Rust 通过使用 Rc<T> 和 RefCell<T> 允许内存泄漏：可以创建引用，其中项在循环中相互引用。 这会造成内存泄漏，因为循环中每个项目的引用计数永远不会达到 0，并且值永远不会被删除。


### Creating a Reference Cycle
Let’s look at how a reference cycle might happen and how to prevent it, starting with the definition of the List enum and a tail method in Listing 15-25:

让我们从清单 15-25 中 List 枚举和 tail 方法的定义开始，看看引用循环是如何发生的以及如何防止它：

Filename: src/main.rs
``````
use crate::List::{Cons, Nil};
use std::cell::RefCell;
use std::rc::Rc;

#[derive(Debug)]
enum List {
    Cons(i32, RefCell<Rc<List>>),
    Nil,
}

impl List {
    fn tail(&self) -> Option<&RefCell<Rc<List>>> {
        match self {
            Cons(_, item) => Some(item),
            Nil => None,
        }
    }
}

fn main() {}
``````
Listing 15-25: A cons list definition that holds a RefCell<T> so we can modify what a Cons variant is referring to

We’re using another variation of the List definition from Listing 15-5. The second element in the Cons variant is now RefCell<Rc<List>>, meaning that instead of having the ability to modify the i32 value as we did in Listing 15-24, we want to modify the List value a Cons variant is pointing to. We’re also adding a tail method to make it convenient for us to access the second item if we have a Cons variant.

In Listing 15-26, we’re adding a main function that uses the definitions in Listing 15-25. This code creates a list in a and a list in b that points to the list in a. Then it modifies the list in a to point to b, creating a reference cycle. There are println! statements along the way to show what the reference counts are at various points in this process.

我们使用清单 15-5 中 List 定义的另一种变体。 Cons 变体中的第二个元素现在是 RefCell<Rc<List>>，这意味着我们不想像清单 15-24 中那样修改 i32 值，而是要修改 Cons 变体指向的 List 值 到。 我们还添加了一个 tail 方法，以便在有 Cons 变体时方便我们访问第二个项目。

在清单 15-26 中，我们添加了一个使用清单 15-25 中的定义的 main 函数。 此代码在 a 中创建一个列表，并在 b 中创建一个指向 a 中列表的列表。 然后它修改a中的列表以指向b，创建一个引用循环。 有打印！ 沿途的语句来显示在此过程中各个点的引用计数。

Filename: src/main.rs
``````
fn main() {
    let a = Rc::new(Cons(5, RefCell::new(Rc::new(Nil))));

    println!("a initial rc count = {}", Rc::strong_count(&a));
    println!("a next item = {:?}", a.tail());

    let b = Rc::new(Cons(10, RefCell::new(Rc::clone(&a))));

    println!("a rc count after b creation = {}", Rc::strong_count(&a));
    println!("b initial rc count = {}", Rc::strong_count(&b));
    println!("b next item = {:?}", b.tail());

    if let Some(link) = a.tail() {
        *link.borrow_mut() = Rc::clone(&b);
    }

    println!("b rc count after changing a = {}", Rc::strong_count(&b));
    println!("a rc count after changing a = {}", Rc::strong_count(&a));

    // Uncomment the next line to see that we have a cycle;
    // it will overflow the stack
    // println!("a next item = {:?}", a.tail());
}
``````
Listing 15-26: Creating a reference cycle of two List values pointing to each other

We create an Rc<List> instance holding a List value in the variable a with an initial list of 5, Nil. We then create an Rc<List> instance holding another List value in the variable b that contains the value 10 and points to the list in a.

We modify a so it points to b instead of Nil, creating a cycle. We do that by using the tail method to get a reference to the RefCell<Rc<List>> in a, which we put in the variable link. Then we use the borrow_mut method on the RefCell<Rc<List>> to change the value inside from an Rc<List> that holds a Nil value to the Rc<List> in b.

我们创建一个 Rc<List> 实例，在变量 a 中保存一个 List 值，初始列表为 5，Nil。 然后，我们创建一个 Rc<List> 实例，在变量 b 中保存另一个 List 值，该值包含值 10 并指向 a 中的列表。

我们修改 a 使其指向 b 而不是 Nil，从而创建一个循环。 为此，我们使用 tail 方法获取对 a 中的 RefCell<Rc<List>> 的引用，并将其放入变量链接中。 然后我们使用 RefCell<Rc<List>> 上的borrow_mut 方法将内部的值从保存 Nil 值的 Rc<List> 更改为 b 中的 Rc<List>。

When we run this code, keeping the last println! commented out for the moment, we’ll get this output:
``````
$ cargo run
   Compiling cons-list v0.1.0 (file:///projects/cons-list)
    Finished dev [unoptimized + debuginfo] target(s) in 0.53s
     Running `target/debug/cons-list`
a initial rc count = 1
a next item = Some(RefCell { value: Nil })
a rc count after b creation = 2
b initial rc count = 1
b next item = Some(RefCell { value: Cons(5, RefCell { value: Nil }) })
b rc count after changing a = 2
a rc count after changing a = 2
``````
The reference count of the Rc<List> instances in both a and b are 2 after we change the list in a to point to b. At the end of main, Rust drops the variable b, which decreases the reference count of the b Rc<List> instance from 2 to 1. The memory that Rc<List> has on the heap won’t be dropped at this point, because its reference count is 1, not 0. Then Rust drops a, which decreases the reference count of the a Rc<List> instance from 2 to 1 as well. This instance’s memory can’t be dropped either, because the other Rc<List> instance still refers to it. The memory allocated to the list will remain uncollected forever. To visualize this reference cycle, we’ve created a diagram in Figure 15-4.

当我们将 a 中的列表更改为指向 b 后，a 和 b 中的 Rc<List> 实例的引用计数均为 2。 在 main 的末尾，Rust 删除了变量 b，这将 b Rc<List> 实例的引用计数从 2 减少到 1。此时 Rc<List> 在堆上的内存不会被删除， 因为它的引用计数是 1，而不是 0。然后 Rust 删除 a，这也会将 a Rc<List> 实例的引用计数从 2 减少到 1。 该实例的内存也无法删除，因为另一个 Rc<List> 实例仍然引用它。 分配给列表的内存将永远保持未回收状态。 为了形象化这个引用循环，我们创建了一个图，如图 15-4 所示。

Reference cycle of lists
Figure 15-4: A reference cycle of lists a and b pointing to each other

If you uncomment the last println! and run the program, Rust will try to print this cycle with a pointing to b pointing to a and so forth until it overflows the stack.

Compared to a real-world program, the consequences of creating a reference cycle in this example aren’t very dire: right after we create the reference cycle, the program ends. However, if a more complex program allocated lots of memory in a cycle and held onto it for a long time, the program would use more memory than it needed and might overwhelm the system, causing it to run out of available memory.

Creating reference cycles is not easily done, but it’s not impossible either. If you have RefCell<T> values that contain Rc<T> values or similar nested combinations of types with interior mutability and reference counting, you must ensure that you don’t create cycles; you can’t rely on Rust to catch them. Creating a reference cycle would be a logic bug in your program that you should use automated tests, code reviews, and other software development practices to minimize.

Another solution for avoiding reference cycles is reorganizing your data structures so that some references express ownership and some references don’t. As a result, you can have cycles made up of some ownership relationships and some non-ownership relationships, and only the ownership relationships affect whether or not a value can be dropped. In Listing 15-25, we always want Cons variants to own their list, so reorganizing the data structure isn’t possible. Let’s look at an example using graphs made up of parent nodes and child nodes to see when non-ownership relationships are an appropriate way to prevent reference cycles.

如果取消最后一个 println! 的注释 并运行程序，Rust 将尝试打印此循环，其中 a 指向 b 指向 a 等等，直到溢出堆栈。

与现实世界的程序相比，在这个例子中创建引用循环的后果并不是很可怕：在我们创建引用循环之后，程序就结束了。 但是，如果更复杂的程序在一个周期中分配大量内存并长时间保留它，则该程序将使用超出其所需的内存，并可能压垮系统，导致可用内存耗尽。

创建参考循环并不容易，但也不是不可能。 如果您的 RefCell<T> 值包含 Rc<T> 值或具有内部可变性和引用计数的类似嵌套类型组合，则必须确保不会创建循环； 你不能依靠 Rust 来捕获它们。 创建引用循环将是程序中的逻辑错误，您应该使用自动化测试、代码审查和其他软件开发实践来最大程度地减少该错误。

避免引用循环的另一个解决方案是重新组织数据结构，以便某些引用表达所有权，而另一些引用则不表达所有权。 因此，您可以拥有由一些所有权关系和一些非所有权关系组成的循环，并且只有所有权关系会影响是否可以删除值。 在清单 15-25 中，我们总是希望 Cons 变体拥有自己的列表，因此重新组织数据结构是不可能的。 让我们看一个使用由父节点和子节点组成的图的示例，看看非所有权关系何时是防止引用循环的适当方法。

### Preventing Reference Cycles: Turning an Rc<T> into a Weak<T>
So far, we’ve demonstrated that calling Rc::clone increases the strong_count of an Rc<T> instance, and an Rc<T> instance is only cleaned up if its strong_count is 0. You can also create a weak reference to the value within an Rc<T> instance by calling Rc::downgrade and passing a reference to the Rc<T>. Strong references are how you can share ownership of an Rc<T> instance. Weak references don’t express an ownership relationship, and their count doesn’t affect when an Rc<T> instance is cleaned up. They won’t cause a reference cycle because any cycle involving some weak references will be broken once the strong reference count of values involved is 0.

When you call Rc::downgrade, you get a smart pointer of type Weak<T>. Instead of increasing the strong_count in the Rc<T> instance by 1, calling Rc::downgrade increases the weak_count by 1. The Rc<T> type uses weak_count to keep track of how many Weak<T> references exist, similar to strong_count. The difference is the weak_count doesn’t need to be 0 for the Rc<T> instance to be cleaned up.

Because the value that Weak<T> references might have been dropped, to do anything with the value that a Weak<T> is pointing to, you must make sure the value still exists. Do this by calling the upgrade method on a Weak<T> instance, which will return an Option<Rc<T>>. You’ll get a result of Some if the Rc<T> value has not been dropped yet and a result of None if the Rc<T> value has been dropped. Because upgrade returns an Option<Rc<T>>, Rust will ensure that the Some case and the None case are handled, and there won’t be an invalid pointer.

As an example, rather than using a list whose items know only about the next item, we’ll create a tree whose items know about their children items and their parent items.

到目前为止，我们已经演示了调用 Rc::clone 会增加 Rc<T> 实例的 Strong_count，并且 Rc<T> 实例仅在其 Strong_count 为 0 时才会被清理。您还可以创建对 通过调用 Rc::downgrade 并传递对 Rc<T> 的引用来获取 Rc<T> 实例中的值。 强引用是您如何共享 Rc<T> 实例的所有权。 弱引用不表达所有权关系，并且它们的计数不会影响 Rc<T> 实例何时被清理。 它们不会导致引用循环，因为一旦涉及的值的强引用计数为 0，任何涉及弱引用的循环都会被破坏。

当你调用 Rc::downgrade 时，你会得到一个 Weak<T> 类型的智能指针。 调用 Rc::downgrade 不是将 Rc<T> 实例中的strong_count增加 1，而是将weak_count增加1。Rc<T> 类型使用weak_count来跟踪存在多少Weak<T>引用，类似于strong_count 。 不同之处在于，要清理 Rc<T> 实例，weak_count 不需要为 0。

由于 Weak<T> 引用的值可能已被删除，因此要对 Weak<T> 指向的值执行任何操作，您必须确保该值仍然存在。 通过调用 Weak<T> 实例上的升级方法来执行此操作，该方法将返回 Option<Rc<T>>。 如果 Rc<T> 值尚未被删除，您将得到 Some 结果；如果 Rc<T> 值已被删除，您将得到 None 结果。 因为 Upgrade 返回一个 Option<Rc<T>>，Rust 将确保处理 Some 情况和 None 情况，并且不会出现无效指针。

例如，我们将创建一个树，其中的项目了解其子项目和父项目，而不是使用其项目仅了解下一个项目的列表。

#### Creating a Tree Data Structure: a Node with Child Nodes
To start, we’ll build a tree with nodes that know about their child nodes. We’ll create a struct named Node that holds its own i32 value as well as references to its children Node values:

首先，我们将构建一棵树，其中的节点了解其子节点。 我们将创建一个名为 Node 的结构体，它保存自己的 i32 值以及对其子 Node 值的引用：

Filename: src/main.rs
``````
use std::cell::RefCell;
use std::rc::Rc;

#[derive(Debug)]
struct Node {
    value: i32,
    children: RefCell<Vec<Rc<Node>>>,
}
``````
We want a Node to own its children, and we want to share that ownership with variables so we can access each Node in the tree directly. To do this, we define the Vec<T> items to be values of type Rc<Node>. We also want to modify which nodes are children of another node, so we have a RefCell<T> in children around the Vec<Rc<Node>>.

Next, we’ll use our struct definition and create one Node instance named leaf with the value 3 and no children, and another instance named branch with the value 5 and leaf as one of its children, as shown in Listing 15-27:

我们希望一个节点拥有它的子节点，并且我们希望与变量共享该所有权，以便我们可以直接访问树中的每个节点。 为此，我们将 Vec<T> 项定义为 Rc<Node> 类型的值。 我们还想修改哪些节点是另一个节点的子节点，因此我们在 Vec<Rc<Node>> 周围的子节点中有一个 RefCell<T>。

接下来，我们将使用结构体定义创建一个名为 leaf 的 Node 实例，其值为 3，没有子节点，以及另一个名为branch 的实例，其值为 5，leaf 作为其子节点之一，如清单 15-27 所示：

Filename: src/main.rs
``````
fn main() {
    let leaf = Rc::new(Node {
        value: 3,
        children: RefCell::new(vec![]),
    });

    let branch = Rc::new(Node {
        value: 5,
        children: RefCell::new(vec![Rc::clone(&leaf)]),
    });
}
``````
Listing 15-27: Creating a leaf node with no children and a branch node with leaf as one of its children

We clone the Rc<Node> in leaf and store that in branch, meaning the Node in leaf now has two owners: leaf and branch. We can get from branch to leaf through branch.children, but there’s no way to get from leaf to branch. The reason is that leaf has no reference to branch and doesn’t know they’re related. We want leaf to know that branch is its parent. We’ll do that next.

我们克隆叶子中的 Rc<Node> 并将其存储在分支中，这意味着叶子中的节点现在有两个所有者：叶子和分支。 我们可以通过branch.children从branch到leaf，但是没有办法从leaf到branch。 原因是叶子没有引用分支并且不知道它们是相关的。 我们希望叶子知道分支是它的父代。 我们接下来会这样做。

### Adding a Reference from a Child to Its Parent
To make the child node aware of its parent, we need to add a parent field to our Node struct definition. The trouble is in deciding what the type of parent should be. We know it can’t contain an Rc<T>, because that would create a reference cycle with leaf.parent pointing to branch and branch.children pointing to leaf, which would cause their strong_count values to never be 0.

Thinking about the relationships another way, a parent node should own its children: if a parent node is dropped, its child nodes should be dropped as well. However, a child should not own its parent: if we drop a child node, the parent should still exist. This is a case for weak references!

So instead of Rc<T>, we’ll make the type of parent use Weak<T>, specifically a RefCell<Weak<Node>>. Now our Node struct definition looks like this:

为了让子节点知道它的父节点，我们需要在 Node 结构定义中添加一个 Parent 字段。 麻烦在于决定父母应该是什么类型。 我们知道它不能包含 Rc<T>，因为这会创建一个引用循环，其中 leaf.parent 指向branch，branch.children 指向 leaf，这将导致它们的 Strong_count 值永远不会为 0。

从另一种角度考虑关系，父节点应该拥有它的子节点：如果父节点被删除，它的子节点也应该被删除。 但是，子节点不应该拥有其父节点：如果我们删除子节点，父节点应该仍然存在。 这是弱引用的情况！

因此，我们将使用 Weak<T> 代替 Rc<T>，特别是 RefCell<Weak<Node>>。 现在我们的 Node 结构定义如下所示：

Filename: src/main.rs
``````
use std::cell::RefCell;
use std::rc::{Rc, Weak};

#[derive(Debug)]
struct Node {
    value: i32,
    parent: RefCell<Weak<Node>>,
    children: RefCell<Vec<Rc<Node>>>,
}
``````
A node will be able to refer to its parent node but doesn’t own its parent. In Listing 15-28, we update main to use this new definition so the leaf node will have a way to refer to its parent, branch:

Filename: src/main.rs
``````
fn main() {
    let leaf = Rc::new(Node {
        value: 3,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![]),
    });

    println!("leaf parent = {:?}", leaf.parent.borrow().upgrade());

    let branch = Rc::new(Node {
        value: 5,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![Rc::clone(&leaf)]),
    });

    *leaf.parent.borrow_mut() = Rc::downgrade(&branch);

    println!("leaf parent = {:?}", leaf.parent.borrow().upgrade());
}
``````
Listing 15-28: A leaf node with a weak reference to its parent node branch

Creating the leaf node looks similar to Listing 15-27 with the exception of the parent field: leaf starts out without a parent, so we create a new, empty Weak<Node> reference instance.

At this point, when we try to get a reference to the parent of leaf by using the upgrade method, we get a None value. We see this in the output from the first println! statement:

创建叶节点看起来与清单 15-27 类似，但父字段除外：叶节点一开始没有父节点，因此我们创建一个新的空 Weak<Node> 引用实例。

此时，当我们尝试使用升级方法获取对 leaf 的父级的引用时，我们得到 None 值。 我们在第一个 println! 的输出中看到了这一点！ 陈述：

``````
leaf parent = None
``````
When we create the branch node, it will also have a new Weak<Node> reference in the parent field, because branch doesn’t have a parent node. We still have leaf as one of the children of branch. Once we have the Node instance in branch, we can modify leaf to give it a Weak<Node> reference to its parent. We use the borrow_mut method on the RefCell<Weak<Node>> in the parent field of leaf, and then we use the Rc::downgrade function to create a Weak<Node> reference to branch from the Rc<Node> in branch.

When we print the parent of leaf again, this time we’ll get a Some variant holding branch: now leaf can access its parent! When we print leaf, we also avoid the cycle that eventually ended in a stack overflow like we had in Listing 15-26; the Weak<Node> references are printed as (Weak):

当我们创建分支节点时，它的父字段中也会有一个新的 Weak<Node> 引用，因为分支没有父节点。 我们仍然有叶子作为分支的子节点之一。 一旦我们在分支中拥有 Node 实例，我们就可以修改 leaf 为其父级提供 Weak<Node> 引用。 我们在 leaf 的parent字段中的RefCell<Weak<Node>>上使用borrow_mut方法，然后使用Rc::downgrade函数从branch中的Rc<Node>创建对branch的Weak<Node>引用。

当我们再次打印 leaf 的父级时，这次我们将得到一个 Some 变体持有分支：现在 leaf 可以访问它的父级！ 当我们打印 leaf 时，我们还避免了最终以堆栈溢出结束的循环，就像清单 15-26 中那样； Weak<Node> 引用打印为 (Weak)：

``````
leaf parent = Some(Node { value: 5, parent: RefCell { value: (Weak) },
children: RefCell { value: [Node { value: 3, parent: RefCell { value: (Weak) },
children: RefCell { value: [] } }] } })
``````
The lack of infinite output indicates that this code didn’t create a reference cycle. We can also tell this by looking at the values we get from calling Rc::strong_count and Rc::weak_count.

缺少无限输出表明该代码没有创建引用循环。 我们还可以通过查看调用 Rc::strong_count 和 Rc::weak_count 获得的值来判断这一点。

### Visualizing Changes to strong_count and weak_count
Let’s look at how the strong_count and weak_count values of the Rc<Node> instances change by creating a new inner scope and moving the creation of branch into that scope. By doing so, we can see what happens when branch is created and then dropped when it goes out of scope. The modifications are shown in Listing 15-29:

让我们看看通过创建新的内部作用域并将分支的创建移动到该作用域中，Rc<Node> 实例的strong_count 和weak_count 值如何变化。 通过这样做，我们可以看到创建分支并在超出范围时删除分支时会发生什么。 修改如清单 15-29 所示：

Filename: src/main.rs
``````
fn main() {
    let leaf = Rc::new(Node {
        value: 3,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![]),
    });

    println!(
        "leaf strong = {}, weak = {}",
        Rc::strong_count(&leaf),
        Rc::weak_count(&leaf),
    );

    {
        let branch = Rc::new(Node {
            value: 5,
            parent: RefCell::new(Weak::new()),
            children: RefCell::new(vec![Rc::clone(&leaf)]),
        });

        *leaf.parent.borrow_mut() = Rc::downgrade(&branch);

        println!(
            "branch strong = {}, weak = {}",
            Rc::strong_count(&branch),
            Rc::weak_count(&branch),
        );

        println!(
            "leaf strong = {}, weak = {}",
            Rc::strong_count(&leaf),
            Rc::weak_count(&leaf),
        );
    }

    println!("leaf parent = {:?}", leaf.parent.borrow().upgrade());
    println!(
        "leaf strong = {}, weak = {}",
        Rc::strong_count(&leaf),
        Rc::weak_count(&leaf),
    );
}
``````
Listing 15-29: Creating branch in an inner scope and examining strong and weak reference counts

After leaf is created, its Rc<Node> has a strong count of 1 and a weak count of 0. In the inner scope, we create branch and associate it with leaf, at which point when we print the counts, the Rc<Node> in branch will have a strong count of 1 and a weak count of 1 (for leaf.parent pointing to branch with a Weak<Node>). When we print the counts in leaf, we’ll see it will have a strong count of 2, because branch now has a clone of the Rc<Node> of leaf stored in branch.children, but will still have a weak count of 0.

When the inner scope ends, branch goes out of scope and the strong count of the Rc<Node> decreases to 0, so its Node is dropped. The weak count of 1 from leaf.parent has no bearing on whether or not Node is dropped, so we don’t get any memory leaks!

If we try to access the parent of leaf after the end of the scope, we’ll get None again. At the end of the program, the Rc<Node> in leaf has a strong count of 1 and a weak count of 0, because the variable leaf is now the only reference to the Rc<Node> again.

All of the logic that manages the counts and value dropping is built into Rc<T> and Weak<T> and their implementations of the Drop trait. By specifying that the relationship from a child to its parent should be a Weak<T> reference in the definition of Node, you’re able to have parent nodes point to child nodes and vice versa without creating a reference cycle and memory leaks.

创建 Leaf 后，其 Rc<Node> 的强计数为 1，弱计数为 0。在内部作用域中，我们创建分支并将其与 leaf 关联，此时当我们打印计数时，Rc<Node > in Branch 的强计数为 1，弱计数为 1（对于 leaf.parent 指向带有 Weak<Node> 的分支）。 当我们打印 leaf 中的计数时，我们会看到它的强计数为 2，因为branch 现在有存储在branch.children 中的 leaf 的 Rc<Node> 的克隆，但仍然有弱计数 0 。

当内部作用域结束时，分支超出作用域，并且 Rc<Node> 的强计数减少到 0，因此它的 Node 被删除。 leaf.parent 的弱计数 1 与 Node 是否被删除无关，因此我们不会出现任何内存泄漏！

如果我们尝试在作用域结束后访问 leaf 的父级，我们将再次得到 None 。 在程序结束时，leaf 中的 Rc<Node> 的强计数为 1，弱计数为 0，因为变量 leaf 现在再次成为对 Rc<Node> 的唯一引用。

管理计数和值删除的所有逻辑都内置于 Rc<T> 和 Weak<T> 及其 Drop 特征的实现中。 通过在 Node 的定义中指定子节点与其父节点的关系应该是 Weak<T> 引用，您可以让父节点指向子节点，反之亦然，而不会创建引用循环和内存泄漏。

## 15.7 Summary
This chapter covered how to use smart pointers to make different guarantees and trade-offs from those Rust makes by default with regular references. The Box<T> type has a known size and points to data allocated on the heap. The Rc<T> type keeps track of the number of references to data on the heap so that data can have multiple owners. The RefCell<T> type with its interior mutability gives us a type that we can use when we need an immutable type but need to change an inner value of that type; it also enforces the borrowing rules at runtime instead of at compile time.

Also discussed were the Deref and Drop traits, which enable a lot of the functionality of smart pointers. We explored reference cycles that can cause memory leaks and how to prevent them using Weak<T>.

If this chapter has piqued your interest and you want to implement your own smart pointers, check out “The Rustonomicon” for more useful information.

本章介绍了如何使用智能指针来做出与 Rust 默认使用常规引用所做的不同的保证和权衡。 Box<T> 类型具有已知的大小并指向堆上分配的数据。 Rc<T> 类型跟踪堆上数据的引用次数，以便数据可以拥有多个所有者。 RefCell<T> 类型及其内部可变性为我们提供了一种类型，当我们需要不可变类型但需要更改该类型的内部值时可以使用该类型； 它还在运行时而不是在编译时强制执行借用规则。

还讨论了 Deref 和 Drop 特征，它们启用了智能指针的许多功能。 我们探讨了可能导致内存泄漏的引用循环以及如何使用 Weak<T> 来防止它们。

如果本章激起了您的兴趣并且您想实现自己的智能指针，请查看“The Rustonomicon”以获取更多有用的信息。

Next, we’ll talk about concurrency in Rust. You’ll even learn about a few new smart pointers.
