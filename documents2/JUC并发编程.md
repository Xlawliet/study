# 1什么是JUC

`java.util.concurent`包，分类

业务：普通线程Thread

**Runnable** 没有返回值，性能相较于`Callable`较低

# 2进程和线程

java 默认两个线程`main` 、`GC` 

**java不能真正开启线程**

## 并发与并行

并发才存在多线程操作同一资源

- 一个cpu核心快速交替执行多个线程。
- **本质：充分利用cpu资源**

并行（多个进程同时执行）

- 多个cpu核心，执行多个线程；线程池

```java
public static void main(String[] args) {
    //CPU密集型，IO密集型
    System.out.println(Runtime.getRuntime().availableProcessors());
}
```

## 线程状态

6个

```java
public enum State {
    //新生
    NEW,

    //运行
    RUNNABLE,

    //阻塞
    BLOCKED,

    //等待（一直等）
    WAITING,

    //等待（过期不候）
    TIMED_WAITING,

    //终止
    TERMINATED;
}
```

## **wait/sleep**

**wait**

- 自Object类；
- 释放锁
- 必须在同步代码块中使用
- 不需要捕获异常（除中断异常，中断异常所有线程都会有 ）

**sleep**

- 自Thread类
- 不释放锁
- 任何地方可使用
- 必须捕获异常

# Lock锁

> 传统方式`synchronized`

本质：队列、锁

```java
public class SaleTicket {
    public static void main(String[] args) {
        Ticket t = new Ticket();
        //并发，多个线程操作一个资源类
        //runnable为函数式接口，可以用匿名内部类
        //lambda 表达式(参数)->{代码}
        new Thread(() -> {
            for (int i = 1; i < 60; i++) {
                t.sale();
            }
        }, "A").start();

        new Thread(() -> {
            for (int i = 1; i < 60; i++) {
                t.sale();
            }
        }, "B").start();

        new Thread(() -> {
            for (int i = 1; i < 60; i++) {
                t.sale();
            }
        }, "C").start();
    }
}

/**
 * 资源类
 */
class Ticket {
    //票数
    private int number = 30;

    //卖票
    public synchronized void sale() {
        if (number > 0) {
            System.out.println(Thread.currentThread().getName() + "卖出了第" + number-- + "张票，剩余：" + number);
        }
    }
}
```

**锁**

公平锁：先来后到

**非公平锁：可以插队（默认）**

## 使用锁

> 1.`Lock lock=new ReentrantLock();`
>
> 2.`lock.lock();`
>
> 3.`try  业务代码... catch`
>
> 3.`finally lock.unlock();`

```java
public class SaleTicket2 {
    public static void main(String[] args) {
        Ticket2 t = new Ticket2();
        new Thread(() -> { for (int i = 1; i < 40; i++) t.sale(); }, "A").start();
        new Thread(() -> { for (int i = 1; i < 40; i++) t.sale(); }, "B").start();
        new Thread(() -> { for (int i = 1; i < 40; i++) t.sale(); }, "C").start();
    }
}

/**
 * 资源类
 */
class Ticket2 {
    //票数
    private int number = 30;
    Lock lock = new ReentrantLock();

    //卖票
    public void sale() {
        //加锁
        lock.lock();
        try {
            //业务代码

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //解锁
            lock.unlock();
        }
        if (number > 0) {
            System.out.println(Thread.currentThread().getName() + "卖出了第" + number-- + "张票，剩余：" + number);
        }
    }
}
```

> Synchronized和Lock区别

1.Synchronized是java关键字，Lock是一个类

2.Synchronized无法判断锁的状态，Lock可以判断是否获取到了锁

3.Synchronized会自动释放锁，Lock必须要手动释放锁，如果不释放锁则导致死锁

4.Synchronized：线程1（获得锁-》阻塞），线程2（等待，一直等）

​	Lock：不一定会等待下去

5.Synchronized可重入锁，不可以中断，非公平；

​	Lock：可重入锁，可以判断锁，可以设置公平非公平

6.Synchronized适合锁少量的代码同步问题，Lock适合锁大量同步代码

## 生产者消费者

> 生产者消费者模型Synchronized版本,Synchronized=>(wait,notify)

```java
/**
 * @author: 1298509345
 * date: 2020/5/1
 * Time: 14:05
 * Describe:线程交替执行，线程间通信问题
 * 等待唤醒，通知唤醒
 */
@SuppressWarnings("all")
public class A {
    public static void main(String[] args) {
        Data data = new Data();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.produce();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "A").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.consume();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "B").start();
    }
}

/**
 * 资源类
 * 判断是否等待-》业务-》通知
 */
class Data {
    private int number = 0;

    //+1
    public synchronized void produce() throws InterruptedException {
        if (number != 0) {
            //等待
            this.wait();
        }
        number++;
        System.out.println(Thread.currentThread().getName() + "->" + number);
        //通知其他线程，加一完毕
        this.notifyAll();
    }

    //-1
    public synchronized void consume() throws InterruptedException {
        if (number == 0) {
            //等待
            this.wait();
        }
        number--;
        System.out.println(Thread.currentThread().getName() + "->" + number);
        //通知其他线程，减一完毕
        this.notifyAll();
    }

}
```

**问题：线程多于两个时！虚假唤醒**

线程也可以唤醒，而不会被通知，中断或超时，即所谓的*虚假唤醒* `即一个线程执行完毕，唤醒了其他所有的线程，但只有部分线程有用，其他唤醒都是无用功`。  虽然这在实践中很少会发生，但应用程序必须通过测试应该使线程被唤醒的条件来防范，并且如果条件不满足则继续等待。  换句话说，等待应该总是出现在循环中，就像这样： 

```java
  synchronized (obj) {
         while (<condition does not hold>)
             obj.wait(timeout);
         ... // Perform action appropriate to condition
     } 
```

解决：`if判断`改为`while循环`

