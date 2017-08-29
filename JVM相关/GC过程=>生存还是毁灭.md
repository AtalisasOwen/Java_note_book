# GC过程=&gt;生存还是毁灭

## 堆的回收

* ### 即使是可达性算法中不可达的对象，也并非一定会死，至少要经历两次标记过程才会真正死亡：

  * ### 如果发现对象是不可达的，标记第一次并且进行一次筛选。筛选条件是是否有必要执行finialize\(\)方法，当对象没有覆盖finalize\(\)方法，或者finalize\(\)已经被调用过了，则视为没有必要执行。
  * ### 如果被判定有必要执行finalize\(\)方法，那么该对象将会放置在一个F-Queue的队列中，并在稍后由一个由虚拟机自动建立的、低优先级的Finalizer线程去执行它的finalize\(\)。但不保证会运行结束。
  * ### 因此在finalize\(\)中对象还有逃脱的机会（仅供演示，没有意义）

```java
public class FinalizeEscapeGC {
    public static FinalizeEscapeGC SAVE_HOOK = null;

    public void isAlive(){
        System.out.println("yes ,i am still alive");
    }

    @Override
    protected void finalize() throws Throwable {
        super.finalize();
        System.out.println("finalize method executed");
        FinalizeEscapeGC.SAVE_HOOK = this;
    }

    public static void main(String...args) throws Throwable {
        SAVE_HOOK = new FinalizeEscapeGC();
        
        //对象第一次自救，成功
        SAVE_HOOK = null;
        System.gc();
        Thread.sleep(500);
        if (SAVE_HOOK != null){
            SAVE_HOOK.isAlive();
        }else {
            System.out.println("i am dead");
        }
        
        //由于finalize()只能调用一次，所以自救失败
        SAVE_HOOK = null;
        System.gc();
        Thread.sleep(500);
        if (SAVE_HOOK != null){
            SAVE_HOOK.isAlive();
        }else {
            System.out.println("i am dead");
        }

    }

}
```

## 方法区的回收：废弃常量和无用的类

* ### 例，常量池中有一个abc的字符串，但是没有任何一个String对象引用abc。有必要的话，这个常量会被清除。
* ### 无用的类判断，同时满足下列条件：

  * ### 1.该类的所有实例被回收\(Java堆中不存在该类的实例\)
  * ### 2.加载该类的ClassLoader已经被回收
  * ### 3.该类对应的Class对象没有被任何地方引用，无法在任何地方通过反射访问该类的方法。





