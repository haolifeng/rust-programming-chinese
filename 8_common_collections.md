# 8. 通用集合
Rust’s standard library includes a number of very useful data structures called collections. Most other data types represent one specific value, but collections can contain multiple values. Unlike the built-in array and tuple types, the data these collections point to is stored on the heap, which means the amount of data does not need to be known at compile time and can grow or shrink as the program runs. Each kind of collection has different capabilities and costs, and choosing an appropriate one for your current situation is a skill you’ll develop over time. In this chapter, we’ll discuss three collections that are used very often in Rust programs:

+ A vector allows you to store a variable number of values next to each other.
+ A string is a collection of characters. We’ve mentioned the String type previously, but in this chapter we’ll talk about it in depth.
+ A hash map allows you to associate a value with a particular key. It’s a particular implementation of the more general data structure called a map.
To learn about the other kinds of collections provided by the standard library, see the documentation.

We’ll discuss how to create and update vectors, strings, and hash maps, as well as what makes each special.

Rust 的标准库包含许多非常有用的数据结构，称为集合。 大多数其他数据类型表示一个特定值，但集合可以包含多个值。 与内置数组和元组类型不同，这些集合指向的数据存储在堆上，这意味着数据量不需要在编译时知道，并且可以随着程序运行而增长或收缩。 每种类型的集合都有不同的功能和成本，选择适合您当前情况的集合是一项您将随着时间的推移而发展的技能。 在本章中，我们将讨论 Rust 程序中经常使用的三个集合：

+ 向量允许您存储彼此相邻的可变数量的值。
+ 字符串是字符的集合。 我们之前已经提到过 String 类型，但在本章中我们将深入讨论它。
+ 哈希映射允许您将值与特定键相关联。 它是称为映射的更通用数据结构的特定实现。

要了解标准库提供的其他类型的集合，请参阅文档。

我们将讨论如何创建和更新向量、字符串和哈希映射，以及它们的特殊之处。

## 8.1 Storing Lists of Values with Vectors
The first collection type we’ll look at is Vec<T>, also known as a vector. Vectors allow you to store more than one value in a single data structure that puts all the values next to each other in memory. Vectors can only store values of the same type. They are useful when you have a list of items, such as the lines of text in a file or the prices of items in a shopping cart.

我们要看到的第一个集合类型是 Vec<T>，也称为向量。 向量允许您在单个数据结构中存储多个值，该数据结构将所有值在内存中彼此相邻。 向量只能存储相同类型的值。 当您有一个项目列表（例如文件中的文本行或购物车中的项目价格）时，它们非常有用。


### 创建新的Vector(Creating a New Vector)
To create a new empty vector, we call the Vec::new function, as shown in Listing 8-1.

为了创建一个新的空向量，我们调用 Vec::new 函数，如清单 8-1 所示。
```
fn main() {
    let v: Vec<i32> = Vec::new();
}
```
Listing 8-1: Creating a new, empty vector to hold values of type i32

Note that we added a type annotation here. Because we aren’t inserting any values into this vector, Rust doesn’t know what kind of elements we intend to store. This is an important point. Vectors are implemented using generics; we’ll cover how to use generics with your own types in Chapter 10. For now, know that the Vec<T> type provided by the standard library can hold any type. When we create a vector to hold a specific type, we can specify the type within angle brackets. In Listing 8-1, we’ve told Rust that the Vec<T> in v will hold elements of the i32 type.

More often, you’ll create a Vec<T> with initial values and Rust will infer the type of value you want to store, so you rarely need to do this type annotation. Rust conveniently provides the vec! macro, which will create a new vector that holds the values you give it. Listing 8-2 creates a new Vec<i32> that holds the values 1, 2, and 3. The integer type is i32 because that’s the default integer type, as we discussed in the “Data Types” section of Chapter 3.

示例 8-1：创建一个新的空向量来保存 i32 类型的值

请注意，我们在这里添加了类型注释。 因为我们没有向这个向量插入任何值，Rust 不知道我们打算存储什么类型的元素。 这是很重要的一点。 向量是使用泛型实现的； 我们将在第 10 章中介绍如何将泛型与您自己的类型一起使用。现在，请知道标准库提供的 Vec<T> 类型可以保存任何类型。 当我们创建一个向量来保存特定类型时，我们可以在尖括号内指定类型。 在清单 8-1 中，我们告诉 Rust v 中的 Vec<T> 将保存 i32 类型的元素。

更常见的是，您将创建一个带有初始值的 Vec<T>，Rust 将推断您想要存储的值的类型，因此您很少需要执行此类型注释。 Rust 方便地提供了 vec！ 宏，它将创建一个新的向量来保存您给它的值。 清单 8-2 创建了一个新的 Vec<i32>，它保存值 1、2 和 3。整数类型是 i32，因为这是默认的整数类型，正如我们在第 3 章的“数据类型”部分中讨论的那样。

```
fn main() {
    let v = vec![1, 2, 3];
}
```

示例 8-2：创建一个包含值的新向量

Because we’ve given initial i32 values, Rust can infer that the type of v is Vec<i32>, and the type annotation isn’t necessary. Next, we’ll look at how to modify a vector.


因为我们已经给出了初始 i32 值，Rust 可以推断出 v 的类型是 Vec<i32>，并且类型注释是不必要的。 接下来，我们将了解如何修改向量。

### 更新(updating a Vector)
To create a vector and then add elements to it, we can use the push method, as shown in Listing 8-3.
```
fn main() {
    let mut v = Vec::new();

    v.push(5);
    v.push(6);
    v.push(7);
    v.push(8);
}
```

示例 8-3：使用 push 方法向向量添加值

As with any variable, if we want to be able to change its value, we need to make it mutable using the mut keyword, as discussed in Chapter 3. The numbers we place inside are all of type i32, and Rust infers this from the data, so we don’t need the Vec<i32> annotation.



与任何变量一样，如果我们希望能够更改其值，我们需要使用 mut 关键字使其可变，如第 3 章中所述。我们放置在其中的数字都是 i32 类型，Rust 从 数据，所以我们不需要 Vec<i32> 注释。

### Reading Elements of Vectors

There are two ways to reference a value stored in a vector: via indexing or using the get method. In the following examples, we’ve annotated the types of the values that are returned from these functions for extra clarity.

Listing 8-4 shows both methods of accessing a value in a vector, with indexing syntax and the get method.


有两种方法可以引用存储在向量中的值：通过索引或使用 get 方法。 在以下示例中，为了更加清晰起见，我们注释了从这些函数返回的值的类型。

清单 8-4 显示了访问向量中的值的两种方法，即索引语法和 get 方法。

```
fn main() {
    let v = vec![1, 2, 3, 4, 5];

    let third: &i32 = &v[2];
    println!("The third element is {third}");

    let third: Option<&i32> = v.get(2);
    match third {
        Some(third) => println!("The third element is {third}"),
        None => println!("There is no third element."),
    }
}
```

Listing 8-4: Using indexing syntax or the get method to access an item in a vector

Note a few details here. We use the index value of 2 to get the third element because vectors are indexed by number, starting at zero. Using & and [] gives us a reference to the element at the index value. When we use the get method with the index passed as an argument, we get an Option<&T> that we can use with match.

