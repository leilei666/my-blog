---
title: ThreadLocal原理分析
date: 2019-08-09 13:40:29
tags:
---
### 简介

早在JDK 1.2的版本中就提供java.lang.ThreadLocal，ThreadLocal为解决多线程程序的并发问题提供了一种新的思路。使用这个工具类可以很简洁地编写出优美的多线程程序。当使用ThreadLocal维护变量时，ThreadLocal为每个使用该变量的线程提供独立的变量副本，所以每一个线程都可以独立地改变自己的副本，而不会影响其它线程所对应的副本。从线程的角度看，目标变量就象是线程的本地变量，这也是类名中“Local”所要表达的意思。所以，在Java中编写线程局部变量的代码相对来说要笨拙一些，因此造成线程局部变量没有在Java开发者中得到很好的普及。人们常说，锁是一种以时间换空间的机制，而ThreadLocal正好是以空间换时间的。

### 和锁的比较

为什么要先强调这一点，因为从简介上看，容易使人们联想到ThreadLocal似乎是一种解决Java多线程环境中线程同步与线程安全方法，其实不然，这里要设计到两个概念：线程安全，线程同步，事实上，ThreadLocal只解决线程安全的问题，并不能解决线程同步的问题，这也造成在刚学习的时候，我总是在苦想，ThreadLocal既然为每个线程拷贝一份变量，那怎么再进行同步呢？查了很多资料后才想明白，ThreadLocal并不是用来解决线程同步的，所以它与锁可以说是没有什么关系的，彼此各有所长，不能代替。有人这样描述：ThreadLocal解决的是同一个线程内的资源共享问题，而synchronized 解决的是多个线程间的资源共享问题，我觉得是有道理的！

#### ThreadLocal类接口很简单，只有4个方法，我们先来了解一下：

    void set(T value)设置当前线程的线程局部变量的值。
    public T get()该方法返回当前线程所对应的线程局部变量。
    public void remove()将当前线程局部变量的值删除，目的是为了减少内存的占用，该方法是JDK 5.0新增的方法。需要指出的是，当线程结束后，对应该线程的局部变量将自动被垃圾回收，所以显式调用该方法清除线程的局部变量并不是必须的操作，但它可以加快内存回收的速度。
    protected T initialValue()返回该线程局部变量的初始值，该方法是一个protected的方法，显然是为了让子类覆盖而设计的。这个方法是一个延迟调用方法，在线程第1次调用get()或set(T value)时才执行，并且仅执行1次。ThreadLocal中的缺省实现直接返回一个null

值得一提的是，在JDK5.0中，ThreadLocal已经支持泛型，该类的类名已经变为ThreadLocal<T>。API方法也相应进行了调整，新版本的API方法分别是void set(T value)、T get()以及T initialValue()。ThreadLocal是如何做到为每一个线程维护变量的副本的呢？其实实现的思路很简单：在ThreadLocal类中有一个Map，用于存储每一个线程的变量副本，Map中元素的键为线程对象，而值对应线程的变量副本。我们自己就可以提供一个简单的实现版本：
 
    package com.test;
 
    public class TestNum {
        // ①通过匿名内部类覆盖ThreadLocal的initialValue()方法，指定初始值
        private static ThreadLocal<Integer> seqNum = new ThreadLocal<Integer>() {
            public Integer initialValue() {
                return 0;
            }
        };
     
        // ②获取下一个序列值
        public int getNextNum() {
            seqNum.set(seqNum.get() + 1);
            return seqNum.get();
        }
     
        public static void main(String[] args) {
            TestNum sn = new TestNum();
            // ③ 3个线程共享sn，各自产生序列号
            TestClient t1 = new TestClient(sn);
            TestClient t2 = new TestClient(sn);
            TestClient t3 = new TestClient(sn);
            t1.start();
            t2.start();
            t3.start();
        }
     
        private static class TestClient extends Thread {
            private TestNum sn;
     
            public TestClient(TestNum sn) {
                this.sn = sn;
            }
     
            public void run() {
                for (int i = 0; i < 3; i++) {
                    // ④每个线程打出3个序列值
                    System.out.println("thread[" + Thread.currentThread().getName() + "] --> sn["
                             + sn.getNextNum() + "]");
                }
            }
        }
    }

