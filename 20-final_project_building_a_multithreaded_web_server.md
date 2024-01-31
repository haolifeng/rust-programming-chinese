# 20. 最后的项目，多线程Web服务器(Final Project: Building a Multithreaded Web Server)
It’s been a long journey, but we’ve reached the end of the book. In this chapter, we’ll build one more project together to demonstrate some of the concepts we covered in the final chapters, as well as recap some earlier lessons.

For our final project, we’ll make a web server that says “hello” and looks like Figure 20-1 in a web browser.

Here is our plan for building the web server:

1. Learn a bit about TCP and HTTP.
2. Listen for TCP connections on a socket.
3. Parse a small number of HTTP requests.
4. Create a proper HTTP response.
5. Improve the throughput of our server with a thread pool.

Before we get started, we should mention one detail: the method we’ll use won’t be the best way to build a web server with Rust. Community members have published a number of production-ready crates available on crates.io that provide more complete web server and thread pool implementations than we’ll build. However, our intention in this chapter is to help you learn, not to take the easy route. Because Rust is a systems programming language, we can choose the level of abstraction we want to work with and can go to a lower level than is possible or practical in other languages. We’ll therefore write the basic HTTP server and thread pool manually so you can learn the general ideas and techniques behind the crates you might use in the future.

这是一段漫长的旅程，但我们已经读完了这本书。 在本章中，我们将一起构建另一个项目来演示我们在最后几章中介绍的一些概念，并回顾一些早期的课程。

对于我们的最终项目，我们将制作一个 Web 服务器，它在 Web 浏览器中显示“hello”，如图 20-1 所示。

这是我们构建 Web 服务器的计划：

1. 了解一些关于 TCP 和 HTTP 的知识。
2. 侦听套接字上的 TCP 连接。
3. 解析少量HTTP请求。
4. 创建正确的 HTTP 响应。
5. 使用线程池提高服务器的吞吐量。

在开始之前，我们应该提到一个细节：我们将使用的方法并不是使用 Rust 构建 Web 服务器的最佳方法。 社区成员在 crates.io 上发布了许多可用于生产的 crate，它们提供了比我们构建的更完整的 Web 服务器和线程池实现。 然而，我们本章的目的是帮助您学习，而不是走捷径。 因为 Rust 是一种系统编程语言，所以我们可以选择我们想要使用的抽象级别，并且可以达到比其他语言可能或实用的更低的级别。 因此，我们将手动编写基本的 HTTP 服务器和线程池，以便您可以了解将来可能使用的 crate 背后的一般思想和技术。

## 20.1 Building a Single-Threaded Web Server

We’ll start by getting a single-threaded web server working. Before we begin, let’s look at a quick overview of the protocols involved in building web servers. The details of these protocols are beyond the scope of this book, but a brief overview will give you the information you need.

The two main protocols involved in web servers are Hypertext Transfer Protocol (HTTP) and Transmission Control Protocol (TCP). Both protocols are request-response protocols, meaning a client initiates requests and a server listens to the requests and provides a response to the client. The contents of those requests and responses are defined by the protocols.

TCP is the lower-level protocol that describes the details of how information gets from one server to another but doesn’t specify what that information is. HTTP builds on top of TCP by defining the contents of the requests and responses. It’s technically possible to use HTTP with other protocols, but in the vast majority of cases, HTTP sends its data over TCP. We’ll work with the raw bytes of TCP and HTTP requests and responses.

我们将从让单线程 Web 服务器运行开始。 在开始之前，让我们快速概述一下构建 Web 服务器所涉及的协议。 这些协议的详细信息超出了本书的范围，但简要概述将为您提供所需的信息。

Web 服务器涉及的两个主要协议是超文本传输协议 (HTTP) 和传输控制协议 (TCP)。 这两种协议都是请求-响应协议，这意味着客户端发起请求，服务器监听请求并向客户端提供响应。 这些请求和响应的内容由协议定义。

TCP 是较低级别的协议，它描述信息如何从一台服务器传输到另一台服务器的详细信息，但不指定该信息是什么。 HTTP 通过定义请求和响应的内容构建在 TCP 之上。 从技术上讲，可以将 HTTP 与其他协议结合使用，但在绝大多数情况下，HTTP 通过 TCP 发送数据。 我们将使用 TCP 和 HTTP 请求和响应的原始字节。

### Listening to the TCP Connection
Our web server needs to listen to a TCP connection, so that’s the first part we’ll work on. The standard library offers a std::net module that lets us do this. Let’s make a new project in the usual fashion:
``````
$ cargo new hello
     Created binary (application) `hello` project
$ cd hello
``````
Now enter the code in Listing 20-1 in src/main.rs to start. This code will listen at the local address 127.0.0.1:7878 for incoming TCP streams. When it gets an incoming stream, it will print Connection established!.

Filename: src/main.rs
``````
use std::net::TcpListener;

fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();

    for stream in listener.incoming() {
        let stream = stream.unwrap();

        println!("Connection established!");
    }
}
``````
Listing 20-1: Listening for incoming streams and printing a message when we receive a stream

Using TcpListener, we can listen for TCP connections at the address 127.0.0.1:7878. In the address, the section before the colon is an IP address representing your computer (this is the same on every computer and doesn’t represent the authors’ computer specifically), and 7878 is the port. We’ve chosen this port for two reasons: HTTP isn’t normally accepted on this port so our server is unlikely to conflict with any other web server you might have running on your machine, and 7878 is rust typed on a telephone.

The bind function in this scenario works like the new function in that it will return a new TcpListener instance. The function is called bind because, in networking, connecting to a port to listen to is known as “binding to a port.”

The bind function returns a Result<T, E>, which indicates that it’s possible for binding to fail. For example, connecting to port 80 requires administrator privileges (nonadministrators can listen only on ports higher than 1023), so if we tried to connect to port 80 without being an administrator, binding wouldn’t work. Binding also wouldn’t work, for example, if we ran two instances of our program and so had two programs listening to the same port. Because we’re writing a basic server just for learning purposes, we won’t worry about handling these kinds of errors; instead, we use unwrap to stop the program if errors happen.

The incoming method on TcpListener returns an iterator that gives us a sequence of streams (more specifically, streams of type TcpStream). A single stream represents an open connection between the client and the server. A connection is the name for the full request and response process in which a client connects to the server, the server generates a response, and the server closes the connection. As such, we will read from the TcpStream to see what the client sent and then write our response to the stream to send data back to the client. Overall, this for loop will process each connection in turn and produce a series of streams for us to handle.

For now, our handling of the stream consists of calling unwrap to terminate our program if the stream has any errors; if there aren’t any errors, the program prints a message. We’ll add more functionality for the success case in the next listing. The reason we might receive errors from the incoming method when a client connects to the server is that we’re not actually iterating over connections. Instead, we’re iterating over connection attempts. The connection might not be successful for a number of reasons, many of them operating system specific. For example, many operating systems have a limit to the number of simultaneous open connections they can support; new connection attempts beyond that number will produce an error until some of the open connections are closed.

Let’s try running this code! Invoke cargo run in the terminal and then load 127.0.0.1:7878 in a web browser. The browser should show an error message like “Connection reset,” because the server isn’t currently sending back any data. But when you look at your terminal, you should see several messages that were printed when the browser connected to the server!

使用TcpListener，我们可以监听地址127.0.0.1:7878的TCP连接。 在地址中，冒号之前的部分是代表您计算机的IP地址（这在每台计算机上都是相同的，并不具体代表作者的计算机），7878是端口。 我们选择此端口有两个原因：此端口通常不接受 HTTP，因此我们的服务器不太可能与您计算机上运行的任何其他 Web 服务器发生冲突，并且 7878 是在电话上键入的。

在这种情况下，bind 函数的工作方式与 new 函数类似，它将返回一个新的 TcpListener 实例。 该函数称为“bind”，因为在网络中，连接到要侦听的端口称为“绑定到端口”。

bind 函数返回一个 Result<T, E>，这表明绑定可能会失败。 例如，连接到端口 80 需要管理员权限（非管理员只能侦听高于 1023 的端口），因此如果我们尝试在没有管理员身份的情况下连接到端口 80，则绑定将不起作用。 绑定也不起作用，例如，如果我们运行程序的两个实例，并且有两个程序侦听同一端口。 因为我们编写一个基本服务器只是为了学习目的，所以我们不会担心处理这些类型的错误； 相反，如果发生错误，我们使用 unwrap 来停止程序。

TcpListener 上的传入方法返回一个迭代器，该迭代器为我们提供了一系列流（更具体地说，TcpStream 类型的流）。 单个流代表客户端和服务器之间的开放连接。 连接是完整的请求和响应过程的名称，其中客户端连接到服务器，服务器生成响应，然后服务器关闭连接。 因此，我们将从 TcpStream 中读取以查看客户端发送的内容，然后将响应写入流以将数据发送回客户端。 总的来说，这个 for 循环将依次处理每个连接并产生一系列流供我们处理。

目前，我们对流的处理包括：如果流有任何错误，则调用 unwrap 来终止我们的程序； 如果没有任何错误，程序会打印一条消息。 我们将在下一个清单中为成功案例添加更多功能。 当客户端连接到服务器时，我们可能会从传入方法收到错误，原因是我们实际上并没有迭代连接。 相反，我们正在迭代连接尝试。 连接可能会因多种原因而失败，其中许多原因是特定于操作系统的。 例如，许多操作系统对其可以支持的同时打开连接的数量有限制； 超过该数量的新连接尝试将产生错误，直到关闭某些打开的连接。

让我们尝试运行这段代码！ 在终端中调用 Cargo Run，然后在 Web 浏览器中加载 127.0.0.1:7878。 浏览器应该显示一条错误消息，例如“连接重置”，因为服务器当前没有发回任何数据。 但是当您查看终端时，您应该会看到浏览器连接到服务器时打印的几条消息！

``````
     Running `target/debug/hello`
Connection established!
Connection established!
Connection established!
``````
Sometimes, you’ll see multiple messages printed for one browser request; the reason might be that the browser is making a request for the page as well as a request for other resources, like the favicon.ico icon that appears in the browser tab.

It could also be that the browser is trying to connect to the server multiple times because the server isn’t responding with any data. When stream goes out of scope and is dropped at the end of the loop, the connection is closed as part of the drop implementation. Browsers sometimes deal with closed connections by retrying, because the problem might be temporary. The important factor is that we’ve successfully gotten a handle to a TCP connection!

Remember to stop the program by pressing ctrl-c when you’re done running a particular version of the code. Then restart the program by invoking the cargo run command after you’ve made each set of code changes to make sure you’re running the newest code.

有时，您会看到针对一个浏览器请求打印多条消息； 原因可能是浏览器正在请求该页面以及其他资源，例如浏览器选项卡中显示的 favicon.ico 图标。

也可能是浏览器多次尝试连接到服务器，因为服务器没有响应任何数据。 当流超出范围并在循环结束时被删除时，连接将作为删除实现的一部分关闭。 浏览器有时会通过重试来处理关闭的连接，因为问题可能是暂时的。 重要的是我们已经成功获得了 TCP 连接的句柄！

请记住在运行完特定版本的代码后按 ctrl-c 来停止程序。 然后，在完成每组代码更改后，通过调用 Cargo run 命令来重新启动程序，以确保运行最新的代码。

### Reading the Request
Let’s implement the functionality to read the request from the browser! To separate the concerns of first getting a connection and then taking some action with the connection, we’ll start a new function for processing connections. In this new handle_connection function, we’ll read data from the TCP stream and print it so we can see the data being sent from the browser. Change the code to look like Listing 20-2.

让我们实现从浏览器读取请求的功能！ 为了区分首先获取连接然后对连接采取一些操作的问题，我们将启动一个新函数来处理连接。 在这个新的handle_connection函数中，我们将从TCP流中读取数据并打印出来，这样我们就可以看到从浏览器发送的数据。 将代码更改为如清单 20-2 所示。

Filename: src/main.rs
``````
use std::{
    io::{prelude::*, BufReader},
    net::{TcpListener, TcpStream},
};

fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();

    for stream in listener.incoming() {
        let stream = stream.unwrap();

        handle_connection(stream);
    }
}

fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let http_request: Vec<_> = buf_reader
        .lines()
        .map(|result| result.unwrap())
        .take_while(|line| !line.is_empty())
        .collect();

    println!("Request: {:#?}", http_request);
}
``````
Listing 20-2: Reading from the TcpStream and printing the data

We bring std::io::prelude and std::io::BufReader into scope to get access to traits and types that let us read from and write to the stream. In the for loop in the main function, instead of printing a message that says we made a connection, we now call the new handle_connection function and pass the stream to it.

