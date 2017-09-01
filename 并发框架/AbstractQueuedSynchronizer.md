# AbstractQueuedSynchronizer

* ### 这货是整个concurrent包的灵魂类
* ### 子类通过继承并需要实现它的方法来管理状态，通过acquire和release的方式来操作状态。然而多线程环境中对状态的操纵必须保证原子性，因此子类需要使用这个同步器提供的getState\(\)、setState\(int\)、compareAndSetState\(int,int\)
* ### 该同步器可以作为排他模式或者共享模式来运行。如ReentrantReadWriteLock内部



