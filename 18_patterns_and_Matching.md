# 18. 模式匹配
Patterns are a special syntax in Rust for matching against the structure of types, both complex and simple. Using patterns in conjunction with match expressions and other constructs gives you more control over a program’s control flow. A pattern consists of some combination of the following:

模式是 Rust 中的一种特殊语法，用于匹配复杂和简单的类型结构。 将模式与匹配表达式和其他结构结合使用可以让您更好地控制程序的控制流。 模式由以下内容的某种组合组成：

+ Literals
+ Destructured arrays, enums, structs, or tuples
+ Variables
+ Wildcards
+ Placeholders

+ 文字
+ 解构数组、枚举、结构或元组
+ 变量
+ 通配符
+ 占位符

Some example patterns include x, (a, 3), and Some(Color::Red). In the contexts in which patterns are valid, these components describe the shape of data. Our program then matches values against the patterns to determine whether it has the correct shape of data to continue running a particular piece of code.

To use a pattern, we compare it to some value. If the pattern matches the value, we use the value parts in our code. Recall the match expressions in Chapter 6 that used patterns, such as the coin-sorting machine example. If the value fits the shape of the pattern, we can use the named pieces. If it doesn’t, the code associated with the pattern won’t run.

This chapter is a reference on all things related to patterns. We’ll cover the valid places to use patterns, the difference between refutable and irrefutable patterns, and the different kinds of pattern syntax that you might see. By the end of the chapter, you’ll know how to use patterns to express many concepts in a clear way.

一些示例模式包括 x、(a, 3) 和 Some(Color::Red)。 在模式有效的上下文中，这些组件描述了数据的形状。 然后，我们的程序将值与模式进行匹配，以确定它是否具有正确的数据形状来继续运行特定的代码段。

为了使用模式，我们将其与某个值进行比较。 如果模式与值匹配，我们将在代码中使用值部分。 回想一下第 6 章中使用模式的匹配表达式，例如硬币分类机的示例。 如果该值适合图案的形状，我们可以使用指定的片段。 如果没有，与该模式关联的代码将不会运行。

本章是所有与模式相关的内容的参考。 我们将介绍使用模式的有效位置、可反驳的模式和不可反驳的模式之间的区别，以及您可能会看到的不同类型的模式语法。 在本章结束时，您将了解如何使用模式来清晰地表达许多概念。

## 18.1 All the Places Patterns Can be Used

Patterns pop up in a number of places in Rust, and you’ve been using them a lot without realizing it! This section discusses all the places where patterns are valid.

模式出现在 Rust 的很多地方，你在没有意识到的情况下经常使用它们！ 本节讨论模式有效的所有地方。

### match Arms
As discussed in Chapter 6, we use patterns in the arms of match expressions. Formally, match expressions are defined as the keyword match, a value to match on, and one or more match arms that consist of a pattern and an expression to run if the value matches that arm’s pattern, like this:

正如第 6 章中所讨论的，我们在匹配表达式的分支中使用模式。 正式地，匹配表达式被定义为关键字 match、要匹配的值以及一个或多个匹配臂，其中包含一个模式和一个在值与该臂的模式匹配时要运行的表达式，如下所示：

``````
match VALUE {
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
}
``````
For example, here's the match expression from Listing 6-5 that matches on an Option<i32> value in the variable x:
``````
match x {
    None => None,
    Some(i) => Some(i + 1),
}
``````
The patterns in this match expression are the None and Some(i) on the left of each arrow.

One requirement for match expressions is that they need to be exhaustive in the sense that all possibilities for the value in the match expression must be accounted for. One way to ensure you’ve covered every possibility is to have a catchall pattern for the last arm: for example, a variable name matching any value can never fail and thus covers every remaining case.

The particular pattern _ will match anything, but it never binds to a variable, so it’s often used in the last match arm. The _ pattern can be useful when you want to ignore any value not specified, for example. We’ll cover the _ pattern in more detail in the “Ignoring Values in a Pattern” section later in this chapter.

此匹配表达式中的模式是每个箭头左侧的 None 和 Some(i)。

匹配表达式的一项要求是它们需要详尽无遗，因为必须考虑匹配表达式中值的所有可能性。 确保涵盖所有可能性的一种方法是为最后一个分支提供一个包罗万象的模式：例如，匹配任何值的变量名称永远不会失败，从而涵盖所有剩余的情况。

特定模式 _ 将匹配任何内容，但它从不绑定到变量，因此它经常在最后一个匹配臂中使用。 例如，当您想要忽略任何未指定的值时， _ 模式可能很有用。 我们将在本章后面的“忽略模式中的值”部分中更详细地介绍 _ 模式。

### Conditional if let Expressions
In Chapter 6 we discussed how to use if let expressions mainly as a shorter way to write the equivalent of a match that only matches one case. Optionally, if let can have a corresponding else containing code to run if the pattern in the if let doesn’t match.

Listing 18-1 shows that it’s also possible to mix and match if let, else if, and else if let expressions. Doing so gives us more flexibility than a match expression in which we can express only one value to compare with the patterns. Also, Rust doesn't require that the conditions in a series of if let, else if, else if let arms relate to each other.

The code in Listing 18-1 determines what color to make your background based on a series of checks for several conditions. For this example, we’ve created variables with hardcoded values that a real program might receive from user input.

在第 6 章中，我们主要讨论了如何使用 if let 表达式作为一种更短的方式来编写仅匹配一种情况的匹配的等效项。 或者，如果 if let 中的模式不匹配，则 if let 可以有一个相应的 else 包含代码来运行。

清单 18-1 表明，也可以混合和匹配 if let、else if 和 else if let 表达式。 这样做为我们提供了比匹配表达式更大的灵活性，在匹配表达式中我们只能表达一个值来与模式进行比较。 此外，Rust 不要求一系列 if let、else if、else if let 分支中的条件相互关联。

清单 18-1 中的代码根据对多个条件的一系列检查来确定背景颜色。 对于此示例，我们创建了具有硬编码值的变量，真实程序可能会从用户输入接收这些值。

Filename: src/main.rs
``````
fn main() {
    let favorite_color: Option<&str> = None;
    let is_tuesday = false;
    let age: Result<u8, _> = "34".parse();

    if let Some(color) = favorite_color {
        println!("Using your favorite color, {color}, as the background");
    } else if is_tuesday {
        println!("Tuesday is green day!");
    } else if let Ok(age) = age {
        if age > 30 {
            println!("Using purple as the background color");
        } else {
            println!("Using orange as the background color");
        }
    } else {
        println!("Using blue as the background color");
    }
}
``````
Listing 18-1: Mixing if let, else if, else if let, and else

If the user specifies a favorite color, that color is used as the background. If no favorite color is specified and today is Tuesday, the background color is green. Otherwise, if the user specifies their age as a string and we can parse it as a number successfully, the color is either purple or orange depending on the value of the number. If none of these conditions apply, the background color is blue.

This conditional structure lets us support complex requirements. With the hardcoded values we have here, this example will print Using purple as the background color.

You can see that if let can also introduce shadowed variables in the same way that match arms can: the line if let Ok(age) = age introduces a new shadowed age variable that contains the value inside the Ok variant. This means we need to place the if age > 30 condition within that block: we can’t combine these two conditions into if let Ok(age) = age && age > 30. The shadowed age we want to compare to 30 isn’t valid until the new scope starts with the curly bracket.

The downside of using if let expressions is that the compiler doesn’t check for exhaustiveness, whereas with match expressions it does. If we omitted the last else block and therefore missed handling some cases, the compiler would not alert us to the possible logic bug.

如果用户指定最喜欢的颜色，则该颜色将用作背景。 如果没有指定最喜欢的颜色并且今天是星期二，则背景颜色为绿色。 否则，如果用户将他们的年龄指定为字符串，并且我们可以成功将其解析为数字，则颜色为紫色或橙色，具体取决于数字的值。 如果这些条件都不适用，则背景颜色为蓝色。

这种条件结构使我们能够支持复杂的需求。 使用此处的硬编码值，此示例将打印“使用紫色作为背景颜色”。

