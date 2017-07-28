# NIO , BIO的实现与区别

* ### BIO方式：同步并阻塞，一个连接一个线程，适用于连接数目比较小且固定的架构，这种方式对服务器资源要求比较高，并发局限于应用中，JDK1.4以前的唯一选择，但程序直观简单易理解。
* ### NIO方式：同步非阻塞，一个请求一个线程，适用于连接数目多且连接比较短（轻操作）的架构，比如聊天服务器，并发局限于应用中，编程比较复杂，JDK1.4开始支持。
* ### AIO方式：异步非阻塞，一个有效连接一个线程，适用于连接数目多且连接比较长（重操作）的架构，比如相册服务器，充分调用OS参与并发操作，编程比较复杂，JDK7开始支持。

### 同步：java自己处理IO读写

### 异步：java委托给操作系统处理IO操作

### 阻塞：只能等待完成

### 非阻塞：可以去干别的，好了会有通知的

![](/assets/37237-20151222220329015-207666376.png)

#### NIO由三个主要的部分组成：

#### 缓冲区（Buffers）、通道（Channels）和非阻塞I/O的核心类组成。

#### NIO本身是基于事件驱动思想来完成的，其主要想解决的是BIO的高并发问题：在使用同步I/O的网络应用中，如果要同时处理多个客户端请求，或是在客户端要同时和多个服务器进行通讯，就必须使用多线程来处理。也就是说，将每一个客户端请求分配给一个线程来单独处理。这样做虽然可以达到我们的要求，但同时又会带来另外一个问题。由于每创建一个线程，就要为这个线程分配一定的内存空间（也叫工作存储器），而且操作系统本身也对线程的总数有一定的限制。

#### 如果客户端的请求过多，服务端程序可能会因为不堪重负而拒绝客户端的请求，甚至服务器可能会因此而瘫痪。

#### NIO基于Reactor，当socket有流可读或可写入socket时，操作系统会相应的通知引用程序进行处理，应用再将流读取到缓冲区或写入操作系统。

#### 也就是说，这个时候，已经不是一个连接就要对应一个处理线程了，而是有效的请求，对应一个线程，当连接没有数据时，是没有工作线程来处理的。

## IO库

* ### 字节流InputStream，OutputStream

```
//OutputStream
write(byte[] output)
flush()
close()

//InputStream
read(byte[] input)
flush()
close()

//过滤器流
//网络 => TelnetInputStream => BufferedInputStream => CipherInputStream => 
                            文本 <= InputstreamReader => <= GZIPInputStream 
```

* ### 字符流Writer，Reader

```java
//文件系统
BufferedReader br = new BufferedReader(new FileReader(filename));

//标准输入
BufferedReader in = new BufferedReader(new InputStreamReader(System.in));
```

### 