The reason Rust provides these two ways to reference an element is so you can choose how the program behaves when you try to use an index value outside the range of existing elements. As an example, let’s see what happens when we have a vector of five elements and then we try to access an element at index 100 with each technique, as shown in Listing 8-5.

示例 8-4：使用索引语法或 get 方法访问向量中的项目

请注意此处的一些细节。 我们使用索引值 2 来获取第三个元素，因为向量是按数字索引的，从零开始。 使用 & 和 [] 为我们提供了对索引值处的元素的引用。 当我们使用 get 方法并将索引作为参数传递时，我们会得到一个 Option<&T> ，我们可以将其与 match 一起使用。

Rust 提供这两种方法来引用元素的原因是，当您尝试使用现有元素范围之外的索引值时，您可以选择程序的行为方式。 作为一个例子，让我们看看当我们有一个包含五个元素的向量，然后我们尝试使用每种技术访问索引 100 处的元素时会发生什么，如清单 8-5 所示。

```
fn main() {
    let v = vec![1, 2, 3, 4, 5];

    let does_not_exist = &v[100];
    let does_not_exist = v.get(100);
}
```
Listing 8-5: Attempting to access the element at index 100 in a vector containing five elements

When we run this code, the first [] method will cause the program to panic because it references a nonexistent element. This method is best used when you want your program to crash if there’s an attempt to access an element past the end of the vector.

When the get method is passed an index that is outside the vector, it returns None without panicking. You would use this method if accessing an element beyond the range of the vector may happen occasionally under normal circumstances. Your code will then have logic to handle having either Some(&element) or None, as discussed in Chapter 6. For example, the index could be coming from a person entering a number. If they accidentally enter a number that’s too large and the program gets a None value, you could tell the user how many items are in the current vector and give them another chance to enter a valid value. That would be more user-friendly than crashing the program due to a typo!

When the program has a valid reference, the borrow checker enforces the ownership and borrowing rules (covered in Chapter 4) to ensure this reference and any other references to the contents of the vector remain valid. Recall the rule that states you can’t have mutable and immutable references in the same scope. That rule applies in Listing 8-6, where we hold an immutable reference to the first element in a vector and try to add an element to the end. This program won’t work if we also try to refer to that element later in the function:

示例 8-5：尝试访问包含五个元素的向量中索引 100 处的元素

当我们运行这段代码时，第一个 [] 方法将导致程序崩溃，因为它引用了一个不存在的元素。 当您希望程序在尝试访问超出向量末尾的元素时崩溃时，最好使用此方法。

当 get 方法传递向量外部的索引时，它会返回 None 而不会出现恐慌。 如果正常情况下偶尔会访问超出向量范围的元素，则可以使用此方法。 然后，您的代码将具有处理 Some(&element) 或 None 的逻辑，如第 6 章中所述。例如，索引可能来自输入数字的人。 如果他们不小心输入了太大的数字并且程序得到了 None 值，您可以告诉用户当前向量中有多少项，并给他们另一次输入有效值的机会。 这比由于拼写错误而导致程序崩溃更加用户友好！

当程序具有有效引用时，借用检查器会强制执行所有权和借用规则（第 4 章中介绍），以确保该引用以及对向量内容的任何其他引用保持有效。 回想一下规则，该规则规定同一范围内不能有可变引用和不可变引用。 该规则适用于清单 8-6，其中我们保存对向量中第一个元素的不可变引用，并尝试将一个元素添加到末尾。 如果我们稍后还尝试在函数中引用该元素，则该程序将无法工作：

```
fn main() {
    let mut v = vec![1, 2, 3, 4, 5];

    let first = &v[0];

    v.push(6);

    println!("The first element is: {first}");
}
```
Listing 8-6: Attempting to add an element to a vector while holding a reference to an item

Compiling this code will result in this error:

```
$ cargo run
   Compiling collections v0.1.0 (file:///projects/collections)
error[E0502]: cannot borrow `v` as mutable because it is also borrowed as immutable
 --> src/main.rs:6:5
  |
4 |     let first = &v[0];
  |                  - immutable borrow occurs here
5 |
6 |     v.push(6);
  |     ^^^^^^^^^ mutable borrow occurs here
7 |
8 |     println!("The first element is: {first}");
  |                                      ----- immutable borrow later used here

For more information about this error, try `rustc --explain E0502`.
error: could not compile `collections` due to previous error

```
The code in Listing 8-6 might look like it should work: why should a reference to the first element care about changes at the end of the vector? This error is due to the way vectors work: because vectors put the values next to each other in memory, adding a new element onto the end of the vector might require allocating new memory and copying the old elements to the new space, if there isn’t enough room to put all the elements next to each other where the vector is currently stored. In that case, the reference to the first element would be pointing to deallocated memory. The borrowing rules prevent programs from ending up in that situation.

Note: For more on the implementation details of the Vec<T> type, see “The Rustonomicon”.

清单 8-6 中的代码可能看起来应该可以工作：为什么对第一个元素的引用应该关心向量末尾的变化？ 此错误是由于向量的工作方式造成的：因为向量将值放在内存中彼此相邻的位置，因此在向量末尾添加新元素可能需要分配新内存并将旧元素复制到新空间（如果有） 没有足够的空间将当前存储向量的所有元素彼此相邻。 在这种情况下，对第一个元素的引用将指向已释放的内存。 借用规则可以防止程序陷入这种情况。

注意：有关 Vec<T> 类型的实现细节的更多信息，请参阅“The Rustonomicon”。

### 迭代遍历(Iterating over the Values in a Vector)
To access each element in a vector in turn, we would iterate through all of the elements rather than use indices to access one at a time. Listing 8-7 shows how to use a for loop to get immutable references to each element in a vector of i32 values and print them.

为了依次访问向量中的每个元素，我们将迭代所有元素，而不是使用索引一次访问一个元素。 清单 8-7 展示了如何使用 for 循环来获取对 i32 值向量中每个元素的不可变引用并打印它们。

```
fn main() {
    let v = vec![100, 32, 57];
    for i in &v {
        println!("{i}");
    }
}
```
Listing 8-7: Printing each element in a vector by iterating over the elements using a for loop
示例 8-7：通过使用 for 循环迭代元素来打印向量中的每个元素

We can also iterate over mutable references to each element in a mutable vector in order to make changes to all the elements. The for loop in Listing 8-8 will add 50 to each element.



我们还可以迭代对可变向量中每个元素的可变引用，以便对所有元素进行更改。 清单 8-8 中的 for 循环将为每个元素添加 50。
```
fn main() {
    let mut v = vec![100, 32, 57];
    for i in &mut v {
        *i += 50;
    }
}
```
Listing 8-8: Iterating over mutable references to elements in a vector

To change the value that the mutable reference refers to, we have to use the * dereference operator to get to the value in i before we can use the += operator. We’ll talk more about the dereference operator in the “Following the Pointer to the Value with the Dereference Operator” section of Chapter 15.

Iterating over a vector, whether immutably or mutably, is safe because of the borrow checker's rules. If we attempted to insert or remove items in the for loop bodies in Listing 8-7 and Listing 8-8, we would get a compiler error similar to the one we got with the code in Listing 8-6. The reference to the vector that the for loop holds prevents simultaneous modification of the whole vector.

示例 8-8：迭代向量中元素的可变引用