> 生产者消费者 Lock版本 Lock=>(await,signal)

  

```java
class BoundedBuffer {
   final Lock lock = new ReentrantLock();
   final Condition notFull  = lock.newCondition(); 
   final Condition notEmpty = lock.newCondition(); 

   final Object[] items = new Object[100];
   int putptr, takeptr, count;

   public void put(Object x) throws InterruptedException {
     lock.lock(); try {
       while (count == items.length)
         notFull.await();
       items[putptr] = x;
       if (++putptr == items.length) putptr = 0;
       ++count;
       notEmpty.signal();
     } finally { lock.unlock(); }
   }

   public Object take() throws InterruptedException {
     lock.lock(); try {
       while (count == 0)
         notEmpty.await();
       Object x = items[takeptr];
       if (++takeptr == items.length) takeptr = 0;
       --count;
       notFull.signal();
       return x;
     } finally { lock.unlock(); }
   }
 } 
```

代码

```java
/**
 * @author: 1298509345
 * date: 2020/5/1
 * Time: 14:05
 * Describe:线程交替执行，线程间通信问题
 * 等待唤醒，通知唤醒
 */
@SuppressWarnings("all")
public class B {
    public static void main(String[] args) {
        Data2 data = new Data2();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.produce();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "A").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.consume();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "B").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.produce();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "C").start();
        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    data.consume();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "D").start();
    }
}

/**
 * 资源类
 * 判断是否等待-》业务-》通知
 */
class Data2 {
    private int number = 0;
    Lock lock = new ReentrantLock();
    Condition condition = lock.newCondition();

    //+1
    public void produce() throws InterruptedException {
        lock.lock();
        try {
            while (number != 0) {
                //等待
                condition.await();
            }
            number++;
            System.out.println(Thread.currentThread().getName() + "->" + number);
            //通知其他线程，加一完毕
            condition.signalAll();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }

    }

    //-1
    public void consume() throws InterruptedException {
        lock.lock();
        try {
            while (number == 0) {
                //等待
                condition.await();
            }
            number--;
            System.out.println(Thread.currentThread().getName() + "->" + number);
            //通知其他线程，减一完毕
            condition.signalAll();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

}
```

> Condition 精准通知和唤醒线程

代码测试

```java
public class C {
    public static void main(String[] args) {
        Data3 data3 = new Data3();
        new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                data3.printA();
            }
        }, "A").start();

        new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                data3.printB();
            }
        }, "B").start();

        new Thread(() -> {
            for (int i = 0; i < 5; i++) {
                data3.printC();
            }
        }, "C").start();
    }
}


class Data3 {
    private Lock lock = new ReentrantLock();
    private Condition condition1 = lock.newCondition();
    private Condition condition2 = lock.newCondition();
    private Condition condition3 = lock.newCondition();
    private int number = 1;

    public void printA() {
        lock.lock();
        try {
            //业务代码
            while (number != 1) {
                //等待
                condition1.await();
            }
            System.out.println(Thread.currentThread().getName() + "在执行");
            //唤醒指定线程,B
            number = 2;
            condition2.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void printB() {
        lock.lock();
        try {
            //业务代码
            while (number != 2) {
                condition2.await();
            }
            number = 3;
            System.out.println(Thread.currentThread().getName() + "在执行");
            //唤醒指定线程,C
            condition3.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void printC() {
        lock.lock();
        try {
            //业务:判断，执行，通知
            while (number != 3) {
                condition3.await();
            }
            number = 1;
            System.out.println(Thread.currentThread().getName() + "在执行");
            //唤醒指定线程,C
            condition1.signal();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```

# 锁的8个问题

```java
package com.lock8;

import java.util.concurrent.TimeUnit;

/**
 * @author: 1298509345
 * date: 2020/5/1
 * Time: 15:54
 * Describe:锁的八个问题
 */
public class Test1 {
    /**
     * 一个资源对象，两个同步方法
     * Synchronized 锁的对象是方法的调用者
     * 由于两个线程的方法调用全是Phone对象，所以谁先拿到这个对象锁谁先执行
     * 即先发短信后打电话
     */
    public void test1() {
        Phone p = new Phone();

        new Thread(() -> {
            p.sendSms();
        }, "A").start();
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        new Thread(() -> {
            p.call();
        }, "B").start();
    }

    /**
     * 个资源对象，一个同步方法，一个延迟同步方法
     * 仍是先发短信，后打电话。因为锁的资源对象相同
     */
    public void test2() {

        Phone p = new Phone();
        new Thread(() -> {
            p.sendSmsDelay();
        }, "A").start();
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        new Thread(() -> {
            p.call();
        }, "B").start();
    }

    /**
     * 一个资源对象，一个普通方法，一个同步方法
     * 先hello，后发短信，因为hello不是同步方法，所以另一个线程不需要等待前面的线程
     */
    public void test3() {
        Phone p = new Phone();

        new Thread(() -> {
            p.sendSmsDelay();
        }, "A").start();
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        new Thread(() -> {
            p.hello();
        }, "B").start();
    }

    /**
     * 两个资源对象，一个同步方法，一个同步延迟方法
     * 先打电话，因为线程B不需要等待线程A（锁不同），后发短信
     */
    public void test4() {
        Phone p = new Phone();
        Phone p2 = new Phone();

        new Thread(() -> {
            p.sendSmsDelay();
        }, "A").start();
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        new Thread(() -> {
            p2.call();
        }, "B").start();
    }

    /**
     * 一个资源对象，两个静态同步方法
     * 两个线程获得的锁都是Phone.Class锁，先发短信，后打电话。
     * 由于静态成员属于类，所以不论资源对象有几个，线程处理仍是先到先得
     */
    public void test5() {
        Phone p = new Phone();
        new Thread(Phone::sendSmsStatic, "A").start();
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        new Thread(Phone::callStatic, "B").start();
    }

    /**
     * 两个对象，两个静态同步方法,
     * 由于是静态方法，锁的是Phone.Class,所以无论几个对象，仍然是按线程的执行顺序确定方法的执行顺序
     */
    public void test6() {
        Phone p = new Phone();
        Phone p2 = new Phone();
        new Thread(() -> {
            p.sendSmsStatic();
        }).start();
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {

        }
        new Thread(() -> {
            p2.callStatic();
        }).start();

    }

    /**
     * 一个资源对象，一个普通同步方法，一个静态同步方法
     * 线程获得的一个为对象锁，一个为Phone.class锁，所以进程B不需要等待进程A
     * 即先打电话后发短信
     */
    public void test7() {
        Phone p = new Phone();
        new Thread(() -> {
            p.sendSmsStatic();
        }, "A").start();
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        new Thread(() -> {
            p.call();
        }, "B").start();
    }

    /**
     * 两个资源，一个普通同步方法，一个静态同步方法
     * 道理同7，仍是先打电话，后发短信
     */
    public void test8() {
        Phone p = new Phone();
        Phone p2 = new Phone();
        new Thread(() -> {
            p.sendSmsStatic();
        }, "A").start();
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        new Thread(() -> {
            p2.call();
        }, "B").start();
    }

    public static void main(String[] args) {
        new Test1().test7();
    }
}

class Phone {
    public synchronized void sendSmsDelay() {
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("发短信");
    }

    public synchronized void sendSms() {
        System.out.println("发短信");
    }

    public synchronized void call() {
        System.out.println("打电话");
    }

    public static synchronized void sendSmsStatic() {
        try {
            TimeUnit.SECONDS.sleep(4);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("发短信");
    }

    public static synchronized void callStatic() {
        System.out.println("打电话");
    }

    public void hello() {
        System.out.println("hello");
    }
}
```

