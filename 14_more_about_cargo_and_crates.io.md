# 14 More about Cargo and Crates.io

So far we’ve used only the most basic features of Cargo to build, run, and test our code, but it can do a lot more. In this chapter, we’ll discuss some of its other, more advanced features to show you how to do the following:

+ Customize your build through release profiles
+ Publish libraries on crates.io
+ Organize large projects with workspaces
+ Install binaries from crates.io
+ Extend Cargo using custom commands

Cargo can do even more than the functionality we cover in this chapter, so for a full explanation of all its features, see its documentation.

到目前为止，我们只使用了 Cargo 最基本的功能来构建、运行和测试我们的代码，但它可以做更多的事情。 在本章中，我们将讨论它的一些其他更高级的功能，以向您展示如何执行以下操作：

+ 通过发布配置文件自定义您的构建
+ 在 crates.io 上发布库
+ 使用工作区组织大型项目
+ 从 crates.io 安装二进制文件
+ 使用自定义命令扩展 Cargo

Cargo 的功能甚至比我们在本章中介绍的功能还要多，因此有关其所有功能的完整说明，请参阅其文档。

## 14.1 Customizing Build with Release Profiles
In Rust, release profiles are predefined and customizable profiles with different configurations that allow a programmer to have more control over various options for compiling code. Each profile is configured independently of the others.

Cargo has two main profiles: the dev profile Cargo uses when you run cargo build and the release profile Cargo uses when you run cargo build --release. The dev profile is defined with good defaults for development, and the release profile has good defaults for release builds.

These profile names might be familiar from the output of your builds:

在 Rust 中，发布配置文件是预定义和可定制的配置文件，具有不同的配置，允许程序员更好地控制编译代码的各种选项。 每个配置文件的配置均独立于其他配置文件。

Cargo 有两个主要配置文件：运行 Cargo build 时 Cargo 使用的 dev 配置文件和运行 Cargo build --release 时 Cargo 使用的发布配置文件。 开发配置文件定义了良好的开发默认值，发布配置文件定义了良好的发布版本默认值。

从构建的输出中，这些配置文件名称可能很熟悉：

``````
$ cargo build
    Finished dev [unoptimized + debuginfo] target(s) in 0.0s
$ cargo build --release
    Finished release [optimized] target(s) in 0.0s
``````
The dev and release are these different profiles used by the compiler.

Cargo has default settings for each of the profiles that apply when you haven't explicitly added any [profile.*] sections in the project’s Cargo.toml file. By adding [profile.*] sections for any profile you want to customize, you override any subset of the default settings. For example, here are the default values for the opt-level setting for the dev and release profiles:

当您未在项目的 Cargo.toml 文件中显式添加任何 [profile.*] 部分时，Cargo 对每个配置文件都有默认设置。 通过为您想要自定义的任何配置文件添加 [profile.*] 部分，您可以覆盖默认设置的任何子集。 例如，以下是开发和发布配置文件的 opt-level 设置的默认值：

Filename: Cargo.toml
``````
[profile.dev]
opt-level = 0

[profile.release]
opt-level = 3
``````
The opt-level setting controls the number of optimizations Rust will apply to your code, with a range of 0 to 3. Applying more optimizations extends compiling time, so if you’re in development and compiling your code often, you’ll want fewer optimizations to compile faster even if the resulting code runs slower. The default opt-level for dev is therefore 0. When you’re ready to release your code, it’s best to spend more time compiling. You’ll only compile in release mode once, but you’ll run the compiled program many times, so release mode trades longer compile time for code that runs faster. That is why the default opt-level for the release profile is 3.

You can override a default setting by adding a different value for it in Cargo.toml. For example, if we want to use optimization level 1 in the development profile, we can add these two lines to our project’s Cargo.toml file:

opt-level 设置控制 Rust 将应用于您的代码的优化数量，范围为 0 到 3。应用更多优化会延长编译时间，因此，如果您经常进行开发和编译代码，则需要更少的优化 即使生成的代码运行速度较慢，也可以进行优化以加快编译速度。 因此，dev 的默认 opt-level 为 0。当您准备好发布代码时，最好花更多时间进行编译。 您只会在发布模式下编译一次，但会多次运行编译后的程序，因此发布模式会用更长的编译时间换取运行速度更快的代码。 这就是为什么发布配置文件的默认 opt-level 为 3。

您可以通过在 Cargo.toml 中添加不同的值来覆盖默认设置。 例如，如果我们想在开发配置文件中使用优化级别 1，我们可以将这两行添加到项目的 Cargo.toml 文件中：

Filename: Cargo.toml
``````
[profile.dev]
opt-level = 1
``````
This code overrides the default setting of 0. Now when we run cargo build, Cargo will use the defaults for the dev profile plus our customization to opt-level. Because we set opt-level to 1, Cargo will apply more optimizations than the default, but not as many as in a release build.

For the full list of configuration options and defaults for each profile, see Cargo’s documentation.

此代码覆盖默认设置 0。现在，当我们运行 Cargo build 时，Cargo 将使用 dev 配置文件的默认值以及我们对 opt-level 的自定义。 因为我们将 opt-level 设置为 1，所以 Cargo 将应用比默认值更多的优化，但不会像发布版本中那么多。

有关每个配置文件的配置选项和默认值的完整列表，请参阅 Cargo 的文档。

## 14.2 Publishing a Crate to Crates.io

We’ve used packages from crates.io as dependencies of our project, but you can also share your code with other people by publishing your own packages. The crate registry at crates.io distributes the source code of your packages, so it primarily hosts code that is open source.

Rust and Cargo have features that make your published package easier for people to find and use. We’ll talk about some of these features next and then explain how to publish a package.

我们使用 crates.io 中的包作为我们项目的依赖项，但您也可以通过发布自己的包来与其他人共享您的代码。 crates.io 上的板条箱注册表分发软件包的源代码，因此它主要托管开源代码。

Rust 和 Cargo 具有使您发布的包更容易被人们找到和使用的功能。 接下来我们将讨论其中一些功能，然后解释如何发布包。