您可以看到，if let 也可以采用与 matcharms 相同的方式引入阴影变量：if let Ok(age)=age 行引入了一个新的阴影年龄变量，其中包含 Ok 变量内的值。 这意味着我们需要将 ifage > 30 条件放在该块中：我们不能将这两个条件组合到 if let Ok(age) =age &&age > 30 中。我们想要与 30 进行比较的影子年龄不是 在新作用域以大括号开始之前一直有效。

使用 if let 表达式的缺点是编译器不会检查是否详尽，而使用 match 表达式会检查。 如果我们省略了最后一个 else 块并因此错过了对某些情况的处理，编译器将不会提醒我们可能的逻辑错误。

### while let Conditional Loops
Similar in construction to if let, the while let conditional loop allows a while loop to run for as long as a pattern continues to match. In Listing 18-2 we code a while let loop that uses a vector as a stack and prints the values in the vector in the opposite order in which they were pushed.

while let 条件循环的结构与 if let 类似，只要模式继续匹配，就允许 while 循环运行。 在清单 18-2 中，我们编写了一个 while let 循环，该循环使用向量作为堆栈，并按照与压入向量相反的顺序打印向量中的值。
``````
    let mut stack = Vec::new();

    stack.push(1);
    stack.push(2);
    stack.push(3);

    while let Some(top) = stack.pop() {
        println!("{}", top);
    }
``````
Listing 18-2: Using a while let loop to print values for as long as stack.pop() returns Some

This example prints 3, 2, and then 1. The pop method takes the last element out of the vector and returns Some(value). If the vector is empty, pop returns None. The while loop continues running the code in its block as long as pop returns Some. When pop returns None, the loop stops. We can use while let to pop every element off our stack.

此示例打印 3、2，然后打印 1。 pop 方法从向量中取出最后一个元素并返回 Some(value)。 如果向量为空，则 pop 返回 None。 只要 pop 返回 Some，while 循环就会继续运行其块中的代码。 当 pop 返回 None 时，循环停止。 我们可以使用 while let 将每个元素从堆栈中弹出。

### for Loops
In a for loop, the value that directly follows the keyword for is a pattern. For example, in for x in y the x is the pattern. Listing 18-3 demonstrates how to use a pattern in a for loop to destructure, or break apart, a tuple as part of the for loop.

在 for 循环中，直接跟在关键字 for 后面的值是一个模式。 例如，在 for x in y 中，x 是模式。 清单 18-3 演示了如何在 for 循环中使用模式来解构或分解元组作为 for 循环的一部分。

``````
    let v = vec!['a', 'b', 'c'];

    for (index, value) in v.iter().enumerate() {
        println!("{} is at index {}", value, index);
    }
``````
Listing 18-3: Using a pattern in a for loop to destructure a tuple

