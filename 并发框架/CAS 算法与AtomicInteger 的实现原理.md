### **CAS 算法与**AtomicInteger 的实现原理

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



