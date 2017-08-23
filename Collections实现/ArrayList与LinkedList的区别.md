# ArrayList与LinkedList的区别

### ![](/assets/20160413184734236.png)

* ### 在AbstractSequentialList中已经定义了很多的处理方法，只要LinkedList实现

```java
public abstract ListIterator<E> listIterator(int index);//实现就可以直接调用add(),remove(),get(),set()等
```

* ### ArrayList是实现了基于动态数组的数据结构，而LinkedList是基于链表的数据结构
* ### 对于随机访问get和set，ArrayList要优于LinkedList，因为LinkedList要移动指针；
* ### 对于添加和删除操作add和remove，一般大家都会说LinkedList要比ArrayList快，因为ArrayList要移动数据。但是实际情况并非这样，对于添加或删除，LinkedList和ArrayList**并不能明确说明谁快谁慢**，因为ArrayList会复制后继元素，而LinkedList会移动index指针。
* ### 在列表头上插入，肯定LinkedList快。。。其他情况下，ArrayList的get效率和insert效率都比LinkedList要高得多

```java
public class ListTest {
    static List<Integer> array=new ArrayList<Integer>();
    static List<Integer> linked=new LinkedList<Integer>();

    public static void main(String[] args) {

        //首先分别给两者插入10000条数据
        for(int i=0;i<10000;i++){
            array.add(i);
            linked.add(i);
        }
        //获得两者随机访问的时间
        System.out.println("array time:"+getTime(array));
        System.out.println("linked time:"+getTime(linked));
        //获得两者插入数据的时间
        System.out.println("array insert time:"+insertTime(array));
        System.out.println("linked insert time:"+insertTime(linked));

    }
    public static long getTime(List<Integer> list){
        long time=System.currentTimeMillis();
        for(int i = 0; i < 10000; i++){
            int index = Collections.binarySearch(list, list.get(i));
            if(index != i){
                System.out.println("ERROR!");
            }
        }
        return System.currentTimeMillis()-time;
    }

    //插入数据
    public static long insertTime(List<Integer> list){
        /*
         * 插入的数据量和插入的位置是决定两者性能的主要方面，
         * 我们可以通过修改这两个数据，来测试两者的性能
         */
        long num = 10000; //表示要插入的数据量
        int index = 9000; //表示从哪个位置插入
        long time=System.currentTimeMillis();
        for(int i = 1; i < num; i++){
            list.add(index, i);
        }
        return System.currentTimeMillis()-time;

    }
}

在index=10出插入结果：  
array time:22
linked time:787
array insert time:54
linked insert time:3
  
在index=1000处插入结果：  
array time:1
linked time:595
array insert time:41
linked insert time:44
  
在index=5000处插入结果：  
array time:12
linked time:537
array insert time:30
linked insert time:197

在index=9000处插入结果：  
array time:11
linked time:634
array insert time:15
linked insert time:225


```

## 部分源码分析

### LinkedList

```
//获得第index个节点的值  
public E get(int index) {  
    checkElementIndex(index);  
    return node(index).item;  
}  
  
//设置第index元素的值  
public E set(int index, E element) {  
    checkElementIndex(index);  
    Node<E> x = node(index);  
    E oldVal = x.item;  
    x.item = element;  
    return oldVal;  
}  
  
//在index个节点之前添加新的节点  
public void add(int index, E element) {  
    checkPositionIndex(index);  
  
    if (index == size)  
        linkLast(element);  
    else  
        linkBefore(element, node(index));  
}  
  
//删除第index个节点  
public E remove(int index) {  
    checkElementIndex(index);  
    return unlink(node(index));  
}  
  
//定位index处的节点  
Node<E> node(int index) {  
    // assert isElementIndex(index);  
    //index<size/2时，从头开始找  
    if (index < (size >> 1)) {  
        Node<E> x = first;  
        for (int i = 0; i < index; i++)  
            x = x.next;  
        return x;  
    } else { //index>=size/2时，从尾开始找  
        Node<E> x = last;  
        for (int i = size - 1; i > index; i--)  
            x = x.prev;  
        return x;  
    }  
} 
```

### ArrayList

```
//获取index位置的元素值  
public E get(int index) {  
    rangeCheck(index); //首先判断index的范围是否合法  
  
    return elementData(index);  
}  
  
//将index位置的值设为element，并返回原来的值  
public E set(int index, E element) {  
    rangeCheck(index);  
  
    E oldValue = elementData(index);  
    elementData[index] = element;  
    return oldValue;  
}  
  
//将element添加到ArrayList的指定位置  
public void add(int index, E element) {  
    rangeCheckForAdd(index);  
  
    ensureCapacityInternal(size + 1);  // Increments modCount!!  
    //将index以及index之后的数据复制到index+1的位置往后，即从index开始向后挪了一位  
    System.arraycopy(elementData, index, elementData, index + 1,  
                     size - index);   
    elementData[index] = element; //然后在index处插入element  
    size++;  
}  
  
//删除ArrayList指定位置的元素  
public E remove(int index) {  
    rangeCheck(index);  
  
    modCount++;  
    E oldValue = elementData(index);  
  
    int numMoved = size - index - 1;  
    if (numMoved > 0)  
        //向左挪一位，index位置原来的数据已经被覆盖了  
        System.arraycopy(elementData, index+1, elementData, index,  
                         numMoved);  
    //多出来的最后一位删掉  
    elementData[--size] = null; // clear to let GC do its work  
  
    return oldValue;  
}
```



