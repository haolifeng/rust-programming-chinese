# 17 面向对象编程
Object-oriented programming (OOP) is a way of modeling programs. Objects as a programmatic concept were introduced in the programming language Simula in the 1960s. Those objects influenced Alan Kay’s programming architecture in which objects pass messages to each other. To describe this architecture, he coined the term object-oriented programming in 1967. Many competing definitions describe what OOP is, and by some of these definitions Rust is object-oriented, but by others it is not. In this chapter, we’ll explore certain characteristics that are commonly considered object-oriented and how those characteristics translate to idiomatic Rust. We’ll then show you how to implement an object-oriented design pattern in Rust and discuss the trade-offs of doing so versus implementing a solution using some of Rust’s strengths instead.

面向对象编程（OOP）是一种程序建模方法。 对象作为编程概念于 20 世纪 60 年代在编程语言 Simula 中引入。 这些对象影响了 Alan Kay 的编程架构，其中对象相互传递消息。 为了描述这种架构，他在 1967 年创造了面向对象编程这个术语。许多相互竞争的定义描述了 OOP 是什么，根据其中一些定义，Rust 是面向对象的，但根据其他定义，它不是。 在本章中，我们将探讨通常被认为是面向对象的某些特征，以及这些特征如何转化为惯用的 Rust。 然后，我们将向您展示如何在 Rust 中实现面向对象的设计模式，并讨论这样做与使用 Rust 的一些优势实现解决方案之间的权衡。

## 17.1 面向对象语言的特征
There is no consensus in the programming community about what features a language must have to be considered object-oriented. Rust is influenced by many programming paradigms, including OOP; for example, we explored the features that came from functional programming in Chapter 13. Arguably, OOP languages share certain common characteristics, namely objects, encapsulation, and inheritance. Let’s look at what each of those characteristics means and whether Rust supports it.

对于一种语言必须具备哪些特性才能被视为面向对象，编程社区尚未达成共识。 Rust 受到许多编程范式的影响，包括 OOP； 例如，我们在第 13 章中探讨了来自函数式编程的特性。可以说，OOP 语言具有某些共同的特性，即对象、封装和继承。 让我们看看每个特性的含义以及 Rust 是否支持它们。

### 包含数据和行为的对象(Objects Contain Data and Behavior)
The book Design Patterns: Elements of Reusable Object-Oriented Software by Erich Gamma, Richard Helm, Ralph Johnson, and John Vlissides (Addison-Wesley Professional, 1994), colloquially referred to as The Gang of Four book, is a catalog of object-oriented design patterns. It defines OOP this way:

Object-oriented programs are made up of objects. An object packages both data and the procedures that operate on that data. The procedures are typically called methods or operations.

Using this definition, Rust is object-oriented: structs and enums have data, and impl blocks provide methods on structs and enums. Even though structs and enums with methods aren’t called objects, they provide the same functionality, according to the Gang of Four’s definition of objects.

Erich Gamma、Richard Helm、Ralph Johnson 和 John Vlissides 所著的《设计模式：可重用面向对象软件的元素》一书（Addison-Wesley Professional，1994 年）通俗地称为“四人帮”一书，是一本面向对象的目录 面向设计模式。 它这样定义 OOP：

面向对象的程序是由对象组成的。 对象封装了数据和操作该数据的过程。 这些过程通常称为方法或操作。

使用这个定义，Rust 是面向对象的：结构和枚举具有数据，并且 impl 块提供结构和枚举上的方法。 尽管具有方法的结构体和枚举不称为对象，但根据四人帮对对象的定义，它们提供相同的功能。

### 隐藏细节的封装Encapsulation that Hides Implementation Details
Another aspect commonly associated with OOP is the idea of encapsulation, which means that the implementation details of an object aren’t accessible to code using that object. Therefore, the only way to interact with an object is through its public API; code using the object shouldn’t be able to reach into the object’s internals and change data or behavior directly. This enables the programmer to change and refactor an object’s internals without needing to change the code that uses the object.

We discussed how to control encapsulation in Chapter 7: we can use the pub keyword to decide which modules, types, functions, and methods in our code should be public, and by default everything else is private. For example, we can define a struct AveragedCollection that has a field containing a vector of i32 values. The struct can also have a field that contains the average of the values in the vector, meaning the average doesn’t have to be computed on demand whenever anyone needs it. In other words, AveragedCollection will cache the calculated average for us. Listing 17-1 has the definition of the AveragedCollection struct:

通常与 OOP 相关的另一个方面是封装的思想，这意味着使用该对象的代码无法访问该对象的实现细节。 因此，与对象交互的唯一方法是通过其公共 API； 使用对象的代码不应该能够进入对象的内部并直接更改数据或行为。 这使得程序员能够更改和重构对象的内部结构，而无需更改使用该对象的代码。

我们在第 7 章中讨论了如何控制封装：我们可以使用 pub 关键字来决定代码中的哪些模块、类型、函数和方法应该是公共的，默认情况下其他所有内容都是私有的。 例如，我们可以定义一个 struct AveragedCollection，它有一个包含 i32 值向量的字段。 该结构还可以有一个包含向量中值的平均值的字段，这意味着在任何人需要时都不必按需计算平均值。 换句话说，AvergedCollection 会为我们缓存计算出的平均值。 清单 17-1 包含 AveragedCollection 结构的定义：

Filename: src/lib.rs
``````
pub struct AveragedCollection {
    list: Vec<i32>,
    average: f64,
}
``````
Listing 17-1: An AveragedCollection struct that maintains a list of integers and the average of the items in the collection

The struct is marked pub so that other code can use it, but the fields within the struct remain private. This is important in this case because we want to ensure that whenever a value is added or removed from the list, the average is also updated. We do this by implementing add, remove, and average methods on the struct, as shown in Listing 17-2:

该结构被标记为 pub，以便其他代码可以使用它，但该结构中的字段仍然是私有的。 这在本例中很重要，因为我们希望确保每当在列表中添加或删除值时，平均值也会更新。 我们通过在结构体上实现 add、remove 和average 方法来实现这一点，如清单 17-2 所示：

Filename: src/lib.rs
``````
impl AveragedCollection {
    pub fn add(&mut self, value: i32) {
        self.list.push(value);
        self.update_average();
    }

    pub fn remove(&mut self) -> Option<i32> {
        let result = self.list.pop();
        match result {
            Some(value) => {
                self.update_average();
                Some(value)
            }
            None => None,
        }
    }

    pub fn average(&self) -> f64 {
        self.average
    }

    fn update_average(&mut self) {
        let total: i32 = self.list.iter().sum();
        self.average = total as f64 / self.list.len() as f64;
    }
}
``````
Listing 17-2: Implementations of the public methods add, remove, and average on AveragedCollection

The public methods add, remove, and average are the only ways to access or modify data in an instance of AveragedCollection. When an item is added to list using the add method or removed using the remove method, the implementations of each call the private update_average method that handles updating the average field as well.

We leave the list and average fields private so there is no way for external code to add or remove items to or from the list field directly; otherwise, the average field might become out of sync when the list changes. The average method returns the value in the average field, allowing external code to read the average but not modify it.

Because we’ve encapsulated the implementation details of the struct AveragedCollection, we can easily change aspects, such as the data structure, in the future. For instance, we could use a HashSet<i32> instead of a Vec<i32> for the list field. As long as the signatures of the add, remove, and average public methods stay the same, code using AveragedCollection wouldn’t need to change. If we made list public instead, this wouldn’t necessarily be the case: HashSet<i32> and Vec<i32> have different methods for adding and removing items, so the external code would likely have to change if it were modifying list directly.

If encapsulation is a required aspect for a language to be considered object-oriented, then Rust meets that requirement. The option to use pub or not for different parts of code enables encapsulation of implementation details.

公共方法 add、remove 和average 是访问或修改 AveragedCollection 实例中数据的唯一方法。 当使用 add 方法将项目添加到列表或使用 remove 方法删除项目时，每个项目的实现都会调用私有 update_average 方法，该方法也处理更新平均字段。

我们将列表和平均字段保留为私有，因此外部代码无法直接向列表字段添加或删除项目； 否则，当列表更改时，平均字段可能会变得不同步。 Average 方法返回平均值字段中的值，允许外部代码读取平均值但不能修改它。

因为我们已经封装了 struct AveragedCollection 的实现细节，所以将来我们可以轻松地更改数据结构等方面。 例如，我们可以使用 HashSet<i32> 而不是 Vec<i32> 作为列表字段。 只要添加、删除和平均公共方法的签名保持不变，使用 AveragedCollection 的代码就不需要更改。 如果我们将列表设为公开，则情况不一定如此：HashSet<i32> 和 Vec<i32> 有不同的添加和删除项目的方法，因此如果直接修改列表，外部代码可能必须更改。

