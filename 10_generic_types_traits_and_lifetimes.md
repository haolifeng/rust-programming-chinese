# 泛型， Traits和生命周期
Every programming language has tools for effectively handling the duplication of concepts. In Rust, one such tool is generics: abstract stand-ins for concrete types or other properties. We can express the behavior of generics or how they relate to other generics without knowing what will be in their place when compiling and running the code.

Functions can take parameters of some generic type, instead of a concrete type like i32 or String, in the same way a function takes parameters with unknown values to run the same code on multiple concrete values. In fact, we’ve already used generics in Chapter 6 with Option<T>, Chapter 8 with Vec<T> and HashMap<K, V>, and Chapter 9 with Result<T, E>. In this chapter, you’ll explore how to define your own types, functions, and methods with generics!

First, we’ll review how to extract a function to reduce code duplication. We’ll then use the same technique to make a generic function from two functions that differ only in the types of their parameters. We’ll also explain how to use generic types in struct and enum definitions.

Then you’ll learn how to use traits to define behavior in a generic way. You can combine traits with generic types to constrain a generic type to accept only those types that have a particular behavior, as opposed to just any type.

Finally, we’ll discuss lifetimes: a variety of generics that give the compiler information about how references relate to each other. Lifetimes allow us to give the compiler enough information about borrowed values so that it can ensure references will be valid in more situations than it could without our help.

每种编程语言都有可以有效处理概念重复的工具。 在 Rust 中，这样的工具之一就是泛型：具体类型或其他属性的抽象替代品。 我们可以表达泛型的行为或它们与其他泛型的关系，而无需知道编译和运行代码时它们的位置是什么。

函数可以采用某种泛型类型的参数，而不是像 i32 或 String 这样的具体类型，就像函数采用未知值的参数在多个具体值上运行相同的代码一样。 事实上，我们已经在第 6 章中使用了泛型 Option<T>，在第 8 章中使用了 Vec<T> 和 HashMap<K, V>，在第 9 章中使用了 Result<T, E>。 在本章中，您将探索如何使用泛型定义自己的类型、函数和方法！

首先，我们将回顾如何提取函数以减少代码重复。 然后，我们将使用相同的技术从两个仅参数类型不同的函数创建一个通用函数。 我们还将解释如何在结构和枚举定义中使用泛型类型。

然后您将学习如何使用特征以通用方式定义行为。 您可以将特征与泛型类型结合起来，以限制泛型类型仅接受具有特定行为的类型，而不是仅接受任何类型。

最后，我们将讨论生命周期：各种泛型，为编译器提供有关引用如何相互关联的信息。 生命周期允许我们为编译器提供有关借用值的足够信息，以便它可以确保引用在更多情况下比没有我们的帮助时有效。

### Removing Duplication by Extracting a Function
Generics allow us to replace specific types with a placeholder that represents multiple types to remove code duplication. Before diving into generics syntax, then, let’s first look at how to remove duplication in a way that doesn’t involve generic types by extracting a function that replaces specific values with a placeholder that represents multiple values. Then we’ll apply the same technique to extract a generic function! By looking at how to recognize duplicated code you can extract into a function, you’ll start to recognize duplicated code that can use generics.

We begin with the short program in Listing 10-1 that finds the largest number in a list.

泛型允许我们用代表多种类型的占位符替换特定类型，以消除代码重复。 在深入研究泛型语法之前，我们首先看看如何以不涉及泛型类型的方式删除重复，方法是提取一个用表示多个值的占位符替换特定值的函数。 然后我们将应用相同的技术来提取通用函数！ 通过了解如何识别可以提取到函数中的重复代码，您将开始识别可以使用泛型的重复代码。

我们从清单 10-1 中的短程序开始，该程序查找列表中的最大数字。

Filename: src/main.rs
```
fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let mut largest = &number_list[0];

    for number in &number_list {
        if number > largest {
            largest = number;
        }
    }

    println!("The largest number is {}", largest);
}
```
Listing 10-1: Finding the largest number in a list of numbers

We store a list of integers in the variable number_list and place a reference to the first number in the list in a variable named largest. We then iterate through all the numbers in the list, and if the current number is greater than the number stored in largest, replace the reference in that variable. However, if the current number is less than or equal to the largest number seen so far, the variable doesn’t change, and the code moves on to the next number in the list. After considering all the numbers in the list, largest should refer to the largest number, which in this case is 100.

We've now been tasked with finding the largest number in two different lists of numbers. To do so, we can choose to duplicate the code in Listing 10-1 and use the same logic at two different places in the program, as shown in Listing 10-2.

我们将整数列表存储在变量 number_list 中，并将对列表中第一个数字的引用放置在名为largest 的变量中。 然后，我们迭代列表中的所有数字，如果当前数字大于 Maximum 中存储的数字，则替换该变量中的引用。 但是，如果当前数字小于或等于迄今为止看到的最大数字，则变量不会更改，并且代码将移至列表中的下一个数字。 考虑完列表中的所有数字后，最大应指最大的数字，在本例中为 100。

我们现在的任务是在两个不同的数字列表中找到最大的数字。 为此，我们可以选择复制清单 10-1 中的代码，并在程序中的两个不同位置使用相同的逻辑，如清单 10-2 所示。

Filename: src/main.rs
```
fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let mut largest = &number_list[0];

    for number in &number_list {
        if number > largest {
            largest = number;
        }
    }

    println!("The largest number is {}", largest);

    let number_list = vec![102, 34, 6000, 89, 54, 2, 43, 8];

    let mut largest = &number_list[0];

    for number in &number_list {
        if number > largest {
            largest = number;
        }
    }

    println!("The largest number is {}", largest);
}
```
Listing 10-2: Code to find the largest number in two lists of numbers

Although this code works, duplicating code is tedious and error prone. We also have to remember to update the code in multiple places when we want to change it.

To eliminate this duplication, we’ll create an abstraction by defining a function that operates on any list of integers passed in a parameter. This solution makes our code clearer and lets us express the concept of finding the largest number in a list abstractly.

In Listing 10-3, we extract the code that finds the largest number into a function named largest. Then we call the function to find the largest number in the two lists from Listing 10-2. We could also use the function on any other list of i32 values we might have in the future.

尽管此代码有效，但复制代码非常乏味且容易出错。 当我们想要更改代码时，我们还必须记住在多个地方更新代码。

为了消除这种重复，我们将通过定义一个函数来创建一个抽象，该函数对参数中传递的任何整数列表进行操作。 这个解决方案使我们的代码更加清晰，并让我们抽象地表达了在列表中查找最大数字的概念。

在清单 10-3 中，我们将查找最大数的代码提取到名为largest 的函数中。 然后我们调用该函数来查找清单 10-2 中两个列表中的最大数字。 我们还可以在将来可能拥有的任何其他 i32 值列表上使用该函数。

Filename: src/main.rs
``````
fn largest(list: &[i32]) -> &i32 {
    let mut largest = &list[0];

    for item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {}", result);

    let number_list = vec![102, 34, 6000, 89, 54, 2, 43, 8];

    let result = largest(&number_list);
    println!("The largest number is {}", result);
}
``````
Listing 10-3: Abstracted code to find the largest number in two lists

The largest function has a parameter called list, which represents any concrete slice of i32 values we might pass into the function. As a result, when we call the function, the code runs on the specific values that we pass in.

In summary, here are the steps we took to change the code from Listing 10-2 to Listing 10-3:

Identify duplicate code.
Extract the duplicate code into the body of the function and specify the inputs and return values of that code in the function signature.
Update the two instances of duplicated code to call the function instead.
Next, we’ll use these same steps with generics to reduce code duplication. In the same way that the function body can operate on an abstract list instead of specific values, generics allow code to operate on abstract types.

For example, say we had two functions: one that finds the largest item in a slice of i32 values and one that finds the largest item in a slice of char values. How would we eliminate that duplication? Let’s find out!

最大的函数有一个名为 list 的参数，它代表我们可能传递给函数的 i32 值的任何具体切片。 因此，当我们调用该函数时，代码会根据我们传入的特定值运行。

总之，我们将代码从清单 10-2 更改为清单 10-3 所采取的步骤如下：

识别重复代码。
将重复代码提取到函数主体中，并在函数签名中指定该代码的输入和返回值。
更新重复代码的两个实例以改为调用该函数。
接下来，我们将使用这些相同的步骤与泛型来减少代码重复。 就像函数体可以对抽象列表而不是特定值进行操作一样，泛型允许代码对抽象类型进行操作。

例如，假设我们有两个函数：一个函数查找 i32 值切片中的最大项目，另一个函数查找 char 值切片中的最大项目。 我们如何消除这种重复？ 让我们来看看吧！

## 10.1 Generic Data Types

We use generics to create definitions for items like function signatures or structs, which we can then use with many different concrete data types. Let’s first look at how to define functions, structs, enums, and methods using generics. Then we’ll discuss how generics affect code performance.

我们使用泛型来创建函数签名或结构等项目的定义，然后我们可以将其与许多不同的具体数据类型一起使用。 我们首先看看如何使用泛型定义函数、结构体、枚举和方法。 然后我们将讨论泛型如何影响代码性能。

### In Function Definitions
When defining a function that uses generics, we place the generics in the signature of the function where we would usually specify the data types of the parameters and return value. Doing so makes our code more flexible and provides more functionality to callers of our function while preventing code duplication.

Continuing with our largest function, Listing 10-4 shows two functions that both find the largest value in a slice. We'll then combine these into a single function that uses generics.

当定义使用泛型的函数时，我们将泛型放在函数的签名中，我们通常会在其中指定参数和返回值的数据类型。 这样做使我们的代码更加灵活，并为函数的调用者提供更多功能，同时防止代码重复。

继续我们的最大函数，清单 10-4 显示了两个函数，它们都找到切片中的最大值。 然后，我们将它们组合成一个使用泛型的函数。