### Making Useful Documentation Comments
Accurately documenting your packages will help other users know how and when to use them, so it’s worth investing the time to write documentation. In Chapter 3, we discussed how to comment Rust code using two slashes, //. Rust also has a particular kind of comment for documentation, known conveniently as a documentation comment, that will generate HTML documentation. The HTML displays the contents of documentation comments for public API items intended for programmers interested in knowing how to use your crate as opposed to how your crate is implemented.

Documentation comments use three slashes, ///, instead of two and support Markdown notation for formatting the text. Place documentation comments just before the item they’re documenting. Listing 14-1 shows documentation comments for an add_one function in a crate named my_crate.

准确记录您的包将帮助其他用户了解如何以及何时使用它们，因此值得投入时间编写文档。 在第 3 章中，我们讨论了如何使用两个斜杠 // 来注释 Rust 代码。 Rust 还有一种特殊的文档注释，方便地称为文档注释，它将生成 HTML 文档。 HTML 显示公共 API 项目的文档注释内容，供有兴趣了解如何使用您的 crate 的程序员使用，而不是了解您的 crate 的实现方式。

文档注释使用三个斜杠 ///，而不是两个，并支持 Markdown 表示法来格式化文本。 将文档注释放在他们正在记录的项目之前。 清单 14-1 显示了名为 my_crate 的 crate 中 add_one 函数的文档注释。

Filename: src/lib.rs
``````
/// Adds one to the number given.
///
/// # Examples
///
/// ```
/// let arg = 5;
/// let answer = my_crate::add_one(arg);
///
/// assert_eq!(6, answer);
/// ```
pub fn add_one(x: i32) -> i32 {
    x + 1
}
``````
Listing 14-1: A documentation comment for a function

Here, we give a description of what the add_one function does, start a section with the heading Examples, and then provide code that demonstrates how to use the add_one function. We can generate the HTML documentation from this documentation comment by running cargo doc. This command runs the rustdoc tool distributed with Rust and puts the generated HTML documentation in the target/doc directory.

For convenience, running cargo doc --open will build the HTML for your current crate’s documentation (as well as the documentation for all of your crate’s dependencies) and open the result in a web browser. Navigate to the add_one function and you’ll see how the text in the documentation comments is rendered, as shown in Figure 14-1:

Rendered HTML documentation for the `add_one` function of `my_crate`
Figure 14-1: HTML documentation for the add_one function

在这里，我们描述了 add_one 函数的作用，以示例标题开始一节，然后提供演示如何使用 add_one 函数的代码。 我们可以通过运行 Cargo doc 从此文档注释生成 HTML 文档。 此命令运行随 Rust 一起分发的 rustdoc 工具，并将生成的 HTML 文档放入 target/doc 目录中。

为了方便起见，运行 Cargo doc --open 将为当前板条箱的文档（以及所有板条箱依赖项的文档）构建 HTML，并在 Web 浏览器中打开结果。 导航到 add_one 函数，您将看到文档注释中的文本是如何呈现的，如图 14-1 所示：

“my_crate”的“add_one”函数的渲染 HTML 文档
图 14-1：add_one 函数的 HTML 文档

### Commonly Used Sections
We used the # Examples Markdown heading in Listing 14-1 to create a section in the HTML with the title “Examples.” Here are some other sections that crate authors commonly use in their documentation:

+ Panics: The scenarios in which the function being documented could panic. Callers of the function who don’t want their programs to panic should make sure they don’t call the function in these situations.
+ Errors: If the function returns a Result, describing the kinds of errors that might occur and what conditions might cause those errors to be returned can be helpful to callers so they can write code to handle the different kinds of errors in different ways.
+ Safety: If the function is unsafe to call (we discuss unsafety in Chapter 19), there should be a section explaining why the function is unsafe and covering the invariants that the function expects callers to uphold.

Most documentation comments don’t need all of these sections, but this is a good checklist to remind you of the aspects of your code users will be interested in knowing about.

我们使用清单 14-1 中的 # Examples Markdown 标题在 HTML 中创建一个标题为“Examples”的部分。 以下是 crate 作者在其文档中常用的其他一些部分：

+ 恐慌：正在记录的函数可能会出现恐慌的情况。 不希望程序出现恐慌的函数调用者应确保在这些情况下不会调用该函数。
+ 错误：如果函数返回结果，则描述可能发生的错误类型以及可能导致返回这些错误的条件对调用者很有帮助，以便他们可以编写代码以不同的方式处理不同类型的错误。
+ 安全性：如果函数调用不安全（我们在第 19 章中讨论不安全性），应该有一个部分解释为什么该函数不安全并涵盖该函数期望调用者维护的不变量。

大多数文档注释不需要所有这些部分，但这是一个很好的清单，可以提醒您用户有兴趣了解代码的哪些方面。

### Documentation Comments as Tests
Adding example code blocks in your documentation comments can help demonstrate how to use your library, and doing so has an additional bonus: running cargo test will run the code examples in your documentation as tests! Nothing is better than documentation with examples. But nothing is worse than examples that don’t work because the code has changed since the documentation was written. If we run cargo test with the documentation for the add_one function from Listing 14-1, we will see a section in the test results like this:

在文档注释中添加示例代码块可以帮助演示如何使用您的库，这样做还有一个额外的好处：运行货物测试将运行文档中的代码示例作为测试！ 没有什么比带有示例的文档更好的了。 但没有什么比示例无法运行更糟糕的了，因为自文档编写以来代码已发生更改。 如果我们使用清单 14-1 中 add_one 函数的文档运行货物测试，我们将在测试结果中看到如下部分：

``````
   Doc-tests my_crate

running 1 test
test src/lib.rs - add_one (line 5) ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.27s
``````
Now if we change either the function or the example so the assert_eq! in the example panics and run cargo test again, we’ll see that the doc tests catch that the example and the code are out of sync with each other!

现在，如果我们更改函数或示例，则assert_eq！ 在示例中出现恐慌并再次运行货物测试，我们将看到文档测试发现示例和代码彼此不同步！

### Commenting Contained Items
The style of doc comment //! adds documentation to the item that contains the comments rather than to the items following the comments. We typically use these doc comments inside the crate root file (src/lib.rs by convention) or inside a module to document the crate or the module as a whole.

For example, to add documentation that describes the purpose of the my_crate crate that contains the add_one function, we add documentation comments that start with //! to the beginning of the src/lib.rs file, as shown in Listing 14-2:

文档注释的风格 //! 将文档添加到包含注释的项目，而不是添加到注释后面的项目。 我们通常在 crate 根文件（按照惯例 src/lib.rs）或模块内部使用这些文档注释来记录 crate 或整个模块。

例如，要添加描述包含 add_one 函数的 my_crate 箱用途的文档，我们添加以 //! 开头的文档注释。 到 src/lib.rs 文件的开头，如清单 14-2 所示：

Filename: src/lib.rs
``````
//! # My Crate
//!
//! `my_crate` is a collection of utilities to make performing certain
//! calculations more convenient.

/// Adds one to the number given.
// --snip--
``````
Listing 14-2: Documentation for the my_crate crate as a whole

Notice there isn’t any code after the last line that begins with //!. Because we started the comments with //! instead of ///, we’re documenting the item that contains this comment rather than an item that follows this comment. In this case, that item is the src/lib.rs file, which is the crate root. These comments describe the entire crate.

When we run cargo doc --open, these comments will display on the front page of the documentation for my_crate above the list of public items in the crate, as shown in Figure 14-2:

Rendered HTML documentation with a comment for the crate as a whole
Figure 14-2: Rendered documentation for my_crate, including the comment describing the crate as a whole

Documentation comments within items are useful for describing crates and modules especially. Use them to explain the overall purpose of the container to help your users understand the crate’s organization.

请注意，以 //! 开头的最后一行之后没有任何代码。 因为我们以//开始评论！ 我们正在记录包含此评论的项目，而不是记录此评论之后的项目，而不是 ///。 在本例中，该项目是 src/lib.rs 文件，它是 crate 根目录。 这些评论描述了整个箱子。

当我们运行 Cargo doc --open 时，这些注释将显示在 my_crate 文档的首页上，位于 crate 中的公共项列表上方，如图 14-2 所示：

渲染的 HTML 文档，带有整个 crate 的注释
图 14-2：my_crate 的渲染文档，包括描述整个 crate 的注释

项目中的文档注释对于描述包和模块尤其有用。 使用它们来解释容器的总体用途，以帮助您的用户了解板条箱的组织。

### Exporting a Convenient Public API with pub use
The structure of your public API is a major consideration when publishing a crate. People who use your crate are less familiar with the structure than you are and might have difficulty finding the pieces they want to use if your crate has a large module hierarchy.

In Chapter 7, we covered how to make items public using the pub keyword, and bring items into a scope with the use keyword. However, the structure that makes sense to you while you’re developing a crate might not be very convenient for your users. You might want to organize your structs in a hierarchy containing multiple levels, but then people who want to use a type you’ve defined deep in the hierarchy might have trouble finding out that type exists. They might also be annoyed at having to enter use my_crate::some_module::another_module::UsefulType; rather than use my_crate::UsefulType;.

The good news is that if the structure isn’t convenient for others to use from another library, you don’t have to rearrange your internal organization: instead, you can re-export items to make a public structure that’s different from your private structure by using pub use. Re-exporting takes a public item in one location and makes it public in another location, as if it were defined in the other location instead.

For example, say we made a library named art for modeling artistic concepts. Within this library are two modules: a kinds module containing two enums named PrimaryColor and SecondaryColor and a utils module containing a function named mix, as shown in Listing 14-3:

发布 crate 时，公共 API 的结构是一个主要考虑因素。 使用你的板条箱的人不像你那么熟悉结构，如果你的板条箱有一个很大的模块层次结构，他们可能很难找到他们想要使用的部分。

在第 7 章中，我们介绍了如何使用 pub 关键字将项目公开，以及如何使用 use 关键字将项目引入范围。 然而，在开发 crate 时对您有意义的结构可能对您的用户来说不太方便。 您可能希望在包含多个级别的层次结构中组织结构体，但是想要使用您在层次结构深处定义的类型的人可能很难发现该类型的存在。 他们可能还会因为必须输入 use my_crate::some_module::another_module::UsefulType; 而感到恼火。 而不是使用 my_crate::UsefulType;。

好消息是，如果该结构不方便其他人从另一个库使用，您不必重新安排您的内部组织：相反，您可以重新导出项目以创建与您的私有结构不同的公共结构 通过使用酒吧使用。 重新导出会在一个位置获取公共项目，并将其在另一个位置公开，就像它是在另一个位置定义的一样。

例如，假设我们创建了一个名为 art 的库来建模艺术概念。 该库中有两个模块：一个包含两个名为 PrimaryColor 和 secondaryColor 的枚举的 kind 模块，一个包含名为 mix 的函数的 utils 模块，如清单 14-3 所示：

Filename: src/lib.rs
``````
//! # Art
//!
//! A library for modeling artistic concepts.

pub mod kinds {
    /// The primary colors according to the RYB color model.
    pub enum PrimaryColor {
        Red,
        Yellow,
        Blue,
    }

    /// The secondary colors according to the RYB color model.
    pub enum SecondaryColor {
        Orange,
        Green,
        Purple,
    }
}

pub mod utils {
    use crate::kinds::*;

    /// Combines two primary colors in equal amounts to create
    /// a secondary color.
    pub fn mix(c1: PrimaryColor, c2: PrimaryColor) -> SecondaryColor {
        // --snip--
    }
}
``````
Listing 14-3: An art library with items organized into kinds and utils modules

Figure 14-3 shows what the front page of the documentation for this crate generated by cargo doc would look like:

Rendered documentation for the `art` crate that lists the `kinds` and `utils` modules
Figure 14-3: Front page of the documentation for art that lists the kinds and utils modules

Note that the PrimaryColor and SecondaryColor types aren’t listed on the front page, nor is the mix function. We have to click kinds and utils to see them.

Another crate that depends on this library would need use statements that bring the items from art into scope, specifying the module structure that’s currently defined. Listing 14-4 shows an example of a crate that uses the PrimaryColor and mix items from the art crate:

图14-3显示了由cargo doc生成的这个crate的文档的首页是什么样子的：

列出了“kinds”和“utils”模块的“art”包的渲染文档
图 14-3：art 文档的首页，列出了种类和 utils 模块

