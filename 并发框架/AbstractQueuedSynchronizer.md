# AbstractQueuedSynchronizer

* ### 这货是整个concurrent包的灵魂类
* ### 子类通过继承并需要实现它的方法来管理状态，通过acquire和release的方式来操作状态。然而多线程环境中对状态的操纵必须保证原子性，因此子类需要使用这个同步器提供的getState\(\)、setState\(int\)、compareAndSetState\(int,int\)
* ### 该同步器可以作为排他模式或者共享模式来运行。如ReentrantReadWriteLock内部，写锁的实现为排他，而读锁的实现为共享。
* ### 内部实现依赖于FIFO队列，队列中的元素Node保存着线程引用和线程状态的容器，每个线程的访问可以视为队列中的一个节点Node

```java
Node {
        /**
         * SIGNAL:      该节点的下一个节点被堵塞了，所以当前节点在取消或者完成后，
         *              必须唤醒它的下一个节点。为了避免竞态，acquire方法需要
         *              指示需要一个信号。然后重试原子获取
         * CANCELLED:   这个节点由于超时或者打断而取消了。一旦变成这个状态后，不会
         *              发生改变，此外，也不会阻塞队列了
         * CONDITION:   该节点在条件队列中，该节点不会被用作同步队列的节点，直到状态
         *              被改变（改变后，会为0）
         * PROPAGATE:   当共享模式的节点释放后，会将锁传递给下一个节点用doReleaseShared
         *              来确保传递会持续
         * 0:           表示什么都不做
         * 非负数:       表示该节点不需要被通知
         *
         * 这个变量会使用CAS来改变
         */
        volatile int waitStatus;int waitStatus;
         /**
         * 指向上一个节点
         * 当当前节点取消时，会使用node.prev.next = node.next来删除节点
         * 头节点不可能被取消，因为头结点表示成功获取了锁
         * 而取消的节点也不可能获取锁，并且线程只能取消自己，不能取消别的线程
         */
        volatile Node prev;
        //指向下一个节点
        volatile Node next;
        //该节点保存的线程
        volatile Thread thread;

        /**
         * 连接下一个在等待中的线程，或者SHARED节点
         * 取值为SHARED或者下一个节点
         */
        Node nextWaiter;
}
```

* ### 同步队列的结构

### ![](/assets/21.png)

### head节点: 当前占有锁且正在运行

* ### API说明

  | protected boolean tryAcquire\(int arg\) | 排他模式的获取这个状态。这个方法的实现需要查询当前状态是否允许获取,然后再获取\(使用compareAndSetState\)状态 |
  | :--- | :--- |
  | protected boolean tryRelease\(int arg\) | 排他模式的释放状态 |
  | protected int tryAcquireShared\(int arg\) | 共享模式获取状态 |
  | protected int tryReleaseShared\(int arg\) | 共享模式释放状态 |
  | protected boolean isHeldExclusively | 排他模式。状态是否被占有。 |
* # 示例=&gt;独占模式

```java
//独占模式
public class Mutex implements Lock,Serializable {

    private static class Sync extends AbstractQueuedSynchronizer{

        //状态为0时，获取锁
        @Override
        protected boolean tryAcquire(int arg) {
            assert arg == 1;
            //compareAndSetState()成功说明获取锁成功，使用CAS算法
            if (compareAndSetState(0,1)){
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
            return false;
        }

        @Override
        protected boolean tryRelease(int arg) {
            assert arg == 1;
            if (getState() == 0) throw new IllegalMonitorStateException();
            setExclusiveOwnerThread(null);
            setState(0);
            return true;
        }

        //是否处于被占有状态
        @Override
        protected boolean isHeldExclusively() {
            return getState() == 1;
        }

        Condition newCondition(){
            return new ConditionObject();
        }
    }

    private final Sync sync = new Sync();

    @Override
    public void lock() {
        sync.acquire(1);
    }

    @Override
    public void lockInterruptibly() throws InterruptedException {
        sync.acquireInterruptibly(1);
    }

    @Override
    public boolean tryLock() {
        return sync.tryAcquire(1);
    }

    @Override
    public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
        return sync.tryAcquireSharedNanos(1,unit.toNanos(time));
    }

    @Override
    public void unlock() {
        sync.release(1);

    }

    @Override
    public Condition newCondition() {
        return sync.newCondition();
    }
}
```

* ### public final void acquire\(int arg\)

  * ### 状态的维护：通过setState和getState来修改状态
  * ### 状态的获取：一旦成功地获取了状态，就被设置为头结点
  * ### 同步器队列的维护：在获取资源未果的过程中条件不符合的情况下\(不该自己，前驱节点不是头节点或者没有获取到资源\)进入睡眠状态，停止线程调度器对当前节点线程的调度
  * ### 在获取资源未果的过程中条件不符合的情况下\(不该自己，前驱节点不是头节点或者没有获取到资源\)进入睡眠状态，停止线程调度器对当前节点线程的调度

