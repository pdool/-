《多线程系列三》我有

## 1、公平锁和非公平锁，乐观锁和悲观锁

​		公平锁：当线程A获取访问该对象，获取到锁后，此时内部存在一个计数器num+1，其他线程想访问该对象，就会进行排队等待(等待队列最前一个线程处于待唤醒状态)，直到线程A释放锁(num = 0)，此时会唤醒处于待唤醒状态的线程进行获取锁的操作，一直循环。如果线程A再次尝试获取该对象锁时，会检查该对象锁释放已经被占用，如果还是当前线程占用锁，则直接获得锁，不用进入排队。

　　非公平锁：当线程A在释放锁后，等待对象的线程会进行资源竞争，竞争成功的线程将获取该锁，其他线程继续睡眠。

　　公平锁是严格的以FIFO的方式进行锁的竞争，但是非公平锁是无序的锁竞争，刚释放锁的线程很大程度上能比较快的获取到锁，队列中的线程只能等待，所以非公平锁可能会有“饥饿”的问题。但是重复的锁获取能减小线程之间的切换，而公平锁则是严格的线程切换，这样对操作系统的影响是比较大的，所以非公平锁的吞吐量是大于公平锁的，这也是为什么JDK将非公平锁作为默认的实现。

​    悲观锁：总是假设最坏的情况，每次想要使用数据的时候就恰好别人也要修改数据，一切是以安全第一，所以在每次操作资源的时候都会先加锁，不管有没有人抢，然后独占资源。Java中`synchronized`和`ReentrantLock`等独占锁就是悲观锁思想的实现

​	乐观锁：乐观锁和悲观锁刚好相反，自己使用资源的时候没有人抢，所以不需要上锁。乐观锁的实现方案一般来说有两种： `版本号机制` 和 `CAS实现` 。乐观锁多适用于多度的应用类型，这样可以提高吞吐量。

在Java中`java.util.concurrent.atomic`包下面的原子变量类就是使用了乐观锁的一种实现方式CAS实现的。

## 2、synchronize 的使用方式

| 场景       | 具体分类 | 锁对象            | 代码示例                                          |
| ---------- | -------- | ----------------- | ------------------------------------------------- |
| 修饰方法   | 实例方法 | 当前实例对象      | public synchronized void method () {  ...  }      |
| ...        | 静态方法 | 当前类的Class对象 | public static synchronized void method () { ... } |
| 修饰代码块 | 代码块   | `( )`中配置的对象 | synchronized(object) { ... }                      |

## 3、synchronized 的实现原理

```
    private static Object lock = new Object();

    public static synchronized void testSyn() {
        System.out.println("香菜");
    }
    public synchronized void testSyn2() {
        System.out.println("香菜");
    }
    public static void testObj() {
        synchronized (lock) {
            System.out.println("香菜");
        }
    }
```

看下字节码：

![image-20210312225204760](D:\wechat\gameWathcer\img\20210312\3.png)

可以看到synchronized 的地方使用的是monitorenter指令，每个对象都和一个monitor对象关联，主要用来控制互斥资源的访问，如果你想要加锁必须先获得monitor的批准，如果现在正有线程访问，会把申请的线程加入到等待队列。

## 4、总结

1、 无论synchronized关键字加在方法上还是对象上，如果它作用的对象是非静态的，则它取得的锁是对象；如果synchronized作用的对象是一个静态方法或一个类，则它取得的锁是对class对象的锁，该类所有的对象同一把锁。 
2、每个对象只有一个锁（lock）与之相关联，谁拿到这个锁谁就可以运行它所控制的那段代码。 
3、实现同步是要很大的系统开销作为代价的，甚至可能造成死锁，所以尽量避免无谓的同步控制，避免做嵌套synchronized 的使用。

4、synchronized 要尽量控制范围，不能范围太大，否则会损失系统性能。



Lock



1、先看下类图

![image-20210314204927150](D:\wechat\gameWathcer\img\20210312\4.png)



https://www.cnblogs.com/jojop/p/14022029.html



https://www.cnblogs.com/minikobe/p/12123065.html