要更改可变引用引用的值，我们必须先使用 * 取消引用运算符来获取 i 中的值，然后才能使用 += 运算符。 我们将在第 15 章的“使用解引用运算符跟随指向值的指针”部分详细讨论解引用运算符。

由于借用检查器的规则，无论是不可变的还是可变的，迭代向量都是安全的。 如果我们尝试在清单 8-7 和清单 8-8 中的 for 循环体中插入或删除项目，我们将得到一个与清单 8-6 中的代码类似的编译器错误。 对 for 循环保存的向量的引用可以防止同时修改整个向量。

### Using an Enum to Store Multiple Types

Vectors can only store values that are the same type. This can be inconvenient; there are definitely use cases for needing to store a list of items of different types. Fortunately, the variants of an enum are defined under the same enum type, so when we need one type to represent elements of different types, we can define and use an enum!

For example, say we want to get values from a row in a spreadsheet in which some of the columns in the row contain integers, some floating-point numbers, and some strings. We can define an enum whose variants will hold the different value types, and all the enum variants will be considered the same type: that of the enum. Then we can create a vector to hold that enum and so, ultimately, holds different types. We’ve demonstrated this in Listing 8-9.

向量只能存储相同类型的值。 这可能会带来不便； 肯定存在需要存储不同类型的项目列表的用例。 幸运的是，枚举的变体是在同一枚举类型下定义的，因此当我们需要一种类型来表示不同类型的元素时，我们可以定义并使用枚举！

例如，假设我们想要从电子表格中的一行获取值，其中该行中的某些列包含整数、一些浮点数和一些字符串。 我们可以定义一个枚举，其变体将保存不同的值类型，并且所有枚举变体将被视为同一类型：枚举的类型。 然后我们可以创建一个向量来保存该枚举，因此最终保存不同的类型。 我们在清单 8-9 中演示了这一点。

```
fn main() {
    enum SpreadsheetCell {
        Int(i32),
        Float(f64),
        Text(String),
    }

    let row = vec![
        SpreadsheetCell::Int(3),
        SpreadsheetCell::Text(String::from("blue")),
        SpreadsheetCell::Float(10.12),
    ];
}
```
Listing 8-9: Defining an enum to store values of different types in one vector

Rust needs to know what types will be in the vector at compile time so it knows exactly how much memory on the heap will be needed to store each element. We must also be explicit about what types are allowed in this vector. If Rust allowed a vector to hold any type, there would be a chance that one or more of the types would cause errors with the operations performed on the elements of the vector. Using an enum plus a match expression means that Rust will ensure at compile time that every possible case is handled, as discussed in Chapter 6.

If you don’t know the exhaustive set of types a program will get at runtime to store in a vector, the enum technique won’t work. Instead, you can use a trait object, which we’ll cover in Chapter 17.

Now that we’ve discussed some of the most common ways to use vectors, be sure to review the API documentation for all the many useful methods defined on Vec<T> by the standard library. For example, in addition to push, a pop method removes and returns the last element.

示例 8-9：定义一个枚举以在一个向量中存储不同类型的值

Rust 需要在编译时知道向量中的类型，以便准确地知道堆上需要多少内存来存储每个元素。 我们还必须明确该向量中允许的类型。 如果 Rust 允许向量保存任何类型，则一种或多种类型可能会导致对向量元素执行的操作出错。 使用枚举加上匹配表达式意味着 Rust 将确保在编译时处理每种可能的情况，如第 6 章所述。

如果您不知道程序在运行时将获得并存储在向量中的详尽类型集，则枚举技术将不起作用。 相反，您可以使用特征对象，我们将在第 17 章中介绍它。

现在我们已经讨论了使用向量的一些最常见的方法，请务必查看 API 文档，了解标准库在 Vec<T> 上定义的所有有用方法。 例如，除了推送之外，弹出方法还会删除并返回最后一个元素。

### Dropping a Vector Drops Its Elements
Like any other struct, a vector is freed when it goes out of scope, as annotated in Listing 8-10.

```
fn main() {
    {
        let v = vec![1, 2, 3, 4];

        // do stuff with v
    } // <- v goes out of scope and is freed here
}
```
Listing 8-10: Showing where the vector and its elements are dropped

When the vector gets dropped, all of its contents are also dropped, meaning the integers it holds will be cleaned up. The borrow checker ensures that any references to contents of a vector are only used while the vector itself is valid.

Let’s move on to the next collection type: String!

## 8.2 字符串(Storing UTF-8 Encoded Text With Strings)

We talked about strings in Chapter 4, but we’ll look at them in more depth now. New Rustaceans commonly get stuck on strings for a combination of three reasons: Rust’s propensity for exposing possible errors, strings being a more complicated data structure than many programmers give them credit for, and UTF-8. These factors combine in a way that can seem difficult when you’re coming from other programming languages.

We discuss strings in the context of collections because strings are implemented as a collection of bytes, plus some methods to provide useful functionality when those bytes are interpreted as text. In this section, we’ll talk about the operations on String that every collection type has, such as creating, updating, and reading. We’ll also discuss the ways in which String is different from the other collections, namely how indexing into a String is complicated by the differences between how people and computers interpret String data.

我们在第 4 章中讨论了字符串，但现在我们将更深入地研究它们。 Rustace 新人通常会因为三个原因而陷入字符串困境：Rust 暴露可能错误的倾向、字符串是一种比许多程序员所认为的更复杂的数据结构，以及 UTF-8。 当您来自其他编程语言时，这些因素以一种看似困难的方式结合在一起。

我们在集合的上下文中讨论字符串，因为字符串是作为字节集合实现的，另外还有一些方法可以在这些字节解释为文本时提供有用的功能。 在本节中，我们将讨论每种集合类型对 String 的操作，例如创建、更新和读取。 我们还将讨论 String 与其他集合的不同之处，即由于人和计算机解释 String 数据的方式不同，对 String 的索引变得复杂。

### What is a String?
We’ll first define what we mean by the term string. Rust has only one string type in the core language, which is the string slice str that is usually seen in its borrowed form &str. In Chapter 4, we talked about string slices, which are references to some UTF-8 encoded string data stored elsewhere. String literals, for example, are stored in the program’s binary and are therefore string slices.

The String type, which is provided by Rust’s standard library rather than coded into the core language, is a growable, mutable, owned, UTF-8 encoded string type. When Rustaceans refer to “strings” in Rust, they might be referring to either the String or the string slice &str types, not just one of those types. Although this section is largely about String, both types are used heavily in Rust’s standard library, and both String and string slices are UTF-8 encoded.

我们首先定义术语字符串的含义。 Rust 在核心语言中只有一种字符串类型，即字符串切片 str，通常以其借用形式 &str 出现。 在第 4 章中，我们讨论了字符串切片，它是对存储在其他地方的一些 UTF-8 编码字符串数据的引用。 例如，字符串文字存储在程序的二进制文件中，因此是字符串切片。

String 类型由 Rust 的标准库提供，而不是编码到核心语言中，是一种可增长的、可变的、拥有的、UTF-8 编码的字符串类型。 当 Rustaceans 在 Rust 中引用“字符串”时，他们可能指的是 String 或字符串切片 &str 类型，而不仅仅是这些类型之一。 虽然本节主要讨论 String，但这两种类型在 Rust 标准库中都大量使用，并且 String 和字符串切片都是 UTF-8 编码的。