# 集合类不安全

## List

解决方法

```java
List<Integer> list = new Vector<>();
List<Integer> list2 = Collections.synchronizedList(new ArrayList<>());
List<Integer> list3 = new CopyOnWriteArrayList<>();
```

> CopyOnWriteArrayList，写入时复制，避免写入时被覆盖
>
> 好于Vector是因为vector在修改时使用的synchronized关键字，而CopyOnWriteArrayList在方法中使用的是ReentreantLock

## Set

```java
public static void main(String[] args) {
    Set<String> set = new HashSet<>();
    Set<String> set2 = Collections.synchronizedSet(new HashSet<>());
    Set<String> set3=new CopyOnWriteArraySet<>();
    for (int i = 1; i <= 30; i++) {
        new Thread(() -> {
            set3.add(UUID.randomUUID().toString().substring(0, 5));
            System.out.println(set3);
        }, String.valueOf(i)).start();
    }
}
```

## Map

```java
Map<String, Object>map=new ConcurrentHashMap<String, Object>(16, (float) 0.75);
```

# Callable

类似`Runnable`接口，有返回值，可以跑出一次（`Runnable`不能），方法是`call()`

但是`Callable`无法直接启动，需要通过`Runnable`启动 

```java
public class MyCallable {
    public static void main(String[] args) {
        FutureTask runnable=new FutureTask<>(new MyThread());
        new Thread(runnable,"callable test").start();
        //这里只打印一个call，结果会被缓存，结果可能需要等待
        new Thread(runnable,"callable test2").start();
        try {
            //这个get可能会产生阻塞
            String o = (String) runnable.get();
            System.out.println(o);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
}

class MyThread implements Callable<String> {
    @Override
    public String call() throws Exception {
        System.out.println("call()");
        return "测试callable";
    }
}
```

# 常用辅助类

## CountDownLatch

==减法计数器==

```java
public class CountDownLatchTest {
    public static void main(String[] args) throws InterruptedException {
        //初始值
        CountDownLatch countDownLatch = new CountDownLatch(6);
        for (int i = 0; i < 6; i++) {
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + "go out");
                //数量减一
                countDownLatch.countDown();
            }, String.valueOf(i)).start();
        }
        //等待计数器归零再向下执行
        countDownLatch.await();
        System.out.println("关门");
    }
}
```

## CyclicBarrier

==加法计数器==

```java
public static void main(String[] args) {
    CyclicBarrier cyclicBarrier = new CyclicBarrier(7, () -> {
        System.out.println("ok。。");
    });
    for (int i = 0; i < 7; i++) {
        final int temp = i;
        new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + " 收集了" + temp + "个");
            try {
                cyclicBarrier.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
        }).start();
    }
}
```

## Semaphore

==信号量==

```java
public static void main(String[] args) {
    // 线程数量，坑。限流时用的到
    Semaphore semaphore = new Semaphore(3);
    for (int i = 0; i < 6; i++) {
        new Thread(() -> {
            //acquire得到
            try {
                semaphore.acquire();
                System.out.println(Thread.currentThread().getName()+"抢到坑位");
                TimeUnit.SECONDS.sleep(2);
                System.out.println(Thread.currentThread().getName()+"离开坑位");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }finally {
                //release释放
                semaphore.release();
            }
        },"线程"+String.valueOf(i)).start();
    }
}
```

多个线程使用有限的资源（资源不能同时被多个线程使用）时使用

# 读写锁

