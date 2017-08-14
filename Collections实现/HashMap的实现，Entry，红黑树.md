# HashMap的实现

* ### Map类型的实现类，要求key为不可变对象，即其hashCode不能改变，如果发生改变的话，Map对象就可能定位不到映射的value。

* ### 从实现来说：由数组+单链表+红黑树\(1.8新增\)![](/assets/hashMap内存结构图.png)

* ### 数组：transient  Node&lt;K,V&gt;\[\]  table;初始化和扩展数组在resize\(\)方法内，在第一次调用HashMap的put\(\)时会去初始化数组，默认大小为16，门槛threshold为size\*0.75，当put等操作后会判断是否需要去扩充。
* ### Node