如果封装是一种语言被认为是面向对象的必需方面，那么 Rust 就满足了这个要求。 对代码的不同部分使用或不使用 pub 的选项可以实现实现细节的封装。

### 继承(Inheritance as a Type System and as Code Sharing)
Inheritance is a mechanism whereby an object can inherit elements from another object’s definition, thus gaining the parent object’s data and behavior without you having to define them again.

If a language must have inheritance to be an object-oriented language, then Rust is not one. There is no way to define a struct that inherits the parent struct’s fields and method implementations without using a macro.

However, if you’re used to having inheritance in your programming toolbox, you can use other solutions in Rust, depending on your reason for reaching for inheritance in the first place.

You would choose inheritance for two main reasons. One is for reuse of code: you can implement particular behavior for one type, and inheritance enables you to reuse that implementation for a different type. You can do this in a limited way in Rust code using default trait method implementations, which you saw in Listing 10-14 when we added a default implementation of the summarize method on the Summary trait. Any type implementing the Summary trait would have the summarize method available on it without any further code. This is similar to a parent class having an implementation of a method and an inheriting child class also having the implementation of the method. We can also override the default implementation of the summarize method when we implement the Summary trait, which is similar to a child class overriding the implementation of a method inherited from a parent class.

The other reason to use inheritance relates to the type system: to enable a child type to be used in the same places as the parent type. This is also called polymorphism, which means that you can substitute multiple objects for each other at runtime if they share certain characteristics.

继承是一种机制，一个对象可以从另一个对象的定义继承元素，从而获得父对象的数据和行为，而无需再次定义它们。

如果一种语言必须具有继承才能成为面向对象的语言，那么 Rust 就不是。 如果不使用宏，就无法定义继承父结构体的字段和方法实现的结构体。

但是，如果您习惯于在编程工具箱中使用继承，则可以在 Rust 中使用其他解决方案，具体取决于您首先实现继承的原因。

您选择继承有两个主要原因。 一是为了代码的重用：您可以为一种类型实现特定的行为，而继承使您能够为不同的类型重用该实现。 您可以使用默认特征方法实现在 Rust 代码中以有限的方式执行此操作，当我们在 Summary 特征上添加 summarize 方法的默认实现时，您可以在清单 10-14 中看到。 任何实现 Summary 特征的类型都可以使用 Summarize 方法，而无需任何进一步的代码。 这类似于具有方法的实现的父类和也具有该方法的实现的继承子类。 当我们实现Summary特征时，我们还可以覆盖summary方法的默认实现，这类似于子类覆盖从父类继承的方法的实现。

使用继承的另一个原因与类型系统有关：使子类型能够在与父类型相同的位置使用。 这也称为多态性，这意味着如果多个对象共享某些特征，则可以在运行时相互替换多个对象。

``````
Polymorphism
To many people, polymorphism is synonymous with inheritance. But it’s actually a more general concept that refers to code that can work with data of multiple types. For inheritance, those types are generally subclasses.

Rust instead uses generics to abstract over different possible types and trait bounds to impose constraints on what those types must provide. This is sometimes called bounded parametric polymorphism.
``````
Inheritance has recently fallen out of favor as a programming design solution in many programming languages because it’s often at risk of sharing more code than necessary. Subclasses shouldn’t always share all characteristics of their parent class but will do so with inheritance. This can make a program’s design less flexible. It also introduces the possibility of calling methods on subclasses that don’t make sense or that cause errors because the methods don’t apply to the subclass. In addition, some languages will only allow single inheritance (meaning a subclass can only inherit from one class), further restricting the flexibility of a program’s design.

For these reasons, Rust takes the different approach of using trait objects instead of inheritance. Let’s look at how trait objects enable polymorphism in Rust.

最近，在许多编程语言中，继承作为一种编程设计解决方案已经不再受欢迎，因为它经常面临共享过多代码的风险。 子类不应该总是共享其父类的所有特征，但可以通过继承来实现。 这会降低程序设计的灵活性。 它还引入了在子类上调用没有意义或导致错误的方法的可能性，因为这些方法不适用于子类。 此外，有些语言只允许单继承（即子类只能继承一个类），这进一步限制了程序设计的灵活性。

由于这些原因，Rust 采用了不同的方法，即使用特征对象而不是继承。 让我们看看 Trait 对象如何在 Rust 中实现多态性。

## 17.2 Using Trait Objects That Allow for Values of Different Types

In Chapter 8, we mentioned that one limitation of vectors is that they can store elements of only one type. We created a workaround in Listing 8-9 where we defined a SpreadsheetCell enum that had variants to hold integers, floats, and text. This meant we could store different types of data in each cell and still have a vector that represented a row of cells. This is a perfectly good solution when our interchangeable items are a fixed set of types that we know when our code is compiled.

However, sometimes we want our library user to be able to extend the set of types that are valid in a particular situation. To show how we might achieve this, we’ll create an example graphical user interface (GUI) tool that iterates through a list of items, calling a draw method on each one to draw it to the screen—a common technique for GUI tools. We’ll create a library crate called gui that contains the structure of a GUI library. This crate might include some types for people to use, such as Button or TextField. In addition, gui users will want to create their own types that can be drawn: for instance, one programmer might add an Image and another might add a SelectBox.

We won’t implement a fully fledged GUI library for this example but will show how the pieces would fit together. At the time of writing the library, we can’t know and define all the types other programmers might want to create. But we do know that gui needs to keep track of many values of different types, and it needs to call a draw method on each of these differently typed values. It doesn’t need to know exactly what will happen when we call the draw method, just that the value will have that method available for us to call.

To do this in a language with inheritance, we might define a class named Component that has a method named draw on it. The other classes, such as Button, Image, and SelectBox, would inherit from Component and thus inherit the draw method. They could each override the draw method to define their custom behavior, but the framework could treat all of the types as if they were Component instances and call draw on them. But because Rust doesn’t have inheritance, we need another way to structure the gui library to allow users to extend it with new types.

在第 8 章中，我们提到向量的一个限制是它们只能存储一种类型的元素。 我们在清单 8-9 中创建了一个解决方法，其中定义了一个 SpreadsheetCell 枚举，它具有保存整数、浮点数和文本的变体。 这意味着我们可以在每个单元格中存储不同类型的数据，并且仍然有一个代表一行单元格的向量。 当我们的可互换项是我们在编译代码时知道的一组固定类型时，这是一个非常好的解决方案。

然而，有时我们希望我们的库用户能够扩展在特定情况下有效的类型集。 为了展示我们如何实现这一目标，我们将创建一个示例图形用户界面 (GUI) 工具，该工具循环访问项目列表，调用每个项目的绘图方法将其绘制到屏幕上 - 这是 GUI 工具的常用技术。 我们将创建一个名为 gui 的库包，其中包含 GUI 库的结构。 这个箱子可能包含一些供人们使用的类型，例如 Button 或 TextField。 此外，GUI 用户将希望创建自己的可绘制类型：例如，一个程序员可能会添加一个 Image，另一个程序员可能会添加一个 SelectBox。

我们不会在这个例子中实现一个成熟的 GUI 库，但会展示这些部分如何组合在一起。 在编写库时，我们无法知道和定义其他程序员可能想要创建的所有类型。 但我们确实知道 gui 需要跟踪许多不同类型的值，并且它需要对每个不同类型的值调用一个绘制方法。 它不需要确切地知道当我们调用draw方法时会发生什么，只需要知道该值将具有可供我们调用的方法即可。

要在具有继承的语言中执行此操作，我们可以定义一个名为 Component 的类，该类具有名为 draw 的方法。 其他类，例如 Button、Image 和 SelectBox，将继承自 Component，从而继承 draw 方法。 它们每个都可以重写draw方法来定义它们的自定义行为，但是框架可以将所有类型视为组件实例并在它们上调用draw。 但由于 Rust 没有继承，我们需要另一种方式来构建 gui 库，以允许用户使用新类型对其进行扩展。

### Defining a Trait for Common Behavior
To implement the behavior we want gui to have, we’ll define a trait named Draw that will have one method named draw. Then we can define a vector that takes a trait object. A trait object points to both an instance of a type implementing our specified trait and a table used to look up trait methods on that type at runtime. We create a trait object by specifying some sort of pointer, such as a & reference or a Box<T> smart pointer, then the dyn keyword, and then specifying the relevant trait. (We’ll talk about the reason trait objects must use a pointer in Chapter 19 in the section “Dynamically Sized Types and the Sized Trait.”) We can use trait objects in place of a generic or concrete type. Wherever we use a trait object, Rust’s type system will ensure at compile time that any value used in that context will implement the trait object’s trait. Consequently, we don’t need to know all the possible types at compile time.

