# 写自动测试(Writing Automated Tests)
In his 1972 essay “The Humble Programmer,” Edsger W. Dijkstra said that “Program testing can be a very effective way to show the presence of bugs, but it is hopelessly inadequate for showing their absence.” That doesn’t mean we shouldn’t try to test as much as we can!

Correctness in our programs is the extent to which our code does what we intend it to do. Rust is designed with a high degree of concern about the correctness of programs, but correctness is complex and not easy to prove. Rust’s type system shoulders a huge part of this burden, but the type system cannot catch everything. As such, Rust includes support for writing automated software tests.

Say we write a function add_two that adds 2 to whatever number is passed to it. This function’s signature accepts an integer as a parameter and returns an integer as a result. When we implement and compile that function, Rust does all the type checking and borrow checking that you’ve learned so far to ensure that, for instance, we aren’t passing a String value or an invalid reference to this function. But Rust can’t check that this function will do precisely what we intend, which is return the parameter plus 2 rather than, say, the parameter plus 10 or the parameter minus 50! That’s where tests come in.

We can write tests that assert, for example, that when we pass 3 to the add_two function, the returned value is 5. We can run these tests whenever we make changes to our code to make sure any existing correct behavior has not changed.

Testing is a complex skill: although we can’t cover every detail about how to write good tests in one chapter, we’ll discuss the mechanics of Rust’s testing facilities. We’ll talk about the annotations and macros available to you when writing your tests, the default behavior and options provided for running your tests, and how to organize tests into unit tests and integration tests.

Edsger W. Dijkstra 在他 1972 年的文章“谦虚的程序员”中说，“程序测试可以是显示错误存在的非常有效的方法，但它完全不足以显示错误的存在。” 这并不意味着我们不应该尝试尽可能多地进行测试！

程序的正确性是指我们的代码执行我们想要执行的操作的程度。 Rust 的设计高度关注程序的正确性，但正确性很复杂且不易证明。 Rust 的类型系统承担了这一负担的很大一部分，但类型系统无法捕获所有内容。 因此，Rust 支持编写自动化软件测试。

假设我们编写了一个函数 add_two，它将 2 与传递给它的任何数字相加。 该函数的签名接受一个整数作为参数并返回一个整数作为结果。 当我们实现和编译该函数时，Rust 会执行您迄今为止学到的所有类型检查和借用检查，以确保我们不会传递 String 值或对此函数的无效引用。 但是 Rust 无法检查这个函数是否会精确地执行我们想要的操作，即返回参数加 2，而不是参数加 10 或参数减 50！ 这就是测试的用武之地。

我们可以编写测试来断言，例如，当我们将 3 传递给 add_two 函数时，返回值为 5。每当我们更改代码时，我们都可以运行这些测试，以确保任何现有的正确行为没有改变。

测试是一项复杂的技能：虽然我们无法在一章中涵盖如何编写良好测试的所有细节，但我们将讨论 Rust 测试工具的机制。 我们将讨论编写测试时可用的注释和宏、为运行测试提供的默认行为和选项，以及如何将测试组织为单元测试和集成测试。

## 11.1 如何写测试
Tests are Rust functions that verify that the non-test code is functioning in the expected manner. The bodies of test functions typically perform these three actions:

1. Set up any needed data or state.
2. Run the code you want to test.
3. Assert the results are what you expect.

Let’s look at the features Rust provides specifically for writing tests that take these actions, which include the test attribute, a few macros, and the should_panic attribute.

测试是 Rust 函数，用于验证非测试代码是否按预期方式运行。 测试函数体通常执行以下三个操作：

1. 设置任何需要的数据或状态。
2. 运行您想要测试的代码。
3. 断言结果是您所期望的。

让我们看一下 Rust 专门为编写执行这些操作的测试而提供的功能，其中包括 test 属性、一些宏和 should_panic 属性。

### 测试函数的剖析
最简单的是，Rust 中的测试是一个用 test 属性注释的函数。 属性是有关 Rust 代码片段的元数据； 一个例子是我们在第 5 章中与结构一起使用的 derive 属性。要将函数更改为测试函数，请在 fn 之前的行中添加 #[test]。 当您使用 Cargo test 命令运行测试时，Rust 会构建一个测试运行程序二进制文件，该二进制文件运行带注释的函数并报告每个测试函数是否通过或失败。

每当我们使用 Cargo 创建一个新的库项目时，都会自动为我们生成一个包含测试函数的测试模块。 该模块为您提供了一个用于编写测试的模板，因此您不必在每次开始新项目时查找确切的结构和语法。 您可以根据需要添加任意数量的附加测试功能和测试模块！

在实际测试任何代码之前，我们将通过试验模板测试来探索测试如何工作的某些方面。 然后我们将编写一些现实世界的测试，调用我们编写的一些代码并断言其行为是正确的。

让我们创建一个名为 adder 的新库项目，它将两个数字相加：

```
$ cargo new adder --lib
     Created library `adder` project
$ cd adder

```
The contents of the src/lib.rs file in your adder library should look like Listing 11-1.

Filename: src/lib.rs
```
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        let result = 2 + 2;
        assert_eq!(result, 4);
    }
}
```
Listing 11-1: The test module and function generated automatically by cargo new

For now, let’s ignore the top two lines and focus on the function. Note the #[test] annotation: this attribute indicates this is a test function, so the test runner knows to treat this function as a test. We might also have non-test functions in the tests module to help set up common scenarios or perform common operations, so we always need to indicate which functions are tests.

The example function body uses the assert_eq! macro to assert that result, which contains the result of adding 2 and 2, equals 4. This assertion serves as an example of the format for a typical test. Let’s run it to see that this test passes.

现在，让我们忽略最上面的两行，专注于函数。 请注意 #[test] 注释：此属性表明这是一个测试函数，因此测试运行者知道将此函数视为测试。 我们还可能在测试模块中包含非测试函数来帮助设置常见场景或执行常见操作，因此我们始终需要指示哪些函数是测试。

示例函数体使用了assert_eq! 宏来断言结果（包含 2 和 2 相加的结果）等于 4。该断言充当典型测试的格式示例。 让我们运行一下看看这个测试是否通过。

The cargo test command runs all tests in our project, as shown in Listing 11-2.

```
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.57s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s


```
Listing 11-2: The output from running the automatically generated test

Cargo compiled and ran the test. We see the line running 1 test. The next line shows the name of the generated test function, called it_works, and that the result of running that test is ok. The overall summary test result: ok. means that all the tests passed, and the portion that reads 1 passed; 0 failed totals the number of tests that passed or failed.

It’s possible to mark a test as ignored so it doesn’t run in a particular instance; we’ll cover that in the “Ignoring Some Tests Unless Specifically Requested” section later in this chapter. Because we haven’t done that here, the summary shows 0 ignored. We can also pass an argument to the cargo test command to run only tests whose name matches a string; this is called filtering and we’ll cover that in the “Running a Subset of Tests by Name” section. We also haven’t filtered the tests being run, so the end of the summary shows 0 filtered out.

The 0 measured statistic is for benchmark tests that measure performance. Benchmark tests are, as of this writing, only available in nightly Rust. See the documentation about benchmark tests to learn more.

The next part of the test output starting at Doc-tests adder is for the results of any documentation tests. We don’t have any documentation tests yet, but Rust can compile any code examples that appear in our API documentation. This feature helps keep your docs and your code in sync! We’ll discuss how to write documentation tests in the “Documentation Comments as Tests” section of Chapter 14. For now, we’ll ignore the Doc-tests output.

Let’s start to customize the test to our own needs. First change the name of the it_works function to a different name, such as exploration, like so:

Filename: src/lib.rs

清单 11-2：运行自动生成的测试的输出

Cargo 编译并运行了测试。 我们看到该行正在运行 1 个测试。 下一行显示生成的测试函数的名称，称为 it_works，并且运行该测试的结果是正常的。 总体总结测试结果：好的。 表示所有测试都通过了，读为1的部分通过了； 0 failed 是通过或失败的测试总数。