In the handle_connection function, we create a new BufReader instance that wraps a mutable reference to the stream. BufReader adds buffering by managing calls to the std::io::Read trait methods for us.

We create a variable named http_request to collect the lines of the request the browser sends to our server. We indicate that we want to collect these lines in a vector by adding the Vec<_> type annotation.

BufReader implements the std::io::BufRead trait, which provides the lines method. The lines method returns an iterator of Result<String, std::io::Error> by splitting the stream of data whenever it sees a newline byte. To get each String, we map and unwrap each Result. The Result might be an error if the data isn’t valid UTF-8 or if there was a problem reading from the stream. Again, a production program should handle these errors more gracefully, but we’re choosing to stop the program in the error case for simplicity.

The browser signals the end of an HTTP request by sending two newline characters in a row, so to get one request from the stream, we take lines until we get a line that is the empty string. Once we’ve collected the lines into the vector, we’re printing them out using pretty debug formatting so we can take a look at the instructions the web browser is sending to our server.

Let’s try this code! Start the program and make a request in a web browser again. Note that we’ll still get an error page in the browser, but our program’s output in the terminal will now look similar to this:

我们将 std::io::prelude 和 std::io::BufReader 引入作用域，以访问允许我们读取和写入流的特征和类型。 在主函数的 for 循环中，我们现在调用新的 handle_connection 函数并将流传递给它，而不是打印一条表示我们已建立连接的消息。

在handle_connection函数中，我们创建一个新的BufReader实例，它包装了对流的可变引用。 BufReader 通过管理对 std::io::Read 特征方法的调用来添加缓冲。

我们创建一个名为 http_request 的变量来收集浏览器发送到服务器的请求行。 我们通过添加 Vec<_> 类型注释来表明我们想要将这些行收集到向量中。

BufReader 实现了 std::io::BufRead 特征，它提供了lines方法。 lines 方法通过每当看到换行字节时分割数据流来返回 Result<String, std::io::Error> 的迭代器。 为了获取每个字符串，我们映射并解包每个结果。 如果数据不是有效的 UTF-8 或者从流读取时出现问题，结果可能会出现错误。 同样，生产程序应该更优雅地处理这些错误，但为了简单起见，我们选择在错误情况下停止程序。

浏览器通过连续发送两个换行符来表示 HTTP 请求的结束，因此为了从流中获取一个请求，我们需要获取一行，直到得到一行空字符串。 一旦我们将这些行收集到向量中，我们就会使用漂亮的调试格式将它们打印出来，这样我们就可以查看网络浏览器发送到我们服务器的指令。

让我们试试这个代码！ 启动程序并再次在网络浏览器中发出请求。 请注意，我们仍然会在浏览器中看到错误页面，但我们的程序在终端中的输出现在看起来与此类似：

``````
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished dev [unoptimized + debuginfo] target(s) in 0.42s
     Running `target/debug/hello`
Request: [
    "GET / HTTP/1.1",
    "Host: 127.0.0.1:7878",
    "User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:99.0) Gecko/20100101 Firefox/99.0",
    "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8",
    "Accept-Language: en-US,en;q=0.5",
    "Accept-Encoding: gzip, deflate, br",
    "DNT: 1",
    "Connection: keep-alive",
    "Upgrade-Insecure-Requests: 1",
    "Sec-Fetch-Dest: document",
    "Sec-Fetch-Mode: navigate",
    "Sec-Fetch-Site: none",
    "Sec-Fetch-User: ?1",
    "Cache-Control: max-age=0",
]
``````
Depending on your browser, you might get slightly different output. Now that we’re printing the request data, we can see why we get multiple connections from one browser request by looking at the path after GET in the first line of the request. If the repeated connections are all requesting /, we know the browser is trying to fetch / repeatedly because it’s not getting a response from our program.

根据您的浏览器，您可能会得到略有不同的输出。 现在我们正在打印请求数据，通过查看请求第一行中 GET 之后的路径，我们可以了解为什么我们从一个浏览器请求获得多个连接。 如果重复的连接都在请求 /，我们就知道浏览器正在尝试重复获取 /，因为它没有从我们的程序获得响应。

Let’s break down this request data to understand what the browser is asking of our program.

### A Closer Look at an HTTP Request
HTTP is a text-based protocol, and a request takes this format:
``````
Method Request-URI HTTP-Version CRLF
headers CRLF
message-body
``````
The first line is the request line that holds information about what the client is requesting. The first part of the request line indicates the method being used, such as GET or POST, which describes how the client is making this request. Our client used a GET request, which means it is asking for information.

The next part of the request line is /, which indicates the Uniform Resource Identifier (URI) the client is requesting: a URI is almost, but not quite, the same as a Uniform Resource Locator (URL). The difference between URIs and URLs isn’t important for our purposes in this chapter, but the HTTP spec uses the term URI, so we can just mentally substitute URL for URI here.

The last part is the HTTP version the client uses, and then the request line ends in a CRLF sequence. (CRLF stands for carriage return and line feed, which are terms from the typewriter days!) The CRLF sequence can also be written as \r\n, where \r is a carriage return and \n is a line feed. The CRLF sequence separates the request line from the rest of the request data. Note that when the CRLF is printed, we see a new line start rather than \r\n.

Looking at the request line data we received from running our program so far, we see that GET is the method, / is the request URI, and HTTP/1.1 is the version.

After the request line, the remaining lines starting from Host: onward are headers. GET requests have no body.

Try making a request from a different browser or asking for a different address, such as 127.0.0.1:7878/test, to see how the request data changes.

Now that we know what the browser is asking for, let’s send back some data!

第一行是请求行，保存有关客户端请求的信息。 请求行的第一部分指示正在使用的方法，例如 GET 或 POST，它描述了客户端如何发出此请求。 我们的客户使用了 GET 请求，这意味着它正在询问信息。

请求行的下一部分是 /，它指示客户端正在请求的统一资源标识符 (URI)：URI 与统一资源定位符 (URL) 几乎相同，但不完全相同。 URI 和 URL 之间的区别对于我们本章的目的来说并不重要，但 HTTP 规范使用术语 URI，因此我们可以在这里用 URL 代替 URI。

最后一部分是客户端使用的 HTTP 版本，然后请求行以 CRLF 序列结束。 （CRLF 代表回车符和换行符，这是打字机时代的术语！）CRLF 序列也可以写为 \r\n，其中 \r 是回车符，\n 是换行符。 CRLF 序列将请求行与请求数据的其余部分分开。 请注意，当打印 CRLF 时，我们看到一个新行开始，而不是 \r\n。

查看到目前为止运行程序收到的请求行数据，我们看到 GET 是方法，/ 是请求 URI，HTTP/1.1 是版本。

在请求行之后，从 Host: 开始的其余行是标头。 GET 请求没有正文。

尝试从不同的浏览器发出请求或请求不同的地址，例如 127.0.0.1:7878/test，以查看请求数据如何变化。

现在我们知道浏览器要求什么，让我们发回一些数据！

### Writing a Response
We’re going to implement sending data in response to a client request. Responses have the following format:
``````
HTTP-Version Status-Code Reason-Phrase CRLF
headers CRLF
message-body
``````
The first line is a status line that contains the HTTP version used in the response, a numeric status code that summarizes the result of the request, and a reason phrase that provides a text description of the status code. After the CRLF sequence are any headers, another CRLF sequence, and the body of the response.

Here is an example response that uses HTTP version 1.1, has a status code of 200, an OK reason phrase, no headers, and no body:
``````
HTTP/1.1 200 OK\r\n\r\n
``````
The status code 200 is the standard success response. The text is a tiny successful HTTP response. Let’s write this to the stream as our response to a successful request! From the handle_connection function, remove the println! that was printing the request data and replace it with the code in Listing 20-3.

Filename: src/main.rs
``````
fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let http_request: Vec<_> = buf_reader
        .lines()
        .map(|result| result.unwrap())
        .take_while(|line| !line.is_empty())
        .collect();

    let response = "HTTP/1.1 200 OK\r\n\r\n";

    stream.write_all(response.as_bytes()).unwrap();
}
``````
Listing 20-3: Writing a tiny successful HTTP response to the stream

The first new line defines the response variable that holds the success message’s data. Then we call as_bytes on our response to convert the string data to bytes. The write_all method on stream takes a &[u8] and sends those bytes directly down the connection. Because the write_all operation could fail, we use unwrap on any error result as before. Again, in a real application you would add error handling here.

With these changes, let’s run our code and make a request. We’re no longer printing any data to the terminal, so we won’t see any output other than the output from Cargo. When you load 127.0.0.1:7878 in a web browser, you should get a blank page instead of an error. You’ve just hand-coded receiving an HTTP request and sending a response!

第一行定义保存成功消息数据的响应变量。 然后我们在响应中调用 as_bytes 将字符串数据转换为字节。 流上的 write_all 方法采用 &[u8] 并直接通过连接发送这些字节。 由于 write_all 操作可能会失败，因此我们像以前一样对任何错误结果使用 unwrap。 同样，在实际应用程序中，您将在此处添加错误处理。

完成这些更改后，让我们运行代码并发出请求。 我们不再将任何数据打印到终端，因此除了 Cargo 的输出之外，我们不会看到任何输出。 当您在 Web 浏览器中加载 127.0.0.1:7878 时，您应该看到一个空白页面而不是错误。 您刚刚手动编写了接收 HTTP 请求并发送响应的代码！

### Returning Real HTML
Let’s implement the functionality for returning more than a blank page. Create the new file hello.html in the root of your project directory, not in the src directory. You can input any HTML you want; Listing 20-4 shows one possibility.

让我们实现返回多个空白页面的功能。 在项目目录的根目录中创建新文件 hello.html，而不是在 src 目录中。 您可以输入任何您想要的HTML； 清单 20-4 显示了一种可能性。

Filename: hello.html
``````
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Hello!</title>
  </head>
  <body>
    <h1>Hello!</h1>
    <p>Hi from Rust</p>
  </body>
</html>
``````
Listing 20-4: A sample HTML file to return in a response

This is a minimal HTML5 document with a heading and some text. To return this from the server when a request is received, we’ll modify handle_connection as shown in Listing 20-5 to read the HTML file, add it to the response as a body, and send it.

这是一个最小的 HTML5 文档，带有标题和一些文本。 为了在收到请求时从服务器返回此信息，我们将修改handle_connection（如清单20-5所示）以读取HTML文件，将其作为正文添加到响应中，然后发送它。

Filename: src/main.rs
``````
use std::{
    fs,
    io::{prelude::*, BufReader},
    net::{TcpListener, TcpStream},
};
// --snip--

fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let http_request: Vec<_> = buf_reader
        .lines()
        .map(|result| result.unwrap())
        .take_while(|line| !line.is_empty())
        .collect();

    let status_line = "HTTP/1.1 200 OK";
    let contents = fs::read_to_string("hello.html").unwrap();
    let length = contents.len();

    let response =
        format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");

    stream.write_all(response.as_bytes()).unwrap();
}
``````
Listing 20-5: Sending the contents of hello.html as the body of the response

We’ve added fs to the use statement to bring the standard library’s filesystem module into scope. The code for reading the contents of a file to a string should look familiar; we used it in Chapter 12 when we read the contents of a file for our I/O project in Listing 12-4.

Next, we use format! to add the file’s contents as the body of the success response. To ensure a valid HTTP response, we add the Content-Length header which is set to the size of our response body, in this case the size of hello.html.

Run this code with cargo run and load 127.0.0.1:7878 in your browser; you should see your HTML rendered!

Currently, we’re ignoring the request data in http_request and just sending back the contents of the HTML file unconditionally. That means if you try requesting 127.0.0.1:7878/something-else in your browser, you’ll still get back this same HTML response. At the moment, our server is very limited and does not do what most web servers do. We want to customize our responses depending on the request and only send back the HTML file for a well-formed request to /.

我们已将 fs 添加到 use 语句中，以将标准库的文件系统模块纳入范围。 将文件内容读取为字符串的代码应该看起来很熟悉； 我们在第 12 章中使用了它，当时我们读取清单 12-4 中 I/O 项目的文件内容。

接下来，我们使用格式！ 将文件的内容添加为成功响应的正文。 为了确保有效的 HTTP 响应，我们添加了 Content-Length 标头，该标头设置为响应正文的大小，在本例中为 hello.html 的大小。

使用 Cargo Run 运行此代码并在浏览器中加载 127.0.0.1:7878； 你应该看到你的 HTML 被渲染了！

