# JUC多线程及高并发

> 相关代码证明在java学习中的juc中

## 一、volatile的理解

### 1、volatile是java虚拟机提供的轻量级的同步机制

- 保证可见性

> 线程间主内存变量修改后的通知

-  ==**不保证原子性**==

> 1）多个线程同时往主内存写值，然后出现数据丢失
>
> 2）如何保证原子性：方法加sync同步关键字（性能差），使用AtomicInteger类去处理。

- 禁止指令重排

> 比如：int x=1; int y=x; 不会先运行int y=x

### 2、AtomicInteger底层是cas

### 3、哪些地方用到过volatile

- 单例模式DCL（double check lock双端检索机制）代码：存在指令重排，所以需要添加volatile

```java
public class SingletonDemo {
    //添加volatile，防止DCL模式代码的多线程不安全。
    private static volatile SingletonDemo instance = null;

    private SingletonDemo() {
        System.out.println(Thread.currentThread().getName() + "构造方法");
    }
    //DCL模式（double check lock双端检索机制）
    //双端检索机制不一定线程安全，因为有指令重排的存在，加入volatile禁止指令重排
    /**
     * **重排说明**
     * 正常流程：1、分配对象内存；2、初始化对象；3、设置instance指向刚分配的内存地址，此时instance!=null
     * 重排情况：1、分配对象内存；2、设置instance指向刚分配的地址，此时instance不为空，但是初始化未完成；3、初始化对象
     */
    public static SingletonDemo getInstance() {
        if (instance == null) {
            //同步块前和同步块后都进行if判断
            synchronized (SingletonDemo.class) {
                if (instance == null) {
                    instance = new SingletonDemo();
                }
            }
        }
        return instance;
    }

    public static void main(String[] args) {
		//多线程调用
    }
}
```

## 二、JMM：java内存模型

### 1、定义

JMM是一种抽象的概念并不真实存在，它描述的是一组规则或规范，规范定义了程序中各个变量（字实例字段、静态字段和构成数组对象的元素）的访问方式。

### 2、JMM关于同步的规定

- 线程解锁前，必须把共享变量的值刷新回主内存。
- 线程加锁前，必须读取主内存的最新的值到自己的工作内存。
- 加锁解锁是同一把锁。

![](images/jmm01.png)

## 三、CAS（compareAndSet比较并交换）

### 1、AtomicInteger为啥不使用synchronized却保证原子性

> CAS是真实值和期望值相等进行修改，失败再进行获取最新值，然后再比较，直到成功。

**底层原理：**

- CAS思想（自旋）
- UnSafe类：基于该类可以直接操作特定内存的数据，==该类存在于sun.misc包中，其内部方法操作可以像C的指针一样直接操作内存。==CAS执行过程不允许被中断，也就是说CAS是一条CPU的原子指令。

```java
private static final jdk.internal.misc.Unsafe U = jdk.internal.misc.Unsafe.getUnsafe();    
private static final long VALUE = U.objectFieldOffset(AtomicInteger.class, "value");
public final int getAndIncrement() {
    // this：当前对象；VALUE：内存偏移量（内存地址）；1：固定写死加1
    return U.getAndAddInt(this, VALUE, 1);
}

//UnSafe类
@HotSpotIntrinsicCandidate
public final int getAndAddInt(Object o, long offset, int delta) {
    int v;
    do {
        v = getIntVolatile(o, offset);
    } while (!weakCompareAndSetInt(o, offset, v, v + delta));
    return v;
}

//可以看出UnSafe底层都是调用native本地方法
@HotSpotIntrinsicCandidate
public native int     getIntVolatile(Object o, long offset);

@HotSpotIntrinsicCandidate
public final native boolean compareAndSetInt(Object o, long offset,
                                             int expected,
                                             int x);

@HotSpotIntrinsicCandidate
public final boolean weakCompareAndSetInt(Object o, long offset,
                                          int expected,
                                          int x) {
    return compareAndSetInt(o, offset, expected, x);
}
```

### 2、CAS缺点

- 循环时间长，开销大
- 只能保证一个共享变量的原子性操作
- **引出来ABA问题**

### 3、CAS缺点的ABA问题

#### 1）说明

ABA说明：线程a假设10s执行，线程b假设2s执行，对于共享内存，线程b可以对共享变量进行修改再改回来，这样线程a就当作没有修改成功执行。

==即首尾没问题，中间会产生了操作。==

如果不介意中间变化可以不处理。

#### 2）解决方案AtomicStampedReference.java

==**原子引用+新的机制（就是修改版本号（类似时间戳））**==

- 理解原子引用AtomicReference.java

```java
public class AtomicReferenceDemo {
    public static void main(String[] args) {
        User zs = new User("zs", 3);
        User ls = new User("ls", 5);
        AtomicReference<User> atomicReference = new AtomicReference<>();
        atomicReference.set(zs);
        System.out.println(atomicReference.compareAndSet(zs, ls) + "\t" + atomicReference.get().toString());
        System.out.println(atomicReference.compareAndSet(zs, ls) + "\t" + atomicReference.get().toString());
    }
}
```

- 带有时间戳的对象引用AtomicStampedReference.java