```java
    public final void acquire(int arg) {
        //1.首先尝试获取，成功直接return
        if (!tryAcquire(arg) &&
        //2.获取失败，(addWaiter)将当前节点构造成Node，并进入FIFO队列
        //3.(acquireQueued)再次尝试获取，如果没有获取到，那么将当前线程从调度器上摘下，进入等待状态
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg)) 

            selfInterrupt();
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
//尾节点为空，原子化的分配一个头结点，并将尾节点指向头节点，这是初始化
//在循环中，直到当前节点入队为止
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

![](/assets/t_9683_1379328542_928191748.png)

* ### public final boolean release\(int arg\)

```java
    public final boolean release(int arg) {
        if (tryRelease(arg)){//尝试释放状态
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);//唤醒后继节点所包含的线程
            return true;
        }
        return false;
    }

    private void unparkSuccessor(Node node) {
        //对当前节点设置为完成状态
        int ws = node.waitStatus;
        if (ws < 0)
            compareAndSetWaitStatus(node, ws, 0);
        //获取后继节点，如果满足要求，那么唤醒
        //如果没满足条件，从队列尾寻找符合要求的节点唤醒
        Node s = node.next;
        if (s == null || s.waitStatus > 0) {
            s = null;
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0)
                    s = t;
        }
        if (s != null)
            LockSupport.unpark(s.thread);
    }
```

* ### protected boolean tryAcquire\(int arg\)

#### 这个是自定义同步器需要实现的方法, 用于独占模式

#### 下面是ReentrantLock的非公平锁和公平锁的实现

```java
      final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }

       protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
```

* ### public final void acquireInterruptibly\(int arg\)

该方法提供获取状态能力，当然在无法获取状态的情况下会进入sync队列进行排队，这类似acquire，但是和acquire不同的地方在于它能够在外界对当前线程进行中断的时候提前结束获取状态的操作，换句话说，就是在类似synchronized获取锁时，外界能够对当前线程进行中断，并且获取锁的这个操作能够响应中断并提前返回。一个线程处于synchronized块中或者进行同步I/O操作时，对该线程进行中断操作，这时该线程的中断标识位被设置为true，但是线程依旧继续运行。

如果在获取一个通过网络交互实现的锁时，这个锁资源突然进行了销毁，那么使用acquireInterruptibly的获取方式就能够让该时刻尝试获取锁的线程提前返回。而同步器的这个特性被实现Lock接口中的lockInterruptibly方法。根据Lock的语义，在被中断时，lockInterruptibly将会抛出InterruptedException来告知使用者。

```java
    public final void acquireInterruptibly(int arg)
            throws InterruptedException {
        if (Thread.interrupted())                            //检查当前线程是否中断
            throw new InterruptedException();
        if (!tryAcquire(arg))                                //尝试获取，成功直接返回
            doAcquireInterruptibly(arg);                     //否则构造节点并入队列
    }
    
    private void doAcquireInterruptibly(int arg)
        throws InterruptedException {
        final Node node = addWaiter(Node.EXCLUSIVE);
        boolean failed = true;
        try {
            for (;;) {
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return;
                }
                if (shouldParkAfterFailedAcquire(p, node) &&    //中断检查
                    parkAndCheckInterrupt())
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```

* ## private boolean doAcquireNanos\(int arg, long nanosTimeout\)

```java
    private boolean doAcquireNanos(int arg, long nanosTimeout)
            throws InterruptedException {
        if (nanosTimeout <= 0L)
            return false;
        final long deadline = System.nanoTime() + nanosTimeout;
        final Node node = addWaiter(Node.EXCLUSIVE);                //加入队列
        boolean failed = true;
        try {
            for (;;) {
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) {                //前驱节点或者获取锁了，直接返回
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return true;
                }
                nanosTimeout = deadline - System.nanoTime();      
                if (nanosTimeout <= 0L)
                    return false;
                if (shouldParkAfterFailedAcquire(p, node) &&        //获取失败，则等待一段时间
                    nanosTimeout > spinForTimeoutThreshold)
                    LockSupport.parkNanos(this, nanosTimeout);
                if (Thread.interrupted())
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```

![](/assets/t_9683_1379328891_568034081.png)

* # 示例=&gt;共享模式
* ### public final void acquireShared\(int arg\)

共享模式里，多个线程可以同时读文件，但此时写操作是堵塞的。

![](/assets/t_9683_1379328959_1702388031.png)

```java
    public final void acquireShared(int arg) {
        if (tryAcquireShared(arg) < 0)                //如果获取成功则立刻返回
            doAcquireShared(arg);                     //否则进入sync队列
    }
    
    protected int tryAcquireShared(int arg) {
        throw new UnsupportedOperationException();
    }
    
    private void doAcquireShared(int arg) {
        final Node node = addWaiter(Node.SHARED);    
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {                                //循环内判断退出队列条件
                final Node p = node.predecessor();
                if (p == head) {
                    int r = tryAcquireShared(arg);
                    if (r >= 0) {
                        setHeadAndPropagate(node, r); //如果当前节点的前驱节点是头结点并且获取共享状态成功
                        p.next = null; // help GC
                        if (interrupted)
                            selfInterrupt();
                        failed = false;
                        return;
                    }
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }

```

* ### protected int tryAcquireShared\(int arg\)和protected boolean tryReleaseShared\(int release\)

#### CountDownLatch的实现

```java
        protected int tryAcquireShared(int acquires) {
            return (getState() == 0) ? 1 : -1;
        }
        
        protected boolean tryReleaseShared(int releases) {
            // Decrement count; signal when transition to zero
            for (;;) {
                int c = getState();
                if (c == 0)
                    return false;
                int nextc = c-1;
                if (compareAndSetState(c, nextc))
                    return nextc == 0;
            }
        }
```