目前，我们忽略 http_request 中的请求数据，只是无条件发回 HTML 文件的内容。 这意味着如果您尝试在浏览器中请求 127.0.0.1:7878/something-else，您仍然会得到相同的 HTML 响应。 目前，我们的服务器非常有限，无法完成大多数网络服务器的工作。 我们希望根据请求自定义响应，并且仅将格式正确的请求的 HTML 文件发送回 /。

### Validating the Request and Selectively Responding
Right now, our web server will return the HTML in the file no matter what the client requested. Let’s add functionality to check that the browser is requesting / before returning the HTML file and return an error if the browser requests anything else. For this we need to modify handle_connection, as shown in Listing 20-6. This new code checks the content of the request received against what we know a request for / looks like and adds if and else blocks to treat requests differently.

现在，无论客户端请求什么，我们的 Web 服务器都会返回文件中的 HTML。 让我们添加功能来检查浏览器是否正在请求 / 在返回 HTML 文件之前，如果浏览器请求其他任何内容，则返回错误。 为此，我们需要修改handle_connection，如清单20-6所示。 这段新代码根据我们所知道的 / 请求检查收到的请求内容，并添加 if 和 else 块以不同方式处理请求。

Filename: src/main.rs
``````
// --snip--

fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let request_line = buf_reader.lines().next().unwrap().unwrap();

    if request_line == "GET / HTTP/1.1" {
        let status_line = "HTTP/1.1 200 OK";
        let contents = fs::read_to_string("hello.html").unwrap();
        let length = contents.len();

        let response = format!(
            "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
        );

        stream.write_all(response.as_bytes()).unwrap();
    } else {
        // some other request
    }
}
``````
Listing 20-6: Handling requests to / differently from other requests

We’re only going to be looking at the first line of the HTTP request, so rather than reading the entire request into a vector, we’re calling next to get the first item from the iterator. The first unwrap takes care of the Option and stops the program if the iterator has no items. The second unwrap handles the Result and has the same effect as the unwrap that was in the map added in Listing 20-2.

Next, we check the request_line to see if it equals the request line of a GET request to the / path. If it does, the if block returns the contents of our HTML file.

If the request_line does not equal the GET request to the / path, it means we’ve received some other request. We’ll add code to the else block in a moment to respond to all other requests.

Run this code now and request 127.0.0.1:7878; you should get the HTML in hello.html. If you make any other request, such as 127.0.0.1:7878/something-else, you’ll get a connection error like those you saw when running the code in Listing 20-1 and Listing 20-2.

Now let’s add the code in Listing 20-7 to the else block to return a response with the status code 404, which signals that the content for the request was not found. We’ll also return some HTML for a page to render in the browser indicating the response to the end user.

我们只会查看 HTTP 请求的第一行，因此我们不会将整个请求读入向量，而是调用 next 从迭代器中获取第一项。 第一个解包负责处理选项，并在迭代器没有项目时停止程序。 第二个解包处理 Result，并且与清单 20-2 中添加的映射中的解包具有相同的效果。

接下来，我们检查 request_line 是否等于对 / 路径的 GET 请求的请求行。 如果是，if 块将返回 HTML 文件的内容。

如果 request_line 不等于 / 路径的 GET 请求，则意味着我们收到了其他请求。 我们稍后将向 else 块添加代码以响应所有其他请求。

现在运行此代码并请求 127.0.0.1:7878; 您应该在 hello.html 中获取 HTML。 如果您发出任何其他请求，例如 127.0.0.1:7878/something-else，您将收到连接错误，如运行清单 20-1 和清单 20-2 中的代码时看到的那样。

现在，我们将清单 20-7 中的代码添加到 else 块中，以返回状态代码为 404 的响应，这表示未找到请求的内容。 我们还将返回一些 HTML 页面以在浏览器中呈现，指示对最终用户的响应。

Filename: src/main.rs
``````
    // --snip--
    } else {
        let status_line = "HTTP/1.1 404 NOT FOUND";
        let contents = fs::read_to_string("404.html").unwrap();
        let length = contents.len();

        let response = format!(
            "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
        );

        stream.write_all(response.as_bytes()).unwrap();
    }
``````
Listing 20-7: Responding with status code 404 and an error page if anything other than / was requested

Here, our response has a status line with status code 404 and the reason phrase NOT FOUND. The body of the response will be the HTML in the file 404.html. You’ll need to create a 404.html file next to hello.html for the error page; again feel free to use any HTML you want or use the example HTML in Listing 20-8.

在这里，我们的响应有一个状态行，状态代码为 404 以及原因短语 NOT FOUND。 响应正文将是文件 404.html 中的 HTML。 您需要在错误页面的 hello.html 旁边创建一个 404.html 文件； 再次，您可以随意使用任何您想要的 HTML 或使用清单 20-8 中的示例 HTML。

Filename: 404.html
``````
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Hello!</title>
  </head>
  <body>
    <h1>Oops!</h1>
    <p>Sorry, I don't know what you're asking for.</p>
  </body>
</html>
``````
Listing 20-8: Sample content for the page to send back with any 404 response

With these changes, run your server again. Requesting 127.0.0.1:7878 should return the contents of hello.html, and any other request, like 127.0.0.1:7878/foo, should return the error HTML from 404.html.

### A Touch of Refactoring
At the moment the if and else blocks have a lot of repetition: they’re both reading files and writing the contents of the files to the stream. The only differences are the status line and the filename. Let’s make the code more concise by pulling out those differences into separate if and else lines that will assign the values of the status line and the filename to variables; we can then use those variables unconditionally in the code to read the file and write the response. Listing 20-9 shows the resulting code after replacing the large if and else blocks.

目前，if 和 else 块有很多重复：它们都在读取文件并将文件内容写入流。 唯一的区别是状态行和文件名。 让我们将这些差异提取到单独的 if 和 else 行中，将状态行和文件名的值分配给变量，从而使代码更加简洁； 然后我们可以在代码中无条件地使用这些变量来读取文件并写入响应。 清单 20-9 显示了替换大的 if 和 else 块后的结果代码。

Filename: src/main.rs
``````
// --snip--

fn handle_connection(mut stream: TcpStream) {
    // --snip--

    let (status_line, filename) = if request_line == "GET / HTTP/1.1" {
        ("HTTP/1.1 200 OK", "hello.html")
    } else {
        ("HTTP/1.1 404 NOT FOUND", "404.html")
    };

    let contents = fs::read_to_string(filename).unwrap();
    let length = contents.len();

    let response =
        format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");

    stream.write_all(response.as_bytes()).unwrap();
}
``````
Listing 20-9: Refactoring the if and else blocks to contain only the code that differs between the two cases

Now the if and else blocks only return the appropriate values for the status line and filename in a tuple; we then use destructuring to assign these two values to status_line and filename using a pattern in the let statement, as discussed in Chapter 18.

The previously duplicated code is now outside the if and else blocks and uses the status_line and filename variables. This makes it easier to see the difference between the two cases, and it means we have only one place to update the code if we want to change how the file reading and response writing work. The behavior of the code in Listing 20-9 will be the same as that in Listing 20-8.

Awesome! We now have a simple web server in approximately 40 lines of Rust code that responds to one request with a page of content and responds to all other requests with a 404 response.

Currently, our server runs in a single thread, meaning it can only serve one request at a time. Let’s examine how that can be a problem by simulating some slow requests. Then we’ll fix it so our server can handle multiple requests at once.

现在，if 和 else 块仅返回元组中状态行和文件名的适当值； 然后，我们使用 let 语句中的模式，使用解构将这两个值分配给 status_line 和 filename，如第 18 章所述。

之前复制的代码现在位于 if 和 else 块之外，并使用 status_line 和 filename 变量。 这使得更容易看出两种情况之间的差异，这意味着如果我们想要更改文件读取和响应写入的工作方式，我们只有一个地方可以更新代码。 清单 20-9 中代码的行为与清单 20-8 中的代码相同。

惊人的！ 我们现在有一个简单的 Web 服务器，大约有 40 行 Rust 代码，它用一页内容响应一个请求，并用 404 响应响应所有其他请求。

目前，我们的服务器在单线程中运行，这意味着它一次只能服务一个请求。 让我们通过模拟一些缓慢的请求来检查这可能是一个问题。 然后我们将修复它，以便我们的服务器可以同时处理多个请求。

## 20.2 Turning Our Single-Threaded Server into a Multithreaded Server

Right now, the server will process each request in turn, meaning it won’t process a second connection until the first is finished processing. If the server received more and more requests, this serial execution would be less and less optimal. If the server receives a request that takes a long time to process, subsequent requests will have to wait until the long request is finished, even if the new requests can be processed quickly. We’ll need to fix this, but first, we’ll look at the problem in action.

现在，服务器将依次处理每个请求，这意味着在第一个连接处理完成之前它不会处理第二个连接。 如果服务器收到越来越多的请求，这种串行执行将越来越不理想。 如果服务器收到一个需要很长时间处理的请求，即使新的请求可以很快处理，后续的请求也必须等到这个长请求完成。 我们需要解决这个问题，但首先，我们将看看实际问题。

### Simulating a Slow Request in the Current Server Implementation
We’ll look at how a slow-processing request can affect other requests made to our current server implementation. Listing 20-10 implements handling a request to /sleep with a simulated slow response that will cause the server to sleep for 5 seconds before responding.

我们将了解处理缓慢的请求如何影响对当前服务器实现发出的其他请求。 清单 20-10 实现了通过模拟慢速响应来处理对 /sleep 的请求，这将导致服务器在响应之前休眠 5 秒。

Filename: src/main.rs
``````
use std::{
    fs,
    io::{prelude::*, BufReader},
    net::{TcpListener, TcpStream},
    thread,
    time::Duration,
};
// --snip--

fn handle_connection(mut stream: TcpStream) {
    // --snip--

    let (status_line, filename) = match &request_line[..] {
        "GET / HTTP/1.1" => ("HTTP/1.1 200 OK", "hello.html"),
        "GET /sleep HTTP/1.1" => {
            thread::sleep(Duration::from_secs(5));
            ("HTTP/1.1 200 OK", "hello.html")
        }
        _ => ("HTTP/1.1 404 NOT FOUND", "404.html"),
    };

    // --snip--
}
``````
Listing 20-10: Simulating a slow request by sleeping for 5 seconds

We switched from if to match now that we have three cases. We need to explicitly match on a slice of request_line to pattern match against the string literal values; match doesn’t do automatic referencing and dereferencing like the equality method does.

The first arm is the same as the if block from Listing 20-9. The second arm matches a request to /sleep. When that request is received, the server will sleep for 5 seconds before rendering the successful HTML page. The third arm is the same as the else block from Listing 20-9.

You can see how primitive our server is: real libraries would handle the recognition of multiple requests in a much less verbose way!

Start the server using cargo run. Then open two browser windows: one for http://127.0.0.1:7878/ and the other for http://127.0.0.1:7878/sleep. If you enter the / URI a few times, as before, you’ll see it respond quickly. But if you enter /sleep and then load /, you’ll see that / waits until sleep has slept for its full 5 seconds before loading.

There are multiple techniques we could use to avoid requests backing up behind a slow request; the one we’ll implement is a thread pool.

现在我们有三种情况，我们从 if 切换到 match。 我们需要显式匹配 request_line 的一部分，以与字符串文字值进行模式匹配； match 不像 equal 方法那样进行自动引用和取消引用。

第一个臂与清单 20-9 中的 if 块相同。 第二个臂匹配 /sleep 请求。 收到该请求后，服务器将休眠 5 秒，然后再渲染成功的 HTML 页面。 第三个分支与清单 20-9 中的 else 块相同。

您可以看到我们的服务器是多么原始：真正的库将以更简洁的方式处理多个请求的识别！

使用货物运行启动服务器。 然后打开两个浏览器窗口：一个为 http://127.0.0.1:7878/，另一个为 http://127.0.0.1:7878/sleep。 如果像以前一样输入 / URI 几次，您会看到它快速响应。 但如果你输入 /sleep 然后加载 /，你会看到 / 会等到 sleep 睡满 5 秒才加载。

我们可以使用多种技术来避免请求在缓慢的请求后面备份； 我们要实现的是线程池。

### Improving Throughput with a Thread Pool
A thread pool is a group of spawned threads that are waiting and ready to handle a task. When the program receives a new task, it assigns one of the threads in the pool to the task, and that thread will process the task. The remaining threads in the pool are available to handle any other tasks that come in while the first thread is processing. When the first thread is done processing its task, it’s returned to the pool of idle threads, ready to handle a new task. A thread pool allows you to process connections concurrently, increasing the throughput of your server.

We’ll limit the number of threads in the pool to a small number to protect us from Denial of Service (DoS) attacks; if we had our program create a new thread for each request as it came in, someone making 10 million requests to our server could create havoc by using up all our server’s resources and grinding the processing of requests to a halt.