Filename: src/main.rs
``````
fn largest_i32(list: &[i32]) -> &i32 {
    let mut largest = &list[0];

    for item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn largest_char(list: &[char]) -> &char {
    let mut largest = &list[0];

    for item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest_i32(&number_list);
    println!("The largest number is {}", result);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest_char(&char_list);
    println!("The largest char is {}", result);
}
``````
Listing 10-4: Two functions that differ only in their names and the types in their signatures

The largest_i32 function is the one we extracted in Listing 10-3 that finds the largest i32 in a slice. The largest_char function finds the largest char in a slice. The function bodies have the same code, so let’s eliminate the duplication by introducing a generic type parameter in a single function.

To parameterize the types in a new single function, we need to name the type parameter, just as we do for the value parameters to a function. You can use any identifier as a type parameter name. But we’ll use T because, by convention, type parameter names in Rust are short, often just a letter, and Rust’s type-naming convention is UpperCamelCase. Short for “type,” T is the default choice of most Rust programmers.

When we use a parameter in the body of the function, we have to declare the parameter name in the signature so the compiler knows what that name means. Similarly, when we use a type parameter name in a function signature, we have to declare the type parameter name before we use it. To define the generic largest function, place type name declarations inside angle brackets, <>, between the name of the function and the parameter list, like this:

fn largest<T>(list: &[T]) -> &T {
We read this definition as: the function largest is generic over some type T. This function has one parameter named list, which is a slice of values of type T. The largest function will return a reference to a value of the same type T.

Listing 10-5 shows the combined largest function definition using the generic data type in its signature. The listing also shows how we can call the function with either a slice of i32 values or char values. Note that this code won’t compile yet, but we’ll fix it later in this chapter.

Maximum_i32 函数是我们在清单 10-3 中提取的函数，用于查找切片中最大的 i32。 Maximum_char 函数查找切片中最大的字符。 函数体具有相同的代码，因此让我们通过在单个函数中引入泛型类型参数来消除重复。

要参数化新的单个函数中的类型，我们需要命名类型参数，就像我们为函数的值参数所做的那样。 您可以使用任何标识符作为类型参数名称。 但我们将使用 T，因为按照惯例，Rust 中的类型参数名称很短，通常只是一个字母，并且 Rust 的类型命名约定是 UpperCamelCase。 T 是“type”的缩写，是大多数 Rust 程序员的默认选择。

当我们在函数体中使用参数时，我们必须在签名中声明参数名称，以便编译器知道该名称的含义。 同样，当我们在函数签名中使用类型参数名称时，我们必须在使用它之前声明类型参数名称。 要定义通用最大函数，请将类型名称声明放在函数名称和参数列表之间的尖括号 <> 内，如下所示：

fn 最大<T>(列表: &[T]) -> &T {
我们将这个定义理解为：函数largest是某种类型T的泛型。该函数有一个名为list的参数，它是T类型值的切片。largest函数将返回对相同类型T的值的引用。

清单 10-5 显示了在其签名中使用通用数据类型的组合最大函数定义。 该清单还显示了如何使用 i32 值或 char 值的切片来调用该函数。 请注意，这段代码还无法编译，但我们将在本章稍后修复它。

Filename: src/main.rs
``````
This code does not compile!
fn largest<T>(list: &[T]) -> &T {
    let mut largest = &list[0];

    for item in list {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {}", result);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest(&char_list);
    println!("The largest char is {}", result);
}
``````
Listing 10-5: The largest function using generic type parameters; this doesn’t yet compile

If we compile this code right now, we’ll get this error:
``````
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0369]: binary operation `>` cannot be applied to type `&T`
 --> src/main.rs:5:17
  |
5 |         if item > largest {
  |            ---- ^ ------- &T
  |            |
  |            &T
  |
help: consider restricting type parameter `T`
  |
1 | fn largest<T: std::cmp::PartialOrd>(list: &[T]) -> &T {
  |             ++++++++++++++++++++++

For more information about this error, try `rustc --explain E0369`.
error: could not compile `chapter10` due to previous error
``````
The help text mentions std::cmp::PartialOrd, which is a trait, and we’re going to talk about traits in the next section. For now, know that this error states that the body of largest won’t work for all possible types that T could be. Because we want to compare values of type T in the body, we can only use types whose values can be ordered. To enable comparisons, the standard library has the std::cmp::PartialOrd trait that you can implement on types (see Appendix C for more on this trait). By following the help text's suggestion, we restrict the types valid for T to only those that implement PartialOrd and this example will compile, because the standard library implements PartialOrd on both i32 and char.

帮助文本提到了 std::cmp::PartialOrd，这是一个特征，我们将在下一节中讨论特征。 现在，请知道此错误表明最大的主体不适用于 T 可能的所有可能类型。 因为我们要比较主体中类型 T 的值，所以我们只能使用值可以排序的类型。 为了启用比较，标准库具有 std::cmp::PartialOrd 特征，您可以在类型上实现该特征（有关此特征的更多信息，请参阅附录 C）。 通过遵循帮助文本的建议，我们将对 T 有效的类型限制为仅实现 PartialOrd 的类型，并且此示例将编译，因为标准库在 i32 和 char 上实现 PartialOrd。

### In Struct Definitions
We can also define structs to use a generic type parameter in one or more fields using the <> syntax. Listing 10-6 defines a Point<T> struct to hold x and y coordinate values of any type.

Filename: src/main.rs
``````
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let integer = Point { x: 5, y: 10 };
    let float = Point { x: 1.0, y: 4.0 };
}
``````
Listing 10-6: A Point<T> struct that holds x and y values of type T

The syntax for using generics in struct definitions is similar to that used in function definitions. First, we declare the name of the type parameter inside angle brackets just after the name of the struct. Then we use the generic type in the struct definition where we would otherwise specify concrete data types.

Note that because we’ve used only one generic type to define Point<T>, this definition says that the Point<T> struct is generic over some type T, and the fields x and y are both that same type, whatever that type may be. If we create an instance of a Point<T> that has values of different types, as in Listing 10-7, our code won’t compile.

在结构定义中使用泛型的语法与在函数定义中使用的语法类似。 首先，我们在结构名称后面的尖括号内声明类型参数的名称。 然后，我们在结构定义中使用泛型类型，否则我们将指定具体的数据类型。

请注意，因为我们仅使用一种泛型类型来定义 Point<T>，所以该定义表明 Point<T> 结构对于某种类型 T 是泛型的，并且字段 x 和 y 都是相同的类型，无论该类型如何 或许。 如果我们创建一个具有不同类型值的 Point<T> 实例（如清单 10-7 所示），我们的代码将无法编译。

Filename: src/main.rs
``````
This code does not compile!
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let wont_work = Point { x: 5, y: 4.0 };
}
``````
Listing 10-7: The fields x and y must be the same type because both have the same generic data type T.

In this example, when we assign the integer value 5 to x, we let the compiler know that the generic type T will be an integer for this instance of Point<T>. Then when we specify 4.0 for y, which we’ve defined to have the same type as x, we’ll get a type mismatch error like this:

在此示例中，当我们将整数值 5 分配给 x 时，我们让编译器知道泛型类型 T 将是 Point<T> 实例的整数。 然后，当我们为 y 指定 4.0（我们已将其定义为与 x 具有相同类型）时，我们将收到如下类型不匹配错误：

``````
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0308]: mismatched types
 --> src/main.rs:7:38
  |
7 |     let wont_work = Point { x: 5, y: 4.0 };
  |                                      ^^^ expected integer, found floating-point number

For more information about this error, try `rustc --explain E0308`.
error: could not compile `chapter10` due to previous error
``````
To define a Point struct where x and y are both generics but could have different types, we can use multiple generic type parameters. For example, in Listing 10-8, we change the definition of Point to be generic over types T and U where x is of type T and y is of type U.

Filename: src/main.rs
``````
struct Point<T, U> {
    x: T,
    y: U,
}

fn main() {
    let both_integer = Point { x: 5, y: 10 };
    let both_float = Point { x: 1.0, y: 4.0 };
    let integer_and_float = Point { x: 5, y: 4.0 };
}
``````
Listing 10-8: A Point<T, U> generic over two types so that x and y can be values of different types

Now all the instances of Point shown are allowed! You can use as many generic type parameters in a definition as you want, but using more than a few makes your code hard to read. If you're finding you need lots of generic types in your code, it could indicate that your code needs restructuring into smaller pieces.

现在显示的所有 Point 实例都是允许的！ 您可以在定义中使用任意数量的泛型类型参数，但使用过多会使您的代码难以阅读。 如果您发现代码中需要大量泛型类型，则可能表明您的代码需要重组为更小的部分。

### In Enum Definitions
As we did with structs, we can define enums to hold generic data types in their variants. Let’s take another look at the Option<T> enum that the standard library provides, which we used in Chapter 6:

正如我们对结构所做的那样，我们可以定义枚举来保存其变体中的通用数据类型。 让我们再看一下标准库提供的 Option<T> 枚举，我们在第 6 章中使用了它：

``````
enum Option<T> {
    Some(T),
    None,
}
``````
This definition should now make more sense to you. As you can see, the Option<T> enum is generic over type T and has two variants: Some, which holds one value of type T, and a None variant that doesn’t hold any value. By using the Option<T> enum, we can express the abstract concept of an optional value, and because Option<T> is generic, we can use this abstraction no matter what the type of the optional value is.

这个定义现在对您来说应该更有意义了。 正如您所看到的，Option<T> 枚举是 T 类型的泛型，并且有两种变体：Some，它保存一个 T 类型的值，以及 None 变体，不保存任何值。 通过使用 Option<T> 枚举，我们可以表达可选值的抽象概念，并且由于 Option<T> 是泛型的，所以无论可选值是什么类型，我们都可以使用这个抽象。

Enums can use multiple generic types as well. The definition of the Result enum that we used in Chapter 9 is one example:
``````
enum Result<T, E> {
    Ok(T),
    Err(E),
}
``````
The Result enum is generic over two types, T and E, and has two variants: Ok, which holds a value of type T, and Err, which holds a value of type E. This definition makes it convenient to use the Result enum anywhere we have an operation that might succeed (return a value of some type T) or fail (return an error of some type E). In fact, this is what we used to open a file in Listing 9-3, where T was filled in with the type std::fs::File when the file was opened successfully and E was filled in with the type std::io::Error when there were problems opening the file.

When you recognize situations in your code with multiple struct or enum definitions that differ only in the types of the values they hold, you can avoid duplication by using generic types instead.

Result 枚举对于 T 和 E 两种类型是通用的，并且有两个变体：Ok，它保存类型 T 的值，Err，它保存类型 E 的值。此定义使得在任何地方使用 Result 枚举都变得很方便 我们有一个操作可能会成功（返回某种类型 T 的值）或失败（返回某种类型 E 的错误）。 事实上，这就是清单 9-3 中我们用来打开文件的方式，其中当文件打开成功时 T 填充为 std::fs::File 类型，E 填充为 std:: io::打开文件时出现问题时出错。

当您认识到代码中具有多个结构或枚举定义的情况仅在它们所保存的值的类型不同时，您可以通过使用泛型类型来避免重复。

### In Method Definitions
We can implement methods on structs and enums (as we did in Chapter 5) and use generic types in their definitions, too. Listing 10-9 shows the Point<T> struct we defined in Listing 10-6 with a method named x implemented on it.

我们可以在结构和枚举上实现方法（就像我们在第 5 章中所做的那样），并在它们的定义中使用泛型类型。 清单 10-9 显示了我们在清单 10-6 中定义的 Point<T> 结构，并在其上实现了名为 x 的方法。

Filename: src/main.rs
``````
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

fn main() {
    let p = Point { x: 5, y: 10 };

    println!("p.x = {}", p.x());
}
``````
Listing 10-9: Implementing a method named x on the Point<T> struct that will return a reference to the x field of type T

Here, we’ve defined a method named x on Point<T> that returns a reference to the data in the field x.

Note that we have to declare T just after impl so we can use T to specify that we’re implementing methods on the type Point<T>. By declaring T as a generic type after impl, Rust can identify that the type in the angle brackets in Point is a generic type rather than a concrete type. We could have chosen a different name for this generic parameter than the generic parameter declared in the struct definition, but using the same name is conventional. Methods written within an impl that declares the generic type will be defined on any instance of the type, no matter what concrete type ends up substituting for the generic type.

We can also specify constraints on generic types when defining methods on the type. We could, for example, implement methods only on Point<f32> instances rather than on Point<T> instances with any generic type. In Listing 10-10 we use the concrete type f32, meaning we don’t declare any types after impl.

在这里，我们在 Point<T> 上定义了一个名为 x 的方法，该方法返回对字段 x 中数据的引用。

请注意，我们必须在 impl 之后声明 T，以便我们可以使用 T 来指定我们正在类型 Point<T> 上实现方法。 通过在 impl 之后将 T 声明为泛型类型，Rust 可以识别 Point 中尖括号中的类型是泛型类型而不是具体类型。 我们可以为此泛型参数选择与结构定义中声明的泛型参数不同的名称，但使用相同的名称是常规做法。 在声明泛型类型的 impl 中编写的方法将在该类型的任何实例上定义，无论最终替换泛型类型的具体类型是什么。

我们还可以在类型上定义方法时指定泛型类型的约束。 例如，我们可以仅在 Point<f32> 实例上实现方法，而不是在任何泛型类型的 Point<T> 实例上实现方法。 在清单 10-10 中，我们使用具体类型 f32，这意味着我们在 impl 之后不声明任何类型。

Filename: src/main.rs
``````
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
``````
Listing 10-10: An impl block that only applies to a struct with a particular concrete type for the generic type parameter T

This code means the type Point<f32> will have a distance_from_origin method; other instances of Point<T> where T is not of type f32 will not have this method defined. The method measures how far our point is from the point at coordinates (0.0, 0.0) and uses mathematical operations that are available only for floating point types.

Generic type parameters in a struct definition aren’t always the same as those you use in that same struct’s method signatures. Listing 10-11 uses the generic types X1 and Y1 for the Point struct and X2 Y2 for the mixup method signature to make the example clearer. The method creates a new Point instance with the x value from the self Point (of type X1) and the y value from the passed-in Point (of type Y2).

此代码意味着类型 Point<f32> 将具有 distance_from_origin 方法； Point<T> 的其他实例（其中 T 不是 f32 类型）将不会定义此方法。 该方法测量我们的点距坐标 (0.0, 0.0) 处的点有多远，并使用仅适用于浮点类型的数学运算。

结构定义中的通用类型参数并不总是与在同一结构的方法签名中使用的参数相同。 清单 10-11 使用泛型类型 X1 和 Y1 作为 Point 结构，使用 X2 Y2 作为混合方法签名，以使示例更加清晰。 该方法使用来自自身 Point（类型为 X1）的 x 值和来自传入 Point（类型为 Y2）的 y 值创建一个新的 Point 实例。

Filename: src/main.rs
``````
struct Point<X1, Y1> {
    x: X1,
    y: Y1,
}

impl<X1, Y1> Point<X1, Y1> {
    fn mixup<X2, Y2>(self, other: Point<X2, Y2>) -> Point<X1, Y2> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
}

fn main() {
    let p1 = Point { x: 5, y: 10.4 };
    let p2 = Point { x: "Hello", y: 'c' };

    let p3 = p1.mixup(p2);

    println!("p3.x = {}, p3.y = {}", p3.x, p3.y);
}
``````
Listing 10-11: A method that uses generic types different from its struct’s definition

In main, we’ve defined a Point that has an i32 for x (with value 5) and an f64 for y (with value 10.4). The p2 variable is a Point struct that has a string slice for x (with value "Hello") and a char for y (with value c). Calling mixup on p1 with the argument p2 gives us p3, which will have an i32 for x, because x came from p1. The p3 variable will have a char for y, because y came from p2. The println! macro call will print p3.x = 5, p3.y = c.

The purpose of this example is to demonstrate a situation in which some generic parameters are declared with impl and some are declared with the method definition. Here, the generic parameters X1 and Y1 are declared after impl because they go with the struct definition. The generic parameters X2 and Y2 are declared after fn mixup, because they’re only relevant to the method.

Performance of Code Using Generics
You might be wondering whether there is a runtime cost when using generic type parameters. The good news is that using generic types won't make your program run any slower than it would with concrete types.

Rust accomplishes this by performing monomorphization of the code using generics at compile time. Monomorphization is the process of turning generic code into specific code by filling in the concrete types that are used when compiled. In this process, the compiler does the opposite of the steps we used to create the generic function in Listing 10-5: the compiler looks at all the places where generic code is called and generates code for the concrete types the generic code is called with.

在 main 中，我们定义了一个 Point，它的 x 为 i32（值为 5），y 为 f64（值为 10.4）。 p2 变量是一个 Point 结构，它具有 x 的字符串切片（值为“Hello”）和 y 的 char（值为 c）。 使用参数 p2 在 p1 上调用 mixup 会得到 p3，其中 x 将会有一个 i32，因为 x 来自 p1。 p3 变量将有一个 y 字符，因为 y 来自 p2。 打印！ 宏调用将打印 p3.x = 5, p3.y = c。

此示例的目的是演示一些泛型参数使用 impl 声明而另一些泛型参数使用方法定义声明的情况。 这里，通用参数 X1 和 Y1 在 impl 之后声明，因为它们与结构定义一致。 通用参数 X2 和 Y2 在 fn mixup 之后声明，因为它们仅与方法相关。

使用泛型的代码性能
您可能想知道使用泛型类型参数时是否存在运行时成本。 好消息是，使用泛型类型不会使程序运行速度比使用具体类型慢。

Rust 通过在编译时使用泛型执行代码的单态化来实现这一点。 单态化是通过填充编译时使用的具体类型将通用代码转换为特定代码的过程。 在此过程中，编译器执行与清单 10-5 中创建泛型函数相反的步骤：编译器查看调用泛型代码的所有位置，并为调用泛型代码的具体类型生成代码 。

Let’s look at how this works by using the standard library’s generic Option<T> enum:
``````
let integer = Some(5);
let float = Some(5.0);
``````
When Rust compiles this code, it performs monomorphization. During that process, the compiler reads the values that have been used in Option<T> instances and identifies two kinds of Option<T>: one is i32 and the other is f64. As such, it expands the generic definition of Option<T> into two definitions specialized to i32 and f64, thereby replacing the generic definition with the specific ones.

The monomorphized version of the code looks similar to the following (the compiler uses different names than what we’re using here for illustration):

当 Rust 编译此代码时，它会执行单态化。 在此过程中，编译器读取 Option<T> 实例中使用的值并识别两种 Option<T>：一种是 i32，另一种是 f64。 因此，它将 Option<T> 的通用定义扩展为专门用于 i32 和 f64 的两个定义，从而用特定定义替换通用定义。

代码的单态版本类似于以下内容（编译器使用的名称与我们此处用于说明的名称不同）：

Filename: src/main.rs
``````
enum Option_i32 {
    Some(i32),
    None,
}

enum Option_f64 {
    Some(f64),
    None,
}

fn main() {
    let integer = Option_i32::Some(5);
    let float = Option_f64::Some(5.0);
}
``````
The generic Option<T> is replaced with the specific definitions created by the compiler. Because Rust compiles generic code into code that specifies the type in each instance, we pay no runtime cost for using generics. When the code runs, it performs just as it would if we had duplicated each definition by hand. The process of monomorphization makes Rust’s generics extremely efficient at runtime.

通用 Option<T> 被编译器创建的特定定义替换。 因为 Rust 将泛型代码编译为指定每个实例中的类型的代码，所以我们无需为使用泛型支付运行时成本。 当代码运行时，它的执行效果就像我们手动复制每个定义一样。 单态化过程使得 Rust 的泛型在运行时非常高效。



## 10.2 Traits: Defining Shared Behavior
A trait defines functionality a particular type has and can share with other types. We can use traits to define shared behavior in an abstract way. We can use trait bounds to specify that a generic type can be any type that has certain behavior.

特征定义特定类型具有并可以与其他类型共享的功能。 我们可以使用特征以抽象的方式定义共享行为。 我们可以使用特征边界来指定泛型类型可以是具有特定行为的任何类型。

Note: Traits are similar to a feature often called interfaces in other languages, although with some differences.

### Defining a Trait
A type’s behavior consists of the methods we can call on that type. Different types share the same behavior if we can call the same methods on all of those types. Trait definitions are a way to group method signatures together to define a set of behaviors necessary to accomplish some purpose.

For example, let’s say we have multiple structs that hold various kinds and amounts of text: a NewsArticle struct that holds a news story filed in a particular location and a Tweet that can have at most 280 characters along with metadata that indicates whether it was a new tweet, a retweet, or a reply to another tweet.

We want to make a media aggregator library crate named aggregator that can display summaries of data that might be stored in a NewsArticle or Tweet instance. To do this, we need a summary from each type, and we’ll request that summary by calling a summarize method on an instance. Listing 10-12 shows the definition of a public Summary trait that expresses this behavior.

类型的行为由我们可以在该类型上调用的方法组成。 如果我们可以对所有这些类型调用相同的方法，那么不同的类型就会共享相同的行为。 特征定义是将方法签名分组在一起以定义实现某些目的所需的一组行为的方法。

例如，假设我们有多个结构体，用于保存各种类型和数量的文本：一个 NewsArticle 结构体，用于保存在特定位置归档的新闻报道；以及一条最多可包含 280 个字符的推文，以及指示它是否是一条消息的元数据。 新推文、转发或对另一条推文的回复。

我们想要创建一个名为 aggregator 的媒体聚合器库 crate，它可以显示可能存储在 NewsArticle 或 Tweet 实例中的数据摘要。 为此，我们需要每种类型的摘要，并且我们将通过调用实例上的 summarize 方法来请求该摘要。 清单 10-12 显示了表达此行为的公共摘要特征的定义。

Filename: src/lib.rs
``````
pub trait Summary {
    fn summarize(&self) -> String;
}
``````
Listing 10-12: A Summary trait that consists of the behavior provided by a summarize method

Here, we declare a trait using the trait keyword and then the trait’s name, which is Summary in this case. We’ve also declared the trait as pub so that crates depending on this crate can make use of this trait too, as we’ll see in a few examples. Inside the curly brackets, we declare the method signatures that describe the behaviors of the types that implement this trait, which in this case is fn summarize(&self) -> String.

After the method signature, instead of providing an implementation within curly brackets, we use a semicolon. Each type implementing this trait must provide its own custom behavior for the body of the method. The compiler will enforce that any type that has the Summary trait will have the method summarize defined with this signature exactly.

A trait can have multiple methods in its body: the method signatures are listed one per line and each line ends in a semicolon.

在这里，我们使用 Trait 关键字声明一个特征，然后使用该特征的名称，在本例中为 Summary。 我们还将该特征声明为 pub，以便依赖于该箱子的箱子也可以利用该特征，正如我们将在几个示例中看到的那样。 在大括号内，我们声明了描述实现此特征的类型的行为的方法签名，在本例中为 fn summarize(&self) -> String。

在方法签名之后，我们不使用大括号内提供实现，而是使用分号。 实现此特征的每种类型都必须为方法体提供自己的自定义行为。 编译器将强制任何具有 Summary 特征的类型都将使用此签名准确定义的方法 summarise。

特征的主体中可以有多个方法：每行列出一个方法签名，并且每行以分号结尾。

### Implementing a Trait on a Type
Now that we’ve defined the desired signatures of the Summary trait’s methods, we can implement it on the types in our media aggregator. Listing 10-13 shows an implementation of the Summary trait on the NewsArticle struct that uses the headline, the author, and the location to create the return value of summarize. For the Tweet struct, we define summarize as the username followed by the entire text of the tweet, assuming that tweet content is already limited to 280 characters.

现在我们已经定义了 Summary 特征方法的所需签名，我们可以在媒体聚合器中的类型上实现它。 清单 10-13 显示了 NewsArticle 结构上 Summary 特征的实现，它使用标题、作者和位置来创建 summarize 的返回值。 对于 Tweet 结构，我们将 summarise 定义为用户名后跟推文的整个文本，假设推文内容已限制为 280 个字符。

Filename: src/lib.rs
``````
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
``````
Listing 10-13: Implementing the Summary trait on the NewsArticle and Tweet types

Implementing a trait on a type is similar to implementing regular methods. The difference is that after impl, we put the trait name we want to implement, then use the for keyword, and then specify the name of the type we want to implement the trait for. Within the impl block, we put the method signatures that the trait definition has defined. Instead of adding a semicolon after each signature, we use curly brackets and fill in the method body with the specific behavior that we want the methods of the trait to have for the particular type.

Now that the library has implemented the Summary trait on NewsArticle and Tweet, users of the crate can call the trait methods on instances of NewsArticle and Tweet in the same way we call regular methods. The only difference is that the user must bring the trait into scope as well as the types. Here’s an example of how a binary crate could use our aggregator library crate:

在类型上实现特征与实现常规方法类似。 不同的是，在 impl 之后，我们放置我们想要实现的特征名称，然后使用 for 关键字，然后指定我们想要实现特征的类型的名称。 在 impl 块中，我们放置了特征定义所定义的方法签名。 我们不是在每个签名后添加分号，而是使用大括号并使用我们希望特征方法对于特定类型具有的特定行为填充方法体。

现在，库已经在 NewsArticle 和 Tweet 上实现了 Summary 特征，板条箱的用户可以以与我们调用常规方法相同的方式在 NewsArticle 和 Tweet 实例上调用特征方法。 唯一的区别是用户必须将特征以及类型纳入范围。 以下是二进制板条箱如何使用我们的聚合器库板条箱的示例：

``````
use aggregator::{Summary, Tweet};

fn main() {
    let tweet = Tweet {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        retweet: false,
    };

    println!("1 new tweet: {}", tweet.summarize());
}
``````
This code prints 1 new tweet: horse_ebooks: of course, as you probably already know, people.

Other crates that depend on the aggregator crate can also bring the Summary trait into scope to implement Summary on their own types. One restriction to note is that we can implement a trait on a type only if at least one of the trait or the type is local to our crate. For example, we can implement standard library traits like Display on a custom type like Tweet as part of our aggregator crate functionality, because the type Tweet is local to our aggregator crate. We can also implement Summary on Vec<T> in our aggregator crate, because the trait Summary is local to our aggregator crate.

But we can’t implement external traits on external types. For example, we can’t implement the Display trait on Vec<T> within our aggregator crate, because Display and Vec<T> are both defined in the standard library and aren’t local to our aggregator crate. This restriction is part of a property called coherence, and more specifically the orphan rule, so named because the parent type is not present. This rule ensures that other people’s code can’t break your code and vice versa. Without the rule, two crates could implement the same trait for the same type, and Rust wouldn’t know which implementation to use.

依赖于聚合器 crate 的其他 crate 也可以将 Summary 特征纳入范围，以在自己的类型上实现 Summary。 需要注意的一个限制是，只有当特征或类型中至少有一个是我们的板条箱本地的时，我们才能在类型上实现特征。 例如，我们可以在自定义类型（如 Tweet）上实现标准库特征（如 Display），作为聚合器箱功能的一部分，因为类型 Tweet 是聚合器箱的本地类型。 我们还可以在聚合器箱中的 Vec<T> 上实现 Summary，因为特征 Summary 是聚合器箱的本地特征。

但我们不能在外部类型上实现外部特征。 例如，我们无法在聚合器板条箱内的 Vec<T> 上实现 Display 特征，因为 Display 和 Vec<T> 都是在标准库中定义的，并且不是我们聚合器板条箱本地的。 此限制是称为一致性的属性的一部分，更具体地说是孤立规则，如此命名是因为父类型不存在。 这条规则确保其他人的代码不能破坏你的代码，反之亦然。 如果没有规则，两个板条箱可以为同一类型实现相同的特征，而 Rust 不知道要使用哪个实现。

### Default Implementations
Sometimes it’s useful to have default behavior for some or all of the methods in a trait instead of requiring implementations for all methods on every type. Then, as we implement the trait on a particular type, we can keep or override each method’s default behavior.

In Listing 10-14 we specify a default string for the summarize method of the Summary trait instead of only defining the method signature, as we did in Listing 10-12.

有时，为特征中的部分或全部方法提供默认行为很有用，而不是要求每种类型的所有方法都实现。 然后，当我们在特定类型上实现该特征时，我们可以保留或覆盖每个方法的默认行为。

在清单10-14中，我们为Summary特征的summary方法指定了一个默认字符串，而不是像清单10-12中那样只定义方法签名。

Filename: src/lib.rs
``````
pub trait Summary {
    fn summarize(&self) -> String {
        String::from("(Read more...)")
    }
}
``````
Listing 10-14: Defining a Summary trait with a default implementation of the summarize method

To use a default implementation to summarize instances of NewsArticle, we specify an empty impl block with impl Summary for NewsArticle {}.

Even though we’re no longer defining the summarize method on NewsArticle directly, we’ve provided a default implementation and specified that NewsArticle implements the Summary trait. As a result, we can still call the summarize method on an instance of NewsArticle, like this:

要使用默认实现来汇总 NewsArticle 的实例，我们使用 impl Summary for NewsArticle {} 指定一个空的 impl 块。

尽管我们不再直接在 NewsArticle 上定义 Summarize 方法，但我们提供了默认实现并指定 NewsArticle 实现 Summary 特征。 因此，我们仍然可以在 NewsArticle 实例上调用 summarize 方法，如下所示：

``````
    let article = NewsArticle {
        headline: String::from("Penguins win the Stanley Cup Championship!"),
        location: String::from("Pittsburgh, PA, USA"),
        author: String::from("Iceburgh"),
        content: String::from(
            "The Pittsburgh Penguins once again are the best \
             hockey team in the NHL.",
        ),
    };

    println!("New article available! {}", article.summarize());
This code prints New article available! (Read more...).
``````

Creating a default implementation doesn’t require us to change anything about the implementation of Summary on Tweet in Listing 10-13. The reason is that the syntax for overriding a default implementation is the same as the syntax for implementing a trait method that doesn’t have a default implementation.

Default implementations can call other methods in the same trait, even if those other methods don’t have a default implementation. In this way, a trait can provide a lot of useful functionality and only require implementors to specify a small part of it. For example, we could define the Summary trait to have a summarize_author method whose implementation is required, and then define a summarize method that has a default implementation that calls the summarize_author method:

创建默认实现不需要我们对清单 10-13 中的 Summary on Tweet 的实现进行任何更改。 原因是覆盖默认实现的语法与实现没有默认实现的特征方法的语法相同。

默认实现可以调用同一特征中的其他方法，即使这些其他方法没有默认实现。 通过这种方式，一个特征可以提供很多有用的功能，并且只需要实现者指定其中的一小部分。 例如，我们可以将 Summary 特征定义为具有需要实现的 summarise_author 方法，然后定义一个具有调用 summarise_author 方法的默认实现的 summarise 方法：

``````
pub trait Summary {
    fn summarize_author(&self) -> String;

    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
}
``````
To use this version of Summary, we only need to define summarize_author when we implement the trait on a type:
``````
impl Summary for Tweet {
    fn summarize_author(&self) -> String {
        format!("@{}", self.username)
    }
}
``````
After we define summarize_author, we can call summarize on instances of the Tweet struct, and the default implementation of summarize will call the definition of summarize_author that we’ve provided. Because we’ve implemented summarize_author, the Summary trait has given us the behavior of the summarize method without requiring us to write any more code.

定义summary_author后，我们可以在Tweet结构体的实例上调用summary，并且summary的默认实现将调用我们提供的summary_author的定义。 因为我们已经实现了summary_author，所以Summary 特征为我们提供了summary 方法的行为，而无需我们编写更多代码。

``````
    let tweet = Tweet {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        retweet: false,
    };

    println!("1 new tweet: {}", tweet.summarize());
This code prints 1 new tweet: (Read more from @horse_ebooks...).
``````
Note that it isn’t possible to call the default implementation from an overriding implementation of that same method.

### Traits as Parameters
Now that you know how to define and implement traits, we can explore how to use traits to define functions that accept many different types. We'll use the Summary trait we implemented on the NewsArticle and Tweet types in Listing 10-13 to define a notify function that calls the summarize method on its item parameter, which is of some type that implements the Summary trait. To do this, we use the impl Trait syntax, like this:

现在您已经知道如何定义和实现特征，我们可以探索如何使用特征来定义接受许多不同类型的函数。 我们将使用在清单 10-13 中的 NewsArticle 和 Tweet 类型上实现的 Summary 特征来定义一个通知函数，该函数在其 item 参数上调用 Summarize 方法，该参数是实现 Summary 特征的某种类型。 为此，我们使用 impl Trait 语法，如下所示：

``````
pub fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
``````
Instead of a concrete type for the item parameter, we specify the impl keyword and the trait name. This parameter accepts any type that implements the specified trait. In the body of notify, we can call any methods on item that come from the Summary trait, such as summarize. We can call notify and pass in any instance of NewsArticle or Tweet. Code that calls the function with any other type, such as a String or an i32, won’t compile because those types don’t implement Summary.

我们指定 impl 关键字和特征名称，而不是 item 参数的具体类型。 此参数接受实现指定特征的任何类型。 在notify的主体中，我们可以调用来自Summary特征的item上的任何方法，例如summary。 我们可以调用notify并传入NewsArticle或Tweet的任何实例。 使用任何其他类型（例如 String 或 i32）调用该函数的代码将无法编译，因为这些类型不实现 Summary。


### Trait Bound Syntax
The impl Trait syntax works for straightforward cases but is actually syntax sugar for a longer form known as a trait bound; it looks like this:

impl Trait 语法适用于简单的情况，但实际上是称为特征绑定的较长形式的语法糖； 它看起来像这样：

``````
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
``````
This longer form is equivalent to the example in the previous section but is more verbose. We place trait bounds with the declaration of the generic type parameter after a colon and inside angle brackets.

The impl Trait syntax is convenient and makes for more concise code in simple cases, while the fuller trait bound syntax can express more complexity in other cases. For example, we can have two parameters that implement Summary. Doing so with the impl Trait syntax looks like this:

这种较长的形式相当于上一节中的示例，但更冗长。 我们将特征边界与泛型类型参数的声明放在冒号后面和尖括号内。

impl Trait 语法很方便，在简单情况下可以使代码更简洁，而更完整的特征绑定语法可以在其他情况下表达更复杂的情况。 例如，我们可以有两个实现 Summary 的参数。 使用 impl Trait 语法执行此操作如下所示：

``````
pub fn notify(item1: &impl Summary, item2: &impl Summary) {
``````
Using impl Trait is appropriate if we want this function to allow item1 and item2 to have different types (as long as both types implement Summary). If we want to force both parameters to have the same type, however, we must use a trait bound, like this:

如果我们希望此函数允许 item1 和 item2 具有不同的类型（只要这两种类型都实现 Summary），那么使用 impl Trait 是合适的。 但是，如果我们想强制两个参数具有相同的类型，则必须使用特征绑定，如下所示：

``````
pub fn notify<T: Summary>(item1: &T, item2: &T) {
``````
The generic type T specified as the type of the item1 and item2 parameters constrains the function such that the concrete type of the value passed as an argument for item1 and item2 must be the same.

指定为 item1 和 item2 参数类型的泛型类型 T 会限制该函数，使得作为 item1 和 item2 参数传递的值的具体类型必须相同。

### Specifying Multiple Trait Bounds with the + Syntax
We can also specify more than one trait bound. Say we wanted notify to use display formatting as well as summarize on item: we specify in the notify definition that item must implement both Display and Summary. We can do so using the + syntax:
``````
pub fn notify(item: &(impl Summary + Display)) {
``````
The + syntax is also valid with trait bounds on generic types:
``````
pub fn notify<T: Summary + Display>(item: &T) {
``````
With the two trait bounds specified, the body of notify can call summarize and use {} to format item.

### Clearer Trait Bounds with where Clauses
Using too many trait bounds has its downsides. Each generic has its own trait bounds, so functions with multiple generic type parameters can contain lots of trait bound information between the function’s name and its parameter list, making the function signature hard to read. For this reason, Rust has alternate syntax for specifying trait bounds inside a where clause after the function signature. So instead of writing this:

使用太多的特征界限有其缺点。 每个泛型都有自己的特征边界，因此具有多个泛型类型参数的函数可以在函数名称与其参数列表之间包含大量特征绑定信息，从而使函数签名难以阅读。 因此，Rust 有替代语法用于在函数签名后的 where 子句内指定特征边界。 所以不要这样写：

``````
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {
``````
we can use a where clause, like this:
``````
fn some_function<T, U>(t: &T, u: &U) -> i32
where
    T: Display + Clone,
    U: Clone + Debug,
{
``````
This function’s signature is less cluttered: the function name, parameter list, and return type are close together, similar to a function without lots of trait bounds.

Returning Types that Implement Traits
We can also use the impl Trait syntax in the return position to return a value of some type that implements a trait, as shown here:

该函数的签名不太混乱：函数名称、参数列表和返回类型紧密结合在一起，类似于没有大量特征边界的函数。

返回实现特征的类型
我们还可以在返回位置使用 impl Trait 语法来返回实现特征的某种类型的值，如下所示：

``````
fn returns_summarizable() -> impl Summary {
    Tweet {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        retweet: false,
    }
}
``````
By using impl Summary for the return type, we specify that the returns_summarizable function returns some type that implements the Summary trait without naming the concrete type. In this case, returns_summarizable returns a Tweet, but the code calling this function doesn’t need to know that.

The ability to specify a return type only by the trait it implements is especially useful in the context of closures and iterators, which we cover in Chapter 13. Closures and iterators create types that only the compiler knows or types that are very long to specify. The impl Trait syntax lets you concisely specify that a function returns some type that implements the Iterator trait without needing to write out a very long type.

However, you can only use impl Trait if you’re returning a single type. For example, this code that returns either a NewsArticle or a Tweet with the return type specified as impl Summary wouldn’t work:

通过使用 impl Summary 作为返回类型，我们指定 returns_summarizes 函数返回某种实现 Summary 特征的类型，而无需命名具体类型。 在这种情况下，returns_summarizable 返回一条推文，但调用此函数的代码不需要知道这一点。

仅通过其实现的特征来指定返回类型的能力在闭包和迭代器的上下文中特别有用，我们将在第 13 章中介绍这一点。闭包和迭代器创建只有编译器知道的类型或需要很长的类型。 impl Trait 语法可让您简洁地指定函数返回某种实现 Iterator 特征的类型，而无需写出非常长的类型。

但是，如果您返回单一类型，则只能使用 impl Trait。 例如，返回 NewsArticle 或 Tweet（返回类型指定为 impl Summary）的代码将不起作用：

``````
This code does not compile!
fn returns_summarizable(switch: bool) -> impl Summary {
    if switch {
        NewsArticle {
            headline: String::from(
                "Penguins win the Stanley Cup Championship!",
            ),
            location: String::from("Pittsburgh, PA, USA"),
            author: String::from("Iceburgh"),
            content: String::from(
                "The Pittsburgh Penguins once again are the best \
                 hockey team in the NHL.",
            ),
        }
    } else {
        Tweet {
            username: String::from("horse_ebooks"),
            content: String::from(
                "of course, as you probably already know, people",
            ),
            reply: false,
            retweet: false,
        }
    }
}
``````
Returning either a NewsArticle or a Tweet isn’t allowed due to restrictions around how the impl Trait syntax is implemented in the compiler. We’ll cover how to write a function with this behavior in the “Using Trait Objects That Allow for Values of Different Types” section of Chapter 17.

由于编译器中如何实现 impl Trait 语法的限制，不允许返回 NewsArticle 或 Tweet。 我们将在第 17 章的“使用允许不同类型值的 Trait 对象”部分介绍如何编写具有这种行为的函数。

### Using Trait Bounds to Conditionally Implement Methods
By using a trait bound with an impl block that uses generic type parameters, we can implement methods conditionally for types that implement the specified traits. For example, the type Pair<T> in Listing 10-15 always implements the new function to return a new instance of Pair<T> (recall from the “Defining Methods” section of Chapter 5 that Self is a type alias for the type of the impl block, which in this case is Pair<T>). But in the next impl block, Pair<T> only implements the cmp_display method if its inner type T implements the PartialOrd trait that enables comparison and the Display trait that enables printing.

通过使用与使用泛型类型参数的 impl 块绑定的特征，我们可以有条件地为实现指定特征的类型实现方法。 例如，清单 10-15 中的类型 Pair<T> 总是实现 new 函数来返回 Pair<T> 的新实例（回想一下第 5 章的“定义方法”部分，Self 是该类型的类型别名） impl 块，在本例中为 Pair<T>）。 但在下一个 impl 块中，如果 Pair<T> 的内部类型 T 实现了启用比较的 PartialOrd 特征和启用打印的 Display 特征，则 Pair<T> 仅实现 cmp_display 方法。

Filename: src/lib.rs
``````
use std::fmt::Display;

struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self { x, y }
    }
}

impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
``````
Listing 10-15: Conditionally implementing methods on a generic type depending on trait bounds

We can also conditionally implement a trait for any type that implements another trait. Implementations of a trait on any type that satisfies the trait bounds are called blanket implementations and are extensively used in the Rust standard library. For example, the standard library implements the ToString trait on any type that implements the Display trait. The impl block in the standard library looks similar to this code:

我们还可以有条件地为任何实现另一个特征的类型实现一个特征。 满足特征边界的任何类型上特征的实现称为毯子实现，并在 Rust 标准库中广泛使用。 例如，标准库在任何实现 Display 特征的类型上实现 ToString 特征。 标准库中的 impl 块看起来类似于以下代码：

``````
impl<T: Display> ToString for T {
    // --snip--
}
``````
Because the standard library has this blanket implementation, we can call the to_string method defined by the ToString trait on any type that implements the Display trait. For example, we can turn integers into their corresponding String values like this because integers implement Display:

由于标准库具有此一揽子实现，因此我们可以在实现 Display 特征的任何类型上调用由 ToString 特征定义的 to_string 方法。 例如，我们可以将整数转换为相应的 String 值，如下所示，因为整数实现了 Display：

``````
let s = 3.to_string();
``````
Blanket implementations appear in the documentation for the trait in the “Implementors” section.

Traits and trait bounds let us write code that uses generic type parameters to reduce duplication but also specify to the compiler that we want the generic type to have particular behavior. The compiler can then use the trait bound information to check that all the concrete types used with our code provide the correct behavior. In dynamically typed languages, we would get an error at runtime if we called a method on a type which didn’t define the method. But Rust moves these errors to compile time so we’re forced to fix the problems before our code is even able to run. Additionally, we don’t have to write code that checks for behavior at runtime because we’ve already checked at compile time. Doing so improves performance without having to give up the flexibility of generics.

特征和特征边界让我们可以编写使用泛型类型参数来减少重复的代码，同时也向编译器指定我们希望泛型类型具有特定的行为。 然后，编译器可以使用特征绑定信息来检查与我们的代码一起使用的所有具体类型是否提供了正确的行为。 在动态类型语言中，如果我们调用未定义该方法的类型的方法，则会在运行时收到错误。 但 Rust 将这些错误移至编译时，因此我们被迫在代码运行之前修复这些问题。 此外，我们不必编写在运行时检查行为的代码，因为我们已经在编译时进行了检查。 这样做可以提高性能，而不必放弃泛型的灵活性。

## 10.3 验证带生命周期的引用(Validating References with Lifetimes)

Lifetimes are another kind of generic that we’ve already been using. Rather than ensuring that a type has the behavior we want, lifetimes ensure that references are valid as long as we need them to be.

One detail we didn’t discuss in the “References and Borrowing” section in Chapter 4 is that every reference in Rust has a lifetime, which is the scope for which that reference is valid. Most of the time, lifetimes are implicit and inferred, just like most of the time, types are inferred. We must only annotate types when multiple types are possible. In a similar way, we must annotate lifetimes when the lifetimes of references could be related in a few different ways. Rust requires us to annotate the relationships using generic lifetime parameters to ensure the actual references used at runtime will definitely be valid.

Annotating lifetimes is not a concept most other programming languages have, so this is going to feel unfamiliar. Although we won’t cover lifetimes in their entirety in this chapter, we’ll discuss common ways you might encounter lifetime syntax so you can get comfortable with the concept.

生命周期是我们已经使用过的另一种泛型。 生命周期不是确保类型具有我们想要的行为，而是确保引用在我们需要的时间内有效。

我们在第 4 章的“引用和借用”部分中没有讨论的一个细节是 Rust 中的每个引用都有一个生命周期，即该引用的有效范围。 大多数时候，生命周期是隐式的和推断的，就像大多数时候类型是推断的一样。 仅当可能存在多种类型时，我们才必须对类型进行注释。 以类似的方式，当引用的生命周期可以通过几种不同的方式关联时，我们必须注释生命周期。 Rust 要求我们使用通用生命周期参数来注释关系，以确保运行时使用的实际引用肯定是有效的。

注释生命周期并不是大多数其他编程语言所具有的概念，因此这会让人感到陌生。 尽管我们不会在本章中完整地介绍生命周期，但我们将讨论您可能遇到生命周期语法的常见方式，以便您能够熟悉这个概念。

### Preventing Dangling References with Lifetimes
The main aim of lifetimes is to prevent dangling references, which cause a program to reference data other than the data it’s intended to reference. Consider the program in Listing 10-16, which has an outer scope and an inner scope.

生命周期的主要目的是防止悬空引用，悬空引用会导致程序引用其打算引用的数据之外的数据。 考虑清单 10-16 中的程序，它有一个外部作用域和一个内部作用域。

``````
This code does not compile!
fn main() {
    let r;

    {
        let x = 5;
        r = &x;
    }

    println!("r: {}", r);
}
``````
Listing 10-16: An attempt to use a reference whose value has gone out of scope

Note: The examples in Listings 10-16, 10-17, and 10-23 declare variables without giving them an initial value, so the variable name exists in the outer scope. At first glance, this might appear to be in conflict with Rust’s having no null values. However, if we try to use a variable before giving it a value, we’ll get a compile-time error, which shows that Rust indeed does not allow null values.

The outer scope declares a variable named r with no initial value, and the inner scope declares a variable named x with the initial value of 5. Inside the inner scope, we attempt to set the value of r as a reference to x. Then the inner scope ends, and we attempt to print the value in r. This code won’t compile because what the value r is referring to has gone out of scope before we try to use it. Here is the error message:

注意：清单 10-16、10-17 和 10-23 中的示例声明变量时没有赋予它们初始值，因此变量名称存在于外部作用域中。 乍一看，这似乎与 Rust 没有空值相冲突。 然而，如果我们在给变量赋值之前尝试使用它，我们会得到一个编译时错误，这表明 Rust 确实不允许空值。

外部作用域声明了一个名为 r 的变量，没有初始值，内部作用域声明了一个名为 x 的变量，初始值为 5。在内部作用域内，我们尝试将 r 的值设置为对 x 的引用。 然后内部作用域结束，我们尝试打印 r 中的值。 这段代码不会编译，因为在我们尝试使用它之前，r 所指的值已经超出了范围。 这是错误消息：

``````
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0597]: `x` does not live long enough
 --> src/main.rs:6:13
  |
6 |         r = &x;
  |             ^^ borrowed value does not live long enough
7 |     }
  |     - `x` dropped here while still borrowed
8 |
9 |     println!("r: {}", r);
  |                       - borrow later used here

For more information about this error, try `rustc --explain E0597`.
error: could not compile `chapter10` due to previous error
``````
The variable x doesn’t “live long enough.” The reason is that x will be out of scope when the inner scope ends on line 7. But r is still valid for the outer scope; because its scope is larger, we say that it “lives longer.” If Rust allowed this code to work, r would be referencing memory that was deallocated when x went out of scope, and anything we tried to do with r wouldn’t work correctly. So how does Rust determine that this code is invalid? It uses a borrow checker.

变量 x “活得不够长”。 原因是当内部作用域在第 7 行结束时，x 将超出作用域。但 r 对于外部作用域仍然有效； 因为它的范围更大，所以我们说它“寿命更长”。 如果 Rust 允许这段代码工作，r 将引用当 x 超出范围时被释放的内存，并且我们尝试对 r 执行的任何操作都将无法正常工作。 那么 Rust 是如何判断这段代码无效的呢？ 它使用借用检查器。

### The Borrow Checker
The Rust compiler has a borrow checker that compares scopes to determine whether all borrows are valid. Listing 10-17 shows the same code as Listing 10-16 but with annotations showing the lifetimes of the variables.

Rust 编译器有一个借用检查器，可以比较范围以确定所有借用是否有效。 清单 10-17 显示了与清单 10-16 相同的代码，但带有显示变量生命周期的注释。
``````
This code does not compile!
fn main() {
    let r;                // ---------+-- 'a
                          //          |
    {                     //          |
        let x = 5;        // -+-- 'b  |
        r = &x;           //  |       |
    }                     // -+       |
                          //          |
    println!("r: {}", r); //          |
}                         // ---------+
``````
Listing 10-17: Annotations of the lifetimes of r and x, named 'a and 'b, respectively

Here, we’ve annotated the lifetime of r with 'a and the lifetime of x with 'b. As you can see, the inner 'b block is much smaller than the outer 'a lifetime block. At compile time, Rust compares the size of the two lifetimes and sees that r has a lifetime of 'a but that it refers to memory with a lifetime of 'b. The program is rejected because 'b is shorter than 'a: the subject of the reference doesn’t live as long as the reference.

在这里，我们用“a”注释了 r 的生命周期，用“b”注释了 x 的生命周期。 正如您所看到的，内部 'b 块比外部 'a 生命周期块小得多。 在编译时，Rust 比较两个生命周期的大小，发现 r 的生命周期为“a”，但它引用的内存的生命周期为“b”。 该程序被拒绝，因为 'b 比 'a 短：引用的主题没有引用那么长。

``````
Listing 10-18 fixes the code so it doesn’t have a dangling reference and compiles without any errors.

fn main() {
    let x = 5;            // ----------+-- 'b
                          //           |
    let r = &x;           // --+-- 'a  |
                          //   |       |
    println!("r: {}", r); //   |       |
                          // --+       |
}                         // ----------+
``````
Listing 10-18: A valid reference because the data has a longer lifetime than the reference

Here, x has the lifetime 'b, which in this case is larger than 'a. This means r can reference x because Rust knows that the reference in r will always be valid while x is valid.

Now that you know where the lifetimes of references are and how Rust analyzes lifetimes to ensure references will always be valid, let’s explore generic lifetimes of parameters and return values in the context of functions.

这里，x 的生命周期为 'b，在本例中大于 'a。 这意味着 r 可以引用 x，因为 Rust 知道当 x 有效时，r 中的引用将始终有效。

现在您已经了解了引用的生命周期在哪里以及 Rust 如何分析生命周期以确保引用始终有效，让我们在函数上下文中探索参数和返回值的通用生命周期。

### Generic Lifetimes in Functions
We’ll write a function that returns the longer of two string slices. This function will take two string slices and return a single string slice. After we’ve implemented the longest function, the code in Listing 10-19 should print The longest string is abcd.

我们将编写一个函数，返回两个字符串切片中较长的一个。 该函数将采用两个字符串切片并返回一个字符串切片。 在我们实现了最长的函数之后，清单 10-19 中的代码应该打印出最长的字符串是 abcd。

Filename: src/main.rs
``````
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest(string1.as_str(), string2);
    println!("The longest string is {}", result);
}
``````
Listing 10-19: A main function that calls the longest function to find the longer of two string slices

Note that we want the function to take string slices, which are references, rather than strings, because we don’t want the longest function to take ownership of its parameters. Refer to the “String Slices as Parameters” section in Chapter 4 for more discussion about why the parameters we use in Listing 10-19 are the ones we want.

请注意，我们希望函数采用字符串切片，它们是引用，而不是字符串，因为我们不希望最长的函数获得其参数的所有权。 请参阅第 4 章中的“字符串切片作为参数”部分，了解有关为什么清单 10-19 中使用的参数是我们想要的参数的更多讨论。

If we try to implement the longest function as shown in Listing 10-20, it won’t compile.

Filename: src/main.rs
``````
This code does not compile!
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
``````
Listing 10-20: An implementation of the longest function that returns the longer of two string slices but does not yet compile

Instead, we get the following error that talks about lifetimes:
``````
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0106]: missing lifetime specifier
 --> src/main.rs:9:33
  |
9 | fn longest(x: &str, y: &str) -> &str {
  |               ----     ----     ^ expected named lifetime parameter
  |
  = help: this function's return type contains a borrowed value, but the signature does not say whether it is borrowed from `x` or `y`
help: consider introducing a named lifetime parameter
  |
9 | fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
  |           ++++     ++          ++          ++

For more information about this error, try `rustc --explain E0106`.
error: could not compile `chapter10` due to previous error
``````
The help text reveals that the return type needs a generic lifetime parameter on it because Rust can’t tell whether the reference being returned refers to x or y. Actually, we don’t know either, because the if block in the body of this function returns a reference to x and the else block returns a reference to y!

When we’re defining this function, we don’t know the concrete values that will be passed into this function, so we don’t know whether the if case or the else case will execute. We also don’t know the concrete lifetimes of the references that will be passed in, so we can’t look at the scopes as we did in Listings 10-17 and 10-18 to determine whether the reference we return will always be valid. The borrow checker can’t determine this either, because it doesn’t know how the lifetimes of x and y relate to the lifetime of the return value. To fix this error, we’ll add generic lifetime parameters that define the relationship between the references so the borrow checker can perform its analysis.

帮助文本显示返回类型需要一个通用的生命周期参数，因为 Rust 无法判断返回的引用是引用 x 还是 y。 实际上，我们也不知道，因为该函数体中的 if 块返回对 x 的引用，而 else 块返回对 y 的引用！

当我们定义这个函数时，我们不知道将传递给这个函数的具体值，因此我们不知道是否会执行 if 情况或 else 情况。 我们也不知道将传入的引用的具体生命周期，因此我们无法像清单 10-17 和 10-18 中那样查看范围来确定我们返回的引用是否始终有效 。 借用检查器也无法确定这一点，因为它不知道 x 和 y 的生命周期与返回值的生命周期有何关系。 为了修复此错误，我们将添加定义引用之间关系的通用生命周期参数，以便借用检查器可以执行其分析。


### 生命周期注解语法(Lifetime Annotation Syntax)
Lifetime annotations don’t change how long any of the references live. Rather, they describe the relationships of the lifetimes of multiple references to each other without affecting the lifetimes. Just as functions can accept any type when the signature specifies a generic type parameter, functions can accept references with any lifetime by specifying a generic lifetime parameter.

Lifetime annotations have a slightly unusual syntax: the names of lifetime parameters must start with an apostrophe (') and are usually all lowercase and very short, like generic types. Most people use the name 'a for the first lifetime annotation. We place lifetime parameter annotations after the & of a reference, using a space to separate the annotation from the reference’s type.

Here are some examples: a reference to an i32 without a lifetime parameter, a reference to an i32 that has a lifetime parameter named 'a, and a mutable reference to an i32 that also has the lifetime 'a.

生命周期注释不会改变任何引用的生存时间。 相反，它们描述了多个引用的生命周期相互之间的关系，而不影响生命周期。 正如当签名指定泛型类型参数时函数可以接受任何类型一样，函数可以通过指定泛型生存期参数来接受具有任何生存期的引用。

生命周期注释的语法有点不寻常：生命周期参数的名称必须以撇号 (') 开头，并且通常都是小写且非常短，就像泛型类型一样。 大多数人使用名称“a”作为第一个生命周期注释。 我们将生命周期参数注释放在引用的 & 之后，使用空格将注释与引用的类型分开。

以下是一些示例：对没有生命周期参数的 i32 的引用、对具有名为“a”的生命周期参数的 i32 的引用，以及对也具有生命周期“a”的 i32 的可变引用。

``````
&i32        // a reference
&'a i32     // a reference with an explicit lifetime
&'a mut i32 // a mutable reference with an explicit lifetime
``````
One lifetime annotation by itself doesn’t have much meaning, because the annotations are meant to tell Rust how generic lifetime parameters of multiple references relate to each other. Let’s examine how the lifetime annotations relate to each other in the context of the longest function.

一个生命周期注释本身并没有多大意义，因为注释的目的是告诉 Rust 多个引用的通用生命周期参数如何相互关联。 让我们检查一下生命周期注释在最长函数的上下文中如何相互关联。

### Lifetime Annotations in Function Signatures
To use lifetime annotations in function signatures, we need to declare the generic lifetime parameters inside angle brackets between the function name and the parameter list, just as we did with generic type parameters.

We want the signature to express the following constraint: the returned reference will be valid as long as both the parameters are valid. This is the relationship between lifetimes of the parameters and the return value. We’ll name the lifetime 'a and then add it to each reference, as shown in Listing 10-21.

要在函数签名中使用生命周期注释，我们需要在函数名称和参数列表之间的尖括号内声明通用生命周期参数，就像我们对通用类型参数所做的那样。

我们希望签名表达以下约束：只要两个参数都有效，返回的引用就有效。 这就是参数的生命周期和返回值之间的关系。 我们将生命周期命名为“a”，然后将其添加到每个引用中，如清单 10-21 所示。

Filename: src/main.rs
``````
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
``````
Listing 10-21: The longest function definition specifying that all the references in the signature must have the same lifetime 'a

This code should compile and produce the result we want when we use it with the main function in Listing 10-19.

The function signature now tells Rust that for some lifetime 'a, the function takes two parameters, both of which are string slices that live at least as long as lifetime 'a. The function signature also tells Rust that the string slice returned from the function will live at least as long as lifetime 'a. In practice, it means that the lifetime of the reference returned by the longest function is the same as the smaller of the lifetimes of the values referred to by the function arguments. These relationships are what we want Rust to use when analyzing this code.

Remember, when we specify the lifetime parameters in this function signature, we’re not changing the lifetimes of any values passed in or returned. Rather, we’re specifying that the borrow checker should reject any values that don’t adhere to these constraints. Note that the longest function doesn’t need to know exactly how long x and y will live, only that some scope can be substituted for 'a that will satisfy this signature.

When annotating lifetimes in functions, the annotations go in the function signature, not in the function body. The lifetime annotations become part of the contract of the function, much like the types in the signature. Having function signatures contain the lifetime contract means the analysis the Rust compiler does can be simpler. If there’s a problem with the way a function is annotated or the way it is called, the compiler errors can point to the part of our code and the constraints more precisely. If, instead, the Rust compiler made more inferences about what we intended the relationships of the lifetimes to be, the compiler might only be able to point to a use of our code many steps away from the cause of the problem.

When we pass concrete references to longest, the concrete lifetime that is substituted for 'a is the part of the scope of x that overlaps with the scope of y. In other words, the generic lifetime 'a will get the concrete lifetime that is equal to the smaller of the lifetimes of x and y. Because we’ve annotated the returned reference with the same lifetime parameter 'a, the returned reference will also be valid for the length of the smaller of the lifetimes of x and y.

Let’s look at how the lifetime annotations restrict the longest function by passing in references that have different concrete lifetimes. Listing 10-22 is a straightforward example.

当我们将这段代码与清单 10-19 中的 main 函数一起使用时，它应该可以编译并产生我们想要的结果。

函数签名现在告诉 Rust，对于某个生命周期 'a，该函数采用两个参数，这两个参数都是字符串切片，其生命周期至少与生命周期 'a 一样长。 函数签名还告诉 Rust，从函数返回的字符串切片的生命周期至少与生命周期 'a 一样长。 实际上，这意味着最长函数返回的引用的生命周期与函数参数引用的值的较小生命周期相同。 这些关系是我们希望 Rust 在分析这段代码时使用的。

请记住，当我们在此函数签名中指定生命周期参数时，我们不会更改传入或返回的任何值的生命周期。 相反，我们指定借用检查器应该拒绝任何不遵守这些约束的值。 请注意，最长的函数不需要确切地知道 x 和 y 的生存时间，只需用某个作用域替换 'a 即可满足此签名。

在函数中注释生命周期时，注释位于函数签名中，而不是函数主体中。 生命周期注释成为函数契约的一部分，就像签名中的类型一样。 让函数签名包含生命周期契约意味着 Rust 编译器所做的分析可以更简单。 如果函数的注释方式或调用方式存在问题，编译器错误可以更准确地指出我们的代码部分和约束。 相反，如果 Rust 编译器对我们想要的生命周期关系进行更多推断，则编译器可能只能指出我们代码的使用情况与问题原因相差很多步。

当我们传递对longest的具体引用时，替换'a的具体生命周期是x的范围与y的范围重叠的部分。 换句话说，通用生命周期 'a 将获得等于 x 和 y 生命周期中较小者的具体生命周期。 因为我们使用相同的生命周期参数 'a 注释了返回的引用，所以返回的引用也将在 x 和 y 生命周期中较小者的长度内有效。

让我们看看生命周期注释如何通过传入具有不同具体生命周期的引用来限制最长函数。 清单 10-22 是一个简单的示例。

Filename: src/main.rs
``````
fn main() {
    let string1 = String::from("long string is long");

    {
        let string2 = String::from("xyz");
        let result = longest(string1.as_str(), string2.as_str());
        println!("The longest string is {}", result);
    }
}
``````
Listing 10-22: Using the longest function with references to String values that have different concrete lifetimes

In this example, string1 is valid until the end of the outer scope, string2 is valid until the end of the inner scope, and result references something that is valid until the end of the inner scope. Run this code, and you’ll see that the borrow checker approves; it will compile and print The longest string is long string is long.

Next, let’s try an example that shows that the lifetime of the reference in result must be the smaller lifetime of the two arguments. We’ll move the declaration of the result variable outside the inner scope but leave the assignment of the value to the result variable inside the scope with string2. Then we’ll move the println! that uses result to outside the inner scope, after the inner scope has ended. The code in Listing 10-23 will not compile.

在此示例中，string1 在外部作用域结束之前有效，string2 在内部作用域结束之前有效，而 result 引用在内部作用域结束之前有效的内容。 运行此代码，您将看到借用检查器已批准； 它将编译并打印最长的字符串是长字符串是长的。

接下来，让我们尝试一个示例，该示例表明结果中引用的生命周期必须是两个参数中较小的生命周期。 我们将把结果变量的声明移到内部作用域之外，但将值对结果变量的赋值保留在 string2 的作用域内。 然后我们将移动 println! 在内部作用域结束后，将结果用于内部作用域之外。 清单 10-23 中的代码将无法编译。

Filename: src/main.rs
``````
This code does not compile!
fn main() {
    let string1 = String::from("long string is long");
    let result;
    {
        let string2 = String::from("xyz");
        result = longest(string1.as_str(), string2.as_str());
    }
    println!("The longest string is {}", result);
}
``````
Listing 10-23: Attempting to use result after string2 has gone out of scope

When we try to compile this code, we get this error:
``````
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0597]: `string2` does not live long enough
 --> src/main.rs:6:44
  |
6 |         result = longest(string1.as_str(), string2.as_str());
  |                                            ^^^^^^^^^^^^^^^^ borrowed value does not live long enough
7 |     }
  |     - `string2` dropped here while still borrowed
8 |     println!("The longest string is {}", result);
  |                                          ------ borrow later used here

For more information about this error, try `rustc --explain E0597`.
error: could not compile `chapter10` due to previous error
``````
The error shows that for result to be valid for the println! statement, string2 would need to be valid until the end of the outer scope. Rust knows this because we annotated the lifetimes of the function parameters and return values using the same lifetime parameter 'a.

As humans, we can look at this code and see that string1 is longer than string2 and therefore result will contain a reference to string1. Because string1 has not gone out of scope yet, a reference to string1 will still be valid for the println! statement. However, the compiler can’t see that the reference is valid in this case. We’ve told Rust that the lifetime of the reference returned by the longest function is the same as the smaller of the lifetimes of the references passed in. Therefore, the borrow checker disallows the code in Listing 10-23 as possibly having an invalid reference.

Try designing more experiments that vary the values and lifetimes of the references passed in to the longest function and how the returned reference is used. Make hypotheses about whether or not your experiments will pass the borrow checker before you compile; then check to see if you’re right!

Thinking in Terms of Lifetimes
The way in which you need to specify lifetime parameters depends on what your function is doing. For example, if we changed the implementation of the longest function to always return the first parameter rather than the longest string slice, we wouldn’t need to specify a lifetime on the y parameter. The following code will compile:

该错误表明结果对于 println! 有效。 语句中，string2 需要在外部作用域结束之前保持有效。 Rust 知道这一点，因为我们使用相同的生命周期参数“a”注释了函数参数和返回值的生命周期。

作为人类，我们可以查看这段代码，发现 string1 比 string2 长，因此结果将包含对 string1 的引用。 因为 string1 尚未超出范围，所以对 string1 的引用对于 println! 仍然有效！ 陈述。 但是，在这种情况下，编译器无法看到引用是否有效。 我们已经告诉 Rust，最长函数返回的引用的生命周期与传入引用的生命周期中较小的一个相同。因此，借用检查器不允许清单 10-23 中的代码可能具有无效引用 。

尝试设计更多实验来改变传递给最长函数的引用的值和生命周期以及如何使用返回的引用。 在编译之前假设你的实验是否会通过借用检查器； 然后检查一下你是否正确！

从生命的角度思考
您需要指定生命周期参数的方式取决于您的函数正在执行的操作。 例如，如果我们更改最长函数的实现以始终返回第一个参数而不是最长的字符串切片，则不需要在 y 参数上指定生命周期。 以下代码将编译：

Filename: src/main.rs
``````
fn longest<'a>(x: &'a str, y: &str) -> &'a str {
    x
}
``````
We’ve specified a lifetime parameter 'a for the parameter x and the return type, but not for the parameter y, because the lifetime of y does not have any relationship with the lifetime of x or the return value.

When returning a reference from a function, the lifetime parameter for the return type needs to match the lifetime parameter for one of the parameters. If the reference returned does not refer to one of the parameters, it must refer to a value created within this function. However, this would be a dangling reference because the value will go out of scope at the end of the function. Consider this attempted implementation of the longest function that won’t compile:

我们为参数 x 和返回类型指定了生命周期参数 'a，但没有为参数 y 指定生命周期参数 'a，因为 y 的生命周期与 x 的生命周期或返回值没有任何关系。

从函数返回引用时，返回类型的生命周期参数需要与参数之一的生命周期参数匹配。 如果返回的引用不引用参数之一，则它必须引用在此函数中创建的值。 但是，这将是一个悬空引用，因为该值将在函数末尾超出范围。 考虑一下无法编译的最长函数的尝试实现：

Filename: src/main.rs
``````
This code does not compile!
fn longest<'a>(x: &str, y: &str) -> &'a str {
    let result = String::from("really long string");
    result.as_str()
}
``````
Here, even though we’ve specified a lifetime parameter 'a for the return type, this implementation will fail to compile because the return value lifetime is not related to the lifetime of the parameters at all. Here is the error message we get:
``````
$ cargo run
   Compiling chapter10 v0.1.0 (file:///projects/chapter10)
error[E0515]: cannot return reference to local variable `result`
  --> src/main.rs:11:5
   |
11 |     result.as_str()
   |     ^^^^^^^^^^^^^^^ returns a reference to data owned by the current function

For more information about this error, try `rustc --explain E0515`.
error: could not compile `chapter10` due to previous error
``````
The problem is that result goes out of scope and gets cleaned up at the end of the longest function. We’re also trying to return a reference to result from the function. There is no way we can specify lifetime parameters that would change the dangling reference, and Rust won’t let us create a dangling reference. In this case, the best fix would be to return an owned data type rather than a reference so the calling function is then responsible for cleaning up the value.

Ultimately, lifetime syntax is about connecting the lifetimes of various parameters and return values of functions. Once they’re connected, Rust has enough information to allow memory-safe operations and disallow operations that would create dangling pointers or otherwise violate memory safety.

问题是结果超出范围并在最长函数结束时被清除。 我们还尝试返回对函数结果的引用。 我们无法指定会更改悬空引用的生命周期参数，并且 Rust 不允许我们创建悬空引用。 在这种情况下，最好的解决方法是返回拥有的数据类型而不是引用，以便调用函数负责清理该值。

最终，生命周期语法是关于连接函数的各种参数和返回值的生命周期。 一旦它们连接起来，Rust 就有足够的信息来允许内存安全操作并禁止会创建悬空指针或以其他方式违反内存安全的操作。

### Lifetime Annotations in Struct Definitions
So far, the structs we’ve defined all hold owned types. We can define structs to hold references, but in that case we would need to add a lifetime annotation on every reference in the struct’s definition. Listing 10-24 has a struct named ImportantExcerpt that holds a string slice.

到目前为止，我们定义的结构体都拥有自有类型。 我们可以定义结构体来保存引用，但在这种情况下，我们需要在结构体定义中的每个引用上添加生命周期注释。 清单10-24有一个名为ImportantExcerpt的结构体，它保存一个字符串切片。

Filename: src/main.rs
``````
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.').next().expect("Could not find a '.'");
    let i = ImportantExcerpt {
        part: first_sentence,
    };
}
``````
Listing 10-24: A struct that holds a reference, requiring a lifetime annotation

This struct has the single field part that holds a string slice, which is a reference. As with generic data types, we declare the name of the generic lifetime parameter inside angle brackets after the name of the struct so we can use the lifetime parameter in the body of the struct definition. This annotation means an instance of ImportantExcerpt can’t outlive the reference it holds in its part field.

The main function here creates an instance of the ImportantExcerpt struct that holds a reference to the first sentence of the String owned by the variable novel. The data in novel exists before the ImportantExcerpt instance is created. In addition, novel doesn’t go out of scope until after the ImportantExcerpt goes out of scope, so the reference in the ImportantExcerpt instance is valid.

该结构具有保存字符串切片的单个字段部分，该字符串切片是一个引用。 与通用数据类型一样，我们在结构名称后面的尖括号内声明通用生命周期参数的名称，以便我们可以在结构定义主体中使用生命周期参数。 此注释意味着 importantExcerpt 的实例不能比它在其部分字段中保存的引用更长寿。

这里的 main 函数创建了一个 ImportExcerpt 结构体的实例，它保存了对变量 novel 所拥有的字符串的第一句话的引用。 小说中的数据在创建ImportantExcerpt 实例之前就已存在。 另外，在ImportantExcerpt超出范围之前，novel不会超出范围，因此ImportantExcerpt实例中的引用是有效的。

### 生命周期省略(Lifetime Elision)
You’ve learned that every reference has a lifetime and that you need to specify lifetime parameters for functions or structs that use references. However, in Chapter 4 we had a function in Listing 4-9, shown again in Listing 10-25, that compiled without lifetime annotations.

您已经了解到每个引用都有一个生命周期，并且您需要为使用引用的函数或结构指定生命周期参数。 然而，在第 4 章中，我们在清单 4-9 中有一个函数，在清单 10-25 中再次显示，该函数在编译时没有使用生命周期注释。

Filename: src/lib.rs
``````
fn first_word(s: &str) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}
``````
Listing 10-25: A function we defined in Listing 4-9 that compiled without lifetime annotations, even though the parameter and return type are references

