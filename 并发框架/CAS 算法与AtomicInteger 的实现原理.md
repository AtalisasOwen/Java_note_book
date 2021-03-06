### **CAS \(campare and swap\)算法与**AtomicInteger 的实现原理

* ### 原理：

  * ### CAS有三个操作数：内存值V，预期值A，更新值B
  * ### 如果V==A, 则V被赋值为B
  * ### 否则不干任何事（或循环，直到上述判断成立）

```java
public class AtomicInteger extends Number implements java.io.Serializable {  

    private volatile int value;              //用volatile来实现可见性


    public final int get() {  
        return value;  
    }  

    public final int getAndIncrement() {  
        for (;;) {  
            int current = get();              //获取当前值
            int next = current + 1;           //加1
            if (compareAndSet(current, next)) //比较这此期间，值有没有被篡改过，如果有则重新来过
                return current;               //成功了，则返回+1以后的值
        }  
    }  

    public final boolean compareAndSet(int expect, int update) {  
        return unsafe.compareAndSwapInt(this, valueOffset, expect, update);  
    }
```

* ### compareAndSet利用JNI来完成CPU指令的操作。

```c
int compare_and_swap (int* reg, int oldval, int newval) 
{
  ATOMIC();
  int old_reg_val = *reg;
  if (old_reg_val == oldval) 
     *reg = newval;
  END_ATOMIC();
  return old_reg_val;
}
```

* ### Java8中CAS的增强

```java
public final int getAndIncrement() {
        return unsafe.getAndAddInt(this, valueOffset, 1);
}
```

* ### 1.  **ABA问题**

  #### 。因为CAS需要在操作值的时候检查下值有没有发生变化，如果没有发生变化则更新，但是如果一个值原来是A，变成了B，又变成了A，那么使用CAS进行检查时会发现它的值没有发生变化，但是实际上却变化了。ABA问题的解决思路就是使用版本号。在变量前面追加上版本号，每次变量更新的时候把版本号加一，那么A－B－A 就会变成1A-2B－3A。

  * #### **从Java1**.5开始JDK的atomic包里提供了一个类AtomicStampedReference来解决ABA问题。这个类的compareAndSet方法作用是首先检查当前引用是否等于预期引用，并且当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用和该标志的值设置为给定的更新值。

* ### **2. 循环时间长开销大**

  #### 。自旋CAS如果长时间不成功，会给CPU带来非常大的执行开销。如果JVM能支持处理器提供的pause指令那么效率会有一定的提升，pause指令有两个作用，第一它可以延迟流水线执行指令（de-pipeline）,使CPU不会消耗过多的执行资源，延迟的时间取决于具体实现的版本，在一些处理器上延迟时间是零。第二它可以避免在退出循环的时候因内存顺序冲突（memory order violation）而引起CPU流水线被清空（CPU pipeline flush），从而提高CPU的执行效率。
* ### **3. 只能保证一个共享变量的原子操作**

  #### 。当对一个共享变量执行操作时，我们可以使用循环CAS的方式来保证原子操作，但是对多个共享变量操作时，循环CAS就无法保证操作的原子性，这个时候就可以用锁，或者有一个取巧的办法，就是把多个共享变量合并成一个共享变量来操作。比如有两个共享变量i＝2,j=a，合并一下ij=2a，然后用CAS来操作ij。从Java1.5开始JDK提供了**AtomicReference类来保证引用对象之间的原子性，你可以把多个变量放在一个对象里来进行CAS操作。**



