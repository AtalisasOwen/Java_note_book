# String，StringBuilder，StringBuffer

* ### String: final类，内部有一个value\[\], 用+拼接的过程：ClassLoader.loadClass\(StringBuilder\)
* ### StringBuffer与StringBuilder都是继承于AbstractStringBuilder
* ### StringBuffer线程安全，StringBuilder线程不安全，但单线程的效率上StringBuilder比较高。



