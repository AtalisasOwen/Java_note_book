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

* ## 使用通道

```java
//注意区分只读文件和可读写文件
FileInputStream input = new FileInputStream("filename");
FileChannel channel = input.getChannel();

channel.write(buffer);        //InputStrem只能读，不能写
channel.read(buffer);         //将文件内容读入channel
```

```java
public class Test {
    public static void main(String...args) throws IOException{
        ReadableByteChannel source = Channels.newChannel(System.in);
        WritableByteChannel destin = Channels.newChannel(System.out);

        channelCopy2(source,destin);

        source.close();
        destin.close();
    }

    private static void channelCopy2(ReadableByteChannel src,
                                     WritableByteChannel des) throws IOException{
        ByteBuffer buffer = ByteBuffer.allocateDirect(10);
        while (src.read(buffer) != -1){     //将输入的内容读取到buffer
            buffer.flip();                  //切换为写状态
            while (buffer.hasRemaining()){  //比如只用了5个字节
                des.write(buffer);          //那就只写5个字节
            }
            buffer.clear();                 //清空缓存区
        }
    }
}
```



