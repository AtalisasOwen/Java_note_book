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