The reason this function compiles without lifetime annotations is historical: in early versions (pre-1.0) of Rust, this code wouldn’t have compiled because every reference needed an explicit lifetime. At that time, the function signature would have been written like this:

该函数在没有生命周期注释的情况下进行编译的原因是历史性的：在 Rust 的早期版本（1.0 之前）中，该代码不会编译，因为每个引用都需要显式生命周期。 那时，函数签名会这样写：

``````
fn first_word<'a>(s: &'a str) -> &'a str {
``````
After writing a lot of Rust code, the Rust team found that Rust programmers were entering the same lifetime annotations over and over in particular situations. These situations were predictable and followed a few deterministic patterns. The developers programmed these patterns into the compiler’s code so the borrow checker could infer the lifetimes in these situations and wouldn’t need explicit annotations.

This piece of Rust history is relevant because it’s possible that more deterministic patterns will emerge and be added to the compiler. In the future, even fewer lifetime annotations might be required.

The patterns programmed into Rust’s analysis of references are called the lifetime elision rules. These aren’t rules for programmers to follow; they’re a set of particular cases that the compiler will consider, and if your code fits these cases, you don’t need to write the lifetimes explicitly.

The elision rules don’t provide full inference. If Rust deterministically applies the rules but there is still ambiguity as to what lifetimes the references have, the compiler won’t guess what the lifetime of the remaining references should be. Instead of guessing, the compiler will give you an error that you can resolve by adding the lifetime annotations.