可以将测试标记为忽略，这样它就不会在特定实例中运行； 我们将在本章后面的“除非特别要求，否则忽略一些测试”部分中介绍这一点。 因为我们没有在这里这样做，所以摘要显示 0 被忽略。 我们还可以将参数传递给货物测试命令，以仅运行名称与字符串匹配的测试； 这称为过滤，我们将在“按名称运行测试子集”部分中介绍这一点。 我们还没有过滤正在运行的测试，因此摘要的末尾显示 0 被过滤掉。

0 测量统计数据用于衡量性能的基准测试。 截至撰写本文时，基准测试仅在夜间 Rust 中可用。 请参阅有关基准测试的文档以了解更多信息。

从 Doc-tests adder 开始的测试输出的下一部分是任何文档测试的结果。 我们还没有任何文档测试，但 Rust 可以编译 API 文档中出现的任何代码示例。 此功能有助于保持您的文档和代码同步！ 我们将在第 14 章的“文档注释作为测试”部分讨论如何编写文档测试。现在，我们将忽略 Doc-tests 输出。

让我们开始根据自己的需求定制测试。 首先将 it_works 函数的名称更改为不同的名称，例如exploration，如下所示：

文件名：src/lib.rs
```
#[cfg(test)]
mod tests {
    #[test]
    fn exploration() {
        assert_eq!(2 + 2, 4);
    }
}
```
Then run cargo test again. The output now shows exploration instead of it_works:

```
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.59s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 1 test
test tests::exploration ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s


```
Now we’ll add another test, but this time we’ll make a test that fails! Tests fail when something in the test function panics. Each test is run in a new thread, and when the main thread sees that a test thread has died, the test is marked as failed. In Chapter 9, we talked about how the simplest way to panic is to call the panic! macro. Enter the new test as a function named another, so your src/lib.rs file looks like Listing 11-3.

Filename: src/lib.rs
```
#[cfg(test)]
mod tests {
    #[test]
    fn exploration() {
        assert_eq!(2 + 2, 4);
    }

    #[test]
    fn another() {
        panic!("Make this test fail");
    }
}
```
Listing 11-3: Adding a second test that will fail because we call the panic! macro

Run the tests again using cargo test. The output should look like Listing 11-4, which shows that our exploration test passed and another failed.

```
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.72s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 2 tests
test tests::another ... FAILED
test tests::exploration ... ok

failures:

---- tests::another stdout ----
thread 'tests::another' panicked at 'Make this test fail', src/lib.rs:10:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::another

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass `--lib`

```
Listing 11-4: Test results when one test passes and one test fails

Instead of ok, the line test tests::another shows FAILED. Two new sections appear between the individual results and the summary: the first displays the detailed reason for each test failure. In this case, we get the details that another failed because it panicked at 'Make this test fail' on line 10 in the src/lib.rs file. The next section lists just the names of all the failing tests, which is useful when there are lots of tests and lots of detailed failing test output. We can use the name of a failing test to run just that test to more easily debug it; we’ll talk more about ways to run tests in the “Controlling How Tests Are Run” section.

The summary line displays at the end: overall, our test result is FAILED. We had one test pass and one test fail.

Now that you’ve seen what the test results look like in different scenarios, let’s look at some macros other than panic! that are useful in tests.

清单 11-4：一项测试通过而一项测试失败时的测试结果

test test::another 行显示 FAILED，而不是 ok。 各个结果和摘要之间出现两个新部分：第一个部分显示每个测试失败的详细原因。 在本例中，我们获得了另一个失败的详细信息，因为它在 src/lib.rs 文件中第 10 行的“使此测试失败”处出现恐慌。 下一部分仅列出所有失败测试的名称，这在存在大量测试和大量详细失败测试输出时非常有用。 我们可以使用失败测试的名称来运行该测试，以便更轻松地调试它； 我们将在“控制测试的运行方式”部分详细讨论运行测试的方法。

摘要行显示在最后：总的来说，我们的测试结果是“失败”。 我们有一项测试通过，一项测试失败。

现在您已经了解了不同场景下的测试结果，让我们看看除了panic之外的一些宏！ 这在测试中很有用。

### Checking Results with the assert! Macro

The assert! macro, provided by the standard library, is useful when you want to ensure that some condition in a test evaluates to true. We give the assert! macro an argument that evaluates to a Boolean. If the value is true, nothing happens and the test passes. If the value is false, the assert! macro calls panic! to cause the test to fail. Using the assert! macro helps us check that our code is functioning in the way we intend.

In Chapter 5, Listing 5-15, we used a Rectangle struct and a can_hold method, which are repeated here in Listing 11-5. Let’s put this code in the src/lib.rs file, then write some tests for it using the assert! macro.

Filename: src/lib.rs

断言！ 当您想要确保测试中的某些条件计算为 true 时，标准库提供的宏非常有用。 我们给出断言！ 宏是一个计算结果为布尔值的参数。 如果该值为 true，则不会发生任何事情并且测试通过。 如果值为 false，则断言！ 宏呼恐慌！ 导致测试失败。 使用断言！ 宏帮助我们检查代码是否按照我们预期的方式运行。

在第 5 章清单 5-15 中，我们使用了 Rectangle 结构体和 can_hold 方法，清单 11-5 中重复了这些方法。 让我们将此代码放在 src/lib.rs 文件中，然后使用断言为其编写一些测试！ 宏。

文件名：src/lib.rs

```
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```

Listing 11-5: Using the Rectangle struct and its can_hold method from Chapter 5

The can_hold method returns a Boolean, which means it’s a perfect use case for the assert! macro. In Listing 11-6, we write a test that exercises the can_hold method by creating a Rectangle instance that has a width of 8 and a height of 7 and asserting that it can hold another Rectangle instance that has a width of 5 and a height of 1.

Filename: src/lib.rs

```
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn larger_can_hold_smaller() {
        let larger = Rectangle {
            width: 8,
            height: 7,
        };
        let smaller = Rectangle {
            width: 5,
            height: 1,
        };

        assert!(larger.can_hold(&smaller));
    }
}
```
Listing 11-6: A test for can_hold that checks whether a larger rectangle can indeed hold a smaller rectangle

Note that we’ve added a new line inside the tests module: use super::*;. The tests module is a regular module that follows the usual visibility rules we covered in Chapter 7 in the “Paths for Referring to an Item in the Module Tree” section. Because the tests module is an inner module, we need to bring the code under test in the outer module into the scope of the inner module. We use a glob here so anything we define in the outer module is available to this tests module.

We’ve named our test larger_can_hold_smaller, and we’ve created the two Rectangle instances that we need. Then we called the assert! macro and passed it the result of calling larger.can_hold(&smaller). This expression is supposed to return true, so our test should pass. Let’s find out!

示例 11-6：can_hold 的测试，检查较大的矩形是否确实可以容纳较小的矩形

请注意，我们在测试模块中添加了一个新行：use super::*;。 测试模块是一个常规模块，遵循我们在第 7 章“引用模块树中项目的路径”部分中介绍的常见可见性规则。 因为tests模块是一个内部模块，所以我们需要将外部模块中的被测代码纳入到内部模块的作用域中。 我们在这里使用 glob，因此我们在外部模块中定义的任何内容都可供该测试模块使用。

我们将测试命名为large_can_hold_smaller，并创建了我们需要的两个 Rectangle 实例。 然后我们调用了断言！ 宏并将调用larger.can_hold(&smaller)的结果传递给它。 该表达式应该返回 true，因此我们的测试应该通过。 让我们来看看吧！

```
$ cargo test
   Compiling rectangle v0.1.0 (file:///projects/rectangle)
    Finished test [unoptimized + debuginfo] target(s) in 0.66s
     Running unittests src/lib.rs (target/debug/deps/rectangle-6584c4561e48942e)

running 1 test
test tests::larger_can_hold_smaller ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests rectangle

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s


```
It does pass! Let’s add another test, this time asserting that a smaller rectangle cannot hold a larger rectangle:

Filename: src/lib.rs

```
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn larger_can_hold_smaller() {
        // --snip--
    }

    #[test]
    fn smaller_cannot_hold_larger() {
        let larger = Rectangle {
            width: 8,
            height: 7,
        };
        let smaller = Rectangle {
            width: 5,
            height: 1,
        };

        assert!(!smaller.can_hold(&larger));
    }
}
```
Because the correct result of the can_hold function in this case is false, we need to negate that result before we pass it to the assert! macro. As a result, our test will pass if can_hold returns false:

```
$ cargo test
   Compiling rectangle v0.1.0 (file:///projects/rectangle)
    Finished test [unoptimized + debuginfo] target(s) in 0.66s
     Running unittests src/lib.rs (target/debug/deps/rectangle-6584c4561e48942e)

running 2 tests
test tests::larger_can_hold_smaller ... ok
test tests::smaller_cannot_hold_larger ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests rectangle

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s


```

Two tests that pass! Now let’s see what happens to our test results when we introduce a bug in our code. We’ll change the implementation of the can_hold method by replacing the greater-than sign with a less-than sign when it compares the widths:
```
// --snip--
impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width < other.width && self.height > other.height
    }
}

```
Running the tests now produces the following:

```
$ cargo test
   Compiling rectangle v0.1.0 (file:///projects/rectangle)
    Finished test [unoptimized + debuginfo] target(s) in 0.66s
     Running unittests src/lib.rs (target/debug/deps/rectangle-6584c4561e48942e)

running 2 tests
test tests::larger_can_hold_smaller ... FAILED
test tests::smaller_cannot_hold_larger ... ok

failures:

---- tests::larger_can_hold_smaller stdout ----
thread 'tests::larger_can_hold_smaller' panicked at 'assertion failed: larger.can_hold(&smaller)', src/lib.rs:28:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::larger_can_hold_smaller

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass `--lib`

```
Our tests caught the bug! Because larger.width is 8 and smaller.width is 5, the comparison of the widths in can_hold now returns false: 8 is not less than 5.

### Testing Equality with the assert_eq! and assert_ne! Macros

A common way to verify functionality is to test for equality between the result of the code under test and the value you expect the code to return. You could do this using the assert! macro and passing it an expression using the == operator. However, this is such a common test that the standard library provides a pair of macros—assert_eq! and assert_ne!—to perform this test more conveniently. These macros compare two arguments for equality or inequality, respectively. They’ll also print the two values if the assertion fails, which makes it easier to see why the test failed; conversely, the assert! macro only indicates that it got a false value for the == expression, without printing the values that led to the false value.

In Listing 11-7, we write a function named add_two that adds 2 to its parameter, then we test this function using the assert_eq! macro.

Filename: src/lib.rs

验证功能的常见方法是测试被测代码的结果与您期望代码返回的值之间是否相等。 您可以使用断言来做到这一点！ 宏并使用 == 运算符向其传递一个表达式。 然而，这是一个非常常见的测试，标准库提供了一对宏——assert_eq! 和assert_ne!——更方便地执行此测试。 这些宏分别比较两个参数的相等或不相等。 如果断言失败，他们还会打印这两个值，这使得更容易了解测试失败的原因； 反之，则断言！ 宏仅指示它得到 == 表达式的错误值，而不打印导致错误值的值。

在清单 11-7 中，我们编写了一个名为 add_two 的函数，该函数将 2 添加到其参数中，然后我们使用 assert_eq! 测试该函数。 宏。

文件名：src/lib.rs

```
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_adds_two() {
        assert_eq!(4, add_two(2));
    }
}
```
Listing 11-7: Testing the function add_two using the assert_eq! macro

Let’s check that it passes!
```
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.58s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 1 test
test tests::it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s


```
We pass 4 as the argument to assert_eq!, which is equal to the result of calling add_two(2). The line for this test is test tests::it_adds_two ... ok, and the ok text indicates that our test passed!

Let’s introduce a bug into our code to see what assert_eq! looks like when it fails. Change the implementation of the add_two function to instead add 3:

```
pub fn add_two(a: i32) -> i32 {
    a + 3
}

```
Run the tests again: 
```
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.61s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 1 test
test tests::it_adds_two ... FAILED

failures:

---- tests::it_adds_two stdout ----
thread 'tests::it_adds_two' panicked at 'assertion failed: `(left == right)`
  left: `4`,
 right: `5`', src/lib.rs:11:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::it_adds_two

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass `--lib`

```
Our test caught the bug! The it_adds_two test failed, and the message tells us that the assertion that fails was assertion failed: `(left == right)` and what the left and right values are. This message helps us start debugging: the left argument was 4 but the right argument, where we had add_two(2), was 5. You can imagine that this would be especially helpful when we have a lot of tests going on.

Note that in some languages and test frameworks, the parameters to equality assertion functions are called expected and actual, and the order in which we specify the arguments matters. However, in Rust, they’re called left and right, and the order in which we specify the value we expect and the value the code produces doesn’t matter. We could write the assertion in this test as assert_eq!(add_two(2), 4), which would result in the same failure message that displays assertion failed: `(left == right)`.

The assert_ne! macro will pass if the two values we give it are not equal and fail if they’re equal. This macro is most useful for cases when we’re not sure what a value will be, but we know what the value definitely shouldn’t be. For example, if we’re testing a function that is guaranteed to change its input in some way, but the way in which the input is changed depends on the day of the week that we run our tests, the best thing to assert might be that the output of the function is not equal to the input.

Under the surface, the assert_eq! and assert_ne! macros use the operators == and !=, respectively. When the assertions fail, these macros print their arguments using debug formatting, which means the values being compared must implement the PartialEq and Debug traits. All primitive types and most of the standard library types implement these traits. For structs and enums that you define yourself, you’ll need to implement PartialEq to assert equality of those types. You’ll also need to implement Debug to print the values when the assertion fails. Because both traits are derivable traits, as mentioned in Listing 5-12 in Chapter 5, this is usually as straightforward as adding the #[derive(PartialEq, Debug)] annotation to your struct or enum definition. See Appendix C, “Derivable Traits,” for more details about these and other derivable traits.

我们的测试发现了这个错误！ it_adds_two 测试失败，消息告诉我们失败的断言是断言失败：`(left == right)`以及左值和右值是什么。 这条消息帮助我们开始调试：左边的参数是 4，但右边的参数（我们有 add_two(2)）是 5。你可以想象，当我们正在进行大量测试时，这会特别有用。

请注意，在某些语言和测试框架中，相等断言函数的参数称为预期参数和实际参数，并且我们指定参数的顺序很重要。 然而，在 Rust 中，它们被称为左和右，我们指定期望值和代码生成值的顺序并不重要。 我们可以将此测试中的断言编写为assert_eq!(add_two(2), 4)，这将导致显示断言失败的相同失败消息：“(left == right)”。

断言_ne！ 如果我们给它的两个值不相等，则宏将通过；如果相等，则宏将失败。 当我们不确定某个值是什么，但我们知道该值绝对不应该是什么时，这个宏最有用。 例如，如果我们正在测试一个保证以某种方式更改其输入的函数，但更改输入的方式取决于我们运行测试的星期几，那么最好的断言可能是 函数的输出不等于输入。

在表面之下，assert_eq! 并断言！ 宏分别使用运算符 == 和 !=。 当断言失败时，这些宏使用调试格式打印它们的参数，这意味着要比较的值必须实现 PartialEq 和 Debug 特征。 所有原始类型和大多数标准库类型都实现这些特征。 对于您自己定义的结构体和枚举，您需要实现 PartialEq 来断言这些类型的相等性。 您还需要实现调试以在断言失败时打印值。 因为这两个特征都是可派生特征，如第 5 章中的清单 5-12 中所述，所以这通常就像将 #[derive(PartialEq, Debug)] 注释添加到结构或枚举定义中一样简单。 有关这些和其他可衍生特征的更多详细信息，请参阅附录 C“可衍生特征”。

### 添加自定义失败消息Adding Custom Failure Messages
You can also add a custom message to be printed with the failure message as optional arguments to the assert!, assert_eq!, and assert_ne! macros. Any arguments specified after the required arguments are passed along to the format! macro (discussed in Chapter 8 in the “Concatenation with the + Operator or the format! Macro” section), so you can pass a format string that contains {} placeholders and values to go in those placeholders. Custom messages are useful for documenting what an assertion means; when a test fails, you’ll have a better idea of what the problem is with the code.

For example, let’s say we have a function that greets people by name and we want to test that the name we pass into the function appears in the output:

您还可以添加要与失败消息一起打印的自定义消息，作为assert!、assert_eq! 和assert_ne! 的可选参数。 宏。 在必需参数之后指定的任何参数都将传递到格式！ 宏（在第 8 章“使用 + 运算符或格式连接！宏”部分中讨论），因此您可以传递包含 {} 占位符和要放入这些占位符中的值的格式字符串。 自定义消息对于记录断言的含义很有用； 当测试失败时，您将更好地了解代码的问题所在。

例如，假设我们有一个通过名字向人们打招呼的函数，我们想测试传递给函数的名字是否出现在输出中：

Filename: src/lib.rs
```
pub fn greeting(name: &str) -> String {
    format!("Hello {}!", name)
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn greeting_contains_name() {
        let result = greeting("Carol");
        assert!(result.contains("Carol"));
    }
}
```
该计划的要求尚未达成一致，我们非常确定问候语开头的“Hello”文本将会发生变化。 我们决定不想在需求发生变化时更新测试，因此我们只需断言输出包含输入参数的文本，而不是检查与greeting函数返回的值是否完全相等。

现在，让我们通过更改问候语以排除名称来引入一个错误到此代码中，以查看默认测试失败的情况：

```
pub fn greeting(name: &str) -> String {
    String::from("Hello!")
}

```
Running this test produces the following:
```
$ cargo test
   Compiling greeter v0.1.0 (file:///projects/greeter)
    Finished test [unoptimized + debuginfo] target(s) in 0.91s
     Running unittests src/lib.rs (target/debug/deps/greeter-170b942eb5bf5e3a)

running 1 test
test tests::greeting_contains_name ... FAILED

failures:

---- tests::greeting_contains_name stdout ----
thread 'tests::greeting_contains_name' panicked at 'assertion failed: result.contains(\"Carol\")', src/lib.rs:12:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::greeting_contains_name

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass `--lib`

```
This result just indicates that the assertion failed and which line the assertion is on. A more useful failure message would print the value from the greeting function. Let’s add a custom failure message composed of a format string with a placeholder filled in with the actual value we got from the greeting function:

```
    #[test]
    fn greeting_contains_name() {
        let result = greeting("Carol");
        assert!(
            result.contains("Carol"),
            "Greeting did not contain name, value was `{}`",
            result
        );
    }

```
Now when we run the test, we’ll get a more informative error message:

```
$ cargo test
   Compiling greeter v0.1.0 (file:///projects/greeter)
    Finished test [unoptimized + debuginfo] target(s) in 0.93s
     Running unittests src/lib.rs (target/debug/deps/greeter-170b942eb5bf5e3a)

running 1 test
test tests::greeting_contains_name ... FAILED

failures:

---- tests::greeting_contains_name stdout ----
thread 'tests::greeting_contains_name' panicked at 'Greeting did not contain name, value was `Hello!`', src/lib.rs:12:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::greeting_contains_name

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass `--lib`

```
We can see the value we actually got in the test output, which would help us debug what happened instead of what we were expecting to happen.

### Checking for Panics with should_panic
In addition to checking return values, it’s important to check that our code handles error conditions as we expect. For example, consider the Guess type that we created in Chapter 9, Listing 9-13. Other code that uses Guess depends on the guarantee that Guess instances will contain only values between 1 and 100. We can write a test that ensures that attempting to create a Guess instance with a value outside that range panics.

We do this by adding the attribute should_panic to our test function. The test passes if the code inside the function panics; the test fails if the code inside the function doesn’t panic.

Listing 11-8 shows a test that checks that the error conditions of Guess::new happen when we expect them to.

除了检查返回值之外，检查我们的代码是否按预期处理错误条件也很重要。 例如，考虑我们在第 9 章清单 9-13 中创建的 Guess 类型。 使用 Guess 的其他代码取决于 Guess 实例仅包含 1 到 100 之间的值的保证。我们可以编写一个测试来确保尝试创建具有超出该范围的值的 Guess 实例会发生恐慌。

我们通过将属性 should_panic 添加到我们的测试函数中来做到这一点。 如果函数内的代码发生恐慌，则测试通过； 如果函数内的代码没有出现恐慌，则测试失败。

清单 11-8 显示了一个测试，该测试检查 Guess::new 的错误条件是否在我们期望的时间发生。

Filename: src/lib.rs
```
pub struct Guess {
    value: i32,
}

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess value must be between 1 and 100, got {}.", value);
        }

        Guess { value }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic]
    fn greater_than_100() {
        Guess::new(200);
    }
}
```
Listing 11-8: Testing that a condition will cause a panic!

We place the #[should_panic] attribute after the #[test] attribute and before the test function it applies to. Let’s look at the result when this test passes:

```
$ cargo test
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished test [unoptimized + debuginfo] target(s) in 0.58s
     Running unittests src/lib.rs (target/debug/deps/guessing_game-57d70c3acb738f4d)

running 1 test
test tests::greater_than_100 - should panic ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests guessing_game

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s


```
Looks good! Now let’s introduce a bug in our code by removing the condition that the new function will panic if the value is greater than 100:

```
// --snip--
impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 {
            panic!("Guess value must be between 1 and 100, got {}.", value);
        }

        Guess { value }
    }
}

```
When we run the test in Listing 11-8, it will fail:

```
$ cargo test
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished test [unoptimized + debuginfo] target(s) in 0.62s
     Running unittests src/lib.rs (target/debug/deps/guessing_game-57d70c3acb738f4d)

running 1 test
test tests::greater_than_100 - should panic ... FAILED

failures:

---- tests::greater_than_100 stdout ----
note: test did not panic as expected

failures:
    tests::greater_than_100

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass `--lib`

```

We don’t get a very helpful message in this case, but when we look at the test function, we see that it’s annotated with #[should_panic]. The failure we got means that the code in the test function did not cause a panic.

Tests that use should_panic can be imprecise. A should_panic test would pass even if the test panics for a different reason from the one we were expecting. To make should_panic tests more precise, we can add an optional expected parameter to the should_panic attribute. The test harness will make sure that the failure message contains the provided text. For example, consider the modified code for Guess in Listing 11-9 where the new function panics with different messages depending on whether the value is too small or too large.

在这种情况下，我们没有得到非常有用的消息，但是当我们查看测试函数时，我们看到它带有#[should_panic]注释。 我们得到的失败意味着测试函数中的代码没有引起恐慌。

使用 should_panic 的测试可能不精确。 即使测试因与我们预期不同的原因而发生恐慌，should_panic 测试也会通过。 为了使should_panic测试更加精确，我们可以向should_panic属性添加一个可选的预期参数。 测试工具将确保失败消息包含提供的文本。 例如，考虑清单 11-9 中 Guess 的修改后的代码，其中新函数根据值是否太小或太大而发出不同的消息。

Filename: src/lib.rs

```
// --snip--

impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 {
            panic!(
                "Guess value must be greater than or equal to 1, got {}.",
                value
            );
        } else if value > 100 {
            panic!(
                "Guess value must be less than or equal to 100, got {}.",
                value
            );
        }

        Guess { value }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic(expected = "less than or equal to 100")]
    fn greater_than_100() {
        Guess::new(200);
    }
}
```
Listing 11-9: Testing for a panic! with a panic message containing a specified substring

This test will pass because the value we put in the should_panic attribute’s expected parameter is a substring of the message that the Guess::new function panics with. We could have specified the entire panic message that we expect, which in this case would be Guess value must be less than or equal to 100, got 200. What you choose to specify depends on how much of the panic message is unique or dynamic and how precise you want your test to be. In this case, a substring of the panic message is enough to ensure that the code in the test function executes the else if value > 100 case.

To see what happens when a should_panic test with an expected message fails, let’s again introduce a bug into our code by swapping the bodies of the if value < 1 and the else if value > 100 blocks:

该测试将通过，因为我们放入 should_panic 属性的预期参数中的值是 Guess::new 函数发生恐慌的消息的子字符串。 我们可以指定我们期望的整个紧急消息，在这种情况下，猜测值必须小于或等于 100，得到 200。您选择指定的内容取决于紧急消息中有多少是唯一的或动态的，以及 您希望测试有多精确。 在这种情况下，恐慌消息的子字符串足以确保测试函数中的代码执行 else if value > 100 的情况。