Rather than spawning unlimited threads, then, we’ll have a fixed number of threads waiting in the pool. Requests that come in are sent to the pool for processing. The pool will maintain a queue of incoming requests. Each of the threads in the pool will pop off a request from this queue, handle the request, and then ask the queue for another request. With this design, we can process up to N requests concurrently, where N is the number of threads. If each thread is responding to a long-running request, subsequent requests can still back up in the queue, but we’ve increased the number of long-running requests we can handle before reaching that point.

This technique is just one of many ways to improve the throughput of a web server. Other options you might explore are the fork/join model, the single-threaded async I/O model, or the multi-threaded async I/O model. If you’re interested in this topic, you can read more about other solutions and try to implement them; with a low-level language like Rust, all of these options are possible.

Before we begin implementing a thread pool, let’s talk about what using the pool should look like. When you’re trying to design code, writing the client interface first can help guide your design. Write the API of the code so it’s structured in the way you want to call it; then implement the functionality within that structure rather than implementing the functionality and then designing the public API.

Similar to how we used test-driven development in the project in Chapter 12, we’ll use compiler-driven development here. We’ll write the code that calls the functions we want, and then we’ll look at errors from the compiler to determine what we should change next to get the code to work. Before we do that, however, we’ll explore the technique we’re not going to use as a starting point.

线程池是一组正在等待并准备好处理任务的派生线程。 当程序接收到一个新任务时，它会将池中的线程之一分配给该任务，并且该线程将处理该任务。 池中的剩余线程可用于处理第一个线程正在处理时进入的任何其他任务。 当第一个线程处理完其任务后，它会返回到空闲线程池，准备处理新任务。 线程池允许您同时处理连接，从而提高服务器的吞吐量。

我们将池中的线程数量限制为少量，以保护我们免受拒绝服务 (DoS) 攻击； 如果我们让程序为每个传入的请求创建一个新线程，那么某人向我们的服务器发出 1000 万个请求可能会耗尽我们所有服务器的资源并使请求处理停止，从而造成严重破坏。

那么，我们将有固定数量的线程在池中等待，而不是产生无限的线程。 传入的请求将发送到池中进行处理。 该池将维护传入请求的队列。 池中的每个线程都会从该队列中弹出一个请求，处理该请求，然后向队列询问另一个请求。 通过这种设计，我们最多可以同时处理 N 个请求，其中 N 是线程数。 如果每个线程都响应长时间运行的请求，则后续请求仍然可以在队列中备份，但我们增加了在到达该点之前可以处理的长时间运行请求的数量。

该技术只是提高 Web 服务器吞吐量的众多方法之一。 您可能探索的其他选项包括 fork/join 模型、单线程异步 I/O 模型或多线程异步 I/O 模型。 如果您对此主题感兴趣，可以阅读有关其他解决方案的更多信息并尝试实施它们； 使用 Rust 这样的低级语言，所有这些选项都是可能的。

在开始实现线程池之前，我们先讨论一下线程池的使用方式。 当您尝试设计代码时，首先编写客户端界面可以帮助指导您的设计。 编写代码的 API，使其按照您想要的调用方式构建； 然后在该结构中实现功能，而不是实现功能然后设计公共 API。

与我们在第 12 章的项目中使用测试驱动开发的方式类似，我们在这里将使用编译器驱动开发。 我们将编写调用我们想要的函数的代码，然后我们将查看编译器中的错误，以确定下一步应该更改哪些内容以使代码正常工作。 然而，在此之前，我们将探索我们不打算用作起点的技术。


### Spawning a Thread for Each Request
First, let’s explore how our code might look if it did create a new thread for every connection. As mentioned earlier, this isn’t our final plan due to the problems with potentially spawning an unlimited number of threads, but it is a starting point to get a working multithreaded server first. Then we’ll add the thread pool as an improvement, and contrasting the two solutions will be easier. Listing 20-11 shows the changes to make to main to spawn a new thread to handle each stream within the for loop.

首先，让我们探讨一下如果我们的代码为每个连接创建一个新线程，它会是什么样子。 如前所述，这不是我们的最终计划，因为可能会产生无限数量的线程，但它是首先获得工作多线程服务器的起点。 然后我们将添加线程池作为改进，对比这两种解决方案会更容易。 清单 20-11 显示了对 main 进行的更改，以生成一个新线程来处理 for 循环中的每个流。

Filename: src/main.rs
``````
fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();

    for stream in listener.incoming() {
        let stream = stream.unwrap();

        thread::spawn(|| {
            handle_connection(stream);
        });
    }
}
``````
Listing 20-11: Spawning a new thread for each stream

As you learned in Chapter 16, thread::spawn will create a new thread and then run the code in the closure in the new thread. If you run this code and load /sleep in your browser, then / in two more browser tabs, you’ll indeed see that the requests to / don’t have to wait for /sleep to finish. However, as we mentioned, this will eventually overwhelm the system because you’d be making new threads without any limit.

正如您在第 16 章中了解到的，thread::spawn 将创建一个新线程，然后在新线程中运行闭包中的代码。 如果您运行此代码并在浏览器中加载 /sleep，然后在另外两个浏览器选项卡中加载 /，您确实会看到对 / 的请求不必等待 /sleep 完成。 然而，正如我们所提到的，这最终将使系统不堪重负，因为您将无限制地创建新线程。

### Creating a Finite Number of Threads
We want our thread pool to work in a similar, familiar way so switching from threads to a thread pool doesn’t require large changes to the code that uses our API. Listing 20-12 shows the hypothetical interface for a ThreadPool struct we want to use instead of thread::spawn.

我们希望线程池以类似、熟悉的方式工作，因此从线程切换到线程池不需要对使用 API 的代码进行大量更改。 清单 20-12 显示了我们想要使用的 ThreadPool 结构的假设接口，而不是 thread::spawn。

Filename: src/main.rs
``````
This code does not compile!
fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
    let pool = ThreadPool::new(4);

    for stream in listener.incoming() {
        let stream = stream.unwrap();

        pool.execute(|| {
            handle_connection(stream);
        });
    }
}
``````
Listing 20-12: Our ideal ThreadPool interface

We use ThreadPool::new to create a new thread pool with a configurable number of threads, in this case four. Then, in the for loop, pool.execute has a similar interface as thread::spawn in that it takes a closure the pool should run for each stream. We need to implement pool.execute so it takes the closure and gives it to a thread in the pool to run. This code won’t yet compile, but we’ll try so the compiler can guide us in how to fix it.

我们使用 ThreadPool::new 创建一个具有可配置线程数量的新线程池，在本例中为四个。 然后，在 for 循环中，pool.execute 具有与 thread::spawn 类似的接口，因为它需要一个池应该为每个流运行的闭包。 我们需要实现 pool.execute，以便它获取闭包并将其交给池中的线程来运行。 该代码尚未编译，但我们会尝试，以便编译器可以指导我们如何修复它。


### Building ThreadPool Using Compiler Driven Development
Make the changes in Listing 20-12 to src/main.rs, and then let’s use the compiler errors from cargo check to drive our development. Here is the first error we get:
``````
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
error[E0433]: failed to resolve: use of undeclared type `ThreadPool`
  --> src/main.rs:11:16
   |
11 |     let pool = ThreadPool::new(4);
   |                ^^^^^^^^^^ use of undeclared type `ThreadPool`

For more information about this error, try `rustc --explain E0433`.
error: could not compile `hello` due to previous error
``````
Great! This error tells us we need a ThreadPool type or module, so we’ll build one now. Our ThreadPool implementation will be independent of the kind of work our web server is doing. So, let’s switch the hello crate from a binary crate to a library crate to hold our ThreadPool implementation. After we change to a library crate, we could also use the separate thread pool library for any work we want to do using a thread pool, not just for serving web requests.

伟大的！ 这个错误告诉我们我们需要一个 ThreadPool 类型或模块，所以我们现在就构建一个。 我们的 ThreadPool 实现将独立于我们的 Web 服务器正在执行的工作类型。 因此，让我们将 hello crate 从二进制 crate 切换为库 crate 以保存我们的 ThreadPool 实现。 更改为库箱后，我们还可以使用单独的线程池库来完成我们想要使用线程池完成的任何工作，而不仅仅是用于服务 Web 请求。

Create a src/lib.rs that contains the following, which is the simplest definition of a ThreadPool struct that we can have for now:

Filename: src/lib.rs
``````
pub struct ThreadPool;
``````
Then edit main.rs file to bring ThreadPool into scope from the library crate by adding the following code to the top of src/main.rs:

Filename: src/main.rs
``````
use hello::ThreadPool;
``````
This code still won’t work, but let’s check it again to get the next error that we need to address:
``````
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
error[E0599]: no function or associated item named `new` found for struct `ThreadPool` in the current scope
  --> src/main.rs:12:28
   |
12 |     let pool = ThreadPool::new(4);
   |                            ^^^ function or associated item not found in `ThreadPool`

For more information about this error, try `rustc --explain E0599`.
error: could not compile `hello` due to previous error
``````
This error indicates that next we need to create an associated function named new for ThreadPool. We also know that new needs to have one parameter that can accept 4 as an argument and should return a ThreadPool instance. Let’s implement the simplest new function that will have those characteristics:

这个错误表明接下来我们需要为ThreadPool创建一个名为new的关联函数。 我们还知道 new 需要有一个参数，该参数可以接受 4 作为参数，并且应该返回一个 ThreadPool 实例。 让我们实现具有这些特征的最简单的新函数：

Filename: src/lib.rs
``````
pub struct ThreadPool;

impl ThreadPool {
    pub fn new(size: usize) -> ThreadPool {
        ThreadPool
    }
}
``````
We chose usize as the type of the size parameter, because we know that a negative number of threads doesn’t make any sense. We also know we’ll use this 4 as the number of elements in a collection of threads, which is what the usize type is for, as discussed in the “Integer Types” section of Chapter 3.

我们选择 usize 作为 size 参数的类型，因为我们知道负数的线程没有任何意义。 我们还知道我们将使用 4 作为线程集合中的元素数量，这就是 usize 类型的用途，如第 3 章的“整数类型”部分所述。

Let’s check the code again:
``````
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
error[E0599]: no method named `execute` found for struct `ThreadPool` in the current scope
  --> src/main.rs:17:14
   |