```java
/**
 * @author: 1298509345
 * date: 2020/6/14
 * Time: 17:01
 * Describe:排它锁（写锁，同时仅一个线程占用），共享锁（读锁，同时多个线程占用）
 */
public class ReadWriteLockTest {
    public static void main(String[] args) {
        MyCache myCache = new MyCache();
        //写入
        for (int i = 0; i < 5; i++) {
            int finalI = i;
            new Thread(() -> {
                myCache.put(finalI + "", finalI + "");
            }, "线程" + i).start();
        }

        //读取
        for (int i = 0; i < 5; i++) {
            int finalI = i;
            new Thread(() -> {
                myCache.get(finalI + "");
            }, "线程" + i).start();
        }
    }
}

/**
 * 自定义缓存
 */
class MyCache {
    private volatile Map<String, Object> map = new HashMap<>();
    //读写锁,比ReentrantLock更细粒度
    private ReadWriteLock lock = new ReentrantReadWriteLock();
    //写，希望同时只有一个线程写
    public void put(String key, Object val) {
        lock.writeLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + "写入key");
            map.put(key, val);
            System.out.println(Thread.currentThread().getName() + "写入ok");
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.writeLock().unlock();
        }

    }

    //读
    public void get(String key) {
        lock.readLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + "读取");
            Object o = map.get(key);
            System.out.println(Thread.currentThread().getName() + "读取ok");
        }catch (Exception e){
            e.printStackTrace();
        }finally {
            lock.readLock().unlock();
        }

    }
}
```

# 阻塞队列

队列为空，必须等待队列生产才能消费。队列为满，必须等待消费才能继续生产。

## BlockingQueue

使用，4组API

| 方式       | 抛出异常 | 有返回值，不抛出异常 | 阻塞等待(一直等待) | 超时等待(超过时间退出等待)                |
| ---------- | -------- | -------------------- | ------------------ | ----------------------------------------- |
| 添加       | add      | offer                | put                | `offer(E e, long timeout, TimeUnit unit)` |
| 移除       | remove   | poll                 | take               | `poll(long timeout, TimeUnit unit)`       |
| 查看队列首 | element  | peek                 | -                  | -                                         |

阻塞等待

```java
public static void test1() throws InterruptedException {
    ArrayBlockingQueue<String> arrayBlockingQueue = new ArrayBlockingQueue<>(3);
    arrayBlockingQueue.put("1");
    arrayBlockingQueue.put("2");
    arrayBlockingQueue.put("3");
    System.out.println(arrayBlockingQueue.take());
    System.out.println(arrayBlockingQueue.take());
    System.out.println(arrayBlockingQueue.take());
    System.out.println(arrayBlockingQueue.take());
    arrayBlockingQueue.put("4");
}
```

超时等待

```java
public static void test2() throws InterruptedException {
    ArrayBlockingQueue<String> arrayBlockingQueue = new ArrayBlockingQueue<>(3);
    arrayBlockingQueue.offer("1");
    arrayBlockingQueue.offer("2");
    arrayBlockingQueue.offer("3");
    arrayBlockingQueue.offer("4", 2, TimeUnit.SECONDS);
}
 public static void test3() throws InterruptedException {
        ArrayBlockingQueue<String> arrayBlockingQueue = new ArrayBlockingQueue<>(3);
        arrayBlockingQueue.offer("1");
        arrayBlockingQueue.offer("2");
        arrayBlockingQueue.offer("3");
        System.out.println(arrayBlockingQueue.poll());
        System.out.println(arrayBlockingQueue.poll());
        System.out.println(arrayBlockingQueue.poll());
        System.out.println(arrayBlockingQueue.poll(2,TimeUnit.SECONDS));
    }
```

> SynchronousQueue 同步队列
>
> 最多只放一个元素，进去一个元素，必须等待元素出来才能加入。

```java
public static void main(String[] args) throws InterruptedException {
    SynchronousQueue<String> objects = new SynchronousQueue<>();
    new Thread(()->{
        try {
            System.out.println(Thread.currentThread().getName()+"put:1");
            objects.put("1");
            System.out.println(Thread.currentThread().getName()+"put:2");
            objects.put("2");
            System.out.println(Thread.currentThread().getName()+"put:3");
            objects.put("3");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    },"t1").start();
    new Thread(()->{
        try {
            TimeUnit.SECONDS.sleep(2);
            System.out.println(Thread.currentThread().getName()+"get:"+objects.take());
            TimeUnit.SECONDS.sleep(2);
            System.out.println(Thread.currentThread().getName()+"get:"+objects.take());
            TimeUnit.SECONDS.sleep(2);
            System.out.println(Thread.currentThread().getName()+"get:"+objects.take());
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    },"t2").start();
}
```

# 线程池

> 池化技术
>
> 程序的运行就会占用系统资源，所以要优化资源的使用！=>池化技术
>
> 线程池，连接池，内存池，对象池
>
> 池化技术即事先准备好资源，有对象使用资源来拿，用完之后返还给资源池

线程池的好处

1. 降低资源的消耗
2. 提高响应效率
3. 方便管理，方便线程复用，控制最大并发数

> 线程池：三大方法，七大参数，四种拒绝策略

- 三大方法

```java
public static void main(String[] args) {
    //单个线程
    ExecutorService threadPool = Executors.newSingleThreadExecutor();
    //固定线程数量
    ExecutorService threadPool = Executors.newFixedThreadPool(5);
    //可伸缩线程数目
    ExecutorService threadPool = Executors.newCachedThreadPool();

    try {
        for (int i = 0; i < 100; i++) {
            //线程池创建线程
            threadPool.execute(() -> {
                System.out.println(Thread.currentThread().getName() + "->ok");
            });
        }
    } finally {
        //线程池用完需要关闭
        threadPool.shutdown();
    }

}
```

> 创建一个线程池