为了看看当带有预期消息的 should_panic 测试失败时会发生什么，让我们通过交换 if value < 1 和 else if value > 100 块的主体，再次在代码中引入一个错误：

```
        if value < 1 {
            panic!(
                "Guess value must be less than or equal to 100, got {}.",
                value
            );
        } else if value > 100 {
            panic!(
                "Guess value must be greater than or equal to 1, got {}.",
                value
            );
        }

```
This time when we run the should_panic test, it will fail:

```
$ cargo test
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished test [unoptimized + debuginfo] target(s) in 0.66s
     Running unittests src/lib.rs (target/debug/deps/guessing_game-57d70c3acb738f4d)

running 1 test
test tests::greater_than_100 - should panic ... FAILED

failures:

---- tests::greater_than_100 stdout ----
thread 'tests::greater_than_100' panicked at 'Guess value must be greater than or equal to 1, got 200.', src/lib.rs:13:13
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
note: panic did not contain expected string
      panic message: `"Guess value must be greater than or equal to 1, got 200."`,
 expected substring: `"less than or equal to 100"`

failures:
    tests::greater_than_100

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass `--lib`

```
The failure message indicates that this test did indeed panic as we expected, but the panic message did not include the expected string 'Guess value must be less than or equal to 100'. The panic message that we did get in this case was Guess value must be greater than or equal to 1, got 200. Now we can start figuring out where our bug is!

失败消息表明此测试确实按照我们的预期发生了恐慌，但恐慌消息不包含预期的字符串“猜测值必须小于或等于100”。 在这种情况下，我们确实收到的恐慌消息是猜测值必须大于或等于 1，得到 200。现在我们可以开始找出我们的错误在哪里！

### Using Result<T, E> in Tests

Our tests so far all panic when they fail. We can also write tests that use Result<T, E>! Here’s the test from Listing 11-1, rewritten to use Result<T, E> and return an Err instead of panicking:

```
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() -> Result<(), String> {
        if 2 + 2 == 4 {
            Ok(())
        } else {
            Err(String::from("two plus two does not equal four"))
        }
    }
}
```

The it_works function now has the Result<(), String> return type. In the body of the function, rather than calling the assert_eq! macro, we return Ok(()) when the test passes and an Err with a String inside when the test fails.

Writing tests so they return a Result<T, E> enables you to use the question mark operator in the body of tests, which can be a convenient way to write tests that should fail if any operation within them returns an Err variant.

You can’t use the #[should_panic] annotation on tests that use Result<T, E>. To assert that an operation returns an Err variant, don’t use the question mark operator on the Result<T, E> value. Instead, use assert!(value.is_err()).

Now that you know several ways to write tests, let’s look at what is happening when we run our tests and explore the different options we can use with cargo test.

it_works 函数现在具有 Result<(), String> 返回类型。 在函数体中，而不是调用assert_eq！ 宏，当测试通过时，我们返回 Ok(())；当测试失败时，我们返回一个带有字符串的 Err。

编写测试使其返回 Result<T, E> 使您能够在测试主体中使用问号运算符，这可以是编写测试的便捷方法，如果测试中的任何操作返回 Err 变体，这些测试就会失败。

您不能在使用 Result<T, E> 的测试中使用 #[should_panic] 注释。 要断言操作返回 Err 变体，请勿在 Result<T, E> 值上使用问号运算符。 相反，请使用断言！(value.is_err())。

现在您已经知道了编写测试的几种方法，让我们看看运行测试时发生的情况，并探索可以与货物测试一起使用的不同选项。


## 11.2 控制测试的运行方式
Just as cargo run compiles your code and then runs the resulting binary, cargo test compiles your code in test mode and runs the resulting test binary. The default behavior of the binary produced by cargo test is to run all the tests in parallel and capture output generated during test runs, preventing the output from being displayed and making it easier to read the output related to the test results. You can, however, specify command line options to change this default behavior.

Some command line options go to cargo test, and some go to the resulting test binary. To separate these two types of arguments, you list the arguments that go to cargo test followed by the separator -- and then the ones that go to the test binary. Running cargo test --help displays the options you can use with cargo test, and running cargo test -- --help displays the options you can use after the separator.

正如cargo run 编译您的代码然后运行生成的二进制文件一样，cargo test 在测试模式下编译您的代码并运行生成的测试二进制文件。 Cargo test 生成的二进制文件的默认行为是并行运行所有测试并捕获测试运行期间生成的输出，从而防止显示输出并使其更容易读取与测试结果相关的输出。 但是，您可以指定命令行选项来更改此默认行为。

一些命令行选项转到货物测试，一些选项转到生成的测试二进制文件。 要分隔这两种类型的参数，您可以列出进入 Cargo test 的参数，后跟分隔符，然后列出进入测试二进制文件的参数。 运行cargo test --help 显示可与cargo test 一起使用的选项，运行cargo test -- --help 显示可在分隔符后使用的选项。

### Running Tests in Parallel or Consecutively

When you run multiple tests, by default they run in parallel using threads, meaning they finish running faster and you get feedback quicker. Because the tests are running at the same time, you must make sure your tests don’t depend on each other or on any shared state, including a shared environment, such as the current working directory or environment variables.

For example, say each of your tests runs some code that creates a file on disk named test-output.txt and writes some data to that file. Then each test reads the data in that file and asserts that the file contains a particular value, which is different in each test. Because the tests run at the same time, one test might overwrite the file in the time between another test writing and reading the file. The second test will then fail, not because the code is incorrect but because the tests have interfered with each other while running in parallel. One solution is to make sure each test writes to a different file; another solution is to run the tests one at a time.

If you don’t want to run the tests in parallel or if you want more fine-grained control over the number of threads used, you can send the --test-threads flag and the number of threads you want to use to the test binary. Take a look at the following example:

当您运行多个测试时，默认情况下它们使用线程并行运行，这意味着它们更快地完成运行，并且您可以更快地获得反馈。 由于测试是同时运行的，因此您必须确保测试不相互依赖或不依赖于任何共享状态，包括共享环境，例如当前工作目录或环境变量。

例如，假设每个测试都运行一些代码，这些代码在磁盘上创建一个名为 test-output.txt 的文件，并将一些数据写入该文件。 然后，每个测试都会读取该文件中的数据，并断言该文件包含特定值，该值在每个测试中都不同。 由于测试是同时运行的，因此一个测试可能会在另一个测试写入和读取文件之间的时间内覆盖该文件。 第二个测试将失败，不是因为代码不正确，而是因为测试在并行运行时相互干扰。 一种解决方案是确保每个测试写入不同的文件； 另一种解决方案是一次运行一个测试。

如果您不想并行运行测试，或者希望对所使用的线程数进行更细粒度的控制，则可以发送 --test-threads 标志和要用于测试的线程数 二进制。 看一下下面的例子：

```
$ cargo test -- --test-threads=1

```
We set the number of test threads to 1, telling the program not to use any parallelism. Running the tests using one thread will take longer than running them in parallel, but the tests won’t interfere with each other if they share state.

### Showing Function Output

By default, if a test passes, Rust’s test library captures anything printed to standard output. For example, if we call println! in a test and the test passes, we won’t see the println! output in the terminal; we’ll see only the line that indicates the test passed. If a test fails, we’ll see whatever was printed to standard output with the rest of the failure message.

As an example, Listing 11-10 has a silly function that prints the value of its parameter and returns 10, as well as a test that passes and a test that fails.

默认情况下，如果测试通过，Rust 的测试库会捕获打印到标准输出的任何内容。 例如，如果我们调用 println! 在测试中并且测试通过，我们将不会看到 println! 在终端中输出； 我们只会看到表明测试已通过的行。 如果测试失败，我们将看到打印到标准输出的内容以及失败消息的其余部分。

例如，清单 11-10 有一个愚蠢的函数，它打印其参数的值并返回 10，以及一个通过的测试和一个失败的测试。

Filename: src/lib.rs

```
fn prints_and_returns_10(a: i32) -> i32 {
    println!("I got the value {}", a);
    10
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn this_test_will_pass() {
        let value = prints_and_returns_10(4);
        assert_eq!(10, value);
    }

    #[test]
    fn this_test_will_fail() {
        let value = prints_and_returns_10(8);
        assert_eq!(5, value);
    }
}
```

