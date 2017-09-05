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

* ## 使用通道和关闭通道

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

#### 调用通道的close\(\)方法，可能会导致在通道关闭底层IO服务中，似的线程暂时阻塞。

#### 可以通过isOpen\(\)来查看通道是否开着。当一个通道实现了InterruptibleChannel接口，当对被阻塞的通道调用interrupt\(\)方法时，通道也会自动关闭。

* ## Scatter/Gather

![](/assets/屏幕快照 2017-09-05 上午11.07.17.png)

![](/assets/屏幕快照 2017-09-05 下午12.31.19.png)

* ## 文件通道

![](/assets/屏幕快照 2017-09-05 下午12.33.33.png)

| FileChannel | RandomAccessFile | POSIX system call |
| :--- | :--- | :--- |
| read\(\) | read\(\) | read\(\) |
| write\(\) | write\(\) | write\(\) |
| size\(\) | length\(\) | fstat\(\): 成功返回0，失败返回1 |
| position\(\):表示文件中当前字节位置 | getFilePointer\(\):返回当前偏移 | lseek\(\) |
| position\(long newPosition\):设置位置 | seek\(\):设置文件偏移 | lseek\(\) |
| truncate\(long size\):删减到指定大小 | setLength\(\) | ftruncate\(\) |
| force\(boolean metaData\):参数不管，这个方法的作用相当于flush\(\) | getFD\(\).sync\(\) | fsync\(\) |

* ## 文件锁定

```java
//独占锁，锁定全部文件
public final FileLock lock();
//开始位置，和锁定区域的大小，以及是共享锁和独占锁
public abstract FileLock lock(long position, long size, boolean shared);

//不能立即获取，则返回null
public final FileLock tryLock();
//开始位置，和锁定区域的大小，以及是共享锁和独占锁 => 尝试获取
public abstract FileLock tryLock(long position, long size, boolean shared);
```

### FileLock

```java
public abstract class FileLock{
    public final FileChannel channel();                        //返回创建锁的Channel
    public final long position();                              //锁定的起始位置
    public final long size();                                  //锁定的大小
    public final boolean isShared();                           //判断是独占还是共享锁
    public final boolean overlaps(long position, long size);   //是否与一个指定的区域有重叠
    public abstract boolean isValid();                         //锁的有效性
    public abstract void release() throws IOException;         //释放锁
}
```

* ## 内存映射文件

#### 调用map\(\)会创建一个由磁盘文件支持的虚拟内存映射，并在那块虚拟内存空间外部封装一个MappedByteBuffer对象。

#### ByteBuffer是在内存中建立缓存，而MappedByteBuffer是在磁盘文件建立一个缓存区。

```java
public abstract class FileChannel ...{

    public abstract MappedByteBuffer map(MapMode mode, long position, long size);

    public static class MapMode{

        public static final MapMode READ_ONLY         //只可读
            = new MapMode("READ_ONLY");

        public static final MapMode READ_WRITE        //可读写
            = new MapMode("READ_WRITE");

        public static final MapMode PRIVATE            //写时拷贝，不会对底层文件做任何修改
            = new MapMode("PRIVATE");
    }
}
```

### Channel-to-channel传输

```java
public abstract class FileChannel ...{
    public abstract long transferTo(long position, long count, WriteableByteChannel target);
    public abstract long transferFrom(ReadableByteChannel src, long position, long count);
}
```