Lifetimes on function or method parameters are called input lifetimes, and lifetimes on return values are called output lifetimes.

The compiler uses three rules to figure out the lifetimes of the references when there aren’t explicit annotations. The first rule applies to input lifetimes, and the second and third rules apply to output lifetimes. If the compiler gets to the end of the three rules and there are still references for which it can’t figure out lifetimes, the compiler will stop with an error. These rules apply to fn definitions as well as impl blocks.

The first rule is that the compiler assigns a lifetime parameter to each parameter that’s a reference. In other words, a function with one parameter gets one lifetime parameter: fn foo<'a>(x: &'a i32); a function with two parameters gets two separate lifetime parameters: fn foo<'a, 'b>(x: &'a i32, y: &'b i32); and so on.

The second rule is that, if there is exactly one input lifetime parameter, that lifetime is assigned to all output lifetime parameters: fn foo<'a>(x: &'a i32) -> &'a i32.

The third rule is that, if there are multiple input lifetime parameters, but one of them is &self or &mut self because this is a method, the lifetime of self is assigned to all output lifetime parameters. This third rule makes methods much nicer to read and write because fewer symbols are necessary.

Let’s pretend we’re the compiler. We’ll apply these rules to figure out the lifetimes of the references in the signature of the first_word function in Listing 10-25. The signature starts without any lifetimes associated with the references:

在编写大量 Rust 代码后，Rust 团队发现 Rust 程序员在特定情况下一遍又一遍地输入相同的生命周期注释。 这些情况是可以预测的，并且遵循一些确定性模式。 开发人员将这些模式编程到编译器的代码中，以便借用检查器可以推断这些情况下的生命周期，并且不需要显式注释。

Rust 的这段历史是相关的，因为更多确定性模式可能会出现并被添加到编译器中。 将来，可能需要更少的生命周期注释。

Rust 引用分析中编程的模式称为生命周期省略规则。 这些不是程序员必须遵守的规则； 它们是编译器将考虑的一组特殊情况，如果您的代码适合这些情况，则无需显式编写生命周期。

省略规则不提供完整的推理。 如果 Rust 确定性地应用规则，但引用的生命周期仍然不明确，则编译器不会猜测其余引用的生命周期应该是多少。 编译器不会猜测，而是会给您一个错误，您可以通过添加生命周期注释来解决该错误。

函数或方法参数的生命周期称为输入生命周期，返回值的生命周期称为输出生命周期。

当没有显式注释时，编译器使用三个规则来计算引用的生命周期。 第一个规则适用于输入生命周期，第二个和第三个规则适用于输出生命周期。 如果编译器到达三个规则的末尾并且仍然存在无法计算出生命周期的引用，则编译器将因错误而停止。 这些规则适用于 fn 定义以及 impl 块。