### Creating a New String

Many of the same operations available with Vec<T> are available with String as well, because String is actually implemented as a wrapper around a vector of bytes with some extra guarantees, restrictions, and capabilities. An example of a function that works the same way with Vec<T> and String is the new function to create an instance, shown in Listing 8-11.

```
fn main() {
    let mut s = String::new();
}
```
Listing 8-11: Creating a new, empty String

This line creates a new empty string called s, which we can then load data into. Often, we’ll have some initial data that we want to start the string with. For that, we use the to_string method, which is available on any type that implements the Display trait, as string literals do. Listing 8-12 shows two examples.

```
fn main() {
    let data = "initial contents";

    let s = data.to_string();

    // the method also works on a literal directly:
    let s = "initial contents".to_string();
}
```
Listing 8-12: Using the to_string method to create a String from a string literal

This code creates a string containing initial contents.

We can also use the function String::from to create a String from a string literal. The code in Listing 8-13 is equivalent to the code from Listing 8-12 that uses to_string.

示例 8-12：使用 to_string 方法从字符串文字创建字符串

此代码创建一个包含初始内容的字符串。

我们还可以使用函数 String::from 从字符串文字创建字符串。 清单 8-13 中的代码与清单 8-12 中使用 to_string 的代码等效。
```
fn main() {
    let s = String::from("initial contents");
}
```
Listing 8-13: Using the String::from function to create a String from a string literal

Because strings are used for so many things, we can use many different generic APIs for strings, providing us with a lot of options. Some of them can seem redundant, but they all have their place! In this case, String::from and to_string do the same thing, so which you choose is a matter of style and readability.

Remember that strings are UTF-8 encoded, so we can include any properly encoded data in them, as shown in Listing 8-14.

```
fn main() {
    let hello = String::from("السلام عليكم");
    let hello = String::from("Dobrý den");
    let hello = String::from("Hello");
    let hello = String::from("שָׁלוֹם");
    let hello = String::from("नमस्ते");
    let hello = String::from("こんにちは");
    let hello = String::from("안녕하세요");
    let hello = String::from("你好");
    let hello = String::from("Olá");
    let hello = String::from("Здравствуйте");
    let hello = String::from("Hola");
}
```
Listing 8-14: Storing greetings in different languages in strings

All of these are valid String values.

### Updating a String

A String can grow in size and its contents can change, just like the contents of a Vec<T>, if you push more data into it. In addition, you can conveniently use the + operator or the format! macro to concatenate String values.

如果将更多数据放入 String 中，则 String 的大小可能会增加，并且其内容可能会发生变化，就像 Vec<T> 的内容一样。 此外，您还可以方便地使用+运算符或格式！ 用于连接字符串值的宏。

#### Appending to a String with push_str and push
We can grow a String by using the push_str method to append a string slice, as shown in Listing 8-15.
```
fn main() {
    let mut s = String::from("foo");
    s.push_str("bar");
}
```
Listing 8-15: Appending a string slice to a String using the push_str method

After these two lines, s will contain foobar. The push_str method takes a string slice because we don’t necessarily want to take ownership of the parameter. For example, in the code in Listing 8-16, we want to be able to use s2 after appending its contents to s1.

示例 8-15：使用 push_str 方法将字符串切片附加到字符串

这两行之后，s 将包含 foobar。 Push_str 方法接受一个字符串切片，因为我们不一定想要获得参数的所有权。 例如，在清单 8-16 的代码中，我们希望在将 s2 的内容附加到 s1 后能够使用 s2。
```
fn main() {
    let mut s1 = String::from("foo");
    let s2 = "bar";
    s1.push_str(s2);
    println!("s2 is {s2}");
}
```

Listing 8-16: Using a string slice after appending its contents to a String

If the push_str method took ownership of s2, we wouldn’t be able to print its value on the last line. However, this code works as we’d expect!

The push method takes a single character as a parameter and adds it to the String. Listing 8-17 adds the letter “l” to a String using the push method.

示例 8-16：将字符串切片的内容附加到字符串后使用它

如果push_str方法获得了s2的所有权，我们将无法在最后一行打印它的值。 然而，这段代码的工作原理正如我们所期望的！

Push 方法将单个字符作为参数并将其添加到 String 中。 清单 8-17 使用 push 方法将字母“l”添加到字符串中。
```
fn main() {
    let mut s = String::from("lo");
    s.push('l');
}
```
Listing 8-17: Adding one character to a String value using push

As a result, s will contain lol.

#### Concatenation with the + Operator or the format! Macro

Often, you’ll want to combine two existing strings. One way to do so is to use the + operator, as shown in Listing 8-18.
```
fn main() {
    let s1 = String::from("Hello, ");
    let s2 = String::from("world!");
    let s3 = s1 + &s2; // note s1 has been moved here and can no longer be used
}
```
Listing 8-18: Using the + operator to combine two String values into a new String value

The string s3 will contain Hello, world!. The reason s1 is no longer valid after the addition, and the reason we used a reference to s2, has to do with the signature of the method that’s called when we use the + operator. The + operator uses the add method, whose signature looks something like this:

```
fn add(self, s: &str) -> String {}
```
In the standard library, you'll see add defined using generics and associated types. Here, we’ve substituted in concrete types, which is what happens when we call this method with String values. We’ll discuss generics in Chapter 10. This signature gives us the clues we need to understand the tricky bits of the + operator.

First, s2 has an &, meaning that we’re adding a reference of the second string to the first string. This is because of the s parameter in the add function: we can only add a &str to a String; we can’t add two String values together. But wait—the type of &s2 is &String, not &str, as specified in the second parameter to add. So why does Listing 8-18 compile?

The reason we’re able to use &s2 in the call to add is that the compiler can coerce the &String argument into a &str. When we call the add method, Rust uses a deref coercion, which here turns &s2 into &s2[..]. We’ll discuss deref coercion in more depth in Chapter 15. Because add does not take ownership of the s parameter, s2 will still be a valid String after this operation.

Second, we can see in the signature that add takes ownership of self, because self does not have an &. This means s1 in Listing 8-18 will be moved into the add call and will no longer be valid after that. So although let s3 = s1 + &s2; looks like it will copy both strings and create a new one, this statement actually takes ownership of s1, appends a copy of the contents of s2, and then returns ownership of the result. In other words, it looks like it’s making a lot of copies but isn’t; the implementation is more efficient than copying.

If we need to concatenate multiple strings, the behavior of the + operator gets unwieldy:

在标准库中，您将看到使用泛型和关联类型添加定义。 在这里，我们替换为具体类型，这就是当我们使用 String 值调用此方法时会发生的情况。 我们将在第 10 章讨论泛型。这个签名为我们提供了理解 + 运算符的棘手部分所需的线索。

首先，s2 有一个 &，这意味着我们将第二个字符串的引用添加到第一个字符串。 这是因为add函数中的s参数：我们只能将&str添加到String中； 我们不能将两个字符串值相加。 但是等等，&s2 的类型是 &String，而不是 &str，如要添加的第二个参数中指定的那样。 那么清单 8-18 为什么可以编译呢？

