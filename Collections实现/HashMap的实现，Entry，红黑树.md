# HashMap的实现

* ### Map类型的实现类，要求key为不可变对象，即其hashCode不能改变，如果发生改变的话，Map对象就可能定位不到映射的value。
* ### 从实现来说：由数组+单链表+红黑树\(1.8新增\)![](/assets/hashMap内存结构图.png)
* ### 数组：transient  Node&lt;K,V&gt;\[\]  table;初始化和扩展数组在resize\(\)方法内，在第一次调用HashMap的put\(\)时会去初始化数组，默认大小为16，门槛threshold为size\*0.75，当put等操作后会判断是否需要去扩充。
* ### 单链表：Node实现自Map.Entry接口

| Fields | Methods |
| :--- | :--- |
| final  int  hash | K  getKey\(\) |
| K  key | V  getValue\(\) |
| V  value | V setValue\(\)=&gt;用于更新 |
| Node&lt;K,V&gt;  next | hashCode\(\), equal\(\), toString\(\) |

* ### 计算hash

```java
static final int hash(Object key) {   //jdk1.8 & jdk1.7
     int h;
     // h = key.hashCode() 为第一步 取hashCode值
     // h ^ (h >>> 16)  为第二步 取高16位与低16位参与异或运算
     return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
```

![](/assets/hashMap哈希算法例图.png)

* ### put\(\)操作![](/assets/hashMap put方法执行流程图.png)

```java
 1 public V put(K key, V value) {
 2     // 对key的hashCode()做hash
 3     return putVal(hash(key), key, value, false, true);
 4 }
 5
 6 final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
 7                boolean evict) {
 8     Node<K,V>[] tab; Node<K,V> p; int n, i;
 9     // 步骤①：tab为空则创建
10     if ((tab = table) == null || (n = tab.length) == 0)
11         n = (tab = resize()).length;
12     // 步骤②：计算index，并对null做处理 
13     if ((p = tab[i = (n - 1) & hash]) == null) 
14         tab[i] = newNode(hash, key, value, null);
15     else {
16         Node<K,V> e; K k;
17         // 步骤③：节点key存在，直接覆盖value
18         if (p.hash == hash &&
19             ((k = p.key) == key || (key != null && key.equals(k))))
20             e = p;
21         // 步骤④：判断该链为红黑树
22         else if (p instanceof TreeNode)
23             e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
24         // 步骤⑤：该链为链表
25         else {
26             for (int binCount = 0; ; ++binCount) {
27                 if ((e = p.next) == null) {
28                     p.next = newNode(hash, key,value,null);
                        //链表长度大于8转换为红黑树进行处理
29                     if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st  
30                         treeifyBin(tab, hash);
31                     break;
32                 }
                    // key已经存在直接覆盖value
33                 if (e.hash == hash &&
34                     ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
36                 p = e;
37             }
38         }
39        
40         if (e != null) { // existing mapping for key
41             V oldValue = e.value;
42             if (!onlyIfAbsent || oldValue == null)
43                 e.value = value;
44             afterNodeAccess(e);
45             return oldValue;
46         }
47     }

48     ++modCount;
49     // 步骤⑥：超过最大容量 就扩容
50     if (++size > threshold)
51         resize();
52     afterNodeInsertion(evict);
53     return null;
54 }
```

* ### resize\(\)方法

```

```