第一条规则是编译器为每个引用参数分配一个生命周期参数。 换句话说，具有一个参数的函数获得一个生命周期参数： fn foo<'a>(x: &'a i32); 具有两个参数的函数获得两个独立的生命周期参数： fn foo<'a, 'b>(x: &'a i32, y: &'b i32); 等等。

第二条规则是，如果只有一个输入生命周期参数，则该生命周期将分配给所有输出生命周期参数：fn foo<'a>(x: &'a i32) -> &'a i32。

第三条规则是，如果有多个输入生命周期参数，但其中一个是 &self 或 &mut self 因为这是一种方法，则 self 的生命周期将分配给所有输出生命周期参数。 第三条规则使方法更易于读写，因为所需的符号更少。

假设我们是编译器。 我们将应用这些规则来计算清单 10-25 中的first_word 函数签名中引用的生命周期。 签名开始时没有与引用关联的任何生命周期：

``````
fn first_word(s: &str) -> &str {
``````
Then the compiler applies the first rule, which specifies that each parameter gets its own lifetime. We’ll call it 'a as usual, so now the signature is this:
``````
fn first_word<'a>(s: &'a str) -> &str {
``````
The second rule applies because there is exactly one input lifetime. The second rule specifies that the lifetime of the one input parameter gets assigned to the output lifetime, so the signature is now this:
``````
fn first_word<'a>(s: &'a str) -> &'a str {
``````
Now all the references in this function signature have lifetimes, and the compiler can continue its analysis without needing the programmer to annotate the lifetimes in this function signature.

