---
title: java原子类及内部原理
date: 2019-10-12 16:46:43
tags:
---
### 一、引入

       原子是世界上的最小单位，具有不可分割性。比如 a=0；（a非long和double类型） 这个操作是不可分割的，那么我们说这个操作是原子操作。再比如：a++；

这个操作实际是a = a + 1；是可分割的，所以他不是一个原子操作。非原子操作都会存在线程安全问题，需要我们使用同步技术（sychronized）来让它变成一

个原子操作。

       但是，像i++这种非原子操作，我们除了使用synchroinzed关键字实现同步外，还可以使用java.util.concurrent.atomic提供的线程安全的原子类来实现，例如

AtomicInteger、AtomicLong、AtomicReference等。下面我们就基于AtomicInteger为例，来看看其内部实现。

### 二、AtomicInteger的内部实现

public class AtomicInteger extends Number implements java.io.Serializable {
    private static final long serialVersionUID = 6214790243416807050L;

    // setup to use Unsafe.compareAndSwapInt for updates
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;

    static {
        try {
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }

    private volatile int value;
这段代码我们需要注意一下几个方面：

（1）unsafe字段，AtomicInteger包含了一个Unsafe类的实例，unsafe就是用来实现CAS机制的，CAS机制我们在后面会讲到；

（2）value字段，表示当前对象代码的基本类型的值，AtomicInteger是int型的线程安全包装类，value就代码了AtomicInteger的值。注意，这个字段是volatile的。

（3）valueOfset，指是value在内存中的偏移量，也就是在内存中的地址，通过Unsafe.objectFieldOffset(Field f)获取。这个值在使用CAS机制的时候会用到。

下面我们来看一个AtomicInteger类中的主要方法getAndIncrement()，也就相当于i++操作，只不过它是线程安全的，其实现代码如下：

    public final int getAndIncrement() {
             for (;;) {
                 int current = get();
                 int next = current + 1;
                 if (compareAndSet(current, next))
                     return current;
             }
     }
       这个方法的做法为先获取到当前的 value 属性值，然后将 value 加 1，赋值给一个局部的 next 变量，然而，这两步都是非线程安全的，但是内部有一个死循环，

不断去做compareAndSet操作，直到成功为止，也就是修改的根本在compareAndSet方法里面。compareAndSet()方法的代码如下：

public final boolean compareAndSet(int expect, int update) {
        return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
 }
      compareAndSet 传入的为执行方法时获取到的 value 属性值，update为加 1 后的值， compareAndSet所做的为调用 Sun 的 UnSafe 的 compareAndSwapInt

方法来完成，此方法为 native 方法，compareAndSwapInt 基于的是CPU 的 CAS指令来实现的。下面我们将详细的来介绍一下CAS的实现原理。

### 三、CAS机制

CAS是英文单词Compare And Swap的缩写，翻译过来就是比较并替换。CAS机制当中使用了3个基本操作数：

（1）内存地址V，也就是AtomicInteger中的valueOffset。

（2）旧的预期值A，也就是getAndIncrement方法中的current。

（3）要修改的新值B，也就是getAndIncrement方法中的next。

CAS机制中，更新一个变量的时候，只有当变量的预期值A和内存地址V当中的实际值相同时，才会将内存地址V对应的值修改为B。下面我们来看一个具体的例子：

（1）在内存地址V当中，存储着值为10的变量。

（2）此时线程1想要把变量的值增加1。对线程1来说，旧的预期值A=10，要修改的新值B=11。

（3）但是，在线程1要提交更新之前，另一个线程2抢先一步，把内存地址V中的变量值率先更新成了11。

（4）此时，线程1开始提交更新，首先进行A和地址V的实际值比较（Compare），发现A不等于V的实际值，提交失败。

（5）线程1重新获取内存地址V的当前值，并重新计算想要修改的新值。此时对线程1来说，A=11，B=12。这个重新尝试的过程被称为自旋。

（6）这一次比较幸运，没有其他线程改变地址V的值。线程1进行Compare，发现A和地址V的实际值是相等的。

（7）线程1进行替换，把地址V的值替换为B，也就是12。

      对比Synchronized，我们可以发现，Synchronized属于悲观锁，悲观地认为程序中的并发情况严重，所以严防死守。CAS属于乐观锁，乐观地认为程序中的并发情况不那么

严重，所以让线程不断去尝试更新。

但是CAS机制通常也存在以下缺点：

（1）ABA问题

       如果V的初始值是A，在准备赋值的时候检查到它仍然是A，那么能说它没有改变过吗？也许V经历了这样一个过程：它先变成了B，又变成了A，使用CAS检查时

以为它没变，其实却已经改变过了。

（2）CPU开销较大

     在并发量比较高的情况下，如果许多线程反复尝试更新某一个变量，却又一直更新不成功，循环往复，会给CPU带来很大的压力。

（3）不能保证代码块的原子性

     CAS机制所保证的只是一个变量的原子性操作，而不能保证整个代码块的原子性。比如需要保证3个变量共同进行原子性的更新，就不得不使用Synchronized了。