请注意，PrimaryColor 和 secondaryColor 类型未在首页上列出，混合函数也未列出。 我们必须单击“种类”和“实用程序”才能看到它们。

依赖于该库的另一个板条箱将需要 use 语句将艺术中的项目带入范围，指定当前定义的模块结构。 清单 14-4 显示了一个使用 PrimaryColor 并混合 art crate 中项目的 crate 示例：

Filename: src/main.rs
``````
use art::kinds::PrimaryColor;
use art::utils::mix;

fn main() {
    let red = PrimaryColor::Red;
    let yellow = PrimaryColor::Yellow;
    mix(red, yellow);
}
``````
Listing 14-4: A crate using the art crate’s items with its internal structure exported

The author of the code in Listing 14-4, which uses the art crate, had to figure out that PrimaryColor is in the kinds module and mix is in the utils module. The module structure of the art crate is more relevant to developers working on the art crate than to those using it. The internal structure doesn’t contain any useful information for someone trying to understand how to use the art crate, but rather causes confusion because developers who use it have to figure out where to look, and must specify the module names in the use statements.

To remove the internal organization from the public API, we can modify the art crate code in Listing 14-3 to add pub use statements to re-export the items at the top level, as shown in Listing 14-5:

清单 14-4 中使用 art crate 的代码的作者必须弄清楚 PrimaryColor 位于 kinds 模块中， mix 位于 utils 模块中。 Art crate 的模块结构与处理 Art crate 的开发人员比与使用 Art crate 的开发人员更相关。 对于试图了解如何使用 art crate 的人来说，内部结构不包含任何有用的信息，反而会造成混乱，因为使用它的开发人员必须弄清楚在哪里查看，并且必须在 use 语句中指定模块名称。

要从公共 API 中删除内部组织，我们可以修改清单 14-3 中的 art crate 代码，添加 pub use 语句以重新导出顶层的项目，如清单 14-5 所示：

Filename: src/lib.rs
``````
//! # Art
//!
//! A library for modeling artistic concepts.

pub use self::kinds::PrimaryColor;
pub use self::kinds::SecondaryColor;
pub use self::utils::mix;

pub mod kinds {
    // --snip--
}

pub mod utils {
    // --snip--
}
``````
Listing 14-5: Adding pub use statements to re-export items

The API documentation that cargo doc generates for this crate will now list and link re-exports on the front page, as shown in Figure 14-4, making the PrimaryColor and SecondaryColor types and the mix function easier to find.

Rendered documentation for the `art` crate with the re-exports on the front page
Figure 14-4: The front page of the documentation for art that lists the re-exports

The art crate users can still see and use the internal structure from Listing 14-3 as demonstrated in Listing 14-4, or they can use the more convenient structure in Listing 14-5, as shown in Listing 14-6:

Cargo doc 为此 crate 生成的 API 文档现在将在首页上列出并链接重新导出，如图 14-4 所示，使 PrimaryColor 和 secondaryColor 类型以及 mix 函数更容易找到。

“art”箱子的渲染文档，并在首页上重新导出
图 14-4：列出再出口的艺术品文档的首页

art crate 用户仍然可以看到并使用清单 14-3 中的内部结构，如清单 14-4 所示，或者他们可以使用清单 14-5 中更方便的结构，如清单 14-6 所示：

Filename: src/main.rs
``````
use art::mix;
use art::PrimaryColor;

fn main() {
    // --snip--
}
``````
Listing 14-6: A program using the re-exported items from the art crate

In cases where there are many nested modules, re-exporting the types at the top level with pub use can make a significant difference in the experience of people who use the crate. Another common use of pub use is to re-export definitions of a dependency in the current crate to make that crate's definitions part of your crate’s public API.

Creating a useful public API structure is more of an art than a science, and you can iterate to find the API that works best for your users. Choosing pub use gives you flexibility in how you structure your crate internally and decouples that internal structure from what you present to your users. Look at some of the code of crates you’ve installed to see if their internal structure differs from their public API.

在有许多嵌套模块的情况下，使用 pub use 在顶层重新导出类型可以对使用 crate 的人的体验产生显着影响。 pub use 的另一个常见用途是重新导出当前 crate 中依赖项的定义，以使该 crate 的定义成为 crate 公共 API 的一部分。

创建有用的公共 API 结构与其说是一门科学，不如说是一门艺术，您可以迭代找到最适合您的用户的 API。 选择 pub 使用可以让您灵活地在内部构建您的 crate，并将内部结构与您向用户呈现的内容分离。 查看您安装的 crate 的一些代码，看看它们的内部结构是否与其公共 API 不同。

### Setting Up a Crates.io Account
Before you can publish any crates, you need to create an account on crates.io and get an API token. To do so, visit the home page at crates.io and log in via a GitHub account. (The GitHub account is currently a requirement, but the site might support other ways of creating an account in the future.) Once you’re logged in, visit your account settings at https://crates.io/me/ and retrieve your API key. Then run the cargo login command with your API key, like this:

在发布任何 crate 之前，您需要在 crates.io 上创建一个帐户并获取 API 令牌。 为此，请访问 crates.io 主页并通过 GitHub 帐户登录。 （目前需要 GitHub 帐户，但该网站将来可能支持其他创建帐户的方式。）登录后，请访问您的帐户设置（网址为 https://crates.io/me/）并检索您的帐户 API 密钥。 然后使用您的 API 密钥运行货物登录命令，如下所示：

``````
$ cargo login abcdefghijklmnopqrstuvwxyz012345
``````
This command will inform Cargo of your API token and store it locally in ~/.cargo/credentials. Note that this token is a secret: do not share it with anyone else. If you do share it with anyone for any reason, you should revoke it and generate a new token on crates.io.

此命令将通知 Cargo 您的 API 令牌并将其本地存储在 ~/.cargo/credentials 中。 请注意，此令牌是一个秘密：请勿与其他任何人共享。 如果您出于任何原因与任何人共享它，您应该撤销它并在 crates.io 上生成一个新令牌。

### Adding Metadata to a New Crate
Let’s say you have a crate you want to publish. Before publishing, you’ll need to add some metadata in the [package] section of the crate’s Cargo.toml file.