我们能够在 add 调用中使用 &s2 的原因是编译器可以将 &String 参数强制转换为 &str。 当我们调用 add 方法时，Rust 使用 deref 强制转换，将 &s2 转换为 &s2[..]。 我们将在第 15 章更深入地讨论 deref 强制转换。因为 add 不获取 s 参数的所有权，所以在此操作之后 s2 仍然是一个有效的字符串。

其次，我们可以在签名中看到 add 拥有 self 的所有权，因为 self 没有 &。 这意味着清单 8-18 中的 s1 将被移至 add 调用中，并且此后将不再有效。 所以虽然让 s3 = s1 + &s2; 看起来它会复制两个字符串并创建一个新字符串，该语句实际上获取 s1 的所有权，附加 s2 内容的副本，然后返回结果的所有权。 换句话说，它看起来像是制作了很多副本，但事实并非如此； 执行比复制更有效率。

如果我们需要连接多个字符串，+ 运算符的行为就会变得笨拙：
```
fn main() {
    let s1 = String::from("tic");
    let s2 = String::from("tac");
    let s3 = String::from("toe");

    let s = s1 + "-" + &s2 + "-" + &s3;
}
```
At this point, s will be tic-tac-toe. With all of the + and " characters, it’s difficult to see what’s going on. For more complicated string combining, we can instead use the format! macro:

```
fn main() {
    let s1 = String::from("tic");
    let s2 = String::from("tac");
    let s3 = String::from("toe");

    let s = format!("{s1}-{s2}-{s3}");
}
```
This code also sets s to tic-tac-toe. The format! macro works like println!, but instead of printing the output to the screen, it returns a String with the contents. The version of the code using format! is much easier to read, and the code generated by the format! macro uses references so that this call doesn’t take ownership of any of its parameters.

此代码还将 s 设置为 tic-tac-toe。 格式！ 宏的工作方式与 println! 类似，但它不是将输出打印到屏幕上，而是返回一个包含内容的字符串。 版本代码使用格式！ 更容易阅读，而且格式生成的代码！ 宏使用引用，因此该调用不会取得其任何参数的所有权。

### 索引到字符串(Indexing into Strings)
In many other programming languages, accessing individual characters in a string by referencing them by index is a valid and common operation. However, if you try to access parts of a String using indexing syntax in Rust, you’ll get an error. Consider the invalid code in Listing 8-19.

在许多其他编程语言中，通过索引引用字符串中的各个字符来访问它们是有效且常见的操作。 但是，如果您尝试在 Rust 中使用索引语法访问字符串的一部分，您将收到错误。 考虑清单 8-19 中的无效代码。

```
fn main() {
    let s1 = String::from("hello");
    let h = s1[0];
}
```
Listing 8-19: Attempting to use indexing syntax with a String

This code will result in the following error:
```
$ cargo run
   Compiling collections v0.1.0 (file:///projects/collections)
error[E0277]: the type `String` cannot be indexed by `{integer}`
 --> src/main.rs:3:13
  |
3 |     let h = s1[0];
  |             ^^^^^ `String` cannot be indexed by `{integer}`
  |
  = help: the trait `Index<{integer}>` is not implemented for `String`
  = help: the following other types implement trait `Index<Idx>`:
            <String as Index<RangeFrom<usize>>>
            <String as Index<RangeFull>>
            <String as Index<RangeInclusive<usize>>>
            <String as Index<RangeTo<usize>>>
            <String as Index<RangeToInclusive<usize>>>
            <String as Index<std::ops::Range<usize>>>

For more information about this error, try `rustc --explain E0277`.
error: could not compile `collections` due to previous error

```

The error and the note tell the story: Rust strings don’t support indexing. But why not? To answer that question, we need to discuss how Rust stores strings in memory.

#### internal Representation
A String is a wrapper over a Vec<u8>. Let’s look at some of our properly encoded UTF-8 example strings from Listing 8-14. First, this one:
```
 let hello = String::from("Hola");
```
In this case, len will be 4, which means the vector storing the string “Hola” is 4 bytes long. Each of these letters takes 1 byte when encoded in UTF-8. The following line, however, may surprise you. (Note that this string begins with the capital Cyrillic letter Ze, not the number 3.)
```
    let hello = String::from("Здравствуйте");

```
Asked how long the string is, you might say 12. In fact, Rust’s answer is 24: that’s the number of bytes it takes to encode “Здравствуйте” in UTF-8, because each Unicode scalar value in that string takes 2 bytes of storage. Therefore, an index into the string’s bytes will not always correlate to a valid Unicode scalar value. To demonstrate, consider this invalid Rust code:
当被问到字符串有多长时，你可能会说 12。事实上，Rust 的答案是 24：这是用 UTF-8 编码“Здравствуйте”所需的字节数，因为该字符串中的每个 Unicode 标量值需要 2 个字节的存储空间 。 因此，字符串字节的索引并不总是与有效的 Unicode 标量值相关。 为了进行演示，请考虑以下无效的 Rust 代码：
```
let hello = "Здравствуйте";
let answer = &hello[0];
```
You already know that answer will not be З, the first letter. When encoded in UTF-8, the first byte of З is 208 and the second is 151, so it would seem that answer should in fact be 208, but 208 is not a valid character on its own. Returning 208 is likely not what a user would want if they asked for the first letter of this string; however, that’s the only data that Rust has at byte index 0. Users generally don’t want the byte value returned, even if the string contains only Latin letters: if &"hello"[0] were valid code that returned the byte value, it would return 104, not h.

The answer, then, is that to avoid returning an unexpected value and causing bugs that might not be discovered immediately, Rust doesn’t compile this code at all and prevents misunderstandings early in the development process.

您已经知道答案不会是第一个字母 З。 当以 UTF-8 编码时，З 的第一个字节是 208，第二个字节是 151，因此看起来答案实际上应该是 208，但 208 本身并不是有效字符。 如果用户询问该字符串的第一个字母，那么返回 208 可能不是用户想要的结果； 然而，这是 Rust 在字节索引 0 处拥有的唯一数据。用户通常不希望返回字节值，即使字符串仅包含拉丁字母：如果 &"hello"[0] 是返回字节值的有效代码 ，它将返回 104，而不是 h。

那么答案是，为了避免返回意外的值并导致可能无法立即发现的错误，Rust 根本不编译此代码，并防止在开发过程的早期产生误解。

#### 字节和标量值以及字素簇！ 天啊！(Bytes and Scalar Values and Grapheme Clusters! Oh My!)
Another point about UTF-8 is that there are actually three relevant ways to look at strings from Rust’s perspective: as bytes, scalar values, and grapheme clusters (the closest thing to what we would call letters).

If we look at the Hindi word “नमस्ते” written in the Devanagari script, it is stored as a vector of u8 values that looks like this:
关于 UTF-8 的另一点是，从 Rust 的角度来看，实际上有三种相关的方式来看待字符串：字节、标量值和字素簇（最接近我们所说的字母）。