Listing 11-10: Tests for a function that calls println!

When we run these tests with cargo test, we’ll see the following output:

```
$ cargo test
   Compiling silly-function v0.1.0 (file:///projects/silly-function)
    Finished test [unoptimized + debuginfo] target(s) in 0.58s
     Running unittests src/lib.rs (target/debug/deps/silly_function-160869f38cff9166)

running 2 tests
test tests::this_test_will_fail ... FAILED
test tests::this_test_will_pass ... ok

failures:

---- tests::this_test_will_fail stdout ----
I got the value 8
thread 'tests::this_test_will_fail' panicked at 'assertion failed: `(left == right)`
  left: `5`,
 right: `10`', src/lib.rs:19:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::this_test_will_fail

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass `--lib`

```
Note that nowhere in this output do we see I got the value 4, which is what is printed when the test that passes runs. That output has been captured. The output from the test that failed, I got the value 8, appears in the section of the test summary output, which also shows the cause of the test failure.

If we want to see printed values for passing tests as well, we can tell Rust to also show the output of successful tests with --show-output.

请注意，在此输出中，我们没有看到值 4，这是运行通过的测试时打印的值。 该输出已被捕获。 失败的测试的输出（我得到的值是 8）出现在测试摘要输出部分，其中还显示了测试失败的原因。

如果我们还想查看通过测试的打印值，我们可以告诉 Rust 也使用 --show-output 显示成功测试的输出。

```
$ cargo test -- --show-output

```
When we run the tests in Listing 11-10 again with the --show-output flag, we see the following output:

```
$ cargo test -- --show-output
   Compiling silly-function v0.1.0 (file:///projects/silly-function)
    Finished test [unoptimized + debuginfo] target(s) in 0.60s
     Running unittests src/lib.rs (target/debug/deps/silly_function-160869f38cff9166)

running 2 tests
test tests::this_test_will_fail ... FAILED
test tests::this_test_will_pass ... ok

successes:

---- tests::this_test_will_pass stdout ----
I got the value 4


successes:
    tests::this_test_will_pass

failures:

---- tests::this_test_will_fail stdout ----
I got the value 8
thread 'tests::this_test_will_fail' panicked at 'assertion failed: `(left == right)`
  left: `5`,
 right: `10`', src/lib.rs:19:9
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace


failures:
    tests::this_test_will_fail

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

error: test failed, to rerun pass `--lib`

```

### Running a Subset of Tests by Name
Sometimes, running a full test suite can take a long time. If you’re working on code in a particular area, you might want to run only the tests pertaining to that code. You can choose which tests to run by passing cargo test the name or names of the test(s) you want to run as an argument.

To demonstrate how to run a subset of tests, we’ll first create three tests for our add_two function, as shown in Listing 11-11, and choose which ones to run.

有时，运行完整的测试套件可能需要很长时间。 如果您正在处理特定区域的代码，您可能只想运行与该代码相关的测试。 您可以通过将要运行的测试的名称作为参数传递给 Cargo test 来选择要运行的测试。

为了演示如何运行测试子集，我们首先为 add_two 函数创建三个测试，如清单 11-11 所示，然后选择要运行的测试。

Filename: src/lib.rs
```
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn add_two_and_two() {
        assert_eq!(4, add_two(2));
    }

    #[test]
    fn add_three_and_two() {
        assert_eq!(5, add_two(3));
    }

    #[test]
    fn one_hundred() {
        assert_eq!(102, add_two(100));
    }
}
```
Listing 11-11: Three tests with three different names

If we run the tests without passing any arguments, as we saw earlier, all the tests will run in parallel:

```
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.62s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 3 tests
test tests::add_three_and_two ... ok
test tests::add_two_and_two ... ok
test tests::one_hundred ... ok

test result: ok. 3 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s


```

### Running Single Tests

We can pass the name of any test function to cargo test to run only that test:

```
$ cargo test one_hundred
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.69s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 1 test
test tests::one_hundred ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 2 filtered out; finished in 0.00s


```

Only the test with the name one_hundred ran; the other two tests didn’t match that name. The test output lets us know we had more tests that didn’t run by displaying 2 filtered out at the end.

We can’t specify the names of multiple tests in this way; only the first value given to cargo test will be used. But there is a way to run multiple tests.

仅运行名为 one_hundred 的测试； 其他两个测试与该名称不匹配。 测试输出最后显示 2 个被过滤掉的结果，让我们知道还有更多未运行的测试。

我们不能通过这种方式指定多个测试的名称； 仅使用给予货物测试的第一个值。 但有一种方法可以运行多个测试。

### Filtering to Run Multiple Tests
We can specify part of a test name, and any test whose name matches that value will be run. For example, because two of our tests’ names contain add, we can run those two by running cargo

我们可以指定测试名称的一部分，并且名称与该值匹配的任何测试都将运行。 例如，因为我们的两个测试名称包含 add，所以我们可以通过运行 Cargo 来运行这两个测试

```
$ cargo test add
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.61s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 2 tests
test tests::add_three_and_two ... ok
test tests::add_two_and_two ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 1 filtered out; finished in 0.00s


```
This command ran all tests with add in the name and filtered out the test named one_hundred. Also note that the module in which a test appears becomes part of the test’s name, so we can run all the tests in a module by filtering on the module’s name.

此命令运行名称中包含 add 的所有测试，并过滤掉名为 one_hundred 的测试。 另请注意，测试所在的模块将成为测试名称的一部分，因此我们可以通过过滤模块名称来运行模块中的所有测试。

### Ignoring Some Tests Unless Specifically Requested
Sometimes a few specific tests can be very time-consuming to execute, so you might want to exclude them during most runs of cargo test. Rather than listing as arguments all tests you do want to run, you can instead annotate the time-consuming tests using the ignore attribute to exclude them, as shown here:

有时，执行一些特定的测试可能非常耗时，因此您可能希望在大多数货物测试运行期间排除它们。 您可以使用ignore属性注释耗时的测试以排除它们，而不是将您想要运行的所有测试作为参数列出，如下所示：

Filename: src/lib.rs
```
#[test]
fn it_works() {
    assert_eq!(2 + 2, 4);
}

#[test]
#[ignore]
fn expensive_test() {
    // code that takes an hour to run
}
```
After #[test] we add the #[ignore] line to the test we want to exclude. Now when we run our tests, it_works runs, but expensive_test doesn’t:
```
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.60s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 2 tests
test expensive_test ... ignored
test it_works ... ok

test result: ok. 1 passed; 0 failed; 1 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s


```
The expensive_test function is listed as ignored. If we want to run only the ignored tests, we can use cargo test -- --ignored:

```
$ cargo test -- --ignored
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.61s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 1 test
test expensive_test ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 1 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s


```
By controlling which tests run, you can make sure your cargo test results will be fast. When you’re at a point where it makes sense to check the results of the ignored tests and you have time to wait for the results, you can run cargo test -- --ignored instead. If you want to run all tests whether they’re ignored or not, you can run cargo test -- --include-ignored.

通过控制运行哪些测试，您可以确保快速获得货物测试结果。 当您需要检查被忽略的测试的结果并且您有时间等待结果时，您可以运行 Cargo test -- --ignored 来代替。 如果你想运行所有测试，无论它们是否被忽略，你可以运行cargo test -- --include-ignored。
## 11.3 Test Organization

As mentioned at the start of the chapter, testing is a complex discipline, and different people use different terminology and organization. The Rust community thinks about tests in terms of two main categories: unit tests and integration tests. Unit tests are small and more focused, testing one module in isolation at a time, and can test private interfaces. Integration tests are entirely external to your library and use your code in the same way any other external code would, using only the public interface and potentially exercising multiple modules per test.

Writing both kinds of tests is important to ensure that the pieces of your library are doing what you expect them to, separately and together.



正如本章开头提到的，测试是一门复杂的学科，不同的人使用不同的术语和组织。 Rust 社区从两个主要类别来考虑测试：单元测试和集成测试。 单元测试规模较小且更有针对性，一次单独测试一个模块，并且可以测试私有接口。 集成测试完全在您的库外部，并以与任何其他外部代码相同的方式使用您的代码，仅使用公共接口，并且每个测试可能会执行多个模块。