We’ve mentioned that, in Rust, we refrain from calling structs and enums “objects” to distinguish them from other languages’ objects. In a struct or enum, the data in the struct fields and the behavior in impl blocks are separated, whereas in other languages, the data and behavior combined into one concept is often labeled an object. However, trait objects are more like objects in other languages in the sense that they combine data and behavior. But trait objects differ from traditional objects in that we can’t add data to a trait object. Trait objects aren’t as generally useful as objects in other languages: their specific purpose is to allow abstraction across common behavior.

为了实现我们希望 gui 具有的行为，我们将定义一个名为 Draw 的特征，它将有一个名为 draw 的方法。 然后我们可以定义一个带有特征对象的向量。 特征对象既指向实现我们指定特征的类型的实例，又指向用于在运行时查找该类型的特征方法的表。 我们通过指定某种指针（例如 & 引用或 Box<T> 智能指针）、dyn 关键字，然后指定相关特征来创建特征对象。 （我们将在第 19 章“动态大小类型和大小特征”一节中讨论特征对象必须使用指针的原因。）我们可以使用特征对象来代替泛型或具体类型。 无论我们在哪里使用特征对象，Rust 的类型系统都会在编译时确保在该上下文中使用的任何值都将实现特征对象的特征。 因此，我们不需要在编译时知道所有可能的类型。

我们已经提到过，在 Rust 中，我们避免将结构和枚举称为“对象”，以将它们与其他语言的对象区分开来。 在结构体或枚举中，结构体字段中的数据和 impl 块中的行为是分开的，而在其他语言中，组合成一个概念的数据和行为通常被标记为一个对象。 然而，特征对象更像其他语言中的对象，因为它们结合了数据和行为。 但特征对象与传统对象的不同之处在于我们不能向特征对象添加数据。 Trait 对象并不像其他语言中的对象那么有用：它们的特定目的是允许对常见行为进行抽象。

Listing 17-3 shows how to define a trait named Draw with one method named draw:

Filename: src/lib.rs
``````
pub trait Draw {
    fn draw(&self);
}
``````
Listing 17-3: Definition of the Draw trait

This syntax should look familiar from our discussions on how to define traits in Chapter 10. Next comes some new syntax: Listing 17-4 defines a struct named Screen that holds a vector named components. This vector is of type Box<dyn Draw>, which is a trait object; it’s a stand-in for any type inside a Box that implements the Draw trait.

从我们在第 10 章中如何定义 Trait 的讨论中，这个语法应该看起来很熟悉。接下来是一些新语法：清单 17-4 定义了一个名为 Screen 的结构体，它包含一个名为 Components 的向量。 该向量的类型为 Box<dyn Draw>，它是一个特征对象； 它是 Box 内任何实现 Draw 特征的类型的替代品。

Filename: src/lib.rs
``````
pub struct Screen {
    pub components: Vec<Box<dyn Draw>>,
}
``````
Listing 17-4: Definition of the Screen struct with a components field holding a vector of trait objects that implement the Draw trait

On the Screen struct, we’ll define a method named run that will call the draw method on each of its components, as shown in Listing 17-5:

Filename: src/lib.rs
``````
impl Screen {
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}
``````
Listing 17-5: A run method on Screen that calls the draw method on each component

This works differently from defining a struct that uses a generic type parameter with trait bounds. A generic type parameter can only be substituted with one concrete type at a time, whereas trait objects allow for multiple concrete types to fill in for the trait object at runtime. For example, we could have defined the Screen struct using a generic type and a trait bound as in Listing 17-6:

这与定义使用具有特征边界的泛型类型参数的结构不同。 泛型类型参数一次只能替换为一种具体类型，而特征对象允许在运行时填充多个具体类型。 例如，我们可以使用泛型类型和特征绑定来定义 Screen 结构，如清单 17-6 所示：

Filename: src/lib.rs
``````
pub struct Screen<T: Draw> {
    pub components: Vec<T>,
}

impl<T> Screen<T>
where
    T: Draw,
{
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}
``````
Listing 17-6: An alternate implementation of the Screen struct and its run method using generics and trait bounds

This restricts us to a Screen instance that has a list of components all of type Button or all of type TextField. If you’ll only ever have homogeneous collections, using generics and trait bounds is preferable because the definitions will be monomorphized at compile time to use the concrete types.

On the other hand, with the method using trait objects, one Screen instance can hold a Vec<T> that contains a Box\<Button\> as well as a Box\<TextField\>. Let’s look at how this works, and then we’ll talk about the runtime performance implications.

这将我们限制为一个 Screen 实例，该实例具有所有类型为 Button 或所有类型为 TextField 的组件列表。 如果您只拥有同构集合，那么最好使用泛型和特征边界，因为定义将在编译时进行单态化以使用具体类型。

另一方面，通过使用特征对象的方法，一个 Screen 实例可以保存一个包含 Box\<Button\> 以及 Box\<TextField\> 的 Vec<T>。 让我们看看它是如何工作的，然后我们将讨论运行时性能的影响。

### Implementing the Trait
Now we’ll add some types that implement the Draw trait. We’ll provide the Button type. Again, actually implementing a GUI library is beyond the scope of this book, so the draw method won’t have any useful implementation in its body. To imagine what the implementation might look like, a Button struct might have fields for width, height, and label, as shown in Listing 17-7:

现在我们将添加一些实现 Draw 特征的类型。 我们将提供 Button 类型。 再次强调，实际实现 GUI 库超出了本书的范围，因此 draw 方法在其主体中不会有任何有用的实现。 想象一下实现的样子，Button 结构可能具有宽度、高度和标签字段，如清单 17-7 所示：

Filename: src/lib.rs
``````
pub struct Button {
    pub width: u32,
    pub height: u32,
    pub label: String,
}

impl Draw for Button {
    fn draw(&self) {
        // code to actually draw a button
    }
}
``````
Listing 17-7: A Button struct that implements the Draw trait

The width, height, and label fields on Button will differ from the fields on other components; for example, a TextField type might have those same fields plus a placeholder field. Each of the types we want to draw on the screen will implement the Draw trait but will use different code in the draw method to define how to draw that particular type, as Button has here (without the actual GUI code, as mentioned). The Button type, for instance, might have an additional impl block containing methods related to what happens when a user clicks the button. These kinds of methods won’t apply to types like TextField.

If someone using our library decides to implement a SelectBox struct that has width, height, and options fields, they implement the Draw trait on the SelectBox type as well, as shown in Listing 17-8:

Button 上的宽度、高度和标签字段将与其他组件上的字段不同； 例如，TextField 类型可能具有相同的字段以及占位符字段。 我们想要在屏幕上绘制的每种类型都将实现 Draw 特征，但将在绘制方法中使用不同的代码来定义如何绘制该特定类型，如 Button 此处所示（如上所述，没有实际的 GUI 代码）。 例如，Button 类型可能有一个附加的 impl 块，其中包含与用户单击按钮时发生的情况相关的方法。 这些方法不适用于 TextField 等类型。

如果使用我们库的人决定实现一个具有宽度、高度和选项字段的 SelectBox 结构，他们也会在 SelectBox 类型上实现 Draw 特征，如清单 17-8 所示：

Filename: src/main.rs
``````
use gui::Draw;

struct SelectBox {
    width: u32,
    height: u32,
    options: Vec<String>,
}

impl Draw for SelectBox {
    fn draw(&self) {
        // code to actually draw a select box
    }
}
``````
Listing 17-8: Another crate using gui and implementing the Draw trait on a SelectBox struct

Our library’s user can now write their main function to create a Screen instance. To the Screen instance, they can add a SelectBox and a Button by putting each in a Box<T> to become a trait object. They can then call the run method on the Screen instance, which will call draw on each of the components. Listing 17-9 shows this implementation:

我们库的用户现在可以编写他们的主函数来创建 Screen 实例。 对于 Screen 实例，他们可以添加一个 SelectBox 和一个 Button，方法是将它们放入 Box<T> 中以成为特征对象。 然后，他们可以调用 Screen 实例上的 run 方法，该方法将调用每个组件上的 draw 。 清单 17-9 显示了这个实现：

Filename: src/main.rs
``````
use gui::{Button, Screen};

fn main() {
    let screen = Screen {
        components: vec![
            Box::new(SelectBox {
                width: 75,
                height: 10,
                options: vec![
                    String::from("Yes"),
                    String::from("Maybe"),
                    String::from("No"),
                ],
            }),
            Box::new(Button {
                width: 50,
                height: 10,
                label: String::from("OK"),
            }),
        ],
    };

    screen.run();
}
``````
Listing 17-9: Using trait objects to store values of different types that implement the same trait

When we wrote the library, we didn’t know that someone might add the SelectBox type, but our Screen implementation was able to operate on the new type and draw it because SelectBox implements the Draw trait, which means it implements the draw method.