17 |         pool.execute(|| {
   |              ^^^^^^^ method not found in `ThreadPool`

For more information about this error, try `rustc --explain E0599`.
error: could not compile `hello` due to previous error
``````
Now the error occurs because we don’t have an execute method on ThreadPool. Recall from the “Creating a Finite Number of Threads” section that we decided our thread pool should have an interface similar to thread::spawn. In addition, we’ll implement the execute function so it takes the closure it’s given and gives it to an idle thread in the pool to run.

We’ll define the execute method on ThreadPool to take a closure as a parameter. Recall from the “Moving Captured Values Out of the Closure and the Fn Traits” section in Chapter 13 that we can take closures as parameters with three different traits: Fn, FnMut, and FnOnce. We need to decide which kind of closure to use here. We know we’ll end up doing something similar to the standard library thread::spawn implementation, so we can look at what bounds the signature of thread::spawn has on its parameter. The documentation shows us the following:

现在发生错误是因为我们在 ThreadPool 上没有执行方法。 回想一下“创建有限数量的线程”部分，我们决定线程池应该有一个类似于 thread::spawn 的接口。 此外，我们将实现执行函数，以便它获取给定的闭包并将其提供给池中的空闲线程来运行。

我们将在 ThreadPool 上定义执行方法以将闭包作为参数。 回想一下第 13 章中的“将捕获的值移出闭包和 Fn 特征”部分，我们可以将闭包作为具有三种不同特征的参数：Fn、FnMut 和 FnOnce。 我们需要决定在这里使用哪种闭包。 我们知道我们最终会做一些类似于标准库 thread::spawn 实现的事情，所以我们可以看看 thread::spawn 的签名对其参数的限制。 该文档向我们展示了以下内容：

``````
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
``````
The F type parameter is the one we’re concerned with here; the T type parameter is related to the return value, and we’re not concerned with that. We can see that spawn uses FnOnce as the trait bound on F. This is probably what we want as well, because we’ll eventually pass the argument we get in execute to spawn. We can be further confident that FnOnce is the trait we want to use because the thread for running a request will only execute that request’s closure one time, which matches the Once in FnOnce.

The F type parameter also has the trait bound Send and the lifetime bound 'static, which are useful in our situation: we need Send to transfer the closure from one thread to another and 'static because we don’t know how long the thread will take to execute. Let’s create an execute method on ThreadPool that will take a generic parameter of type F with these bounds:

F类型参数是我们这里关心的； T类型参数与返回值有关，我们不关心这个。 我们可以看到spawn使用FnOnce作为F上的trait绑定。这可能也是我们想要的，因为我们最终会将execute中获得的参数传递给spawn。 我们可以进一步确信 FnOnce 是我们想要使用的特征，因为运行请求的线程只会执行该请求的闭包一次，这与 FnOnce 中的 Once 匹配。

F 类型参数还具有特征绑定 Send 和生命周期绑定 'static，这在我们的情况下很有用：我们需要 Send 将闭包从一个线程传输到另一个线程，并且需要 'static ，因为我们不知道线程将持续多长时间 采取执行。 让我们在 ThreadPool 上创建一个执行方法，该方法将采用具有以下边界的 F 类型泛型参数：

Filename: src/lib.rs
``````
impl ThreadPool {
    // --snip--
    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
    }
}
``````
We still use the () after FnOnce because this FnOnce represents a closure that takes no parameters and returns the unit type (). Just like function definitions, the return type can be omitted from the signature, but even if we have no parameters, we still need the parentheses.

Again, this is the simplest implementation of the execute method: it does nothing, but we’re trying only to make our code compile. Let’s check it again:

我们仍然在 FnOnce 之后使用 ()，因为这个 FnOnce 代表一个不带参数并返回单位类型 () 的闭包。 就像函数定义一样，签名中可以省略返回类型，但即使我们没有参数，我们仍然需要括号。

同样，这是执行方法最简单的实现：它什么也不做，但我们只是尝试让我们的代码编译。 我们再检查一下：

``````
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
    Finished dev [unoptimized + debuginfo] target(s) in 0.24s
``````
It compiles! But note that if you try cargo run and make a request in the browser, you’ll see the errors in the browser that we saw at the beginning of the chapter. Our library isn’t actually calling the closure passed to execute yet!

Note: A saying you might hear about languages with strict compilers, such as Haskell and Rust, is “if the code compiles, it works.” But this saying is not universally true. Our project compiles, but it does absolutely nothing! If we were building a real, complete project, this would be a good time to start writing unit tests to check that the code compiles and has the behavior we want.

它编译了！ 但请注意，如果您尝试运行货物并在浏览器中发出请求，您将在浏览器中看到我们在本章开头看到的错误。 我们的库实际上还没有调用传递来执行的闭包！

注意：您可能听说过关于具有严格编译器的语言（例如 Haskell 和 Rust）的一句话是“如果代码可以编译，它就可以工作”。 但这句话并不普遍正确。 我们的项目可以编译，但它什么也没做！ 如果我们正在构建一个真实的、完整的项目，那么这将是开始编写单元测试以检查代码是否编译并具有我们想要的行为的好时机。

### Validating the Number of Threads in new
We aren’t doing anything with the parameters to new and execute. Let’s implement the bodies of these functions with the behavior we want. To start, let’s think about new. Earlier we chose an unsigned type for the size parameter, because a pool with a negative number of threads makes no sense. However, a pool with zero threads also makes no sense, yet zero is a perfectly valid usize. We’ll add code to check that size is greater than zero before we return a ThreadPool instance and have the program panic if it receives a zero by using the assert! macro, as shown in Listing 20-13.

我们没有对 new 和execute 的参数做任何事情。 让我们用我们想要的行为来实现这些函数的主体。 首先，让我们考虑一下新的。 之前我们为大小参数选择了无符号类型，因为线程数为负数的池没有意义。 然而，具有零线程的池也没有意义，但零是完全有效的使用。 在返回 ThreadPool 实例之前，我们将添加代码来检查大小是否大于零，并且如果使用断言收到零，则程序会发生恐慌！ 宏，如清单20-13所示。

Filename: src/lib.rs
``````
impl ThreadPool {
    /// Create a new ThreadPool.
    ///
    /// The size is the number of threads in the pool.
    ///
    /// # Panics
    ///
    /// The `new` function will panic if the size is zero.
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        ThreadPool
    }

    // --snip--
}
``````
Listing 20-13: Implementing ThreadPool::new to panic if size is zero

We’ve also added some documentation for our ThreadPool with doc comments. Note that we followed good documentation practices by adding a section that calls out the situations in which our function can panic, as discussed in Chapter 14. Try running cargo doc --open and clicking the ThreadPool struct to see what the generated docs for new look like!

Instead of adding the assert! macro as we’ve done here, we could change new into build and return a Result like we did with Config::build in the I/O project in Listing 12-9. But we’ve decided in this case that trying to create a thread pool without any threads should be an unrecoverable error. If you’re feeling ambitious, try to write a function named build with the following signature to compare with the new function:

我们还为我们的线程池添加了一些带有文档注释的文档。 请注意，我们遵循了良好的文档实践，添加了一个部分来指出我们的函数可能会出现恐慌的情况，如第 14 章中所讨论的。尝试运行 Cargo doc --open 并单击 ThreadPool 结构以查看生成的文档的新外观 喜欢！

而不是添加断言！ 正如我们在这里所做的那样，我们可以将 new 更改为 build 并返回一个结果，就像我们在清单 12-9 中的 I/O 项目中使用 Config::build 所做的那样。 但在这种情况下，我们决定尝试创建一个没有任何线程的线程池应该是一个不可恢复的错误。 如果您雄心勃勃，请尝试编写一个名为 build 的函数，并使用以下签名来与新函数进行比较：

``````
pub fn build(size: usize) -> Result<ThreadPool, PoolCreationError> {
``````
### Creating Space to Store the Threads
Now that we have a way to know we have a valid number of threads to store in the pool, we can create those threads and store them in the ThreadPool struct before returning the struct. But how do we “store” a thread? Let’s take another look at the thread::spawn signature:

现在我们有办法知道我们有有效数量的线程存储在池中，我们可以创建这些线程并将它们存储在 ThreadPool 结构中，然后返回该结构。 但是我们如何“存储”线程呢？ 让我们再看一下 thread::spawn 签名：

``````
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
``````
The spawn function returns a JoinHandle<T>, where T is the type that the closure returns. Let’s try using JoinHandle too and see what happens. In our case, the closures we’re passing to the thread pool will handle the connection and not return anything, so T will be the unit type ().

The code in Listing 20-14 will compile but doesn’t create any threads yet. We’ve changed the definition of ThreadPool to hold a vector of thread::JoinHandle<()> instances, initialized the vector with a capacity of size, set up a for loop that will run some code to create the threads, and returned a ThreadPool instance containing them.

spawn 函数返回一个 JoinHandle<T>，其中 T 是闭包返回的类型。 我们也尝试使用 JoinHandle 看看会发生什么。 在我们的例子中，我们传递给线程池的闭包将处理连接并且不返回任何内容，因此 T 将是单元类型 ()。

清单 20-14 中的代码将编译，但尚未创建任何线程。 我们更改了 ThreadPool 的定义以保存 thread::JoinHandle<()> 实例的向量，用 size 的容量初始化该向量，设置一个 for 循环，该循环将运行一些代码来创建线程，并返回一个 包含它们的 ThreadPool 实例。

Filename: src/lib.rs
``````
This code does not produce the desired behavior.
use std::thread;

pub struct ThreadPool {
    threads: Vec<thread::JoinHandle<()>>,
}

impl ThreadPool {
    // --snip--
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let mut threads = Vec::with_capacity(size);

        for _ in 0..size {
            // create some threads and store them in the vector
        }

        ThreadPool { threads }
    }
    // --snip--
}
``````
Listing 20-14: Creating a vector for ThreadPool to hold the threads

We’ve brought std::thread into scope in the library crate, because we’re using thread::JoinHandle as the type of the items in the vector in ThreadPool.

Once a valid size is received, our ThreadPool creates a new vector that can hold size items. The with_capacity function performs the same task as Vec::new but with an important difference: it preallocates space in the vector. Because we know we need to store size elements in the vector, doing this allocation up front is slightly more efficient than using Vec::new, which resizes itself as elements are inserted.

When you run cargo check again, it should succeed.

我们已将 std::thread 纳入库箱的范围内，因为我们使用 thread::JoinHandle 作为 ThreadPool 中向量中项目的类型。

一旦收到有效的大小，我们的线程池就会创建一个可以容纳大小项的新向量。 with_capacity 函数执行与 Vec::new 相同的任务，但有一个重要的区别：它在向量中预先分配空间。 因为我们知道需要在向量中存储 size 元素，所以预先进行此分配比使用 Vec::new 稍微更有效，后者会在插入元素时调整自身大小。

当您再次运行货物检查时，它应该会成功。

### A Worker Struct Responsible for Sending Code from the ThreadPool to a Thread
We left a comment in the for loop in Listing 20-14 regarding the creation of threads. Here, we’ll look at how we actually create threads. The standard library provides thread::spawn as a way to create threads, and thread::spawn expects to get some code the thread should run as soon as the thread is created. However, in our case, we want to create the threads and have them wait for code that we’ll send later. The standard library’s implementation of threads doesn’t include any way to do that; we have to implement it manually.

We’ll implement this behavior by introducing a new data structure between the ThreadPool and the threads that will manage this new behavior. We’ll call this data structure Worker, which is a common term in pooling implementations. The Worker picks up code that needs to be run and runs the code in the Worker’s thread. Think of people working in the kitchen at a restaurant: the workers wait until orders come in from customers, and then they’re responsible for taking those orders and fulfilling them.

Instead of storing a vector of JoinHandle<()> instances in the thread pool, we’ll store instances of the Worker struct. Each Worker will store a single JoinHandle<()> instance. Then we’ll implement a method on Worker that will take a closure of code to run and send it to the already running thread for execution. We’ll also give each worker an id so we can distinguish between the different workers in the pool when logging or debugging.

Here is the new process that will happen when we create a ThreadPool. We’ll implement the code that sends the closure to the thread after we have Worker set up in this way:

+ Define a Worker struct that holds an id and a JoinHandle<()>.
+ Change ThreadPool to hold a vector of Worker instances.
+ Define a Worker::new function that takes an id number and returns a Worker instance that holds + the id and a thread spawned with an empty closure.
+ In ThreadPool::new, use the for loop counter to generate an id, create a new Worker with that id, and store the worker in the vector.

If you’re up for a challenge, try implementing these changes on your own before looking at the code in Listing 20-15.

Ready? Here is Listing 20-15 with one way to make the preceding modifications.

我们在清单 20-14 的 for 循环中留下了关于线程创建的注释。 在这里，我们将看看如何实际创建线程。 标准库提供了 thread::spawn 作为创建线程的方式，并且 thread::spawn 期望在创建线程后立即获取线程应该运行的一些代码。 然而，在我们的例子中，我们想要创建线程并让它们等待我们稍后发送的代码。 标准库的线程实现不包含任何方法来做到这一点； 我们必须手动实现它。

我们将通过在 ThreadPool 和管理此新行为的线程之间引入新的数据结构来实现此行为。 我们将这种数据结构称为 Worker，这是池化实现中的常用术语。 Worker 选取需要运行的代码，并在 Worker 的线程中运行该代码。 想象一下在餐厅厨房工作的人：工人们等待顾客下单，然后负责接受并履行这些订单。

我们将存储 Worker 结构体的实例，而不是在线程池中存储 JoinHandle<()> 实例的向量。 每个 Worker 将存储一个 JoinHandle<()> 实例。 然后我们将在 Worker 上实现一个方法，该方法将获取要运行的代码闭包并将其发送到已经运行的线程来执行。 我们还将为每个工作人员提供一个 ID，以便我们在记录或调试时可以区分池中的不同工作人员。

这是当我们创建线程池时将发生的新进程。 在以这种方式设置 Worker 后，我们将实现将闭包发送到线程的代码：

+ 定义一个 Worker 结构体，其中包含一个 id 和一个 JoinHandle<()>。
+ 更改 ThreadPool 以保存 Worker 实例的向量。
+ 定义一个 Worker::new 函数，它接受一个 id 号并返回一个 Worker 实例，该实例保存该 id 和一个用空闭包生成的线程。
+ 在ThreadPool::new中，使用for循环计数器生成一个id，使用该id创建一个新的Worker，并将该worker存储在向量中。

如果您准备迎接挑战，请在查看清单 20-15 中的代码之前尝试自己实现这些更改。

准备好？ 清单 20-15 提供了进行上述修改的一种方法。

Filename: src/lib.rs
``````
use std::thread;

pub struct ThreadPool {
    workers: Vec<Worker>,
}

impl ThreadPool {
    // --snip--
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let mut workers = Vec::with_capacity(size);

        for id in 0..size {
            workers.push(Worker::new(id));
        }

        ThreadPool { workers }
    }
    // --snip--
}