```java
public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
}
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
}
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

- 七大参数

```java
public ThreadPoolExecutor(int corePoolSize,//核心线程池大小，一直为开启状态的线程数量
                          int maximumPoolSize,//线程池最大容量，需求量较多时，可开启的最多线程数
                          long keepAliveTime,//超时未用释放
           TimeUnit unit,//超时单位
           BlockingQueue<Runnable> workQueue,//阻塞队列，线程数量不满足需求的是等待队列
           ThreadFactory threadFactory,//线程工厂
           RejectedExecutionHandler handler/*拒绝策略，当线程数量不够且阻塞队列已满时的拒绝策略*/) {
    if (corePoolSize < 0 ||
        maximumPoolSize <= 0 ||
        maximumPoolSize < corePoolSize ||
        keepAliveTime < 0)
        throw new IllegalArgumentException();
    if (workQueue == null || threadFactory == null || handler == null)
        throw new NullPointerException();
    this.acc = System.getSecurityManager() == null ?
            null :
            AccessController.getContext();
    this.corePoolSize = corePoolSize;
    this.maximumPoolSize = maximumPoolSize;
    this.workQueue = workQueue;
    this.keepAliveTime = unit.toNanos(keepAliveTime);
    this.threadFactory = threadFactory;
    this.handler = handler;
}
```

- 4种拒绝策略

```java
public static void main(String[] args) {
    //手动创建线程池
    ExecutorService threadPool = new ThreadPoolExecutor(
            2,
            5,
            3,
            TimeUnit.SECONDS,
            new LinkedBlockingQueue<>(3),
            Executors.defaultThreadFactory(),
            new ThreadPoolExecutor.DiscardOldestPolicy());
    try {
        /**
         max=maxNumPoolSize+Queue.size
         AbortPolicy：超出最大数目，抛出异常RejectedExecutionException
         CallerRunsPolicy：哪个线程调用的资源线程，返回那个线程执行，此情况即为返回main线程执行
         DiscardPolicy：阻塞队列满了，会丢掉任务，不抛出异常。
         DiscardOldestPolicy：抛弃阻塞队列中最旧的任务（即最早入队的），添加新的任务进去
         */
        for (int i = 0; i < 9; i++) {
            //线程池创建线程
            threadPool.execute(() -> {
                System.out.println(Thread.currentThread().getName() + "->ok");
            });
        }
    } finally {
        //线程池用完需要关闭
        threadPool.shutdown();
    }
}
```

> ```java
> 线程池最大线程数量定义
> CPU密集型：CPU可执行线程的核数 Runtime.getRuntime().availableProcessors()
> IO密集型：大于程序中十分号IO的线程数目 
> ```

# 四大函数式接口（重点）

新时代程序员，lambda表达式，链式编程，函数式接口，Stream流式计算

> 函数式接口：只有一个方法的接口
>
> 四大函数式接口：`Consumer，Function，Predicate，Supplier`

**函数型接口**

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```

```java
public static void main(String[] args) {
    //函数型街口
    //有一个输入参数，有一个输出
    //函数接口就可用lambda表达式简化
    Function<String, String> function = new Function<String, String>() {
        @Override
        public String apply(String str) {
            return str;
        }
    };
    Function<String, String> function2 = (str) -> {
        return str;
    };
    System.out.println(function2.apply("asd"));
}
```

**判定型接口**

```java
@FunctionalInterface
public interface Predicate<T> {
    /**
     * 判断输入参数.
     */
    boolean test(T t);
}
```

```java
public static void main(String[] args) {
    Predicate<String> predicate = new Predicate<String>() {

        @Override
        public boolean test(String str) {
            return str.isEmpty();
        }
    };
    Predicate<String> predicate2 = (str) -> {
        return str.isEmpty();
    };
    
    System.out.println(predicate.test(""));
}
```

**消费型接口,供给型接口**

```java
@FunctionalInterface
public interface Consumer<T> {

    /**
     * 只有输入
     */
    void accept(T t);
 }
```

```java
public static void main(String[] args) {
    Consumer<String> consumer = new Consumer<String>() {
        @Override
        public void accept(String o) {
            System.out.println(o);
        }
    };
    Consumer<String> consumer2 = o -> System.out.println(o);
    consumer.accept("s");
	
    Supplier<Integer> supplier=new Supplier<Integer>() {
            @Override
            public Integer get() {
                return 1024;
            }
     };
    supplier.get();
}

```

# Stream流式计算

> 什么是流式计算
>
> 集合，数据库都是作为存储，计算交给流

```java
public class Demo1 {
    public static void main(String[] args) {
        User a = new User(1, "a", 21);
        User b = new User(2, "b", 22);
        User c = new User(3, "c", 23);
        User d = new User(4, "d", 24);
        User e = new User(6, "e", 25);
        List<User> users = Arrays.asList(a, b, c, d, e);
        /**
         * id为偶数
         * 年龄大于23
         * 名字转为大写
         * 倒序
         * 输出限制
         */
        users.stream().filter((u) -> u.getId() % 2 == 0)
                .filter(user -> user.getAge() > 23)
                .map(user -> user.getName().toUpperCase())
                .sorted((u1, u2) -> {
                    return u2.compareTo(u1);
                })
                .limit(1)
                .forEach(System.out::println);
    }
}

@Data
@AllArgsConstructor
@NoArgsConstructor
class User {
    Integer id;
    String name;
    Integer age;
}
```

# ForkJoin

> 分支合并，并行执行任务，提高效率
>
> 特点：工作窃取（每个线程的任务是个双端队列，一个线程任务完成，窃取其他线程的任务来做），不让线程等待
>
> 但必须在大数据量情况下使用，否则有可能造成线程抢占同一个任务造成问题

**操作**

```java
//ForkJoin计算类
public class ForkJoinDemo extends RecursiveTask<Long> {
    Long start;
    Long end;
    Long threshold = 10000L;

    public ForkJoinDemo(Long start, Long end) {
        this.start = start;
        this.end = end;
    }
    /**
     * 计算方法，
     *
     * @return
     */
    @Override
    protected Long compute() {
        if (end - start <= threshold) {
            long sum = 0L;
            for (Long i = start; i <= end; i++) {
                sum += i;
            }
            return sum;
        } else {//大于临界值用分治合并计算
            long mid = (start + end) / 2;
            ForkJoinDemo task1 = new ForkJoinDemo(start, mid);
            //把任务加入线程队列
            task1.fork();
            ForkJoinDemo task2 = new ForkJoinDemo(mid + 1, end);
            task2.fork();
            return task1.join() + task2.join();
        }
    }
}
```

