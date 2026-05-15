---
title: Java锁
status: 进行中
tags:
  - 并发
  - 多线程
---


[参考文章](https://blog.csdn.net/2301_78813969/article/details/136863584?ops_request_misc=elastic_search_misc&request_id=de1705ac4b7208f0e2ceebe00e455547&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-136863584-null-null.142^v102^pc_search_result_base4&utm_term=java%E9%94%81&spm=1018.2226.3001.4187)

# 对象头
Java 多线程的锁都是基于对象的，因为每个对象的对象头里有**Mark Word**，存着锁状态标志、持有线程ID、偏向时间戳等，是java语言的机制；对象能当锁，就是因为这块内存天生就被分配了“锁状态记录本”，所以可以说Java对象都可以被作为一个锁。常说的类锁其实也是对象锁，因为类本身也是一个Class对象。

# synchronized
synchronized 是 Java 内置的悲观锁机制

## 修饰位置
- 实例方法上添加：锁位于this,对象本身的对象头；多线程调用同一个对象实例的上锁方法，才会互斥。保护单个实例自己的状态
- 静态方法上添加：锁位于类对象xxx.class, 对象所属类的对象头中。调用这个类的静态方法，会互斥。保护类级别的静态资源d
- 修饰代码块：声明一个对象来分配内存空间，从而利用他的对象头 mark word 来记录锁的状态；锁位于这个new出来的对象，最小化锁范围，最推荐使用；可以精确指定锁对象，以实现对特定资源的同步访问。把不需要同步的耗时代码移出锁块，提升性能。
```java
// 修饰实例方法
public synchronized void synchronizedMethod() {
    // 同步代码
}
// 修饰静态方法
public static synchronized void synchronizedMethod() {
 // 同步代码
}

// 修饰代码块
// 单独声明一个锁对象
private final Object lock = new Object();

public void doSomething() {
    // 这里可以放无需同步的准备工作
    synchronized(lock) {
        counter++;  // 只锁这一小段关键区
    }
    // 后续耗时操作也可无锁
}
```

示例
```java
// 类
public class EntityLock {
    private int entityCounter = 0;

    public synchronized int add(){
        return ++entityCounter;
    }
}


public class Main {
    public static int counter = 0;
    public static final int INCREMENT = 10;

    public static synchronized int increment() {
        return ++counter;
    }

    private static void doSomeWork() {
        try {
            Thread.sleep(0, 100); // ??????
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }


    public static void main(String[] args) throws InterruptedException {

        System.out.println("/* ****************************?????**************************** */");
        Thread thread1 = new Thread(() -> {
            for(int i = 0; i < INCREMENT; i++){
                doSomeWork();
                int value = increment();    // 如果执行这个方法时，已经被上锁了，会在这里阻塞
                System.out.printf("Thread 1: %d\n", value);
            }
        });

        Thread thread2 = new Thread(() -> {
            for(int i = 0; i < INCREMENT; i++){
                doSomeWork();
                int value = increment();    // 如果执行这个方法时，已经被上锁了，会在这里阻塞
                System.out.printf("Thread 2: %d\n", value);
            }
        });
        thread1.start(); thread2.start();
        thread1.join(); thread2.join();
        System.out.printf("Expected: %d, Actual: %d\n",
                2 * INCREMENT, counter);


        System.out.println("/* ******两个锁实例，两个线程单独操作每个实例的共享资源，相互并不影响******** */");
        EntityLock entityLock1 = new EntityLock();
        EntityLock entityLock2 = new EntityLock();

        Thread thread3 = new Thread(() -> {
            for(int i = 0; i < INCREMENT; i++){
                int value = entityLock1.add();
                System.out.printf("Thread 3: %d\n", value);
            }
        });

        Thread thread4 = new Thread(() -> {
            for(int i = 0; i < INCREMENT + 2; i++){                // 为了区分这里 +2
                int value = entityLock2.add();
                System.out.printf("Thread 4: %d\n", value);
            }
        });

        thread3.start(); thread4.start();
        thread3.join(); thread4.join();
        System.out.printf("线程3 %d, 线程4：%d",  entityLock1.entityCounter, entityLock2.entityCounter);
    }
}
```


## to partition
> [!notes]- 基本机制
>**1. 对象头是基础**
>
>2. 轻量级抢锁用CAS- 无锁状态下，线程用**硬件级原子指令CAS**去改Mark Word，写“持有者=我”
>- 总线仲裁或缓存一致性协议保证：同时CAS只能有一个成功，另一个必失败
>3. 抢不到就升级
>- CAS失败 → 自旋重试几次 → 还是抢不到 → 锁膨胀为重量级
>- Mark Word改为指向Monitor对象（`ObjectMonitor`），里面用OS的Mutex挂起线程

**对象头Mark Word是锁状态的载体; CAS是争抢的原子操作; 重量级锁的Monitor是抢不到时的阻塞队列; 整个synchronized就是在Mark Word上玩CAS，玩不转就甩给OS。**


# ReentrantLock

**使用**
```java
Lock lock = new ReentrantLock();
lock.lock();
try {
    // 临界区代码
} finally {
    lock.unlock();  // 保证一定解锁
}
```

**常用方法**

| 方法                    | 作用                 | 特点                  |
| --------------------- | ------------------ | ------------------- |
| `lock()`              | 获取锁，拿不到就死等         | 类似 synchronized     |
| `unlock()`            | 释放锁                | **必须在 finally 里调用** |
| `tryLock()`           | 尝试获取，拿不到立即返回 false | 非阻塞                 |
| `tryLock(time, unit)` | 限时等待，超时返回 false    | 可中断，避免死等            |

**优势**
- 可以**尝试**获取锁（不阻塞） 
- 可以**限时等待**（避免死等）
- 可中断的锁获取
- 支持公平锁
- 但必须手动解锁，更容易出错

**简单记：synchronized 够用就用它，需要"尝试"或"超时"控制时才用 Lock。**

示例
```java
public class ReentrantLockExample {
    static int counter = 0;
    static final Lock lock = new ReentrantLock();
    // 相当于synchronized
    static void add(){
        lock.lock();
        try {
            counter++;
        }finally {
            lock.unlock();
        }
    }
	
	// 超时中断
    static void addWithTimeOut() throws InterruptedException {
        if(lock.tryLock(1, TimeUnit.MICROSECONDS)){
            try {
                counter++;
            }finally {
                lock.unlock();
            }
        } else {
            System.out.println("超过3us，不在等待");
        }
    }

    static void main(String[] args) throws InterruptedException {
		// 省略两个线程调用 addWithTimeOut()
        System.out.printf("执行总数:%d",counter);
        System.out.printf("超时次数:%d",2 * INCREMENT - counter);
    }
}
```





# 锁状态
锁的状态从低到高依次为无锁->偏向锁->轻量级锁->重量级锁，升级的过程就是从低到高，降级在一定条件也是有可能发生的。
#todo 