通常我们通过匿名内部类的方式定义ThreadLocal的子类，提供初始的变量值，如例子中①处所示。TestClient线程产生一组序列号，在③处，我们生成3个TestClient，它们共享同一个TestNum实例。运行以上代码，在控制台上输出以下的结果：

    thread[Thread-0] --> sn[1]
    thread[Thread-1] --> sn[1]
    thread[Thread-2] --> sn[1]
    thread[Thread-1] --> sn[2]
    thread[Thread-0] --> sn[2]
    thread[Thread-1] --> sn[3]
    thread[Thread-2] --> sn[2]
    thread[Thread-0] --> sn[3]
    thread[Thread-2] --> sn[3]

考察输出的结果信息，我们发现每个线程所产生的序号虽然都共享同一个TestNum实例，但它们并没有发生相互干扰的情况，而是各自产生独立的序列号，这是因为我们通过ThreadLocal为每一个线程提供了单独的副本。
 ### Thread同步机制的比较
 ThreadLocal和线程同步机制相比有什么优势呢？ThreadLocal和线程同步机制都是为了解决多线程中相同变量的访问冲突问题。在同步机制中，通过对象的锁机制保证同一时间只有一个线程访问变量。这时该变量是多个线程共享的，使用同步机制要求程序慎密地分析什么时候对变量进行读写，什么时候需要锁定某个对象，什么时候释放对象锁等繁杂的问题，程序设计和编写难度相对较大。而ThreadLocal则从另一个角度来解决多线程的并发访问。ThreadLocal会为每一个线程提供一个独立的变量副本，从而隔离了多个线程对数据的访问冲突。因为每一个线程都拥有自己的变量副本，从而也就没有必要对该变量进行同步了。ThreadLocal提供了线程安全的共享对象，在编写多线程代码时，可以把不安全的变量封装进ThreadLocal。由于ThreadLocal中可以持有任何类型的对象，低版本JDK所提供的get()返回的是Object对象，需要强制类型转换。但JDK 5.0通过泛型很好的解决了这个问题，在一定程度地简化ThreadLocal的使用。

同一线程贯通三层这样你就可以根据需要，将一些非线程安全的变量以ThreadLocal存放，在同一次请求响应的调用线程中，所有关联的对象引用到的都是同一个变量。下面的实例能够体现Spring对有状态Bean的改造思路：代码清单3 TestDao：非线程安全

    package com.test;
     
    import java.sql.Connection;
    import java.sql.SQLException;
    import java.sql.Statement;
     
    public class TestDao {
        private Connection conn;// ①一个非线程安全的变量
     
        public void addTopic() throws SQLException {
            Statement stat = conn.createStatement();// ②引用非线程安全变量
            // …
        }
    }

由于①处的conn是成员变量，因为addTopic()方法是非线程安全的，必须在使用时创建一个新TopicDao实例（非singleton）。下面使用ThreadLocal对conn这个非线程安全的“状态”进行改造：代码清单4 TestDao：线程安全

    package com.test;
     
    import java.sql.Connection;
    import java.sql.SQLException;
    import java.sql.Statement;
     
    public class TestDaoNew {
        // ①使用ThreadLocal保存Connection变量
        private static ThreadLocal<Connection> connThreadLocal = new ThreadLocal<Connection>();
     
        public static Connection getConnection() {
            // ②如果connThreadLocal没有本线程对应的Connection创建一个新的Connection，
            // 并将其保存到线程本地变量中。
            if (connThreadLocal.get() == null) {
                Connection conn = getConnection();
                connThreadLocal.set(conn);
                return conn;
            } else {
                return connThreadLocal.get();// ③直接返回线程本地变量
            }
        }
     
        public void addTopic() throws SQLException {
            // ④从ThreadLocal中获取线程对应的Connection
            Statement stat = getConnection().createStatement();
        }
    }

