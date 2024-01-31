# 19. 高级特性

By now, you’ve learned the most commonly used parts of the Rust programming language. Before we do one more project in Chapter 20, we’ll look at a few aspects of the language you might run into every once in a while, but may not use every day. You can use this chapter as a reference for when you encounter any unknowns. The features covered here are useful in very specific situations. Although you might not reach for them often, we want to make sure you have a grasp of all the features Rust has to offer.

到目前为止，您已经学习了Rust编程语言中最常用的部分。在我们做第20章中的另一个项目之前，我们将了解语言的几个方面，您可能会偶尔遇到这种语言，但可能不会每天都使用。当你遇到任何未知的事情时，你可以把本章作为参考。这里介绍的功能在非常具体的情况下非常有用。尽管您可能不经常接触它们，但我们希望确保您掌握Rust提供的所有功能。

In this chapter, we’ll cover:

+ Unsafe Rust: how to opt out of some of Rust’s guarantees and take responsibility for manually upholding those guarantees
+ Advanced traits: associated types, default type parameters, fully qualified syntax, supertraits, and the newtype pattern in relation to traits
+ Advanced types: more about the newtype pattern, type aliases, the never type, and dynamically sized types
+ Advanced functions and closures: function pointers and returning closures
+ Macros: ways to define code that defines more code at compile time

+ 不安全的Rust：如何选择退出Rust的一些安全，并承担手动维护这些安全的责任
+ 高级特征：关联类型、默认类型参数、完全限定语法、超特征以及与特征相关的newtype模式
+ 高级类型：更多关于newtype模式、类型别名、从不类型和动态大小类型的信息
+ 高级函数和闭包：函数指针和返回闭包
+ 宏：定义在编译时定义更多代码的代码的方法

It’s a panoply of Rust features with something for everyone! Let’s dive in!
这是一整套 Rust 功能，适合每个人！ 让我们深入了解一下！

## 19.1 Unsafe Rust

All the code we’ve discussed so far has had Rust’s memory safety guarantees enforced at compile time. However, Rust has a second language hidden inside it that doesn’t enforce these memory safety guarantees: it’s called unsafe Rust and works just like regular Rust, but gives us extra superpowers.

到目前为止，我们讨论的所有代码都在编译时强制执行了Rust的内存安全保证。然而，Rust内部隐藏着第二种语言，它不强制执行这些内存安全保证：它被称为不安全的Rust，与普通Rust一样工作，但给了我们额外的超能力。

Unsafe Rust exists because, by nature, static analysis is conservative. When the compiler tries to determine whether or not code upholds the guarantees, it’s better for it to reject some valid programs than to accept some invalid programs. Although the code might be okay, if the Rust compiler doesn’t have enough information to be confident, it will reject the code. In these cases, you can use unsafe code to tell the compiler, “Trust me, I know what I’m doing.” Be warned, however, that you use unsafe Rust at your own risk: if you use unsafe code incorrectly, problems can occur due to memory unsafety, such as null pointer dereferencing.

不安全的Rust之所以存在，是因为从本质上讲，静态分析是保守的。当编译器试图确定代码是否支持保证时，与其接受一些无效程序，不如拒绝一些有效程序。尽管代码可能还可以，但如果Rust编译器没有足够的信息来确信，它将拒绝代码。在这些情况下，您可以使用不安全的代码告诉编译器，“相信我，我知道我在做什么。”但是，请注意，您使用不安全Rust的风险自负：如果您不正确地使用不安全代码，可能会由于内存不安全而出现问题，例如空指针解引用。

Another reason Rust has an unsafe alter ego is that the underlying computer hardware is inherently unsafe. If Rust didn’t let you do unsafe operations, you couldn’t do certain tasks. Rust needs to allow you to do low-level systems programming, such as directly interacting with the operating system or even writing your own operating system. Working with low-level systems programming is one of the goals of the language. Let’s explore what we can do with unsafe Rust and how to do it.

另外一个原因是Rust有一个不安全的另一个自己，即底层计算机硬件本质上是不安全的。如果Rust不让你做不安全的操作，你就无法完成某些任务。Rust需要允许您进行低级系统编程，例如直接与操作系统交互，甚至编写自己的操作系统。使用低级系统编程是该语言的目标之一。让我们探讨一下我们可以对不安全的Rust做些什么以及如何做。

### 不安全的超能力(Unsafe Superpowers)
To switch to unsafe Rust, use the unsafe keyword and then start a new block that holds the unsafe code. You can take five actions in unsafe Rust that you can’t in safe Rust, which we call unsafe superpowers. Those superpowers include the ability to:

要切换到不安全的Rust，请使用unsafe关键字，然后启动一个包含不安全代码的新块。你可以在不安全的Rust中采取五种我们称之为不安全超能力的安全Rust中无法采取的行动。这些超级大国包括以下能力：

+ Dereference a raw pointer
+ Call an unsafe function or method
+ Access or modify a mutable static variable
+ Implement an unsafe trait
+ Access fields of unions

+ 取消引用原始指针
+ 调用不安全的函数或方法
+ 访问或修改可变静态变量
+ 实施不安全特征
+ 访问union的字段

It’s important to understand that unsafe doesn’t turn off the borrow checker or disable any other of Rust’s safety checks: if you use a reference in unsafe code, it will still be checked. The unsafe keyword only gives you access to these five features that are then not checked by the compiler for memory safety. You’ll still get some degree of safety inside of an unsafe block.

重要的是要理解，unsafe不会关闭借用检查器或禁用Rust的任何其他安全检查：如果你在unsafe代码中使用引用，它仍然会被检查。unsafe关键字只允许您访问这五个功能，编译器不会对这些功能进行内存安全检查。在一个不安全的街区内，你仍然会得到一定程度的安全。

In addition, unsafe does not mean the code inside the block is necessarily dangerous or that it will definitely have memory safety problems: the intent is that as the programmer, you’ll ensure the code inside an unsafe block will access memory in a valid way.

此外，不安全并不意味着块内的代码一定是危险的，也不意味着它肯定会有内存安全问题：其目的是作为程序员，你将确保不安全块内的编码以有效的方式访问内存。

People are fallible, and mistakes will happen, but by requiring these five unsafe operations to be inside blocks annotated with unsafe you’ll know that any errors related to memory safety must be within an unsafe block. Keep unsafe blocks small; you’ll be thankful later when you investigate memory bugs.

人们容易犯错，错误也会发生，但通过要求这五个不安全的操作位于注释有unsafe的块内，您就会知道任何与内存安全相关的错误都必须位于不安全的块内。将不安全的块保持在较小的位置；稍后当你调查内存错误时，你会心存感激。

To isolate unsafe code as much as possible, it’s best to enclose unsafe code within a safe abstraction and provide a safe API, which we’ll discuss later in the chapter when we examine unsafe functions and methods. Parts of the standard library are implemented as safe abstractions over unsafe code that has been audited. Wrapping unsafe code in a safe abstraction prevents uses of unsafe from leaking out into all the places that you or your users might want to use the functionality implemented with unsafe code, because using a safe abstraction is safe.

为了尽可能地隔离不安全代码，最好将不安全代码包含在安全抽象中，并提供安全的API，我们将在本章稍后检查不安全的函数和方法时对此进行讨论。标准库的某些部分被实现为对已审核的不安全代码的安全抽象。将不安全的代码封装在安全抽象中可以防止不安全的使用泄漏到您或您的用户可能想要使用不安全代码实现的功能的所有地方，因为使用安全抽象是安全的。

Let’s look at each of the five unsafe superpowers in turn. We’ll also look at some abstractions that provide a safe interface to unsafe code.

让我们依次看看五个不安全的超级大国中的每一个。我们还将研究一些为不安全代码提供安全接口的抽象。

### Dereferencing a Raw Pointer
In Chapter 4, in the “Dangling References” section, we mentioned that the compiler ensures references are always valid. Unsafe Rust has two new types called raw pointers that are similar to references. As with references, raw pointers can be immutable or mutable and are written as *const T and *mut T, respectively. The asterisk isn’t the dereference operator; it’s part of the type name. In the context of raw pointers, immutable means that the pointer can’t be directly assigned to after being dereferenced.

在第4章的“挂起引用”一节中，我们提到编译器确保引用始终有效。Unsafe Rust有两种称为原始指针的新类型，它们类似于引用。与引用一样，原始指针可以是不可变的，也可以是可变的，分别写成*const T和*mut T。星号不是取消引用运算符；它是类型名称的一部分。在原始指针的上下文中，不可变意味着指针在被取消引用后不能直接分配给。

Different from references and smart pointers, raw pointers:

+ Are allowed to ignore the borrowing rules by having both immutable and mutable pointers or multiple mutable pointers to the same location
+ Aren’t guaranteed to point to valid memory
+ Are allowed to be null
+ Don’t implement any automatic cleanup

+ 允许通过将不可变和可变指针或多个可变指针指向同一位置来忽略借用规则
+ 不能保证指向有效内存
+ 允许为null
+ 不执行任何自动清理

By opting out of having Rust enforce these guarantees, you can give up guaranteed safety in exchange for greater performance or the ability to interface with another language or hardware where Rust’s guarantees don’t apply.

通过选择不让Rust强制执行这些保证，您可以放弃有保证的安全性，以换取更高的性能或与其他语言或硬件接口的能力，而Rust的保证不适用。

Listing 19-1 shows how to create an immutable and a mutable raw pointer from references.
``````
    let mut num = 5;

    let r1 = &num as *const i32;
    let r2 = &mut num as *mut i32;
``````
Listing 19-1: Creating raw pointers from references

Notice that we don’t include the unsafe keyword in this code. We can create raw pointers in safe code; we just can’t dereference raw pointers outside an unsafe block, as you’ll see in a bit.

请注意，我们在这段代码中没有包含unsafe关键字。我们可以在安全代码中创建原始指针；我们只是不能在不安全的块之外取消引用原始指针，正如您稍后将看到的那样。

We’ve created raw pointers by using as to cast an immutable and a mutable reference into their corresponding raw pointer types. Because we created them directly from references guaranteed to be valid, we know these particular raw pointers are valid, but we can’t make that assumption about just any raw pointer.

我们创建了原始指针，方法是使用as将一个不可变引用和一个可变引用强制转换为它们对应的原始指针类型。因为我们直接从保证有效的引用中创建它们，所以我们知道这些特定的原始指针是有效的，但我们不能仅对任何原始指针做出这种假设。

To demonstrate this, next we’ll create a raw pointer whose validity we can’t be so certain of. Listing 19-2 shows how to create a raw pointer to an arbitrary location in memory. Trying to use arbitrary memory is undefined: there might be data at that address or there might not, the compiler might optimize the code so there is no memory access, or the program might error with a segmentation fault. Usually, there is no good reason to write code like this, but it is possible.

为了证明这一点，接下来我们将创建一个原始指针，其有效性我们无法确定。清单19-2展示了如何创建指向内存中任意位置的原始指针。尝试使用任意内存是未定义的：该地址可能有数据，也可能没有数据，编译器可能会优化代码，使其无法访问内存，或者程序可能会出现分段错误。通常，没有充分的理由编写这样的代码，但这是可能的。

``````
    let address = 0x012345usize;
    let r = address as *const i32;
``````
Listing 19-2: Creating a raw pointer to an arbitrary memory address

Recall that we can create raw pointers in safe code, but we can’t dereference raw pointers and read the data being pointed to. In Listing 19-3, we use the dereference operator * on a raw pointer that requires an unsafe block.

回想一下，我们可以在安全代码中创建原始指针，但不能取消引用原始指针并读取所指向的数据。在清单19-3中，我们对需要不安全块的原始指针使用取消引用运算符*。

``````
    let mut num = 5;

    let r1 = &num as *const i32;
    let r2 = &mut num as *mut i32;

    unsafe {
        println!("r1 is: {}", *r1);
        println!("r2 is: {}", *r2);
    }
``````
Listing 19-3: Dereferencing raw pointers within an unsafe block

Creating a pointer does no harm; it’s only when we try to access the value that it points at that we might end up dealing with an invalid value.

创建指针没有害处；只有当我们试图访问它所指向的值时，我们才可能最终处理一个无效的值。

Note also that in Listing 19-1 and 19-3, we created *const i32 and *mut i32 raw pointers that both pointed to the same memory location, where num is stored. If we instead tried to create an immutable and a mutable reference to num, the code would not have compiled because Rust’s ownership rules don’t allow a mutable reference at the same time as any immutable references. With raw pointers, we can create a mutable pointer and an immutable pointer to the same location and change data through the mutable pointer, potentially creating a data race. Be careful!

还要注意，在清单19-1和19-3中，我们创建了*const i32和*mut i32原始指针，它们都指向存储num的同一内存位置。如果我们尝试创建一个对num的不可变和可变引用，代码就不会编译，因为Rust的所有权规则不允许在任何不可变引用的同时使用可变引用。使用原始指针，我们可以创建一个指向同一位置的可变指针和一个不可变指针，并通过可变指针更改数据，这可能会造成数据竞争。小心！

With all of these dangers, why would you ever use raw pointers? One major use case is when interfacing with C code, as you’ll see in the next section, “Calling an Unsafe Function or Method.” Another case is when building up safe abstractions that the borrow checker doesn’t understand. We’ll introduce unsafe functions and then look at an example of a safe abstraction that uses unsafe code.

有了所有这些危险，你为什么要使用原始指针？一个主要的用例是在与C代码接口时，正如您将在下一节“调用不安全的函数或方法”中看到的那样。另一个用例是在构建借用检查器不理解的安全抽象时。我们将介绍不安全的函数，然后看一个使用不安全代码的安全抽象的例子。

### 调用一个不安全的函数和方法(Calling an Unsafe Function or Method)
The second type of operation you can perform in an unsafe block is calling unsafe functions. Unsafe functions and methods look exactly like regular functions and methods, but they have an extra unsafe before the rest of the definition. The unsafe keyword in this context indicates the function has requirements we need to uphold when we call this function, because Rust can’t guarantee we’ve met these requirements. By calling an unsafe function within an unsafe block, we’re saying that we’ve read this function’s documentation and take responsibility for upholding the function’s contracts.

在不安全块中可以执行的第二种操作是调用不安全函数。不安全的函数和方法看起来与常规函数和方法完全一样，但在定义的其余部分之前，它们有一个额外的不安全。此上下文中的unsafe关键字表示当我们调用该函数时，该函数有需要维护的要求，因为Rust不能保证我们满足了这些要求。通过调用不安全块中的不安全函数，我们表示我们已经阅读了该函数的文档，并负责维护该函数的合同。

Here is an unsafe function named dangerous that doesn’t do anything in its body:
``````
    unsafe fn dangerous() {}

    unsafe {
        dangerous();
    }
``````
We must call the dangerous function within a separate unsafe block. If we try to call dangerous without the unsafe block, we’ll get an error:

我们必须在一个单独的不安全块中调用危险函数。如果我们试图在没有unsafe块的情况下调用dangerous，我们将得到一个错误：