struct Worker {
    id: usize,
    thread: thread::JoinHandle<()>,
}

impl Worker {
    fn new(id: usize) -> Worker {
        let thread = thread::spawn(|| {});

        Worker { id, thread }
    }
}
``````
Listing 20-15: Modifying ThreadPool to hold Worker instances instead of holding threads directly

We’ve changed the name of the field on ThreadPool from threads to workers because it’s now holding Worker instances instead of JoinHandle<()> instances. We use the counter in the for loop as an argument to Worker::new, and we store each new Worker in the vector named workers.

External code (like our server in src/main.rs) doesn’t need to know the implementation details regarding using a Worker struct within ThreadPool, so we make the Worker struct and its new function private. The Worker::new function uses the id we give it and stores a JoinHandle<()> instance that is created by spawning a new thread using an empty closure.
``````
Note: If the operating system can’t create a thread because there aren’t enough system resources, thread::spawn will panic. That will cause our whole server to panic, even though the creation of some threads might succeed. For simplicity’s sake, this behavior is fine, but in a production thread pool implementation, you’d likely want to use std::thread::Builder and its spawn method that returns Result instead.
``````

This code will compile and will store the number of Worker instances we specified as an argument to ThreadPool::new. But we’re still not processing the closure that we get in execute. Let’s look at how to do that next.

我们已将 ThreadPool 上的字段名称从“threads”更改为“workers”，因为它现在保存的是 Worker 实例，而不是 JoinHandle<()> 实例。 我们使用 for 循环中的计数器作为 Worker::new 的参数，并将每个新 Worker 存储在名为workers 的向量中。

外部代码（如 src/main.rs 中的服务器）不需要知道有关在 ThreadPool 中使用 Worker 结构的实现细节，因此我们将 Worker 结构及其新函数设为私有。 Worker::new 函数使用我们给它的 id 并存储一个 JoinHandle<()> 实例，该实例是通过使用空闭包生成一个新线程而创建的。
``````
注意：如果操作系统由于没有足够的系统资源而无法创建线程，thread::spawn 将会出现紧急情况。 即使某些线程的创建可能成功，这也会导致整个服务器出现恐慌。 为了简单起见，这种行为很好，但在生产线程池实现中，您可能希望使用 std::thread::Builder 及其返回 Result 的 spawn 方法。
``````

该代码将编译并存储我们指定为 ThreadPool::new 参数的 Worker 实例数量。 但我们仍然没有处理执行中得到的闭包。 接下来让我们看看如何做到这一点。

### Sending Requests to Threads via Channels
The next problem we’ll tackle is that the closures given to thread::spawn do absolutely nothing. Currently, we get the closure we want to execute in the execute method. But we need to give thread::spawn a closure to run when we create each Worker during the creation of the ThreadPool.

We want the Worker structs that we just created to fetch the code to run from a queue held in the ThreadPool and send that code to its thread to run.

The channels we learned about in Chapter 16—a simple way to communicate between two threads—would be perfect for this use case. We’ll use a channel to function as the queue of jobs, and execute will send a job from the ThreadPool to the Worker instances, which will send the job to its thread. Here is the plan:

1. The ThreadPool will create a channel and hold on to the sender.
2. Each Worker will hold on to the receiver.
3.  We’ll create a new Job struct that will hold the closures we want to send down the channel.
4. The execute method will send the job it wants to execute through the sender.
5. In its thread, the Worker will loop over its receiver and execute the closures of any jobs it receives.

Let’s start by creating a channel in ThreadPool::new and holding the sender in the ThreadPool instance, as shown in Listing 20-16. The Job struct doesn’t hold anything for now but will be the type of item we’re sending down the channel.

我们要解决的下一个问题是给 thread::spawn 的闭包绝对不做任何事情。 目前，我们在execute方法中得到了想要执行的闭包。 但是，当我们在创建 ThreadPool 期间创建每个 Worker 时，我们需要给 thread::spawn 一个要运行的闭包。

我们希望刚刚创建的 Worker 结构从 ThreadPool 中保存的队列中获取要运行的代码，并将该代码发送到其线程来运行。

我们在第 16 章中了解到的通道（一种在两个线程之间进行通信的简单方法）非常适合此用例。 我们将使用通道作为作业队列，并执行将作业从 ThreadPool 发送到 Worker 实例，Worker 实例将作业发送到其线程。 这是计划：

1. ThreadPool 将创建一个通道并保留发送者。
2. 每个工人将握住接收器。
3. 我们将创建一个新的 Job 结构，它将保存我们想要沿着通道发送的闭包。
4.execute方法会通过发送者发送它想要执行的作业。
5. 在其线程中，Worker 将循环其接收器并执行其接收到的任何作业的关闭。

首先，我们在 ThreadPool::new 中创建一个通道，并将发送者保存在 ThreadPool 实例中，如清单 20-16 所示。 Job 结构目前不包含任何内容，但将是我们沿着通道发送的项目类型。

Filename: src/lib.rs
``````
use std::{sync::mpsc, thread};

pub struct ThreadPool {
    workers: Vec<Worker>,
    sender: mpsc::Sender<Job>,
}

struct Job;

impl ThreadPool {
    // --snip--
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let (sender, receiver) = mpsc::channel();

        let mut workers = Vec::with_capacity(size);

        for id in 0..size {
            workers.push(Worker::new(id));
        }

        ThreadPool { workers, sender }
    }
    // --snip--
}
``````
Listing 20-16: Modifying ThreadPool to store the sender of a channel that transmits Job instances

In ThreadPool::new, we create our new channel and have the pool hold the sender. This will successfully compile.

Let’s try passing a receiver of the channel into each worker as the thread pool creates the channel. We know we want to use the receiver in the thread that the workers spawn, so we’ll reference the receiver parameter in the closure. The code in Listing 20-17 won’t quite compile yet.

在 ThreadPool::new 中，我们创建新通道并让池保存发送者。 这样就可以编译成功了。

让我们尝试在线程池创建通道时将通道的接收器传递给每个工作线程。 我们知道我们想要在工作线程生成的线程中使用接收器，因此我们将在闭包中引用接收器参数。 清单 20-17 中的代码还不能完全编译。

Filename: src/lib.rs
``````
This code does not compile!
impl ThreadPool {
    // --snip--
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let (sender, receiver) = mpsc::channel();

        let mut workers = Vec::with_capacity(size);

        for id in 0..size {
            workers.push(Worker::new(id, receiver));
        }

        ThreadPool { workers, sender }
    }
    // --snip--
}

// --snip--

impl Worker {
    fn new(id: usize, receiver: mpsc::Receiver<Job>) -> Worker {
        let thread = thread::spawn(|| {
            receiver;
        });

        Worker { id, thread }
    }
}
``````
Listing 20-17: Passing the receiver to the workers

We’ve made some small and straightforward changes: we pass the receiver into Worker::new, and then we use it inside the closure.

When we try to check this code, we get this error:
``````
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
error[E0382]: use of moved value: `receiver`
  --> src/lib.rs:26:42
   |
21 |         let (sender, receiver) = mpsc::channel();
   |                      -------- move occurs because `receiver` has type `std::sync::mpsc::Receiver<Job>`, which does not implement the `Copy` trait
...
26 |             workers.push(Worker::new(id, receiver));
   |                                          ^^^^^^^^ value moved here, in previous iteration of loop

For more information about this error, try `rustc --explain E0382`.
error: could not compile `hello` due to previous error
``````
The code is trying to pass receiver to multiple Worker instances. This won’t work, as you’ll recall from Chapter 16: the channel implementation that Rust provides is multiple producer, single consumer. This means we can’t just clone the consuming end of the channel to fix this code. We also don’t want to send a message multiple times to multiple consumers; we want one list of messages with multiple workers such that each message gets processed once.

Additionally, taking a job off the channel queue involves mutating the receiver, so the threads need a safe way to share and modify receiver; otherwise, we might get race conditions (as covered in Chapter 16).

Recall the thread-safe smart pointers discussed in Chapter 16: to share ownership across multiple threads and allow the threads to mutate the value, we need to use Arc<Mutex<T>>. The Arc type will let multiple workers own the receiver, and Mutex will ensure that only one worker gets a job from the receiver at a time. Listing 20-18 shows the changes we need to make.

该代码尝试将接收器传递给多个 Worker 实例。 这是行不通的，你会记得第 16 章：Rust 提供的通道实现是多个生产者、单个消费者。 这意味着我们不能仅仅克隆通道的消费端来修复此代码。 我们也不希望向多个消费者多次发送消息； 我们想要一个包含多个工作人员的消息列表，以便每条消息都被处理一次。

此外，从通道队列中取出作业涉及改变接收器，因此线程需要一种安全的方法来共享和修改接收器； 否则，我们可能会遇到竞争条件（如第 16 章所述）。

回想一下第 16 章中讨论的线程安全智能指针：为了在多个线程之间共享所有权并允许线程改变值，我们需要使用 Arc<Mutex<T>>。 Arc类型将让多个worker拥有接收器，而Mutex将确保一次只有一个worker从接收器获得工作。 清单 20-18 显示了我们需要进行的更改。

Filename: src/lib.rs
``````
use std::{
    sync::{mpsc, Arc, Mutex},
    thread,
};
// --snip--

impl ThreadPool {
    // --snip--
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let (sender, receiver) = mpsc::channel();

        let receiver = Arc::new(Mutex::new(receiver));

        let mut workers = Vec::with_capacity(size);

        for id in 0..size {
            workers.push(Worker::new(id, Arc::clone(&receiver)));
        }

        ThreadPool { workers, sender }
    }

    // --snip--
}

// --snip--

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        // --snip--
    }
}
``````
Listing 20-18: Sharing the receiver among the workers using Arc and Mutex

In ThreadPool::new, we put the receiver in an Arc and a Mutex. For each new worker, we clone the Arc to bump the reference count so the workers can share ownership of the receiver.

With these changes, the code compiles! We’re getting there!

在 ThreadPool::new 中，我们将接收器放入 Arc 和 Mutex 中。 对于每个新的工作人员，我们克隆 Arc 以增加引用计数，以便工作人员可以共享接收器的所有权。

通过这些更改，代码可以编译了！ 我们快到了！

### Implementing the execute Method
Let’s finally implement the execute method on ThreadPool. We’ll also change Job from a struct to a type alias for a trait object that holds the type of closure that execute receives. As discussed in the “Creating Type Synonyms with Type Aliases” section of Chapter 19, type aliases allow us to make long types shorter for ease of use. Look at Listing 20-19.

最后我们来实现ThreadPool上的execute方法。 我们还将 Job 从结构体更改为特征对象的类型别名，该特征对象保存执行接收的闭包类型。 正如第 19 章“使用类型别名创建类型同义词”部分所讨论的，类型别名允许我们将长类型缩短以方便使用。 请看清单 20-19。

Filename: src/lib.rs
``````
// --snip--

type Job = Box<dyn FnOnce() + Send + 'static>;

impl ThreadPool {
    // --snip--

    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
        let job = Box::new(f);

        self.sender.send(job).unwrap();
    }
}

// --snip--
``````
Listing 20-19: Creating a Job type alias for a Box that holds each closure and then sending the job down the channel

After creating a new Job instance using the closure we get in execute, we send that job down the sending end of the channel. We’re calling unwrap on send for the case that sending fails. This might happen if, for example, we stop all our threads from executing, meaning the receiving end has stopped receiving new messages. At the moment, we can’t stop our threads from executing: our threads continue executing as long as the pool exists. The reason we use unwrap is that we know the failure case won’t happen, but the compiler doesn’t know that.

But we’re not quite done yet! In the worker, our closure being passed to thread::spawn still only references the receiving end of the channel. Instead, we need the closure to loop forever, asking the receiving end of the channel for a job and running the job when it gets one. Let’s make the change shown in Listing 20-20 to Worker::new.

使用执行中的闭包创建新的 Job 实例后，我们将该作业发送到通道的发送端。 对于发送失败的情况，我们会在发送时调用 unwrap。 例如，如果我们停止所有线程的执行，这意味着接收端已停止接收新消息，则可能会发生这种情况。 目前，我们无法阻止线程执行：只要池存在，我们的线程就会继续执行。 我们使用 unwrap 的原因是我们知道失败情况不会发生，但编译器不知道。

但我们还没有完成！ 在工作线程中，传递给 thread::spawn 的闭包仍然只引用通道的接收端。 相反，我们需要闭包永远循环，向通道的接收端请求一项作业，并在收到作业时运行该作业。 让我们将清单 20-20 中所示的更改更改为 Worker::new。

