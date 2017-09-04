# Java NIO-ByteBuffer

* ## 大端字节顺序和小端字节顺序

![](/assets/20140614182759796.png)

### IP协议规定了使用大端的网络字节顺序，所有在IP分组报文的协议部分中使用的多字节数值必须现在本地主机字节顺序和通用的网络字节顺序之间进行转换。

```java
public final class ByteOrder{
    public static final ByteOrder BIG_ENDIAN;       //大端 
    public static final ByteOrder LITTLE_ENDIAN;    //小端

    public static ByteOrder nativeOrder()           //返回运行JVM机器的字节顺序
}

public abstract class CharBuffer extends Buffer ...{
    public final ByteOrder order();                //返回运行JVM机器的字节顺序
}
```

* ## 直接缓存区

### 只有ByteBuffer才能参与IO操作，但操作系统的IO操作的目标内存区域必须是连续的字节序列，而byte\[\]在JVM中的实现并不是连续的字节。所以引入了直接缓存区的概念。

### 直接缓存区被用于通道与固有的IO线程交互

```java
public static ByteBuffer allocateDirect(int capacity)    //分配直接缓存区
public abstract boolean isDirect()
```

* ## 视图缓存区

### 例：如将ByteBuffer转换为CharBuffer, 每2个字节转换为1个char

```java
asCharBuffer()
asShortBuffer()
asIntBuffer()
asLongBuffer()
asFloatBuffer()
asDoubleBuffer()

ByteBuffer byteBuffer = ByteBuffer.allocate(7).order(ByteOrder.BIG_ENDIAN);
CharBuffer charBuffer = byteBuffer.asCharBuffer()

//将2个字节转换为1个字符，UTF-16编码刚刚好
byte[] bytes = "Hello".getBytes(Charset.forName("UTF-16"));
ByteBuffer b = ByteBuffer.wrap(bytes);
CharBuffer c = b.asCharBuffer();
System.out.println(c);
```

![](/assets/20140616002626656.jpeg)

* ### 数据元素视图

```java
ByteBuffer buff = ByteBuffer.allocate(100);  
buff.putShort((short)100);  
buff.putInt(200);  
buff.putLong(300L);  
buff.putFloat(400.5f);  
buff.putDouble(500.56);  
buff.putChar('A');  

buff.flip();  

buff.getShort();//return 100  
buff.getInt();//return 200  
buff.getLong();//return 300  
buff.getFloat();//return 400.5  
buff.getDouble();//return 500.56  
buff.getChar();//return 'A' 
```



