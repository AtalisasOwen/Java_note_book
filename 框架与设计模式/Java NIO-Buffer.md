# Java NIO-Buffer![](/assets/20140610223236484.jpeg)

## 属性：

* ### Capacity: 容量，缓存区能够容纳的数据元素的最大数量，在缓冲区创建时设定，并且永远不能改变
* ### Limit: 上界，缓存区中已使用的数据量
* ### Position: 位置，下一个要被读或写的元素\(如byte\)的索引
* ### Mark: 标记，做一个书签，便于操作

## 创建Buffer:

```java
//使用allocate静态方法
ByteBuffer buffer = ByteBuffer.allocate(10);//容量为100

//使用wrap静态方法
char[] myChars = new char[10];
CharBuffer buffer = CharBuffer.wrap(myChars);//将myChars数组作为内部封装的缓存区

//指定position=3,limit=8的缓存区
char[] myChars = new char[10];
CharBuffer buffer = CharBuffer.wrap(myChars, 3, 8);//将myChars数组作为内部封装的缓存区

//CharBuffer独有的函数:
//public static CharBuffer CharBuffer.wrap(CharSequence csq,int start.int end);
CharBuffer b = CharBuffer.wrap("Hello World");//这样创建的Buffer是只读的...

public final boolean hasArray()
public final char[] array()
public final int arrayOffset()//返回缓存区数据在数组中存储的开始位置的偏移量
```

## 操作Buffer:

* ### 存取

```java
ByteBuffer buff = ByteBuffer.allocate(10);  
//get() => get(int index)
buff.put((byte) 'A');  
buff.put((byte) 'B');  
buff.put((byte) 'C');  
buff.put((byte) 'D');
```

![](/assets/20140611223142031.jpeg)

* ### 翻转:可用于将写入状态转换为读取状态

```java
buff.flip()    //=> buff.limit(buff.position()).position(0)
//buff.rewind() => 仅将position设为0
```

![](/assets/20140611224150484.jpeg)

* ### 读取

```java
//put(byte b) => put(int index, byte b)
System.out.println((char)buff.get());  
System.out.println((char)buff.get());  

//hasRemaining()和remaining()
for (int i = 0; buff.hasRemaining(); i++){
    System.out.print(buff.get());
}

int count = buff.remaining();
for (int j = 0; j < count; j++){
    System.out.print(buff.get());
}
```

![](/assets/20140611223543203.jpeg)

* ### 压缩

compact\(\)：将position之后的元素复制到缓存区首部，其余位置的元素不变

```java
// 创建一个容量为10的byte数据缓冲区  
ByteBuffer buff = ByteBuffer.allocate(10);  
// 填充缓冲区  
buff.put((byte)'A');  
buff.put((byte)'B');  
buff.put((byte)'C');  
buff.put((byte)'D');  
System.out.println("first put : " + new String(buff.array()));  
//翻转  
buff.flip();  
//释放  
System.out.println((char)buff.get());  
System.out.println((char)buff.get());  
//压缩  
buff.compact();  
System.out.println("compact after get : " + new String(buff.array()));  
//继续填充  
buff.put((byte)'E');  
buff.put((byte)'F');  
//输出所有  
System.out.println("put after compact : " + new String(buff.array()));  

/*
first put : ABCD
A
B
compact after get : CDCD        //将原先2，3位置的字节复制到0，1位置。原位置字节不变
put after compact : CDEF        //并将position指向复制位置的下一个位置
*/
```

* ### 标记

remark\(\)：标记就是记住当前位置\(使用mark\(\)方法标记\)，之后可以将位置恢复到标记处\(使用reset\(\)方法恢复\)

```java
public final Buffer mark() {  
    mark = position;  
    return this;  
}  

public final Buffer reset() {  
    int m = mark;  
    if (m < 0)  
        throw new InvalidMarkException();  
    position = m;  
    return this;  
}
```

* ### 比较

equals\(Object obj\)：比较两个缓存区的每个值, 可以是比较不同的缓存区（返回false）

compareTo\(T that\)：比较相同类型的缓存区，比较position到limit之间的缓存值

![](/assets/20140611232314265.jpeg)

![](/assets/20140611232502468.jpeg)

* > ### 批量移动：高效移动数据，这是缓存区的目的

```java
//批量取出数据
public CharBuffer get(char[] dst)
public CharBuffer get(char[] dst,int offset,int length)

//批量存入数据
public final CharBuffer put(char[] src)
public final CharBuffer put(char[] src, int offset,int length)
public final CharBuffer put(CharBuffer src)

public final CharBuffer put(String src)
public final CharBuffer put(String src,int start,int end)
```

## 复制缓存区

```java
public abstract CharBuffer duplicate();           //复制为一个可读可写的缓存区
public abstract CharBuffer asReadOnlyBuffer();    //复制为一个只读缓冲区
public abstract CharBuffer slice();               //复制一个从源缓冲position到limit的新缓冲区
```



