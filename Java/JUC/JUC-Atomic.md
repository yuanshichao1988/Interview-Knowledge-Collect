## JUC Atomic 原子操作  

从相对简单的Atomic入手（java.util.concurrent是基于Queue的并发包，而Queue，很多情况下使用到了Atomic操作，因此首先从这里开始）。很多情况下我们只是需要一个简单的、高效的、线程安全的递增递减方案。注意，这里有三个条件：简单，意味着程序员尽可能少的操作底层或者实现起来要比较容易；高效意味着耗用资源要少，程序处理速度要快；线程安全也非常重要，这个在多线程下能保证数据的正确性。这三个条件看起来比较简单，但是实现起来却难以令人满意。

通常情况下，在Java里面，++i或者--i不是线程安全的，这里面有三个独立的操作：或者变量当前值，为该值+1/-1，然后写回新的值。在没有额外资源可以利用的情况下，只能使用加锁才能保证读-改-写这三个操作时“原子性”的。

Doug Lea在未将[backport-util-concurrent](http://backport-jsr166.sourceforge.net/)合并到[JSR 166](http://jcp.org/en/jsr/detail?id=166)里面来之前，是采用纯Java实现的，于是不可避免的采用了synchronized关键字。

public final synchronized void set(int newValue);

public final synchronized int getAndSet(int newValue);

public final synchronized int incrementAndGet();

同时在变量上使用了volatile （后面会具体来讲volatile到底是个什么东东）来保证get()的时候不用加锁。尽管synchronized的代价还是很高的，但是在没有JNI的手段下纯Java语言还是不能实现此操作的。

JSR 166提上日程后，backport-util-concurrent就合并到JDK 5.0里面了，在这里面重复使用了现代CPU的特性来降低锁的消耗。后本章的最后小结中会谈到这些原理和特性。在此之前先看看API的使用。

一切从java.util.concurrent.atomic.AtomicInteger开始。

int addAndGet(int delta)
          以原子方式将给定值与当前值相加。 实际上就是等于线程安全版本的i =i+delta操作。

boolean compareAndSet(int expect, int update)
          如果当前值 == 预期值，则以原子方式将该值设置为给定的更新值。 如果成功就返回true，否则返回false，并且不修改原值。

int decrementAndGet()
          以原子方式将当前值减 1。 相当于线程安全版本的--i操作。

int get()
          获取当前值。

int getAndAdd(int delta)
          以原子方式将给定值与当前值相加。 相当于线程安全版本的t=i;i+=delta;return t;操作。

int getAndDecrement()
          以原子方式将当前值减 1。 相当于线程安全版本的i--操作。

int getAndIncrement()
          以原子方式将当前值加 1。 相当于线程安全版本的i++操作。

int getAndSet(int newValue)
          以原子方式设置为给定值，并返回旧值。 相当于线程安全版本的t=i;i=newValue;return t;操作。

int incrementAndGet()
          以原子方式将当前值加 1。 相当于线程安全版本的++i操作。 

void lazySet(int newValue)
          最后设置为给定值。 延时设置变量值，这个等价于set()方法，但是由于字段是volatile类型的，因此次字段的修改会比普通字段（非volatile字段）有稍微的性能延时（尽管可以忽略），所以如果不是想立即读取设置的新值，允许在“后台”修改值，那么此方法就很有用。如果还是难以理解，这里就类似于启动一个后台线程如执行修改新值的任务，原线程就不等待修改结果立即返回（这种解释其实是不正确的，但是可以这么理解）。

void set(int newValue)
          设置为给定值。 直接修改原始值，也就是i=newValue操作。

boolean weakCompareAndSet(int expect, int update)
          如果当前值 == 预期值，则以原子方式将该设置为给定的更新值。JSR规范中说：以原子方式读取和有条件地写入变量但*不* 创建任何 happen-before 排序，因此不提供与除 `weakCompareAndSet` 目标外任何变量以前或后续读取或写入操作有关的任何保证。大意就是说调用weakCompareAndSet时并不能保证不存在happen-before的发生（也就是可能存在指令重排序导致此操作失败）。但是从Java源码来看，其实此方法并没有实现JSR规范的要求，最后效果和compareAndSet是等效的，都调用了unsafe.compareAndSwapInt()完成操作。

下面的代码是一个测试样例，为了省事就写在一个方法里面来了。 

```
package xylz.study.concurrency.atomic;

import java.util.concurrent.atomic.AtomicInteger;

import org.junit.Test;

import static org.junit.Assert.*;

public class AtomicIntegerTest {

    @Test
    public void testAll() throws InterruptedException{
        final AtomicInteger value = new AtomicInteger(10);
        assertEquals(value.compareAndSet(1, 2), false);
        assertEquals(value.get(), 10);
        assertTrue(value.compareAndSet(10, 3));
        assertEquals(value.get(), 3);
        value.set(0);
        //
        assertEquals(value.incrementAndGet(), 1);
        assertEquals(value.getAndAdd(2),1);
        assertEquals(value.getAndSet(5),3);
        assertEquals(value.get(),5);
        //
        final int threadSize = 10;
        Thread[] ts = new Thread[threadSize];
        for (int i = 0; i < threadSize; i++) {
            ts[i] = new Thread() {
                public void run() {
                    value.incrementAndGet();
                }
            };
        }
        //
        for(Thread t:ts) {
            t.start();
        }
        for(Thread t:ts) {
            t.join();
        }
        //
        assertEquals(value.get(), 5+threadSize);
    }

}
```