```java
static AtomicReference<Integer> atomicReference = new AtomicReference<>(10);
static AtomicStampedReference<Integer> atomicStampedReference = new AtomicStampedReference<>(10, 1);

public static void main(String[] args) {
    System.out.println("*******************ABA问题产生*****************");
    new Thread(() -> {
        // 两次修改完成ABA操作
        atomicReference.compareAndSet(10, 20);//10改成20
        atomicReference.compareAndSet(20, 10);//20改成10
    }, "t1").start();

    new Thread(() -> {
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(atomicReference.compareAndSet(10, 30) + "\t" + atomicReference.get());
    }, "t2").start();
    try {
        TimeUnit.SECONDS.sleep(2);//保证上面线程执行完成
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println("*******************ABA问题产生*****************");
    System.out.println("*******************解决ABA问题*****************");
    new Thread(() -> {
        int stamp = atomicStampedReference.getStamp();
        System.out.println(Thread.currentThread().getName() + "\t第一次版本号" + stamp);
        try {
            TimeUnit.SECONDS.sleep(1);//保证上面线程执行完成
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        atomicStampedReference.compareAndSet(10, 20, atomicStampedReference.getStamp(), atomicStampedReference.getStamp() + 1);
        System.out.println(Thread.currentThread().getName() + "\t第二次版本号" + atomicStampedReference.getStamp());
        atomicStampedReference.compareAndSet(20, 10, atomicStampedReference.getStamp(), atomicStampedReference.getStamp() + 1);
        System.out.println(Thread.currentThread().getName() + "\t第三次版本号" + atomicStampedReference.getStamp());
    }, "t3").start();
    new Thread(() -> {
        int stamp = atomicStampedReference.getStamp();
        System.out.println(Thread.currentThread().getName() + "\t第一次版本号" + stamp);
        try {
            TimeUnit.SECONDS.sleep(3);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        boolean r = atomicStampedReference.compareAndSet(10, 30, stamp, stamp + 1);
        System.out.println(Thread.currentThread().getName() + "\t修改结果：" + r + "|最新版本号：" + atomicStampedReference.getStamp() + "|最新引用：" + atomicStampedReference.getReference());
    }, "t4").start();
}
```

## 四：集合不安全

### 1、ArrayList

1）并发异常java.util.ConcurrentModificationException

> 导致原因：在一个写的期间，并另一个线程抢过去操作

2）解决方案

- 1）new Vector<>()
 *    2）Collections.synchronizedList(new ArrayList<String>())
 *    3）new CopyOnWriteArrayList<>()

```java
// CopyOnWriteArrayList类方法
public boolean add(E e) {
    synchronized (lock) {
        Object[] es = getArray();
        int len = es.length;
        es = Arrays.copyOf(es, len + 1);
        es[len] = e;
        setArray(es);
        return true;
    }
}
```

### 2、HashSet

> 解决方式同ArrayList的第2，3点

### 3、HashMap

> 解决方式：        
>
> 1）Map<String, String> map = new ConcurrentHashMap<>();
> 2）Collections.synchronizedMap()；

## 五、java锁

### 1、公平锁、非公平锁

```java
// ReentrantLock类实现公平和非公平锁
public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
```

1）定义

- 公平锁：指多个线程按照申请锁的顺序来获取锁，类似排队打饭，先来后到
- 非公平锁：指多个线程获取锁的顺序并不是按照申请锁的顺序，有可能后申请的线程比先申请的线程优先获取所；**在高并发情况下，有可能造成优先级反转或饥饿现象**。

2）区别

- 公平锁性能差一点，但保证顺序
- 公平锁按申请锁顺序获取

### 2、可重入锁（又名递归锁）

1）定义

指的是同一线程外层函数获得锁之后，内层递归函数仍然能获取该锁的代码，在同一线程在外层方法获取锁的时候，在进入内层方法会自动获取锁。即==**线程可以进入任何一个它已经拥有的锁所同步着的代码块**==

```java
// 代码案例
public sync void method1(){
    method2();//调用method2
}
public sync void method2(){
    // 代码
}
/**
 *ReentrantLock/sync就是典型的可重入锁
 */
```

**ReentrantLock/sync就是典型的可重入锁**！！！

2）作用

可重入锁最大的作用就是避免死锁

```java
java中，解决死锁一般有如下方法：
1)尽量使用tryLock(long timeout, TimeUnit unit)的方法(ReentrantLock、ReentrantReadWriteLock)，设置超时时间，超时可以退出防止死锁。 
2)尽量使用java.util.concurrent(jdk 1.5以上)包的并发类代替手写控制并发，比较常用的是ConcurrentHashMap、ConcurrentLinkedQueue、AtomicBoolean等等，实际应用中java.util.concurrent.atomic十分有用，简单方便且效率比使用Lock更高 
3)尽量降低锁的使用粒度，尽量不要几个功能用同一把锁 
4)尽量减少同步的代码块
```

### 3、自旋锁

1）定义

指尝试获取锁的线程不会立即阻塞，而是==**采用循环的方式去尝试获取锁**==，这样的好处是减少线程上下文切换的消耗，缺点是循环灰消耗cpu。

CAS思想（自旋）

```java
//UnSafe类
@HotSpotIntrinsicCandidate
public final int getAndAddInt(Object o, long offset, int delta) {
    int v;
    do {
        v = getIntVolatile(o, offset);
    } while (!weakCompareAndSetInt(o, offset, v, v + delta));
    return v;
}
```

4、独占锁（写锁）/共享锁（读锁）/互斥锁