如果我们看一下用梵文脚本编写的印地语单词“नमस्ते”，它存储为 u8 值的向量，如下所示：
```[224, 164, 168, 224, 164, 174, 224, 164, 184, 224, 165, 141, 224, 164, 164,
224, 165, 135]
```
That’s 18 bytes and is how computers ultimately store this data. If we look at them as Unicode scalar values, which are what Rust’s char type is, those bytes look like this:
这是 18 个字节，也是计算机最终存储这些数据的方式。 如果我们将它们视为 Unicode 标量值（Rust 的 char 类型），那么这些字节如下所示：
```
['न', 'म', 'स', '्', 'त', 'े']
```
There are six char values here, but the fourth and sixth are not letters: they’re diacritics that don’t make sense on their own. Finally, if we look at them as grapheme clusters, we’d get what a person would call the four letters that make up the Hindi word:

这里有六个 char 值，但第四个和第六个不是字母：它们是本身没有意义的变音符号。 最后，如果我们将它们视为字素簇，我们就会得到人们所说的构成印地语单词的四个字母：
```
["न", "म", "स्", "ते"]
```
Rust 提供了不同的方式来解释计算机存储的原始字符串数据，以便每个程序都可以选择它需要的解释，无论数据采用哪种人类语言。

Rust 不允许我们索引字符串来获取字符的最后一个原因是索引操作预计总是需要恒定的时间 (O(1))。 但不可能保证字符串的性能，因为 Rust 必须从头到索引遍历内容以确定有多少个有效字符。

### Slicing Strings
Indexing into a string is often a bad idea because it’s not clear what the return type of the string-indexing operation should be: a byte value, a character, a grapheme cluster, or a string slice. If you really need to use indices to create string slices, therefore, Rust asks you to be more specific.

Rather than indexing using [] with a single number, you can use [] with a range to create a string slice containing particular bytes:

对字符串进行索引通常是一个坏主意，因为不清楚字符串索引操作的返回类型应该是什么：字节值、字符、字素簇或字符串切片。 因此，如果您确实需要使用索引来创建字符串切片，Rust 会要求您更加具体。

您可以使用带有范围的 [] 来创建包含特定字节的字符串切片，而不是使用带有单个数字的 [] 进行索引：

```
let hello = "Здравствуйте";

let s = &hello[0..4];
```
Here, s will be a &str that contains the first 4 bytes of the string. Earlier, we mentioned that each of these characters was 2 bytes, which means s will be Зд.

If we were to try to slice only part of a character’s bytes with something like &hello[0..1], Rust would panic at runtime in the same way as if an invalid index were accessed in a vector:

这里，s 将是一个 &str，包含字符串的前 4 个字节。 之前，我们提到每个字符都是 2 个字节，这意味着 s 将是 Зд。

如果我们尝试使用 &hello[0..1] 之类的内容仅切片字符字节的一部分，Rust 会在运行时出现恐慌，就像在向量中访问无效索引一样：

```
$ cargo run
   Compiling collections v0.1.0 (file:///projects/collections)
    Finished dev [unoptimized + debuginfo] target(s) in 0.43s
     Running `target/debug/collections`
thread 'main' panicked at 'byte index 1 is not a char boundary; it is inside 'З' (bytes 0..2) of `Здравствуйте`', src/main.rs:4:14
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace

```
You should use ranges to create string slices with caution, because doing so can crash your program.

### 迭代字符串的方法
The best way to operate on pieces of strings is to be explicit about whether you want characters or bytes. For individual Unicode scalar values, use the chars method. Calling chars on “Зд” separates out and returns two values of type char, and you can iterate over the result to access each element:

```
for c in "Зд".chars() {
    println!("{c}");
}

```
This code will the following:
```
З
д

```
Alternatively, the bytes method returns each raw byte, which might be appropriate for your domain:
```
for b in "Зд".bytes() {
    println!("{b}");
}

```
this code will print the four bytes that make up this string:
```
208
151
208
180

```
But be sure to remember that valid Unicode scalar values may be made up of more than 1 byte.

Getting grapheme clusters from strings as with the Devanagari script is complex, so this functionality is not provided by the standard library. Crates are available on crates.io if this is the functionality you need.

但请务必记住，有效的 Unicode 标量值可能由超过 1 个字节组成。

与使用梵文脚本一样从字符串获取字素簇很复杂，因此标准库不提供此功能。 如果您需要此功能，可以在 crates.io 上找到 crate。

### String Are Not So Simple

To summarize, strings are complicated. Different programming languages make different choices about how to present this complexity to the programmer. Rust has chosen to make the correct handling of String data the default behavior for all Rust programs, which means programmers have to put more thought into handling UTF-8 data upfront. This trade-off exposes more of the complexity of strings than is apparent in other programming languages, but it prevents you from having to handle errors involving non-ASCII characters later in your development life cycle.

The good news is that the standard library offers a lot of functionality built off the String and &str types to help handle these complex situations correctly. Be sure to check out the documentation for useful methods like contains for searching in a string and replace for substituting parts of a string with another string.

Let’s switch to something a bit less complex: hash maps!

总而言之，字符串很复杂。 不同的编程语言对于如何向程序员呈现这种复杂性做出了不同的选择。 Rust 选择将正确处理字符串数据作为所有 Rust 程序的默认行为，这意味着程序员必须预先考虑如何处理 UTF-8 数据。 与其他编程语言相比，这种权衡暴露了更多的字符串复杂性，但它使您不必在开发生命周期的后期处理涉及非 ASCII 字符的错误。

好消息是，标准库提供了许多基于 String 和 &str 类型构建的功能，以帮助正确处理这些复杂的情况。 请务必查看文档以了解有用的方法，例如用于在字符串中搜索的 contains 以及用于用另一个字符串替换字符串的一部分的替换。

让我们切换到不太复杂的东西：哈希映射！
## 8.3 Storing Keys with Associated Values in Hash Maps
The last of our common collections is the hash map. The type HashMap<K, V> stores a mapping of keys of type K to values of type V using a hashing function, which determines how it places these keys and values into memory. Many programming languages support this kind of data structure, but they often use a different name, such as hash, map, object, hash table, dictionary, or associative array, just to name a few.

Hash maps are useful when you want to look up data not by using an index, as you can with vectors, but by using a key that can be of any type. For example, in a game, you could keep track of each team’s score in a hash map in which each key is a team’s name and the values are each team’s score. Given a team name, you can retrieve its score.

We’ll go over the basic API of hash maps in this section, but many more goodies are hiding in the functions defined on HashMap<K, V> by the standard library. As always, check the standard library documentation for more information.

我们最后一个常见的集合是哈希图。 HashMap<K, V> 类型使用哈希函数存储 K 类型的键到 V 类型的值的映射，该映射决定如何将这些键和值放入内存中。 许多编程语言都支持这种数据结构，但它们经常使用不同的名称，例如哈希、映射、对象、哈希表、字典或关联数组等等。

当您想要查找数据时，哈希映射非常有用，而不是像使用向量那样使用索引，而是使用可以是任何类型的键来查找数据。 例如，在游戏中，您可以在哈希映射中跟踪每个团队的得分，其中每个键是团队的名称，值是每个团队的得分。 给定团队名称，您可以检索其得分。

我们将在本节中介绍哈希映射的基本 API，但标准库在 HashMap<K, V> 上定义的函数中隐藏着更多好东西。 与往常一样，请检查标准库文档以获取更多信息。

### Creating a New Hash Map
One way to create an empty hash map is using new and adding elements with insert. In Listing 8-20, we’re keeping track of the scores of two teams whose names are Blue and Yellow. The Blue team starts with 10 points, and the Yellow team starts with 50.