编写这两种测试对于确保库的各个部分分别或一起执行您期望的操作非常重要。

### 单元测试

The purpose of unit tests is to test each unit of code in isolation from the rest of the code to quickly pinpoint where code is and isn’t working as expected. You’ll put unit tests in the src directory in each file with the code that they’re testing. The convention is to create a module named tests in each file to contain the test functions and to annotate the module with cfg(test).

单元测试的目的是独立于其余代码来测试每个代码单元，以快速查明代码在何处按预期工作以及在何处未按预期工作。 您将使用它们正在测试的代码将单元测试放在每个文件的 src 目录中。 约定是在每个文件中创建一个名为tests的模块来包含测试函数并用cfg(test)注释该模块。

#### The Tests Module and #[cfg(test)]

The #[cfg(test)] annotation on the tests module tells Rust to compile and run the test code only when you run cargo test, not when you run cargo build. This saves compile time when you only want to build the library and saves space in the resulting compiled artifact because the tests are not included. You’ll see that because integration tests go in a different directory, they don’t need the #[cfg(test)] annotation. However, because unit tests go in the same files as the code, you’ll use #[cfg(test)] to specify that they shouldn’t be included in the compiled result.

Recall that when we generated the new adder project in the first section of this chapter, Cargo generated this code for us:

测试模块上的#[cfg(test)]注释告诉Rust仅在运行cargo test时编译并运行测试代码，而不是在运行cargo build时。 当您只想构建库时，这可以节省编译时间，并节省生成的编译工件的空间，因为不包含测试。 您会看到，由于集成测试位于不同的目录中，因此它们不需要 #[cfg(test)] 注释。 但是，由于单元测试与代码位于同一文件中，因此您将使用 #[cfg(test)] 来指定它们不应包含在编译结果中。

回想一下，当我们在本章第一部分生成新的加法器项目时，Cargo 为我们生成了以下代码：

Filename: src/lib.rs

```
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        let result = 2 + 2;
        assert_eq!(result, 4);
    }
}
```
This code is the automatically generated test module. The attribute cfg stands for configuration and tells Rust that the following item should only be included given a certain configuration option. In this case, the configuration option is test, which is provided by Rust for compiling and running tests. By using the cfg attribute, Cargo compiles our test code only if we actively run the tests with cargo test. This includes any helper functions that might be within this module, in addition to the functions annotated with #[test].

该代码是自动生成的测试模块。 属性 cfg 代表配置，并告诉 Rust 仅在给定特定配置选项的情况下才应包含以下项目。 在这种情况下，配置选项是 test，这是 Rust 提供的用于编译和运行测试的选项。 通过使用 cfg 属性，只有当我们主动使用 Cargo test 运行测试时，Cargo 才会编译我们的测试代码。 除了用 #[test] 注释的函数之外，这还包括此模块中可能存在的任何辅助函数。

#### Testing Private Functions
There’s debate within the testing community about whether or not private functions should be tested directly, and other languages make it difficult or impossible to test private functions. Regardless of which testing ideology you adhere to, Rust’s privacy rules do allow you to test private functions. Consider the code in Listing 11-12 with the private function internal_adder.

测试社区内部存在关于是否应该直接测试私有函数的争论，而其他语言使得测试私有函数变得困难或不可能。 无论您遵循哪种测试理念，Rust 的隐私规则都允许您测试私有函数。 考虑清单 11-12 中带有私有函数internal_adder 的代码。

Filename: src/lib.rs

```
pub fn add_two(a: i32) -> i32 {
    internal_adder(a, 2)
}

fn internal_adder(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn internal() {
        assert_eq!(4, internal_adder(2, 2));
    }
}
```

Listing 11-12: Testing a private function

Note that the internal_adder function is not marked as pub. Tests are just Rust code, and the tests module is just another module. As we discussed in the “Paths for Referring to an Item in the Module Tree” section, items in child modules can use the items in their ancestor modules. In this test, we bring all of the test module’s parent’s items into scope with use super::*, and then the test can call internal_adder. If you don’t think private functions should be tested, there’s nothing in Rust that will compel you to do so.

请注意，internal_adder 函数未标记为 pub。 测试只是 Rust 代码，测试模块只是另一个模块。 正如我们在“引用模块树中的项目的路径”部分中讨论的那样，子模块中的项目可以使用其祖先模块中的项目。 在此测试中，我们使用 super::* 将测试模块的所有父项纳入作用域，然后测试可以调用internal_adder。 如果您认为不应该测试私有函数，Rust 中没有任何内容会迫使您这样做。

#### Integration Tests
In Rust, integration tests are entirely external to your library. They use your library in the same way any other code would, which means they can only call functions that are part of your library’s public API. Their purpose is to test whether many parts of your library work together correctly. Units of code that work correctly on their own could have problems when integrated, so test coverage of the integrated code is important as well. To create integration tests, you first need a tests directory.

在 Rust 中，集成测试完全在您的库外部。 他们以与任何其他代码相同的方式使用您的库，这意味着他们只能调用属于您的库公共 API 一部分的函数。 它们的目的是测试库的许多部分是否可以正常工作。 单独正常工作的代码单元在集成时可能会出现问题，因此集成代码的测试覆盖率也很重要。 要创建集成测试，您首先需要一个测试目录。

#### The tests Directory
We create a tests directory at the top level of our project directory, next to src. Cargo knows to look for integration test files in this directory. We can then make as many test files as we want, and Cargo will compile each of the files as an individual crate.

Let’s create an integration test. With the code in Listing 11-12 still in the src/lib.rs file, make a tests directory, and create a new file named tests/integration_test.rs. Your directory structure should look like this:

我们在项目目录的顶层、src 旁边创建一个测试目录。 Cargo 知道在此目录中查找集成测试文件。 然后我们可以制作任意数量的测试文件，Cargo 会将每个文件编译为单独的 crate。

让我们创建一个集成测试。 清单 11-12 中的代码仍在 src/lib.rs 文件中，创建一个测试目录，并创建一个名为tests/integration_test.rs 的新文件。 您的目录结构应如下所示：

```
adder
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    └── integration_test.rs

```

Enter the code in Listing 11-13 into the tests/integration_test.rs file:

Filename: tests/integration_test.rs
```
use adder;

#[test]
fn it_adds_two() {
    assert_eq!(4, adder::add_two(2));
}
```
Listing 11-13: An integration test of a function in the adder crate

Each file in the tests directory is a separate crate, so we need to bring our library into each test crate’s scope. For that reason we add use adder at the top of the code, which we didn’t need in the unit tests.

We don’t need to annotate any code in tests/integration_test.rs with #[cfg(test)]. Cargo treats the tests directory specially and compiles files in this directory only when we run cargo test. Run cargo test now:

测试目录中的每个文件都是一个单独的板条箱，因此我们需要将我们的库纳入每个测试板条箱的范围。 因此，我们在代码顶部添加了 use adder，这在单元测试中是不需要的。

我们不需要用#[cfg(test)]注释tests/integration_test.rs中的任何代码。 Cargo 对测试目录进行了特殊处理，仅当我们运行 Cargo test 时才编译该目录中的文件。 现在运行货物测试：

```
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 1.31s
     Running unittests src/lib.rs (target/debug/deps/adder-1082c4b063a8fbe6)

running 1 test
test tests::internal ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running tests/integration_test.rs (target/debug/deps/integration_test-1082c4b063a8fbe6)

running 1 test
test it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s


```

The three sections of output include the unit tests, the integration test, and the doc tests. Note that if any test in a section fails, the following sections will not be run. For example, if a unit test fails, there won’t be any output for integration and doc tests because those tests will only be run if all unit tests are passing.

The first section for the unit tests is the same as we’ve been seeing: one line for each unit test (one named internal that we added in Listing 11-12) and then a summary line for the unit tests.

The integration tests section starts with the line Running tests/integration_test.rs. Next, there is a line for each test function in that integration test and a summary line for the results of the integration test just before the Doc-tests adder section starts.

Each integration test file has its own section, so if we add more files in the tests directory, there will be more integration test sections.

