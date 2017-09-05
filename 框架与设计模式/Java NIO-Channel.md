# Java NIO-Channel

![](/assets/屏幕快照 2017-09-05 上午9.31.34.png)

* ## 打开通道

```java
//文件通道：只能通过RandomAccessFile, FileInputStream, FileOutputStream
RandomAccessFile file = new RandomAccessFile("somefile","r")
FileChannel fc = file.getChannel();

//Socket通道
SocketChannel sc = SocketChannel.open()
sc.connect(new InetSocket("119.28.83.71",2222));

ServerSocketChannel ssc = ServerSocketChannel.open();
ssc.socket().bind(new InetSocket("119.28.83.71",2222));
```