The code in Listing 18-3 will print the following:
``````
$ cargo run
   Compiling patterns v0.1.0 (file:///projects/patterns)
    Finished dev [unoptimized + debuginfo] target(s) in 0.52s
     Running `target/debug/patterns`
a is at index 0
b is at index 1
c is at index 2
``````
We adapt an iterator using the enumerate method so it produces a value and the index for that value, placed into a tuple. The first value produced is the tuple (0, 'a'). When this value is matched to the pattern (index, value), index will be 0 and value will be 'a', printing the first line of the output.

我们使用枚举方法调整迭代器，以便它生成一个值以及该值的索引，并将其放入元组中。 生成的第一个值是元组 (0, 'a')。 当该值与模式（索引，值）匹配时，索引将为 0，值将为“a”，打印输出的第一行。

### let Statements
Prior to this chapter, we had only explicitly discussed using patterns with match and if let, but in fact, we’ve used patterns in other places as well, including in let statements. For example, consider this straightforward variable assignment with let:

在本章之前，我们只明确讨论了在 match 和 if let 中使用模式，但事实上，我们也在其他地方使用了模式，包括在 let 语句中。 例如，考虑使用 let 进行的简单变量赋值：

``````
let x = 5;
``````
Every time you've used a let statement like this you've been using patterns, although you might not have realized it! More formally, a let statement looks like this:
``````
let PATTERN = EXPRESSION;
``````
In statements like let x = 5; with a variable name in the PATTERN slot, the variable name is just a particularly simple form of a pattern. Rust compares the expression against the pattern and assigns any names it finds. So in the let x = 5; example, x is a pattern that means “bind what matches here to the variable x.” Because the name x is the whole pattern, this pattern effectively means “bind everything to the variable x, whatever the value is.”

To see the pattern matching aspect of let more clearly, consider Listing 18-4, which uses a pattern with let to destructure a tuple.

在像 let x = 5; 这样的语句中 当 PATTERN 槽中有变量名时，变量名只是模式的一种特别简单的形式。 Rust 将表达式与模式进行比较，并分配它找到的任何名称。 所以让 x = 5; 例如，x 是一个模式，意思是“将此处匹配的内容绑定到变量 x”。 因为名称 x 是整个模式，所以该模式实际上意味着“将所有内容绑定到变量 x，无论值是什么。”

为了更清楚地了解 let 的模式匹配方面，请考虑清单 18-4，它使用 let 的模式来解构元组。

``````
    let (x, y, z) = (1, 2, 3);
``````
Listing 18-4: Using a pattern to destructure a tuple and create three variables at once

Here, we match a tuple against a pattern. Rust compares the value (1, 2, 3) to the pattern (x, y, z) and sees that the value matches the pattern, so Rust binds 1 to x, 2 to y, and 3 to z. You can think of this tuple pattern as nesting three individual variable patterns inside it.

If the number of elements in the pattern doesn’t match the number of elements in the tuple, the overall type won’t match and we’ll get a compiler error. For example, Listing 18-5 shows an attempt to destructure a tuple with three elements into two variables, which won’t work.

在这里，我们将元组与模式进行匹配。 Rust 将值 (1, 2, 3) 与模式 (x, y, z) 进行比较，发现该值与模式匹配，因此 Rust 将 1 绑定到 x，2 绑定到 y，3 绑定到 z。 您可以将此元组模式视为其中嵌套了三个单独的变量模式。

如果模式中的元素数量与元组中的元素数量不匹配，则整体类型将不匹配，我们将收到编译器错误。 例如，清单 18-5 展示了将一个包含三个元素的元组解构为两个变量的尝试，但这是行不通的。

``````
    let (x, y) = (1, 2, 3);
``````
Listing 18-5: Incorrectly constructing a pattern whose variables don’t match the number of elements in the tuple

Attempting to compile this code results in this type error:
``````
$ cargo run
   Compiling patterns v0.1.0 (file:///projects/patterns)
error[E0308]: mismatched types
 --> src/main.rs:2:9
  |
2 |     let (x, y) = (1, 2, 3);
  |         ^^^^^^   --------- this expression has type `({integer}, {integer}, {integer})`
  |         |
  |         expected a tuple with 3 elements, found one with 2 elements
  |
  = note: expected tuple `({integer}, {integer}, {integer})`
             found tuple `(_, _)`

For more information about this error, try `rustc --explain E0308`.
error: could not compile `patterns` due to previous error
``````
To fix the error, we could ignore one or more of the values in the tuple using _ or .., as you’ll see in the “Ignoring Values in a Pattern” section. If the problem is that we have too many variables in the pattern, the solution is to make the types match by removing variables so the number of variables equals the number of elements in the tuple.

要修复该错误，我们可以使用 _ 或 .. 忽略元组中的一个或多个值，正如您将在“忽略模式中的值”部分中看到的那样。 如果问题是模式中的变量太多，解决方案是通过删除变量来使类型匹配，以便变量的数量等于元组中元素的数量。

### Function Parameters
Function parameters can also be patterns. The code in Listing 18-6, which declares a function named foo that takes one parameter named x of type i32, should by now look familiar.

函数参数也可以是模式。 清单 18-6 中的代码声明了一个名为 foo 的函数，该函数接受一个名为 x 的 i32 类型参数，现在看起来应该很熟悉。

``````
fn foo(x: i32) {
    // code goes here
}
``````
Listing 18-6: A function signature uses patterns in the parameters

The x part is a pattern! As we did with let, we could match a tuple in a function’s arguments to the pattern. Listing 18-7 splits the values in a tuple as we pass it to a function.

x 部分是一个模式！ 正如我们对 let 所做的那样，我们可以将函数参数中的元组与模式进行匹配。 清单 18-7 当我们将元组传递给函数时，将其拆分为元组中的值。

Filename: src/main.rs
``````
fn print_coordinates(&(x, y): &(i32, i32)) {
    println!("Current location: ({}, {})", x, y);
}

fn main() {
    let point = (3, 5);
    print_coordinates(&point);
}
``````
Listing 18-7: A function with parameters that destructure a tuple

This code prints Current location: (3, 5). The values &(3, 5) match the pattern &(x, y), so x is the value 3 and y is the value 5.

We can also use patterns in closure parameter lists in the same way as in function parameter lists, because closures are similar to functions, as discussed in Chapter 13.

At this point, you’ve seen several ways of using patterns, but patterns don’t work the same in every place we can use them. In some places, the patterns must be irrefutable; in other circumstances, they can be refutable. We’ll discuss these two concepts next.

此代码打印当前位置：(3, 5)。 值 &(3, 5) 与模式 &(x, y) 匹配，因此 x 是值 3，y 是值 5。

我们还可以像在函数参数列表中一样在闭包参数列表中使用模式，因为闭包与函数类似，如第 13 章所述。

至此，您已经了解了使用模式的多种方法，但是模式并不是在我们可以使用它们的每个地方都以相同的方式工作。 在某些地方，模式必须是无可辩驳的；在某些地方，模式必须是无可辩驳的。 在其他情况下，它们是可以反驳的。 接下来我们将讨论这两个概念。

## 18.2 Refutability: Whether a Pattern Might Fail to Match
Patterns come in two forms: refutable and irrefutable. Patterns that will match for any possible value passed are irrefutable. An example would be x in the statement let x = 5; because x matches anything and therefore cannot fail to match. Patterns that can fail to match for some possible value are refutable. An example would be Some(x) in the expression if let Some(x) = a_value because if the value in the a_value variable is None rather than Some, the Some(x) pattern will not match.

Function parameters, let statements, and for loops can only accept irrefutable patterns, because the program cannot do anything meaningful when values don’t match. The if let and while let expressions accept refutable and irrefutable patterns, but the compiler warns against irrefutable patterns because by definition they’re intended to handle possible failure: the functionality of a conditional is in its ability to perform differently depending on success or failure.

In general, you shouldn’t have to worry about the distinction between refutable and irrefutable patterns; however, you do need to be familiar with the concept of refutability so you can respond when you see it in an error message. In those cases, you’ll need to change either the pattern or the construct you’re using the pattern with, depending on the intended behavior of the code.

Let’s look at an example of what happens when we try to use a refutable pattern where Rust requires an irrefutable pattern and vice versa. Listing 18-8 shows a let statement, but for the pattern we’ve specified Some(x), a refutable pattern. As you might expect, this code will not compile.

模式有两种形式：可反驳的和不可反驳的。 与任何可能传递的值相匹配的模式是无可辩驳的。 例如，语句 let x = 5; 中的 x 因为 x 匹配任何内容，因此不可能匹配失败。 无法匹配某些可能值的模式是可以反驳的。 例如，表达式 if let Some(x) = a_value 中的 Some(x)，因为如果 a_value 变量中的值为 None 而不是 Some，则 Some(x) 模式将不匹配。

函数参数、let 语句和 for 循环只能接受无可辩驳的模式，因为当值不匹配时程序无法执行任何有意义的操作。 if let 和 while let 表达式接受可反驳和不可反驳的模式，但编译器会警告不可反驳的模式，因为根据定义，它们旨在处理可能的失败：条件的功能在于其能够根据成功或失败执行不同的操作。

一般来说，您不必担心可反驳模式和不可反驳模式之间的区别； 但是，您确实需要熟悉可反驳性的概念，以便在错误消息中看到它时可以做出响应。 在这些情况下，您需要更改模式或使用该模式的构造，具体取决于代码的预期行为。

让我们看一个例子，看看当我们尝试使用可反驳的模式时会发生什么，而 Rust 需要不可反驳的模式，反之亦然。 示例 18-8 显示了一个 let 语句，但对于模式我们指定了 Some(x)，这是一个可反驳的模式。 正如您所料，这段代码将无法编译。

``````
    let Some(x) = some_option_value;
``````
Listing 18-8: Attempting to use a refutable pattern with let

If some_option_value was a None value, it would fail to match the pattern Some(x), meaning the pattern is refutable. However, the let statement can only accept an irrefutable pattern because there is nothing valid the code can do with a None value. At compile time, Rust will complain that we’ve tried to use a refutable pattern where an irrefutable pattern is required:

如果 some_option_value 是 None 值，则它将无法匹配模式 Some(x)，这意味着该模式是可反驳的。 但是，let 语句只能接受无可辩驳的模式，因为代码无法对 None 值执行任何有效操作。 在编译时，Rust 会抱怨我们试图在需要不可反驳模式的地方使用可反驳模式：

``````
$ cargo run
   Compiling patterns v0.1.0 (file:///projects/patterns)
error[E0005]: refutable pattern in local binding: `None` not covered
 --> src/main.rs:3:9
  |
3 |     let Some(x) = some_option_value;
  |         ^^^^^^^ pattern `None` not covered
  |
  = note: `let` bindings require an "irrefutable pattern", like a `struct` or an `enum` with only one variant
  = note: for more information, visit https://doc.rust-lang.org/book/ch18-02-refutability.html
note: `Option<i32>` defined here
 --> /rustc/d5a82bbd26e1ad8b7401f6a718a9c57c96905483/library/core/src/option.rs:518:1
  |
  = note: 
/rustc/d5a82bbd26e1ad8b7401f6a718a9c57c96905483/library/core/src/option.rs:522:5: not covered
  = note: the matched value is of type `Option<i32>`
help: you might want to use `if let` to ignore the variant that isn't matched
  |
3 |     let x = if let Some(x) = some_option_value { x } else { todo!() };
  |     ++++++++++                                 ++++++++++++++++++++++
help: alternatively, you might want to use let else to handle the variant that isn't matched
  |
3 |     let Some(x) = some_option_value else { todo!() };
  |                                     ++++++++++++++++

For more information about this error, try `rustc --explain E0005`.
error: could not compile `patterns` due to previous error
``````
Because we didn’t cover (and couldn’t cover!) every valid value with the pattern Some(x), Rust rightfully produces a compiler error.

If we have a refutable pattern where an irrefutable pattern is needed, we can fix it by changing the code that uses the pattern: instead of using let, we can use if let. Then if the pattern doesn’t match, the code will just skip the code in the curly brackets, giving it a way to continue validly. Listing 18-9 shows how to fix the code in Listing 18-8.

因为我们没有用 Some(x) 模式覆盖（也无法覆盖！）每个有效值，Rust 理所当然地会产生编译器错误。

如果我们有一个需要不可反驳模式的可反驳模式，我们可以通过更改使用该模式的代码来修复它：我们可以使用 if let，而不是使用 let。 然后，如果模式不匹配，代码将跳过大括号中的代码，从而有效地继续。 清单 18-9 显示了如何修复清单 18-8 中的代码。

``````
    if let Some(x) = some_option_value {
        println!("{}", x);
    }
``````
Listing 18-9: Using if let and a block with refutable patterns instead of let

We’ve given the code an out! This code is perfectly valid, although it means we cannot use an irrefutable pattern without receiving an error. If we give if let a pattern that will always match, such as x, as shown in Listing 18-10, the compiler will give a warning.

我们已经给出了代码！ 这段代码是完全有效的，尽管这意味着我们不能在不收到错误的情况下使用无可辩驳的模式。 如果我们给 if let 一个始终匹配的模式，例如 x，如清单 18-10 所示，编译器将给出警告。

``````
    if let x = 5 {
        println!("{}", x);
    };
``````
Listing 18-10: Attempting to use an irrefutable pattern with if let

Rust complains that it doesn’t make sense to use if let with an irrefutable pattern:
``````
$ cargo run
   Compiling patterns v0.1.0 (file:///projects/patterns)
warning: irrefutable `if let` pattern
 --> src/main.rs:2:8
  |
2 |     if let x = 5 {
  |        ^^^^^^^^^
  |
  = note: this pattern will always match, so the `if let` is useless
  = help: consider replacing the `if let` with a `let`
  = note: `#[warn(irrefutable_let_patterns)]` on by default

warning: `patterns` (bin "patterns") generated 1 warning
    Finished dev [unoptimized + debuginfo] target(s) in 0.39s
     Running `target/debug/patterns`
5
``````
For this reason, match arms must use refutable patterns, except for the last arm, which should match any remaining values with an irrefutable pattern. Rust allows us to use an irrefutable pattern in a match with only one arm, but this syntax isn’t particularly useful and could be replaced with a simpler let statement.

Now that you know where to use patterns and the difference between refutable and irrefutable patterns, let’s cover all the syntax we can use to create patterns.

因此，匹配臂必须使用可反驳的模式，最后一个臂除外，它应该将任何剩余的值与不可反驳的模式相匹配。 Rust 允许我们在只有一只手臂的比赛中使用无可辩驳的模式，但这种语法并不是特别有用，可以用更简单的 let 语句替换。

现在您知道了在哪里使用模式以及可反驳模式和不可反驳模式之间的区别，让我们介绍可用于创建模式的所有语法。

## 18.3 Pattern Syntax
In this section, we gather all the syntax valid in patterns and discuss why and when you might want to use each one.

在本节中，我们收集模式中有效的所有语法，并讨论您可能想要使用每一种语法的原因和时间。

### Matching Literals
As you saw in Chapter 6, you can match patterns against literals directly. The following code gives some examples:
``````
    let x = 1;

    match x {
        1 => println!("one"),
        2 => println!("two"),
        3 => println!("three"),
        _ => println!("anything"),
    }
``````
This code prints one because the value in x is 1. This syntax is useful when you want your code to take an action if it gets a particular concrete value.

### Matching Named Variables
Named variables are irrefutable patterns that match any value, and we’ve used them many times in the book. However, there is a complication when you use named variables in match expressions. Because match starts a new scope, variables declared as part of a pattern inside the match expression will shadow those with the same name outside the match construct, as is the case with all variables. In Listing 18-11, we declare a variable named x with the value Some(5) and a variable y with the value 10. We then create a match expression on the value x. Look at the patterns in the match arms and println! at the end, and try to figure out what the code will print before running this code or reading further.

命名变量是匹配任何值的无可辩驳的模式，我们在书中多次使用它们。 但是，当您在匹配表达式中使用命名变量时，会出现一些复杂情况。 因为 match 启动一个新的作用域，所以声明为 match 表达式内模式一部分的变量将遮蔽 match 构造外部的同名变量，就像所有变量的情况一样。 在清单 18-11 中，我们声明了一个名为 x 的变量，其值为 Some(5) 和一个变量 y，其值为 10。然后，我们对值 x 创建一个匹配表达式。 看看火柴臂和 println 中的图案！ 最后，并在运行此代码或进一步阅读之前尝试弄清楚代码将打印什么。

Filename: src/main.rs
``````
    let x = Some(5);
    let y = 10;

    match x {
        Some(50) => println!("Got 50"),
        Some(y) => println!("Matched, y = {y}"),
        _ => println!("Default case, x = {:?}", x),
    }

    println!("at the end: x = {:?}, y = {y}", x);
``````
Listing 18-11: A match expression with an arm that introduces a shadowed variable y

Let’s walk through what happens when the match expression runs. The pattern in the first match arm doesn’t match the defined value of x, so the code continues.

The pattern in the second match arm introduces a new variable named y that will match any value inside a Some value. Because we’re in a new scope inside the match expression, this is a new y variable, not the y we declared at the beginning with the value 10. This new y binding will match any value inside a Some, which is what we have in x. Therefore, this new y binds to the inner value of the Some in x. That value is 5, so the expression for that arm executes and prints Matched, y = 5.

If x had been a None value instead of Some(5), the patterns in the first two arms wouldn’t have matched, so the value would have matched to the underscore. We didn’t introduce the x variable in the pattern of the underscore arm, so the x in the expression is still the outer x that hasn’t been shadowed. In this hypothetical case, the match would print Default case, x = None.

When the match expression is done, its scope ends, and so does the scope of the inner y. The last println! produces at the end: x = Some(5), y = 10.

To create a match expression that compares the values of the outer x and y, rather than introducing a shadowed variable, we would need to use a match guard conditional instead. We’ll talk about match guards later in the “Extra Conditionals with Match Guards” section.

让我们看一下匹配表达式运行时会发生什么。 第一个匹配臂中的模式与 x 的定义值不匹配，因此代码继续。

第二个匹配臂中的模式引入了一个名为 y 的新变量，它将匹配 Some 值内的任何值。 因为我们处于匹配表达式内的新作用域，所以这是一个新的 y 变量，而不是我们在开头声明的值为 10 的 y。这个新的 y 绑定将匹配 Some 内的任何值，这就是我们所拥有的 在 x 中。 因此，这个新的 y 绑定到 x 中 Some 的内部值。 该值为 5，因此执行该臂的表达式并打印 Matched, y = 5。

如果 x 是 None 值而不是 Some(5)，则前两个臂中的模式将不匹配，因此该值将与下划线匹配。 我们没有在下划线臂的模式中引入 x 变量，因此表达式中的 x 仍然是没有被遮蔽的外部 x。 在这个假设的情况下，匹配将打印默认情况，x = None。

当匹配表达式完成时，它的范围结束，内部 y 的范围也结束。 最后的打印！ 最后产生：x = Some(5), y = 10。

要创建一个比较外部 x 和 y 值的匹配表达式，而不是引入隐藏变量，我们需要使用匹配防护条件。 我们稍后将在“带有 Match Guards 的额外条件”部分讨论 match Guards。

### Multiple Patterns
In match expressions, you can match multiple patterns using the | syntax, which is the pattern or operator. For example, in the following code we match the value of x against the match arms, the first of which has an or option, meaning if the value of x matches either of the values in that arm, that arm’s code will run:

在匹配表达式中，您可以使用 | 来匹配多个模式。 语法，即模式或运算符。 例如，在下面的代码中，我们将 x 的值与匹配臂进行匹配，其中第一个匹配臂有一个 or 选项，这意味着如果 x 的值与该臂中的任何一个值匹配，则该臂的代码将运行：

``````
    let x = 1;

    match x {
        1 | 2 => println!("one or two"),
        3 => println!("three"),
        _ => println!("anything"),
    }
``````
This code prints one or two.

### Matching Ranges of Values with ..=
The ..= syntax allows us to match to an inclusive range of values. In the following code, when a pattern matches any of the values within the given range, that arm will execute:
``````
    let x = 5;

    match x {
        1..=5 => println!("one through five"),
        _ => println!("something else"),
    }
``````
If x is 1, 2, 3, 4, or 5, the first arm will match. This syntax is more convenient for multiple match values than using the | operator to express the same idea; if we were to use | we would have to specify 1 | 2 | 3 | 4 | 5. Specifying a range is much shorter, especially if we want to match, say, any number between 1 and 1,000!

The compiler checks that the range isn’t empty at compile time, and because the only types for which Rust can tell if a range is empty or not are char and numeric values, ranges are only allowed with numeric or char values.

Here is an example using ranges of char values:
``````
    let x = 'c';

    match x {
        'a'..='j' => println!("early ASCII letter"),
        'k'..='z' => println!("late ASCII letter"),
        _ => println!("something else"),
    }
``````
Rust can tell that 'c' is within the first pattern’s range and prints early ASCII letter.

### Destructuring to Break Apart Values
We can also use patterns to destructure structs, enums, and tuples to use different parts of these values. Let’s walk through each value.

#### Destructuring Structs
Listing 18-12 shows a Point struct with two fields, x and y, that we can break apart using a pattern with a let statement.

Filename: src/main.rs
``````
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };

    let Point { x: a, y: b } = p;
    assert_eq!(0, a);
    assert_eq!(7, b);
}
``````
Listing 18-12: Destructuring a struct’s fields into separate variables

This code creates the variables a and b that match the values of the x and y fields of the p struct. This example shows that the names of the variables in the pattern don’t have to match the field names of the struct. However, it’s common to match the variable names to the field names to make it easier to remember which variables came from which fields. Because of this common usage, and because writing let Point { x: x, y: y } = p; contains a lot of duplication, Rust has a shorthand for patterns that match struct fields: you only need to list the name of the struct field, and the variables created from the pattern will have the same names. Listing 18-13 behaves in the same way as the code in Listing 18-12, but the variables created in the let pattern are x and y instead of a and b.

此代码创建与 p 结构的 x 和 y 字段的值匹配的变量 a 和 b。 此示例表明模式中变量的名称不必与结构的字段名称匹配。 但是，通常将变量名称与字段名称相匹配，以便更容易记住哪些变量来自哪些字段。 由于这种常见用法，并且因为写作 let Point { x: x, y: y } = p; 包含大量重复，Rust 有一个匹配结构体字段的模式的简写：你只需要列出结构体字段的名称，从该模式创建的变量将具有相同的名称。 清单 18-13 的行为与清单 18-12 中的代码相同，但在 let 模式中创建的变量是 x 和 y，而不是 a 和 b。

Filename: src/main.rs
``````
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };

    let Point { x, y } = p;
    assert_eq!(0, x);
    assert_eq!(7, y);
}
``````
Listing 18-13: Destructuring struct fields using struct field shorthand

This code creates the variables x and y that match the x and y fields of the p variable. The outcome is that the variables x and y contain the values from the p struct.

We can also destructure with literal values as part of the struct pattern rather than creating variables for all the fields. Doing so allows us to test some of the fields for particular values while creating variables to destructure the other fields.

In Listing 18-14, we have a match expression that separates Point values into three cases: points that lie directly on the x axis (which is true when y = 0), on the y axis (x = 0), or neither.

此代码创建与 p 变量的 x 和 y 字段匹配的变量 x 和 y。 结果是变量 x 和 y 包含 p 结构中的值。

我们还可以使用文字值作为结构模式的一部分进行解构，而不是为所有字段创建变量。 这样做允许我们测试某些字段的特定值，同时创建变量来解构其他字段。

在清单 18-14 中，我们有一个匹配表达式，它将 Point 值分为三种情况：直接位于 x 轴上的点（当 y = 0 时为真）、位于 y 轴上的点 (x = 0) 或两者都不位于。

Filename: src/main.rs
``````
fn main() {
    let p = Point { x: 0, y: 7 };

    match p {
        Point { x, y: 0 } => println!("On the x axis at {x}"),
        Point { x: 0, y } => println!("On the y axis at {y}"),
        Point { x, y } => {
            println!("On neither axis: ({x}, {y})");
        }
    }
}
``````
Listing 18-14: Destructuring and matching literal values in one pattern

The first arm will match any point that lies on the x axis by specifying that the y field matches if its value matches the literal 0. The pattern still creates an x variable that we can use in the code for this arm.

Similarly, the second arm matches any point on the y axis by specifying that the x field matches if its value is 0 and creates a variable y for the value of the y field. The third arm doesn’t specify any literals, so it matches any other Point and creates variables for both the x and y fields.

In this example, the value p matches the second arm by virtue of x containing a 0, so this code will print On the y axis at 7.

Remember that a match expression stops checking arms once it has found the first matching pattern, so even though Point { x: 0, y: 0} is on the x axis and the y axis, this code would only print On the x axis at 0.

第一个臂将匹配位于 x 轴上的任何点，方法是指定 y 字段的值与文字 0 匹配时匹配。该模式仍然创建一个 x 变量，我们可以在该臂的代码中使用该变量。

类似地，第二个臂通过指定 x 字段的值为 0 时匹配 y 轴上的任意点，并为 y 字段的值创建变量 y。 第三个臂没有指定任何文字，因此它匹配任何其他 Point 并为 x 和 y 字段创建变量。

在此示例中，值 p 通过包含 0 的 x 与第二个臂匹配，因此此代码将在 y 轴上打印 7。

请记住，一旦找到第一个匹配模式，匹配表达式就会停止检查臂，因此即使 Point { x: 0, y: 0} 位于 x 轴和 y 轴上，此代码也只会打印 On the x axis at 0。

#### Destructuring Enums
We've destructured enums in this book (for example, Listing 6-5 in Chapter 6), but haven’t yet explicitly discussed that the pattern to destructure an enum corresponds to the way the data stored within the enum is defined. As an example, in Listing 18-15 we use the Message enum from Listing 6-2 and write a match with patterns that will destructure each inner value.

我们在本书中解构了枚举（例如，第 6 章中的清单 6-5），但尚未明确讨论解构枚举的模式与枚举中存储的数据的定义方式相对应。 例如，在清单 18-15 中，我们使用清单 6-2 中的 Message 枚举，并编写一个匹配模式，该模式将解构每个内部值。

Filename: src/main.rs
``````
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

fn main() {
    let msg = Message::ChangeColor(0, 160, 255);

    match msg {
        Message::Quit => {
            println!("The Quit variant has no data to destructure.");
        }
        Message::Move { x, y } => {
            println!("Move in the x direction {x} and in the y direction {y}");
        }
        Message::Write(text) => {
            println!("Text message: {text}");
        }
        Message::ChangeColor(r, g, b) => {
            println!("Change the color to red {r}, green {g}, and blue {b}",)
        }
    }
}
``````
Listing 18-15: Destructuring enum variants that hold different kinds of values

This code will print Change the color to red 0, green 160, and blue 255. Try changing the value of msg to see the code from the other arms run.

For enum variants without any data, like Message::Quit, we can’t destructure the value any further. We can only match on the literal Message::Quit value, and no variables are in that pattern.

For struct-like enum variants, such as Message::Move, we can use a pattern similar to the pattern we specify to match structs. After the variant name, we place curly brackets and then list the fields with variables so we break apart the pieces to use in the code for this arm. Here we use the shorthand form as we did in Listing 18-13.

For tuple-like enum variants, like Message::Write that holds a tuple with one element and Message::ChangeColor that holds a tuple with three elements, the pattern is similar to the pattern we specify to match tuples. The number of variables in the pattern must match the number of elements in the variant we’re matching.

此代码将打印 Change color to red 0, green 160, and blue 255。尝试更改 msg 的值以查看其他分支运行的代码。

对于没有任何数据的枚举变体，例如 Message::Quit，我们无法进一步解构该值。 我们只能匹配文字 Message::Quit 值，并且该模式中没有变量。

对于类似结构体的枚举变体，例如 Message::Move，我们可以使用与我们指定的模式类似的模式来匹配结构体。 在变体名称之后，我们放置大括号，然后列出带有变量的字段，以便我们分解各个部分以在该手臂的代码中使用。 这里我们使用简写形式，如清单 18-13 所示。

对于类似元组的枚举变体，例如包含一个元素的元组的 Message::Write 和包含三个元素的元组的 Message::ChangeColor，该模式类似于我们指定匹配元组的模式。 模式中的变量数量必须与我们要匹配的变体中的元素数量相匹配。

#### Destructuring Nested Structs and Enums
So far, our examples have all been matching structs or enums one level deep, but matching can work on nested items too! For example, we can refactor the code in Listing 18-15 to support RGB and HSV colors in the ChangeColor message, as shown in Listing 18-16.

到目前为止，我们的示例都匹配一级深度的结构或枚举，但匹配也可以用于嵌套项！ 例如，我们可以重构清单18-15中的代码，以支持ChangeColor消息中的RGB和HSV颜色，如清单18-16所示。
``````
enum Color {
    Rgb(i32, i32, i32),
    Hsv(i32, i32, i32),
}

enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(Color),
}