Your crate will need a unique name. While you’re working on a crate locally, you can name a crate whatever you’d like. However, crate names on crates.io are allocated on a first-come, first-served basis. Once a crate name is taken, no one else can publish a crate with that name. Before attempting to publish a crate, search for the name you want to use. If the name has been used, you will need to find another name and edit the name field in the Cargo.toml file under the [package] section to use the new name for publishing, like so:

假设您有一个想要发布的箱子。 在发布之前，您需要在包的 Cargo.toml 文件的 [package] 部分添加一些元数据。

您的板条箱需要一个唯一的名称。 当您在本地处理板条箱时，您可以根据自己的喜好命名板条箱。 然而，crates.io 上的板条箱名称是按照先到先得的原则分配的。 一旦使用了 crate 名称，其他人就无法发布具有该名称的 crate。 在尝试发布 crate 之前，请搜索您要使用的名称。 如果该名称已被使用，您需要找到另一个名称并编辑 Cargo.toml 文件中 [package] 部分下的名称字段，以使用新名称进行发布，如下所示：

Filename: Cargo.toml
``````
[package]
name = "guessing_game"
``````
Even if you’ve chosen a unique name, when you run cargo publish to publish the crate at this point, you’ll get a warning and then an error:
``````
$ cargo publish
    Updating crates.io index
warning: manifest has no description, license, license-file, documentation, homepage or repository.
See https://doc.rust-lang.org/cargo/reference/manifest.html#package-metadata for more info.
--snip--
error: failed to publish to registry at https://crates.io

Caused by:
  the remote server responded with an error: missing or empty metadata fields: description, license. Please see https://doc.rust-lang.org/cargo/reference/manifest.html for how to upload metadata
``````
This errors because you’re missing some crucial information: a description and license are required so people will know what your crate does and under what terms they can use it. In Cargo.toml, add a description that's just a sentence or two, because it will appear with your crate in search results. For the license field, you need to give a license identifier value. The Linux Foundation’s Software Package Data Exchange (SPDX) lists the identifiers you can use for this value. For example, to specify that you’ve licensed your crate using the MIT License, add the MIT identifier:

出现此错误是因为您缺少一些关键信息：需要描述和许可证，以便人们知道您的板条箱的用途以及可以在什么条款下使用它。 在 Cargo.toml 中，添加一两句话的描述，因为它会与您的箱子一起出现在搜索结果中。 对于许可证字段，您需要提供许可证标识符值。 Linux 基金会的软件包数据交换 (SPDX) 列出了可用于该值的标识符。 例如，要指定您已使用 MIT 许可证对您的板条箱进行许可，请添加 MIT 标识符：

Filename: Cargo.toml
``````
[package]
name = "guessing_game"
license = "MIT"
``````
If you want to use a license that doesn’t appear in the SPDX, you need to place the text of that license in a file, include the file in your project, and then use license-file to specify the name of that file instead of using the license key.

Guidance on which license is appropriate for your project is beyond the scope of this book. Many people in the Rust community license their projects in the same way as Rust by using a dual license of MIT OR Apache-2.0. This practice demonstrates that you can also specify multiple license identifiers separated by OR to have multiple licenses for your project.

With a unique name, the version, your description, and a license added, the Cargo.toml file for a project that is ready to publish might look like this:

如果您想使用 SPDX 中未出现的许可证，则需要将该许可证的文本放入文件中，将该文件包含在您的项目中，然后使用许可证文件来指定该文件的名称 使用许可证密钥。

关于哪种许可证适合您的项目的指导超出了本书的范围。 Rust 社区中的许多人使用 MIT 或 Apache-2.0 的双重许可证，以与 Rust 相同的方式许可他们的项目。 此实践表明，您还可以指定由 OR 分隔的多个许可证标识符，以便为您的项目拥有多个许可证。

使用唯一的名称、版本、描述和添加的许可证后，准备发布的项目的 Cargo.toml 文件可能如下所示：

Filename: Cargo.toml
``````
[package]
name = "guessing_game"
version = "0.1.0"
edition = "2021"
description = "A fun game where you guess what number the computer has chosen."
license = "MIT OR Apache-2.0"

[dependencies]
``````
Cargo’s documentation describes other metadata you can specify to ensure others can discover and use your crate more easily.

Cargo 的文档描述了您可以指定的其他元数据，以确保其他人可以更轻松地发现和使用您的 crate。

### Publishing to Crates.io
Now that you’ve created an account, saved your API token, chosen a name for your crate, and specified the required metadata, you’re ready to publish! Publishing a crate uploads a specific version to crates.io for others to use.

Be careful, because a publish is permanent. The version can never be overwritten, and the code cannot be deleted. One major goal of crates.io is to act as a permanent archive of code so that builds of all projects that depend on crates from crates.io will continue to work. Allowing version deletions would make fulfilling that goal impossible. However, there is no limit to the number of crate versions you can publish.

现在您已经创建了一个帐户，保存了 API 令牌，为您的包选择了名称，并指定了所需的元数据，您就可以发布了！ 发布 crate 会将特定版本上传到 crates.io 供其他人使用。

请小心，因为发布是永久性的。 版本永远无法被覆盖，代码也无法被删除。 crates.io 的一个主要目标是充当代码的永久存档，以便依赖 crates.io 的 crate 的所有项目的构建都将继续工作。 允许版本删除将使实现该目标变得不可能。 但是，您可以发布的 crate 版本数量没有限制。