```java
//测试类
public class ForkJoinTest {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        //sum=500000000500000000时间:2911
        //test1();
        //sum=500000000500000000时间:1900
        //test2();
        //sum=499999999500000000时间:118
        test3();
    }

    public static void test1() {
        long start = System.currentTimeMillis();
        long sum = 0L;
        for (long i = 1L; i <= 10_0000_0000; i++) {
            sum += i;
        }
        long end = System.currentTimeMillis();
        System.out.println("sum=" + sum + "时间:" + (end - start));
    }

    public static void test2() throws ExecutionException, InterruptedException {
        long start = System.currentTimeMillis();
        /**
         * ForkJoin使用，
         * ForkJoinPool.execute(ForkJoinTask<?>task)
         * 计算类要继承RecursiveTask<?>类
         */
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        ForkJoinDemo task = new ForkJoinDemo(0L, 10_0000_0000L);
        //提交任务
        ForkJoinTask<Long> res = forkJoinPool.submit(task);
        Long sum = res.get();
        long end = System.currentTimeMillis();
        System.out.println("sum=" + sum + "时间:" + (end - start));
    }

    public static void test3() {
        long start = System.currentTimeMillis();
        //Stream并行流
        /**
         * range 左闭右闭区间
         * rangeClose 左闭右开区间
         */
        long reduce = LongStream.range(0L, 10_0000_0000L).parallel().reduce(0, Long::sum);
        long end = System.currentTimeMillis();
        System.out.println("sum=" + reduce + "时间:" + (end - start));
    }
}
```

# 异步回调

> Future,对将来的某个事件的结果进行建模 

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    //没有返回值的异步回调
    CompletableFuture<Void> completableFuture = CompletableFuture.runAsync(() -> {
        try {
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + "runAsync");
    });

    //有返回值的异步回调
    CompletableFuture<Integer> completableFuture2 = CompletableFuture.supplyAsync(() -> {
        int i = 10 / 0;
        System.out.println(Thread.currentThread().getName() + "supplyAsync=>Integer");
        return 1024;
    });

    //获取异步执行结果
    System.out.println(completableFuture2.whenComplete((t, u) -> {
        //正常的返回结果
        System.out.println("t = " + t);
        //有错返回错误信息，java.util.concurrent.CompletionException: java.lang.ArithmeticException: / by zero
        System.out.println("u = " + u);
    }).exceptionally((e) -> {
        System.out.println(e.getMessage());
        //获取错误返回结果
        return 233;
    }).get());
}
```

# JMM

JMM：java内存模型，一种概念。

**一些同步约定**：

1. 线程解锁前，必须把共享变量==立刻==刷回主存（内存在工作时会把主存的变量复制一份到自己的工作内存，操作的即时工作内存中的变量）
2. 线程加锁前，必须读取主存中的最新值到工作内存中。
3. 加锁和解锁是同一个锁。

![image-20200615184721799](JUC并发编程.assets/image-20200615184721799.png)

若线程B在执行时修改了flag，线程A不能及时可见

> **内存交互操作有8种，虚拟机实现必须保证每一个操作都是原子的，不可在分的（对于double和long类型的变量来说，load、store、read和write操作在某些平台上允许例外）**
>
> - lock   （锁定）：作  用于主内存的变量，把一个变量标识为线程独占状态
>
> - unlock （解锁）：作用于主内存的变量，它把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定
> - read  （读取）：作用于主内存变量，它把一个变量的值从主内存传输到线程的工作内存中，以便随后的load动作使用
> - load   （载入）：作用于工作内存的变量，它把read操作从主存中变量放入工作内存中
> - use   （使用）：作用于工作内存中的变量，它把工作内存中的变量传输给执行引擎，每当虚拟机遇到一个需要使用到变量的值，就会使用到这个指令
> - assign （赋值）：作用于工作内存中的变量，它把一个从执行引擎中接受到的值放入工作内存的变量副本中
> - store  （存储）：作用于主内存中的变量，它把一个从工作内存中一个变量的值传送到主内存中，以便后续的write使用
> - write 　（写入）：作用于主内存中的变量，它把store操作从工作内存中得到的变量的值放入主内存的变量中
>
> 　**JMM对这八种指令的使用，制定了如下规则：**
>
> - 不允许read和load、store和write操作之一单独出现。即使用了read必须load，使用了store必须write
>
> - 不允许线程丢弃他最近的assign操作，即工作变量的数据改变了之后，必须告知主存
> - 不允许一个线程将没有assign的数据从工作内存同步回主内存
> - 一个新的变量必须在主内存中诞生，不允许工作内存直接使用一个未被初始化的变量。就是怼变量实施use、store操作之前，必须经过assign和load操作
> - 一个变量同一时间只有一个线程能对其进行lock。多次lock后，必须执行相同次数的unlock才能解锁
> - 如果对一个变量进行lock操作，会清空所有工作内存中此变量的值，在执行引擎使用这个变量前，必须重新load或assign操作初始化变量的值
> - 如果一个变量没有被lock，就不能对其进行unlock操作。也不能unlock一个被其他线程锁住的变量
> - 对一个变量进行unlock操作之前，必须把此变量同步回主内存



> volatile，java虚拟机提供的**轻量级同步机制**
>
> 1. 保证可见性
> 2. ==不保证原子性==
> 3. 禁止指令重排

```java
//保证可见性
public class JMMDemo {
    static volatile int anInt=0;
    public static void main(String[] args) throws InterruptedException {
        new Thread(() -> {
            while (anInt==0){

            }
        },"A").start();
        TimeUnit.SECONDS.sleep(1);
        //main线程改变anInt，线程A不能获取最新值
        //加上volatile使得线程A可见最新值，保证可见性
        anInt=1;
        System.out.println("anInt = " + anInt);
    }
}
```

原子性：不可分割。

线程A在执行任务的时候不能被打扰，也不能分割，同时成功或者同时失败

```java
//不保证原子性
public class VolatileDemo {
    static volatile int num1 = 0;
	//解决方法，原子类的int
    static AtomicInteger num = new AtomicInteger();
    
