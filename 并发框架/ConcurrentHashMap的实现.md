# ConcurrentHashMap的实现

* ### 并不能保证线程安全

```java
//通过加锁和HashMap来实现线程安全
public class TestClass{
    private Map<String,Integer> map = new HashMap<>();
    public synchronized void add(String key){
        Integer value = map.get(key);
        if(value == null){
            map.put(key,1);
        }else{
            map.put(key,value+1);
        }
    }
}

//通过ConcurrentHashMap不能实现线程安全
public class TestClass{
    private Map<String,Integer> map = new ConcurrentHashMap<>();
    public void add(String key){
        Integer value = map.get(key);
        if(value == null){
            map.put(key,1);
        }else{
            map.put(key,value+1);
        }
    }
}


```

### 上述代码不是ConcurrentHashMap的锅，因为线程13和线程11同时进行操作，线程13将值设置为14，然后线程11又将线程设置为12。所以总值会小



