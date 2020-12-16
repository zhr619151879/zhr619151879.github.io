---
title: "[面试]-校招Java知识"
date: 2020-12-16 11:30:54
description: "Java"
tags:
-
series:
- Interview
categories:
- Code

image: images/feature2/workflow.png
---





## 概述



### 基础

#### 数据类型(字节 1byte=8bits)



- byte/1
- char/2
- short/2
- int/4
- float/4
- long/8
- double/8
- boolean/~ : true、false，可以使用 1 bit 来存储，但是具体大小没有明确规定。JVM 会在编译时期将 boolean 类型的数据转换为 int，使用 1 来表示 true，0 表示 false。JVM 支持 boolean 数组，但是是通过读写 byte 数组来实现的。



#### 包装类型

基本类型都有对应的包装类型，基本类型与其对应的包装类型之间的赋值使用自动装箱与拆箱完成。(包装类相当于基本类型的缓存)

```java
Integer x = 2;     // 装箱 调用了 Integer.valueOf(2)
int y = x;         // 拆箱 调用了 X.intValue()
```

![image-20200828100722898](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200828100722898.png)







#### 缓存池

new Integer(123) 与 Integer.valueOf(123) 的区别在于：

- new Integer(123) 每次都会新建一个对象；
- Integer.valueOf(123) 会使用缓存池中的对象，多次调用会取得同一个对象的引用。
- ![image-20200820222714502](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200820222714502.png)

* ![image-20200811181120345](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200811181120345.png)

* > JVM中一个字节以下的整型数据（即[-128,127]）会在JVM启动时加载进内存，所以除非用new Integer()显示的创建对象，否则都是同一对象.



### String(引用类型)