``````
$ cargo run
   Compiling unsafe-example v0.1.0 (file:///projects/unsafe-example)
error[E0133]: call to unsafe function is unsafe and requires unsafe function or block
 --> src/main.rs:4:5
  |
4 |     dangerous();
  |     ^^^^^^^^^^^ call to unsafe function
  |
  = note: consult the function's documentation for information on how to avoid undefined behavior

For more information about this error, try `rustc --explain E0133`.
error: could not compile `unsafe-example` due to previous error
``````
With the unsafe block, we’re asserting to Rust that we’ve read the function’s documentation, we understand how to use it properly, and we’ve verified that we’re fulfilling the contract of the function.

对于不安全的块，我们向Rust断言，我们已经阅读了该函数的文档，了解如何正确使用它，并且我们已经验证了我们正在履行该函数的合同。

Bodies of unsafe functions are effectively unsafe blocks, so to perform other unsafe operations within an unsafe function, we don’t need to add another unsafe block.

不安全函数体实际上是不安全块，因此要在不安全函数中执行其他不安全操作，我们不需要添加另一个不安全块。

### Creating a Safe Abstraction over Unsafe Code

Just because a function contains unsafe code doesn’t mean we need to mark the entire function as unsafe. In fact, wrapping unsafe code in a safe function is a common abstraction. As an example, let’s study the split_at_mut function from the standard library, which requires some unsafe code. We’ll explore how we might implement it. This safe method is defined on mutable slices: it takes one slice and makes it two by splitting the slice at the index given as an argument. Listing 19-4 shows how to use split_at_mut.

仅仅因为一个函数包含不安全的代码并不意味着我们需要将整个函数标记为不安全。事实上，在安全函数中包装不安全的代码是一种常见的抽象。例如，让我们研究标准库中的split_at_mut函数，它需要一些不安全的代码。我们将探讨如何实现它。这个安全的方法是在可变切片上定义的：它取一个切片，并通过在作为参数给定的索引处拆分切片来将其一分为二。清单19-4展示了如何使用split_at_mut。

``````
    let mut v = vec![1, 2, 3, 4, 5, 6];

    let r = &mut v[..];

    let (a, b) = r.split_at_mut(3);

    assert_eq!(a, &mut [1, 2, 3]);
    assert_eq!(b, &mut [4, 5, 6]);
``````
Listing 19-4: Using the safe split_at_mut function

We can’t implement this function using only safe Rust. An attempt might look something like Listing 19-5, which won’t compile. For simplicity, we’ll implement split_at_mut as a function rather than a method and only for slices of i32 values rather than for a generic type T.

我们不能仅使用安全的Rust来实现此功能。一个尝试可能看起来像清单19-5，它不会编译。为了简单起见，我们将split_at_mut实现为一个函数，而不是一个方法，并且仅用于i32值的切片，而不是通用类型T。

``````
This code does not compile!
fn split_at_mut(values: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = values.len();

    assert!(mid <= len);

    (&mut values[..mid], &mut values[mid..])
}
``````
Listing 19-5: An attempted implementation of split_at_mut using only safe Rust

This function first gets the total length of the slice. Then it asserts that the index given as a parameter is within the slice by checking whether it’s less than or equal to the length. The assertion means that if we pass an index that is greater than the length to split the slice at, the function will panic before it attempts to use that index.

此函数首先获取切片的总长度。然后，它通过检查索引是否小于或等于长度来断言作为参数给定的索引在切片内。断言意味着，如果我们传递的索引大于分割切片的长度，则函数在尝试使用该索引之前会死机。

Then we return two mutable slices in a tuple: one from the start of the original slice to the mid index and another from mid to the end of the slice.

然后，我们在元组中返回两个可变切片：一个从原始切片的开始到中间索引，另一个从切片的中间到结尾。

When we try to compile the code in Listing 19-5, we’ll get an error.
``````
$ cargo run
   Compiling unsafe-example v0.1.0 (file:///projects/unsafe-example)
error[E0499]: cannot borrow `*values` as mutable more than once at a time
 --> src/main.rs:6:31
  |
1 | fn split_at_mut(values: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
  |                         - let's call the lifetime of this reference `'1`
...
6 |     (&mut values[..mid], &mut values[mid..])
  |     --------------------------^^^^^^--------
  |     |     |                   |
  |     |     |                   second mutable borrow occurs here
  |     |     first mutable borrow occurs here
  |     returning this value requires that `*values` is borrowed for `'1`

For more information about this error, try `rustc --explain E0499`.
error: could not compile `unsafe-example` due to previous error
``````
Rust’s borrow checker can’t understand that we’re borrowing different parts of the slice; it only knows that we’re borrowing from the same slice twice. Borrowing different parts of a slice is fundamentally okay because the two slices aren’t overlapping, but Rust isn’t smart enough to know this. When we know code is okay, but Rust doesn’t, it’s time to reach for unsafe code.

Rust 的借用检查器无法理解我们正在借用切片的不同部分； 它只知道我们从同一个切片中借用了两次。 借用切片的不同部分基本上是可以的，因为两个切片不重叠，但 Rust 不够聪明，无法知道这一点。 当我们知道代码没问题，但 Rust 不行时，就该去寻找不安全的代码了。

Listing 19-6 shows how to use an unsafe block, a raw pointer, and some calls to unsafe functions to make the implementation of split_at_mut work.

清单 19-6 显示了如何使用不安全块、原始指针以及对不安全函数的一些调用来使 split_at_mut 的实现正常工作。

``````
use std::slice;

fn split_at_mut(values: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = values.len();
    let ptr = values.as_mut_ptr();

    assert!(mid <= len);

    unsafe {
        (
            slice::from_raw_parts_mut(ptr, mid),
            slice::from_raw_parts_mut(ptr.add(mid), len - mid),
        )
    }
}
``````
Listing 19-6: Using unsafe code in the implementation of the split_at_mut function

Recall from “The Slice Type” section in Chapter 4 that slices are a pointer to some data and the length of the slice. We use the len method to get the length of a slice and the as_mut_ptr method to access the raw pointer of a slice. In this case, because we have a mutable slice to i32 values, as_mut_ptr returns a raw pointer with the type *mut i32, which we’ve stored in the variable ptr.

We keep the assertion that the mid index is within the slice. Then we get to the unsafe code: the slice::from_raw_parts_mut function takes a raw pointer and a length, and it creates a slice. We use this function to create a slice that starts from ptr and is mid items long. Then we call the add method on ptr with mid as an argument to get a raw pointer that starts at mid, and we create a slice using that pointer and the remaining number of items after mid as the length.

The function slice::from_raw_parts_mut is unsafe because it takes a raw pointer and must trust that this pointer is valid. The add method on raw pointers is also unsafe, because it must trust that the offset location is also a valid pointer. Therefore, we had to put an unsafe block around our calls to slice::from_raw_parts_mut and add so we could call them. By looking at the code and by adding the assertion that mid must be less than or equal to len, we can tell that all the raw pointers used within the unsafe block will be valid pointers to data within the slice. This is an acceptable and appropriate use of unsafe.

Note that we don’t need to mark the resulting split_at_mut function as unsafe, and we can call this function from safe Rust. We’ve created a safe abstraction to the unsafe code with an implementation of the function that uses unsafe code in a safe way, because it creates only valid pointers from the data this function has access to.

In contrast, the use of slice::from_raw_parts_mut in Listing 19-7 would likely crash when the slice is used. This code takes an arbitrary memory location and creates a slice 10,000 items long.

回想一下第 4 章中的“切片类型”部分，切片是指向某些数据和切片长度的指针。 我们使用 len 方法来获取切片的长度，并使用 as_mut_ptr 方法来访问切片的原始指针。 在本例中，因为我们有一个 i32 值的可变切片，所以 as_mut_ptr 返回类型为 *mut i32 的原始指针，我们将其存储在变量 ptr 中。

我们保留中间索引位于切片内的断言。 然后我们看到不安全的代码：slice::from_raw_parts_mut 函数接受一个原始指针和一个长度，并创建一个切片。 我们使用此函数创建一个从 ptr 开始、长度为 mid items 的切片。 然后，我们以 mid 作为参数调用 ptr 上的 add 方法，以获取从 mid 开始的原始指针，并使用该指针和 mid 之后的剩余项目数作为长度创建一个切片。

函数 slice::from_raw_parts_mut 是不安全的，因为它采用原始指针并且必须相信该指针是有效的。 原始指针上的 add 方法也是不安全的，因为它必须相信偏移位置也是一个有效的指针。 因此，我们必须在对 slice::from_raw_parts_mut 和 add 的调用周围放置一个不安全的块，以便我们可以调用它们。 通过查看代码并添加 mid 必须小于或等于 len 的断言，我们可以知道不安全块中使用的所有原始指针都将是指向切片中数据的有效指针。 这是 unsafe 的可接受且适当的使用。

请注意，我们不需要将生成的 split_at_mut 函数标记为不安全，并且我们可以从安全的 Rust 中调用该函数。 我们通过以安全方式使用不安全代码的函数实现创建了对不安全代码的安全抽象，因为它仅根据该函数有权访问的数据创建有效指针。

相反，清单 19-7 中使用 slice::from_raw_parts_mut 可能会在使用切片时崩溃。 此代码采用任意内存位置并创建一个长度为 10,000 个项目的切片。

``````
    use std::slice;

    let address = 0x01234usize;
    let r = address as *mut i32;

    let values: &[i32] = unsafe { slice::from_raw_parts_mut(r, 10000) };
``````
Listing 19-7: Creating a slice from an arbitrary memory location

We don’t own the memory at this arbitrary location, and there is no guarantee that the slice this code creates contains valid i32 values. Attempting to use values as though it’s a valid slice results in undefined behavior.

我们不拥有这个任意位置的内存，并且不能保证此代码创建的切片包含有效的 i32 值。 尝试将值视为有效切片会导致未定义的行为。

### 使用 extern 函数调用外部代码(Using extern Functions to Call External Code)
Sometimes, your Rust code might need to interact with code written in another language. For this, Rust has the keyword extern that facilitates the creation and use of a Foreign Function Interface (FFI). An FFI is a way for a programming language to define functions and enable a different (foreign) programming language to call those functions.

有时，您的 Rust 代码可能需要与用另一种语言编写的代码进行交互。 为此，Rust 有关键字 extern，它有助于外部函数接口 (FFI) 的创建和使用。 FFI 是一种编程语言定义函数并使不同（外国）编程语言能够调用这些函数的方法。

Listing 19-8 demonstrates how to set up an integration with the abs function from the C standard library. Functions declared within extern blocks are always unsafe to call from Rust code. The reason is that other languages don’t enforce Rust’s rules and guarantees, and Rust can’t check them, so responsibility falls on the programmer to ensure safety.

清单19-8演示了如何设置与C标准库中的abs函数的集成。 从 Rust 代码调用 extern 块中声明的函数始终是不安全的。 原因是其他语言不执行 Rust 的规则和保证，并且 Rust 无法检查它们，因此确保安全的责任落在程序员身上。


Filename: src/main.rs
``````
extern "C" {
    fn abs(input: i32) -> i32;
}

fn main() {
    unsafe {
        println!("Absolute value of -3 according to C: {}", abs(-3));
    }
}
``````
Listing 19-8: Declaring and calling an extern function defined in another language

Within the extern "C" block, we list the names and signatures of external functions from another language we want to call. The "C" part defines which application binary interface (ABI) the external function uses: the ABI defines how to call the function at the assembly level. The "C" ABI is the most common and follows the C programming language’s ABI.

在 extern“C”块中，我们列出了我们想要调用的另一种语言的外部函数的名称和签名。 “C”部分定义外部函数使用哪个应用程序二进制接口（ABI）：ABI 定义如何在汇编级别调用该函数。 “C”ABI 是最常见的，并且遵循 C 编程语言的 ABI。

``````
Calling Rust Functions from Other Languages
We can also use extern to create an interface that allows other languages to call Rust functions. Instead of creating a whole extern block, we add the extern keyword and specify the ABI to use just before the fn keyword for the relevant function. We also need to add a #[no_mangle] annotation to tell the Rust compiler not to mangle the name of this function. Mangling is when a compiler changes the name we’ve given a function to a different name that contains more information for other parts of the compilation process to consume but is less human readable. Every programming language compiler mangles names slightly differently, so for a Rust function to be nameable by other languages, we must disable the Rust compiler’s name mangling.

In the following example, we make the call_from_c function accessible from C code, after it’s compiled to a shared library and linked from C:

#[no_mangle]
pub extern "C" fn call_from_c() {
    println!("Just called a Rust function from C!");
}
This usage of extern does not require unsafe.
``````
### 访问或修改可变静态变量(Accessing or Modifying a Mutable Static Variable)
In this book, we’ve not yet talked about global variables, which Rust does support but can be problematic with Rust’s ownership rules. If two threads are accessing the same mutable global variable, it can cause a data race.

In Rust, global variables are called static variables. Listing 19-9 shows an example declaration and use of a static variable with a string slice as a value.

在本书中，我们还没有讨论全局变量，Rust 确实支持全局变量，但可能会对 Rust 的所有权规则产生问题。 如果两个线程访问同一个可变全局变量，则可能会导致数据争用。

在 Rust 中，全局变量称为静态变量。 清单 19-9 显示了以字符串切片作为值的静态变量的声明和使用示例。

Filename: src/main.rs
``````
static HELLO_WORLD: &str = "Hello, world!";

fn main() {
    println!("name is: {}", HELLO_WORLD);
}
``````
Listing 19-9: Defining and using an immutable static variable

Static variables are similar to constants, which we discussed in the “Differences Between Variables and Constants” section in Chapter 3. The names of static variables are in SCREAMING_SNAKE_CASE by convention. Static variables can only store references with the 'static lifetime, which means the Rust compiler can figure out the lifetime and we aren’t required to annotate it explicitly. Accessing an immutable static variable is safe.

A subtle difference between constants and immutable static variables is that values in a static variable have a fixed address in memory. Using the value will always access the same data. Constants, on the other hand, are allowed to duplicate their data whenever they’re used. Another difference is that static variables can be mutable. Accessing and modifying mutable static variables is unsafe. Listing 19-10 shows how to declare, access, and modify a mutable static variable named COUNTER.

静态变量与常量类似，我们在第 3 章“变量和常量之间的差异”一节中讨论过。按照惯例，静态变量的名称采用 SCREAMING_SNAKE_CASE 格式。 静态变量只能存储具有 'static 生命周期的引用，这意味着 Rust 编译器可以计算出生命周期，我们不需要显式注释它。 访问不可变的静态变量是安全的。

常量和不可变静态变量之间的一个细微区别是静态变量中的值在内存中具有固定地址。 使用该值将始终访问相同的数据。 另一方面，常量在使用时可以复制其数据。 另一个区别是静态变量可以是可变的。 访问和修改可变静态变量是不安全的。 清单 19-10 显示了如何声明、访问和修改名为 COUNTER 的可变静态变量。

Filename: src/main.rs
``````
static mut COUNTER: u32 = 0;

fn add_to_count(inc: u32) {
    unsafe {
        COUNTER += inc;
    }
}

fn main() {
    add_to_count(3);

    unsafe {
        println!("COUNTER: {}", COUNTER);
    }
}
``````
Listing 19-10: Reading from or writing to a mutable static variable is unsafe

As with regular variables, we specify mutability using the mut keyword. Any code that reads or writes from COUNTER must be within an unsafe block. This code compiles and prints COUNTER: 3 as we would expect because it’s single threaded. Having multiple threads access COUNTER would likely result in data races.

With mutable data that is globally accessible, it’s difficult to ensure there are no data races, which is why Rust considers mutable static variables to be unsafe. Where possible, it’s preferable to use the concurrency techniques and thread-safe smart pointers we discussed in Chapter 16 so the compiler checks that data accessed from different threads is done safely.

与常规变量一样，我们使用 mut 关键字指定可变性。 从 COUNTER 读取或写入的任何代码都必须位于不安全块内。 此代码按照我们的预期编译并打印 COUNTER: 3，因为它是单线程的。 让多个线程访问 COUNTER 可能会导致数据竞争。

对于全局可访问的可变数据，很难确保不存在数据争用，这就是 Rust 认为可变静态变量不安全的原因。 在可能的情况下，最好使用我们在第 16 章中讨论的并发技术和线程安全智能指针，以便编译器检查从不同线程访问的数据是否安全。

### Implementing an Unsafe Trait

We can use unsafe to implement an unsafe trait. A trait is unsafe when at least one of its methods has some invariant that the compiler can’t verify. We declare that a trait is unsafe by adding the unsafe keyword before trait and marking the implementation of the trait as unsafe too, as shown in Listing 19-11.

我们可以使用 unsafe 来实现不安全的特征。 当特征的至少一个方法具有编译器无法验证的某些不变量时，该特征是不安全的。 我们通过在 Trait 之前添加 unsafe 关键字并将该 Trait 的实现也标记为不安全来声明 Trait 不安全，如清单 19-11 所示。

``````
unsafe trait Foo {
    // methods go here
}

unsafe impl Foo for i32 {
    // method implementations go here
}

fn main() {}
``````
Listing 19-11: Defining and implementing an unsafe trait

By using unsafe impl, we’re promising that we’ll uphold the invariants that the compiler can’t verify.

As an example, recall the Sync and Send marker traits we discussed in the “Extensible Concurrency with the Sync and Send Traits” section in Chapter 16: the compiler implements these traits automatically if our types are composed entirely of Send and Sync types. If we implement a type that contains a type that is not Send or Sync, such as raw pointers, and we want to mark that type as Send or Sync, we must use unsafe. Rust can’t verify that our type upholds the guarantees that it can be safely sent across threads or accessed from multiple threads; therefore, we need to do those checks manually and indicate as such with unsafe.

通过使用 unsafe impl，我们承诺将维护编译器无法验证的不变量。

作为一个例子，回想一下我们在第 16 章的“具有同步和发送特征的可扩展并发”部分中讨论的同步和发送标记特征：如果我们的类型完全由发送和同步类型组成，编译器会自动实现这些特征。 如果我们实现的类型包含非 Send 或 Sync 的类型（例如原始指针），并且我们想要将该类型标记为 Send 或 Sync，则必须使用 unsafe。 Rust 无法验证我们的类型是否能够保证它可以安全地跨线程发送或从多个线程访问； 因此，我们需要手动进行这些检查并用不安全来指示。

### Accessing Fields of a Union
The final action that works only with unsafe is accessing fields of a union. A union is similar to a struct, but only one declared field is used in a particular instance at one time. Unions are primarily used to interface with unions in C code. Accessing union fields is unsafe because Rust can’t guarantee the type of the data currently being stored in the union instance. You can learn more about unions in the Rust Reference.

仅适用于不安全的最后一个操作是访问联合的字段。 联合类似于结构，但在特定实例中一次仅使用一个声明的字段。 联合主要用于与 C 代码中的联合进行交互。 访问联合字段是不安全的，因为 Rust 无法保证当前存储在联合实例中的数据的类型。 您可以在 Rust 参考中了解有关联合的更多信息。

### When to Use Unsafe Code
Using unsafe to take one of the five actions (superpowers) just discussed isn’t wrong or even frowned upon. But it is trickier to get unsafe code correct because the compiler can’t help uphold memory safety. When you have a reason to use unsafe code, you can do so, and having the explicit unsafe annotation makes it easier to track down the source of problems when they occur.

使用 unsafe 来采取刚才讨论的五种行动（超能力）之一并没有错，甚至不会令人皱眉。 但要使不安全的代码正确是比较棘手的，因为编译器无法帮助维护内存安全。 当您有理由使用不安全代码时，您可以这样做，并且使用显式不安全注释可以更轻松地在问题发生时追踪问题的根源。

## 19.2 Advanced Traits

We first covered traits in the “Traits: Defining Shared Behavior” section of Chapter 10, but we didn’t discuss the more advanced details. Now that you know more about Rust, we can get into the nitty-gritty.

我们首先在第 10 章的“特征：定义共享行为”部分介绍了特征，但我们没有讨论更高级的细节。 现在您对 Rust 了解更多了，我们可以深入了解细节了。

Specifying Placeholder Types in Trait Definitions with Associated Types
Associated types connect a type placeholder with a trait such that the trait method definitions can use these placeholder types in their signatures. The implementor of a trait will specify the concrete type to be used instead of the placeholder type for the particular implementation. That way, we can define a trait that uses some types without needing to know exactly what those types are until the trait is implemented.

We’ve described most of the advanced features in this chapter as being rarely needed. Associated types are somewhere in the middle: they’re used more rarely than features explained in the rest of the book but more commonly than many of the other features discussed in this chapter.

One example of a trait with an associated type is the Iterator trait that the standard library provides. The associated type is named Item and stands in for the type of the values the type implementing the Iterator trait is iterating over. The definition of the Iterator trait is as shown in Listing 19-12.

在具有关联类型的特征定义中指定占位符类型
关联类型将类型占位符与特征连接起来，以便特征方法定义可以在其签名中使用这些占位符类型。 特征的实现者将指定要使用的具体类型，而不是特定实现的占位符类型。 这样，我们就可以定义一个使用某些类型的特征，而无需在该特征实现之前确切地知道这些类型是什么。

我们在本章中描述了大多数很少需要的高级功能。 关联类型处于中间位置：它们比本书其余部分中解释的功能更少使用，但比本章讨论的许多其他功能更常见。

具有关联类型的特征的一个示例是标准库提供的迭代器特征。 关联的类型名为 Item，代表实现 Iterator 特征的类型正在迭代的值的类型。 Iterator 特征的定义如清单 19-12 所示。

``````
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;
}
``````
Listing 19-12: The definition of the Iterator trait that has an associated type Item

The type Item is a placeholder, and the next method’s definition shows that it will return values of type Option<Self::Item>. Implementors of the Iterator trait will specify the concrete type for Item, and the next method will return an Option containing a value of that concrete type.

Associated types might seem like a similar concept to generics, in that the latter allow us to define a function without specifying what types it can handle. To examine the difference between the two concepts, we’ll look at an implementation of the Iterator trait on a type named Counter that specifies the Item type is u32:

Item 类型是一个占位符，next 方法的定义表明它将返回 Option<Self::Item> 类型的值。 Iterator 特征的实现者将指定 Item 的具体类型，并且 next 方法将返回包含该具体类型的值的 Option。

关联类型可能看起来与泛型类似，因为后者允许我们定义一个函数而不指定它可以处理什么类型。 为了检查这两个概念之间的差异，我们将查看 Iterator 特征在名为 Counter 的类型上的实现，该类型指定 Item 类型为 u32：

Filename: src/lib.rs
``````
impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        // --snip--
``````
This syntax seems comparable to that of generics. So why not just define the Iterator trait with generics, as shown in Listing 19-13?
``````
pub trait Iterator<T> {
    fn next(&mut self) -> Option<T>;
}
``````
Listing 19-13: A hypothetical definition of the Iterator trait using generics

The difference is that when using generics, as in Listing 19-13, we must annotate the types in each implementation; because we can also implement Iterator<String> for Counter or any other type, we could have multiple implementations of Iterator for Counter. In other words, when a trait has a generic parameter, it can be implemented for a type multiple times, changing the concrete types of the generic type parameters each time. When we use the next method on Counter, we would have to provide type annotations to indicate which implementation of Iterator we want to use.

With associated types, we don’t need to annotate types because we can’t implement a trait on a type multiple times. In Listing 19-12 with the definition that uses associated types, we can only choose what the type of Item will be once, because there can only be one impl Iterator for Counter. We don’t have to specify that we want an iterator of u32 values everywhere that we call next on Counter.

Associated types also become part of the trait’s contract: implementors of the trait must provide a type to stand in for the associated type placeholder. Associated types often have a name that describes how the type will be used, and documenting the associated type in the API documentation is good practice.

不同之处在于，当使用泛型时，如清单 19-13 所示，我们必须在每个实现中注释类型； 因为我们还可以为 Counter 或任何其他类型实现 Iterator<String>，所以我们可以为 Counter 提供多个 Iterator 实现。 换句话说，当一个特征具有泛型参数时，它可以针对一个类型多次实现，每次都更改泛型类型参数的具体类型。 当我们在 Counter 上使用 next 方法时，我们必须提供类型注释来指示我们要使用的 Iterator 的实现。

对于关联类型，我们不需要注释类型，因为我们不能多次在类型上实现特征。 在清单 19-12 中，使用关联类型的定义，我们只能选择一次 Item 的类型，因为 Counter 只能有一个 impl Iterator。 我们不必指定我们想要一个 u32 值的迭代器，我们在 Counter 上调用 next 。

关联类型也成为特征契约的一部分：特征的实现者必须提供一个类型来代表关联类型占位符。 关联类型通常有一个名称来描述如何使用该类型，并且在 API 文档中记录关联类型是一种很好的做法。

### Default Generic Type Parameters and Operator Overloading
When we use generic type parameters, we can specify a default concrete type for the generic type. This eliminates the need for implementors of the trait to specify a concrete type if the default type works. You specify a default type when declaring a generic type with the <PlaceholderType=ConcreteType> syntax.

A great example of a situation where this technique is useful is with operator overloading, in which you customize the behavior of an operator (such as +) in particular situations.

Rust doesn’t allow you to create your own operators or overload arbitrary operators. But you can overload the operations and corresponding traits listed in std::ops by implementing the traits associated with the operator. For example, in Listing 19-14 we overload the + operator to add two Point instances together. We do this by implementing the Add trait on a Point struct:

当我们使用泛型类型参数时，我们可以为泛型类型指定一个默认的具体类型。 如果默认类型有效，则特征的实现者无需指定具体类型。 您可以在使用 <PlaceholderType=ConcreteType> 语法声明泛型类型时指定默认类型。

此技术有用的一个很好的例子是运算符重载，其中您可以在特定情况下自定义运算符（例如 +）的行为。

Rust 不允许您创建自己的运算符或重载任意运算符。 但是您可以通过实现与运算符关联的特征来重载 std::ops 中列出的操作和相应特征。 例如，在清单 19-14 中，我们重载了 + 运算符以将两个 Point 实例添加在一起。 我们通过在 Point 结构上实现 Add 特征来做到这一点：

Filename: src/main.rs
``````
use std::ops::Add;

#[derive(Debug, Copy, Clone, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}

impl Add for Point {
    type Output = Point;

    fn add(self, other: Point) -> Point {
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}

fn main() {
    assert_eq!(
        Point { x: 1, y: 0 } + Point { x: 2, y: 3 },
        Point { x: 3, y: 3 }
    );
}
``````
Listing 19-14: Implementing the Add trait to overload the + operator for Point instances

The add method adds the x values of two Point instances and the y values of two Point instances to create a new Point. The Add trait has an associated type named Output that determines the type returned from the add method.

add 方法将两个 Point 实例的 x 值和两个 Point 实例的 y 值相加以创建一个新 Point。 Add 特征有一个名为 Output 的关联类型，它确定从 add 方法返回的类型。

The default generic type in this code is within the Add trait. Here is its definition:
``````
trait Add<Rhs=Self> {
    type Output;

    fn add(self, rhs: Rhs) -> Self::Output;
}
``````
This code should look generally familiar: a trait with one method and an associated type. The new part is Rhs=Self: this syntax is called default type parameters. The Rhs generic type parameter (short for “right hand side”) defines the type of the rhs parameter in the add method. If we don’t specify a concrete type for Rhs when we implement the Add trait, the type of Rhs will default to Self, which will be the type we’re implementing Add on.


When we implemented Add for Point, we used the default for Rhs because we wanted to add two Point instances. Let’s look at an example of implementing the Add trait where we want to customize the Rhs type rather than using the default.

We have two structs, Millimeters and Meters, holding values in different units. This thin wrapping of an existing type in another struct is known as the newtype pattern, which we describe in more detail in the “Using the Newtype Pattern to Implement External Traits on External Types” section. We want to add values in millimeters to values in meters and have the implementation of Add do the conversion correctly. We can implement Add for Millimeters with Meters as the Rhs, as shown in Listing 19-15.

这段代码看起来应该很熟悉：具有一个方法和关联类型的特征。 新部分是 Rhs=Self：此语法称为默认类型参数。 Rhs 泛型类型参数（“右手边”的缩写）定义了 add 方法中 rhs 参数的类型。 如果我们在实现 Add 特征时没有为 Rhs 指定具体类型，那么 Rhs 的类型将默认为 Self，这将是我们实现 Add 的类型。


当我们为 Point 实现 Add 时，我们使用了 Rhs 的默认值，因为我们想要添加两个 Point 实例。 让我们看一个实现 Add 特征的示例，其中我们想要自定义 Rhs 类型而不是使用默认值。

我们有两个结构体，毫米和米，保存不同单位的值。 这种对现有类型在另一个结构中的薄包装称为 newtype 模式，我们在“使用 Newtype 模式在外部类型上实现外部特征”部分中对其进行了更详细的描述。 我们希望将以毫米为单位的值与以米为单位的值相加，并让 Add 的实现正确执行转换。 我们可以用米作为 Rhs 来实现毫米的加法，如清单 19-15 所示。

Filename: src/lib.rs
``````
use std::ops::Add;

struct Millimeters(u32);
struct Meters(u32);

impl Add<Meters> for Millimeters {
    type Output = Millimeters;

    fn add(self, other: Meters) -> Millimeters {
        Millimeters(self.0 + (other.0 * 1000))
    }
}
``````
Listing 19-15: Implementing the Add trait on Millimeters to add Millimeters to Meters

To add Millimeters and Meters, we specify impl Add<Meters> to set the value of the Rhs type parameter instead of using the default of Self.

要添加毫米和米，我们指定 impl Add<Meters> 来设置 Rhs 类型参数的值，而不是使用默认的 Self。

You’ll use default type parameters in two main ways:

+ To extend a type without breaking existing code
+ To allow customization in specific cases most users won’t need

+ 在不破坏现有代码的情况下扩展类型
+ 允许在大多数用户不需要的特定情况下进行定制

The standard library’s Add trait is an example of the second purpose: usually, you’ll add two like types, but the Add trait provides the ability to customize beyond that. Using a default type parameter in the Add trait definition means you don’t have to specify the extra parameter most of the time. In other words, a bit of implementation boilerplate isn’t needed, making it easier to use the trait.

The first purpose is similar to the second but in reverse: if you want to add a type parameter to an existing trait, you can give it a default to allow extension of the functionality of the trait without breaking the existing implementation code.

标准库的 Add 特征是第二个目的的一个示例：通常，您将添加两个类似的类型，但 Add 特征提供了除此之外的自定义功能。 在添加特征定义中使用默认类型参数意味着大多数时候您不必指定额外的参数。 换句话说，不需要一些实现样板，从而更容易使用该特征。

第一个目的与第二个目的类似，但相反：如果您想向现有特征添加类型参数，您可以为其指定默认值，以允许扩展特征的功能而不破坏现有的实现代码。

### 用于消除歧义的完全限定语法：调用同名方法
Nothing in Rust prevents a trait from having a method with the same name as another trait’s method, nor does Rust prevent you from implementing both traits on one type. It’s also possible to implement a method directly on the type with the same name as methods from traits.

When calling methods with the same name, you’ll need to tell Rust which one you want to use. Consider the code in Listing 19-16 where we’ve defined two traits, Pilot and Wizard, that both have a method called fly. We then implement both traits on a type Human that already has a method named fly implemented on it. Each fly method does something different.

Rust 中的任何内容都不会阻止一个特征具有与另一个特征的方法同名的方法，Rust 也不会阻止您在一种类型上实现这两个特征。 还可以直接在类型上实现与特征中的方法同名的方法。

当调用同名方法时，你需要告诉 Rust 你想使用哪一个。 考虑清单 19-16 中的代码，其中我们定义了两个特征：Pilot 和 Wizard，它们都有一个名为 Fly 的方法。 然后，我们在 Human 类型上实现这两个特征，该类型已经实现了名为 Fly 的方法。 每种飞行方法都有不同的作用。

Filename: src/main.rs
``````
trait Pilot {
    fn fly(&self);
}

trait Wizard {
    fn fly(&self);
}

struct Human;

impl Pilot for Human {
    fn fly(&self) {
        println!("This is your captain speaking.");
    }
}

impl Wizard for Human {
    fn fly(&self) {
        println!("Up!");
    }
}

impl Human {
    fn fly(&self) {
        println!("*waving arms furiously*");
    }
}
``````
Listing 19-16: Two traits are defined to have a fly method and are implemented on the Human type, and a fly method is implemented on Human directly

When we call fly on an instance of Human, the compiler defaults to calling the method that is directly implemented on the type, as shown in Listing 19-17.

当我们在 Human 实例上调用 Fly 时，编译器默认调用直接在该类型上实现的方法，如清单 19-17 所示。

Filename: src/main.rs
``````
fn main() {
    let person = Human;
    person.fly();
}
``````
Listing 19-17: Calling fly on an instance of Human

Running this code will print *waving arms furiously*, showing that Rust called the fly method implemented on Human directly.

To call the fly methods from either the Pilot trait or the Wizard trait, we need to use more explicit syntax to specify which fly method we mean. Listing 19-18 demonstrates this syntax.

运行这段代码会打印出*wawingarmsfuriously*，表明Rust直接调用了Human上实现的fly方法。

要从 Pilot 特征或 Wizard 特征调用 Fly 方法，我们需要使用更明确的语法来指定我们指的是哪个 Fly 方法。 清单 19-18 演示了这种语法。

Filename: src/main.rs
``````
fn main() {
    let person = Human;
    Pilot::fly(&person);
    Wizard::fly(&person);
    person.fly();
}
``````
Listing 19-18: Specifying which trait’s fly method we want to call

Specifying the trait name before the method name clarifies to Rust which implementation of fly we want to call. We could also write Human::fly(&person), which is equivalent to the person.fly() that we used in Listing 19-18, but this is a bit longer to write if we don’t need to disambiguate.

在方法名称之前指定特征名称可以向 Rust 阐明我们要调用哪个 Fly 实现。 我们还可以编写 Human::fly(&person)，它相当于我们在清单 19-18 中使用的 person.fly()，但是如果我们不需要消除歧义，那么写起来会有点长。

Running this code prints the following:
``````
$ cargo run
   Compiling traits-example v0.1.0 (file:///projects/traits-example)
    Finished dev [unoptimized + debuginfo] target(s) in 0.46s
     Running `target/debug/traits-example`
This is your captain speaking.
Up!
*waving arms furiously*
``````
Because the fly method takes a self parameter, if we had two types that both implement one trait, Rust could figure out which implementation of a trait to use based on the type of self.

However, associated functions that are not methods don’t have a self parameter. When there are multiple types or traits that define non-method functions with the same function name, Rust doesn't always know which type you mean unless you use fully qualified syntax. For example, in Listing 19-19 we create a trait for an animal shelter that wants to name all baby dogs Spot. We make an Animal trait with an associated non-method function baby_name. The Animal trait is implemented for the struct Dog, on which we also provide an associated non-method function baby_name directly.

因为 Fly 方法采用 self 参数，所以如果我们有两种都实现一个特征的类型，Rust 可以根据 self 的类型确定要使用哪个特征的实现。

但是，不是方法的关联函数没有 self 参数。 当有多个类型或特征定义具有相同函数名称的非方法函数时，Rust 并不总是知道您指的是哪种类型，除非您使用完全限定语法。 例如，在清单 19-19 中，我们为动物收容所创建了一个特征，想要将所有小狗命名为 Spot。 我们使用关联的非方法函数baby_name 创建一个 Animal 特征。 Animal 特征是为 struct Dog 实现的，我们还直接在其上提供了关联的非方法函数baby_name。

Filename: src/main.rs
``````
trait Animal {
    fn baby_name() -> String;
}

struct Dog;

impl Dog {
    fn baby_name() -> String {
        String::from("Spot")
    }
}

impl Animal for Dog {
    fn baby_name() -> String {
        String::from("puppy")
    }
}

fn main() {
    println!("A baby dog is called a {}", Dog::baby_name());
}
``````
Listing 19-19: A trait with an associated function and a type with an associated function of the same name that also implements the trait

We implement the code for naming all puppies Spot in the baby_name associated function that is defined on Dog. The Dog type also implements the trait Animal, which describes characteristics that all animals have. Baby dogs are called puppies, and that is expressed in the implementation of the Animal trait on Dog in the baby_name function associated with the Animal trait.

In main, we call the Dog::baby_name function, which calls the associated function defined on Dog directly. This code prints the following:

我们在 Dog 上定义的baby_name 关联函数中实现了用于命名所有小狗 Spot 的代码。 Dog 类型还实现了 Animal 特征，它描述了所有动物都具有的特征。 小狗被称为小狗，这通过与 Animal 特征关联的baby_name 函数中 Dog 上的 Animal 特征的实现来表达。

在 main 中，我们调用 Dog::baby_name 函数，该函数直接调用 Dog 上定义的关联函数。 此代码打印以下内容：

``````
$ cargo run
   Compiling traits-example v0.1.0 (file:///projects/traits-example)
    Finished dev [unoptimized + debuginfo] target(s) in 0.54s
     Running `target/debug/traits-example`
A baby dog is called a Spot
``````
This output isn’t what we wanted. We want to call the baby_name function that is part of the Animal trait that we implemented on Dog so the code prints A baby dog is called a puppy. The technique of specifying the trait name that we used in Listing 19-18 doesn’t help here; if we change main to the code in Listing 19-20, we’ll get a compilation error.

这个输出不是我们想要的。 我们想要调用baby_name 函数，该函数是我们在Dog 上实现的Animal 特征的一部分，因此代码会打印出A Baby Dog is called a puppy。 我们在清单 19-18 中使用的指定特征名称的技术在这里没有帮助； 如果我们将 main 更改为清单 19-20 中的代码，我们将收到编译错误。

Filename: src/main.rs
``````
fn main() {
    println!("A baby dog is called a {}", Animal::baby_name());
}
``````
Listing 19-20: Attempting to call the baby_name function from the Animal trait, but Rust doesn’t know which implementation to use

Because Animal::baby_name doesn’t have a self parameter, and there could be other types that implement the Animal trait, Rust can’t figure out which implementation of Animal::baby_name we want. We’ll get this compiler error:

因为 Animal::baby_name 没有 self 参数，并且可能有其他类型实现 Animal 特征，Rust 无法确定我们想要哪种 Animal::baby_name 实现。 我们会得到这个编译器错误：

``````
$ cargo run
   Compiling traits-example v0.1.0 (file:///projects/traits-example)
error[E0790]: cannot call associated function on trait without specifying the corresponding `impl` type
  --> src/main.rs:20:43
   |
2  |     fn baby_name() -> String;
   |     ------------------------- `Animal::baby_name` defined here
...
20 |     println!("A baby dog is called a {}", Animal::baby_name());
   |                                           ^^^^^^^^^^^^^^^^^ cannot call associated function of trait
   |
help: use the fully-qualified path to the only available implementation
   |
20 |     println!("A baby dog is called a {}", <Dog as Animal>::baby_name());
   |                                           +++++++       +

For more information about this error, try `rustc --explain E0790`.
error: could not compile `traits-example` due to previous error
``````
To disambiguate and tell Rust that we want to use the implementation of Animal for Dog as opposed to the implementation of Animal for some other type, we need to use fully qualified syntax. Listing 19-21 demonstrates how to use fully qualified syntax.

为了消除歧义并告诉 Rust，我们想要使用 Dog 的 Animal 实现，而不是其他类型的 Animal 实现，我们需要使用完全限定语法。 清单 19-21 演示了如何使用完全限定语法。

Filename: src/main.rs
``````
fn main() {
    println!("A baby dog is called a {}", <Dog as Animal>::baby_name());
}
``````
Listing 19-21: Using fully qualified syntax to specify that we want to call the baby_name function from the Animal trait as implemented on Dog

We’re providing Rust with a type annotation within the angle brackets, which indicates we want to call the baby_name method from the Animal trait as implemented on Dog by saying that we want to treat the Dog type as an Animal for this function call. This code will now print what we want:

我们为 Rust 提供了尖括号内的类型注释，这表明我们想要从 Animal 特征中调用 Baby_name 方法（如在 Dog 上实现），表示我们希望将 Dog 类型视为 Animal 来进行此函数调用。 此代码现在将打印我们想要的内容：

``````
$ cargo run
   Compiling traits-example v0.1.0 (file:///projects/traits-example)
    Finished dev [unoptimized + debuginfo] target(s) in 0.48s
     Running `target/debug/traits-example`
A baby dog is called a puppy
``````
In general, fully qualified syntax is defined as follows:
``````
<Type as Trait>::function(receiver_if_method, next_arg, ...);
``````
For associated functions that aren’t methods, there would not be a receiver: there would only be the list of other arguments. You could use fully qualified syntax everywhere that you call functions or methods. However, you’re allowed to omit any part of this syntax that Rust can figure out from other information in the program. You only need to use this more verbose syntax in cases where there are multiple implementations that use the same name and Rust needs help to identify which implementation you want to call.

对于不是方法的关联函数，不会有接收者：只有其他参数的列表。 您可以在调用函数或方法的任何地方使用完全限定语法。 但是，您可以省略此语法中 Rust 可以从程序中的其他信息中找出的任何部分。 仅当存在多个使用相同名称的实现并且 Rust 需要帮助来识别您要调用哪个实现时，您才需要使用这种更详细的语法。

### Using Supertraits to Require One Trait’s Functionality Within Another Trait
Sometimes, you might write a trait definition that depends on another trait: for a type to implement the first trait, you want to require that type to also implement the second trait. You would do this so that your trait definition can make use of the associated items of the second trait. The trait your trait definition is relying on is called a supertrait of your trait.

For example, let’s say we want to make an OutlinePrint trait with an outline_print method that will print a given value formatted so that it's framed in asterisks. That is, given a Point struct that implements the standard library trait Display to result in (x, y), when we call outline_print on a Point instance that has 1 for x and 3 for y, it should print the following:

有时，您可能会编写依赖于另一个特征的特征定义：对于实现第一个特征的类型，您希望要求该类型也实现第二个特征。 您可以这样做，以便您的特征定义可以利用第二个特征的关联项。 你的特质定义所依赖的特质被称为你特质的超特质。

例如，假设我们想要使用outline_print方法创建一个OutlinePrint特征，该方法将打印给定的格式化值，以便它被星号框住。 也就是说，给定一个实现标准库特征 Display 来生成 (x, y) 的 Point 结构，当我们在 x 为 1、y 为 3 的 Point 实例上调用 Outline_print 时，它应该打印以下内容：

``````
**********
*        *
* (1, 3) *
*        *
**********
``````
In the implementation of the outline_print method, we want to use the Display trait’s functionality. Therefore, we need to specify that the OutlinePrint trait will work only for types that also implement Display and provide the functionality that OutlinePrint needs. We can do that in the trait definition by specifying OutlinePrint: Display. This technique is similar to adding a trait bound to the trait. Listing 19-22 shows an implementation of the OutlinePrint trait.

在outline_print方法的实现中，我们想要使用Display特征的功能。 因此，我们需要指定 OutlinePrint 特征仅适用于也实现 Display 并提供 OutlinePrint 所需功能的类型。 我们可以在特征定义中通过指定 OutlinePrint: Display 来做到这一点。 此技术类似于添加与特征绑定的特征。 清单 19-22 显示了 OutlinePrint 特征的实现。

Filename: src/main.rs
``````
use std::fmt;

trait OutlinePrint: fmt::Display {
    fn outline_print(&self) {
        let output = self.to_string();
        let len = output.len();
        println!("{}", "*".repeat(len + 4));
        println!("*{}*", " ".repeat(len + 2));
        println!("* {} *", output);
        println!("*{}*", " ".repeat(len + 2));
        println!("{}", "*".repeat(len + 4));
    }
}
``````
Listing 19-22: Implementing the OutlinePrint trait that requires the functionality from Display

Because we’ve specified that OutlinePrint requires the Display trait, we can use the to_string function that is automatically implemented for any type that implements Display. If we tried to use to_string without adding a colon and specifying the Display trait after the trait name, we’d get an error saying that no method named to_string was found for the type &Self in the current scope.

Let’s see what happens when we try to implement OutlinePrint on a type that doesn’t implement Display, such as the Point struct:

因为我们已经指定 OutlinePrint 需要 Display 特征，所以我们可以使用为任何实现 Display 的类型自动实现的 to_string 函数。 如果我们尝试使用 to_string 而不添加冒号并在特征名称后指定 Display 特征，我们会收到一条错误，指出在当前作用域中找不到类型 &Self 的名为 to_string 的方法。

让我们看看当我们尝试在未实现 Display 的类型（例如 Point 结构）上实现 OutlinePrint 时会发生什么：

Filename: src/main.rs
``````
This code does not compile!
struct Point {
    x: i32,
    y: i32,
}

impl OutlinePrint for Point {}
``````
We get an error saying that Display is required but not implemented:
``````
$ cargo run
   Compiling traits-example v0.1.0 (file:///projects/traits-example)
error[E0277]: `Point` doesn't implement `std::fmt::Display`
  --> src/main.rs:20:6
   |
20 | impl OutlinePrint for Point {}
   |      ^^^^^^^^^^^^ `Point` cannot be formatted with the default formatter
   |
   = help: the trait `std::fmt::Display` is not implemented for `Point`
   = note: in format strings you may be able to use `{:?}` (or {:#?} for pretty-print) instead
note: required by a bound in `OutlinePrint`
  --> src/main.rs:3:21
   |
3  | trait OutlinePrint: fmt::Display {
   |                     ^^^^^^^^^^^^ required by this bound in `OutlinePrint`

For more information about this error, try `rustc --explain E0277`.
error: could not compile `traits-example` due to previous error
``````
To fix this, we implement Display on Point and satisfy the constraint that OutlinePrint requires, like so:

为了解决这个问题，我们实现 Display on Point 并满足 OutlinePrint 所需的约束，如下所示：

Filename: src/main.rs
``````
use std::fmt;

impl fmt::Display for Point {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "({}, {})", self.x, self.y)
    }
}
``````
Then implementing the OutlinePrint trait on Point will compile successfully, and we can call outline_print on a Point instance to display it within an outline of asterisks.

Using the Newtype Pattern to Implement External Traits on External Types
In Chapter 10 in the “Implementing a Trait on a Type” section, we mentioned the orphan rule that states we’re only allowed to implement a trait on a type if either the trait or the type are local to our crate. It’s possible to get around this restriction using the newtype pattern, which involves creating a new type in a tuple struct. (We covered tuple structs in the “Using Tuple Structs without Named Fields to Create Different Types” section of Chapter 5.) The tuple struct will have one field and be a thin wrapper around the type we want to implement a trait for. Then the wrapper type is local to our crate, and we can implement the trait on the wrapper. Newtype is a term that originates from the Haskell programming language. There is no runtime performance penalty for using this pattern, and the wrapper type is elided at compile time.

As an example, let’s say we want to implement Display on Vec<T>, which the orphan rule prevents us from doing directly because the Display trait and the Vec<T> type are defined outside our crate. We can make a Wrapper struct that holds an instance of Vec<T>; then we can implement Display on Wrapper and use the Vec<T> value, as shown in Listing 19-23.

然后在 Point 上实现 OutlinePrint 特征将成功编译，我们可以在 Point 实例上调用 Outline_print 将其显示在星号轮廓内。

使用 Newtype 模式在外部类型上实现外部特征
在第 10 章“在类型上实现特征”部分中，我们提到了孤儿规则，该规则规定，只有当特征或类型对于我们的板条箱来说是本地的时，才允许我们在类型上实现特征。 可以使用 newtype 模式来绕过此限制，该模式涉及在元组结构中创建新类型。 （我们在第 5 章的“使用不带命名字段的元组结构创建不同类型”一节中介绍了元组结构。）元组结构将有一个字段，并且是我们要为其实现特征的类型的薄包装。 那么包装器类型对于我们的板条箱来说是本地的，我们可以在包装器上实现该特征。 Newtype 是一个源自 Haskell 编程语言的术语。 使用此模式不会造成运行时性能损失，并且包装器类型会在编译时被忽略。

举个例子，假设我们想要在 Vec<T> 上实现 Display，孤儿规则阻止我们直接执行此操作，因为 Display 特征和 Vec<T> 类型是在我们的板条箱外部定义的。 我们可以创建一个 Wrapper 结构来保存 Vec<T> 的实例； 然后我们可以在 Wrapper 上实现 Display 并使用 Vec<T> 值，如清单 19-23 所示。

Filename: src/main.rs
``````
use std::fmt;

struct Wrapper(Vec<String>);

impl fmt::Display for Wrapper {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}

fn main() {
    let w = Wrapper(vec![String::from("hello"), String::from("world")]);
    println!("w = {}", w);
}
``````
Listing 19-23: Creating a Wrapper type around Vec<String> to implement Display

The implementation of Display uses self.0 to access the inner Vec<T>, because Wrapper is a tuple struct and Vec<T> is the item at index 0 in the tuple. Then we can use the functionality of the Display type on Wrapper.

The downside of using this technique is that Wrapper is a new type, so it doesn’t have the methods of the value it’s holding. We would have to implement all the methods of Vec<T> directly on Wrapper such that the methods delegate to self.0, which would allow us to treat Wrapper exactly like a Vec<T>. If we wanted the new type to have every method the inner type has, implementing the Deref trait (discussed in Chapter 15 in the “Treating Smart Pointers Like Regular References with the Deref Trait” section) on the Wrapper to return the inner type would be a solution. If we don’t want the Wrapper type to have all the methods of the inner type—for example, to restrict the Wrapper type’s behavior—we would have to implement just the methods we do want manually.

This newtype pattern is also useful even when traits are not involved. Let’s switch focus and look at some advanced ways to interact with Rust’s type system.

Display 的实现使用 self.0 来访问内部 Vec<T>，因为 Wrapper 是一个元组结构体，而 Vec<T> 是元组中索引 0 处的项。 然后我们就可以在Wrapper上使用Display类型的功能了。

使用这种技术的缺点是 Wrapper 是一种新类型，因此它没有其所持有的值的方法。 我们必须直接在 Wrapper 上实现 Vec<T> 的所有方法，以便这些方法委托给 self.0，这将允许我们像对待 Vec<T> 一样对待 Wrapper。 如果我们希望新类型具有内部类型具有的所有方法，则在包装器上实现 Deref 特征（在第 15 章“使用 Deref 特征像常规引用一样对待智能指针”部分中讨论）以返回内部类型将是 一个办法。 如果我们不希望 Wrapper 类型拥有内部类型的所有方法（例如，限制 Wrapper 类型的行为），我们就必须手动实现我们确实想要的方法。

即使不涉及特征，这种新类型模式也很有用。 让我们转移焦点，看看一些与 Rust 类型系统交互的高级方法。

## 19.3 Advanced Types

The Rust type system has some features that we’ve so far mentioned but haven’t yet discussed. We’ll start by discussing newtypes in general as we examine why newtypes are useful as types. Then we’ll move on to type aliases, a feature similar to newtypes but with slightly different semantics. We’ll also discuss the ! type and dynamically sized types.

Rust 类型系统具有一些我们到目前为止已经提到但尚未讨论的功能。 当我们研究为什么新类型作为类型有用时，我们将首先讨论新类型。 然后我们将继续讨论类型别名，这是一个与新类型类似的功能，但语义略有不同。 我们还将讨论！ 类型和动态大小的类型。

### Using the Newtype Pattern for Type Safety and Abstraction
Note: This section assumes you’ve read the earlier section “Using the Newtype Pattern to Implement External Traits on External Types.”

The newtype pattern is also useful for tasks beyond those we’ve discussed so far, including statically enforcing that values are never confused and indicating the units of a value. You saw an example of using newtypes to indicate units in Listing 19-15: recall that the Millimeters and Meters structs wrapped u32 values in a newtype. If we wrote a function with a parameter of type Millimeters, we couldn’t compile a program that accidentally tried to call that function with a value of type Meters or a plain u32.

We can also use the newtype pattern to abstract away some implementation details of a type: the new type can expose a public API that is different from the API of the private inner type.

Newtypes can also hide internal implementation. For example, we could provide a People type to wrap a HashMap<i32, String> that stores a person’s ID associated with their name. Code using People would only interact with the public API we provide, such as a method to add a name string to the People collection; that code wouldn’t need to know that we assign an i32 ID to names internally. The newtype pattern is a lightweight way to achieve encapsulation to hide implementation details, which we discussed in the “Encapsulation that Hides Implementation Details” section of Chapter 17.

newtype 模式对于我们迄今为止讨论过的任务也很有用，包括静态地强制值永远不会混淆并指示值的单位。 您在清单 19-15 中看到了一个使用新类型来指示单位的示例：回想一下，毫米和米结构将 u32 值包装在新类型中。 如果我们编写一个带有毫米类型参数的函数，我们就无法编译一个意外尝试使用米类型值或普通 u32 调用该函数的程序。

我们还可以使用 newtype 模式来抽象出类型的一些实现细节：新类型可以公开与私有内部类型的 API 不同的公共 API。

新类型还可以隐藏内部实现。 例如，我们可以提供 People 类型来包装 HashMap<i32, String>，该 HashMap<i32, String> 存储与其姓名关联的人员 ID。 使用 People 的代码只会与我们提供的公共 API 进行交互，例如向 People 集合添加名称字符串的方法； 该代码不需要知道我们在内部为名称分配了 i32 ID。 newtype 模式是一种实现封装以隐藏实现细节的轻量级方法，我们在第 17 章的“隐藏实现细节的封装”部分中对此进行了讨论。

### Creating Type Synonyms with Type Aliases

Rust provides the ability to declare a type alias to give an existing type another name. For this we use the type keyword. For example, we can create the alias Kilometers to i32 like so:

Rust 提供了声明类型别名的能力，以便为现有类型提供另一个名称。 为此，我们使用 type 关键字。 例如，我们可以创建 i32 的别名 Kilometres，如下所示：

``````
    type Kilometers = i32;
``````
Now, the alias Kilometers is a synonym for i32; unlike the Millimeters and Meters types we created in Listing 19-15, Kilometers is not a separate, new type. Values that have the type Kilometers will be treated the same as values of type i32:

现在，别名“Kilometers”是 i32 的同义词； 与我们在清单 19-15 中创建的 Millimeters 和 Meters 类型不同，Kilometers 不是一个单独的新类型。 公里类型的值将被视为与 i32 类型的值相同：

``````
    type Kilometers = i32;

    let x: i32 = 5;
    let y: Kilometers = 5;

    println!("x + y = {}", x + y);
``````
Because Kilometers and i32 are the same type, we can add values of both types and we can pass Kilometers values to functions that take i32 parameters. However, using this method, we don’t get the type checking benefits that we get from the newtype pattern discussed earlier. In other words, if we mix up Kilometers and i32 values somewhere, the compiler will not give us an error.

The main use case for type synonyms is to reduce repetition. For example, we might have a lengthy type like this:

由于 Kilometers 和 i32 是相同类型，因此我们可以将两种类型的值相加，并且可以将 Kilometers 值传递给采用 i32 参数的函数。 但是，使用此方法，我们无法获得从前面讨论的 newtype 模式中获得的类型检查优势。 换句话说，如果我们在某处混合了千米和 i32 值，编译器不会给我们错误。

类型同义词的主要用例是减少重复。 例如，我们可能有一个像这样的冗长类型：

``````
Box<dyn Fn() + Send + 'static>
``````
Writing this lengthy type in function signatures and as type annotations all over the code can be tiresome and error prone. Imagine having a project full of code like that in Listing 19-24.

在整个代码中将这种冗长的类型写入函数签名和类型注释可能会很烦人并且容易出错。 想象一下，有一个项目充满了如清单 19-24 所示的代码。

``````
    let f: Box<dyn Fn() + Send + 'static> = Box::new(|| println!("hi"));

    fn takes_long_type(f: Box<dyn Fn() + Send + 'static>) {
        // --snip--
    }

    fn returns_long_type() -> Box<dyn Fn() + Send + 'static> {
        // --snip--
    }
``````
Listing 19-24: Using a long type in many places

A type alias makes this code more manageable by reducing the repetition. In Listing 19-25, we’ve introduced an alias named Thunk for the verbose type and can replace all uses of the type with the shorter alias Thunk.

类型别名通过减少重复使该代码更易于管理。 在清单 19-25 中，我们为详细类型引入了一个名为 Thunk 的别名，并且可以用较短的别名 Thunk 替换该类型的所有使用。

``````
    type Thunk = Box<dyn Fn() + Send + 'static>;

    let f: Thunk = Box::new(|| println!("hi"));

    fn takes_long_type(f: Thunk) {
        // --snip--
    }

    fn returns_long_type() -> Thunk {
        // --snip--
    }
``````
Listing 19-25: Introducing a type alias Thunk to reduce repetition

This code is much easier to read and write! Choosing a meaningful name for a type alias can help communicate your intent as well (thunk is a word for code to be evaluated at a later time, so it’s an appropriate name for a closure that gets stored).

Type aliases are also commonly used with the Result<T, E> type for reducing repetition. Consider the std::io module in the standard library. I/O operations often return a Result<T, E> to handle situations when operations fail to work. This library has a std::io::Error struct that represents all possible I/O errors. Many of the functions in std::io will be returning Result<T, E> where the E is std::io::Error, such as these functions in the Write trait:

这段代码更容易阅读和编写！ 为类型别名选择一个有意义的名称也可以帮助传达您的意图（thunk 是一个词，表示稍后要评估的代码，因此它是存储闭包的合适名称）。

类型别名也常与 Result<T, E> 类型一起使用，以减少重复。 考虑标准库中的 std::io 模块。 I/O 操作通常返回 Result<T, E> 来处理操作失败的情况。 该库有一个 std::io::Error 结构，表示所有可能的 I/O 错误。 std::io 中的许多函数将返回 Result<T, E>，其中 E 是 std::io::Error，例如 Write 特征中的这些函数：

``````
use std::fmt;
use std::io::Error;

pub trait Write {
    fn write(&mut self, buf: &[u8]) -> Result<usize, Error>;
    fn flush(&mut self) -> Result<(), Error>;

    fn write_all(&mut self, buf: &[u8]) -> Result<(), Error>;
    fn write_fmt(&mut self, fmt: fmt::Arguments) -> Result<(), Error>;
}
``````
The Result<..., Error> is repeated a lot. As such, std::io has this type alias declaration:
``````
type Result<T> = std::result::Result<T, std::io::Error>;
``````
Because this declaration is in the std::io module, we can use the fully qualified alias std::io::Result<T>; that is, a Result<T, E> with the E filled in as std::io::Error. The Write trait function signatures end up looking like this:

因为此声明位于 std::io 模块中，所以我们可以使用完全限定别名 std::io::Result<T>; 即，一个 Result<T, E>，其中 E 填写为 std::io::Error。 Write 特征函数签名最终看起来像这样：

``````
pub trait Write {
    fn write(&mut self, buf: &[u8]) -> Result<usize>;
    fn flush(&mut self) -> Result<()>;

    fn write_all(&mut self, buf: &[u8]) -> Result<()>;
    fn write_fmt(&mut self, fmt: fmt::Arguments) -> Result<()>;
}
``````
The type alias helps in two ways: it makes code easier to write and it gives us a consistent interface across all of std::io. Because it’s an alias, it’s just another Result<T, E>, which means we can use any methods that work on Result<T, E> with it, as well as special syntax like the ? operator.

类型别名有两个作用：它使代码更容易编写，并为我们提供了跨所有 std::io 的一致接口。 因为它是一个别名，所以它只是另一个 Result<T, E>，这意味着我们可以使用任何适用于 Result<T, E> 的方法，以及像 ? 这样的特殊语法。 操作员。

### The Never Type that Never Returns
Rust has a special type named ! that’s known in type theory lingo as the empty type because it has no values. We prefer to call it the never type because it stands in the place of the return type when a function will never return. Here is an example:

Rust 有一个特殊类型，名为 ! 这在类型论术语中被称为空类型，因为它没有值。 我们更喜欢将其称为 never 类型，因为当函数永远不会返回时，它代表返回类型。 这是一个例子：

``````
fn bar() -> ! {
    // --snip--
}
``````
This code is read as “the function bar returns never.” Functions that return never are called diverging functions. We can’t create values of the type ! so bar can never possibly return.

But what use is a type you can never create values for? Recall the code from Listing 2-5, part of the number guessing game; we’ve reproduced a bit of it here in Listing 19-26.

这段代码被解读为“函数栏从不返回”。 从不返回的函数称为发散函数。 我们无法创建该类型的值！ 所以酒吧永远不可能回来。

但是，永远无法为其创建值的类型有什么用呢？ 回想一下清单 2-5 中的代码，这是猜数字游戏的一部分； 我们在清单 19-26 中复制了其中的一部分。

``````
        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };
``````
Listing 19-26: A match with an arm that ends in continue

At the time, we skipped over some details in this code. In Chapter 6 in “The match Control Flow Operator” section, we discussed that match arms must all return the same type. So, for example, the following code doesn’t work:

当时，我们跳过了这段代码中的一些细节。 在第 6 章“匹配控制流运算符”部分中，我们讨论了匹配臂必须返回相同的类型。 因此，例如，以下代码不起作用：
``````
This code does not compile!
    let guess = match guess.trim().parse() {
        Ok(_) => 5,
        Err(_) => "hello",
    };
``````
The type of guess in this code would have to be an integer and a string, and Rust requires that guess have only one type. So what does continue return? How were we allowed to return a u32 from one arm and have another arm that ends with continue in Listing 19-26?

As you might have guessed, continue has a ! value. That is, when Rust computes the type of guess, it looks at both match arms, the former with a value of u32 and the latter with a ! value. Because ! can never have a value, Rust decides that the type of guess is u32.

The formal way of describing this behavior is that expressions of type ! can be coerced into any other type. We’re allowed to end this match arm with continue because continue doesn’t return a value; instead, it moves control back to the top of the loop, so in the Err case, we never assign a value to guess.

The never type is useful with the panic! macro as well. Recall the unwrap function that we call on Option<T> values to produce a value or panic with this definition:

这段代码中的猜测类型必须是整数和字符串，而 Rust 要求猜测只有一种类型。 那么 continue 返回什么呢？ 我们如何被允许从一个分支返回 u32 并让另一个分支以清单 19-26 中的 continue 结尾？

正如您可能已经猜到的，继续有一个！ 价值。 也就是说，当 Rust 计算猜测类型时，它会查看两个匹配臂，前者的值为 u32，后者的值为 ! 价值。 因为 ！ 永远不可能有值，Rust 决定猜测的类型是 u32。

描述这种行为的正式方式是类型表达式 ! 可以强制转换为任何其他类型。 我们可以用 continue 来结束这个匹配臂，因为 continue 不返回值； 相反，它将控制移回循环顶部，因此在 Err 情况下，我们永远不会分配猜测值。

never 类型对于恐慌很有用！ 宏观也是如此。 回想一下我们对 Option<T> 值调用的 unwrap 函数，以使用以下定义生成值或恐慌：

``````
impl<T> Option<T> {
    pub fn unwrap(self) -> T {
        match self {
            Some(val) => val,
            None => panic!("called `Option::unwrap()` on a `None` value"),
        }
    }
}
``````
In this code, the same thing happens as in the match in Listing 19-26: Rust sees that val has the type T and panic! has the type !, so the result of the overall match expression is T. This code works because panic! doesn’t produce a value; it ends the program. In the None case, we won’t be returning a value from unwrap, so this code is valid.

在此代码中，发生了与清单 19-26 中的匹配相同的情况：Rust 看到 val 具有类型 T 并恐慌！ 具有类型 !，因此整个匹配表达式的结果是 T。此代码之所以有效，是因为panic! 不产生价值； 它结束程序。 在 None 情况下，我们不会从 unwrap 返回值，因此此代码是有效的。

One final expression that has the type ! is a loop:
``````
    print!("forever ");

    loop {
        print!("and ever ");
    }
``````
Here, the loop never ends, so ! is the value of the expression. However, this wouldn’t be true if we included a break, because the loop would terminate when it got to the break.

### Dynamically Sized Types and the Sized Trait
Rust needs to know certain details about its types, such as how much space to allocate for a value of a particular type. This leaves one corner of its type system a little confusing at first: the concept of dynamically sized types. Sometimes referred to as DSTs or unsized types, these types let us write code using values whose size we can know only at runtime.

Let’s dig into the details of a dynamically sized type called str, which we’ve been using throughout the book. That’s right, not &str, but str on its own, is a DST. We can’t know how long the string is until runtime, meaning we can’t create a variable of type str, nor can we take an argument of type str. Consider the following code, which does not work:

Rust 需要了解有关其类型的某些详细信息，例如为特定类型的值分配多少空间。 这使得其类型系统的一个角落一开始有点令人困惑：动态大小类型的概念。 有时称为 DST 或未调整大小的类型，这些类型允许我们使用只能在运行时知道其大小的值来编写代码。

让我们深入研究一个名为 str 的动态大小类型的细节，我们在整本书中一直在使用它。 没错，不是 &str，而是 str 本身就是 DST。 直到运行时我们才知道字符串有多长，这意味着我们不能创建 str 类型的变量，也不能接受 str 类型的参数。 考虑下面的代码，它不起作用：

``````
    let s1: str = "Hello there!";
    let s2: str = "How's it going?";
``````
Rust needs to know how much memory to allocate for any value of a particular type, and all values of a type must use the same amount of memory. If Rust allowed us to write this code, these two str values would need to take up the same amount of space. But they have different lengths: s1 needs 12 bytes of storage and s2 needs 15. This is why it’s not possible to create a variable holding a dynamically sized type.

So what do we do? In this case, you already know the answer: we make the types of s1 and s2 a &str rather than a str. Recall from the “String Slices” section of Chapter 4 that the slice data structure just stores the starting position and the length of the slice. So although a &T is a single value that stores the memory address of where the T is located, a &str is two values: the address of the str and its length. As such, we can know the size of a &str value at compile time: it’s twice the length of a usize. That is, we always know the size of a &str, no matter how long the string it refers to is. In general, this is the way in which dynamically sized types are used in Rust: they have an extra bit of metadata that stores the size of the dynamic information. The golden rule of dynamically sized types is that we must always put values of dynamically sized types behind a pointer of some kind.

We can combine str with all kinds of pointers: for example, Box<str> or Rc<str>. In fact, you’ve seen this before but with a different dynamically sized type: traits. Every trait is a dynamically sized type we can refer to by using the name of the trait. In Chapter 17 in the “Using Trait Objects That Allow for Values of Different Types” section, we mentioned that to use traits as trait objects, we must put them behind a pointer, such as &dyn Trait or Box<dyn Trait> (Rc<dyn Trait> would work too).

To work with DSTs, Rust provides the Sized trait to determine whether or not a type’s size is known at compile time. This trait is automatically implemented for everything whose size is known at compile time. In addition, Rust implicitly adds a bound on Sized to every generic function. That is, a generic function definition like this:

Rust 需要知道为特定类型的任何值分配多少内存，并且同一类型的所有值必须使用相同的内存量。 如果 Rust 允许我们编写这段代码，那么这两个 str 值将需要占用相同的空间量。 但它们的长度不同：s1 需要 12 个字节的存储空间，而 s2 需要 15 个字节。这就是为什么无法创建保存动态大小类型的变量的原因。

那么我们该怎么办？ 在这种情况下，您已经知道答案：我们将 s1 和 s2 的类型设为 &str 而不是 str。 回想一下第 4 章的“字符串切片”部分，切片数据结构只存储切片的起始位置和长度。 因此，虽然 &T 是存储 T 所在内存地址的单个值，但 &str 是两个值：str 的地址及其长度。 因此，我们可以在编译时知道 &str 值的大小：它是 usize 长度的两倍。 也就是说，我们总是知道 &str 的大小，无论它引用的字符串有多长。 一般来说，这是 Rust 中使用动态大小类型的方式：它们有一个额外的元数据来存储动态信息的大小。 动态大小类型的黄金法则是，我们必须始终将动态大小类型的值放在某种指针后面。

我们可以将 str 与各种指针组合：例如 Box<str> 或 Rc<str>。 事实上，您以前已经见过这种情况，但具有不同的动态大小类型：特征。 每个特征都是动态大小的类型，我们可以通过使用特征的名称来引用。 在第 17 章“使用允许不同类型值的 Trait 对象”部分中，我们提到要将 Trait 用作 Trait 对象，我们必须将它们放在指针后面，例如 &dyn Trait 或 Box<dyn Trait> (Rc< dyn Trait> 也可以工作）。

为了使用 DST，Rust 提供了 Sized 特征来确定类型的大小在编译时是否已知。 对于编译时大小已知的所有内容，都会自动实现此特征。 此外，Rust 隐式地为每个泛型函数添加了 Sized 绑定。 也就是说，像这样的通用函数定义：

``````
fn generic<T>(t: T) {
    // --snip--
}
``````
is actually treated as though we had written this:
``````
fn generic<T: Sized>(t: T) {
    // --snip--
}
``````
By default, generic functions will work only on types that have a known size at compile time. However, you can use the following special syntax to relax this restriction:
``````
fn generic<T: ?Sized>(t: &T) {
    // --snip--
}
``````
A trait bound on ?Sized means “T may or may not be Sized” and this notation overrides the default that generic types must have a known size at compile time. The ?Trait syntax with this meaning is only available for Sized, not any other traits.

Also note that we switched the type of the t parameter from T to &T. Because the type might not be Sized, we need to use it behind some kind of pointer. In this case, we’ve chosen a reference.

Next, we’ll talk about functions and closures!

?Sized 上的特征绑定意味着“T 可能会或可能不会被调整大小”，并且此表示法会覆盖泛型类型在编译时必须具有已知大小的默认值。 具有此含义的 ?Trait 语法仅适用于 Sized，不适用于任何其他特征。

另请注意，我们将 t 参数的类型从 T 切换为 &T。 因为该类型可能没有调整大小，所以我们需要在某种指针后面使用它。 在本例中，我们选择了一个参考。

接下来，我们将讨论函数和闭包！

## 19.4 Advanced Functions and Closures
This section explores some advanced features related to functions and closures, including function pointers and returning closures.

本节探讨与函数和闭包相关的一些高级功能，包括函数指针和返回闭包。

### Function Pointers
We’ve talked about how to pass closures to functions; you can also pass regular functions to functions! This technique is useful when you want to pass a function you’ve already defined rather than defining a new closure. Functions coerce to the type fn (with a lowercase f), not to be confused with the Fn closure trait. The fn type is called a function pointer. Passing functions with function pointers will allow you to use functions as arguments to other functions.

The syntax for specifying that a parameter is a function pointer is similar to that of closures, as shown in Listing 19-27, where we’ve defined a function add_one that adds one to its parameter. The function do_twice takes two parameters: a function pointer to any function that takes an i32 parameter and returns an i32, and one i32 value. The do_twice function calls the function f twice, passing it the arg value, then adds the two function call results together. The main function calls do_twice with the arguments add_one and 5.

我们已经讨论了如何将闭包传递给函数； 您还可以将常规函数传递给函数！ 当您想要传递已经定义的函数而不是定义新的闭包时，此技术非常有用。 函数强制转换为 fn 类型（f 小写），不要与 Fn 闭包特征混淆。 fn 类型称为函数指针。 使用函数指针传递函数将允许您将函数用作其他函数的参数。

指定参数为函数指针的语法与闭包的语法类似，如清单 19-27 所示，其中我们定义了一个函数 add_one ，该函数将其参数加一。 函数 do_twice 有两个参数：一个指向任何采用 i32 参数并返回 i32 的函数的函数指针，以及一个 i32 值。 do_twice 函数调用函数 f 两次，向其传递 arg 值，然后将两次函数调用结果相加。 main 函数使用参数 add_one 和 5 调用 do_twice。

Filename: src/main.rs
``````
fn add_one(x: i32) -> i32 {
    x + 1
}

fn do_twice(f: fn(i32) -> i32, arg: i32) -> i32 {
    f(arg) + f(arg)
}

fn main() {
    let answer = do_twice(add_one, 5);

    println!("The answer is: {}", answer);
}
``````
Listing 19-27: Using the fn type to accept a function pointer as an argument

This code prints The answer is: 12. We specify that the parameter f in do_twice is an fn that takes one parameter of type i32 and returns an i32. We can then call f in the body of do_twice. In main, we can pass the function name add_one as the first argument to do_twice.

Unlike closures, fn is a type rather than a trait, so we specify fn as the parameter type directly rather than declaring a generic type parameter with one of the Fn traits as a trait bound.

Function pointers implement all three of the closure traits (Fn, FnMut, and FnOnce), meaning you can always pass a function pointer as an argument for a function that expects a closure. It’s best to write functions using a generic type and one of the closure traits so your functions can accept either functions or closures.

That said, one example of where you would want to only accept fn and not closures is when interfacing with external code that doesn’t have closures: C functions can accept functions as arguments, but C doesn’t have closures.

As an example of where you could use either a closure defined inline or a named function, let’s look at a use of the map method provided by the Iterator trait in the standard library. To use the map function to turn a vector of numbers into a vector of strings, we could use a closure, like this:

这段代码打印答案是： 12. 我们指定 do_twice 中的参数 f 是一个 fn，它接受一个 i32 类型的参数并返回一个 i32。 然后我们可以在 do_twice 主体中调用 f。 在 main 中，我们可以将函数名 add_one 作为第一个参数传递给 do_twice。

与闭包不同，fn 是一种类型而不是特征，因此我们直接指定 fn 作为参数类型，而不是使用 Fn 特征之一作为特征边界来声明泛型类型参数。

函数指针实现了所有三个闭包特征（Fn、FnMut 和 FnOnce），这意味着您始终可以将函数指针作为参数传递给需要闭包的函数。 最好使用泛型类型和闭包特征之一来编写函数，以便您的函数可以接受函数或闭包。

也就是说，您只想接受 fn 而不是闭包的一个例子是与没有闭包的外部代码交互时：C 函数可以接受函数作为参数，但 C 没有闭包。

作为可以使用内联定义的闭包或命名函数的示例，让我们看一下标准库中 Iterator 特征提供的 map 方法的使用。 要使用 map 函数将数字向量转换为字符串向量，我们可以使用闭包，如下所示：

``````
    let list_of_numbers = vec![1, 2, 3];
    let list_of_strings: Vec<String> =
        list_of_numbers.iter().map(|i| i.to_string()).collect();
``````
Or we could name a function as the argument to map instead of the closure, like this:
``````
    let list_of_numbers = vec![1, 2, 3];
    let list_of_strings: Vec<String> =
        list_of_numbers.iter().map(ToString::to_string).collect();
``````
Note that we must use the fully qualified syntax that we talked about earlier in the “Advanced Traits” section because there are multiple functions available named to_string. Here, we’re using the to_string function defined in the ToString trait, which the standard library has implemented for any type that implements Display.

Recall from the “Enum values” section of Chapter 6 that the name of each enum variant that we define also becomes an initializer function. We can use these initializer functions as function pointers that implement the closure traits, which means we can specify the initializer functions as arguments for methods that take closures, like so:

请注意，我们必须使用前面在“高级特征”部分中讨论过的完全限定语法，因为有多个名为 to_string 的函数可用。 在这里，我们使用 ToString 特征中定义的 to_string 函数，标准库已为任何实现 Display 的类型实现了该函数。

回想一下第 6 章的“枚举值”部分，我们定义的每个枚举变量的名称也成为一个初始化函数。 我们可以使用这些初始化函数作为实现闭包特征的函数指针，这意味着我们可以将初始化函数指定为采用闭包的方法的参数，如下所示：

``````
    enum Status {
        Value(u32),
        Stop,
    }

    let list_of_statuses: Vec<Status> = (0u32..20).map(Status::Value).collect();
``````
Here we create Status::Value instances using each u32 value in the range that map is called on by using the initializer function of Status::Value. Some people prefer this style, and some people prefer to use closures. They compile to the same code, so use whichever style is clearer to you.

在这里，我们使用 Status::Value 的初始化函数调用映射范围内的每个 u32 值来创建 Status::Value 实例。 有些人喜欢这种风格，有些人喜欢使用闭包。 它们编译为相同的代码，因此请使用您更清楚的样式。

### Returning Closures
Closures are represented by traits, which means you can’t return closures directly. In most cases where you might want to return a trait, you can instead use the concrete type that implements the trait as the return value of the function. However, you can’t do that with closures because they don’t have a concrete type that is returnable; you’re not allowed to use the function pointer fn as a return type, for example.

闭包由特征表示，这意味着您不能直接返回闭包。 在大多数情况下，您可能想要返回特征，您可以使用实现该特征的具体类型作为函数的返回值。 然而，你不能用闭包来做到这一点，因为它们没有可返回的具体类型； 例如，您不能使用函数指针 fn 作为返回类型。

The following code tries to return a closure directly, but it won’t compile:
``````
fn returns_closure() -> dyn Fn(i32) -> i32 {
    |x| x + 1
}
``````
The compiler error is as follows:
``````
$ cargo build
   Compiling functions-example v0.1.0 (file:///projects/functions-example)
error[E0746]: return type cannot have an unboxed trait object
 --> src/lib.rs:1:25
  |
1 | fn returns_closure() -> dyn Fn(i32) -> i32 {
  |                         ^^^^^^^^^^^^^^^^^^ doesn't have a size known at compile-time
  |
  = note: for information on `impl Trait`, see <https://doc.rust-lang.org/book/ch10-02-traits.html#returning-types-that-implement-traits>
help: use `impl Fn(i32) -> i32` as the return type, as all return paths are of type `[closure@src/lib.rs:2:5: 2:8]`, which implements `Fn(i32) -> i32`
  |
1 | fn returns_closure() -> impl Fn(i32) -> i32 {
  |                         ~~~~~~~~~~~~~~~~~~~

For more information about this error, try `rustc --explain E0746`.
error: could not compile `functions-example` due to previous error
``````
The error references the Sized trait again! Rust doesn’t know how much space it will need to store the closure. We saw a solution to this problem earlier. We can use a trait object:
``````
fn returns_closure() -> Box<dyn Fn(i32) -> i32> {
    Box::new(|x| x + 1)
}
``````
This code will compile just fine. For more about trait objects, refer to the section “Using Trait Objects That Allow for Values of Different Types” in Chapter 17.

Next, let’s look at macros!
## 19.5 Macros

We’ve used macros like println! throughout this book, but we haven’t fully explored what a macro is and how it works. The term macro refers to a family of features in Rust: declarative macros with macro_rules! and three kinds of procedural macros:

我们使用过像 println 这样的宏！ 贯穿本书，但我们还没有完全探讨什么是宏以及它是如何工作的。 术语“宏”指的是 Rust 中的一系列功能：带有 Macro_rules 的声明性宏！ 以及三种过程宏：

+ Custom #[derive] macros that specify code added with the derive attribute used on structs and enums
+ Attribute-like macros that define custom attributes usable on any item
+ Function-like macros that look like function calls but operate on the tokens specified as their argument

+ 自定义#[derive] 宏，指定添加了用于结构和枚举的derive 属性的代码
+ 类似属性的宏，定义可用于任何项目的自定义属性
+ 类似函数的宏，看起来像函数调用，但对指定为其参数的标记进行操作

We’ll talk about each of these in turn, but first, let’s look at why we even need macros when we already have functions.

### The Difference Between Macros and Functions
Fundamentally, macros are a way of writing code that writes other code, which is known as metaprogramming. In Appendix C, we discuss the derive attribute, which generates an implementation of various traits for you. We’ve also used the println! and vec! macros throughout the book. All of these macros expand to produce more code than the code you’ve written manually.

Metaprogramming is useful for reducing the amount of code you have to write and maintain, which is also one of the roles of functions. However, macros have some additional powers that functions don’t.

A function signature must declare the number and type of parameters the function has. Macros, on the other hand, can take a variable number of parameters: we can call println!("hello") with one argument or println!("hello {}", name) with two arguments. Also, macros are expanded before the compiler interprets the meaning of the code, so a macro can, for example, implement a trait on a given type. A function can’t, because it gets called at runtime and a trait needs to be implemented at compile time.

The downside to implementing a macro instead of a function is that macro definitions are more complex than function definitions because you’re writing Rust code that writes Rust code. Due to this indirection, macro definitions are generally more difficult to read, understand, and maintain than function definitions.

Another important difference between macros and functions is that you must define macros or bring them into scope before you call them in a file, as opposed to functions you can define anywhere and call anywhere.

从根本上讲，宏是一种编写代码来编写其他代码的方法，这称为元编程。 在附录 C 中，我们讨论了导出属性，它为您生成各种特征的实现。 我们还使用了 println! 和向量！ 贯穿整本书的宏。 所有这些宏都会扩展以生成比您手动编写的代码更多的代码。

元编程对于减少必须编写和维护的代码量很有用，这也是函数的作用之一。 然而，宏具有一些函数所没有的额外功能。

函数签名必须声明函数具有的参数的数量和类型。 另一方面，宏可以采用可变数量的参数：我们可以使用一个参数调用 println!("hello") ，或者使用两个参数调用 println!("hello {}", name) 。 此外，宏在编译器解释代码含义之前会被扩展，因此宏可以在给定类型上实现特征。 函数不能，因为它在运行时被调用，并且特征需要在编译时实现。

实现宏而不是函数的缺点是宏定义比函数定义更复杂，因为您正在编写 Rust 代码，而 Rust 代码又在编写 Rust 代码。 由于这种间接性，宏定义通常比函数定义更难以阅读、理解和维护。

宏和函数之间的另一个重要区别是，在文件中调用宏之前，您必须定义宏或将它们纳入范围，这与您可以在任何地方定义和调用任何地方的函数相反。

### Declarative Macros with macro_rules! for General Metaprogramming
The most widely used form of macros in Rust is the declarative macro. These are also sometimes referred to as “macros by example,” “macro_rules! macros,” or just plain “macros.” At their core, declarative macros allow you to write something similar to a Rust match expression. As discussed in Chapter 6, match expressions are control structures that take an expression, compare the resulting value of the expression to patterns, and then run the code associated with the matching pattern. Macros also compare a value to patterns that are associated with particular code: in this situation, the value is the literal Rust source code passed to the macro; the patterns are compared with the structure of that source code; and the code associated with each pattern, when matched, replaces the code passed to the macro. This all happens during compilation.

To define a macro, you use the macro_rules! construct. Let’s explore how to use macro_rules! by looking at how the vec! macro is defined. Chapter 8 covered how we can use the vec! macro to create a new vector with particular values. For example, the following macro creates a new vector containing three integers:

Rust 中使用最广泛的宏形式是声明性宏。 这些有时也被称为“宏示例”、“宏规则！ 宏”，或者只是简单的“宏”。 从本质上讲，声明性宏允许您编写类似于 Rust 匹配表达式的内容。 正如第 6 章中所讨论的，匹配表达式是控制结构，它采用表达式，将表达式的结果值与模式进行比较，然后运行与匹配模式关联的代码。 宏还将值与与特定代码关联的模式进行比较：在这种情况下，该值是传递给宏的文字 Rust 源代码； 将模式与源代码的结构进行比较； 与每个模式关联的代码在匹配时将替换传递给宏的代码。 这一切都发生在编译期间。

要定义宏，请使用 Macro_rules！ 构造。 让我们探索一下如何使用macro_rules！ 通过查看vec如何！ 宏已定义。 第 8 章介绍了如何使用 vec！ 宏创建具有特定值的新向量。 例如，以下宏创建一个包含三个整数的新向量：

``````
let v: Vec<u32> = vec![1, 2, 3];
``````
We could also use the vec! macro to make a vector of two integers or a vector of five string slices. We wouldn’t be able to use a function to do the same because we wouldn’t know the number or type of values up front.

我们还可以使用 vec! 宏来创建两个整数的向量或五个字符串切片的向量。 我们无法使用函数来执行相同的操作，因为我们事先不知道值的数量或类型。

Listing 19-28 shows a slightly simplified definition of the vec! macro.

Filename: src/lib.rs
``````
#[macro_export]
macro_rules! vec {
    ( $( $x:expr ),* ) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
}
``````
Listing 19-28: A simplified version of the vec! macro definition


``````
Note: The actual definition of the vec! macro in the standard library includes code to preallocate the correct amount of memory up front. That code is an optimization that we don’t include here to make the example simpler.
``````

The #[macro_export] annotation indicates that this macro should be made available whenever the crate in which the macro is defined is brought into scope. Without this annotation, the macro can’t be brought into scope.

We then start the macro definition with macro_rules! and the name of the macro we’re defining without the exclamation mark. The name, in this case vec, is followed by curly brackets denoting the body of the macro definition.

The structure in the vec! body is similar to the structure of a match expression. Here we have one arm with the pattern ( $( $x:expr ),* ), followed by => and the block of code associated with this pattern. If the pattern matches, the associated block of code will be emitted. Given that this is the only pattern in this macro, there is only one valid way to match; any other pattern will result in an error. More complex macros will have more than one arm.

Valid pattern syntax in macro definitions is different than the pattern syntax covered in Chapter 18 because macro patterns are matched against Rust code structure rather than values. Let’s walk through what the pattern pieces in Listing 19-28 mean; for the full macro pattern syntax, see the Rust Reference.

First, we use a set of parentheses to encompass the whole pattern. We use a dollar sign ($) to declare a variable in the macro system that will contain the Rust code matching the pattern. The dollar sign makes it clear this is a macro variable as opposed to a regular Rust variable. Next comes a set of parentheses that captures values that match the pattern within the parentheses for use in the replacement code. Within $() is $x:expr, which matches any Rust expression and gives the expression the name $x.

The comma following $() indicates that a literal comma separator character could optionally appear after the code that matches the code in $(). The * specifies that the pattern matches zero or more of whatever precedes the *.

When we call this macro with vec![1, 2, 3];, the $x pattern matches three times with the three expressions 1, 2, and 3.

Now let’s look at the pattern in the body of the code associated with this arm: temp_vec.push() within $()* is generated for each part that matches $() in the pattern zero or more times depending on how many times the pattern matches. The $x is replaced with each expression matched. When we call this macro with vec![1, 2, 3];, the code generated that replaces this macro call will be the following:

#[macro_export] 注释指示每当定义该宏的包进入作用域时，该宏就应该可用。 如果没有这个注释，宏就无法进入作用域。

然后我们用macro_rules开始宏定义！ 以及我们定义的不带感叹号的宏的名称。 名称（在本例中为 vec）后跟表示宏定义主体的大括号。

vec! 中的结构 body 类似于匹配表达式的结构。 这里，我们有一个手臂，其模式为 ( $( $x:expr ),* )，后跟 => 以及与该模式相关的代码块。 如果模式匹配，则将发出关联的代码块。 鉴于这是该宏中的唯一模式，因此只有一种有效的匹配方式； 任何其他模式都会导致错误。 更复杂的宏将有多个臂。

宏定义中的有效模式语法与第 18 章中介绍的模式语法不同，因为宏模式与 Rust 代码结构而不是值进行匹配。 让我们看一下清单 19-28 中的模式片段的含义； 有关完整的宏模式语法，请参阅 Rust 参考。

首先，我们使用一组括号来包含整个模式。 我们使用美元符号 ($) 在宏系统中声明一个变量，该变量将包含与模式匹配的 Rust 代码。 美元符号清楚地表明这是一个宏变量，而不是常规的 Rust 变量。 接下来是一组括号，它们捕获与括号内的模式匹配的值，以便在替换代码中使用。 $() 中是 $x:expr，它匹配任何 Rust 表达式，并为表达式指定名称 $x。

$() 后面的逗号表示文字逗号分隔符可以选择出现在与 $() 中的代码匹配的代码之后。 * 指定模式与 * 之前的任何内容匹配零个或多个。

当我们使用 vec![1, 2, 3]; 调用该宏时，$x 模式与三个表达式 1、2 和 3 匹配三次。

现在让我们看看与该手臂相关的代码主体中的模式： $()* 中的 temp_vec.push() 是为与模式中的 $() 匹配零次或多次的每个部分生成的，具体取决于匹配的次数 模式匹配。 $x 被替换为每个匹配的表达式。 当我们使用 vec![1, 2, 3]; 调用该宏时，生成的替换该宏调用的代码将如下所示：

``````
{
    let mut temp_vec = Vec::new();
    temp_vec.push(1);
    temp_vec.push(2);
    temp_vec.push(3);
    temp_vec
}
``````
We’ve defined a macro that can take any number of arguments of any type and can generate code to create a vector containing the specified elements.

To learn more about how to write macros, consult the online documentation or other resources, such as “The Little Book of Rust Macros” started by Daniel Keep and continued by Lukas Wirth.

我们定义了一个宏，它可以接受任意数量、任意类型的参数，并且可以生成代码来创建包含指定元素的向量。

要了解有关如何编写宏的更多信息，请查阅在线文档或其他资源，例如由 Daniel Keep 撰写并由 Lukas Wirth 继续编写的“The Little Book of Rust Macros”。

### Procedural Macros for Generating Code from Attributes
The second form of macros is the procedural macro, which acts more like a function (and is a type of procedure). Procedural macros accept some code as an input, operate on that code, and produce some code as an output rather than matching against patterns and replacing the code with other code as declarative macros do. The three kinds of procedural macros are custom derive, attribute-like, and function-like, and all work in a similar fashion.

When creating procedural macros, the definitions must reside in their own crate with a special crate type. This is for complex technical reasons that we hope to eliminate in the future. In Listing 19-29, we show how to define a procedural macro, where some_attribute is a placeholder for using a specific macro variety.

宏的第二种形式是过程宏，它的作用更像是一个函数（并且是一种过程）。 过程宏接受一些代码作为输入，对该代码进行操作，并生成一些代码作为输出，而不是像声明性宏那样匹配模式并用其他代码替换代码。 这三种过程宏是自定义派生宏、类属性宏和类函数宏，并且都以类似的方式工作。

创建过程宏时，定义必须驻留在其自己的具有特殊 crate 类型的 crate 中。 这是出于复杂的技术原因，我们希望将来能够消除这些原因。 在清单 19-29 中，我们展示了如何定义过程宏，其中 some_attribute 是使用特定宏种类的占位符。

Filename: src/lib.rs
``````
use proc_macro;

#[some_attribute]
pub fn some_name(input: TokenStream) -> TokenStream {
}
``````
Listing 19-29: An example of defining a procedural macro

The function that defines a procedural macro takes a TokenStream as an input and produces a TokenStream as an output. The TokenStream type is defined by the proc_macro crate that is included with Rust and represents a sequence of tokens. This is the core of the macro: the source code that the macro is operating on makes up the input TokenStream, and the code the macro produces is the output TokenStream. The function also has an attribute attached to it that specifies which kind of procedural macro we’re creating. We can have multiple kinds of procedural macros in the same crate.

Let’s look at the different kinds of procedural macros. We’ll start with a custom derive macro and then explain the small dissimilarities that make the other forms different.

定义过程宏的函数将 TokenStream 作为输入并生成 TokenStream 作为输出。 TokenStream 类型由 Rust 附带的 proc_macro 箱定义，表示令牌序列。 这是宏的核心：宏正在操作的源代码构成了输入TokenStream，宏生成的代码是输出TokenStream。 该函数还附加了一个属性，用于指定我们正在创建哪种类型的过程宏。 我们可以在同一个 crate 中拥有多种类型的程序宏。

让我们看看不同类型的过程宏。 我们将从自定义派生宏开始，然后解释使其他形式不同的微小差异。

### How to Write a Custom derive Macro
Let’s create a crate named hello_macro that defines a trait named HelloMacro with one associated function named hello_macro. Rather than making our users implement the HelloMacro trait for each of their types, we’ll provide a procedural macro so users can annotate their type with #[derive(HelloMacro)] to get a default implementation of the hello_macro function. The default implementation will print Hello, Macro! My name is TypeName! where TypeName is the name of the type on which this trait has been defined. In other words, we’ll write a crate that enables another programmer to write code like Listing 19-30 using our crate.

让我们创建一个名为 hello_macro 的包，它定义了一个名为 HelloMacro 的特征以及一个名为 hello_macro 的关联函数。 我们不会让用户为每种类型实现 HelloMacro 特征，而是提供一个过程宏，以便用户可以使用 #[derive(HelloMacro)] 注释其类型，以获得 hello_macro 函数的默认实现。 默认实现将打印 Hello, Macro! 我的名字是类型名称！ 其中 TypeName 是定义此特征的类型的名称。 换句话说，我们将编写一个 crate，使其他程序员能够使用我们的 crate 编写如清单 19-30 所示的代码。

Filename: src/main.rs
``````
This code does not compile!
use hello_macro::HelloMacro;
use hello_macro_derive::HelloMacro;

#[derive(HelloMacro)]
struct Pancakes;

fn main() {
    Pancakes::hello_macro();
}
``````
Listing 19-30: The code a user of our crate will be able to write when using our procedural macro

This code will print Hello, Macro! My name is Pancakes! when we’re done. The first step is to make a new library crate, like this:
``````
$ cargo new hello_macro --lib
``````
Next, we’ll define the HelloMacro trait and its associated function:

Filename: src/lib.rs
``````
pub trait HelloMacro {
    fn hello_macro();
}
``````
We have a trait and its function. At this point, our crate user could implement the trait to achieve the desired functionality, like so:
``````
use hello_macro::HelloMacro;

struct Pancakes;

impl HelloMacro for Pancakes {
    fn hello_macro() {
        println!("Hello, Macro! My name is Pancakes!");
    }
}

fn main() {
    Pancakes::hello_macro();
}
``````
However, they would need to write the implementation block for each type they wanted to use with hello_macro; we want to spare them from having to do this work.

Additionally, we can’t yet provide the hello_macro function with default implementation that will print the name of the type the trait is implemented on: Rust doesn’t have reflection capabilities, so it can’t look up the type’s name at runtime. We need a macro to generate code at compile time.

The next step is to define the procedural macro. At the time of this writing, procedural macros need to be in their own crate. Eventually, this restriction might be lifted. The convention for structuring crates and macro crates is as follows: for a crate named foo, a custom derive procedural macro crate is called foo_derive. Let’s start a new crate called hello_macro_derive inside our hello_macro project:

然而，他们需要为他们想要与 hello_macro 一起使用的每种类型编写实现块； 我们想让他们不必做这项工作。

此外，我们还无法为 hello_macro 函数提供默认实现，该函数将打印实现该特征的类型的名称：Rust 没有反射功能，因此它无法在运行时查找类型的名称。 我们需要一个宏来在编译时生成代码。

下一步是定义程序宏。 在撰写本文时，程序宏需要位于其自己的包中。 最终，这一限制可能会被取消。 构造 crate 和宏 crate 的约定如下：对于名为 foo 的 crate，自定义派生过程宏 crate 称为 foo_derive。 让我们在 hello_macro 项目中启动一个名为 hello_macro_derive 的新包：

``````
$ cargo new hello_macro_derive --lib
``````
Our two crates are tightly related, so we create the procedural macro crate within the directory of our hello_macro crate. If we change the trait definition in hello_macro, we’ll have to change the implementation of the procedural macro in hello_macro_derive as well. The two crates will need to be published separately, and programmers using these crates will need to add both as dependencies and bring them both into scope. We could instead have the hello_macro crate use hello_macro_derive as a dependency and re-export the procedural macro code. However, the way we’ve structured the project makes it possible for programmers to use hello_macro even if they don’t want the derive functionality.

We need to declare the hello_macro_derive crate as a procedural macro crate. We’ll also need functionality from the syn and quote crates, as you’ll see in a moment, so we need to add them as dependencies. Add the following to the Cargo.toml file for hello_macro_derive:

我们的两个 crate 紧密相关，因此我们在 hello_macro crate 的目录中创建程序宏 crate。 如果我们更改 hello_macro 中的特征定义，我们也必须更改 hello_macro_derive 中过程宏的实现。 这两个包需要单独发布，使用这些包的程序员需要将它们添加为依赖项并将它们纳入范围。 我们可以让 hello_macro 箱使用 hello_macro_derive 作为依赖项并重新导出过程宏代码。 然而，我们构建项目的方式使程序员可以使用 hello_macro，即使他们不想要派生功能。

我们需要将 hello_macro_derive 箱声明为程序宏箱。 我们还需要 syn 和 quote 包的功能，稍后您就会看到，因此我们需要将它们添加为依赖项。 将以下内容添加到 hello_macro_derive 的 Cargo.toml 文件中：

Filename: hello_macro_derive/Cargo.toml
``````
[lib]
proc-macro = true

[dependencies]
syn = "1.0"
quote = "1.0"
``````
To start defining the procedural macro, place the code in Listing 19-31 into your src/lib.rs file for the hello_macro_derive crate. Note that this code won’t compile until we add a definition for the impl_hello_macro function.

要开始定义过程宏，请将清单 19-31 中的代码放入 hello_macro_derive crate 的 src/lib.rs 文件中。 请注意，在我们添加 impl_hello_macro 函数的定义之前，此代码不会编译。

Filename: hello_macro_derive/src/lib.rs
``````
This code does not compile!
use proc_macro::TokenStream;
use quote::quote;
use syn;

#[proc_macro_derive(HelloMacro)]
pub fn hello_macro_derive(input: TokenStream) -> TokenStream {
    // Construct a representation of Rust code as a syntax tree
    // that we can manipulate
    let ast = syn::parse(input).unwrap();

    // Build the trait implementation
    impl_hello_macro(&ast)
}
``````
Listing 19-31: Code that most procedural macro crates will require in order to process Rust code

Notice that we’ve split the code into the hello_macro_derive function, which is responsible for parsing the TokenStream, and the impl_hello_macro function, which is responsible for transforming the syntax tree: this makes writing a procedural macro more convenient. The code in the outer function (hello_macro_derive in this case) will be the same for almost every procedural macro crate you see or create. The code you specify in the body of the inner function (impl_hello_macro in this case) will be different depending on your procedural macro’s purpose.

We’ve introduced three new crates: proc_macro, syn, and quote. The proc_macro crate comes with Rust, so we didn’t need to add that to the dependencies in Cargo.toml. The proc_macro crate is the compiler’s API that allows us to read and manipulate Rust code from our code.

The syn crate parses Rust code from a string into a data structure that we can perform operations on. The quote crate turns syn data structures back into Rust code. These crates make it much simpler to parse any sort of Rust code we might want to handle: writing a full parser for Rust code is no simple task.

The hello_macro_derive function will be called when a user of our library specifies #[derive(HelloMacro)] on a type. This is possible because we’ve annotated the hello_macro_derive function here with proc_macro_derive and specified the name HelloMacro, which matches our trait name; this is the convention most procedural macros follow.

The hello_macro_derive function first converts the input from a TokenStream to a data structure that we can then interpret and perform operations on. This is where syn comes into play. The parse function in syn takes a TokenStream and returns a DeriveInput struct representing the parsed Rust code. Listing 19-32 shows the relevant parts of the DeriveInput struct we get from parsing the struct Pancakes; string:

请注意，我们将代码分为 hello_macro_derive 函数（负责解析 TokenStream）和 impl_hello_macro 函数（负责转换语法树）：这使得编写过程宏更加方便。 对于您看到或创建的几乎每个过程宏包，外部函数（在本例中为 hello_macro_derive）中的代码都是相同的。 您在内部函数主体中指定的代码（本例中为 impl_hello_macro）将根据过程宏的用途而有所不同。

我们引入了三个新的 crate：proc_macro、syn 和 quote。 proc_macro crate 附带 Rust，因此我们不需要将其添加到 Cargo.toml 中的依赖项中。 proc_macro crate 是编译器的 API，允许我们从代码中读取和操作 Rust 代码。

Syn crate 将 Rust 代码从字符串解析为我们可以执行操作的数据结构。 quote crate 将 syn 数据结构重新转换为 Rust 代码。 这些板条箱使解析我们可能想要处理的任何类型的 Rust 代码变得更加简单：为 Rust 代码编写完整的解析器并不是一件简单的任务。

当我们库的用户在类型上指定 #[derive(HelloMacro)] 时，将调用 hello_macro_derive 函数。 这是可能的，因为我们在这里用 proc_macro_derive 注释了 hello_macro_derive 函数，并指定了名称 HelloMacro，它与我们的特征名称相匹配； 这是大多数程序宏遵循的约定。

hello_macro_derive 函数首先将输入从 TokenStream 转换为我们可以解释和执行操作的数据结构。 这就是 syn 发挥作用的地方。 syn 中的解析函数采用 TokenStream 并返回表示解析后的 Rust 代码的 DeriveInput 结构。 清单 19-32 显示了我们通过解析 struct Pancakes 得到的 DeriveInput 结构体的相关部分； string：

``````
DeriveInput {
    // --snip--

    ident: Ident {
        ident: "Pancakes",
        span: #0 bytes(95..103)
    },
    data: Struct(
        DataStruct {
            struct_token: Struct,
            fields: Unit,
            semi_token: Some(
                Semi
            )
        }
    )
}
``````
Listing 19-32: The DeriveInput instance we get when parsing the code that has the macro’s attribute in Listing 19-30

The fields of this struct show that the Rust code we’ve parsed is a unit struct with the ident (identifier, meaning the name) of Pancakes. There are more fields on this struct for describing all sorts of Rust code; check the syn documentation for DeriveInput for more information.

Soon we’ll define the impl_hello_macro function, which is where we’ll build the new Rust code we want to include. But before we do, note that the output for our derive macro is also a TokenStream. The returned TokenStream is added to the code that our crate users write, so when they compile their crate, they’ll get the extra functionality that we provide in the modified TokenStream.

You might have noticed that we’re calling unwrap to cause the hello_macro_derive function to panic if the call to the syn::parse function fails here. It’s necessary for our procedural macro to panic on errors because proc_macro_derive functions must return TokenStream rather than Result to conform to the procedural macro API. We’ve simplified this example by using unwrap; in production code, you should provide more specific error messages about what went wrong by using panic! or expect.

Now that we have the code to turn the annotated Rust code from a TokenStream into a DeriveInput instance, let’s generate the code that implements the HelloMacro trait on the annotated type, as shown in Listing 19-33.

这个结构体的字段表明我们解析的Rust代码是一个单位结构体，其ident（标识符，意思是名字）是Pancakes。 该结构体上还有更多字段用于描述各种 Rust 代码； 检查 DeriveInput 的 syn 文档以获取更多信息。

很快我们将定义 impl_hello_macro 函数，我们将在其中构建我们想要包含的新 Rust 代码。 但在此之前，请注意派生宏的输出也是一个 TokenStream。 返回的 TokenStream 被添加到我们的 crate 用户编写的代码中，因此当他们编译他们的 crate 时，他们将获得我们在修改后的 TokenStream 中提供的额外功能。

您可能已经注意到，如果对 syn::parse 函数的调用在此失败，我们调用 unwrap 会导致 hello_macro_derive 函数发生恐慌。 我们的过程宏有必要对错误进行恐慌，因为 proc_macro_derive 函数必须返回 TokenStream 而不是 Result 以符合过程宏 API。 我们通过使用 unwrap 简化了这个例子； 在生产代码中，您应该通过使用恐慌来提供有关出错原因的更具体的错误消息！ 或期待。

现在我们有了将带注释的 Rust 代码从 TokenStream 转换为 DeriveInput 实例的代码，让我们生成在带注释的类型上实现 HelloMacro 特征的代码，如清单 19-33 所示。

Filename: hello_macro_derive/src/lib.rs
``````
fn impl_hello_macro(ast: &syn::DeriveInput) -> TokenStream {
    let name = &ast.ident;
    let gen = quote! {
        impl HelloMacro for #name {
            fn hello_macro() {
                println!("Hello, Macro! My name is {}!", stringify!(#name));
            }
        }
    };
    gen.into()
}
``````
Listing 19-33: Implementing the HelloMacro trait using the parsed Rust code

We get an Ident struct instance containing the name (identifier) of the annotated type using ast.ident. The struct in Listing 19-32 shows that when we run the impl_hello_macro function on the code in Listing 19-30, the ident we get will have the ident field with a value of "Pancakes". Thus, the name variable in Listing 19-33 will contain an Ident struct instance that, when printed, will be the string "Pancakes", the name of the struct in Listing 19-30.

The quote! macro lets us define the Rust code that we want to return. The compiler expects something different to the direct result of the quote! macro’s execution, so we need to convert it to a TokenStream. We do this by calling the into method, which consumes this intermediate representation and returns a value of the required TokenStream type.

The quote! macro also provides some very cool templating mechanics: we can enter #name, and quote! will replace it with the value in the variable name. You can even do some repetition similar to the way regular macros work. Check out the quote crate’s docs for a thorough introduction.

We want our procedural macro to generate an implementation of our HelloMacro trait for the type the user annotated, which we can get by using #name. The trait implementation has the one function hello_macro, whose body contains the functionality we want to provide: printing Hello, Macro! My name is and then the name of the annotated type.

The stringify! macro used here is built into Rust. It takes a Rust expression, such as 1 + 2, and at compile time turns the expression into a string literal, such as "1 + 2". This is different than format! or println!, macros which evaluate the expression and then turn the result into a String. There is a possibility that the #name input might be an expression to print literally, so we use stringify!. Using stringify! also saves an allocation by converting #name to a string literal at compile time.

At this point, cargo build should complete successfully in both hello_macro and hello_macro_derive. Let’s hook up these crates to the code in Listing 19-30 to see the procedural macro in action! Create a new binary project in your projects directory using cargo new pancakes. We need to add hello_macro and hello_macro_derive as dependencies in the pancakes crate’s Cargo.toml. If you’re publishing your versions of hello_macro and hello_macro_derive to crates.io, they would be regular dependencies; if not, you can specify them as path dependencies as follows:

我们使用 ast.ident 获得一个 Ident 结构体实例，其中包含带注释的类型的名称（标识符）。 清单19-32中的结构表明，当我们对清单19-30中的代码运行impl_hello_macro函数时，我们得到的ident将具有值为“Pancakes”的ident字段。 因此，清单 19-33 中的 name 变量将包含一个 Ident 结构体实例，打印时该实例将是字符串“Pancakes”，即清单 19-30 中的结构体名称。

报价单！ 宏让我们定义要返回的 Rust 代码。 编译器期望的结果与引用的直接结果不同！ 宏的执行，所以我们需要将其转换为TokenStream。 我们通过调用 into 方法来完成此操作，该方法使用此中间表示并返回所需的 TokenStream 类型的值。

报价单！ 宏还提供了一些非常酷的模板机制：我们可以输入#name，然后引用！ 会将其替换为变量名称中的值。 您甚至可以像常规宏的工作方式一样进行一些重复。 查看 quote crate 的文档以获取完整的介绍。

我们希望我们的程序宏为用户注释的类型生成 HelloMacro 特征的实现，我们可以通过使用 #name 来获取它。 Trait 实现有一个函数 hello_macro，其主体包含我们想要提供的功能：打印 Hello，Macro！ 我的名字是，然后是注释类型的名称。

串起来！ 这里使用的宏内置于 Rust 中。 它采用 Rust 表达式，例如 1 + 2，并在编译时将表达式转换为字符串文字，例如“1 + 2”。 这与格式不同！ 或 println!，计算表达式然后将结果转换为字符串的宏。 #name 输入可能是一个按字面打印的表达式，因此我们使用 stringify!。 使用字符串化！ 还通过在编译时将 #name 转换为字符串文字来保存分配。

此时，cargo 构建应该在 hello_macro 和 hello_macro_derive 中成功完成。 让我们将这些 crate 连接到清单 19-30 中的代码来查看过程宏的运行情况！ 使用cargo new pancakes 在项目目录中创建一个新的二进制项目。 我们需要在 pancakes crate 的 Cargo.toml 中添加 hello_macro 和 hello_macro_derive 作为依赖项。 如果您将 hello_macro 和 hello_macro_derive 版本发布到 crates.io，它们将是常规依赖项； 如果没有，您可以将它们指定为路径依赖项，如下所示：

``````
hello_macro = { path = "../hello_macro" }
hello_macro_derive = { path = "../hello_macro/hello_macro_derive" }
``````
Put the code in Listing 19-30 into src/main.rs, and run cargo run: it should print Hello, Macro! My name is Pancakes! The implementation of the HelloMacro trait from the procedural macro was included without the pancakes crate needing to implement it; the #[derive(HelloMacro)] added the trait implementation.

将清单 19-30 中的代码放入 src/main.rs 中，然后运行 Cargo run: 它应该打印 Hello, Macro! 我的名字是煎饼！ 程序宏中的 HelloMacro 特征的实现被包含在内，而 pancakes crate 不需要实现它； #[derive(HelloMacro)] 添加了特征实现。

Next, let’s explore how the other kinds of procedural macros differ from custom derive macros.

### Attribute-like macros
Attribute-like macros are similar to custom derive macros, but instead of generating code for the derive attribute, they allow you to create new attributes. They’re also more flexible: derive only works for structs and enums; attributes can be applied to other items as well, such as functions. Here’s an example of using an attribute-like macro: say you have an attribute named route that annotates functions when using a web application framework:

类属性宏与自定义派生宏类似，但它们不是为派生属性生成代码，而是允许您创建新属性。 它们也更灵活：derive 仅适用于结构和枚举； 属性也可以应用于其他项目，例如函数。 下面是使用类似属性的宏的示例：假设您有一个名为 Route 的属性，该属性在使用 Web 应用程序框架时注释函数：

``````
#[route(GET, "/")]
fn index() {
``````
This #[route] attribute would be defined by the framework as a procedural macro. The signature of the macro definition function would look like this:
``````
#[proc_macro_attribute]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {
``````
Here, we have two parameters of type TokenStream. The first is for the contents of the attribute: the GET, "/" part. The second is the body of the item the attribute is attached to: in this case, fn index() {} and the rest of the function’s body.

Other than that, attribute-like macros work the same way as custom derive macros: you create a crate with the proc-macro crate type and implement a function that generates the code you want!

这里，我们有两个 TokenStream 类型的参数。 第一个是属性的内容：GET、“/”部分。 第二个是属性附加到的项目的主体：在本例中是 fn index() {} 和函数主体的其余部分。

除此之外，类属性宏的工作方式与自定义派生宏相同：您使用 proc-macro crate 类型创建一个 crate，并实现一个生成所需代码的函数！

### Function-like macros
Function-like macros define macros that look like function calls. Similarly to macro_rules! macros, they’re more flexible than functions; for example, they can take an unknown number of arguments. However, macro_rules! macros can be defined only using the match-like syntax we discussed in the section “Declarative Macros with macro_rules! for General Metaprogramming” earlier. Function-like macros take a TokenStream parameter and their definition manipulates that TokenStream using Rust code as the other two types of procedural macros do. An example of a function-like macro is an sql! macro that might be called like so:

类函数宏定义看起来像函数调用的宏。 与macro_rules类似！ 宏，它们比函数更灵活； 例如，它们可以接受未知数量的参数。 然而，宏规则！ 宏只能使用我们在“带有 Macro_rules 的声明性宏！”一节中讨论的类似匹配的语法来定义。 通用元编程”之前。 类似函数的宏采用 TokenStream 参数，并且它们的定义使用 Rust 代码操作该 TokenStream，就像其他两种类型的过程宏一样。 类函数宏的一个例子是 sql! 宏可以这样调用：

``````
let sql = sql!(SELECT * FROM posts WHERE id=1);
``````
This macro would parse the SQL statement inside it and check that it’s syntactically correct, which is much more complex processing than a macro_rules! macro can do. The sql! macro would be defined like this:

这个宏会解析其中的 SQL 语句并检查其语法是否正确，这比 Macro_rules 的处理要复杂得多！ 宏可以做。 sql！ 宏定义如下：

``````
#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
``````
This definition is similar to the custom derive macro’s signature: we receive the tokens that are inside the parentheses and return the code we wanted to generate.

此定义类似于自定义派生宏的签名：我们接收括号内的标记并返回我们想要生成的代码。

## 19.6 Summary
Whew! Now you have some Rust features in your toolbox that you likely won’t use often, but you’ll know they’re available in very particular circumstances. We’ve introduced several complex topics so that when you encounter them in error message suggestions or in other peoples’ code, you’ll be able to recognize these concepts and syntax. Use this chapter as a reference to guide you to solutions.

哇！ 现在，您的工具箱中有一些您可能不会经常使用的 Rust 功能，但您会知道它们在非常特殊的情况下可用。 我们介绍了几个复杂的主题，以便当您在错误消息建议或其他人的代码中遇到它们时，您将能够识别这些概念和语法。 使用本章作为参考来指导您找到解决方案。

Next, we’ll put everything we’ve discussed throughout the book into practice and do one more project!