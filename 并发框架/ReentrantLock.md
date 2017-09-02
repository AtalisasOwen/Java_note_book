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

# Condition: 与ReentrantLock合用实现等待/通知模式

| 方法 | 作用 |
| :--- | :--- |
| await\(\), await\(time,TimeUnit\) | 会使当前线程等待\(一段时间\)并释放锁，直到被唤醒 |
| awaitUninterruptibly\(\) | 不会被中断的线程等待 |
| signal\(\), signalAll\(\) | 唤醒线程 |

```java
public class IntLock implements Runnable {
    public static ReentrantLock lock = new ReentrantLock();
    public static Condition condition = lock.newCondition();

    @Override
    public void run() {
        try{
            lock.lock();
            condition.await();
            System.out.println("Thread is going on!");
        }catch (InterruptedException e){}
        finally {
            lock.unlock();
        }
    }

    public static void main(String...args) throws Exception{
        IntLock tl = new IntLock();
        Thread t1 = new Thread(tl);
        t1.start();
        Thread.sleep(2000);
        lock.lock();
        condition.signalAll();        //过2秒后，通知线程继续运行
        lock.unlock();
    }

}
```

* ### 实现原理

```java
final void lock() {
     acquire(1);
}

=>AbstractQueuedSynchronizer.acquire
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&                                //首先尝试获取,未获取则入队列,获取直接返回不打断
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))     //addWaiter,将新请求加入队列尾进行等待，tryAcquire子类实现
        selfInterrupt();                                   //Thread.currentThread().interrupt();
}

=>ReentrantLock.FairSync.tryAcquire
protected final boolean tryAcquire(int acquires) {
        final Thread current = Thread.currentThread();    //获取当前线程
        int c = getState();                               //获取父类AQS中的标志位
        if (c == 0) {
            if (!hasQueuedPredecessors() &&                //如果队列中没有其他线程，说明没有线程占有锁
                compareAndSetState(0, acquires)) {         //修改一下状态位,acquires为1
                setExclusiveOwnerThread(current);          //如果通过CAS操作将状态更新成功，则证明当前线程得锁，并做一个标记
                return true;                               //表示tryAcquire成功
            }
        }
        else if (current == getExclusiveOwnerThread()) {   //如果不为0，则锁被拿走，但是为可重入锁，要判断是不是自己拿的
            int nextc = c + acquires;                      //将lock()的次数变为n+1次，说明也要unlock这么多次
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
}

=>AbstractQueuedSynchronizer.AddWaiter
private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode);        //用当前线程构建一个Node,mode表示是独占的，还是共享的，
    // Try the fast path of enq; backup to full enq on failure
    Node pred = tail;                                          //标准的插入到链表尾部
    if (pred != null) {                                        //在队列不为空
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {                    //修改失败，表示有并发，进入enq(node)死循环
            pred.next = node;
            return node;
        }
    }
    enq(node);                                                //进入死循环
    return node;
}

=>AbstractQueuedSynchronizer.enq
private Node enq(final Node node) {
    for (;;) {                                                //死循环，为了避免竞态失败
        Node t = tail;
        if (t == null) { // Must initialize                   //队列尾为空
            if (compareAndSetHead(new Node()))                //如果无竞态
                tail = head;                                  //尾等于头，说明只有一个节点
        } else {
            node.prev = t;                                    //
            if (compareAndSetTail(t, node)) {                 //不为空则将入队列尾
                t.next = node;
                return t;                                     //无竞态则成功
            }
        }
    }
}

=>AbstractQueuedSynchronizer.acquireQueued
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
             //如果当前的节点是head说明他是队列中第一个“有效的”节点，因此尝试获取，上文中有提到这个类是交给子类去扩展的。
                setHead(node);//成功后，将上图中的黄色节点移除，Node1变成头节点。
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            if (shouldParkAfterFailedAcquire(p, node) && 
            //否则，检查前一个节点的状态为，看当前获取锁失败的线程是否需要挂起。
                parkAndCheckInterrupt()) 
           //如果需要，借助JUC包下的LockSopport类的静态方法Park挂起当前线程。知道被唤醒。
                interrupted = true;
        }
    } finally {
        if (failed) //如果有异常
            cancelAcquire(node);// 取消请求，对应到队列操作，就是将当前节点从队列中移除。
    }
}
```