fn main() {
    let msg = Message::ChangeColor(Color::Hsv(0, 160, 255));

    match msg {
        Message::ChangeColor(Color::Rgb(r, g, b)) => {
            println!("Change color to red {r}, green {g}, and blue {b}");
        }
        Message::ChangeColor(Color::Hsv(h, s, v)) => {
            println!("Change color to hue {h}, saturation {s}, value {v}")
        }
        _ => (),
    }
}
``````
Listing 18-16: Matching on nested enums

The pattern of the first arm in the match expression matches a Message::ChangeColor enum variant that contains a Color::Rgb variant; then the pattern binds to the three inner i32 values. The pattern of the second arm also matches a Message::ChangeColor enum variant, but the inner enum matches Color::Hsv instead. We can specify these complex conditions in one match expression, even though two enums are involved.

匹配表达式中第一个臂的模式与包含 Color::Rgb 变体的 Message::ChangeColor 枚举变体相匹配； 然后该模式绑定到三个内部 i32 值。 第二个臂的模式也与 Message::ChangeColor 枚举变体匹配，但内部枚举与 Color::Hsv 匹配。 即使涉及两个枚举，我们也可以在一个匹配表达式中指定这些复杂的条件。

#### Destructuring Structs and Tuples
We can mix, match, and nest destructuring patterns in even more complex ways. The following example shows a complicated destructure where we nest structs and tuples inside a tuple and destructure all the primitive values out:

我们可以以更复杂的方式混合、匹配和嵌套解构模式。 以下示例显示了一个复杂的解构，其中我们将结构体和元组嵌套在元组内，并解构所有原始值：

``````
    let ((feet, inches), Point { x, y }) = ((3, 10), Point { x: 3, y: -10 });
``````
This code lets us break complex types into their component parts so we can use the values we’re interested in separately.

Destructuring with patterns is a convenient way to use pieces of values, such as the value from each field in a struct, separately from each other.

这段代码让我们可以将复杂类型分解为它们的组成部分，这样我们就可以单独使用我们感兴趣的值。

使用模式解构是一种使用值片段的便捷方法，例如结构中每个字段的值，彼此分开。

#### Ignoring Values in a Pattern
You’ve seen that it’s sometimes useful to ignore values in a pattern, such as in the last arm of a match, to get a catchall that doesn’t actually do anything but does account for all remaining possible values. There are a few ways to ignore entire values or parts of values in a pattern: using the _ pattern (which you’ve seen), using the _ pattern within another pattern, using a name that starts with an underscore, or using .. to ignore remaining parts of a value. Let’s explore how and why to use each of these patterns.

您已经看到，有时忽略模式中的值（例如匹配的最后一个分支中的值）很有用，以获得实际上不执行任何操作但确实考虑了所有剩余可能值的包罗万象的内容。 有几种方法可以忽略模式中的整个值或部分值：使用 _ 模式（您已经见过）、在另一个模式中使用 _ 模式、使用以下划线开头的名称或使用 .. 忽略值的剩余部分。 让我们探讨一下如何以及为何使用这些模式。

