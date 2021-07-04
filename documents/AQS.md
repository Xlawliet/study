# AQS





### 高并发

AbstractQueueSynchronizer

抽象队列同步器





AQS中维护着一个AtomicInteger叫 state

负责管理一个双向队列（CLH等待队列）

竞争到state的使用该资源，否则进入队列进行等待；

竞争到state的线程，通过setClusiveOwnerThread（互斥锁）来加锁得到



对对象进行加锁时，首先对象中的对象头中有两个部分

- Mark Word：用于存储Hashcode和分代年龄以及锁标志位信息
- Klass Point：对象指向它的类元数据的指针



释放锁：通过使用getExclusiveOwnerThread方法来判断当前线程是否唯一，否则易将其他线程的锁一同释放



ReentrantLock

CountDownLatch