创建空哈希映射的一种方法是使用 new 并通过插入添加元素。 在清单 8-20 中，我们记录了蓝队和黄队两支球队的得分。 蓝队起赛10分，黄队起赛50分。

```
fn main() {
    use std::collections::HashMap;

    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);
}
```
Listing 8-20: Creating a new hash map and inserting some keys and values

Note that we need to first use the HashMap from the collections portion of the standard library. Of our three common collections, this one is the least often used, so it’s not included in the features brought into scope automatically in the prelude. Hash maps also have less support from the standard library; there’s no built-in macro to construct them, for example.

Just like vectors, hash maps store their data on the heap. This HashMap has keys of type String and values of type i32. Like vectors, hash maps are homogeneous: all of the keys must have the same type as each other, and all of the values must have the same type.

示例 8-20：创建一个新的哈希映射并插入一些键和值

请注意，我们需要首先使用标准库集合部分中的 HashMap。 在我们的三个常见集合中，这个集合是最不常用的，因此它不包含在序言中自动纳入范围的功能中。 标准库对哈希映射的支持也较少； 例如，没有内置宏来构造它们。

就像向量一样，哈希映射将其数据存储在堆上。 该 HashMap 具有 String 类型的键和 i32 类型的值。 与向量一样，哈希映射是同构的：所有键必须具有相同的类型，并且所有值必须具有相同的类型。

### Accessing Values in a Hash Map
We can get a value out of the hash map by providing its key to the get method, as shown in Listing 8-21.

```
fn main() {
    use std::collections::HashMap;

    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);

    let team_name = String::from("Blue");
    let score = scores.get(&team_name).copied().unwrap_or(0);
}
```
Listing 8-21: Accessing the score for the Blue team stored in the hash map

Here, score will have the value that’s associated with the Blue team, and the result will be 10. The get method returns an Option<&V>; if there’s no value for that key in the hash map, get will return None. This program handles the Option by calling copied to get an Option<i32> rather than an Option<&i32>, then unwrap_or to set score to zero if scores doesn't have an entry for the key.

We can iterate over each key/value pair in a hash map in a similar manner as we do with vectors, using a for loop:

示例 8-21：访问存储在哈希映射中的蓝队得分

这里，score 将具有与蓝队关联的值，结果将为 10。 get 方法返回 Option<&V>； 如果哈希映射中该键没有值，则 get 将返回 None。 该程序通过调用 Copy 来获取 Option<i32> 而不是 Option<&i32> 来处理 Option，然后如果 Score 没有该键的条目，则使用 unwrap_or 将 Score 设置为零。

我们可以使用 for 循环以与向量类似的方式迭代哈希映射中的每个键/值对：
```
fn main() {
    use std::collections::HashMap;

    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);

    for (key, value) in &scores {
        println!("{key}: {value}");
    }
}
```
This code will prient each pair in an arbitrary order:
```
Yellow: 50
Blue: 10
```
### Hash Maps and Ownership
For types that implement the Copy trait, like i32, the values are copied into the hash map. For owned values like String, the values will be moved and the hash map will be the owner of those values, as demonstrated in Listing 8-22.

对于实现 Copy 特征的类型（例如 i32），值将被复制到哈希映射中。 对于像 String 这样的自有值，这些值将被移动，并且哈希映射将成为这些值的所有者，如清单 8-22 所示。

```
fn main() {
    use std::collections::HashMap;

    let field_name = String::from("Favorite color");
    let field_value = String::from("Blue");

    let mut map = HashMap::new();
    map.insert(field_name, field_value);
    // field_name and field_value are invalid at this point, try using them and
    // see what compiler error you get!
}
```

Listing 8-22: Showing that keys and values are owned by the hash map once they’re inserted

We aren’t able to use the variables field_name and field_value after they’ve been moved into the hash map with the call to insert.

If we insert references to values into the hash map, the values won’t be moved into the hash map. The values that the references point to must be valid for at least as long as the hash map is valid. We’ll talk more about these issues in the “Validating References with Lifetimes” section in Chapter 10.

示例 8-22：显示键和值一旦插入就归哈希映射所有

通过调用 insert 将变量 field_name 和 field_value 移动到哈希映射后，我们无法使用它们。

如果我们将对值的引用插入到哈希映射中，则这些值不会移动到哈希映射中。 引用指向的值必须至少在哈希映射有效期间有效。 我们将在第 10 章的“使用生命周期验证引用”部分详细讨论这些问题。

### updating a Hash Map
Although the number of key and value pairs is growable, each unique key can only have one value associated with it at a time (but not vice versa: for example, both the Blue team and the Yellow team could have value 10 stored in the scores hash map).

When you want to change the data in a hash map, you have to decide how to handle the case when a key already has a value assigned. You could replace the old value with the new value, completely disregarding the old value. You could keep the old value and ignore the new value, only adding the new value if the key doesn’t already have a value. Or you could combine the old value and the new value. Let’s look at how to do each of these!

虽然键和值对的数量是可以增长的，但每个唯一键一次只能有一个与其关联的值（反之亦然：例如，蓝队和黄队都可以将值 10 存储在分数中） 哈希映射）。

当您想要更改哈希映射中的数据时，您必须决定如何处理键已分配值的情况。 您可以用新值替换旧值，完全忽略旧值。 您可以保留旧值并忽略新值，仅在键还没有值时才添加新值。 或者您可以将旧值和新值结合起来。 让我们看看如何做到每一个！

#### Overwriting a Value

If we insert a key and a value into a hash map and then insert that same key with a different value, the value associated with that key will be replaced. Even though the code in Listing 8-23 calls insert twice, the hash map will only contain one key/value pair because we’re inserting the value for the Blue team’s key both times.
如果我们将一个键和一个值插入哈希映射，然后插入具有不同值的相同键，则与该键关联的值将被替换。 即使清单 8-23 中的代码调用了两次 insert，哈希映射也只会包含一个键/值对，因为我们两次都插入了蓝队键的值。
```
fn main() {
    use std::collections::HashMap;

    let mut scores = HashMap::new();

    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Blue"), 25);

    println!("{:?}", scores);
}
```
Listing 8-23: Replacing a value stored with a particular key

This code will print {"Blue": 25}. The original value of 10 has been overwritten.

#### Adding a Key and Value Only if a Key isn't Present

It’s common to check whether a particular key already exists in the hash map with a value then take the following actions: if the key does exist in the hash map, the existing value should remain the way it is. If the key doesn’t exist, insert it and a value for it.

Hash maps have a special API for this called entry that takes the key you want to check as a parameter. The return value of the entry method is an enum called Entry that represents a value that might or might not exist. Let’s say we want to check whether the key for the Yellow team has a value associated with it. If it doesn’t, we want to insert the value 50, and the same for the Blue team. Using the entry API, the code looks like Listing 8-24.

通常检查哈希映射中是否已存在特定键及其值，然后采取以下操作：如果哈希映射中确实存在该键，则现有值应保持原样。 如果该键不存在，请插入该键及其值。

