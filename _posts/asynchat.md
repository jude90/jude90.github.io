# asynchat - 异步协议处理 #

asynchat 模块基于asyncore, 使服务器和客户端之间来回传递消息
的协议写起来更简单。async_chat 类是asyncore.dispatcher 的子类，
可以接受数据并寻找消息结束符。
在继承这个类时你只需要指定当数据来的时候做什么，
以及当找到数据结束符时如何做出应答。

## 数据消费者 ##
收到的数据基于数据结束符来分割，由每个 对象（asynchat子类）的set_terminator()
来控制。有三种可能的情况：
1. 如果把一个字符串参数传给set_terminator(),当这个字符串出现在输入数据中时，消息传递完毕
2. 如果set_terminator()的参数是一个数字，当接收到那么同样多字节数之后，消息传递完毕。
3. 如果set_terminator()的参数是None，那么asynchat不会处理消息结束符。

下面的例程EchoServer中既用到了字符串式的消息结束符，也有取决于收到的数据上下文的数字长度结束符。标准库文档中的 HTTP 请求处理器 例程 展示了如何 根据上下文来区分 HTTP头和 HTTP POST 请求部分，以改变结束符的字长。

## Server and Handler ##
为了理解 asynchat 和 asyncore 间的区别，下面的例子用 asynchat 重写了 EchoServer 的程序。有些地方是相似的，有一个服务器对象接受连接请求，处理对象负责与每一个客户端进行通讯，一些客户端对象发起对话。

EchoServer 的实现和 asyncore版本里面的代码一样，因为它不是重点。
EchoHandler 现在的asynchat.async_chat 的子类，而不再是asyncore的。
它有更高的抽象层次，数据的读和写都被自动地处理了。缓冲区需要知道四件事情：

- 如何处理收到的数据（通过重写handler_incoming_data）
- 如何识别数据的结尾（设置set_terminator）
- 当消息完整接收后该做什么（found_terminator）
- 发送什么数据（用push())
- 