#### Ignoring an Entire Value with _
We’ve used the underscore as a wildcard pattern that will match any value but not bind to the value. This is especially useful as the last arm in a match expression, but we can also use it in any pattern, including function parameters, as shown in Listing 18-17.

我们使用下划线作为通配符模式，它将匹配任何值但不绑定到该值。 这作为匹配表达式中的最后一个分支特别有用，但我们也可以在任何模式中使用它，包括函数参数，如清单 18-17 所示。

Filename: src/main.rs
``````
fn foo(_: i32, y: i32) {
    println!("This code only uses the y parameter: {}", y);
}

fn main() {
    foo(3, 4);
}
``````
Listing 18-17: Using _ in a function signature

This code will completely ignore the value 3 passed as the first argument, and will print This code only uses the y parameter: 4.

In most cases when you no longer need a particular function parameter, you would change the signature so it doesn’t include the unused parameter. Ignoring a function parameter can be especially useful in cases when, for example, you're implementing a trait when you need a certain type signature but the function body in your implementation doesn’t need one of the parameters. You then avoid getting a compiler warning about unused function parameters, as you would if you used a name instead.

此代码将完全忽略作为第一个参数传递的值 3，并打印 This code only use the y argument: 4。

在大多数情况下，当您不再需要特定的函数参数时，您可以更改签名，使其不包含未使用的参数。 例如，当您需要某种类型签名但实现中的函数体不需要其中一个参数时，忽略函数参数可能特别有用。 这样，您就可以避免收到有关未使用的函数参数的编译器警告，就像您使用名称一样。

#### Ignoring Parts of a Value with a Nested _
We can also use _ inside another pattern to ignore just part of a value, for example, when we want to test for only part of a value but have no use for the other parts in the corresponding code we want to run. Listing 18-18 shows code responsible for managing a setting’s value. The business requirements are that the user should not be allowed to overwrite an existing customization of a setting but can unset the setting and give it a value if it is currently unset.

我们还可以在另一个模式中使用 _ 来忽略值的一部分，例如，当我们只想测试值的一部分但对我们要运行的相应代码中的其他部分没有用处时。 清单 18-18 显示了负责管理设置值的代码。 业务要求是不允许用户覆盖设置的现有自定义，但可以取消设置并为其指定值（如果当前未设置）。

``````
    let mut setting_value = Some(5);
    let new_setting_value = Some(10);

    match (setting_value, new_setting_value) {
        (Some(_), Some(_)) => {
            println!("Can't overwrite an existing customized value");
        }
        _ => {
            setting_value = new_setting_value;
        }
    }

    println!("setting is {:?}", setting_value);
``````
Listing 18-18: Using an underscore within patterns that match Some variants when we don’t need to use the value inside the Some

This code will print Can't overwrite an existing customized value and then setting is Some(5). In the first match arm, we don’t need to match on or use the values inside either Some variant, but we do need to test for the case when setting_value and new_setting_value are the Some variant. In that case, we print the reason for not changing setting_value, and it doesn’t get changed.

In all other cases (if either setting_value or new_setting_value are None) expressed by the _ pattern in the second arm, we want to allow new_setting_value to become setting_value.

We can also use underscores in multiple places within one pattern to ignore particular values. Listing 18-19 shows an example of ignoring the second and fourth values in a tuple of five items.

此代码将打印 Can't overwrite an existingcustomized value ，然后设置为 Some(5)。 在第一个匹配臂中，我们不需要匹配或使用 Some 变体中的值，但我们确实需要测试setting_value 和 new_setting_value 是 Some 变体时的情况。 在这种情况下，我们打印不更改设置值的原因，并且它不会被更改。

在第二个臂中以 _ 模式表示的所有其他情况下（如果setting_value 或 new_setting_value 为 None），我们希望允许 new_setting_value 变为setting_value。

我们还可以在一种模式中的多个位置使用下划线来忽略特定值。 清单 18-19 显示了忽略五个项目的元组中的第二个和第四个值的示例。

``````
    let numbers = (2, 4, 8, 16, 32);

    match numbers {
        (first, _, third, _, fifth) => {
            println!("Some numbers: {first}, {third}, {fifth}")
        }
    }
``````
Listing 18-19: Ignoring multiple parts of a tuple

This code will print Some numbers: 2, 8, 32, and the values 4 and 16 will be ignored.

Ignoring an Unused Variable by Starting Its Name with _
If you create a variable but don’t use it anywhere, Rust will usually issue a warning because an unused variable could be a bug. However, sometimes it’s useful to be able to create a variable you won’t use yet, such as when you’re prototyping or just starting a project. In this situation, you can tell Rust not to warn you about the unused variable by starting the name of the variable with an underscore. In Listing 18-20, we create two unused variables, but when we compile this code, we should only get a warning about one of them.

通过名称以 _ 开头来忽略未使用的变量
如果您创建一个变量但不在任何地方使用它，Rust 通常会发出警告，因为未使用的变量可能是一个错误。 然而，有时能够创建一个您还不会使用的变量是很有用的，例如当您正在制作原型或刚刚启动一个项目时。 在这种情况下，您可以通过以下划线开头的变量名称来告诉 Rust 不要警告您有关未使用变量的信息。 在清单 18-20 中，我们创建了两个未使用的变量，但是当我们编译此代码时，我们应该只收到有关其中一个的警告。

Filename: src/main.rs
``````
fn main() {
    let _x = 5;
    let y = 10;
}
``````
Listing 18-20: Starting a variable name with an underscore to avoid getting unused variable warnings

Here we get a warning about not using the variable y, but we don’t get a warning about not using _x.

