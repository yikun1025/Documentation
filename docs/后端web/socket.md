#### socket
socket是用于我们可以进行网络通信的方法，使用socket需要两部分信息，*host*和*port*

##### 建立网络连接的条件
- 服务器端的ip和port
- 客户端的ip和port

![[Pasted image 20220503164020.png]]
三次握手，tcp/ip协议这块我们都学过，但是真正使用，写程序是另外一回事

通过ip地址+协议+端口就可以确定唯一pid，上一节已经详细解释过

> socket翻译为套接字，socket是在应用层和传输层之间的一个抽象层，它把TCP/IP层复杂的操作抽象为几个简单的接口供应用层调用已实现进程在网络中通信。

socket实际上做的就是去通信，传送数据获取数据，server类做的就是图上显示的这些事情

##### Server
- 创建socket 
- 绑定socket端口号 
- 监听
- 接受数据
- 读取字符
- 关闭socket

##### Client
- 创建Socket  
- 连接端口 
- 发送数据 ;
- 关闭socket

##### 关于inputstream和outputstream
> socket实际上是一个抽象概念，， 并不是真实存在的，用于你连接网络的概念。如下图所示

```java
      Client                                              Server
1. OutputStream -->                          --> 2. InputStream -->
             Socket <--> network <--> ServerSocket                
4. InputStream  <--                            <--3. OutputStream <--
```

在java，通过socket发送数据你首先要获取*outputstream*,然后再将数据写入
```
OutputStream output = socket.getOutputStream();  
output.write(request.getBytes(StandardCharsets.UTF_8));
```

从soket读取数据，你也是先获取*inputstream*,再从*second stream*读取。
```
InputStream input = socket.getInputStream();  
InputStreamReader reader_data = new InputStreamReader(input);
```

首先socket是可以被看作单向通道,*Client*到*server*是单向的，反之也如此。server和client都具有自己的*socket*

举例子，*server*到*client*步骤是这样的:
1. *server*获取*outputstream*，并且写入数据到*outputstream*
2.  *server*获取*inputstream*，并且从网络读取数据到*inputstream*
3. *client*获取*outputstream*，并且写入数据到*outputstream*
4. *client*获取*inputstream*，并且从网络读取数据到*inputstream*

> 另外，因为socket传送数据过程中用的是*raw bytes*，也就是二进制码，因此我们需要转码的方式读取,最终获取的数据在*reader_data*里

- ![avatar](https://images.weserv.nl/?url=https://article.biliimg.com/bfs/article/7ef673a3ad3b3529e4844c35f243a3e600d506c2.jpg)
我们得到的这一串其实就是浏览器做的事情，浏览器获取响应的数据并解析。
目前实现的是单线程的socket，多线程其实逻辑是一样的，也是按照这种方式分发，感兴趣的可以先实现单线程的。

> 浏览器的核心就是发请求，服务器的核心是处理请求返回响应

##### Client代码
client的socket和server的socket不是同一个，因此我们需要使用*ServerSocket*来建立服务器端数据
```
ServerSocket serverSocket = new ServerSocket(port)
```
服务器实际上是一直等待客户端传送数据的，因此应该使用*while*循环持续等待数据传输，网络连接完成才会继续往下走
```
Socket socket = serverSocket.accept()
```
接下来做的事情就是和socket做的事情一样先发送请求数据，接下来需要拼接响应内容传递给浏览器

我们可以直接拼接最简单的响应内容，包含响应头，响应字符串长度，状态码以及body内容，拼接最简单的内容如下
```
String body = "<html><body>Guoba</body></html>";  
String length = String.format("Content-Length: %s", body.length());  
String response = "HTTP/1.1 200 very OK\r\n" + "Content-Type: text/html;\r\n" +  
        length + "\r\n\r\n" + body;
```

这里能做到的就是给浏览器一个html打包好的内容，没有css样式，没有js，就是一个简单的文字输出在网页，但是这就是服务器端做的最本质的事情。
- ![avatar](https://images.weserv.nl/?url=https://article.biliimg.com/bfs/article/84833242e1e66d956ad5075f432c0256ebe70501.jpg)

> 任何复杂的框架，服务器端，无论套了什么，最本质做的不过是去分发数据罢了。

只有通过自己实现web框架，才能知道server和client做的是什么，不了解底层只会陷入迷雾。

有关ServerSocket，我们可以去看一下底层代码是怎么写的
```
public ServerSocket(int port, int backlog, InetAddress bindAddr) throws IOException {  
    setImpl();  
    if (port < 0 || port > 0xFFFF)  
        throw new IllegalArgumentException(  
                   "Port value out of range: " + port);  
    if (backlog < 1)  
      backlog = 50;  
    try {  
        bind(new InetSocketAddress(bindAddr, port), backlog);  
    } catch(SecurityException e) {  
        close();  
        throw e;  
    } catch(IOException e) {  
        close();  
        throw e;  
    }  
}
```
这里是个构造函数，用于生成ServerSocket类
- 端口超过范围，则报错，端口地址在0到65535之间(2^16)
- 调用*bind*函数绑定端口地址。如果地址为空，系统会自动配置合法地址给端口，因此服务器端接收数据会在最初就需要端口(这也是上一节说的很清楚的，为什么需要端口作为地址)
- *accept*是用于等待外来请求连接，一旦连接则创建新的socket
>Listens for a connection to be made to this socket and accepts it. The method blocks until a connection is made.
A new Socket s is created and, if there is a security manager, the security manager's checkAccept method is called with s.getInetAddress().getHostAddress() and s.getPort() as its arguments to ensure the operation is allowed. This could result in a SecurityException.


##### 二进制
数据在client和server的传输都是用*bytes*传输的，那么什么是bytes?
> 计算机里数据是以二进制存储的，每一位代表一个*bit*,每八位代表一个字节,即*byte*

数字和字符对应的关系，被称为编码表，比如ascii编码，gbk编码，gb2312,utf-8编码

>我们后端开发是尽量以utf-8编码格式的

##### 为何会乱码
假设65在ascii里是A,但是在gbk里可能是&，因此编码格式不对会导致程序读取数据解码后会乱码


##### 字符，字节
字符和字节是两个概念。java里我们都知道字符串的概念，但是实际上java更习惯用*StringBuilder*来生成字符串。因此字符->字节转换方式如下
```
StringBuilder sb = new StringBuilder();
sb.toString().getBytes(StandardCharsets.UTF_8)
```

```
StringBuilder str = new StringBuilder("考试");  
str.toString().getBytes(StandardCharsets.UTF_8);  
str.toString().getBytes(StandardCharsets.US_ASCII);
```

```
field StringBuilder str = 考试

str.toString().getBytes(StandardCharsets.UTF_8) = byte[6] { -24, -128, -125, -24, -81, -107 }

str.toString().getBytes(StandardCharsets.US_ASCII) = byte[2] { 63, 63 }

```

可见，*考试在*utf-8和ascii的编码是完全不同的。