Run the cargo publish command again. It should succeed now:
``````
$ cargo publish
    Updating crates.io index
   Packaging guessing_game v0.1.0 (file:///projects/guessing_game)
   Verifying guessing_game v0.1.0 (file:///projects/guessing_game)
   Compiling guessing_game v0.1.0
(file:///projects/guessing_game/target/package/guessing_game-0.1.0)
    Finished dev [unoptimized + debuginfo] target(s) in 0.19s
   Uploading guessing_game v0.1.0 (file:///projects/guessing_game)

``````
Congratulations! You’ve now shared your code with the Rust community, and anyone can easily add your crate as a dependency of their project.

### Publishing a New Version of an Existing Crate
When you’ve made changes to your crate and are ready to release a new version, you change the version value specified in your Cargo.toml file and republish. Use the Semantic Versioning rules to decide what an appropriate next version number is based on the kinds of changes you’ve made. Then run cargo publish to upload the new version.

当您对 crate 进行更改并准备发布新版本时，您可以更改 Cargo.toml 文件中指定的版本值并重新发布。 使用语义版本控制规则根据您所做的更改类型来决定合适的下一个版本号。 然后运行CargoPublish来上传新版本。

### Deprecating Versions from Crates.io with cargo yank
Although you can’t remove previous versions of a crate, you can prevent any future projects from adding them as a new dependency. This is useful when a crate version is broken for one reason or another. In such situations, Cargo supports yanking a crate version.

Yanking a version prevents new projects from depending on that version while allowing all existing projects that depend on it to continue. Essentially, a yank means that all projects with a Cargo.lock will not break, and any future Cargo.lock files generated will not use the yanked version.

To yank a version of a crate, in the directory of the crate that you’ve previously published, run cargo yank and specify which version you want to yank. For example, if we've published a crate named guessing_game version 1.0.1 and we want to yank it, in the project directory for guessing_game we'd run:

尽管您无法删除包的早期版本，但您可以防止任何未来的项目将它们添加为新的依赖项。 当板条箱版本由于某种原因被破坏时，这非常有用。 在这种情况下，Cargo 支持拉取 crate 版本。

拉取版本可防止新项目依赖于该版本，同时允许依赖于该版本的所有现有项目继续。 本质上，yank 意味着所有具有 Cargo.lock 的项目都不会中断，并且将来生成的任何 Cargo.lock 文件都不会使用 yank 版本。

要拉取 crate 的版本，请在您之前发布的 crate 的目录中运行 Cargo yank 并指定要拉取的版本。 例如，如果我们发布了一个名为guessing_game version 1.0.1的包，并且想要将其拉出，则在guessing_game的项目目录中运行：

``````
$ cargo yank --vers 1.0.1
    Updating crates.io index
        Yank guessing_game@1.0.1
``````
By adding --undo to the command, you can also undo a yank and allow projects to start depending on a version again:
``````
$ cargo yank --vers 1.0.1 --undo
    Updating crates.io index
      Unyank guessing_game@1.0.1
``````
A yank does not delete any code. It cannot, for example, delete accidentally uploaded secrets. If that happens, you must reset those secrets immediately.

猛拉不会删除任何代码。 例如，它无法删除意外上传的机密。 如果发生这种情况，您必须立即重置这些机密。

## 14.3 Cargo Workspaces
In Chapter 12, we built a package that included a binary crate and a library crate. As your project develops, you might find that the library crate continues to get bigger and you want to split your package further into multiple library crates. Cargo offers a feature called workspaces that can help manage multiple related packages that are developed in tandem.

在第 12 章中，我们构建了一个包含二进制 crate 和库 crate 的包。 随着项目的发展，您可能会发现库 crate 不断变大，并且您希望将包进一步拆分为多个库 crate。 Cargo 提供了一个称为工作区的功能，可以帮助管理串联开发的多个相关包。

### 创建一个workspace
A workspace is a set of packages that share the same Cargo.lock and output directory. Let’s make a project using a workspace—we’ll use trivial code so we can concentrate on the structure of the workspace. There are multiple ways to structure a workspace, so we'll just show one common way. We’ll have a workspace containing a binary and two libraries. The binary, which will provide the main functionality, will depend on the two libraries. One library will provide an add_one function, and a second library an add_two function. These three crates will be part of the same workspace. We’ll start by creating a new directory for the workspace:

工作空间是一组共享相同 Cargo.lock 和输出目录的包。 让我们使用工作区创建一个项目 - 我们将使用简单的代码，这样我们就可以专注于工作区的结构。 构建工作区的方法有多种，因此我们仅展示一种常见的方法。 我们将有一个包含二进制文件和两个库的工作区。 提供主要功能的二进制文件将依赖于这两个库。 第一个库将提供 add_one 函数，第二个库将提供 add_two 函数。 这三个板条箱将属于同一工作区。 我们首先为工作区创建一个新目录：

```
mkdir add
cd add

```
Next, in the add directory, we create the Cargo.toml file that will configure the entire workspace. This file won’t have a [package] section. Instead, it will start with a [workspace] section that will allow us to add members to the workspace by specifying the path to the package with our binary crate; in this case, that path is adder:

接下来，在 add 目录中，我们创建将配置整个工作区的 Cargo.toml 文件。 该文件不会有 [package] 部分。 相反，它将以 [workspace] 部分开头，该部分允许我们通过指定二进制板条箱的包路径来将成员添加到工作区； 在本例中，该路径是 adder：

Filename: Cargo.toml
```
[workspace]

members = [
    "adder",
]

```
Next, we’ll create the adder binary crate by running cargo new within the add directory:

```
$ cargo new adder
     Created binary (application) `adder` package

```
At this point, we can build the workspace by running cargo build. The files in your add directory should look like this:
```
├── Cargo.lock
├── Cargo.toml
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target

```
工作区在顶层有一个目标目录，已编译的工件将被放置到其中； 加法器包没有自己的目标目录。 即使我们要从 adder 目录内部运行 Cargo build，编译后的工件仍然会出现在 add/target 中，而不是 add/adder/target 中。 Cargo 像这样在工作区中构建目标目录，因为工作区中的 crate 意味着相互依赖。 如果每个 crate 都有自己的目标目录，则每个 crate 都必须重新编译工作区中的其他每个 crate，以将工件放入其自己的目标目录中。 通过共享一个目标目录，包可以避免不必要的重建。
### 在空间章创建第二包

Next, let’s create another member package in the workspace and call it add_one. Change the top-level Cargo.toml to specify the add_one path in the members list:

Filename: Cargo.toml
```
[workspace]

members = [
    "adder",
    "add_one",
]

```
Then generate a new library crate named add_one:
```
$ cargo new add_one --lib
     Created library `add_one` package

```
Your add directory should now have these directories and files:
```
├── Cargo.lock
├── Cargo.toml
├── add_one
│   ├── Cargo.toml
│   └── src
│       └── lib.rs
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target

```
In the add_one/src/lib.rs file, let’s add an add_one function:

Filename: add_one/src/lib.rs
```
pub fn add_one(x: i32) -> i32 {
    x + 1
}
```
Now we can have the adder package with our binary depend on the add_one package that has our library. First, we’ll need to add a path dependency on add_one to adder/Cargo.toml.

Filename: adder/Cargo.toml
```
[dependencies]
add_one = { path = "../add_one" }

```
Cargo doesn’t assume that crates in a workspace will depend on each other, so we need to be explicit about the dependency relationships.

Next, let’s use the add_one function (from the add_one crate) in the adder crate. Open the adder/src/main.rs file and add a use line at the top to bring the new add_one library crate into scope. Then change the main function to call the add_one function, as in Listing 14-7.

Filename: adder/src/main.rs
```
use add_one;

fn main() {
    let num = 10;
    println!("Hello, world! {num} plus one is {}!", add_one::add_one(num));
}
```
Listing 14-7: Using the add_one library crate from the adder crate

Let’s build the workspace by running cargo build in the top-level add directory!

```
$ cargo build
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 0.68s

```
To run the binary crate from the add directory, we can specify which package in the workspace we want to run by using the -p argument and the package name with cargo run:

```
$ cargo run -p adder
    Finished dev [unoptimized + debuginfo] target(s) in 0.0s
     Running `target/debug/adder`
Hello, world! 10 plus one is 11!

```
This runs the code in adder/src/main.rs, which depends on the add_one crate.

### 在workspace中依赖额外的包
Notice that the workspace has only one Cargo.lock file at the top level, rather than having a Cargo.lock in each crate’s directory. This ensures that all crates are using the same version of all dependencies. If we add the rand package to the adder/Cargo.toml and add_one/Cargo.toml files, Cargo will resolve both of those to one version of rand and record that in the one Cargo.lock. Making all crates in the workspace use the same dependencies means the crates will always be compatible with each other. Let’s add the rand crate to the [dependencies] section in the add_one/Cargo.toml file so we can use the rand crate in the add_one crate:

请注意，工作区在顶层只有一个 Cargo.lock 文件，而不是在每个 crate 的目录中都有一个 Cargo.lock 文件。 这可确保所有包都使用所有依赖项的相同版本。 如果我们将 rand 包添加到 adder/Cargo.toml 和 add_one/Cargo.toml 文件中，Cargo 会将这两个文件解析为一种版本的 rand 并将其记录在一个 Cargo.lock 中。 使工作区中的所有 crate 使用相同的依赖项意味着 crate 将始终相互兼容。 让我们将 rand 箱添加到 add_one/Cargo.toml 文件中的 [dependencies] 部分，以便我们可以在 add_one 箱中使用 rand 箱：

Filename: add_one/Cargo.toml
```
[dependencies]
rand = "0.8.5"

```
We can now add use rand; to the add_one/src/lib.rs file, and building the whole workspace by running cargo build in the add directory will bring in and compile the rand crate. We will get one warning because we aren’t referring to the rand we brought into scope:

我们现在可以添加 use rand; 添加到 add_one/src/lib.rs 文件，并通过在 add 目录中运行 Cargo build 来构建整个工作区，将引入并编译 rand 箱。 我们将收到一个警告，因为我们没有引用我们纳入范围的兰特：

```
$ cargo build
    Updating crates.io index
  Downloaded rand v0.8.5
   --snip--
   Compiling rand v0.8.5
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
warning: unused import: `rand`
 --> add_one/src/lib.rs:1:5
  |
1 | use rand;
  |     ^^^^
  |
  = note: `#[warn(unused_imports)]` on by default

warning: `add_one` (lib) generated 1 warning
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 10.18s

```
The top-level Cargo.lock now contains information about the dependency of add_one on rand. However, even though rand is used somewhere in the workspace, we can’t use it in other crates in the workspace unless we add rand to their Cargo.toml files as well. For example, if we add use rand; to the adder/src/main.rs file for the adder package, we’ll get an error:

顶级 Cargo.lock 现在包含有关 add_one 对 rand 的依赖性的信息。 然而，即使 rand 在工作区的某个地方使用，我们也不能在工作区的其他 crate 中使用它，除非我们也将 rand 添加到它们的 Cargo.toml 文件中。 例如，如果我们添加 use rand; 添加到 adder 包的 adder/src/main.rs 文件中，我们会得到一个错误：

```
$ cargo build
  --snip--
   Compiling adder v0.1.0 (file:///projects/add/adder)
error[E0432]: unresolved import `rand`
 --> adder/src/main.rs:2:5
  |
2 | use rand;
  |     ^^^^ no external crate `rand`

```
To fix this, edit the Cargo.toml file for the adder package and indicate that rand is a dependency for it as well. Building the adder package will add rand to the list of dependencies for adder in Cargo.lock, but no additional copies of rand will be downloaded. Cargo has ensured that every crate in every package in the workspace using the rand package will be using the same version, saving us space and ensuring that the crates in the workspace will be compatible with each other.

要解决此问题，请编辑加法器包的 Cargo.toml 文件，并指示 rand 也是它的依赖项。 构建 adder 包会将 rand 添加到 Cargo.lock 中 adder 的依赖项列表中，但不会下载 rand 的其他副本。 Cargo 确保工作区中使用 rand 包的每个包中的每个 crate 将使用相同的版本，从而节省我们的空间并确保工作区中的 crate 彼此兼容。
#### 添加测试
For another enhancement, let’s add a test of the add_one::add_one function within the add_one crate:

Filename: add_one/src/lib.rs

```
pub fn add_one(x: i32) -> i32 {
    x + 1
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_works() {
        assert_eq!(3, add_one(2));
    }
}
```
Now run cargo test in the top-level add directory. Running cargo test in a workspace structured like this one will run the tests for all the crates in the workspace:

```
$ cargo test
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.27s
     Running unittests src/lib.rs (target/debug/deps/add_one-f0253159197f7841)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running unittests src/main.rs (target/debug/deps/adder-49979ff40686fa8e)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

```
The first section of the output shows that the it_works test in the add_one crate passed. The next section shows that zero tests were found in the adder crate, and then the last section shows zero documentation tests were found in the add_one crate.

We can also run tests for one particular crate in a workspace from the top-level directory by using the -p flag and specifying the name of the crate we want to test:

输出的第一部分显示 add_one 箱中的 it_works 测试已通过。 下一部分显示在 adder 箱中找到零个测试，然后最后一部分显示在 add_one 箱中找到零个文档测试。

我们还可以通过使用 -p 标志并指定我们要测试的包的名称，从顶级目录中对工作区中的一个特定包运行测试：

```
$ cargo test -p add_one
    Finished test [unoptimized + debuginfo] target(s) in 0.00s
     Running unittests src/lib.rs (target/debug/deps/add_one-b3235fea9a156f74)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

```
This output shows cargo test only ran the tests for the add_one crate and didn’t run the adder crate tests.

If you publish the crates in the workspace to crates.io, each crate in the workspace will need to be published separately. Like cargo test, we can publish a particular crate in our workspace by using the -p flag and specifying the name of the crate we want to publish.

For additional practice, add an add_two crate to this workspace in a similar way as the add_one crate!

As your project grows, consider using a workspace: it’s easier to understand smaller, individual components than one big blob of code. Furthermore, keeping the crates in a workspace can make coordination between crates easier if they are often changed at the same time.

此输出显示，cargo test 仅运行了 add_one crate 的测试，而没有运行 adder crate 的测试。

如果您将工作区中的 crate 发布到 crates.io，则工作区中的每个 crate 都需要单独发布。 与货物测试一样，我们可以通过使用 -p 标志并指定我们要发布的包的名称来在工作区中发布特定的包。

如需额外练习，请以与 add_one 箱类似的方式将 add_two 箱添加到此工作区！

随着项目的增长，请考虑使用工作区：与一大堆代码相比，更小的单个组件更容易理解。 此外，如果板条箱经常同时更改，则将板条箱保留在工作空间中可以使板条箱之间的协调变得更容易。
## 14.4 Install Binaries from Crates.io with cargo install
The cargo install command allows you to install and use binary crates locally. This isn’t intended to replace system packages; it’s meant to be a convenient way for Rust developers to install tools that others have shared on crates.io. Note that you can only install packages that have binary targets. A binary target is the runnable program that is created if the crate has a src/main.rs file or another file specified as a binary, as opposed to a library target that isn’t runnable on its own but is suitable for including within other programs. Usually, crates have information in the README file about whether a crate is a library, has a binary target, or both.

All binaries installed with cargo install are stored in the installation root’s bin folder. If you installed Rust using rustup.rs and don’t have any custom configurations, this directory will be $HOME/.cargo/bin. Ensure that directory is in your $PATH to be able to run programs you’ve installed with cargo install.

For example, in Chapter 12 we mentioned that there’s a Rust implementation of the grep tool called ripgrep for searching files. To install ripgrep, we can run the following:

Cargo install 命令允许您在本地安装和使用二进制包。 这并不是为了取代系统包； 它旨在为 Rust 开发人员提供一种便捷的方式来安装其他人在 crates.io 上共享的工具。 请注意，您只能安装具有二进制目标的软件包。 二进制目标是在包具有 src/main.rs 文件或指定为二进制的其他文件时创建的可运行程序，而不是单独运行但适合包含在其他文件中的库目标。 程式。 通常，包的 README 文件中包含有关包是否是库、是否具有二进制目标或两者的信息。

使用 Cargo install 安装的所有二进制文件都存储在安装根目录的 bin 文件夹中。 如果您使用 rustup.rs 安装了 Rust 并且没有任何自定义配置，则此目录将为 $HOME/.cargo/bin。 确保该目录位于您的 $PATH 中，以便能够运行通过 Cargo install 安装的程序。

例如，在第 12 章中我们提到有一个名为 ripgrep 的 grep 工具的 Rust 实现，用于搜索文件。 要安装 ripgrep，我们可以运行以下命令：

``````
$ cargo install ripgrep
    Updating crates.io index
  Downloaded ripgrep v13.0.0
  Downloaded 1 crate (243.3 KB) in 0.88s
  Installing ripgrep v13.0.0
--snip--
   Compiling ripgrep v13.0.0
    Finished release [optimized + debuginfo] target(s) in 3m 10s
  Installing ~/.cargo/bin/rg
   Installed package `ripgrep v13.0.0` (executable `rg`)
``````
The second-to-last line of the output shows the location and the name of the installed binary, which in the case of ripgrep is rg. As long as the installation directory is in your $PATH, as mentioned previously, you can then run rg --help and start using a faster, rustier tool for searching files!

输出的倒数第二行显示已安装的二进制文件的位置和名称，在 ripgrep 的情况下为 rg。 只要安装目录位于您的 $PATH 中，如前所述，您就可以运行 rg --help 并开始使用更快、更可靠的工具来搜索文件！

## 14.5 Extending Cargo with Custom Commonds
Cargo is designed so you can extend it with new subcommands without having to modify Cargo. If a binary in your $PATH is named cargo-something, you can run it as if it was a Cargo subcommand by running cargo something. Custom commands like this are also listed when you run cargo --list. Being able to use cargo install to install extensions and then run them just like the built-in Cargo tools is a super convenient benefit of Cargo’s design!

Cargo 的设计使您可以使用新的子命令对其进行扩展，而无需修改 Cargo。 如果 $PATH 中的二进制文件名为 Cargo-something，则可以通过运行 Cargo Something 来运行它，就好像它是 Cargo 子命令一样。 当您运行 Cargo --list 时，也会列出这样的自定义命令。 能够使用 Cargo install 来安装扩展，然后像内置 Cargo 工具一样运行它们，这是 Cargo 设计的一个超级方便的好处！

### 14.6 Summary
Sharing code with Cargo and crates.io is part of what makes the Rust ecosystem useful for many different tasks. Rust’s standard library is small and stable, but crates are easy to share, use, and improve on a timeline different from that of the language. Don’t be shy about sharing code that’s useful to you on crates.io; it’s likely that it will be useful to someone else as well!

与 Cargo 和 crates.io 共享代码是 Rust 生态系统对许多不同任务有用的一部分。 Rust 的标准库小而稳定，但 crate 很容易在与该语言不同的时间线上共享、使用和改进。 不要羞于在 crates.io 上分享对你有用的代码； 它很可能对其他人也有用！