Filename: src/lib.rs
``````
// --snip--

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        let thread = thread::spawn(move || loop {
            let job = receiver.lock().unwrap().recv().unwrap();

            println!("Worker {id} got a job; executing.");

            job();
        });

        Worker { id, thread }
    }
}
``````
Listing 20-20: Receiving and executing the jobs in the worker’s thread

Here, we first call lock on the receiver to acquire the mutex, and then we call unwrap to panic on any errors. Acquiring a lock might fail if the mutex is in a poisoned state, which can happen if some other thread panicked while holding the lock rather than releasing the lock. In this situation, calling unwrap to have this thread panic is the correct action to take. Feel free to change this unwrap to an expect with an error message that is meaningful to you.

If we get the lock on the mutex, we call recv to receive a Job from the channel. A final unwrap moves past any errors here as well, which might occur if the thread holding the sender has shut down, similar to how the send method returns Err if the receiver shuts down.

The call to recv blocks, so if there is no job yet, the current thread will wait until a job becomes available. The Mutex<T> ensures that only one Worker thread at a time is trying to request a job.

Our thread pool is now in a working state! Give it a cargo run and make some requests:

在这里，我们首先调用接收器上的 lock 来获取互斥体，然后调用 unwrap 来对任何错误进行恐慌。 如果互斥体处于中毒状态，则获取锁可能会失败，如果其他线程在持有锁而不是释放锁时发生恐慌，则可能会发生这种情况。 在这种情况下，调用 unwrap 以使该线程发生恐慌是正确的操作。 请随意将此展开更改为带有对您有意义的错误消息的期望。

如果我们获得了互斥体的锁，我们就调用recv从通道接收Job。 最后的解包也会消除此处的所有错误，如果保存发送方的线程已关闭，则可能会发生这种情况，类似于如果接收方关闭则 send 方法返回 Err 的方式。

对 recv 的调用会阻塞，因此如果还没有作业，当前线程将等待，直到有作业可用。 Mutex<T> 确保一次只有一个工作线程尝试请求作业。

我们的线程池现在处于工作状态！ 让它运行并提出一些要求：

``````
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
warning: field is never read: `workers`
 --> src/lib.rs:7:5
  |
7 |     workers: Vec<Worker>,
  |     ^^^^^^^^^^^^^^^^^^^^
  |
  = note: `#[warn(dead_code)]` on by default

warning: field is never read: `id`
  --> src/lib.rs:48:5
   |
48 |     id: usize,
   |     ^^^^^^^^^

warning: field is never read: `thread`
  --> src/lib.rs:49:5
   |
49 |     thread: thread::JoinHandle<()>,
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

warning: `hello` (lib) generated 3 warnings
    Finished dev [unoptimized + debuginfo] target(s) in 1.40s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
``````
Success! We now have a thread pool that executes connections asynchronously. There are never more than four threads created, so our system won’t get overloaded if the server receives a lot of requests. If we make a request to /sleep, the server will be able to serve other requests by having another thread run them.

Note: if you open /sleep in multiple browser windows simultaneously, they might load one at a time in 5 second intervals. Some web browsers execute multiple instances of the same request sequentially for caching reasons. This limitation is not caused by our web server.

After learning about the while let loop in Chapter 18, you might be wondering why we didn’t write the worker thread code as shown in Listing 20-21.

成功！ 我们现在有一个异步执行连接的线程池。 创建的线程永远不会超过四个，因此如果服务器收到大量请求，我们的系统不会过载。 如果我们向 /sleep 发出请求，服务器将能够通过让另一个线程运行其他请求来服务它们。

注意：如果您同时在多个浏览器窗口中打开 /sleep，它们可能会以 5 秒的间隔一次加载一个。 出于缓存原因，某些 Web 浏览器会顺序执行同一请求的多个实例。 此限制不是由我们的网络服务器造成的。

在学习了第 18 章中的 while let 循环之后，你可能想知道为什么我们没有编写如清单 20-21 所示的工作线程代码。

Filename: src/lib.rs
``````
This code does not produce the desired behavior.
// --snip--

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        let thread = thread::spawn(move || {
            while let Ok(job) = receiver.lock().unwrap().recv() {
                println!("Worker {id} got a job; executing.");

                job();
            }
        });

        Worker { id, thread }
    }
}
``````
Listing 20-21: An alternative implementation of Worker::new using while let

This code compiles and runs but doesn’t result in the desired threading behavior: a slow request will still cause other requests to wait to be processed. The reason is somewhat subtle: the Mutex struct has no public unlock method because the ownership of the lock is based on the lifetime of the MutexGuard<T> within the LockResult<MutexGuard<T>> that the lock method returns. At compile time, the borrow checker can then enforce the rule that a resource guarded by a Mutex cannot be accessed unless we hold the lock. However, this implementation can also result in the lock being held longer than intended if we aren’t mindful of the lifetime of the MutexGuard<T>.

The code in Listing 20-20 that uses let job = receiver.lock().unwrap().recv().unwrap(); works because with let, any temporary values used in the expression on the right hand side of the equals sign are immediately dropped when the let statement ends. However, while let (and if let and match) does not drop temporary values until the end of the associated block. In Listing 20-21, the lock remains held for the duration of the call to job(), meaning other workers cannot receive jobs.

此代码可以编译并运行，但不会产生所需的线程行为：缓慢的请求仍会导致其他请求等待处理。 原因有些微妙：Mutex 结构没有公共解锁方法，因为锁的所有权基于 lock 方法返回的 LockResult<MutexGuard<T>> 内 MutexGuard<T> 的生命周期。 在编译时，借用检查器可以强制执行这样的规则：除非我们持有锁，否则无法访问由互斥锁保护的资源。 然而，如果我们不注意 MutexGuard<T> 的生命周期，这种实现也可能导致锁的持有时间比预期的要长。

清单 20-20 中的代码使用了 let job = receive.lock().unwrap().recv().unwrap(); 之所以有效，是因为使用 let 时，等号右侧表达式中使用的任何临时值都会在 let 语句结束时立即删除。 但是， while let （以及 if let 和 match）在关联块结束之前不会删除临时值。 在清单 20-21 中，锁在调用 job() 期间一直保持，这意味着其他工作线程无法接收作业。

## 20.3 Graceful Shutdown and Cleanup

The code in Listing 20-20 is responding to requests asynchronously through the use of a thread pool, as we intended. We get some warnings about the workers, id, and thread fields that we’re not using in a direct way that reminds us we’re not cleaning up anything. When we use the less elegant ctrl-c method to halt the main thread, all other threads are stopped immediately as well, even if they’re in the middle of serving a request.

Next, then, we’ll implement the Drop trait to call join on each of the threads in the pool so they can finish the requests they’re working on before closing. Then we’ll implement a way to tell the threads they should stop accepting new requests and shut down. To see this code in action, we’ll modify our server to accept only two requests before gracefully shutting down its thread pool.

清单 20-20 中的代码按照我们的预期通过使用线程池异步响应请求。 我们收到一些关于工人、id 和线程字段的警告，我们没有直接使用这些字段，提醒我们没有清理任何东西。 当我们使用不太优雅的 ctrl-c 方法来停止主线程时，所有其他线程也会立即停止，即使它们正在处理请求。

接下来，我们将实现 Drop 特征来调用池中每个线程的 join ，以便它们可以在关闭之前完成正在处理的请求。 然后我们将实现一种方法来告诉线程它们应该停止接受新请求并关闭。 为了查看此代码的实际效果，我们将修改服务器以在正常关闭其线程池之前仅接受两个请求。

### Implementing the Drop Trait on ThreadPool
Let’s start with implementing Drop on our thread pool. When the pool is dropped, our threads should all join to make sure they finish their work. Listing 20-22 shows a first attempt at a Drop implementation; this code won’t quite work yet.

让我们从在线程池上实现 Drop 开始。 当池被删除时，我们的线程应该全部加入以确保它们完成工作。 清单 20-22 显示了 Drop 实现的第一次尝试； 这段代码还不能正常工作。

Filename: src/lib.rs
``````
This code does not compile!
impl Drop for ThreadPool {
    fn drop(&mut self) {
        for worker in &mut self.workers {
            println!("Shutting down worker {}", worker.id);

            worker.thread.join().unwrap();
        }
    }
}
``````
Listing 20-22: Joining each thread when the thread pool goes out of scope

First, we loop through each of the thread pool workers. We use &mut for this because self is a mutable reference, and we also need to be able to mutate worker. For each worker, we print a message saying that this particular worker is shutting down, and then we call join on that worker’s thread. If the call to join fails, we use unwrap to make Rust panic and go into an ungraceful shutdown.

首先，我们循环遍历每个线程池工作线程。 我们为此使用 &mut 是因为 self 是一个可变引用，并且我们还需要能够改变worker。 对于每个工作人员，我们打印一条消息，说明该特定工作人员正在关闭，然后我们在该工作人员的线程上调用 join。 如果 join 调用失败，我们使用 unwrap 使 Rust 恐慌并进入不正常的关闭状态。

Here is the error we get when we compile this code:
``````
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
error[E0507]: cannot move out of `worker.thread` which is behind a mutable reference
  --> src/lib.rs:52:13
   |
52 |             worker.thread.join().unwrap();
   |             ^^^^^^^^^^^^^ ------ `worker.thread` moved due to this method call
   |             |
   |             move occurs because `worker.thread` has type `JoinHandle<()>`, which does not implement the `Copy` trait
   |
note: this function takes ownership of the receiver `self`, which moves `worker.thread`
  --> /rustc/d5a82bbd26e1ad8b7401f6a718a9c57c96905483/library/std/src/thread/mod.rs:1581:17

For more information about this error, try `rustc --explain E0507`.
error: could not compile `hello` due to previous error
``````
The error tells us we can’t call join because we only have a mutable borrow of each worker and join takes ownership of its argument. To solve this issue, we need to move the thread out of the Worker instance that owns thread so join can consume the thread. We did this in Listing 17-15: if Worker holds an Option<thread::JoinHandle<()>> instead, we can call the take method on the Option to move the value out of the Some variant and leave a None variant in its place. In other words, a Worker that is running will have a Some variant in thread, and when we want to clean up a Worker, we’ll replace Some with None so the Worker doesn’t have a thread to run.

首先，我们循环遍历每个线程池工作线程。 我们为此使用 &mut 是因为 self 是一个可变引用，并且我们还需要能够改变worker。 对于每个工作人员，我们打印一条消息，说明该特定工作人员正在关闭，然后我们在该工作人员的线程上调用 join。 如果 join 调用失败，我们使用 unwrap 使 Rust 恐慌并进入不正常的关闭状态。

So we know we want to update the definition of Worker like this:

Filename: src/lib.rs
``````
This code does not compile!
struct Worker {
    id: usize,
    thread: Option<thread::JoinHandle<()>>,
}
``````
Now let’s lean on the compiler to find the other places that need to change. Checking this code, we get two errors:
``````
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
error[E0599]: no method named `join` found for enum `Option` in the current scope
  --> src/lib.rs:52:27
   |
52 |             worker.thread.join().unwrap();
   |                           ^^^^ method not found in `Option<JoinHandle<()>>`
   |
note: the method `join` exists on the type `JoinHandle<()>`
  --> /rustc/d5a82bbd26e1ad8b7401f6a718a9c57c96905483/library/std/src/thread/mod.rs:1581:5
help: consider using `Option::expect` to unwrap the `JoinHandle<()>` value, panicking if the value is an `Option::None`
   |
52 |             worker.thread.expect("REASON").join().unwrap();
   |                          +++++++++++++++++

error[E0308]: mismatched types
  --> src/lib.rs:72:22
   |
72 |         Worker { id, thread }
   |                      ^^^^^^ expected enum `Option`, found struct `JoinHandle`
   |
   = note: expected enum `Option<JoinHandle<()>>`
            found struct `JoinHandle<_>`
help: try wrapping the expression in `Some`
   |
72 |         Worker { id, thread: Some(thread) }
   |                      +++++++++++++      +

Some errors have detailed explanations: E0308, E0599.
For more information about an error, try `rustc --explain E0308`.
error: could not compile `hello` due to 2 previous errors
``````
Let’s address the second error, which points to the code at the end of Worker::new; we need to wrap the thread value in Some when we create a new Worker. Make the following changes to fix this error:

Filename: src/lib.rs
``````
This code does not compile!
impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        // --snip--

        Worker {
            id,
            thread: Some(thread),
        }
    }
}
``````
The first error is in our Drop implementation. We mentioned earlier that we intended to call take on the Option value to move thread out of worker. The following changes will do so:

Filename: src/lib.rs
``````
This code does not produce the desired behavior.
impl Drop for ThreadPool {
    fn drop(&mut self) {
        for worker in &mut self.workers {
            println!("Shutting down worker {}", worker.id);

            if let Some(thread) = worker.thread.take() {
                thread.join().unwrap();
            }
        }
    }
}
``````
As discussed in Chapter 17, the take method on Option takes the Some variant out and leaves None in its place. We’re using if let to destructure the Some and get the thread; then we call join on the thread. If a worker’s thread is already None, we know that worker has already had its thread cleaned up, so nothing happens in that case.

正如第 17 章中所讨论的，Option 上的 take 方法取出 Some 变体并保留 None 。 我们使用 if let 来解构 Some 并获取线程； 然后我们在线程上调用 join 。 如果一个worker的线程已经是None，我们就知道该worker已经清理了它的线程，所以在这种情况下什么也不会发生。

### Signaling to the Threads to Stop Listening for Jobs
With all the changes we’ve made, our code compiles without any warnings. However, the bad news is this code doesn’t function the way we want it to yet. The key is the logic in the closures run by the threads of the Worker instances: at the moment, we call join, but that won’t shut down the threads because they loop forever looking for jobs. If we try to drop our ThreadPool with our current implementation of drop, the main thread will block forever waiting for the first thread to finish.

To fix this problem, we’ll need a change in the ThreadPool drop implementation and then a change in the Worker loop.

First, we’ll change the ThreadPool drop implementation to explicitly drop the sender before waiting for the threads to finish. Listing 20-23 shows the changes to ThreadPool to explicitly drop sender. We use the same Option and take technique as we did with the thread to be able to move sender out of ThreadPool:

经过我们所做的所有更改，我们的代码编译时不会出现任何警告。 然而，坏消息是这段代码还没有按照我们想要的方式运行。 关键是由 Worker 实例的线程运行的闭包中的逻辑：目前，我们调用 join，但这不会关闭线程，因为它们会永远循环寻找作业。 如果我们尝试使用当前的 drop 实现来删除 ThreadPool，则主线程将永远阻塞，等待第一个线程完成。

为了解决这个问题，我们需要更改 ThreadPool drop 实现，然后更改 Worker 循环。

首先，我们将更改 ThreadPool 删除实现，以在等待线程完成之前显式删除发送者。 清单 20-23 显示了对 ThreadPool 进行的更改，以显式删除发送者。 我们使用与线程相同的选项并采用技术，以便能够将发送者移出线程池：

Filename: src/lib.rs
``````
This code does not produce the desired behavior.
pub struct ThreadPool {
    workers: Vec<Worker>,
    sender: Option<mpsc::Sender<Job>>,
}
// --snip--
impl ThreadPool {
    pub fn new(size: usize) -> ThreadPool {
        // --snip--

        ThreadPool {
            workers,
            sender: Some(sender),
        }
    }

    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
        let job = Box::new(f);

        self.sender.as_ref().unwrap().send(job).unwrap();
    }
}

impl Drop for ThreadPool {
    fn drop(&mut self) {
        drop(self.sender.take());

        for worker in &mut self.workers {
            println!("Shutting down worker {}", worker.id);

            if let Some(thread) = worker.thread.take() {
                thread.join().unwrap();
            }
        }
    }
}
``````
Listing 20-23: Explicitly drop sender before joining the worker threads

Dropping sender closes the channel, which indicates no more messages will be sent. When that happens, all the calls to recv that the workers do in the infinite loop will return an error. In Listing 20-24, we change the Worker loop to gracefully exit the loop in that case, which means the threads will finish when the ThreadPool drop implementation calls join on them.

删除发送者将关闭通道，这表明将不再发送消息。 发生这种情况时，工作人员在无限循环中执行的所有对 recv 的调用都将返回错误。 在清单 20-24 中，我们更改了 Worker 循环，以便在这种情况下正常退出循环，这意味着当 ThreadPool drop 实现对线程调用 join 时，线程将完成。

Filename: src/lib.rs
``````
impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        let thread = thread::spawn(move || loop {
            let message = receiver.lock().unwrap().recv();

            match message {
                Ok(job) => {
                    println!("Worker {id} got a job; executing.");

                    job();
                }
                Err(_) => {
                    println!("Worker {id} disconnected; shutting down.");
                    break;
                }
            }
        });

        Worker {
            id,
            thread: Some(thread),
        }
    }
}
``````
Listing 20-24: Explicitly break out of the loop when recv returns an error

To see this code in action, let’s modify main to accept only two requests before gracefully shutting down the server, as shown in Listing 20-25.

Filename: src/main.rs
``````
fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
    let pool = ThreadPool::new(4);

    for stream in listener.incoming().take(2) {
        let stream = stream.unwrap();

        pool.execute(|| {
            handle_connection(stream);
        });
    }

    println!("Shutting down.");
}
``````
Listing 20-25: Shut down the server after serving two requests by exiting the loop

You wouldn’t want a real-world web server to shut down after serving only two requests. This code just demonstrates that the graceful shutdown and cleanup is in working order.

The take method is defined in the Iterator trait and limits the iteration to the first two items at most. The ThreadPool will go out of scope at the end of main, and the drop implementation will run.

Start the server with cargo run, and make three requests. The third request should error, and in your terminal you should see output similar to this:

您不希望现实世界的 Web 服务器在仅处理两个请求后关闭。 此代码仅表明正常关闭和清理工作正常。

take 方法在 Iterator 特征中定义，并将迭代限制为最多前两项。 ThreadPool 将在 main 结束时超出范围，并且 drop 实现将运行。

使用货物运行启动服务器，并发出三个请求。 第三个请求应该出错，并且在您的终端中您应该看到类似于以下内容的输出：

``````
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished dev [unoptimized + debuginfo] target(s) in 1.0s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Shutting down.
Shutting down worker 0
Worker 3 got a job; executing.
Worker 1 disconnected; shutting down.
Worker 2 disconnected; shutting down.
Worker 3 disconnected; shutting down.
Worker 0 disconnected; shutting down.
Shutting down worker 1
Shutting down worker 2
Shutting down worker 3
``````
You might see a different ordering of workers and messages printed. We can see how this code works from the messages: workers 0 and 3 got the first two requests. The server stopped accepting connections after the second connection, and the Drop implementation on ThreadPool starts executing before worker 3 even starts its job. Dropping the sender disconnects all the workers and tells them to shut down. The workers each print a message when they disconnect, and then the thread pool calls join to wait for each worker thread to finish.

Notice one interesting aspect of this particular execution: the ThreadPool dropped the sender, and before any worker received an error, we tried to join worker 0. Worker 0 had not yet gotten an error from recv, so the main thread blocked waiting for worker 0 to finish. In the meantime, worker 3 received a job and then all threads received an error. When worker 0 finished, the main thread waited for the rest of the workers to finish. At that point, they had all exited their loops and stopped.

Congrats! We’ve now completed our project; we have a basic web server that uses a thread pool to respond asynchronously. We’re able to perform a graceful shutdown of the server, which cleans up all the threads in the pool.

您可能会看到打印的工作人员和消息的顺序不同。 我们可以从消息中看到这段代码的工作原理：workers 0 和 3 收到了前两个请求。 服务器在第二个连接后停止接受连接，并且 ThreadPool 上的 Drop 实现在工作线程 3 开始其作业之前就开始执行。 删除发送者会断开所有工作人员的连接并告诉他们关闭。 每个工作线程在断开连接时都会打印一条消息，然后线程池调用 join 等待每个工作线程完成。

请注意此特定执行的一个有趣的方面：ThreadPool 删除了发送者，并且在任何工作程序收到错误之前，我们尝试加入工作程序 0。工作程序 0 尚未从接收中收到错误，因此主线程阻塞等待工作程序 0 完成。 与此同时，worker 3 收到了一个作业，然后所有线程都收到了一个错误。 当worker 0完成后，主线程等待其余worker完成。 那时，他们都退出了循环并停止了。

恭喜！ 我们现在已经完成了我们的项目； 我们有一个基本的 Web 服务器，它使用线程池来异步响应。 我们能够正常关闭服务器，从而清理池中的所有线程。

Here’s the full code for reference:

Filename: src/main.rs
``````
use hello::ThreadPool;
use std::fs;
use std::io::prelude::*;
use std::net::TcpListener;
use std::net::TcpStream;
use std::thread;
use std::time::Duration;

fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
    let pool = ThreadPool::new(4);

    for stream in listener.incoming().take(2) {
        let stream = stream.unwrap();

        pool.execute(|| {
            handle_connection(stream);
        });
    }

    println!("Shutting down.");
}

fn handle_connection(mut stream: TcpStream) {
    let mut buffer = [0; 1024];
    stream.read(&mut buffer).unwrap();

    let get = b"GET / HTTP/1.1\r\n";
    let sleep = b"GET /sleep HTTP/1.1\r\n";

    let (status_line, filename) = if buffer.starts_with(get) {
        ("HTTP/1.1 200 OK", "hello.html")
    } else if buffer.starts_with(sleep) {
        thread::sleep(Duration::from_secs(5));
        ("HTTP/1.1 200 OK", "hello.html")
    } else {
        ("HTTP/1.1 404 NOT FOUND", "404.html")
    };

    let contents = fs::read_to_string(filename).unwrap();

    let response = format!(
        "{}\r\nContent-Length: {}\r\n\r\n{}",
        status_line,
        contents.len(),
        contents
    );

    stream.write_all(response.as_bytes()).unwrap();
    stream.flush().unwrap();
}
``````
Filename: src/lib.rs
``````
use std::{
    sync::{mpsc, Arc, Mutex},
    thread,
};

pub struct ThreadPool {
    workers: Vec<Worker>,
    sender: Option<mpsc::Sender<Job>>,
}

type Job = Box<dyn FnOnce() + Send + 'static>;

impl ThreadPool {
    /// Create a new ThreadPool.
    ///
    /// The size is the number of threads in the pool.
    ///
    /// # Panics
    ///
    /// The `new` function will panic if the size is zero.
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let (sender, receiver) = mpsc::channel();

        let receiver = Arc::new(Mutex::new(receiver));

        let mut workers = Vec::with_capacity(size);

        for id in 0..size {
            workers.push(Worker::new(id, Arc::clone(&receiver)));
        }

        ThreadPool {
            workers,
            sender: Some(sender),
        }
    }

    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
        let job = Box::new(f);

        self.sender.as_ref().unwrap().send(job).unwrap();
    }
}

impl Drop for ThreadPool {
    fn drop(&mut self) {
        drop(self.sender.take());

        for worker in &mut self.workers {
            println!("Shutting down worker {}", worker.id);

            if let Some(thread) = worker.thread.take() {
                thread.join().unwrap();
            }
        }
    }
}

struct Worker {
    id: usize,
    thread: Option<thread::JoinHandle<()>>,
}

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        let thread = thread::spawn(move || loop {
            let message = receiver.lock().unwrap().recv();

            match message {
                Ok(job) => {
                    println!("Worker {id} got a job; executing.");

                    job();
                }
                Err(_) => {
                    println!("Worker {id} disconnected; shutting down.");
                    break;
                }
            }
        });

        Worker {
            id,
            thread: Some(thread),
        }
    }
}
``````
We could do more here! If you want to continue enhancing this project, here are some ideas:

+ Add more documentation to ThreadPool and its public methods.
+ Add tests of the library’s functionality.
+ Change calls to unwrap to more robust error handling.
+ Use ThreadPool to perform some task other than serving web requests.
+ Find a thread pool crate on crates.io and implement a similar web server using the crate instead. Then compare its API and robustness to the thread pool we implemented.

我们可以在这里做得更多！ 如果您想继续增强这个项目，这里有一些想法：

+ 为 ThreadPool 及其公共方法添加更多文档。
+ 添加库功能的测试。
+ 更改对 unwrap 的调用以实现更强大的错误处理。
+ 使用 ThreadPool 执行除处理 Web 请求之外的一些任务。
+ 在 crates.io 上找到一个线程池 crate，并使用该 crate 实现一个类似的 Web 服务器。 然后将它的 API 和健壮性与我们实现的线程池进行比较。

## 20.4 总结
Well done! You’ve made it to the end of the book! We want to thank you for joining us on this tour of Rust. You’re now ready to implement your own Rust projects and help with other peoples’ projects. Keep in mind that there is a welcoming community of other Rustaceans who would love to help you with any challenges you encounter on your Rust journey.

做得好！ 您已经读到了本书的结尾！ 我们要感谢您加入我们的 Rust 之旅。 您现在已准备好实现自己的 Rust 项目并帮助其他人的项目。 请记住，有一个由其他 Rustaceans 组成的热情社区，他们很乐意帮助您解决 Rust 之旅中遇到的任何挑战。