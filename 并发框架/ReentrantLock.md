# ReentrantLock

| 方法 | 作用 |
| :--- | :--- |
| lock\(\) | 加锁 |
| lockInterruptibly\(\) | 可以被打断的锁 |
| tryLock\(5, TimeUnit.SECOND\) | 会等待一段时间。成功返回true，否则返回false |
|  |  |



```java
public class TestLock implement Runnable{
    public static ReentrantLock lock = new ReentrantLock();
    public static int i = 0;
    
    @override
    public void run(){
        for(int j = 0;j < 10000;j++){
            lock.lock();
            try{
                i++;
            }finally{
                lock.unlock();
            }
        }
    }
    
    public static void main(String...args) throws InterruptedException{
        TestLock tl = new TestLock();
        Thread t1 = new Thread(tl);
        Thread t2 = new Thread(t2);
        
        t1.start();
        t2.start();
        
        t1.join();
        t2.join();
        
        System.out.print(i);
    }
    
}
```

* ### 可重入锁使得一个线程连续多次获得同一把锁，但lock\(\)和unlock\(\)的次数要一样。
* ### 与synchronized相比，还多了中断线程的功能。即一个线程在等待锁的时候，可以收到一个通知，去中断线程。可以有效避免死锁

  * ### jps -l :看哪个java类正在运行，并记录VMID
  * ### jstack -l vmid :看堆栈和锁情况

```java
package atalisas.Crawler;

import java.util.concurrent.locks.ReentrantLock;

/**
 * Created by 顾文涛 on 2017/8/27.
 */
public class IntLock implements Runnable {
    public static ReentrantLock lock1 = new ReentrantLock();
    public static ReentrantLock lock2 = new ReentrantLock();

    int lock;

    public IntLock(int lock) {
        this.lock = lock;
    }

    @Override
    public void run() {
        try{
            if (lock == 1){
                lock1.lockInterruptibly();
                try{
                    Thread.sleep(500);
                }catch (InterruptedException e){}
                lock2.lockInterruptibly();
            }
            else{
                lock2.lockInterruptibly();
                try{
                    Thread.sleep(500);
                }catch (InterruptedException e){}
                lock1.lockInterruptibly();
            }
        }catch (InterruptedException e){
            e.printStackTrace();
        }finally {
            if (lock1.isHeldByCurrentThread())
                lock1.unlock();
            if (lock2.isHeldByCurrentThread())
                lock2.unlock();
            System.out.println(Thread.currentThread().getId()+"：Thread Exit");
        }
    }

    public static void main(String...args) throws Exception{
        IntLock r1 = new IntLock(1);
        IntLock r2 = new IntLock(2);
        Thread t1 = new Thread(r1);
        Thread t2 = new Thread(r2);

        t1.start();t2.start();

        Thread.sleep(1000);
        
        t2.interrupt();//中断一个线程
        /*
        java.lang.InterruptedException
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.doAcquireInterruptibly(AbstractQueuedSynchronizer.java:898)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireInterruptibly(AbstractQueuedSynchronizer.java:1222)
	at java.util.concurrent.locks.ReentrantLock.lockInterruptibly(ReentrantLock.java:335)
	at atalisas.Crawler.IntLock.run(IntLock.java:33)
	at java.lang.Thread.run(Thread.java:745)
        13：Thread Exit
        12：Thread Exit
        */
        
    }

}
```

* ### tryLock\(\)的用法

```java
public class TimeLock implements Runnable{
    private static ReentrantLock lock = new ReetrantLock();
    
    @override
    public void run(){
        try{
            if(lock.tryLock(5,TimeUnit.SECOND)){
                Thread.sleep(60000);
            }else{
                System.out.print("get lock failed")
            }
        }catch(InterruptedException e){
        }finally{
            if(lock.isHeldByCurrentThread())
                lock.unlock();
        }
    }
    
    public static void main(String...args){
        TimeLock tl = new TimeLock();
        Thread t1 = new Thread(tl);
        Thread t2 = new Thread(tl);
        
        t1.start();
        t2.start();//这个会失败
    }
}

```

* ### 公平锁和非公平锁：公平锁会先来先得，维护一个队列

```
ReentrantLock lock = new ReentrantLock(true);    //通过构造方法实现公平锁
```





