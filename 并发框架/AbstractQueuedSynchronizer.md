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

* ### API说明

  | protected boolean tryAcquire\(int arg\) | 排他模式的获取这个状态。这个方法的实现需要查询当前状态是否允许获取,然后再获取\(使用compareAndSetState\)状态 |
  | :--- | :--- |
  | protected boolean tryRelease\(int arg\) | 排他模式的释放状态 |
  | protected int tryAcquireShared\(int arg\) | 共享模式获取状态 |
  | protected int tryReleaseShared\(int arg\) | 共享模式释放状态 |
  | protected boolean isHeldExclusively | 排他模式。状态是否被占有。 |
* ### 示例

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