    static void add() {
        //num1++;
        num.getAndIncrement();
    }

    public static void main(String[] args) {
        //理论结果为两万
        for (int i = 0; i < 20; i++) {
            new Thread(() -> {
                for (int j = 0; j < 1000; j++) {
                    add();
                }
            }).start();
        }
        while (Thread.activeCount() > 2) {
            //main,gc
            Thread.yield();
        }
        System.out.println(Thread.currentThread().getName() + " " + num);
    }
}
```

> 如果不加Lock和Synchronized，如何保证原子性
>
> 使用原子类，解决原子性问题。
>
> 原子类的底层与操作系统相关，它在内存中直接修改值，Unsafe类很特殊的存在。

## 什么是指令重排

计算机执行的顺序不一定是代码的编写顺序。

源代码=》编译器优化重排=》指令并行重排=》内存系统也会重排=》执行

==进行指令重拍会考虑数据之间的依赖性，所以不会有错误的重排顺序==

可能造成影响的结果：a，b、c，d默认都为0

| 线程A | 线程B |
| ----- | ----- |
| x = a | y = b |
| b = 1 | a = 2 |

正常结果：x = 0 ,y = 0 。但可能由指令重排，导致两个线程结果不符合预期。

| 线程A | 线程B |
| ----- | ----- |
| b = 1 | a = 2 |
| x = a | y = b |

**volatile可以避免指令重排：**

volatile实现内存屏障，禁止当前指令的前后指令交换顺序。

1. 保证特定操作的执行顺序
2. 保证某些变量的内存可见性

# 单例模式

饿汉式

```java
public class Hungary {
    private final static Hungary HUNGRY = new Hungary();

    private Hungary() {

    }
    public static Hungary getInstance() {
        return HUNGRY;
    }
}
```

懒汉式

```java
public class Lazy {
    //标志变量可解决通过反射构造函数创建对象
    private static boolean flag = false;

    private Lazy() {
        synchronized (Lazy.class) {
            if (!flag) {
                flag = true;
            } else {
                throw new RuntimeException("不要用反射破坏单例");
            }
        }
    }

    private static volatile Lazy lazy;

    public static Lazy getInstance() {
        //双重检测锁，DCL懒汉式
        if (lazy == null) {
            synchronized (Lazy.class) {
                if (lazy == null) {
                    /**
                     * 不是原子性操作
                     * 1.分配内存空间
                     * 2.执行构造方法，初始化对象
                     * 3.把这个对象指向这个空间
                     */
                    return new Lazy();
                }
            }
        }
        return lazy;
    }

    public static void main(String[] args) throws Exception {
        // Lazy instance = Lazy.getInstance();
        //使用反射破坏单例
        Constructor<Lazy> declaredConstructor = Lazy.class.getDeclaredConstructor(null);
        declaredConstructor.setAccessible(true);
        //通过反射控制属性，可破坏单例模式
		Field flag = Lazy.class.getDeclaredField("flag");
        flag.setAccessible(true);
        
        Lazy instance1 = declaredConstructor.newInstance();

        flag.set(instance1,false);

        Lazy instance2 = declaredConstructor.newInstance();
        System.out.println(instance1 == instance2);
    }
}
```

内部类

```java
public class Holder {
    private Holder(){}
    public Holder getInstance(){
        return Inner.HOLDER;
    }
    private static class Inner {
        private static final Holder HOLDER=new Holder();
    }
}
```

枚举类型，保证反射不能创建对象

```java
public enum EnumSingle {
    INSTANCE;

