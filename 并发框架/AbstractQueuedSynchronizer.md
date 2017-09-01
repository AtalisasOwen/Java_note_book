# AbstractQueuedSynchronizer

* ### 这货是整个concurrent包的灵魂类
* ### 子类通过继承并需要实现它的方法来管理状态，通过acquire和release的方式来操作状态。然而多线程环境中对状态的操纵必须保证原子性，因此子类需要使用这个同步器提供的getState\(\)、setState\(int\)、compareAndSetState\(int,int\)
* ### 该同步器可以作为排他模式或者共享模式来运行。如ReentrantReadWriteLock内部，写锁的实现为排他，而读锁的实现为共享。
* ### 内部实现依赖于FIFO队列，队列中的元素Node保存着线程引用和线程状态的容器，每个线程的访问可以视为队列中的一个节点Node







* ### acquire\(int arg\)详解

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&                                //首先尝试获取,未获取则入队列
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))     //addWaiter,将新请求加入队列尾进行等待
        selfInterrupt();
}
    
protected boolean tryAcquire(int arg) {
    throw new UnsupportedOperationException();            //未实现，要子类自己实现
}
```



