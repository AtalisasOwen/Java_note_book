# HashMap与HashTable的区别

![](/assets/java.util.map类图.png)

* ### HashMap不是线程安全的，而HashTable是线程安全的，但是最好用ConcurrentHashMap代替。或Map m = Collections.synchronizeMap\(hashMap\)
* ### 两者的继承类也不同
* ### HashTable的key和value都不可以是null。而HashMap可以有一个键（只能一个）为null，而value的值可以为null。用HashMap的get\(\)方法返回null，有2种可能（1.没有该键；2.该键对应的值为null），所以应该用containsKey\(\)方法来判断。
* ###  哈希值的使用不同，HashTable直接使用对象的hashCode。而HashMap重新计算hash值