We can still run a particular integration test function by specifying the test function’s name as an argument to cargo test. To run all the tests in a particular integration test file, use the --test argument of cargo test followed by the name of the file:

输出的三个部分包括单元测试、集成测试和文档测试。 请注意，如果某个部分中的任何测试失败，则不会运行以下部分。 例如，如果单元测试失败，则集成和文档测试不会有任何输出，因为只有所有单元测试都通过时才会运行这些测试。

单元测试的第一部分与我们所看到的相同：每个单元测试一行（我们在清单 11-12 中添加的名为“internal”的行），然后是单元测试的摘要行。

集成测试部分以“运行测试/integration_test.rs”行开始。 接下来，在 Doc-tests 加法器部分开始之前，该集成测试中的每个测试函数都有一行，还有一个集成测试结果的摘要行。

每个集成测试文件都有自己的部分，因此如果我们在测试目录中添加更多文件，将会有更多的集成测试部分。

我们仍然可以通过将测试函数的名称指定为 Cargo test 的参数来运行特定的集成测试函数。 要运行特定集成测试文件中的所有测试，请使用 Cargo test 的 --test 参数，后跟文件名：

```
$ cargo test --test integration_test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.64s
     Running tests/integration_test.rs (target/debug/deps/integration_test-82e7799c1bc62298)

running 1 test
test it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s


```
This command runs only the tests in the tests/integration_test.rs file.

#### Submodules in Integration Tests

As you add more integration tests, you might want to make more files in the tests directory to help organize them; for example, you can group the test functions by the functionality they’re testing. As mentioned earlier, each file in the tests directory is compiled as its own separate crate, which is useful for creating separate scopes to more closely imitate the way end users will be using your crate. However, this means files in the tests directory don’t share the same behavior as files in src do, as you learned in Chapter 7 regarding how to separate code into modules and files.

The different behavior of tests directory files is most noticeable when you have a set of helper functions to use in multiple integration test files and you try to follow the steps in the “Separating Modules into Different Files” section of Chapter 7 to extract them into a common module. For example, if we create tests/common.rs and place a function named setup in it, we can add some code to setup that we want to call from multiple test functions in multiple test files:

当您添加更多集成测试时，您可能希望在测试目录中创建更多文件以帮助组织它们； 例如，您可以根据测试功能正在测试的功能对测试功能进行分组。 如前所述，tests 目录中的每个文件都被编译为自己单独的 crate，这对于创建单独的范围以更接近地模仿最终用户使用您的 crate 的方式非常有用。 然而，这意味着测试目录中的文件与 src 中的文件不具有相同的行为，正如您在第 7 章中了解的如何将代码分离为模块和文件一样。

当您有一组辅助函数要在多个集成测试文件中使用，并且您尝试按照第 7 章“将模块分离到不同文件”部分中的步骤将它们提取到一个文件中时，测试目录文件的不同行为最为明显。 通用模块。 例如，如果我们创建tests/common.rs并在其中放置一个名为setup的函数，我们可以向setup添加一些我们想要从多个测试文件中的多个测试函数调用的代码：

Filename: tests/common.rs
```
pub fn setup() {
    // setup code specific to your library's tests would go here
}
```
When we run the tests again, we’ll see a new section in the test output for the common.rs file, even though this file doesn’t contain any test functions nor did we call the setup function from anywhere:

```
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished test [unoptimized + debuginfo] target(s) in 0.89s
     Running unittests src/lib.rs (target/debug/deps/adder-92948b65e88960b4)

running 1 test
test tests::internal ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running tests/common.rs (target/debug/deps/common-92948b65e88960b4)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running tests/integration_test.rs (target/debug/deps/integration_test-92948b65e88960b4)

running 1 test
test it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s


```
Having common appear in the test results with running 0 tests displayed for it is not what we wanted. We just wanted to share some code with the other integration test files.

To avoid having common appear in the test output, instead of creating tests/common.rs, we’ll create tests/common/mod.rs. The project directory now looks like this:

在测试结果中出现 common 并显示运行 0 个测试并不是我们想要的。 我们只是想与其他集成测试文件共享一些代码。

为了避免common出现在测试输出中，我们将创建tests/common/mod.rs，而不是创建tests/common.rs。 项目目录现在如下所示：

```
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    ├── common
    │   └── mod.rs
    └── integration_test.rs

```
This is the older naming convention that Rust also understands that we mentioned in the “Alternate File Paths” section of Chapter 7. Naming the file this way tells Rust not to treat the common module as an integration test file. When we move the setup function code into tests/common/mod.rs and delete the tests/common.rs file, the section in the test output will no longer appear. Files in subdirectories of the tests directory don’t get compiled as separate crates or have sections in the test output.

After we’ve created tests/common/mod.rs, we can use it from any of the integration test files as a module. Here’s an example of calling the setup function from the it_adds_two test in tests/integration_test.rs:

这是 Rust 也理解的旧命名约定，我们在第 7 章的“备用文件路径”部分中提到过。以这种方式命名文件告诉 Rust 不要将公共模块视为集成测试文件。 当我们将设置功能代码移至tests/common/mod.rs并删除tests/common.rs文件时，测试输出中的该部分将不再出现。 测试目录的子目录中的文件不会被编译为单独的包，也不会在测试输出中包含部分。

创建tests/common/mod.rs后，我们可以从任何集成测试文件中将其用作模块。 以下是从 test/integration_test.rs 中的 it_adds_two 测试调用设置函数的示例：

Filename: tests/integration_test.rs

```
use adder;

mod common;

#[test]
fn it_adds_two() {
    common::setup();
    assert_eq!(4, adder::add_two(2));
}
```
Note that the mod common; declaration is the same as the module declaration we demonstrated in Listing 7-21. Then in the test function, we can call the common::setup() function.

#### Integration Tests for Binary Crates
If our project is a binary crate that only contains a src/main.rs file and doesn’t have a src/lib.rs file, we can’t create integration tests in the tests directory and bring functions defined in the src/main.rs file into scope with a use statement. Only library crates expose functions that other crates can use; binary crates are meant to be run on their own.

This is one of the reasons Rust projects that provide a binary have a straightforward src/main.rs file that calls logic that lives in the src/lib.rs file. Using that structure, integration tests can test the library crate with use to make the important functionality available. If the important functionality works, the small amount of code in the src/main.rs file will work as well, and that small amount of code doesn’t need to be tested.

如果我们的项目是一个只包含 src/main.rs 文件而没有 src/lib.rs 文件的二进制 crate，则我们无法在测试目录中创建集成测试并引入 src/main 中定义的函数 使用 use 语句将 .rs 文件放入范围内。 只有库 crate 才会公开其他 crate 可以使用的函数； 二进制板条箱应该独立运行。

这是提供二进制文件的 Rust 项目有一个简单的 src/main.rs 文件的原因之一，该文件调用 src/lib.rs 文件中的逻辑。 使用该结构，集成测试可以测试库箱的使用情况，以使重要的功能可用。 如果重要的功能可以工作，那么 src/main.rs 文件中的少量代码也可以工作，并且不需要测试这少量代码。

## 11.4 Summary
Rust’s testing features provide a way to specify how code should function to ensure it continues to work as you expect, even as you make changes. Unit tests exercise different parts of a library separately and can test private implementation details. Integration tests check that many parts of the library work together correctly, and they use the library’s public API to test the code in the same way external code will use it. Even though Rust’s type system and ownership rules help prevent some kinds of bugs, tests are still important to reduce logic bugs having to do with how your code is expected to behave.

Let’s combine the knowledge you learned in this chapter and in previous chapters to work on a project!

Rust 的测试功能提供了一种指定代码应如何运行的方法，以确保即使您进行更改，它也能继续按您的预期工作。 单元测试分别运行库的不同部分，并且可以测试私有实现细节。 集成测试检查库的许多部分是否正确协同工作，并且它们使用库的公共 API 来测试代码，就像外部代码使用它一样。 尽管 Rust 的类型系统和所有权规则有助于防止某些类型的错误，但测试对于减少与代码预期行为方式相关的逻辑错误仍然很重要。

让我们结合您在本章和前面章节中学到的知识来完成一个项目！