# join\(\)和yield\(\)

* ### join\(\)的本质是是让调用线程wait\(\)在当前线程对象实例上

```java
public class JoinMain{

    public static void main(String...args){
        public volatile static int i = 0;
        Thread t1 = new Thread(() -> {
           for(;i < 100000;i++); 
        });
        t1.start();
        t1.join();                //没有这句会打印0或1之类的。
        System.out.println(i);    //此时会打印10000，因为当前线程wait线程t1返回
    }
}   
```

* ### yield\(\)会使让出cpu，但让出的时间不确定，可能马上又得到了。用于优先级较低的线程，害怕会占据大量的cpu资源，因此在适当的时间调用Thread.yield\(\)