This concept—of being concerned only with the messages a value responds to rather than the value’s concrete type—is similar to the concept of duck typing in dynamically typed languages: if it walks like a duck and quacks like a duck, then it must be a duck! In the implementation of run on Screen in Listing 17-5, run doesn’t need to know what the concrete type of each component is. It doesn’t check whether a component is an instance of a Button or a SelectBox, it just calls the draw method on the component. By specifying Box<dyn Draw> as the type of the values in the components vector, we’ve defined Screen to need values that we can call the draw method on.

The advantage of using trait objects and Rust’s type system to write code similar to code using duck typing is that we never have to check whether a value implements a particular method at runtime or worry about getting errors if a value doesn’t implement a method but we call it anyway. Rust won’t compile our code if the values don’t implement the traits that the trait objects need.

当我们编写库时，我们不知道有人可能会添加 SelectBox 类型，但我们的 Screen 实现能够对新类型进行操作并绘制它，因为 SelectBox 实现了 Draw 特征，这意味着它实现了 draw 方法。

这个概念（只关心值响应的消息而不是值的具体类型）类似于动态类型语言中的鸭子类型概念：如果它像鸭子一样行走并且像鸭子一样嘎嘎叫，那么它一定是 鸭子！ 在清单17-5中的run on Screen实现中，run不需要知道每个组件的具体类型是什么。 它不会检查组件是否是 Button 或 SelectBox 的实例，它只是调用组件上的 draw 方法。 通过指定 Box<dyn Draw> 作为组件向量中的值的类型，我们定义了 Screen 来需要我们可以调用绘制方法的值。

使用 Trait 对象和 Rust 的类型系统编写与使用鸭子类型类似的代码的优点是，我们不必在运行时检查一个值是否实现了特定方法，也不必担心如果一个值没有实现某个方法但会出现错误。 无论如何我们都这么称呼它。 如果值没有实现特征对象所需的特征，Rust 将不会编译我们的代码。

For example, Listing 17-10 shows what happens if we try to create a Screen with a String as a component:

Filename: src/main.rs
``````
This code does not compile!
use gui::Screen;

fn main() {
    let screen = Screen {
        components: vec![Box::new(String::from("Hi"))],
    };

    screen.run();
}
``````
Listing 17-10: Attempting to use a type that doesn’t implement the trait object’s trait