Note that there is a subtle difference between using only _ and using a name that starts with an underscore. The syntax _x still binds the value to the variable, whereas _ doesn’t bind at all. To show a case where this distinction matters, Listing 18-21 will provide us with an error.

在这里，我们收到有关不使用变量 y 的警告，但没有收到有关不使用 _x 的警告。

请注意，仅使用 _ 和使用以下划线开头的名称之间存在细微差别。 语法 _x 仍然将值绑定到变量，而 _ 根本不绑定。 为了展示这种区别很重要的情况，清单 18-21 将为我们提供一个错误。

``````
This code does not compile!
    let s = Some(String::from("Hello!"));

    if let Some(_s) = s {
        println!("found a string");
    }

    println!("{:?}", s);
``````
Listing 18-21: An unused variable starting with an underscore still binds the value, which might take ownership of the value

We’ll receive an error because the s value will still be moved into _s, which prevents us from using s again. However, using the underscore by itself doesn’t ever bind to the value. Listing 18-22 will compile without any errors because s doesn’t get moved into _.

我们会收到一个错误，因为 s 值仍然会被移动到 _s 中，这会阻止我们再次使用 s。 但是，单独使用下划线并不绑定到该值。 清单 18-22 编译时不会出现任何错误，因为 s 不会移动到 _ 中。

``````
    let s = Some(String::from("Hello!"));

    if let Some(_) = s {
        println!("found a string");
    }

    println!("{:?}", s);
``````
Listing 18-22: Using an underscore does not bind the value

This code works just fine because we never bind s to anything; it isn’t moved.

### Ignoring Remaining Parts of a Value with ..
With values that have many parts, we can use the .. syntax to use specific parts and ignore the rest, avoiding the need to list underscores for each ignored value. The .. pattern ignores any parts of a value that we haven’t explicitly matched in the rest of the pattern. In Listing 18-23, we have a Point struct that holds a coordinate in three-dimensional space. In the match expression, we want to operate only on the x coordinate and ignore the values in the y and z fields.

对于包含多个部分的值，我们可以使用 .. 语法来使用特定部分并忽略其余部分，从而避免需要为每个忽略的值列出下划线。 .. 模式会忽略我们在模式的其余部分中未显式匹配的值的任何部分。 在清单 18-23 中，我们有一个 Point 结构，它保存三维空间中的坐标。 在匹配表达式中，我们只想对 x 坐标进行操作，而忽略 y 和 z 字段中的值。
``````
    struct Point {
        x: i32,
        y: i32,
        z: i32,
    }

    let origin = Point { x: 0, y: 0, z: 0 };

    match origin {
        Point { x, .. } => println!("x is {}", x),
    }
``````
Listing 18-23: Ignoring all fields of a Point except for x by using ..

We list the x value and then just include the .. pattern. This is quicker than having to list y: _ and z: _, particularly when we’re working with structs that have lots of fields in situations where only one or two fields are relevant.

The syntax .. will expand to as many values as it needs to be. Listing 18-24 shows how to use .. with a tuple.

我们列出 x 值，然后仅包含 .. 模式。 这比列出 y: _ 和 z: _ 更快，特别是当我们在只有一两个字段相关的情况下使用具有大量字段的结构时。

语法 .. 将扩展为所需数量的值。 清单 18-24 展示了如何将 .. 与元组一起使用。

Filename: src/main.rs
``````
fn main() {
    let numbers = (2, 4, 8, 16, 32);

    match numbers {
        (first, .., last) => {
            println!("Some numbers: {first}, {last}");
        }
    }
}
``````
Listing 18-24: Matching only the first and last values in a tuple and ignoring all other values

In this code, the first and last value are matched with first and last. The .. will match and ignore everything in the middle.

However, using .. must be unambiguous. If it is unclear which values are intended for matching and which should be ignored, Rust will give us an error. Listing 18-25 shows an example of using .. ambiguously, so it will not compile.

在此代码中，第一个和最后一个值与第一个和最后一个匹配。 .. 将匹配并忽略中间的所有内容。

然而，使用 .. 必须是明确的。 如果不清楚哪些值要匹配，哪些值应该被忽略，Rust 会给我们一个错误。 清单 18-25 显示了一个不明确地使用 .. 的示例，因此它无法编译。

Filename: src/main.rs
``````
This code does not compile!
fn main() {
    let numbers = (2, 4, 8, 16, 32);

    match numbers {
        (.., second, ..) => {
            println!("Some numbers: {}", second)
        },
    }
}
``````
Listing 18-25: An attempt to use .. in an ambiguous way

