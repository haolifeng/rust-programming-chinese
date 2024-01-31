# 5. 结构

A struct, or structure, is a custom data type that lets you package together and name multiple related values that make up a meaningful group. If you’re familiar with an object-oriented language, a struct is like an object’s data attributes. In this chapter, we’ll compare and contrast tuples with structs to build on what you already know and demonstrate when structs are a better way to group data.

We’ll demonstrate how to define and instantiate structs. We’ll discuss how to define associated functions, especially the kind of associated functions called methods, to specify behavior associated with a struct type. Structs and enums (discussed in Chapter 6) are the building blocks for creating new types in your program’s domain to take full advantage of Rust’s compile-time type checking.

结构体是一种自定义数据类型，可让您将多个相关值打包并命名，从而组成一个有意义的组。 如果您熟悉面向对象的语言，结构就像对象的数据属性。 在本章中，我们将在您已知的基础上比较元组和结构体，并演示结构体何时是更好的数据分组方式。

我们将演示如何定义和实例化结构。 我们将讨论如何定义关联函数，特别是称为方法的关联函数，以指定与结构类型关联的行为。 结构体和枚举（在第 6 章中讨论）是在程序域中创建新类型的构建块，以充分利用 Rust 的编译时类型检查。

## 5.1 定义和实例化(Defining and Instantiating Structs)
Structs are similar to tuples, discussed in “The Tuple Type” section, in that both hold multiple related values. Like tuples, the pieces of a struct can be different types. Unlike with tuples, in a struct you’ll name each piece of data so it’s clear what the values mean. Adding these names means that structs are more flexible than tuples: you don’t have to rely on the order of the data to specify or access the values of an instance.

To define a struct, we enter the keyword struct and name the entire struct. A struct’s name should describe the significance of the pieces of data being grouped together. Then, inside curly brackets, we define the names and types of the pieces of data, which we call fields. For example, Listing 5-1 shows a struct that stores information about a user account.

结构体类似于“元组类型”部分中讨论的元组，因为两者都保存多个相关值。 与元组一样，结构体的各个部分可以是不同的类型。 与元组不同，在结构体中，您将为每条数据命名，以便清楚地了解这些值的含义。 添加这些名称意味着结构比元组更灵活：您不必依赖数据的顺序来指定或访问实例的值。

要定义结构，我们输入关键字 struct 并命名整个结构。 结构体的名称应该描述分组在一起的数据片段的重要性。 然后，在大括号内，我们定义数据块的名称和类型，我们将其称为字段。 例如，清单 5-1 显示了一个存储有关用户帐户信息的结构体。

Filename: src/main.rs
``````
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}
``````
Listing 5-1: A User struct definition

To use a struct after we’ve defined it, we create an instance of that struct by specifying concrete values for each of the fields. We create an instance by stating the name of the struct and then add curly brackets containing key: value pairs, where the keys are the names of the fields and the values are the data we want to store in those fields. We don’t have to specify the fields in the same order in which we declared them in the struct. In other words, the struct definition is like a general template for the type, and instances fill in that template with particular data to create values of the type. For example, we can declare a particular user as shown in Listing 5-2.

要在定义结构后使用它，我们通过为每个字段指定具体值来创建该结构的实例。 我们通过声明结构的名称来创建一个实例，然后添加包含键：值对的大括号，其中键是字段的名称，值是我们要存储在这些字段中的数据。 我们不必按照在结构中声明字段的顺序指定字段。 换句话说，结构定义就像该类型的通用模板，实例使用特定数据填充该模板以创建该类型的值。 例如，我们可以声明一个特定的用户，如清单 5-2 所示。

Filename: src/main.rs
``````
fn main() {
    let user1 = User {
        active: true,
        username: String::from("someusername123"),
        email: String::from("someone@example.com"),
        sign_in_count: 1,
    };
}
``````
Listing 5-2: Creating an instance of the User struct

To get a specific value from a struct, we use dot notation. For example, to access this user’s email address, we use user1.email. If the instance is mutable, we can change a value by using the dot notation and assigning into a particular field. Listing 5-3 shows how to change the value in the email field of a mutable User instance.

为了从结构中获取特定值，我们使用点表示法。 例如，要访问该用户的电子邮件地址，我们使用 user1.email。 如果实例是可变的，我们可以通过使用点符号并分配到特定字段来更改值。 清单 5-3 显示了如何更改可变 User 实例的电子邮件字段中的值。

Filename: src/main.rs
``````
fn main() {
    let mut user1 = User {
        active: true,
        username: String::from("someusername123"),
        email: String::from("someone@example.com"),
        sign_in_count: 1,
    };

    user1.email = String::from("anotheremail@example.com");
}
``````
Listing 5-3: Changing the value in the email field of a User instance

Note that the entire instance must be mutable; Rust doesn’t allow us to mark only certain fields as mutable. As with any expression, we can construct a new instance of the struct as the last expression in the function body to implicitly return that new instance.

Listing 5-4 shows a build_user function that returns a User instance with the given email and username. The active field gets the value of true, and the sign_in_count gets a value of 1.

请注意，整个实例必须是可变的； Rust 不允许我们仅将某些字段标记为可变。 与任何表达式一样，我们可以构造结构体的新实例作为函数体中的最后一个表达式，以隐式返回该新实例。

清单 5-4 显示了一个 build_user 函数，它返回具有给定电子邮件和用户名的 User 实例。 active 字段的值为 true，sign_in_count 的值为 1。

Filename: src/main.rs
``````
fn build_user(email: String, username: String) -> User {
    User {
        active: true,
        username: username,
        email: email,
        sign_in_count: 1,
    }
}
``````
Listing 5-4: A build_user function that takes an email and username and returns a User instance

It makes sense to name the function parameters with the same name as the struct fields, but having to repeat the email and username field names and variables is a bit tedious. If the struct had more fields, repeating each name would get even more annoying. Luckily, there’s a convenient shorthand!

