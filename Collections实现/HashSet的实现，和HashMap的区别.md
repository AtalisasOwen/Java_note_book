# HashSet的实现，和HashMap的区别

* ### 都实现了Set接口，仅存储不同的对象。
* ### HashSet内部封装了一个HashMap对象，仅保存不同的Object作为键。内部有一个private static final Object PRESENT = new Object\(\);作为所有键的值
* ### TreeSet内部封装一个NavigableMap引用，默认构造器传入一个TreeMap对象，也可以传入别的实现NavigableMap接口的对象。（JDK中另一个实现的叫ConcurrentSkipListSet）
* ### LinkedHashSet调用父类HashSet的构造器，改为封装一个LinkedHashMap。LinkedHashMap继承了HashMap,其内部重写了newNode\(\)方法【put\(\)的底层实现】，当调用put\(\)的时候会将先将这个节点放进一个双链表的尾部，为的是遍历Set时候能按照添加顺序来。