不同的线程在使用TopicDao时，先判断connThreadLocal.get()是否是null，如果是null，则说明当前线程还没有对应的Connection对象，这时创建一个Connection对象并添加到本地线程变量中；如果不为null，则说明当前的线程已经拥有了Connection对象，直接使用就可以了。这样，就保证了不同的线程使用线程相关的Connection，而不会使用其它线程的Connection。因此，这个TopicDao就可以做到singleton共享了。当然，这个例子本身很粗糙，将Connection的ThreadLocal直接放在DAO只能做到本DAO的多个方法共享Connection时不发生线程安全问题，但无法和其它DAO共用同一个Connection，要做到同一事务多DAO共享同一Connection，必须在一个共同的外部类使用ThreadLocal保存Connection。ConnectionManager.java

    package com.test;
     
    import java.sql.Connection;
    import java.sql.DriverManager;
    import java.sql.SQLException;
 
    public class ConnectionManager {
     
        private static ThreadLocal<Connection> connectionHolder = new ThreadLocal<Connection>() {
            @Override
            protected Connection initialValue() {
                Connection conn = null;
                try {
                    conn = DriverManager.getConnection(
                            "jdbc:mysql://localhost:3306/test", "username",
                            "password");
                } catch (SQLException e) {
                    e.printStackTrace();
                }
                return conn;
            }
        };
     
        public static Connection getConnection() {
            return connectionHolder.get();
        }
     
        public static void setConnection(Connection conn) {
            connectionHolder.set(conn);
        }
    }

### ThreadLocal\<T>
那么到底ThreadLocal类是如何实现这种“为每个线程提供不同的变量拷贝”的呢？先来看一下ThreadLocal的set()方法的源码是如何实现的：

    /**
         * Sets the current thread's copy of this thread-local variable
         * to the specified value.  Most subclasses will have no need to
         * override this method, relying solely on the {@link #initialValue}
         * method to set the values of thread-locals.
         *
         * @param value the value to be stored in the current thread's copy of
         *        this thread-local.
         */
        public void set(T value) {
            Thread t = Thread.currentThread();
            ThreadLocalMap map = getMap(t);
            if (map != null)
                map.set(this, value);
            else
                createMap(t, value);
        }

在这个方法内部我们看到，首先通过getMap(Thread t)方法获取一个和当前线程相关的ThreadLocalMap，然后将变量的值设置到这个ThreadLocalMap对象中，当然如果获取到的ThreadLocalMap对象为空，就通过createMap方法创建。线程隔离的秘密，就在于ThreadLocalMap这个类。ThreadLocalMap是ThreadLocal类的一个静态内部类，它实现了键值对的设置和获取（对比Map对象来理解），每个线程中都有一个独立的ThreadLocalMap副本，它所存储的值，只能被当前线程读取和修改。ThreadLocal类通过操作每一个线程特有的ThreadLocalMap副本，从而实现了变量访问在不同线程中的隔离。因为每个线程的变量都是自己特有的，完全不会有并发错误。还有一点就是，ThreadLocalMap存储的键值对中的键是this对象指向的ThreadLocal对象，而值就是你所设置的对象了。为了加深理解，我们接着看上面代码中出现的getMap和createMap方法的实现：

    /**
     * Get the map associated with a ThreadLocal. Overridden in
     * InheritableThreadLocal.
     *
     * @param  t the current thread
     * @return the map
     */
    ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }
 
    /**
     * Create the map associated with a ThreadLocal. Overridden in
     * InheritableThreadLocal.
     *
     * @param t the current thread
     * @param firstValue value for the initial entry of the map
     * @param map the map to store.
     */
    void createMap(Thread t, T firstValue) {
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }

接下来再看一下ThreadLocal类中的get()方法:

        /**
         * Returns the value in the current thread's copy of this
         * thread-local variable.  If the variable has no value for the
         * current thread, it is first initialized to the value returned
         * by an invocation of the {@link #initialValue} method.
         *
         * @return the current thread's value of this thread-local
         */
        public T get() {
            Thread t = Thread.currentThread();
            ThreadLocalMap map = getMap(t);
            if (map != null) {
                ThreadLocalMap.Entry e = map.getEntry(this);
                if (e != null)
                    return (T)e.value;
            }
            return setInitialValue();
        }

再来看setInitialValue()方法：

     /**
         * Variant of set() to establish initialValue. Used instead
         * of set() in case user has overridden the set() method.
         *
         * @return the initial value
         */
        private T setInitialValue() {
            T value = initialValue();
            Thread t = Thread.currentThread();
            ThreadLocalMap map = getMap(t);
            if (map != null)
                map.set(this, value);
            else
                createMap(t, value);
            return value;
        }


 