使用与结构字段相同的名称来命名函数参数是有意义的，但必须重复电子邮件和用户名字段名称和变量有点乏味。 如果结构体有更多字段，重复每个名称会变得更加烦人。 幸运的是，有一个方便的简写！

### Using the Field Init Shorthand
Because the parameter names and the struct field names are exactly the same in Listing 5-4, we can use the field init shorthand syntax to rewrite build_user so it behaves exactly the same but doesn’t have the repetition of username and email, as shown in Listing 5-5.

由于清单 5-4 中的参数名称和结构体字段名称完全相同，因此我们可以使用 field init 简写语法来重写 build_user，使其行为完全相同，但没有用户名和电子邮件的重复，如图所示 如清单 5-5 所示。

Filename: src/main.rs
``````
fn build_user(email: String, username: String) -> User {
    User {
        active: true,
        username,
        email,
        sign_in_count: 1,
    }
}
``````
Listing 5-5: A build_user function that uses field init shorthand because the username and email parameters have the same name as struct fields

Here, we’re creating a new instance of the User struct, which has a field named email. We want to set the email field’s value to the value in the email parameter of the build_user function. Because the email field and the email parameter have the same name, we only need to write email rather than email: email.

在这里，我们正在创建 User 结构的一个新实例，其中有一个名为 email 的字段。 我们希望将 email 字段的值设置为 build_user 函数的 email 参数中的值。 因为email字段和email参数同名，所以我们只需要写email而不是email:email。

### Creating Instances from Other Instances with Struct Update Syntax
It’s often useful to create a new instance of a struct that includes most of the values from another instance, but changes some. You can do this using struct update syntax.

First, in Listing 5-6 we show how to create a new User instance in user2 regularly, without the update syntax. We set a new value for email but otherwise use the same values from user1 that we created in Listing 5-2.

创建一个结构体的新实例通常很有用，该实例包含另一个实例的大部分值，但更改了一些值。 您可以使用结构更新语法来完成此操作。

首先，在清单 5-6 中，我们展示了如何定期在 user2 中创建一个新的 User 实例，而不使用 update 语法。 我们为 email 设置了一个新值，但在其他方面使用我们在清单 5-2 中创建的 user1 中的相同值。

Filename: src/main.rs
``````
fn main() {
    // --snip--

    let user2 = User {
        active: user1.active,
        username: user1.username,
        email: String::from("another@example.com"),
        sign_in_count: user1.sign_in_count,
    };
}
``````
Listing 5-6: Creating a new User instance using one of the values from user1

Using struct update syntax, we can achieve the same effect with less code, as shown in Listing 5-7. The syntax .. specifies that the remaining fields not explicitly set should have the same value as the fields in the given instance.

使用struct update语法，我们可以用更少的代码实现相同的效果，如清单5-7所示。 语法 .. 指定未显式设置的其余字段应具有与给定实例中的字段相同的值。

Filename: src/main.rs
``````
fn main() {
    // --snip--

    let user2 = User {
        email: String::from("another@example.com"),
        ..user1
    };
}
``````
Listing 5-7: Using struct update syntax to set a new email value for a User instance but to use the rest of the values from user1

The code in Listing 5-7 also creates an instance in user2 that has a different value for email but has the same values for the username, active, and sign_in_count fields from user1. The ..user1 must come last to specify that any remaining fields should get their values from the corresponding fields in user1, but we can choose to specify values for as many fields as we want in any order, regardless of the order of the fields in the struct’s definition.

Note that the struct update syntax uses = like an assignment; this is because it moves the data, just as we saw in the “Variables and Data Interacting with Move” section. In this example, we can no longer use user1 as a whole after creating user2 because the String in the username field of user1 was moved into user2. If we had given user2 new String values for both email and username, and thus only used the active and sign_in_count values from user1, then user1 would still be valid after creating user2. Both active and sign_in_count are types that implement the Copy trait, so the behavior we discussed in the “Stack-Only Data: Copy” section would apply.

清单 5-7 中的代码还在 user2 中创建了一个实例，该实例的电子邮件值不同，但 username、active 和 sign_in_count 字段的值与 user1 相同。 ..user1 必须放在最后，以指定任何剩余字段应从 user1 中的相应字段获取其值，但我们可以选择以任何顺序为任意数量的字段指定值，而不管中字段的顺序如何 结构体的定义。

请注意，结构体更新语法使用 = 就像赋值一样； 这是因为它移动数据，正如我们在“与移动交互的变量和数据”部分中看到的那样。 在这个例子中，我们在创建user2之后就不能再整体使用user1了，因为user1的用户名字段中的String被移到了user2中。 如果我们为 user2 提供了电子邮件和用户名的新字符串值，因此仅使用了 user1 中的 active 和 sign_in_count 值，则在创建 user2 后，user1 仍然有效。 active 和sign_in_count 都是实现 Copy 特征的类型，因此我们在“仅堆栈数据：复制”部分中讨论的行为将适用。


### Using Tuple Structs Without Named Fields to Create Different Types
Rust also supports structs that look similar to tuples, called tuple structs. Tuple structs have the added meaning the struct name provides but don’t have names associated with their fields; rather, they just have the types of the fields. Tuple structs are useful when you want to give the whole tuple a name and make the tuple a different type from other tuples, and when naming each field as in a regular struct would be verbose or redundant.

To define a tuple struct, start with the struct keyword and the struct name followed by the types in the tuple. For example, here we define and use two tuple structs named Color and Point:

Rust 还支持看起来类似于元组的结构，称为元组结构。 元组结构具有结构名称提供的附加含义，但没有与其字段关联的名称； 相反，它们只有字段的类型。 当您想要为整个元组指定一个名称并使该元组与其他元组具有不同的类型，并且像在常规结构中那样命名每个字段时会很冗长或多余时，元组结构非常有用。

要定义元组结构体，请从 struct 关键字和结构体名称开始，后跟元组中的类型。 例如，这里我们定义并使用两个名为 Color 和 Point 的元组结构：

Filename: src/main.rs
``````
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