    public EnumSingle getIns() {
        return INSTANCE;
    }
}
class Test{
    public static void main(String[] args) throws Exception {
        EnumSingle instance = EnumSingle.INSTANCE;
        //枚举类型的反编译源码构造函数有两个参数
        Constructor<EnumSingle> declaredConstructor = EnumSingle.class.getDeclaredConstructor(String.class,int.class);
        declaredConstructor.setAccessible(true);
        EnumSingle enumSingle = declaredConstructor.newInstance();
        System.out.println(instance==enumSingle);
    }
}
```

# CAS

> 原子类中的 `unsafe` 类可以通过调用 `native` 方法，用`c++`操作内存

```java
// 	setup to use Unsafe.compareAndSwapInt for updates
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;

    static {
        try {
            //获取内存地址的偏移值
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }
	//具体的数值
    private volatile int value;
```

```java
public final int getAndIncrement() {
    return unsafe.getAndAddInt(this, valueOffset, 1);
}

public final int getAndAddInt(Object var1, long var2, int var4) {
        int var5;
        do {
            //获取内存中的值，var1为对象，var2位内存偏移值
            var5 = this.getIntVolatile(var1, var2);
        }//如果对象var1的值还是var5，就将这个偏移值加var4
    	 //由于是内存操作，效率很高。这其实是个自旋锁，不停地判断主存中的值是否和线程工作内存中的值一致
    	while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

        return var5;
}
```

> - CAS：比较当前工作内存中的值和主内存中的值，如果这个值是期望的值（即和工作内存的值一致），则执行操作，否则不执行。
>
> - 缺点：
>   1. 由于是自旋锁，循环会耗费时间
>   2. 一次性只能保证一个共享变量的原子性
>   3. 会存在ABA问题

## **ABA** 问题，狸猫换太子

若两个线程A，B共享一个变量 `a=1`，若线程B执行的比较快，且他进行了两次操作，`CAS(a,1,3),CAS(a,3,1)`，在这种情况下，A对改动不知情，当要操作`a`时，这个值实际上是已经被动过的。

```java
new Thread(()->{
    System.out.println(integer.compareAndSet(2020, 2021));
},"期望的线程").start();
new Thread(()->{
    System.out.println(integer.compareAndSet(2020, 2021));
    System.out.println(integer.compareAndSet(2021, 2020));
},"捣乱的线程").start();
```

# 原子引用

**带版本号的原子操作，解决ABA问题**

==**注意**==

 Integer在`[-128,127]`中的值会在`IntegerCache.cahce`产生，会复用已有对象，超出这个值会在堆上重新产生对象，就不能用`==`比较了。

在`AtomicStampedReference`中泛型用的`==`比较，如果是Integer超出`[-128,127]`会导致不相等，以至于`compareAndSet` 返回 `false`

```java
public static void main(String[] args) {
    //AtomicStampedReference(V initialRef, int initialStamp)
    //不过比较的引用类型一般为类似User，不会像Integer这样会重新生成。
    //初始值和初始版本号
    AtomicStampedReference<Integer> integer = new AtomicStampedReference<>(1, 1);
    new Thread(() -> {
        //获得版本号
        System.out.println("A初始版本号" + integer.getStamp());
        try {
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        //期望的值，新值，期望的版本号，新版本号（当前版本号加一）
        System.out.println("A第一次修改：" + integer.compareAndSet(1, 2,
                integer.getStamp(), integer.getStamp() + 1) +
                "，版本号为：" + integer.getStamp());


        //期望的值，新值，期望的版本号，新版本号（当前版本号加一）
        System.out.println("A第二次修改：" + integer.compareAndSet(2, 1,
                integer.getStamp(), integer.getStamp() + 1) +
                "，版本号为：" + integer.getStamp());

    }, "线程A").start();
    new Thread(() -> {
        int stamp = integer.getStamp();
        System.out.println("B初始版本号" + stamp);
        try {
            TimeUnit.SECONDS.sleep(2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("B第一次修改：" + integer.compareAndSet(1, 6, stamp, stamp + 1)
                + "，版本号为：" + integer.getStamp());
    }, "线程B").start();
}
```

# 各种锁的理解

## 公平锁

非常公平，先来后到，不能插队

## 非公平锁

不公平，可以插队，谁先来谁用（默认锁方式）

```java
public ReentrantLock() {
    sync = new NonfairSync();
}
public ReentrantLock(boolean fair) {
	sync = fair ? new FairSync() : new NonfairSync();
}
```

## 可重入锁

也叫递归锁。

拿到外层的锁，自动获得里层的锁。

```java
public class Test1 {
    public static void main(String[] args) {
        Phone phone = new Phone();
        new Thread(() -> {
            phone.msg();
        }, "A").start();

        new Thread(() -> {
            phone.call();
        }, "B").start();
    }
}
//synchronized版本
class Phone {
    public synchronized void msg() {
        //外层锁包含了里层，所以线程A执行msg方法会自动获得里层的锁，当里层执行完，其他线程才能获得锁。
        System.out.println(Thread.currentThread().getName() + "msg");
        //这里也有锁
        call();
    }

    public synchronized void call() {
        System.out.println(Thread.currentThread().getName() + "call");
    }
}
//lock版本
class Phone2 {
    Lock lock = new ReentrantLock();

    public void msg() {
        //锁必须配对，加锁几次，解锁就几次，否则会死锁
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + "msg");
            //这里也有锁
            call();
        } finally {
            lock.unlock();
        }
    }

    public void call() {
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getName() + "call");
        } finally {
            lock.unlock();
        }
    }
}
```

## 自旋锁

 自定义锁测试。

```java
public class Spinlock {

    AtomicReference<Thread> atomicReference = new AtomicReference<>();

    /**
     * 加锁
     */
    public void myLock() {
        Thread thread = Thread.currentThread();
        System.out.println(thread.getName() + "->myLock");
        //自旋锁
        while (atomicReference.compareAndSet(null, thread)) {

        }
    }
    /**
     * 解锁
     */
    public void myUnLock() {
        Thread thread = Thread.currentThread();
        System.out.println(thread.getName() + "->myUnLock");
        atomicReference.compareAndSet(thread, null);
    }
}
```

```java
public static void main(String[] args) throws InterruptedException {
    Spinlock lock = new Spinlock();
    new Thread(() -> {
        lock.myLock();
        try {
            TimeUnit.SECONDS.sleep(3);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.myUnLock();
        }
    }, "A").start();
    TimeUnit.SECONDS.sleep(1);
    new Thread(() -> {
        lock.myLock();
        try {
            TimeUnit.SECONDS.sleep(5);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.myUnLock();
        }

    }, "B").start();
}
```

# 死锁

线程互相请求对方持有的锁。

**死锁问题**

```java
public class DeadLock {
    public static void main(String[] args) {
        new Thread(new MyThread("lock-A","lock-B"),"线程1").start();
        new Thread(new MyThread("lock-B","lock-A"),"线程2").start();
    }
}

class MyThread implements Runnable {
    String lockA;
    String lockB;

    public MyThread(String lockA, String lockB) {
        this.lockA = lockA;
        this.lockB = lockB;
    }

    @Override
    public void run() {
        synchronized (lockA) {
            System.out.println(Thread.currentThread().getName() +
                    "，得到锁:" + lockA + ",get:" + lockB);
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            synchronized (lockB) {
                System.out.println(Thread.currentThread().getName() +
                        "，得到锁:" + lockB + ",get:" + lockA);
            }
        }
    }
}
```

**死锁排除**

1. 使用`jps-l`定位进程号

   ```shell
   D:\IdeaProjects\JUC>jps -l
   6112 org.jetbrains.idea.maven.server.RemoteMavenServer36
   6704 sun.tools.jps.Jps
   2756 com.lock.DeadLock
   6692 org.jetbrains.kotlin.daemon.KotlinCompileDaemon
   11496 org.jetbrains.jps.cmdline.Launcher
   ```

2. 使用`jstack 进程号`查看进程堆栈信息，找到死锁问题

![image-20200616143108851](JUC并发编程.assets/image-20200616143108851.png)