Let’s look at another example, this time using the longest function that had no lifetime parameters when we started working with it in Listing 10-20:
``````
fn longest(x: &str, y: &str) -> &str {
``````
Let’s apply the first rule: each parameter gets its own lifetime. This time we have two parameters instead of one, so we have two lifetimes:
``````
fn longest<'a, 'b>(x: &'a str, y: &'b str) -> &str {
``````
You can see that the second rule doesn’t apply because there is more than one input lifetime. The third rule doesn’t apply either, because longest is a function rather than a method, so none of the parameters are self. After working through all three rules, we still haven’t figured out what the return type’s lifetime is. This is why we got an error trying to compile the code in Listing 10-20: the compiler worked through the lifetime elision rules but still couldn’t figure out all the lifetimes of the references in the signature.

Because the third rule really only applies in method signatures, we’ll look at lifetimes in that context next to see why the third rule means we don’t have to annotate lifetimes in method signatures very often.

您可以看到第二条规则不适用，因为有多个输入生命周期。 第三条规则也不适用，因为longest是一个函数而不是一个方法，所以没有一个参数是self。 在完成所有三个规则之后，我们仍然没有弄清楚返回类型的生命周期是什么。 这就是为什么我们在尝试编译清单 10-20 中的代码时遇到错误：编译器执行了生命周期省略规则，但仍然无法找出签名中引用的所有生命周期。

因为第三条规则实际上只适用于方法签名，所以接下来我们将在该上下文中查看生命周期，以了解为什么第三条规则意味着我们不必经常在方法签名中注释生命周期。

### Lifetime Annotations in Method Definitions
When we implement methods on a struct with lifetimes, we use the same syntax as that of generic type parameters shown in Listing 10-11. Where we declare and use the lifetime parameters depends on whether they’re related to the struct fields or the method parameters and return values.

Lifetime names for struct fields always need to be declared after the impl keyword and then used after the struct’s name, because those lifetimes are part of the struct’s type.

In method signatures inside the impl block, references might be tied to the lifetime of references in the struct’s fields, or they might be independent. In addition, the lifetime elision rules often make it so that lifetime annotations aren’t necessary in method signatures. Let’s look at some examples using the struct named ImportantExcerpt that we defined in Listing 10-24.

First, we’ll use a method named level whose only parameter is a reference to self and whose return value is an i32, which is not a reference to anything:

当我们在具有生命周期的结构上实现方法时，我们使用与清单 10-11 中所示的泛型类型参数相同的语法。 我们在哪里声明和使用生命周期参数取决于它们是否与结构体字段或方法参数和返回值相关。

结构体字段的生命周期名称始终需要在 impl 关键字之后声明，然后在结构体名称之后使用，因为这些生命周期是结构体类型的一部分。

在 impl 块内的方法签名中，引用可能与结构字段中引用的生命周期相关联，也可能是独立的。 此外，生命周期省略规则通常使得方法签名中不需要生命周期注释。 让我们看一些使用清单 10-24 中定义的名为 ImportExcerpt 的结构的示例。

首先，我们将使用一个名为 level 的方法，其唯一参数是对 self 的引用，其返回值是 i32，它不是对任何内容的引用：

``````
impl<'a> ImportantExcerpt<'a> {
    fn level(&self) -> i32 {
        3
    }
}
``````
The lifetime parameter declaration after impl and its use after the type name are required, but we’re not required to annotate the lifetime of the reference to self because of the first elision rule.

impl 之后的生命周期参数声明及其在类型名称之后的使用是必需的，但由于第一个省略规则，我们不需要注释对 self 的引用的生命周期。

Here is an example where the third lifetime elision rule applies:
``````
impl<'a> ImportantExcerpt<'a> {
    fn announce_and_return_part(&self, announcement: &str) -> &str {
        println!("Attention please: {}", announcement);
        self.part
    }
}
``````
There are two input lifetimes, so Rust applies the first lifetime elision rule and gives both &self and announcement their own lifetimes. Then, because one of the parameters is &self, the return type gets the lifetime of &self, and all lifetimes have been accounted for.

有两个输入生命周期，因此 Rust 应用第一个生命周期省略规则，并为 &self 和announcement 赋予它们自己的生命周期。 然后，因为参数之一是&self，所以返回类型获取&self的生命周期，所有生命周期都已经计算在内。

### The Static Lifetime
One special lifetime we need to discuss is 'static, which denotes that the affected reference can live for the entire duration of the program. All string literals have the 'static lifetime, which we can annotate as follows:

我们需要讨论的一个特殊生命周期是“静态”，它表示受影响的引用可以在程序的整个持续时间内生存。 所有字符串文字都有 'static 生命周期，我们可以将其注释如下：

``````
let s: &'static str = "I have a static lifetime.";
``````
The text of this string is stored directly in the program’s binary, which is always available. Therefore, the lifetime of all string literals is 'static.

You might see suggestions to use the 'static lifetime in error messages. But before specifying 'static as the lifetime for a reference, think about whether the reference you have actually lives the entire lifetime of your program or not, and whether you want it to. Most of the time, an error message suggesting the 'static lifetime results from attempting to create a dangling reference or a mismatch of the available lifetimes. In such cases, the solution is fixing those problems, not specifying the 'static lifetime.

该字符串的文本直接存储在程序的二进制文件中，该二进制文件始终可用。 因此，所有字符串文字的生命周期都是“静态的”。

您可能会在错误消息中看到使用“静态生命周期”的建议。 但在指定 'static 作为引用的生命周期之前，请考虑您所拥有的引用是否实际上存在于程序的整个生命周期中，以及您是否希望它如此。 大多数时候，提示“静态生命周期”的错误消息是由于尝试创建悬空引用或可用生命周期不匹配而导致的。 在这种情况下，解决方案是解决这些问题，而不是指定“静态生命周期”。

### Generic Type Parameters, Trait Bounds, and Lifetimes Together
Let’s briefly look at the syntax of specifying generic type parameters, trait bounds, and lifetimes all in one function!
``````
use std::fmt::Display;

fn longest_with_an_announcement<'a, T>(
    x: &'a str,
    y: &'a str,
    ann: T,
) -> &'a str
where
    T: Display,
{
    println!("Announcement! {}", ann);
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
``````
This is the longest function from Listing 10-21 that returns the longer of two string slices. But now it has an extra parameter named ann of the generic type T, which can be filled in by any type that implements the Display trait as specified by the where clause. This extra parameter will be printed using {}, which is why the Display trait bound is necessary. Because lifetimes are a type of generic, the declarations of the lifetime parameter 'a and the generic type parameter T go in the same list inside the angle brackets after the function name.

这是清单 10-21 中最长的函数，它返回两个字符串切片中较长的一个。 但现在它有一个名为 ann 的泛型类型 T 的额外参数，可以由任何实现 where 子句指定的 Display 特征的类型填充该参数。 这个额外的参数将使用 {} 打印，这就是为什么 Display 特征绑定是必要的。 因为生命周期是泛型类型，所以生命周期参数 'a 和泛型类型参数 T 的声明位于函数名称后面的尖括号内的同一列表中。

## 10.4 总结

We covered a lot in this chapter! Now that you know about generic type parameters, traits and trait bounds, and generic lifetime parameters, you’re ready to write code without repetition that works in many different situations. Generic type parameters let you apply the code to different types. Traits and trait bounds ensure that even though the types are generic, they’ll have the behavior the code needs. You learned how to use lifetime annotations to ensure that this flexible code won’t have any dangling references. And all of this analysis happens at compile time, which doesn’t affect runtime performance!

我们在本章中介绍了很多内容！ 现在您已经了解了泛型类型参数、特征和特征边界以及泛型生命周期参数，您就可以编写无需重复且适用于许多不同情况的代码了。 通用类型参数允许您将代码应用于不同的类型。 特征和特征边界确保即使类型是通用的，它们也将具有代码所需的行为。 您学习了如何使用生命周期注释来确保此灵活的代码不会有任何悬空引用。 所有这些分析都发生在编译时，这不会影响运行时性能！

Believe it or not, there is much more to learn on the topics we discussed in this chapter: Chapter 17 discusses trait objects, which are another way to use traits. There are also more complex scenarios involving lifetime annotations that you will only need in very advanced scenarios; for those, you should read the Rust Reference. But next, you’ll learn how to write tests in Rust so you can make sure your code is working the way it should.


不管你相信与否，关于我们在本章中讨论的主题，还有很多东西需要学习：第 17 章讨论了特征对象，这是使用特征的另一种方式。 还有一些更复杂的场景涉及生命周期注释，只有在非常高级的场景中才需要它们； 对于这些，您应该阅读 Rust 参考。 但接下来，您将学习如何用 Rust 编写测试，以便确保您的代码按应有的方式工作。