fn main() {
    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);
}
``````
Note that the black and origin values are different types because they’re instances of different tuple structs. Each struct you define is its own type, even though the fields within the struct might have the same types. For example, a function that takes a parameter of type Color cannot take a Point as an argument, even though both types are made up of three i32 values. Otherwise, tuple struct instances are similar to tuples in that you can destructure them into their individual pieces, and you can use a . followed by the index to access an individual value.

请注意，黑色和原点值是不同的类型，因为它们是不同元组结构的实例。 您定义的每个结构都是其自己的类型，即使结构中的字段可能具有相同的类型。 例如，采用 Color 类型参数的函数不能采用 Point 作为参数，即使这两种类型均由三个 i32 值组成。 否则，元组结构实例与元组类似，您可以将它们解构为单独的部分，并且可以使用 . 后跟索引以访问单个值。

### Unit-Like Structs Without Any Fields
You can also define structs that don’t have any fields! These are called unit-like structs because they behave similarly to (), the unit type that we mentioned in “The Tuple Type” section. Unit-like structs can be useful when you need to implement a trait on some type but don’t have any data that you want to store in the type itself. We’ll discuss traits in Chapter 10. Here’s an example of declaring and instantiating a unit struct named AlwaysEqual:

您还可以定义没有任何字段的结构！ 这些被称为类单元结构，因为它们的行为类似于我们在“元组类型”部分中提到的单元类型 ()。 当您需要在某种类型上实现特征但没有想要在类型本身中存储任何数据时，类似单元的结构会很有用。 我们将在第 10 章中讨论特征。下面是声明和实例化名为 AlwaysEqual 的单元结构的示例：

Filename: src/main.rs
``````
struct AlwaysEqual;

fn main() {
    let subject = AlwaysEqual;
}
``````
To define AlwaysEqual, we use the struct keyword, the name we want, and then a semicolon. No need for curly brackets or parentheses! Then we can get an instance of AlwaysEqual in the subject variable in a similar way: using the name we defined, without any curly brackets or parentheses. Imagine that later we’ll implement behavior for this type such that every instance of AlwaysEqual is always equal to every instance of any other type, perhaps to have a known result for testing purposes. We wouldn’t need any data to implement that behavior! You’ll see in Chapter 10 how to define traits and implement them on any type, including unit-like structs.

为了定义 AlwaysEqual，我们使用 struct 关键字、我们想要的名称，然后是分号。 不需要大括号或圆括号！ 然后我们可以以类似的方式在主题变量中获取AlwaysEqual的实例：使用我们定义的名称，不带任何大括号或圆括号。 想象一下，稍后我们将实现此类型的行为，使得 AlwaysEqual 的每个实例始终等于任何其他类型的每个实例，也许为了测试目的而获得已知结果。 我们不需要任何数据来实现该行为！ 您将在第 10 章中看到如何定义特征并在任何类型（包括类似单元的结构）上实现它们。

### Ownership of Struct Data
In the User struct definition in Listing 5-1, we used the owned String type rather than the &str string slice type. This is a deliberate choice because we want each instance of this struct to own all of its data and for that data to be valid for as long as the entire struct is valid.

It’s also possible for structs to store references to data owned by something else, but to do so requires the use of lifetimes, a Rust feature that we’ll discuss in Chapter 10. Lifetimes ensure that the data referenced by a struct is valid for as long as the struct is. Let’s say you try to store a reference in a struct without specifying lifetimes, like the following; this won’t work:

在清单 5-1 的 User 结构体定义中，我们使用了拥有的 String 类型而不是 &str 字符串切片类型。 这是一个经过深思熟虑的选择，因为我们希望该结构的每个实例都拥有其所有数据，并且只要整个结构有效，该数据就有效。

结构体也可以存储对其他对象所拥有的数据的引用，但这样做需要使用生命周期，这是我们将在第 10 章中讨论的 Rust 功能。生命周期确保结构体引用的数据在以下情况下有效： 只要结构是。 假设您尝试在结构中存储引用而不指定生命周期，如下所示； 这是行不通的：

Filename: src/main.rs
``````
This code does not compile!
struct User {
    active: bool,
    username: &str,
    email: &str,
    sign_in_count: u64,
}

fn main() {
    let user1 = User {
        active: true,
        username: "someusername123",
        email: "someone@example.com",
        sign_in_count: 1,
    };
}
The compiler will complain that it needs lifetime specifiers:


$ cargo run
   Compiling structs v0.1.0 (file:///projects/structs)
error[E0106]: missing lifetime specifier
 --> src/main.rs:3:15
  |
3 |     username: &str,
  |               ^ expected named lifetime parameter
  |
help: consider introducing a named lifetime parameter
  |
1 ~ struct User<'a> {
2 |     active: bool,
3 ~     username: &'a str,
  |

error[E0106]: missing lifetime specifier
 --> src/main.rs:4:12
  |
4 |     email: &str,
  |            ^ expected named lifetime parameter
  |
help: consider introducing a named lifetime parameter
  |
1 ~ struct User<'a> {
2 |     active: bool,
3 |     username: &str,
4 ~     email: &'a str,
  |

For more information about this error, try `rustc --explain E0106`.
error: could not compile `structs` due to 2 previous errors
``````
In Chapter 10, we’ll discuss how to fix these errors so you can store references in structs, but for now, we’ll fix errors like these using owned types like String instead of references like &str.

在第 10 章中，我们将讨论如何修复这些错误，以便您可以在结构中存储引用，但现在，我们将使用 String 等自有类型而不是 &str 等引用来修复此类错误。

## 5.2 An Example Program Using Structs
To understand when we might want to use structs, let’s write a program that calculates the area of a rectangle. We’ll start by using single variables, and then refactor the program until we’re using structs instead.

Let’s make a new binary project with Cargo called rectangles that will take the width and height of a rectangle specified in pixels and calculate the area of the rectangle. Listing 5-8 shows a short program with one way of doing exactly that in our project’s src/main.rs.

为了了解何时需要使用结构体，让我们编写一个计算矩形面积的程序。 我们将从使用单个变量开始，然后重构程序，直到我们使用结构体。

让我们用 Cargo 创建一个名为矩形的新二进制项目，它将采用以像素为单位指定的矩形的宽度和高度并计算矩形的面积。 清单 5-8 显示了一个简短的程序，其中的一种方法正是在我们项目的 src/main.rs 中执行此操作。

Filename: src/main.rs
``````
fn main() {
    let width1 = 30;
    let height1 = 50;

    println!(
        "The area of the rectangle is {} square pixels.",
        area(width1, height1)
    );
}

fn area(width: u32, height: u32) -> u32 {
    width * height
}
``````
Listing 5-8: Calculating the area of a rectangle specified by separate width and height variables

Now, run this program using cargo run:
``````
$ cargo run
   Compiling rectangles v0.1.0 (file:///projects/rectangles)
    Finished dev [unoptimized + debuginfo] target(s) in 0.42s
     Running `target/debug/rectangles`
The area of the rectangle is 1500 square pixels.
``````
This code succeeds in figuring out the area of the rectangle by calling the area function with each dimension, but we can do more to make this code clear and readable.

The issue with this code is evident in the signature of area:
``````
fn area(width: u32, height: u32) -> u32 {
``````
The area function is supposed to calculate the area of one rectangle, but the function we wrote has two parameters, and it’s not clear anywhere in our program that the parameters are related. It would be more readable and more manageable to group width and height together. We’ve already discussed one way we might do that in “The Tuple Type” section of Chapter 3: by using tuples.

面积函数应该计算一个矩形的面积，但是我们编写的函数有两个参数，并且我们程序中的任何地方都不清楚这些参数是相关的。 将宽度和高度组合在一起会更具可读性和更易于管理。 我们已经在第 3 章的“元组类型”部分讨论了一种可以做到这一点的方法：使用元组。

### 使用元组重构
Listing 5-9 shows another version of our program that uses tuples.

Filename: src/main.rs
``````
fn main() {
    let rect1 = (30, 50);

    println!(
        "The area of the rectangle is {} square pixels.",
        area(rect1)
    );
}

fn area(dimensions: (u32, u32)) -> u32 {
    dimensions.0 * dimensions.1
}
``````
Listing 5-9: Specifying the width and height of the rectangle with a tuple

In one way, this program is better. Tuples let us add a bit of structure, and we’re now passing just one argument. But in another way, this version is less clear: tuples don’t name their elements, so we have to index into the parts of the tuple, making our calculation less obvious.

Mixing up the width and height wouldn’t matter for the area calculation, but if we want to draw the rectangle on the screen, it would matter! We would have to keep in mind that width is the tuple index 0 and height is the tuple index 1. This would be even harder for someone else to figure out and keep in mind if they were to use our code. Because we haven’t conveyed the meaning of our data in our code, it’s now easier to introduce errors.

从某种意义上说，这个程序更好。 元组让我们添加一点结构，现在我们只传递一个参数。 但从另一方面来说，这个版本不太清楚：元组不命名它们的元素，所以我们必须索引元组的各个部分，使我们的计算不那么明显。

混合宽度和高度对于面积计算来说并不重要，但如果我们想在屏幕上绘制矩形，那就很重要了！ 我们必须记住，宽度是元组索引 0，高度是元组索引 1。如果其他人使用我们的代码，则更难弄清楚并记住这一点。 因为我们没有在代码中传达数据的含义，所以现在更容易引入错误。

### Refactoring with Structs: Adding More Meaning
We use structs to add meaning by labeling the data. We can transform the tuple we’re using into a struct with a name for the whole as well as names for the parts, as shown in Listing 5-10.

我们使用结构通过标记数据来添加含义。 我们可以将我们使用的元组转换为一个具有整体名称和部分名称的结构体，如清单 5-10 所示。

Filename: src/main.rs
``````
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!(
        "The area of the rectangle is {} square pixels.",
        area(&rect1)
    );
}

fn area(rectangle: &Rectangle) -> u32 {
    rectangle.width * rectangle.height
}
``````
Listing 5-10: Defining a Rectangle struct

Here we’ve defined a struct and named it Rectangle. Inside the curly brackets, we defined the fields as width and height, both of which have type u32. Then, in main, we created a particular instance of Rectangle that has a width of 30 and a height of 50.

Our area function is now defined with one parameter, which we’ve named rectangle, whose type is an immutable borrow of a struct Rectangle instance. As mentioned in Chapter 4, we want to borrow the struct rather than take ownership of it. This way, main retains its ownership and can continue using rect1, which is the reason we use the & in the function signature and where we call the function.

The area function accesses the width and height fields of the Rectangle instance (note that accessing fields of a borrowed struct instance does not move the field values, which is why you often see borrows of structs). Our function signature for area now says exactly what we mean: calculate the area of Rectangle, using its width and height fields. This conveys that the width and height are related to each other, and it gives descriptive names to the values rather than using the tuple index values of 0 and 1. This is a win for clarity.

这里我们定义了一个结构体并将其命名为 Rectangle。 在大括号内，我们将字段定义为宽度和高度，这两个字段的类型均为 u32。 然后，在 main 中，我们创建了一个特定的 Rectangle 实例，其宽度为 30，高度为 50。

我们的区域函数现在用一个参数定义，我们将其命名为矩形，其类型是 struct Rectangle 实例的不可变借用。 正如第 4 章中提到的，我们想要借用该结构而不是获得它的所有权。 这样，main 保留其所有权并可以继续使用 rect1，这就是我们在函数签名和调用该函数的位置中使用 & 的原因。

area 函数访问 Rectangle 实例的宽度和高度字段（请注意，访问借用的结构体实例的字段不会移动字段值，这就是您经常看到结构体借用的原因）。 我们的面积函数签名现在准确地表达了我们的意思：使用矩形的宽度和高度字段计算矩形的面积。 这表明宽度和高度是相互关联的，并且它为值提供了描述性名称，而不是使用 0 和 1 的元组索引值。为了清晰起见，这是一个胜利。

### Adding Useful Functionality with Derived Traits
It’d be useful to be able to print an instance of Rectangle while we’re debugging our program and see the values for all its fields. Listing 5-11 tries using the println! macro as we have used in previous chapters. This won’t work, however.

当我们调试程序并查看其所有字段的值时，能够打印 Rectangle 的实例会很有用。 清单 5-11 尝试使用 println! 我们在前面的章节中使用过的宏。 然而，这行不通。

Filename: src/main.rs
``````
This code does not compile!
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!("rect1 is {}", rect1);
}
``````
Listing 5-11: Attempting to print a Rectangle instance

When we compile this code, we get an error with this core message:
``````
error[E0277]: `Rectangle` doesn't implement `std::fmt::Display`
``````
The println! macro can do many kinds of formatting, and by default, the curly brackets tell println! to use formatting known as Display: output intended for direct end user consumption. The primitive types we’ve seen so far implement Display by default because there’s only one way you’d want to show a 1 or any other primitive type to a user. But with structs, the way println! should format the output is less clear because there are more display possibilities: Do you want commas or not? Do you want to print the curly brackets? Should all the fields be shown? Due to this ambiguity, Rust doesn’t try to guess what we want, and structs don’t have a provided implementation of Display to use with println! and the {} placeholder.

打印！ 宏可以进行多种格式化，默认情况下，大括号告诉 println! 使用称为“显示”的格式：旨在供最终用户直接使用的输出。 到目前为止，我们看到的基本类型默认实现 Display，因为只有一种方法可以向用户显示 1 或任何其他基本类型。 但是对于结构体， println! 的方式 应该格式化输出不太清楚，因为有更多的显示可能性：您是否想要逗号？ 您想打印大括号吗？ 是否应该显示所有字段？ 由于这种歧义，Rust 不会尝试猜测我们想要什么，并且结构没有提供与 println 一起使用的 Display 实现！ 和 {} 占位符。

If we continue reading the errors, we’ll find this helpful note:
``````
   = help: the trait `std::fmt::Display` is not implemented for `Rectangle`
   = note: in format strings you may be able to use `{:?}` (or {:#?} for pretty-print) instead
``````
Let’s try it! The println! macro call will now look like println!("rect1 is {:?}", rect1);. Putting the specifier :? inside the curly brackets tells println! we want to use an output format called Debug. The Debug trait enables us to print our struct in a way that is useful for developers so we can see its value while we’re debugging our code.

我们来试试吧！ 打印！ 宏调用现在看起来像 println!("rect1 is {:?}", rect1);。 放置说明符：? 大括号里面告诉 println! 我们想要使用一种称为“调试”的输出格式。 调试特征使我们能够以对开发人员有用的方式打印我们的结构，这样我们就可以在调试代码时看到它的值。

Compile the code with this change. Drat! We still get an error:
``````
error[E0277]: `Rectangle` doesn't implement `Debug`
``````
But again, the compiler gives us a helpful note:
``````
   = help: the trait `Debug` is not implemented for `Rectangle`
   = note: add `#[derive(Debug)]` to `Rectangle` or manually `impl Debug for Rectangle`
``````
Rust does include functionality to print out debugging information, but we have to explicitly opt in to make that functionality available for our struct. To do that, we add the outer attribute #[derive(Debug)] just before the struct definition, as shown in Listing 5-12.

Rust 确实包含打印调试信息的功能，但我们必须明确选择使该功能可用于我们的结构。 为此，我们在结构体定义之前添加外部属性 #[derive(Debug)]，如清单 5-12 所示。

Filename: src/main.rs
``````
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!("rect1 is {:?}", rect1);
}
``````
Listing 5-12: Adding the attribute to derive the Debug trait and printing the Rectangle instance using debug formatting

Now when we run the program, we won’t get any errors, and we’ll see the following output:
``````
$ cargo run
   Compiling rectangles v0.1.0 (file:///projects/rectangles)
    Finished dev [unoptimized + debuginfo] target(s) in 0.48s
     Running `target/debug/rectangles`
rect1 is Rectangle { width: 30, height: 50 }
``````
Nice! It’s not the prettiest output, but it shows the values of all the fields for this instance, which would definitely help during debugging. When we have larger structs, it’s useful to have output that’s a bit easier to read; in those cases, we can use {:#?} instead of {:?} in the println! string. In this example, using the {:#?} style will output the following:

好的！ 它不是最漂亮的输出，但它显示了该实例的所有字段的值，这在调试过程中肯定会有所帮助。 当我们有更大的结构时，让输出更容易阅读会很有用； 在这些情况下，我们可以在 println! 中使用 {:#?} 而不是 {:?} 细绳。 在此示例中，使用 {:#?} 样式将输出以下内容：

``````
$ cargo run
   Compiling rectangles v0.1.0 (file:///projects/rectangles)
    Finished dev [unoptimized + debuginfo] target(s) in 0.48s
     Running `target/debug/rectangles`
rect1 is Rectangle {
    width: 30,
    height: 50,
}
``````
Another way to print out a value using the Debug format is to use the dbg! macro, which takes ownership of an expression (as opposed to println!, which takes a reference), prints the file and line number of where that dbg! macro call occurs in your code along with the resultant value of that expression, and returns ownership of the value.

使用调试格式打印值的另一种方法是使用 dbg! 宏，它获取表达式的所有权（与 println! 相反，它获取引用），打印 dbg! 所在位置的文件和行号。 宏调用与该表达式的结果值一起发生在代码中，并返回该值的所有权。

``````
Note: Calling the dbg! macro prints to the standard error console stream (stderr), as opposed to println!, which prints to the standard output console stream (stdout). We’ll talk more about stderr and stdout in the “Writing Error Messages to Standard Error Instead of Standard Output” section in Chapter 12.
``````

Here’s an example where we’re interested in the value that gets assigned to the width field, as well as the value of the whole struct in rect1:
``````
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let scale = 2;
    let rect1 = Rectangle {
        width: dbg!(30 * scale),
        height: 50,
    };

    dbg!(&rect1);
}
``````
We can put dbg! around the expression 30 * scale and, because dbg! returns ownership of the expression’s value, the width field will get the same value as if we didn’t have the dbg! call there. We don’t want dbg! to take ownership of rect1, so we use a reference to rect1 in the next call. Here’s what the output of this example looks like:

我们可以把 dbg! 围绕表达式 30 * 缩放，因为 dbg! 返回表达式值的所有权，宽度字段将获得相同的值，就好像我们没有 dbg 一样！ 打电话到那里。 我们不需要 dbg！ 取得 rect1 的所有权，因此我们在下一次调用中使用对 rect1 的引用。 此示例的输出如下所示：
``````
$ cargo run
   Compiling rectangles v0.1.0 (file:///projects/rectangles)
    Finished dev [unoptimized + debuginfo] target(s) in 0.61s
     Running `target/debug/rectangles`
[src/main.rs:10] 30 * scale = 60
[src/main.rs:14] &rect1 = Rectangle {
    width: 60,
    height: 50,
}
``````
We can see the first bit of output came from src/main.rs line 10 where we’re debugging the expression 30 * scale, and its resultant value is 60 (the Debug formatting implemented for integers is to print only their value). The dbg! call on line 14 of src/main.rs outputs the value of &rect1, which is the Rectangle struct. This output uses the pretty Debug formatting of the Rectangle type. The dbg! macro can be really helpful when you’re trying to figure out what your code is doing!

In addition to the Debug trait, Rust has provided a number of traits for us to use with the derive attribute that can add useful behavior to our custom types. Those traits and their behaviors are listed in Appendix C. We’ll cover how to implement these traits with custom behavior as well as how to create your own traits in Chapter 10. There are also many attributes other than derive; for more information, see the “Attributes” section of the Rust Reference.

Our area function is very specific: it only computes the area of rectangles. It would be helpful to tie this behavior more closely to our Rectangle struct because it won’t work with any other type. Let’s look at how we can continue to refactor this code by turning the area function into an area method defined on our Rectangle type.

我们可以看到输出的第一位来自 src/main.rs 第 10 行，我们正在调试表达式 30 * scale，其结果值为 60（为整数实现的调试格式仅打印它们的值）。 数据库！ src/main.rs 第 14 行调用输出 &rect1 的值，它是 Rectangle 结构。 此输出使用矩形类型的漂亮调试格式。 数据库！ 当您试图弄清楚代码在做什么时，宏非常有用！

除了 Debug 特征之外，Rust 还提供了许多特征供我们与derive 属性一起使用，这些属性可以为我们的自定义类型添加有用的行为。 这些特征及其行为列在附录 C 中。我们将在第 10 章中介绍如何使用自定义行为实现这些特征以及如何创建自己的特征。除了导出之外，还有许多属性；例如： 有关更多信息，请参阅 Rust 参考的“属性”部分。

我们的面积函数非常具体：它只计算矩形的面积。 将此行为与我们的 Rectangle 结构更紧密地联系起来会很有帮助，因为它不适用于任何其他类型。 让我们看看如何通过将area函数转换为在我们的矩形类型上定义的area方法来继续重构这段代码。

## 5.3 Method Syntax

Methods are similar to functions: we declare them with the fn keyword and a name, they can have parameters and a return value, and they contain some code that’s run when the method is called from somewhere else. Unlike functions, methods are defined within the context of a struct (or an enum or a trait object, which we cover in Chapter 6 and Chapter 17, respectively), and their first parameter is always self, which represents the instance of the struct the method is being called on.

方法与函数类似：我们使用 fn 关键字和名称来声明它们，它们可以具有参数和返回值，并且它们包含一些从其他地方调用该方法时运行的代码。 与函数不同，方法是在结构体（或者枚举或特征对象，我们分别在第 6 章和第 17 章中介绍）的上下文中定义的，并且它们的第一个参数始终是 self，它表示结构体的实例 方法正在被调用。

### Defining Methods
Let’s change the area function that has a Rectangle instance as a parameter and instead make an area method defined on the Rectangle struct, as shown in Listing 5-13.

让我们更改以 Rectangle 实例作为参数的区域函数，并在 Rectangle 结构上定义一个区域方法，如清单 5-13 所示。

Filename: src/main.rs
``````
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()
    );
}
``````
Listing 5-13: Defining an area method on the Rectangle struct

To define the function within the context of Rectangle, we start an impl (implementation) block for Rectangle. Everything within this impl block will be associated with the Rectangle type. Then we move the area function within the impl curly brackets and change the first (and in this case, only) parameter to be self in the signature and everywhere within the body. In main, where we called the area function and passed rect1 as an argument, we can instead use method syntax to call the area method on our Rectangle instance. The method syntax goes after an instance: we add a dot followed by the method name, parentheses, and any arguments.

In the signature for area, we use &self instead of rectangle: &Rectangle. The &self is actually short for self: &Self. Within an impl block, the type Self is an alias for the type that the impl block is for. Methods must have a parameter named self of type Self for their first parameter, so Rust lets you abbreviate this with only the name self in the first parameter spot. Note that we still need to use the & in front of the self shorthand to indicate that this method borrows the Self instance, just as we did in rectangle: &Rectangle. Methods can take ownership of self, borrow self immutably, as we’ve done here, or borrow self mutably, just as they can any other parameter.

We chose &self here for the same reason we used &Rectangle in the function version: we don’t want to take ownership, and we just want to read the data in the struct, not write to it. If we wanted to change the instance that we’ve called the method on as part of what the method does, we’d use &mut self as the first parameter. Having a method that takes ownership of the instance by using just self as the first parameter is rare; this technique is usually used when the method transforms self into something else and you want to prevent the caller from using the original instance after the transformation.

The main reason for using methods instead of functions, in addition to providing method syntax and not having to repeat the type of self in every method’s signature, is for organization. We’ve put all the things we can do with an instance of a type in one impl block rather than making future users of our code search for capabilities of Rectangle in various places in the library we provide.

Note that we can choose to give a method the same name as one of the struct’s fields. For example, we can define a method on Rectangle that is also named width:

为了在 Rectangle 的上下文中定义函数，我们为 Rectangle 启动一个 impl（实现）块。 此 impl 块中的所有内容都将与 Rectangle 类型关联。 然后，我们将区域函数移动到 impl 花括号内，并将签名中和正文中的第一个（在本例中是唯一）参数更改为 self。 在 main 中，我们调用了区域函数并传递了 rect1 作为参数，我们可以使用方法语法来调用 Rectangle 实例上的区域方法。 方法语法位于实例之后：我们添加一个点，后跟方法名称、括号和任何参数。

在区域的签名中，我们使用 &self 而不是矩形：&Rectangle。 &self 实际上是 self 的缩写：&Self。 在 impl 块中，Self 类型是 impl 块所属类型的别名。 方法的第一个参数必须有一个名为 self 的 Self 类型参数，因此 Rust 允许您在第一个参数位置仅使用名称 self 来缩写它。 请注意，我们仍然需要在 self 简写前面使用 & 来表示该方法借用 Self 实例，就像我们在矩形中所做的那样：&Rectangle。 方法可以取得 self 的所有权，不变地借用 self，就像我们在这里所做的那样，或者可变地借用 self，就像它们可以使用任何其他参数一样。

我们在这里选择 &self 的原因与我们在函数版本中使用 &Rectangle 的原因相同：我们不想获得所有权，我们只想读取结构中的数据，而不是写入它。 如果我们想要更改调用该方法的实例（作为该方法的一部分），我们可以使用 &mut self 作为第一个参数。 通过仅使用 self 作为第一个参数来获取实例所有权的方法很少见； 当方法将 self 转换为其他内容并且您希望防止调用者在转换后使用原始实例时，通常会使用此技术。

使用方法而不是函数的主要原因，除了提供方法语法并且不必在每个方法的签名中重复 self 类型之外，还有为了组织。 我们已经将一种类型的实例可以做的所有事情都放在一个 impl 块中，而不是让我们代码的未来用户在我们提供的库的各个位置搜索 Rectangle 的功能。

请注意，我们可以选择为方法指定与结构体字段之一相同的名称。 例如，我们可以在 Rectangle 上定义一个方法，也称为 width：

Filename: src/main.rs
``````
impl Rectangle {
    fn width(&self) -> bool {
        self.width > 0
    }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    if rect1.width() {
        println!("The rectangle has a nonzero width; it is {}", rect1.width);
    }
}
``````
Here, we’re choosing to make the width method return true if the value in the instance’s width field is greater than 0 and false if the value is 0: we can use a field within a method of the same name for any purpose. In main, when we follow rect1.width with parentheses, Rust knows we mean the method width. When we don’t use parentheses, Rust knows we mean the field width.

Often, but not always, when we give a method the same name as a field we want it to only return the value in the field and do nothing else. Methods like this are called getters, and Rust does not implement them automatically for struct fields as some other languages do. Getters are useful because you can make the field private but the method public, and thus enable read-only access to that field as part of the type’s public API. We will discuss what public and private are and how to designate a field or method as public or private in Chapter 7.

在这里，如果实例的 width 字段中的值大于 0，我们选择让 width 方法返回 true；如果值为 0，则返回 false：我们可以在同名方法中使用字段用于任何目的。 在 main 中，当我们在 rect1.width 后面加上括号时，Rust 知道我们指的是方法宽度。 当我们不使用括号时，Rust 知道我们指的是字段宽度。

通常，但并非总是，当我们给一个方法提供与字段相同的名称时，我们希望它只返回字段中的值，而不执行其他操作。 像这样的方法称为 getter，Rust 不会像其他语言那样自动为结构体字段实现它们。 Getter 很有用，因为您可以将字段设为私有，但将方法设为公开，从而作为类型的公共 API 的一部分启用对该字段的只读访问。 我们将在第 7 章讨论什么是公共和私有以及如何将字段或方法指定为公共或私有。

``````
Where’s the -> Operator?
In C and C++, two different operators are used for calling methods: you use . if you’re calling a method on the object directly and -> if you’re calling the method on a pointer to the object and need to dereference the pointer first. In other words, if object is a pointer, object->something() is similar to (*object).something().

Rust doesn’t have an equivalent to the -> operator; instead, Rust has a feature called automatic referencing and dereferencing. Calling methods is one of the few places in Rust that has this behavior.

Here’s how it works: when you call a method with object.something(), Rust automatically adds in &, &mut, or * so object matches the signature of the method. In other words, the following are the same:

p1.distance(&p2);
(&p1).distance(&p2);
The first one looks much cleaner. This automatic referencing behavior works because methods have a clear receiver—the type of self. Given the receiver and name of a method, Rust can figure out definitively whether the method is reading (&self), mutating (&mut self), or consuming (self). The fact that Rust makes borrowing implicit for method receivers is a big part of making ownership ergonomic in practice.
``````
### Methods with More Parameters
Let’s practice using methods by implementing a second method on the Rectangle struct. This time we want an instance of Rectangle to take another instance of Rectangle and return true if the second Rectangle can fit completely within self (the first Rectangle); otherwise, it should return false. That is, once we’ve defined the can_hold method, we want to be able to write the program shown in Listing 5-14.

让我们通过在 Rectangle 结构上实现第二个方法来练习使用方法。 这次，我们希望 Rectangle 的实例获取 Rectangle 的另一个实例，如果第二个 Rectangle 可以完全适合 self（第一个 Rectangle），则返回 true； 否则，它应该返回 false。 也就是说，一旦定义了 can_hold 方法，我们就希望能够编写清单 5-14 中所示的程序。

Filename: src/main.rs
``````
fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };
    let rect2 = Rectangle {
        width: 10,
        height: 40,
    };
    let rect3 = Rectangle {
        width: 60,
        height: 45,
    };

    println!("Can rect1 hold rect2? {}", rect1.can_hold(&rect2));
    println!("Can rect1 hold rect3? {}", rect1.can_hold(&rect3));
}
``````
Listing 5-14: Using the as-yet-unwritten can_hold method

The expected output would look like the following because both dimensions of rect2 are smaller than the dimensions of rect1, but rect3 is wider than rect1:
``````
Can rect1 hold rect2? true
Can rect1 hold rect3? false
``````
We know we want to define a method, so it will be within the impl Rectangle block. The method name will be can_hold, and it will take an immutable borrow of another Rectangle as a parameter. We can tell what the type of the parameter will be by looking at the code that calls the method: rect1.can_hold(&rect2) passes in &rect2, which is an immutable borrow to rect2, an instance of Rectangle. This makes sense because we only need to read rect2 (rather than write, which would mean we’d need a mutable borrow), and we want main to retain ownership of rect2 so we can use it again after calling the can_hold method. The return value of can_hold will be a Boolean, and the implementation will check whether the width and height of self are greater than the width and height of the other Rectangle, respectively. Let’s add the new can_hold method to the impl block from Listing 5-13, shown in Listing 5-15.

我们知道我们想要定义一个方法，因此它将位于 impl Rectangle 块内。 方法名称为 can_hold，它将采用另一个 Rectangle 的不可变借用作为参数。 我们可以通过查看调用该方法的代码来判断参数的类型： rect1.can_hold(&rect2) 传入 &rect2，它是对 rect2（矩形的实例）的不可变借用。 这是有道理的，因为我们只需要读取 rect2（而不是写入，这意味着我们需要可变借位），并且我们希望 main 保留 rect2 的所有权，以便我们可以在调用 can_hold 方法后再次使用它。 can_hold的返回值将是一个布尔值，并且实现将分别检查自身的宽度和高度是否大于另一个矩形的宽度和高度。 让我们将新的 can_hold 方法添加到清单 5-13 中的 impl 块中，如清单 5-15 所示。

Filename: src/main.rs
``````
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }

    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
``````
Listing 5-15: Implementing the can_hold method on Rectangle that takes another Rectangle instance as a parameter

When we run this code with the main function in Listing 5-14, we’ll get our desired output. Methods can take multiple parameters that we add to the signature after the self parameter, and those parameters work just like parameters in functions.

当我们使用清单 5-14 中的 main 函数运行这段代码时，我们将得到我们想要的输出。 方法可以采用我们在 self 参数之后添加到签名中的多个参数，这些参数的工作方式就像函数中的参数一样。

### Associated Functions
All functions defined within an impl block are called associated functions because they’re associated with the type named after the impl. We can define associated functions that don’t have self as their first parameter (and thus are not methods) because they don’t need an instance of the type to work with. We’ve already used one function like this: the String::from function that’s defined on the String type.

Associated functions that aren’t methods are often used for constructors that will return a new instance of the struct. These are often called new, but new isn’t a special name and isn’t built into the language. For example, we could choose to provide an associated function named square that would have one dimension parameter and use that as both width and height, thus making it easier to create a square Rectangle rather than having to specify the same value twice:

impl 块中定义的所有函数都称为关联函数，因为它们与以 impl 命名的类型关联。 我们可以定义不将 self 作为第一个参数的关联函数（因此不是方法），因为它们不需要使用该类型的实例。 我们已经使用过这样一个函数：在 String 类型上定义的 String::from 函数。

非方法的关联函数通常用于返回结构的新实例的构造函数。 它们通常被称为 new，但 new 并不是一个特殊的名称，也没有内置到语言中。 例如，我们可以选择提供一个名为 square 的关联函数，该函数具有一个维度参数，并将其用作宽度和高度，从而更容易创建正方形 Rectangle，而不必两次指定相同的值：

Filename: src/main.rs
``````
impl Rectangle {
    fn square(size: u32) -> Self {
        Self {
            width: size,
            height: size,
        }
    }
}
``````
The Self keywords in the return type and in the body of the function are aliases for the type that appears after the impl keyword, which in this case is Rectangle.

To call this associated function, we use the :: syntax with the struct name; let sq = Rectangle::square(3); is an example. This function is namespaced by the struct: the :: syntax is used for both associated functions and namespaces created by modules. We’ll discuss modules in Chapter 7.

返回类型和函数体中的 Self 关键字是 impl 关键字后面出现的类型的别名，在本例中为 Rectangle。

为了调用这个关联函数，我们使用 :: 语法和结构名称； 让 sq = 矩形::正方形(3); 就是一个例子。 该函数由结构体命名：:: 语法用于关联函数和模块创建的命名空间。 我们将在第 7 章中讨论模块。

### Multiple impl Blocks
Each struct is allowed to have multiple impl blocks. For example, Listing 5-15 is equivalent to the code shown in Listing 5-16, which has each method in its own impl block.
``````
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
``````
Listing 5-16: Rewriting Listing 5-15 using multiple impl blocks

There’s no reason to separate these methods into multiple impl blocks here, but this is valid syntax. We’ll see a case in which multiple impl blocks are useful in Chapter 10, where we discuss generic types and traits.

这里没有理由将这些方法分成多个 impl 块，但这是有效的语法。 我们将在第 10 章中看到多个 impl 块很有用的情况，其中我们将讨论泛型类型和特征。

## 5.4 总结
Structs let you create custom types that are meaningful for your domain. By using structs, you can keep associated pieces of data connected to each other and name each piece to make your code clear. In impl blocks, you can define functions that are associated with your type, and methods are a kind of associated function that let you specify the behavior that instances of your structs have.

But structs aren’t the only way you can create custom types: let’s turn to Rust’s enum feature to add another tool to your toolbox.

结构允许您创建对您的域有意义的自定义类型。 通过使用结构，您可以保持关联的数据片段相互连接，并为每个片段命名以使代码清晰。 在 impl 块中，您可以定义与您的类型关联的函数，而方法是一种关联函数，可让您指定结构体实例所具有的行为。

但结构并不是创建自定义类型的唯一方法：让我们转向 Rust 的枚举功能，将另一个工具添加到您的工具箱中。