We’ll get this error because String doesn’t implement the Draw trait:
``````
$ cargo run
   Compiling gui v0.1.0 (file:///projects/gui)
error[E0277]: the trait bound `String: Draw` is not satisfied
 --> src/main.rs:5:26
  |
5 |         components: vec![Box::new(String::from("Hi"))],
  |                          ^^^^^^^^^^^^^^^^^^^^^^^^^^^^ the trait `Draw` is not implemented for `String`
  |
  = help: the trait `Draw` is implemented for `Button`
  = note: required for the cast from `String` to the object type `dyn Draw`

For more information about this error, try `rustc --explain E0277`.
error: could not compile `gui` due to previous error
``````
This error lets us know that either we’re passing something to Screen we didn’t mean to pass and so should pass a different type or we should implement Draw on String so that Screen is able to call draw on it.

### Trait Objects Perform Dynamic Dispatch
Recall in the “Performance of Code Using Generics” section in Chapter 10 our discussion on the monomorphization process performed by the compiler when we use trait bounds on generics: the compiler generates nongeneric implementations of functions and methods for each concrete type that we use in place of a generic type parameter. The code that results from monomorphization is doing static dispatch, which is when the compiler knows what method you’re calling at compile time. This is opposed to dynamic dispatch, which is when the compiler can’t tell at compile time which method you’re calling. In dynamic dispatch cases, the compiler emits code that at runtime will figure out which method to call.

When we use trait objects, Rust must use dynamic dispatch. The compiler doesn’t know all the types that might be used with the code that’s using trait objects, so it doesn’t know which method implemented on which type to call. Instead, at runtime, Rust uses the pointers inside the trait object to know which method to call. This lookup incurs a runtime cost that doesn’t occur with static dispatch. Dynamic dispatch also prevents the compiler from choosing to inline a method’s code, which in turn prevents some optimizations. However, we did get extra flexibility in the code that we wrote in Listing 17-5 and were able to support in Listing 17-9, so it’s a trade-off to consider.

回想一下第 10 章的“使用泛型的代码性能”部分，我们讨论了当我们在泛型上使用特征边界时编译器执行的单态化过程：编译器为我们使用的每个具体类型生成函数和方法的非泛型实现 泛型类型参数。 单态化产生的代码正在执行静态分派，此时编译器知道您在编译时调用什么方法。 这与动态分派相反，动态分派是指编译器无法在编译时判断您正在调用哪个方法。 在动态调度情况下，编译器会发出代码，在运行时将确定要调用哪个方法。

当我们使用特征对象时，Rust 必须使用动态调度。 编译器不知道可能与使用特征对象的代码一起使用的所有类型，因此它不知道要调用哪个类型上实现的哪个方法。 相反，在运行时，Rust 使用特征对象内的指针来知道要调用哪个方法。 此查找会产生静态调度不会发生的运行时成本。 动态分派还阻止编译器选择内联方法的代码，这反过来又阻止了一些优化。 然而，我们在清单 17-5 中编写的代码确实获得了额外的灵活性，并且能够在清单 17-9 中提供支持，因此这是一个需要考虑的权衡。

## 17.3 实现一个面向对象的设计模式-状态模式
The state pattern is an object-oriented design pattern.The crux of the pattern is that we define a set of states a value can have internally. The states are represented by a set of state objects, and the value’s behavior changes based on its state. We’re going to work through an example of a blog post struct that has a field to hold its state, which will be a state object from the set "draft", "review", or "published".

The state objects share functionality: in Rust, of course, we use structs and traits rather than objects and inheritance. Each state object is responsible for its own behavior and for governing when it should change into another state. The value that holds a state object knows nothing about the different behavior of the states or when to transition between states.

The advantage of using the state pattern is that, when the business requirements of the program change, we won’t need to change the code of the value holding the state or the code that uses the value. We’ll only need to update the code inside one of the state objects to change its rules or perhaps add more state objects.

First, we’re going to implement the state pattern in a more traditional object-oriented way, then we’ll use an approach that’s a bit more natural in Rust. Let’s dig in to incrementally implementing a blog post workflow using the state pattern.

状态模式是一种面向对象的设计模式。该模式的关键在于我们定义一个值在内部可以具有的一组状态。 状态由一组状态对象表示，值的行为根据其状态而变化。 我们将研究一个博客文章结构的示例，该结构有一个字段来保存其状态，该字段将是“草稿”、“审阅”或“已发布”集合中的状态对象。

状态对象共享功能：当然，在 Rust 中，我们使用结构和特征，而不是对象和继承。 每个状态对象对其自身的行为负责，并负责管理何时应更改为另一种状态。 保存状态对象的值对状态的不同行为或何时在状态之间转换一无所知。

使用状态模式的优点是，当程序的业务需求发生变化时，我们不需要更改保存状态的值的代码或使用该值的代码。 我们只需要更新状态对象之一内的代码即可更改其规则，或者添加更多状态对象。

首先，我们将以更传统的面向对象的方式实现状态模式，然后我们将使用一种在 Rust 中更自然的方法。 让我们深入研究使用状态模式逐步实现博客文章工作流程。

The final functionality will look like this:

+ A blog post starts as an empty draft.
+ When the draft is done, a review of the post is requested.
+ When the post is approved, it gets published.
+ Only published blog posts return content to print, so unapproved posts can’t accidentally be published.

+ 博客文章以空草稿开始。
+ 草稿完成后，要求对帖子进行审查。
+ 帖子获得批准后，就会发布。
+ 只有已发布的博客文章才会返回要打印的内容，因此不会意外发布未经批准的帖子。

Any other changes attempted on a post should have no effect. For example, if we try to approve a draft blog post before we’ve requested a review, the post should remain an unpublished draft.

Listing 17-11 shows this workflow in code form: this is an example usage of the API we’ll implement in a library crate named blog. This won’t compile yet because we haven’t implemented the blog crate.

对帖子尝试的任何其他更改都不会产生任何影响。 例如，如果我们在请求审核之前尝试批准博客文章草稿，则该文章应保留为未发布的草稿。

清单 17-11 以代码形式显示了此工作流程：这是我们将在名为 blog 的库箱中实现的 API 的示例用法。 这还无法编译，因为我们还没有实现博客箱。

Filename: src/main.rs
``````
This code does not compile!
use blog::Post;

fn main() {
    let mut post = Post::new();

    post.add_text("I ate a salad for lunch today");
    assert_eq!("", post.content());

    post.request_review();
    assert_eq!("", post.content());

    post.approve();
    assert_eq!("I ate a salad for lunch today", post.content());
}
``````
Listing 17-11: Code that demonstrates the desired behavior we want our blog crate to have

We want to allow the user to create a new draft blog post with Post::new. We want to allow text to be added to the blog post. If we try to get the post’s content immediately, before approval, we shouldn’t get any text because the post is still a draft. We’ve added assert_eq! in the code for demonstration purposes. An excellent unit test for this would be to assert that a draft blog post returns an empty string from the content method, but we’re not going to write tests for this example.

Next, we want to enable a request for a review of the post, and we want content to return an empty string while waiting for the review. When the post receives approval, it should get published, meaning the text of the post will be returned when content is called.

Notice that the only type we’re interacting with from the crate is the Post type. This type will use the state pattern and will hold a value that will be one of three state objects representing the various states a post can be in—draft, waiting for review, or published. Changing from one state to another will be managed internally within the Post type. The states change in response to the methods called by our library’s users on the Post instance, but they don’t have to manage the state changes directly. Also, users can’t make a mistake with the states, like publishing a post before it’s reviewed.

我们希望允许用户使用 Post::new 创建新的博客文章草稿。 我们希望允许将文本添加到博客文章中。 如果我们尝试在批准之前立即获取帖子的内容，我们不应该得到任何文本，因为该帖子仍然是草稿。 我们添加了assert_eq！ 在代码中用于演示目的。 一个优秀的单元测试是断言草稿博客文章从 content 方法返回一个空字符串，但我们不打算为此示例编写测试。

接下来，我们要启用对帖子进行审核的请求，并且我们希望内容在等待审核时返回空字符串。 当帖子获得批准后，它应该被发布，这意味着调用内容时将返回帖子的文本。

请注意，我们与板条箱交互的唯一类型是 Post 类型。 此类型将使用状态模式，并保存一个值，该值将是三个状态对象之一，表示帖子可能处于的各种状态 - 草稿、等待审核或已发布。 从一种状态到另一种状态的更改将在 Post 类型内部进行管理。 状态会根据我们库的用户在 Post 实例上调用的方法而发生变化，但他们不必直接管理状态更改。 此外，用户不能对州犯错误，例如在帖子经过审核之前发布帖子。

### Defining Post and Creating a New Instance in the Draft State
Let’s get started on the implementation of the library! We know we need a public Post struct that holds some content, so we’ll start with the definition of the struct and an associated public new function to create an instance of Post, as shown in Listing 17-12. We’ll also make a private State trait that will define the behavior that all state objects for a Post must have.

Then Post will hold a trait object of Box<dyn State> inside an Option<T> in a private field named state to hold the state object. You’ll see why the Option<T> is necessary in a bit.

让我们开始实现该库吧！ 我们知道我们需要一个公共 Post 结构来保存一些内容，因此我们将从结构的定义和关联的公共新函数开始来创建 Post 的实例，如清单 17-12 所示。 我们还将创建一个私有 State 特征，它将定义 Post 的所有状态对象必须具有的行为。

然后 Post 将在名为 state 的私有字段的 Option<T> 内保存 Box<dyn State> 的特征对象，以保存状态对象。 稍后您就会明白为什么 Option<T> 是必要的。

Filename: src/lib.rs
``````
pub struct Post {
    state: Option<Box<dyn State>>,
    content: String,
}

impl Post {
    pub fn new() -> Post {
        Post {
            state: Some(Box::new(Draft {})),
            content: String::new(),
        }
    }
}

trait State {}

struct Draft {}

impl State for Draft {}
``````
Listing 17-12: Definition of a Post struct and a new function that creates a new Post instance, a State trait, and a Draft struct

The State trait defines the behavior shared by different post states. The state objects are Draft, PendingReview, and Published, and they will all implement the State trait. For now, the trait doesn’t have any methods, and we’ll start by defining just the Draft state because that is the state we want a post to start in.

When we create a new Post, we set its state field to a Some value that holds a Box. This Box points to a new instance of the Draft struct. This ensures whenever we create a new instance of Post, it will start out as a draft. Because the state field of Post is private, there is no way to create a Post in any other state! In the Post::new function, we set the content field to a new, empty String.

状态特征定义了不同帖子状态共享的行为。 状态对象是 Draft、PendingReview 和 Published，它们都将实现 State 特征。 目前，该特征没有任何方法，我们将首先定义草稿状态，因为这是我们希望帖子开始的状态。

当我们创建一个新的帖子时，我们将其状态字段设置为包含 Box 的 Some 值。 该 Box 指向 Draft 结构的新实例。 这确保了每当我们创建一个新的 Post 实例时，它都会以草稿形式开始。 因为Post的state字段是私有的，所以没有办法在任何其他状态下创建Post！ 在 Post::new 函数中，我们将内容字段设置为新的空字符串。

### Storing the Text of the Post Content
We saw in Listing 17-11 that we want to be able to call a method named add_text and pass it a &str that is then added as the text content of the blog post. We implement this as a method, rather than exposing the content field as pub, so that later we can implement a method that will control how the content field’s data is read. The add_text method is pretty straightforward, so let’s add the implementation in Listing 17-13 to the impl Post block:

我们在清单 17-11 中看到，我们希望能够调用名为 add_text 的方法并向其传递一个 &str，然后将其添加为博客文章的文本内容。 我们将其实现为一种方法，而不是将内容字段公开为 pub，以便稍后我们可以实现一个方法来控制如何读取内容字段的数据。 add_text 方法非常简单，所以让我们将清单 17-13 中的实现添加到 impl Post 块中：

Filename: src/lib.rs
``````
impl Post {
    // --snip--
    pub fn add_text(&mut self, text: &str) {
        self.content.push_str(text);
    }
}
``````
Listing 17-13: Implementing the add_text method to add text to a post’s content

The add_text method takes a mutable reference to self, because we’re changing the Post instance that we’re calling add_text on. We then call push_str on the String in content and pass the text argument to add to the saved content. This behavior doesn’t depend on the state the post is in, so it’s not part of the state pattern. The add_text method doesn’t interact with the state field at all, but it is part of the behavior we want to support.

add_text 方法采用对 self 的可变引用，因为我们正在更改调用 add_text 的 Post 实例。 然后，我们对内容中的字符串调用push_str，并传递文本参数以添加到保存的内容中。 此行为不取决于帖子所处的状态，因此它不是状态模式的一部分。 add_text 方法根本不与状态字段交互，但它是我们想要支持的行为的一部分。

### Ensuring the Content of a Draft Post Is Empty
Even after we’ve called add_text and added some content to our post, we still want the content method to return an empty string slice because the post is still in the draft state, as shown on line 7 of Listing 17-11. For now, let’s implement the content method with the simplest thing that will fulfill this requirement: always returning an empty string slice. We’ll change this later once we implement the ability to change a post’s state so it can be published. So far, posts can only be in the draft state, so the post content should always be empty. Listing 17-14 shows this placeholder implementation:

即使我们调用了 add_text 并向帖子添加了一些内容，我们仍然希望 content 方法返回一个空字符串切片，因为帖子仍处于草稿状态，如清单 17-11 的第 7 行所示。 现在，让我们用最简单的方法来实现 content 方法来满足此要求：始终返回一个空字符串切片。 一旦我们实现了更改帖子状态以便发布的功能，我们将在稍后更改此设置。 到目前为止，帖子只能处于草稿状态，因此帖子内容应始终为空。 清单 17-14 显示了这个占位符的实现：

Filename: src/lib.rs
``````
impl Post {
    // --snip--
    pub fn content(&self) -> &str {
        ""
    }
}
``````
Listing 17-14: Adding a placeholder implementation for the content method on Post that always returns an empty string slice

With this added content method, everything in Listing 17-11 up to line 7 works as intended.

### Requesting a Review of the Post Changes Its State
Next, we need to add functionality to request a review of a post, which should change its state from Draft to PendingReview. Listing 17-15 shows this code:

接下来，我们需要添加功能来请求对帖子进行审核，这应该将其状态从 Draft 更改为 PendingReview。 清单 17-15 显示了以下代码：

Filename: src/lib.rs
``````
impl Post {
    // --snip--
    pub fn request_review(&mut self) {
        if let Some(s) = self.state.take() {
            self.state = Some(s.request_review())
        }
    }
}

trait State {
    fn request_review(self: Box<Self>) -> Box<dyn State>;
}

struct Draft {}

impl State for Draft {
    fn request_review(self: Box<Self>) -> Box<dyn State> {
        Box::new(PendingReview {})
    }
}

struct PendingReview {}

impl State for PendingReview {
    fn request_review(self: Box<Self>) -> Box<dyn State> {
        self
    }
}
``````
Listing 17-15: Implementing request_review methods on Post and the State trait

We give Post a public method named request_review that will take a mutable reference to self. Then we call an internal request_review method on the current state of Post, and this second request_review method consumes the current state and returns a new state.

We add the request_review method to the State trait; all types that implement the trait will now need to implement the request_review method. Note that rather than having self, &self, or &mut self as the first parameter of the method, we have self: Box<Self>. This syntax means the method is only valid when called on a Box holding the type. This syntax takes ownership of Box<Self>, invalidating the old state so the state value of the Post can transform into a new state.

To consume the old state, the request_review method needs to take ownership of the state value. This is where the Option in the state field of Post comes in: we call the take method to take the Some value out of the state field and leave a None in its place, because Rust doesn’t let us have unpopulated fields in structs. This lets us move the state value out of Post rather than borrowing it. Then we’ll set the post’s state value to the result of this operation.

We need to set state to None temporarily rather than setting it directly with code like self.state = self.state.request_review(); to get ownership of the state value. This ensures Post can’t use the old state value after we’ve transformed it into a new state.

The request_review method on Draft returns a new, boxed instance of a new PendingReview struct, which represents the state when a post is waiting for a review. The PendingReview struct also implements the request_review method but doesn’t do any transformations. Rather, it returns itself, because when we request a review on a post already in the PendingReview state, it should stay in the PendingReview state.

Now we can start seeing the advantages of the state pattern: the request_review method on Post is the same no matter its state value. Each state is responsible for its own rules.

We’ll leave the content method on Post as is, returning an empty string slice. We can now have a Post in the PendingReview state as well as in the Draft state, but we want the same behavior in the PendingReview state. Listing 17-11 now works up to line 10!

我们为 Post 提供了一个名为 request_review 的公共方法，该方法将采用对 self 的可变引用。 然后我们对 Post 的当前状态调用内部 request_review 方法，第二个 request_review 方法消耗当前状态并返回一个新状态。

我们将 request_review 方法添加到 State 特征中； 所有实现该特征的类型现在都需要实现 request_review 方法。 请注意，我们没有使用 self、&self 或 &mut self 作为方法的第一个参数，而是使用 self: Box<Self>。 此语法意味着该方法仅在保存该类型的 Box 上调用时才有效。 此语法获取 Box<Self> 的所有权，使旧状态无效，以便 Post 的状态值可以转换为新状态。

要使用旧状态，request_review 方法需要获取状态值的所有权。 这就是 Post 的 state 字段中的 Option 发挥作用的地方：我们调用 take 方法从 state 字段中取出 Some 值，并在其位置保留 None ，因为 Rust 不允许我们在结构中拥有未填充的字段。 这让我们可以将状态值移出 Post，而不是借用它。 然后我们将帖子的状态值设置为该操作的结果。

我们需要暂时将 state 设置为 None，而不是直接使用 self.state = self.state.request_review(); 之类的代码设置它。 获得国家价值的所有权。 这确保了在我们将旧状态值转换为新状态后，Post 无法使用旧状态值。

Draft 上的 request_review 方法返回一个新的 PendingReview 结构的装箱实例，它表示帖子等待审核时的状态。 PendingReview 结构还实现了 request_review 方法，但不执行任何转换。 相反，它会返回自身，因为当我们请求对已处于 PendingReview 状态的帖子进行审阅时，它应该保持在 PendingReview 状态。

现在我们可以开始看到状态模式的优点：Post 上的 request_review 方法无论其状态值如何都是相同的。 每个州都有自己的规则。

我们将原样保留 Post 上的 content 方法，返回一个空字符串切片。 我们现在可以在 PendingReview 状态和 Draft 状态下都有一个 Post，但我们希望在 PendingReview 状态下有相同的行为。 清单 17-11 现在可以运行到第 10 行！


### Adding approve to Change the Behavior of content
The approve method will be similar to the request_review method: it will set state to the value that the current state says it should have when that state is approved, as shown in Listing 17-16:

approve 方法与 request_review 方法类似：它将 state 设置为当前状态规定的当状态被批准时应具有的值，如清单 17-16 所示：

Filename: src/lib.rs
``````
impl Post {
    // --snip--
    pub fn approve(&mut self) {
        if let Some(s) = self.state.take() {
            self.state = Some(s.approve())
        }
    }
}

trait State {
    fn request_review(self: Box<Self>) -> Box<dyn State>;
    fn approve(self: Box<Self>) -> Box<dyn State>;
}

struct Draft {}

impl State for Draft {
    // --snip--
    fn approve(self: Box<Self>) -> Box<dyn State> {
        self
    }
}

struct PendingReview {}

impl State for PendingReview {
    // --snip--
    fn approve(self: Box<Self>) -> Box<dyn State> {
        Box::new(Published {})
    }
}

struct Published {}

impl State for Published {
    fn request_review(self: Box<Self>) -> Box<dyn State> {
        self
    }

    fn approve(self: Box<Self>) -> Box<dyn State> {
        self
    }
}
``````
Listing 17-16: Implementing the approve method on Post and the State trait

We add the approve method to the State trait and add a new struct that implements State, the Published state.

Similar to the way request_review on PendingReview works, if we call the approve method on a Draft, it will have no effect because approve will return self. When we call approve on PendingReview, it returns a new, boxed instance of the Published struct. The Published struct implements the State trait, and for both the request_review method and the approve method, it returns itself, because the post should stay in the Published state in those cases.

Now we need to update the content method on Post. We want the value returned from content to depend on the current state of the Post, so we’re going to have the Post delegate to a content method defined on its state, as shown in Listing 17-17:

我们将approve方法添加到State特征中，并添加一个实现State（已发布状态）的新结构。

与 PendingReview 上的 request_review 的工作方式类似，如果我们在草稿上调用approve方法，它将不会产生任何效果，因为approve将返回self。 当我们对 PendingReview 调用approve时，它会返回 Published 结构的一个新的装箱实例。 Published 结构实现了 State 特征，对于 request_review 方法和批准方法，它返回自身，因为在这些情况下帖子应该保持在 Published 状态。

现在我们需要更新 Post 上的 content 方法。 我们希望从 content 返回的值取决于 Post 的当前状态，因此我们将 Post 委托给在其状态上定义的 content 方法，如清单 17-17 所示：

Filename: src/lib.rs
``````
This code does not compile!
impl Post {
    // --snip--
    pub fn content(&self) -> &str {
        self.state.as_ref().unwrap().content(self)
    }
    // --snip--
}
``````
Listing 17-17: Updating the content method on Post to delegate to a content method on State

Because the goal is to keep all these rules inside the structs that implement State, we call a content method on the value in state and pass the post instance (that is, self) as an argument. Then we return the value that’s returned from using the content method on the state value.

We call the as_ref method on the Option because we want a reference to the value inside the Option rather than ownership of the value. Because state is an Option<Box<dyn State>>, when we call as_ref, an Option<&Box<dyn State>> is returned. If we didn’t call as_ref, we would get an error because we can’t move state out of the borrowed &self of the function parameter.

We then call the unwrap method, which we know will never panic, because we know the methods on Post ensure that state will always contain a Some value when those methods are done. This is one of the cases we talked about in the “Cases In Which You Have More Information Than the Compiler” section of Chapter 9 when we know that a None value is never possible, even though the compiler isn’t able to understand that.

At this point, when we call content on the &Box<dyn State>, deref coercion will take effect on the & and the Box so the content method will ultimately be called on the type that implements the State trait. That means we need to add content to the State trait definition, and that is where we’ll put the logic for what content to return depending on which state we have, as shown in Listing 17-18:

因为目标是将所有这些规则保留在实现 State 的结构中，所以我们对 state 中的值调用 content 方法，并将 post 实例（即 self）作为参数传递。 然后我们返回对状态值使用 content 方法返回的值。

我们在 Option 上调用 as_ref 方法，因为我们想要引用 Option 内的值而不是该值的所有权。 因为 state 是一个 Option<Box<dyn State>>，所以当我们调用 as_ref 时，会返回一个 Option<&Box<dyn State>>。 如果我们不调用 as_ref，我们会收到错误，因为我们无法将状态移出函数参数借用的 &self 。

然后我们调用 unwrap 方法，我们知道它永远不会恐慌，因为我们知道 Post 上的方法确保当这些方法完成时状态将始终包含 Some 值。 这是我们在第 9 章“比编译器拥有更多信息的情况”部分中讨论的情况之一，当时我们知道 None 值永远不可能，即使编译器无法理解这一点。

此时，当我们在 &Box<dyn State> 上调用 content 时，deref 强制转换将对 & 和 Box 生效，因此最终将在实现 State 特征的类型上调用 content 方法。 这意味着我们需要将内容添加到 State 特征定义中，这就是我们将根据我们拥有的状态来放置返回内容的逻辑的地方，如清单 17-18 所示：

Filename: src/lib.rs
```
trait State {
    // --snip--
    fn content<'a>(&self, post: &'a Post) -> &'a str {
        ""
    }
}

// --snip--
struct Published {}

impl State for Published {
    // --snip--
    fn content<'a>(&self, post: &'a Post) -> &'a str {
        &post.content
    }
}
```
Listing 17-18: Adding the content method to the State trait

We add a default implementation for the content method that returns an empty string slice. That means we don’t need to implement content on the Draft and PendingReview structs. The Published struct will override the content method and return the value in post.content.

Note that we need lifetime annotations on this method, as we discussed in Chapter 10. We’re taking a reference to a post as an argument and returning a reference to part of that post, so the lifetime of the returned reference is related to the lifetime of the post argument.

And we’re done—all of Listing 17-11 now works! We’ve implemented the state pattern with the rules of the blog post workflow. The logic related to the rules lives in the state objects rather than being scattered throughout Post.

我们为返回空字符串切片的 content 方法添加了默认实现。 这意味着我们不需要在 Draft 和 PendingReview 结构上实现内容。 Published 结构将重写 content 方法并返回 post.content 中的值。

请注意，我们需要对此方法进行生命周期注释，正如我们在第 10 章中讨论的那样。我们将对帖子的引用作为参数并返回对该帖子的部分内容的引用，因此返回引用的生命周期与 post 参数的生命周期。

我们就完成了——清单 17-11 的所有内容现在都可以工作了！ 我们已经使用博客文章工作流程的规则实现了状态模式。 与规则相关的逻辑存在于状态对象中，而不是分散在整个 Post 中。
``````
Why Not An Enum?
You may have been wondering why we didn’t use an enum with the different possible post states as variants. That’s certainly a possible solution, try it and compare the end results to see which you prefer! One disadvantage of using an enum is every place that checks the value of the enum will need a match expression or similar to handle every possible variant. This could get more repetitive than this trait object solution.
``````

### Trade-offs of the State Pattern
We’ve shown that Rust is capable of implementing the object-oriented state pattern to encapsulate the different kinds of behavior a post should have in each state. The methods on Post know nothing about the various behaviors. The way we organized the code, we have to look in only one place to know the different ways a published post can behave: the implementation of the State trait on the Published struct.

If we were to create an alternative implementation that didn’t use the state pattern, we might instead use match expressions in the methods on Post or even in the main code that checks the state of the post and changes behavior in those places. That would mean we would have to look in several places to understand all the implications of a post being in the published state! This would only increase the more states we added: each of those match expressions would need another arm.

With the state pattern, the Post methods and the places we use Post don’t need match expressions, and to add a new state, we would only need to add a new struct and implement the trait methods on that one struct.

The implementation using the state pattern is easy to extend to add more functionality. To see the simplicity of maintaining code that uses the state pattern, try a few of these suggestions:

我们已经证明 Rust 能够实现面向对象的状态模式来封装帖子在每种状态下应具有的不同类型的行为。 Post 上的方法对各种行为一无所知。 根据我们组织代码的方式，我们只需查看一个地方即可了解已发布帖子的不同行为方式：已发布结构上 State 特征的实现。

如果我们要创建一个不使用状态模式的替代实现，我们可能会在 Post 的方法中甚至在检查帖子状态并更改这些位置的行为的主代码中使用匹配表达式。 这意味着我们必须在几个地方查看才能理解帖子处于已发布状态的所有含义！ 这只会增加我们添加的更多状态：每个匹配表达式都需要另一个臂。

使用状态模式，Post 方法和我们使用 Post 的地方不需要匹配表达式，并且要添加新状态，我们只需要添加一个新结构并在该结构上实现特征方法。

使用状态模式的实现很容易扩展以添加更多功能。 要了解维护使用状态模式的代码的简单性，请尝试以下一些建议：

+ Add a reject method that changes the post’s state from PendingReview back to Draft.
+ Require two calls to approve before the state can be changed to Published.
+ Allow users to add text content only when a post is in the Draft state. Hint: have the state object responsible for what might change about the content but not responsible for modifying the Post.

+ 添加拒绝方法，将帖子的状态从 PendingReview 更改回草稿。
+ 需要两次调用才能批准，然后才能将状态更改为已发布。
+ 仅当帖子处于草稿状态时才允许用户添加文本内容。 提示：让状态对象负责内容可能发生的变化，但不负责修改帖子。

One downside of the state pattern is that, because the states implement the transitions between states, some of the states are coupled to each other. If we add another state between PendingReview and Published, such as Scheduled, we would have to change the code in PendingReview to transition to Scheduled instead. It would be less work if PendingReview didn’t need to change with the addition of a new state, but that would mean switching to another design pattern.

Another downside is that we’ve duplicated some logic. To eliminate some of the duplication, we might try to make default implementations for the request_review and approve methods on the State trait that return self; however, this would violate object safety, because the trait doesn’t know what the concrete self will be exactly. We want to be able to use State as a trait object, so we need its methods to be object safe.

Other duplication includes the similar implementations of the request_review and approve methods on Post. Both methods delegate to the implementation of the same method on the value in the state field of Option and set the new value of the state field to the result. If we had a lot of methods on Post that followed this pattern, we might consider defining a macro to eliminate the repetition (see the “Macros” section in Chapter 19).

By implementing the state pattern exactly as it’s defined for object-oriented languages, we’re not taking as full advantage of Rust’s strengths as we could. Let’s look at some changes we can make to the blog crate that can make invalid states and transitions into compile time errors.

状态模式的一个缺点是，由于状态实现状态之间的转换，因此某些状态相互耦合。 如果我们在 PendingReview 和 Published 之间添加另一个状态（例如 Scheduled），则必须更改 PendingReview 中的代码以转换为 Scheduled。 如果 PendingReview 不需要因添加新状态而进行更改，那么工作量会减少，但这意味着要切换到另一种设计模式。

另一个缺点是我们重复了一些逻辑。 为了消除一些重复，我们可能会尝试为返回 self 的 State 特征上的 request_review 和批准方法进行默认实现； 然而，这会违反对象安全，因为该特征不知道具体的自我到底是什么。 我们希望能够使用 State 作为特征对象，因此我们需要它的方法是对象安全的。

其他重复包括 Post 上 request_review 和approve 方法的类似实现。 这两个方法都委托对 Option 的 state 字段中的值执行相同的方法，并将 state 字段的新值设置为结果。 如果我们在 Post 上有很多遵循这种模式的方法，我们可能会考虑定义一个宏来消除重复（参见第 19 章中的“宏”部分）。

通过完全按照面向对象语言的定义来实现状态模式，我们并没有充分利用 Rust 的优势。 让我们看一下可以对博客箱进行的一些更改，这些更改可能会导致无效状态并转换为编译时错误。

### Encoding States and Behavior as Types
We’ll show you how to rethink the state pattern to get a different set of trade-offs. Rather than encapsulating the states and transitions completely so outside code has no knowledge of them, we’ll encode the states into different types. Consequently, Rust’s type checking system will prevent attempts to use draft posts where only published posts are allowed by issuing a compiler error.

我们将向您展示如何重新思考状态模式以获得一组不同的权衡。 我们不会完全封装状态和转换，使外部代码不知道它们，而是将状态编码为不同的类型。 因此，Rust 的类型检查系统将通过发出编译器错误来阻止尝试使用仅允许发布的帖子的草稿帖子。

Let’s consider the first part of main in Listing 17-11:

Filename: src/main.rs
``````
fn main() {
    let mut post = Post::new();

    post.add_text("I ate a salad for lunch today");
    assert_eq!("", post.content());
}
``````
We still enable the creation of new posts in the draft state using Post::new and the ability to add text to the post’s content. But instead of having a content method on a draft post that returns an empty string, we’ll make it so draft posts don’t have the content method at all. That way, if we try to get a draft post’s content, we’ll get a compiler error telling us the method doesn’t exist. As a result, it will be impossible for us to accidentally display draft post content in production, because that code won’t even compile. Listing 17-19 shows the definition of a Post struct and a DraftPost struct, as well as methods on each:

我们仍然可以使用 Post::new 在草稿状态下创建新帖子，并能够向帖子内容添加文本。 但是，我们不会在草稿帖子上使用返回空字符串的 content 方法，而是让草稿帖子根本没有 content 方法。 这样，如果我们尝试获取草稿帖子的内容，我们将收到编译器错误，告诉我们该方法不存在。 因此，我们不可能在生产中意外地显示草稿帖子内容，因为该代码甚至无法编译。 清单 17-19 显示了 Post 结构和 DraftPost 结构的定义，以及每个结构的方法：

Filename: src/lib.rs
``````
pub struct Post {
    content: String,
}

pub struct DraftPost {
    content: String,
}

impl Post {
    pub fn new() -> DraftPost {
        DraftPost {
            content: String::new(),
        }
    }

    pub fn content(&self) -> &str {
        &self.content
    }
}

impl DraftPost {
    pub fn add_text(&mut self, text: &str) {
        self.content.push_str(text);
    }
}
``````
Listing 17-19: A Post with a content method and a DraftPost without a content method

Both the Post and DraftPost structs have a private content field that stores the blog post text. The structs no longer have the state field because we’re moving the encoding of the state to the types of the structs. The Post struct will represent a published post, and it has a content method that returns the content.

We still have a Post::new function, but instead of returning an instance of Post, it returns an instance of DraftPost. Because content is private and there aren’t any functions that return Post, it’s not possible to create an instance of Post right now.

The DraftPost struct has an add_text method, so we can add text to content as before, but note that DraftPost does not have a content method defined! So now the program ensures all posts start as draft posts, and draft posts don’t have their content available for display. Any attempt to get around these constraints will result in a compiler error.

Post 和 DraftPost 结构都有一个存储博客文章文本的私有内容字段。 结构体不再具有状态字段，因为我们将状态编码转移到结构体的类型。 Post 结构将表示已发布的帖子，并且它有一个返回内容的 content 方法。

我们仍然有一个 Post::new 函数，但它不是返回 Post 的实例，而是返回 DraftPost 的实例。 由于内容是私有的，并且没有任何返回 Post 的函数，因此现在无法创建 Post 实例。

DraftPost 结构有一个 add_text 方法，因此我们可以像以前一样向内容添加文本，但请注意，DraftPost 没有定义 content 方法！ 因此，现在该程序可确保所有帖子都以草稿帖子开始，而草稿帖子的内容无法显示。 任何绕过这些限制的尝试都会导致编译器错误。

### Implementing Transitions as Transformations into Different Types
So how do we get a published post? We want to enforce the rule that a draft post has to be reviewed and approved before it can be published. A post in the pending review state should still not display any content. Let’s implement these constraints by adding another struct, PendingReviewPost, defining the request_review method on DraftPost to return a PendingReviewPost, and defining an approve method on PendingReviewPost to return a Post, as shown in Listing 17-20:

那么我们如何获得已发布的帖子呢？ 我们希望强制执行这样的规则：草稿帖子必须经过审查和批准才能发布。 处于待审状态的帖子不应显示任何内容。 让我们通过添加另一个结构体 PendingReviewPost 来实现这些约束，在 DraftPost 上定义 request_review 方法以返回 PendingReviewPost，并在 PendingReviewPost 上定义批准方法以返回 Post，如清单 17-20 所示：

Filename: src/lib.rs
``````
impl DraftPost {
    // --snip--
    pub fn request_review(self) -> PendingReviewPost {
        PendingReviewPost {
            content: self.content,
        }
    }
}

pub struct PendingReviewPost {
    content: String,
}

impl PendingReviewPost {
    pub fn approve(self) -> Post {
        Post {
            content: self.content,
        }
    }
}
``````
Listing 17-20: A PendingReviewPost that gets created by calling request_review on DraftPost and an approve method that turns a PendingReviewPost into a published Post

The request_review and approve methods take ownership of self, thus consuming the DraftPost and PendingReviewPost instances and transforming them into a PendingReviewPost and a published Post, respectively. This way, we won’t have any lingering DraftPost instances after we’ve called request_review on them, and so forth. The PendingReviewPost struct doesn’t have a content method defined on it, so attempting to read its content results in a compiler error, as with DraftPost. Because the only way to get a published Post instance that does have a content method defined is to call the approve method on a PendingReviewPost, and the only way to get a PendingReviewPost is to call the request_review method on a DraftPost, we’ve now encoded the blog post workflow into the type system.

But we also have to make some small changes to main. The request_review and approve methods return new instances rather than modifying the struct they’re called on, so we need to add more let post = shadowing assignments to save the returned instances. We also can’t have the assertions about the draft and pending review posts’ contents be empty strings, nor do we need them: we can’t compile code that tries to use the content of posts in those states any longer. The updated code in main is shown in Listing 17-21:

request_review 和approve 方法拥有self 的所有权，从而使用DraftPost 和PendingReviewPost 实例并将它们分别转换为PendingReviewPost 和已发布的Post。 这样，在调用 request_review 等之后，我们就不会再有任何残留的 DraftPost 实例。 PendingReviewPost 结构没有定义内容方法，因此尝试读取其内容会导致编译器错误，就像 DraftPost 一样。 因为获取已定义内容方法的已发布 Post 实例的唯一方法是调用 PendingReviewPost 上的批准方法，而获取 PendingReviewPost 的唯一方法是调用 DraftPost 上的 request_review 方法，所以我们现在已经编码 将博客文章工作流程放入类型系统中。

但我们还必须对 main.js 进行一些小的更改。 request_review 和approve 方法返回新实例，而不是修改它们所调用的结构，因此我们需要添加更多let post =shadowing 赋值来保存返回的实例。 我们也不能让关于草稿和待审帖子内容的断言为空字符串，我们也不需要它们：我们不能再编译尝试使用这些状态下的帖子内容的代码。 main 中更新后的代码如清单 17-21 所示：

Filename: src/main.rs
``````
use blog::Post;

fn main() {
    let mut post = Post::new();

    post.add_text("I ate a salad for lunch today");

    let post = post.request_review();

    let post = post.approve();

    assert_eq!("I ate a salad for lunch today", post.content());
}
``````
Listing 17-21: Modifications to main to use the new implementation of the blog post workflow

The changes we needed to make to main to reassign post mean that this implementation doesn’t quite follow the object-oriented state pattern anymore: the transformations between the states are no longer encapsulated entirely within the Post implementation. However, our gain is that invalid states are now impossible because of the type system and the type checking that happens at compile time! This ensures that certain bugs, such as display of the content of an unpublished post, will be discovered before they make it to production.

Try the tasks suggested at the start of this section on the blog crate as it is after Listing 17-21 to see what you think about the design of this version of the code. Note that some of the tasks might be completed already in this design.

We’ve seen that even though Rust is capable of implementing object-oriented design patterns, other patterns, such as encoding state into the type system, are also available in Rust. These patterns have different trade-offs. Although you might be very familiar with object-oriented patterns, rethinking the problem to take advantage of Rust’s features can provide benefits, such as preventing some bugs at compile time. Object-oriented patterns won’t always be the best solution in Rust due to certain features, like ownership, that object-oriented languages don’t have.

我们需要对 main 进行更改以重新分配 post 意味着此实现不再完全遵循面向对象的状态模式：状态之间的转换不再完全封装在 Post 实现中。 然而，我们的收获是，由于类型系统和编译时发生的类型检查，无效状态现在是不可能的！ 这可以确保某些错误（例如未发布帖子内容的显示）在投入生产之前被发现。

尝试在博客箱上执行本节开头建议的任务（如清单 17-21 之后），看看您对此版本代码的设计有何看法。 请注意，某些任务可能已在此设计中完成。

我们已经看到，尽管 Rust 能够实现面向对象的设计模式，但 Rust 中也可以使用其他模式，例如将状态编码到类型系统中。 这些模式有不同的权衡。 尽管您可能非常熟悉面向对象的模式，但重新思考问题以利用 Rust 的功能可以带来好处，例如在编译时防止一些错误。 由于面向对象语言不具备某些功能（例如所有权），面向对象模式并不总是 Rust 中的最佳解决方案。

## 17.4 Summary
No matter whether or not you think Rust is an object-oriented language after reading this chapter, you now know that you can use trait objects to get some object-oriented features in Rust. Dynamic dispatch can give your code some flexibility in exchange for a bit of runtime performance. You can use this flexibility to implement object-oriented patterns that can help your code’s maintainability. Rust also has other features, like ownership, that object-oriented languages don’t have. An object-oriented pattern won’t always be the best way to take advantage of Rust’s strengths, but is an available option.

无论你在读完本章后是否认为 Rust 是一种面向对象的语言，你现在都知道你可以使用 Trait 对象来获得 Rust 中的一些面向对象的特性。 动态调度可以为您的代码提供一些灵活性，以换取一些运行时性能。 您可以利用这种灵活性来实现面向对象的模式，从而提高代码的可维护性。 Rust 还具有面向对象语言所没有的其他功能，例如所有权。 面向对象的模式并不总是利用 Rust 优势的最佳方式，但它是一个可用的选择。

Next, we’ll look at patterns, which are another of Rust’s features that enable lots of flexibility. We’ve looked at them briefly throughout the book but haven’t seen their full capability yet. Let’s go!