String 被声明为 final，因此它不可被继承。(Integer 等包装类也不能被继承）



#### StringPool

![image-20200820224103405](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200820224103405.png)

* 如果一个 String 对象已经被创建过了，那么就会从 String Pool 中取得引用。只有 String 是不可变的，才可能使用 String Pool。

![image-20200820222953658](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200820222953658.png)



#### StringBuffer StringBuilder

**1. 可变性**

- String 不可变
- StringBuffer 和 StringBuilder 可变

**2. 线程安全**

- String 不可变，因此是线程安全的
- StringBuilder 不是线程安全的
- StringBuffer 是线程安全的，内部使用 synchronized 进行同步





### 关键字



#### Final



**1. 数据**

声明数据为常量，可以是编译时常量，也可以是在运行时被初始化后不能被改变的常量。

- 对于基本类型，final 使数值不变；
- 对于引用类型，final 使引用不变，也就不能引用其它对象，但是被引用的对象本身是可以修改的。

```java
final int x = 1;
// x = 2;  // cannot assign value to final variable 'x'
final A y = new A();
y.a = 1;
```

**2. 方法**

声明方法不能被子类重写。

private 方法隐式地被指定为 final，如果在子类中定义的方法和基类中的一个 private 方法签名相同，此时子类的方法不是重写基类方法，而是在子类中定义了一个新的方法。

**3. 类**

声明类不允许被继承。





#### Static



**1. 静态变量**

- 静态变量：又称为类变量，也就是说这个变量属于类的，类所有的实例都共享静态变量，可以直接通过类名来访问它。静态变量在内存中只存在一份。
- 实例变量：每创建一个实例就会产生一个实例变量，它与该实例同生共死。



**2. 静态方法**

静态方法在类加载的时候就存在了，它不依赖于任何实例。所以静态方法必须有实现，也就是说它不能是抽象方法。



**3. 静态语句块**

静态语句块在类初始化时运行一次。 



**4. 静态内部类**

非静态内部类依赖于外部类的实例，也就是说需要先创建外部类实例，才能用这个实例去创建非静态内部类。而静态内部类不需要。

静态内部类不能访问外部类的非静态的变量和方法。

![image-20200820224711474](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200820224711474.png)





### Object常用方法



```java
public native int hashCode()

public boolean equals(Object obj)

protected native Object clone() throws CloneNotSupportedException

public String toString()

public final native Class<?> getClass()

protected void finalize() throws Throwable {}

public final native void notify()

public final native void notifyAll()

public final native void wait(long timeout) throws InterruptedException

public final void wait(long timeout, int nanos) throws InterruptedException

public final void wait() throws InterruptedException
```

* equal():

![image-20200820225846600](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200820225846600.png)





Clone()

* 浅拷贝: 拷贝的对象是同一引用
* 深拷贝: 不同引用



### 接口与抽象类

![image-20200820231226200](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200820231226200.png)



- 从设计层面上看，抽象类提供了一种 IS-A 关系，需要满足里式替换原则，即子类对象必须能够替换掉所有父类对象。而接口更像是一种 LIKE-A 关系，它只是提供一种方法实现契约，并不要求接口和实现接口的类具有 IS-A 关系。
- 从使用上来看，一个类可以实现多个接口，但是不能继承多个抽象类。
- 接口的字段只能是 static 和 final 类型的，而抽象类的字段没有这种限制。
- 接口的成员只能是 public 的，而抽象类的成员可以有多种访问权限。





### *反射*

反射机制是在运行时，对于任意一个类，都能够知道这个类的所有属性和方法;对于任意个对象，都能 够调用它的任意一个方法。在java中，只要给定类的名字，就可以通过反射机制来获得类的所有信息。

* 这种动态获取的信息以及动态调用对象的方法的功能称为反射机制
* 反射的核心是 JVM 在运行时才动态加载类或调用方法/访问属性，它不需要事先（写代码的时候或编译期）知道运行对象是谁。

![image-20200821102113144](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200821102113144.png)

![image-20200821102154776](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200821102154776.png)

反射的实现:

![image-20200820235351122](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200820235351122.png)

- **Field** ：可以使用 get() 和 set() 方法读取和修改 Field 对象关联的字段；
- **Method** ：可以使用 invoke() 方法调用与 Method 对象关联的方法；
- **Constructor** ：可以用 Constructor 的 newInstance() 创建新的对象。



*动态代理*

![image-20200821100253857](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200821100253857.png)





动态代理类是JVM运行时根据字节码文件动态生成的

![image-20200821102900523](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200821102900523.png)





#### 序列化

 序列化就是将一个对象转换成字节序列，方便存储和传输。

- 序列化：ObjectOutputStream.writeObject()
- 反序列化：ObjectInputStream.readObject()

不会对静态变量进行序列化，因为序列化只是保存对象的状态，静态变量属于类的状态。





* 序列化的类需要实现 Serializable 接口，它只是一个标准，没有任何方法需要实现，但是如果不去实现它的话而进行序列化，会抛出异常。





transient 关键字可以使一些属性不会被序列化。

ArrayList 中存储数据的数组 elementData 是用 transient 修饰的，因为这个数组是动态扩展的，并不是所有的空间都被使用，因此就不需要所有的内容都被序列化。通过重写序列化和反序列化方法，使得可以只序列化数组中有内容的那部分数据。

```java
private transient Object[] elementData;
```



### NIO



![image-20200821104411260](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200821104411260.png)

![image-20200821104615692](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200821104615692.png)





## 容器

容器主要包括 `Collection` 和` Map` 两种，Collection 存储着对象的集合，而 Map 存储着键值对（两个对象）的映射表。



* Collection

![image-20200821104703125](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200821104703125.png)



* Map

![image-20200821104805067](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200821104805067.png)



#### ArrayList

![image-20200821105056451](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200821105056451.png)

![image-20200821105117209](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200821105117209.png)



* Vector的实现与ArrayList相似,但是使用了synchronize.



#### CopyOnWriteArrayList

* 读写分离

写操作在一个复制的数组上进行，读操作还是在原始数组中进行，读写分离，互不影响。

写操作需要加锁，防止并发写入时导致写入数据丢失。

写操作结束之后需要把原始数组指向新的复制数组。



CopyOnWriteArrayList 在写操作的同时允许读操作，大大提高了读操作的性能，因此很适合读多写少的应用场景。

![image-20200911165421238](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200911165421238.png)



#### HashMap

![image-20200821105917227](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200821105917227.png)







* 插入时使用头插法(倒序)

![image-20200831154118850](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200831154118850.png)





##### 为什么HashMap线程不安全



* PUT插入数据时 数据不一致(覆盖)

![image-20200831150210178](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200831150210178.png)

* 扩容时死循环

当两个线程都扩容时, 会出现环形链表, CPU100%



![image-20200831155245319](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200831155245319.png)



![image-20200831155636743](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200831155636743.png)





* 在JDK1.8中, 使用了尾插法, 不会出现环型链表.会出现数据覆盖



#### HashTable



* HashTable使用Synchronize进行
* 默认长度为11 扩容: 11*2 + 1, 因为分散性更高



HashTable 直接使用 hashCode(Key)



而 HashMap:使用hash再一次分散 (前十位异或后十二位)

hash( key.hasCode() )







#### ConcurrentHashMap



* 1.7 

ConcurrentHashMap 和 HashMap 实现上类似，最主要的差别是 ConcurrentHashMap 采用了分段锁（Segment），每个分段锁维护着几个桶（HashEntry），多个线程可以同时访问不同分段锁上的桶，从而使其并发度更高（并发度就是 Segment 的个数）。

(默认16个segment, 即16并发数)

![image-20200821111606289](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200821111606289.png)





ConcurrentHashMap 在默认并发级别会创建包含 16 个 Segment 对象的数组。每个 Segment 的成员对象 table 包含若干个散列表的桶。每个桶是由 HashEntry 链接起来的一个链表。如果键能均匀散列，每个 Segment 大约守护整个散列表中桶总数的 1/16。

![image-20200824140746639](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200824140746639.png)

![image-20200824144248901](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200824144248901.png)



(当删除时会建立一条新的链表(倒序), 不会干扰别的进程读)



* 1.8
* JDK 1.8 使用了 CAS 操作来支持更高的并发度，在 CAS 操作失败时使用内置锁 synchronized。(颗粒度变为Node)

![image-20200824145213444](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200824145213444.png)



![image-20200824145244761](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200824145244761.png)





## 并发



### 基础线程机制

* Executor

> Executor 管理多个异步任务的执行，而无需程序员显式地管理线程的生命周期。这里的异步是指多个任务的执行互不干扰，不需要进行同步操作。

![image-20200822155219858](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200822155219858.png)



* Daemon

> 守护线程是程序运行时在后台提供服务的线程，不属于程序中不可或缺的部分。
>
> 当所有非守护线程结束时，程序也就终止，同时会杀死所有守护线程。
>
> main() 属于非守护线程。
>
> 在线程启动之前使用 setDaemon() 方法可以将一个线程设置为守护线程。

```java
public static void main(String[] args) {
    Thread thread = new Thread(new MyRunnable());
    thread.setDaemon(true);
}
```



* sleep()

Thread.sleep(millisec) 方法会休眠当前正在执行的线程，millisec 单位为毫秒。

sleep() 可能会抛出 InterruptedException，因为异常不能跨线程传播回 main() 中，因此必须在本地进行处理。线程中抛出的其它异常也同样需要在本地进行处理。



* yield()

对静态方法 Thread.yield() 的调用声明了当前线程已经完成了生命周期中最重要的部分，可以切换给其它线程来执行。该方法只是对线程调度器的一个建议，而且也只是建议具有相同优先级的其它线程可以运行。





![image-20200822155512164](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200822155512164.png)





### 线程池



#### Executors构造



Java中创建线程池很简单，只需要调用`Executors`中相应的便捷方法即可，比如`Executors.newFixedThreadPool(int nThreads)`，但是便捷不仅隐藏了复杂性，也为我们埋下了潜在的隐患（OOM，线程耗尽）。

`Executors`创建线程池便捷方法列表：

| 方法名                           | 功能                                                       |
| -------------------------------- | ---------------------------------------------------------- |
| newFixedThreadPool(int nThreads) | 创建固定大小的线程池                                       |
| newSingleThreadExecutor()        | 创建只有一个线程的线程池                                   |
| newCachedThreadPool()            | 创建一个不限线程数上限的线程池，任何提交的任务都将立即执行 |



#### ThreadPoolExecutor构造

![image-20200901091651047](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200901091651047.png)



![image-20200901103223869](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200901103223869.png)





#### 四种拒绝策略

![image-20200901094608378](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200901094608378.png)









---

### 互斥同步

#### 	synchronized



* 只作用于同一个对象,如果调用两个对象上的同步代码块,则不起作用.

同步一个方法:

```java
public synchronized void func () {
    // ...
}
```

它只作用于同一个对象，如果调用两个对象上的同步代码块，就不会进行同步。

对于以下代码，使用 ExecutorService 执行了两个线程，由于调用的是同一个对象的同步代码块，因此这两个线程会进行同步，当一个线程进入同步语句块时，另一个线程就必须等待。

![image-20200822160559053](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200822160559053.png)

![image-20200822160920850](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200822160920850.png)



同步一个类 ( 同步一个静态方法也是作用于整个类)

![image-20200822161120210](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200822161120210.png)



*JDK1.7 1.8 区别*:





#### RentrantLock

ReentrantLock 是 java.util.concurrent（J.U.C）包中的锁。

![image-20200822162026639](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200822162026639.png)



`比较`

> **1. 锁的实现**
>
> synchronized 是 JVM 实现的，而 ReentrantLock 是 JDK 实现的。
>
> **2. 性能**
>
> 新版本 Java 对 synchronized 进行了很多优化，例如自旋锁等，synchronized 与 ReentrantLock 大致相同。
>
> **3. 等待可中断**
>
> 当持有锁的线程长期不释放锁的时候，正在等待的线程可以选择放弃等待，改为处理其他事情。
>
> ReentrantLock 可中断，而 synchronized 不行。
>
> **4. 公平锁**
>
> 公平锁是指多个线程在等待同一个锁时，必须按照申请锁的时间顺序来依次获得锁。
>
> synchronized 中的锁是非公平的，ReentrantLock 默认情况下也是非公平的，但是也可以是公平的。
>
> **5. 锁绑定多个条件**
>
> 一个 ReentrantLock 可以同时绑定多个 Condition 对象。

除非需要使用 ReentrantLock 的高级功能，否则优先使用 synchronized。这是因为 synchronized 是 JVM 实现的一种锁机制，JVM 原生地支持它，而 ReentrantLock 不是所有的 JDK 版本都支持。并且使用 synchronized 不用担心没有释放锁而导致死锁问题，因为 JVM 会确保锁的释放。





### 非阻塞同步(CAS)

![image-20200822171725583](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200822171725583.png)



#### 1. CAS

乐观锁需要操作和冲突检测这两个步骤具备原子性，这里就不能再使用互斥同步来保证了，只能靠硬件来完成。硬件支持的原子性操作最典型的是：比较并交换（Compare-and-Swap，CAS）。CAS 指令需要有 3 个操作数，分别是内存地址 V、旧的预期值 A 和新值 B。当执行操作时，只有当 V 的值等于 A，才将 V 的值更新为 B。



* java中

![image-20200824101810547](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200824101810547.png)



![image-20200824101907397](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200824101907397.png)





* 不足: ABA问题, 自选CPU占用大, 只能对一个原子共享变量生效.



#### 2. AtomicInteger

J.U.C 包里面的整数原子类 AtomicInteger 的方法调用了 Unsafe 类的 CAS 操作。



### 无同步方案



#### 线程本地存储（Thread Local Storage）

![image-20200822172143095](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200822172143095.png)



![image-20200822172255467](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200822172255467.png)





为了理解 ThreadLocal，先看以下代码：

```java
public class ThreadLocalExample1 {
    public static void main(String[] args) {
        ThreadLocal threadLocal1 = new ThreadLocal();
        ThreadLocal threadLocal2 = new ThreadLocal();
        Thread thread1 = new Thread(() -> {
            threadLocal1.set(1);
            threadLocal2.set(1);
        });
        Thread thread2 = new Thread(() -> {
            threadLocal1.set(2);
            threadLocal2.set(2);
        });
        thread1.start();
        thread2.start();
    }
}
```

它所对应的底层结构图为：

![image-20200822173256305](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200822173256305.png)



每个 Thread 都有一个 ThreadLocal.ThreadLocalMap 对象。

```java
/* ThreadLocal values pertaining to this thread. This map is maintained
 * by the ThreadLocal class. */
ThreadLocal.ThreadLocalMap threadLocals = null;
```

当调用一个 ThreadLocal 的 set(T value) 方法时，先得到当前线程的 ThreadLocalMap 对象，然后将 ThreadLocal->value 键值对插入到该 Map 中。

```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```

get() 方法类似。

```java
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
```

ThreadLocal 从理论上讲并不是用来解决多线程并发问题的，因为根本不存在多线程竞争。

在一些场景 (尤其是使用线程池) 下，由于 ThreadLocal.ThreadLocalMap 的底层数据结构导致 ThreadLocal 有内存泄漏的情况，应该尽可能在每次使用 ThreadLocal 后手动调用 remove()，以避免出现 ThreadLocal 经典的内存泄漏甚至是造成自身业务混乱的风险。





* 主要用途是数据隔离.

![image-20200824114612152](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200824114612152.png)



### 线程协作



* join()

在线程中调用另一个线程的 join() 方法，会将当前线程挂起，而不是忙等待，直到目标线程结束。

对于以下代码，虽然 b 线程先启动，但是因为在 b 线程中调用了 a 线程的 join() 方法，b 线程会等待 a 线程结束才继续执行，因此最后能够保证 a 线程的输出先于 b 线程的输出。

<img src="/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200822162516085.png" alt="image-20200822162516085" style="zoom:50%;" />



* wait() notify() notifyAll()



调用 wait() 使得线程等待某个条件满足，线程在等待时会被挂起，当其他线程的运行使得这个条件满足时，其它线程会调用 notify() 或者 notifyAll() 来唤醒挂起的线程。

它们都属于 Object 的一部分，而不属于 Thread。

只能用在同步方法或者同步控制块中使用，否则会在运行时抛出 IllegalMonitorStateException。

使用 wait() 挂起期间，线程会释放锁。这是因为，如果没有释放锁，那么其它线程就无法进入对象的同步方法或者同步控制块中，那么就无法执行 notify() 或者 notifyAll() 来唤醒挂起的线程，造成死锁。

![image-20200822163106609](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200822163106609.png)



**wait() 和 sleep() 的区别**

> wait() 是 Object 的方法，而 sleep() 是 Thread 的静态方法；
>
> wait() 会释放锁，sleep() 不会。





* await() signal() signalAll()

java.util.concurrent 类库中提供了 Condition 类来实现线程之间的协调，可以在 Condition 上调用 await() 方法使线程等待，其它线程调用 signal() 或 signalAll() 方法唤醒等待的线程。

相比于 wait() 这种等待方式，await() 可以指定等待的条件，因此更加灵活。

使用 Lock 来获取一个 Condition 对象。

![image-20200822163856945](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200822163856945.png)

wait() 和 notify() 是 Object的方法, 只能用在同步方法或者同步控制块中使用，否则会在运行时抛出 IllegalMonitorStateException。

而await() 和 signal() 是JUC的方法.



### 线程状态



#### 新建（NEW）

创建后尚未启动。



#### 可运行（RUNABLE）

正在 Java 虚拟机中运行。但是在操作系统层面，它可能处于运行状态，也可能等待资源调度（例如处理器资源），资源调度完成就进入运行状态。所以该状态的可运行是指可以被运行，具体有没有运行要看底层操作系统的资源调度。



#### 阻塞（BLOCKED）

请求获取 monitor lock 从而进入 synchronized 函数或者代码块，但是其它线程已经占用了该 monitor lock，所以出于阻塞状态。要结束该状态进入从而 RUNABLE 需要其他线程释放 monitor lock。



#### 无限期等待（WAITING）



等待其它线程显式地唤醒。

阻塞和等待的区别在于，阻塞是被动的，它是在等待获取 monitor lock。而等待是主动的，通过调用 Object.wait() 等方法进入。

| 进入方法                                   | 退出方法                             |
| ------------------------------------------ | ------------------------------------ |
| 没有设置 Timeout 参数的 Object.wait() 方法 | Object.notify() / Object.notifyAll() |
| 没有设置 Timeout 参数的 Thread.join() 方法 | 被调用的线程执行完毕                 |
| LockSupport.park() 方法                    | LockSupport.unpark(Thread)           |



#### 限期等待（TIMED_WAITING）

无需等待其它线程显式地唤醒，在一定时间之后会被系统自动唤醒。

| 进入方法                                 | 退出方法                                        |
| ---------------------------------------- | ----------------------------------------------- |
| Thread.sleep() 方法                      | 时间结束                                        |
| 设置了 Timeout 参数的 Object.wait() 方法 | 时间结束 / Object.notify() / Object.notifyAll() |
| 设置了 Timeout 参数的 Thread.join() 方法 | 时间结束 / 被调用的线程执行完毕                 |
| LockSupport.parkNanos() 方法             | LockSupport.unpark(Thread)                      |
| LockSupport.parkUntil() 方法             | LockSupport.unpark(Thread)                      |

调用 Thread.sleep() 方法使线程进入限期等待状态时，常常用“使一个线程睡眠”进行描述。调用 Object.wait() 方法使线程进入限期等待或者无限期等待时，常常用“挂起一个线程”进行描述。睡眠和挂起是用来描述行为，而阻塞和等待用来描述状态。



#### 死亡（TERMINATED）



可以是线程结束任务之后自己结束，或者产生了异常而结束。





### *AQS*(重点)



**AQS原理**
AQS：AbstractQuenedSynchronizer抽象的队列式同步器。是除了java自带的synchronized关键字之外的锁机制。
AQS的全称为（AbstractQueuedSynchronizer），这个类在java.util.concurrent.locks包



![image-20200824105159830](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200824105159830.png)

![image-20200824104543189](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200824104543189.png)





![image-20200824105522949](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200824105522949.png)



![image-20200824110554272](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200824110554272.png)

ReentrantLock为例，（可重入独占式锁）：state初始化为0，表示未锁定状态，A线程lock()时，会调用tryAcquire()独占锁并将state+1.之后其他线程再想tryAcquire的时候就会失败，直到A线程unlock（）到state=0为止，其他线程才有机会获取该锁。A释放锁之前，自己也是可以重复获取此锁（state累加），这就是可重入的概念。
注意：获取多少次锁就要释放多少次锁，保证state是能回到零态的。

以CountDownLatch为例，任务分N个子线程去执行，state就初始化 为N，N个线程并行执行，每个线程执行完之后countDown（）一次，state就会CAS减一。当N子线程全部执行完毕，state=0，会unpark()主调用线程，主调用线程就会从await()函数返回，继续之后的动作。

一般来说，自定义同步器要么是独占方法，要么是共享方式，他们也只需实现tryAcquire-tryRelease、tryAcquireShared-tryReleaseShared中的一种即可。但AQS也支持自定义同步器同时**实现独占和共享两种方式，如ReentrantReadWriteLock。**
　在acquire() acquireShared()两种方式下，线程在等待队列中都是忽略中断的，**acquireInterruptibly()/acquireSharedInterruptibly()是支持响应中断**的。



![image-20200824112723607](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200824112723607.png)



#### CountDownLatch(share型方法,非独占)

![image-20200822165239917](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200822165239917.png)



#### CyclicBarrier

用来控制多个线程互相等待，只有当多个线程都到达时，这些线程才会继续执行。

和 CountdownLatch 相似，都是通过维护计数器来实现的。线程执行 await() 方法之后计数器会减 1，并进行等待，直到计数器为 0，所有调用 await() 方法而在等待的线程才能继续执行。

CyclicBarrier 和 CountdownLatch 的一个区别是，CyclicBarrier 的计数器通过调用 reset() 方法可以循环使用，所以它才叫做循环屏障。

CyclicBarrier 有两个构造函数，其中 parties 指示计数器的初始值，barrierAction 在所有线程都到达屏障的时候会执行一次。



### java内存模型

![image-20200822170434251](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200822170434251.png)

> 
>
> 所有的变量都存储在主内存中，每个线程还有自己的工作内存，工作内存存储在高速缓存或者寄存器中，保存了该线程使用的变量的`主内存副本拷贝`。
>
> 线程只能直接操作工作内存中的变量，不同线程之间的变量值传递需要通过主内存来完成 



![image-20200822170542843](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200822170542843.png)





![image-20200822170558601](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200822170558601.png)

- read：把一个变量的值从主内存传输到工作内存中
- load：在 read 之后执行，把 read 得到的值放入工作内存的变量副本中
- use：把工作内存中一个变量的值传递给执行引擎
- assign：把一个从执行引擎接收到的值赋给工作内存的变量
- store：把工作内存的一个变量的值传送到主内存中
- write：在 store 之后执行，把 store 得到的值放入主内存的变量中
- lock：作用于主内存的变量
- unlock



*可见性*

可见性指当一个线程修改了共享变量的值，其它线程能够立即得知这个修改。Java 内存模型是通过在变量修改后将新值同步回主内存，在变量读取前从主内存刷新变量值来实现可见性的。



主要有三种实现可见性的方式：

- volatile
- synchronized，对一个变量执行 unlock 操作之前，必须把变量值同步回主内存。
- final，被 final 关键字修饰的字段在构造器中一旦初始化完成，并且没有发生 this 逃逸（其它线程通过 this 引用访问到初始化了一半的对象），那么其它线程就能看见 final 字段的值。

对前面的线程不安全示例中的 cnt 变量使用 volatile 修饰，不能解决线程不安全问题，因为 volatile 并不能保证操作的原子性。





#### Volatile

![image-20200828095219798](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200828095219798.png)

- volatile写的内存语义：当写一个volatile变量时，JMM会把该线程对应的本地内存中的共享变量值刷新到主内存
- volatile读的内存语义：当读一个volatile变量时，JMM会把该线程对应的本地内存置为无效。线程接下来将从主内存中读取共享变量。

![image-20200828100228720](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200828100228720.png)



* Volatile 保证了共享变量的可见性



为什么`volatile`能够保证变量在线程中的可见性？因为JVM就是通过`volatile`调动了缓存一致性机制，如果对使用了`volatile`的程序，查看JVM解释执行或者JIT编译后生成的汇编代码，你会发现对`volatile`域（被`volatile`修饰的共享变量）的写操作生成的汇编指令会有一个`lock`前缀，该`lock`前缀表示JVM会向CPU发送一个信号，这个信号有两个作用：

- 对该变量的改写立即刷新到主存（也就是说对`volatile`域的写会导致`assgin -> store -> write`的原子性执行）
- 通过总线通知其他CPU该共享变量已被更新，对于也缓存了该共享变量的CPU，如果接收到该通知，那么会在自己的Cache中将共享变量所在的缓存行置为无效状态。CPU在下次读取读取该共享变量时发现缓存行已被置为无效状态，他将重新到主存中读取。





### 锁优化(synchronize)



![image-20200823144147644](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200823144147644.png)



* 重量级锁

![image-20200823144207693](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200823144207693.png)



* 自旋锁

**通过自旋锁，可以减少线程阻塞造成的线程切换**（包括挂起线程和恢复线程）。



互斥同步进入阻塞状态的开销都很大，应该尽量避免。在许多应用中，共享数据的锁定状态只会持续很短的一段时间。自旋锁的思想是让一个线程在请求一个共享数据的锁时执行忙循环（自旋）一段时间，如果在这段时间内能获得锁，就可以避免进入阻塞状态。

自旋锁虽然能避免进入阻塞状态从而减少开销，但是它需要进行忙循环操作占用 CPU 时间，它只适用于共享数据的锁定状态很短的场景。

在 JDK 1.6 中引入了自适应的自旋锁。自适应意味着自旋的次数不再固定了，而是由前一次在同一个锁上的自旋次数及锁的拥有者的状态来决定。



* 锁消除
* 锁粗化

![image-20200822174659637](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200822174659637.png)

* 轻量级锁



解决无竞争还要争夺锁(重量级) 的问题

![image-20200823144848186](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200823144848186.png)

![image-20200822175309239](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200822175309239.png)

* 偏向锁

![image-20200822175526792](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200822175526792.png)

## JVM



#### i++ 与 ++i

![image-20200824102203181](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200824102203181.png)





![image-20200824102052518](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200824102052518.png)





### JVM 结构



<img src="/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200824175604229.png" alt="image-20200824175604229" style="zoom: 50%;" />

* 程序计数器

记录正在执行的虚拟机字节码指令的地址（如果正在执行的是本地方法则为空）。



* Java虚拟机栈



每个 Java 方法在执行的同时会创建一个栈帧用于存储局部变量表、操作数栈、常量池引用等信息。从方法调用直至执行完成的过程，对应着一个栈帧在 Java 虚拟机栈中入栈和出栈的过程。

![image-20200824175804617](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200824175804617.png)



* 本地方法栈

![image-20200824175850249](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200824175850249.png)



* 堆

所有对象都在这里分配内存，是垃圾收集的主要区域（"GC 堆"）。

现代的垃圾收集器基本都是采用分代收集算法，其主要的思想是针对不同类型的对象采取不同的垃圾回收算法。可以将堆分成两块：

- 新生代（Young Generation）
- 老年代（Old Generation）

堆不需要连续内存，并且可以动态增加其内存，增加失败会抛出 OutOfMemoryError 异常



* 方法区

![image-20200824180045747](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200824180045747.png)



* 运行时常量池

运行时常量池是方法区的一部分。

Class 文件中的常量池（编译器生成的字面量和符号引用）会在类加载后被放入这个区域。

除了在编译期生成的常量，还允许动态生成，例如 String 类的 intern()。





#### JDK1.8

![image-20200825102016776](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200825102016776.png)



**在永久代移除后，字符串常量池也不再放在永久代了，但是也没有放到新的方法区---元空间里，而是留在了堆里（为了方便回收？）。运行时常量池当然是随着搬家到了元空间里，毕竟它是装类的重要信息的，有它的地方才称得上是方法区。**



![image-20200825102416330](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200825102416330.png)



class常量池
字面量 包括：1.文本字符串 2.八种基本类型的值 3.被声明为final的常量等;



方法区是一个 JVM 规范，永久代与元空间都是其一种实现方式。在 JDK 1.8 之后，原来永久代的数据被分到了堆和元空间中。元空间存储类的元信息，静态变量和常量池等放入堆中







### 对象



#### 对象创建

![image-20200923095324534](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200923095324534.png)



#### 为什么单例模式的DoubleCheckedLocking中要使用Violate



​	因为创建对象的过程很复杂, 很有可能在重排序之后导致对象的引用被返回了而对象的初始化并未完成, 会导致难以理解的空指针异常.

 

#### 对象内存布局

![image-20200923095906461](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200923095906461.png)



#### 对象死亡回收

![image-20200923145427349](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200923145427349.png)





---

### 垃圾收集

垃圾收集主要是针对堆和方法区进行。程序计数器、虚拟机栈和本地方法栈这三个区域属于线程私有的，只存在于线程的生命周期内，线程结束之后就会消失，因此不需要对这三个区域进行垃圾回收。



#### 判断对象是否可被回收



##### 引用计数



为对象添加一个引用计数器，当对象增加一个引用时计数器加 1，引用失效时计数器减 1。引用计数为 0 的对象可被回收。

在两个对象出现循环引用的情况下，此时引用计数器永远不为 0，导致无法对它们进行回收。正是因为循环引用的存在，因此 Java 虚拟机不使用引用计数算法。



##### 可达性分析



以 GC Roots 为起始点进行搜索，可达的对象都是存活的，不可达的对象可被回收。

Java 虚拟机使用该算法来判断对象是否可被回收，GC Roots 一般包含以下内容：



- 虚拟机栈中局部变量表中引用的对象
- 本地方法栈中 JNI 中引用的对象
- 方法区中类静态属性引用的对象
- 方法区中的常量引用的对象



![image-20200824180738138](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200824180738138.png)



##### 方法区回收

因为方法区主要存放永久代对象，而永久代对象的回收率比新生代低很多，所以在方法区上进行回收性价比不高。

主要是对常量池的回收和对类的卸载。

为了避免内存溢出，在大量使用反射和动态代理的场景都需要虚拟机具备类卸载功能。

类的卸载条件很多，需要满足以下三个条件，并且满足了条件也不一定会被卸载：

- 该类所有的实例都已经被回收，此时堆中不存在该类的任何实例。
- 加载该类的 ClassLoader 已经被回收。
- 该类对应的 Class 对象没有在任何地方被引用，也就无法在任何地方通过反射访问该类方法







### 引用类型

Java 提供了四种强度不同的引用类型。



* 强引用

被强引用关联的对象不会被回收。

使用 new 一个新对象的方式来创建强引用。

* 软引用

被软引用关联的对象只有在内存不够的情况下才会被回收。

使用 SoftReference 类来创建软引用。

* 弱引用

被弱引用关联的对象一定会被回收，也就是说它只能存活到下一次垃圾回收发生之前。

使用 WeakReference 类来创建弱引用。

* 虚引用

又称为幽灵引用或者幻影引用，一个对象是否有虚引用的存在，不会对其生存时间造成影响，也无法通过虚引用得到一个对象。

为一个对象设置虚引用的唯一目的是能在这个对象被回收时收到一个系统通知。

使用 PhantomReference 来创建虚引用。





### 垃圾收集算法(GC算法)



#### 标记-清除



![image-20200825103557334](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200825103557334.png)





#### 标记-整理

![image-20200825103632169](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200825103632169.png)



#### 复制

![image-20200825103710396](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200825103710396.png)



#### 分代收集



现在的商业虚拟机采用分代收集算法，它根据对象存活周期将内存划分为几块，不同块采用适当的收集算法。

一般将堆分为新生代和老年代。

- 新生代使用：复制算法 

- 老年代使用：标记 - 清除 或者 标记 - 整理 算法



Minor GC

因为新生代对象存活时间很短，因此 Minor GC 会频繁执行，执行的速度一般也会比较快。

![image-20200825111618783](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200825111618783.png)



FullGC

回收老年代和新生代，老年代对象其存活时间长，因此 Full GC 很少执行，执行速度会比 Minor GC 慢很多。

分代收集(generation collection)

![image-20200825111756191](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200825111756191.png)





##### 内存分配策略

![image-20200827162554769](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200827162554769.png)



![image-20200827163040285](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200827163040285.png)



15岁进入老年代

![image-20200825112106156](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200825112106156.png)



![image-20200923162007531](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200923162007531.png)

##### Full GC 触发条件

![image-20200825112200407](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200825112200407.png)





#### 收集器



* 吞吐量:  CPU 用于运行用户程序的时间占总时间的比值



![image-20200825112956120](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200825112956120.png)

- 单线程与多线程：单线程指的是垃圾收集器只使用一个线程，而多线程使用多个线程；
- 串行与并行：串行指的是垃圾收集器与用户程序交替执行，这意味着在执行垃圾收集的时候需要停顿用户程序；并行指的是垃圾收集器和用户程序同时执行。除了 CMS 和 G1 之外，其它垃圾收集器都是以串行的方式执行



##### 1. Serial 收集器

![image-20200825113313599](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200825113313599.png)

Serial 翻译为串行，也就是说它以串行的方式执行。

它是单线程的收集器，只会使用一个线程进行垃圾收集工作。

它的优点是简单高效，在单个 CPU 环境下，由于没有线程交互的开销，因此拥有最高的单线程收集效率。

它是 `Client` 场景下的默认新生代收集器，因为在该场景下内存一般来说不会很大。它收集一两百兆垃圾的停顿时间可以控制在一百多毫秒以内，只要不是太频繁，这点停顿时间是可以接受的。





##### 2.ParNew收集器

![image-20200825113343165](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200825113343165.png)



它是 Serial 收集器的多线程版本。

它是 `Server` 场景下默认的新生代收集器，除了性能原因外，主要是因为除了 Serial 收集器，只有它能与 CMS 收集器配合使用。



##### 3. Parallel Scavenge

![image-20200825113621429](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200825113621429.png)



##### 4. CMS收集器

是一种以**获取最短回收停顿时间**为目标的收集器，它非常符合那些集中在互联网站或者B/S系统的服务端上的Java应用，这些应用都非常重视服务的响应速度。



![image-20200825113753976](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200825113753976.png)



- **初始标记（CMS initial mark）**：仅仅只是标记一下GC Roots能直接关联到的对象，速度很快，需要“Stop The World”。
- **并发标记（CMS concurrent mark）**：进行**GC Roots Tracing**的过程，在整个过程中耗时最长。
- **重新标记（CMS remark）**：为了修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，这个阶段的停顿时间一般会比初始标记阶段稍长一些，但远比并发标记的时间短。此阶段也需要“Stop The World”。
- **并发清除（CMS concurrent sweep）**



优点

CMS是一款优秀的收集器，它的主要**优点**在名字上已经体现出来了：**并发收集**、**低停顿**，因此CMS收集器也被称为**并发低停顿收集器（Concurrent Low Pause Collector）**。

缺点

![image-20200825114041062](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200825114041062.png)





##### 5.G1收集器

G1（Garbage-First），它是一款面向服务端应用的垃圾收集器，在多 CPU 和大内存的场景下有很好的性能。HotSpot 开发团队赋予它的使命是未来可以替换掉 CMS 收集器。

![image-20200825115310204](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200825115310204.png)





![image-20200825115414799](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200825115414799.png)



![image-20200825115539147](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200825115539147.png)







![image-20200825114757525](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200825114757525.png)





---

## 类加载机制



类是在运行期间第一次使用时动态加载的，而不是一次性加载所有类。因为如果一次性加载，那么会占用很多的内存。





### 类的生命周期





![image-20200825104239996](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200825104239996.png)





### 类加载过程

包含了加载、验证、准备、解析和初始化这 5 个阶段。

![image-20200825143340915](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200825143340915.png)

* 加载

![image-20200825104451499](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200825104451499.png)

* 验证

确保 Class 文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全。

* 准备

![image-20200825104635015](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200825104635015.png)

* 解析

将常量池的符号引用替换为直接引用的过程。

其中解析过程在某些情况下可以在初始化阶段之后再开始，这是为了支持 Java 的动态绑定。

* 初始化

初始化阶段才真正开始执行类中定义的 Java 程序代码。初始化阶段是虚拟机执行类构造器 <clinit>() 方法的过程。在准备阶段，类变量已经赋过一次系统要求的初始值，而在初始化阶段，根据程序员通过程序制定的主观计划去初始化类变量和其它资源。



*初始化时机*

![image-20200825105854030](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200825105854030.png)



![image-20200825105950831](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200825105950831.png)



### 双亲委派机制:

加载类时, 先由parent加载器解决, 不行再回传



* bootstrap加载器: java, javax, sun.xx lib 包的类加载
* extension加载器: ext 加载器
* application加载器: 用户diy类加载



优点:

减少类重复加载

防止攻击







## Spring



* 循环依赖是什么, Spring怎么解决(三级缓存)

![image-20200828091051841](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200828091051841.png)

![image-20200828091401020](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200828091401020.png)







* Spring设计模式



工厂: Bean工厂, applicationContext: 一次性加载, beanfactory : 延迟加载

代理: AOP

适配器: SpringMVC

单例: Bean(singleton)









## 面试





内存溢出OOM(outOfMemory) 排查

* 堆溢出
* 元空间溢出
* 栈溢出

*使用工具*: MAT, VisualVM

Eclipse MAT(Memory Analyzer Tool)



* 排查过程

![image-20200913095412846](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200913095412846.png)

![image-20200913095645905](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200913095645905.png)









IO流使用的设计模式

* ![image-20200826151218500](/Users/zhuhaoran/Library/Application Support/typora-user-images/image-20200826151218500.png)





Maven中dependencies与dependencyManagement的区别

* management管理全局依赖, 子模块只需要输入parent, 不需要版本号,统一管理
* 如果要不同版本号, 只需在对应的dependencies修改即可





如何从大量的两个文件中找出相同的xxx?



* 一般都是分治策略. 将文件A的url按hash(url)%xxx 分成 n个文件, B也同样操作. 接着在两文件对应的 a1,b1 里寻找.



索引失效的情况

* 条件中带有or
* like 以 % 开头
* 多列索引, 没有用第一个

* 某个表中，有两列（id和c_id）都建了单独索引，下面这种查询条件不会走索引

  select * from test where id=c_id;

  这种情况会被认为还不如走全表扫描

* 条件中包括函数

* is not, not null