哈希映射有一个特殊的 API，称为条目，它将您要检查的键作为参数。 Entry 方法的返回值是一个名为 Entry 的枚举，它表示一个可能存在也可能不存在的值。 假设我们要检查黄队的密钥是否有与之关联的值。 如果没有，我们要插入值 50，蓝队也是如此。 使用入口 API，代码如清单 8-24 所示。

```
fn main() {
    use std::collections::HashMap;

    let mut scores = HashMap::new();
    scores.insert(String::from("Blue"), 10);

    scores.entry(String::from("Yellow")).or_insert(50);
    scores.entry(String::from("Blue")).or_insert(50);

    println!("{:?}", scores);
}
```
Listing 8-24: Using the entry method to only insert if the key does not already have a value

The or_insert method on Entry is defined to return a mutable reference to the value for the corresponding Entry key if that key exists, and if not, inserts the parameter as the new value for this key and returns a mutable reference to the new value. This technique is much cleaner than writing the logic ourselves and, in addition, plays more nicely with the borrow checker.

Running the code in Listing 8-24 will print {"Yellow": 50, "Blue": 10}. The first call to entry will insert the key for the Yellow team with the value 50 because the Yellow team doesn’t have a value already. The second call to entry will not change the hash map because the Blue team already has the value 10.

示例 8-24：仅当键还没有值时才使用输入方法插入

Entry 上的 or_insert 方法被定义为：如果该键存在，则返回对相应 Entry 键的值的可变引用；如果不存在，则插入该参数作为该键的新值，并返回对新值的可变引用。 这种技术比我们自己编写逻辑要干净得多，此外，与借用检查器配合得更好。

运行清单 8-24 中的代码将打印 {"Yellow": 50, "Blue": 10}。 第一次调用 Entry 将插入值为 50 的黄队密钥，因为黄队还没有值。 第二次调用 Entry 不会更改哈希映射，因为蓝队已经拥有值 10。

#### Updating a Value Based on the Old Value
Another common use case for hash maps is to look up a key’s value and then update it based on the old value. For instance, Listing 8-25 shows code that counts how many times each word appears in some text. We use a hash map with the words as keys and increment the value to keep track of how many times we’ve seen that word. If it’s the first time we’ve seen a word, we’ll first insert the value 0.
```
fn main() {
    use std::collections::HashMap;

    let text = "hello world wonderful world";

    let mut map = HashMap::new();

    for word in text.split_whitespace() {
        let count = map.entry(word).or_insert(0);
        *count += 1;
    }

    println!("{:?}", map);
}
```

Listing 8-25: Counting occurrences of words using a hash map that stores words and counts
示例 8-25：使用存储单词和计数的哈希映射来计算单词的出现次数
This code will print {"world": 2, "hello": 1, "wonderful": 1}. You might see the same key/value pairs printed in a different order: recall from the “Accessing Values in a Hash Map” section that iterating over a hash map happens in an arbitrary order.

The split_whitespace method returns an iterator over sub-slices, separated by whitespace, of the value in text. The or_insert method returns a mutable reference (&mut V) to the value for the specified key. Here we store that mutable reference in the count variable, so in order to assign to that value, we must first dereference count using the asterisk (*). The mutable reference goes out of scope at the end of the for loop, so all of these changes are safe and allowed by the borrowing rules.



此代码将打印 {"world": 2, "hello": 1, "wonderful": 1}。 您可能会看到以不同顺序打印的相同键/值对：回想一下“访问哈希映射中的值”部分，哈希映射上的迭代以任意顺序发生。

split_whitespace 方法返回文本中值的子切片（以空格分隔）的迭代器。 or_insert 方法返回对指定键的值的可变引用 (&mut V)。 在这里，我们将该可变引用存储在 count 变量中，因此为了分配给该值，我们必须首先使用星号 (*) 取消引用 count。 可变引用在 for 循环结束时超出范围，因此所有这些更改都是安全的并且是借用规则允许的。

### Hashing Functions
By default, HashMap uses a hashing function called SipHash that can provide resistance to Denial of Service (DoS) attacks involving hash tables1. This is not the fastest hashing algorithm available, but the trade-off for better security that comes with the drop in performance is worth it. If you profile your code and find that the default hash function is too slow for your purposes, you can switch to another function by specifying a different hasher. A hasher is a type that implements the BuildHasher trait. We’ll talk about traits and how to implement them in Chapter 10. You don’t necessarily have to implement your own hasher from scratch; crates.io has libraries shared by other Rust users that provide hashers implementing many common hashing algorithms.

默认情况下，HashMap 使用名为 SipHash 的哈希函数，该函数可以抵抗涉及哈希表的拒绝服务 (DoS) 攻击。 这不是可用的最快的哈希算法，但为了获得更好的安全性而带来的性能下降是值得的。 如果您分析代码并发现默认哈希函数对于您的目的而言太慢，则可以通过指定不同的哈希器来切换到另一个函数。 哈希器是一种实现 BuildHasher 特征的类型。 我们将在第 10 章中讨论特征以及如何实现它们。你不一定要从头开始实现你自己的哈希器； crates.io 拥有其他 Rust 用户共享的库，这些库提供了实现许多常见哈希算法的哈希器。

## 8.4 Summary
Vectors, strings, and hash maps will provide a large amount of functionality necessary in programs when you need to store, access, and modify data. Here are some exercises you should now be equipped to solve:

+ Given a list of integers, use a vector and return the median (when sorted, the value in the middle position) and mode (the value that occurs most often; a hash map will be helpful here) of the list.
+ Convert strings to pig latin. The first consonant of each word is moved to the end of the word and “ay” is added, so “first” becomes “irst-fay.” Words that start with a vowel have “hay” added to the end instead (“apple” becomes “apple-hay”). Keep in mind the details about UTF-8 encoding!
+ Using a hash map and vectors, create a text interface to allow a user to add employee names to a department in a company. For example, “Add Sally to Engineering” or “Add Amir to Sales.” Then let the user retrieve a list of all people in a department or all people in the company by department, sorted alphabetically.
The standard library API documentation describes methods that vectors, strings, and hash maps have that will be helpful for these exercises!

We’re getting into more complex programs in which operations can fail, so, it’s a perfect time to discuss error handling. We’ll do that next!

当您需要存储、访问和修改数据时，向量、字符串和哈希映射将提供程序中所需的大量功能。 以下是您现在应该能够解决的一些练习：

+ 给定一个整数列表，使用向量并返回列表的中位数（排序后，中间位置的值）和众数（最常出现的值；哈希映射在这里会很有帮助）。
+ 将字符串转换为 Pig Latin。 每个单词的第一个辅音移至单词末尾并添加“ay”，因此“first”变为“irst-fay”。 以元音开头的单词会在末尾添加“hay”（“apple”变成“apple-hay”）。 请记住有关 UTF-8 编码的详细信息！
+ 使用哈希映射和向量创建一个文本界面，以允许用户将员工姓名添加到公司的部门。 例如，“将莎莉添加到工程部门”或“将阿米尔添加到销售部门”。 然后让用户检索一个部门中的所有人员或按部门按字母顺序排序的公司中的所有人员的列表。
标准库 API 文档描述了向量、字符串和哈希映射所具有的方法，这些方法对这些练习很有帮助！

我们正在研究更复杂的程序，其中操作可能会失败，因此，现在是讨论错误处理的最佳时机。 我们接下来就这样做！