When we compile this example, we get this error:
``````
$ cargo run
   Compiling patterns v0.1.0 (file:///projects/patterns)
error: `..` can only be used once per tuple pattern
 --> src/main.rs:5:22
  |
5 |         (.., second, ..) => {
  |          --          ^^ can only be used once per tuple pattern
  |          |
  |          previously used here

error: could not compile `patterns` due to previous error
``````
It’s impossible for Rust to determine how many values in the tuple to ignore before matching a value with second and then how many further values to ignore thereafter. This code could mean that we want to ignore 2, bind second to 4, and then ignore 8, 16, and 32; or that we want to ignore 2 and 4, bind second to 8, and then ignore 16 and 32; and so forth. The variable name second doesn’t mean anything special to Rust, so we get a compiler error because using .. in two places like this is ambiguous.

Rust 不可能确定在将某个值与第二个值匹配之前要忽略元组中的多少个值，以及此后要忽略多少个值。 这段代码可能意味着我们想要忽略 2，将第二个绑定到 4，然后忽略 8、16 和 32； 或者我们想忽略 2 和 4，将第二个绑定到 8，然后忽略 16 和 32； 等等。 变量名 Second 对 Rust 来说没有任何特殊意义，所以我们会得到一个编译器错误，因为在这样的两个地方使用 .. 是不明确的。

### Extra Conditionals with Match Guards
A match guard is an additional if condition, specified after the pattern in a match arm, that must also match for that arm to be chosen. Match guards are useful for expressing more complex ideas than a pattern alone allows.

The condition can use variables created in the pattern. Listing 18-26 shows a match where the first arm has the pattern Some(x) and also has a match guard of if x % 2 == 0 (which will be true if the number is even).

匹配防护是一个附加的 if 条件，在匹配臂中的模式之后指定，它也必须匹配才能选择该臂。 匹配守卫对于表达比单独模式所允许的更复杂的想法非常有用。

条件可以使用模式中创建的变量。 清单 18-26 显示了一个匹配，其中第一个臂具有模式 Some(x)，并且还具有 if x % 2 == 0 的匹配防护（如果数字为偶数，则为 true）。

``````
    let num = Some(4);

    match num {
        Some(x) if x % 2 == 0 => println!("The number {} is even", x),
        Some(x) => println!("The number {} is odd", x),
        None => (),
    }
``````
Listing 18-26: Adding a match guard to a pattern

This example will print The number 4 is even. When num is compared to the pattern in the first arm, it matches, because Some(4) matches Some(x). Then the match guard checks whether the remainder of dividing x by 2 is equal to 0, and because it is, the first arm is selected.

If num had been Some(5) instead, the match guard in the first arm would have been false because the remainder of 5 divided by 2 is 1, which is not equal to 0. Rust would then go to the second arm, which would match because the second arm doesn’t have a match guard and therefore matches any Some variant.

There is no way to express the if x % 2 == 0 condition within a pattern, so the match guard gives us the ability to express this logic. The downside of this additional expressiveness is that the compiler doesn't try to check for exhaustiveness when match guard expressions are involved.

In Listing 18-11, we mentioned that we could use match guards to solve our pattern-shadowing problem. Recall that we created a new variable inside the pattern in the match expression instead of using the variable outside the match. That new variable meant we couldn’t test against the value of the outer variable. Listing 18-27 shows how we can use a match guard to fix this problem.

此示例将打印 The number 4 is Even。 当 num 与第一个臂中的模式进行比较时，它匹配，因为 Some(4) 与 Some(x) 匹配。 然后比赛守卫检查 x 除以 2 的余数是否等于 0，因为是，所以选择第一个臂。

如果 num 为 Some(5)，则第一个分支中的匹配守卫将为 false，因为 5 除以 2 的余数为 1，不等于 0。然后 Rust 将转到第二个分支，这将 匹配，因为第二条臂没有匹配守卫，因此匹配任何 Some 变体。

无法在模式中表达 if x % 2 == 0 条件，因此匹配守卫使我们能够表达此逻辑。 这种额外表达能力的缺点是，当涉及匹配保护表达式时，编译器不会尝试检查是否详尽。

在清单18-11中，我们提到我们可以使用匹配守卫来解决我们的模式阴影问题。 回想一下，我们在匹配表达式的模式内部创建了一个新变量，而不是使用匹配外部的变量。 这个新变量意味着我们无法测试外部变量的值。 清单18-27展示了我们如何使用匹配守卫来解决这个问题。

Filename: src/main.rs
``````
fn main() {
    let x = Some(5);
    let y = 10;

    match x {
        Some(50) => println!("Got 50"),
        Some(n) if n == y => println!("Matched, n = {n}"),
        _ => println!("Default case, x = {:?}", x),
    }

    println!("at the end: x = {:?}, y = {y}", x);
}
``````
Listing 18-27: Using a match guard to test for equality with an outer variable

This code will now print Default case, x = Some(5). The pattern in the second match arm doesn’t introduce a new variable y that would shadow the outer y, meaning we can use the outer y in the match guard. Instead of specifying the pattern as Some(y), which would have shadowed the outer y, we specify Some(n). This creates a new variable n that doesn’t shadow anything because there is no n variable outside the match.

The match guard if n == y is not a pattern and therefore doesn’t introduce new variables. This y is the outer y rather than a new shadowed y, and we can look for a value that has the same value as the outer y by comparing n to y.

You can also use the or operator | in a match guard to specify multiple patterns; the match guard condition will apply to all the patterns. Listing 18-28 shows the precedence when combining a pattern that uses | with a match guard. The important part of this example is that the if y match guard applies to 4, 5, and 6, even though it might look like if y only applies to 6.

此代码现在将打印默认情况，x = Some(5)。 第二个匹配臂中的模式不会引入新的变量 y 来遮蔽外部 y，这意味着我们可以在匹配防护中使用外部 y。 我们没有将模式指定为 Some(y)，因为这会遮盖外部 y，而是指定 Some(n)。 这会创建一个新变量 n，它不会隐藏任何内容，因为匹配之外没有 n 变量。

匹配守卫 if n == y 不是一个模式，因此不会引入新变量。 这个y是外部y而不是新的阴影y，我们可以通过比较n和y来寻找与外部y具有相同值的值。

您还可以使用或运算符 | 在比赛守卫中指定多种模式； 比赛守卫条件将应用于所有模式。 清单 18-28 显示了组合使用 | 的模式时的优先级。 与一名比赛后卫。 这个例子的重要部分是 if y 匹配守卫适用于 4、5 和 6，尽管它可能看起来像 if y 仅适用于 6。

``````
    let x = 4;
    let y = false;

    match x {
        4 | 5 | 6 if y => println!("yes"),
        _ => println!("no"),
    }
``````
Listing 18-28: Combining multiple patterns with a match guard

The match condition states that the arm only matches if the value of x is equal to 4, 5, or 6 and if y is true. When this code runs, the pattern of the first arm matches because x is 4, but the match guard if y is false, so the first arm is not chosen. The code moves on to the second arm, which does match, and this program prints no. The reason is that the if condition applies to the whole pattern 4 | 5 | 6, not only to the last value 6. In other words, the precedence of a match guard in relation to a pattern behaves like this:

匹配条件规定，仅当 x 的值等于 4、5 或 6 并且 y 为 true 时，该臂才匹配。 当此代码运行时，第一个臂的模式匹配，因为 x 是 4，但如果 y 为 false，则匹配守卫，因此不会选择第一个臂。 代码移至第二条臂，它确实匹配，并且该程序打印 no。 原因是 if 条件适用于整个模式 4 | 5 | 6，不仅仅是最后一个值 6。换句话说，匹配守卫相对于模式的优先级的行为如下：

``````
(4 | 5 | 6) if y => ...
``````
rather than this:
``````
4 | 5 | (6 if y) => ...
``````
After running the code, the precedence behavior is evident: if the match guard were applied only to the final value in the list of values specified using the | operator, the arm would have matched and the program would have printed yes.

运行代码后，优先行为很明显：如果匹配守卫仅应用于使用 | 指定的值列表中的最终值。 操作员，手臂会匹配，程序会打印 yes。

### @ Bindings
The at operator @ lets us create a variable that holds a value at the same time as we’re testing that value for a pattern match. In Listing 18-29, we want to test that a Message::Hello id field is within the range 3..=7. We also want to bind the value to the variable id_variable so we can use it in the code associated with the arm. We could name this variable id, the same as the field, but for this example we’ll use a different name.

at 运算符 @ 允许我们创建一个变量，该变量在测试模式匹配值的同时保存一个值。 在清单 18-29 中，我们要测试 Message::Hello id 字段是否在 3..=7 范围内。 我们还希望将该值绑定到变量 id_variable，以便我们可以在与手臂相关的代码中使用它。 我们可以将此变量命名为 id，与字段相同，但在本例中我们将使用不同的名称。

``````
    enum Message {
        Hello { id: i32 },
    }

    let msg = Message::Hello { id: 5 };

    match msg {
        Message::Hello {
            id: id_variable @ 3..=7,
        } => println!("Found an id in range: {}", id_variable),
        Message::Hello { id: 10..=12 } => {
            println!("Found an id in another range")
        }
        Message::Hello { id } => println!("Found some other id: {}", id),
    }
``````
Listing 18-29: Using @ to bind to a value in a pattern while also testing it

This example will print Found an id in range: 5. By specifying id_variable @ before the range 3..=7, we’re capturing whatever value matched the range while also testing that the value matched the range pattern.

In the second arm, where we only have a range specified in the pattern, the code associated with the arm doesn’t have a variable that contains the actual value of the id field. The id field’s value could have been 10, 11, or 12, but the code that goes with that pattern doesn’t know which it is. The pattern code isn’t able to use the value from the id field, because we haven’t saved the id value in a variable.

In the last arm, where we’ve specified a variable without a range, we do have the value available to use in the arm’s code in a variable named id. The reason is that we’ve used the struct field shorthand syntax. But we haven’t applied any test to the value in the id field in this arm, as we did with the first two arms: any value would match this pattern.

Using @ lets us test a value and save it in a variable within one pattern.

此示例将打印 Found an id in range: 5。通过在范围 3..=7 之前指定 id_variable @，我们将捕获与范围匹配的任何值，同时还测试该值是否与范围模式匹配。

在第二个臂中，我们仅在模式中指定了一个范围，与该臂关联的代码没有包含 id 字段实际值的变量。 id 字段的值可能是 10、11 或 12，但与该模式相关的代码不知道它是什么。 模式代码无法使用 id 字段中的值，因为我们尚未将 id 值保存在变量中。

在最后一个手臂中，我们指定了一个没有范围的变量，我们确实有可在手臂代码中名为 id 的变量中使用的值。 原因是我们使用了 struct field 简写语法。 但是我们没有像对前两个臂所做的那样对此臂中的 id 字段中的值应用任何测试：任何值都将与此模式匹配。

使用@可以让我们测试一个值并将其保存在一个模式内的变量中。


## 18.4 Summary
Rust’s patterns are very useful in distinguishing between different kinds of data. When used in match expressions, Rust ensures your patterns cover every possible value, or your program won’t compile. Patterns in let statements and function parameters make those constructs more useful, enabling the destructuring of values into smaller parts at the same time as assigning to variables. We can create simple or complex patterns to suit our needs.

Rust 的模式对于区分不同类型的数据非常有用。 当在匹配表达式中使用时，Rust 确保您的模式覆盖所有可能的值，否则您的程序将无法编译。 let 语句和函数参数中的模式使这些构造更加有用，可以在分配给变量的同时将值解构为更小的部分。 我们可以创建简单或复杂的图案来满足我们的需求。

Next, for the penultimate chapter of the book, we’ll look at some advanced aspects of a variety of Rust’s features.