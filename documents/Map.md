# Map

SynchronizedMap（线程安全）

HashMap&Hashtable&concurrentHashmap



Hashmap：

数组+链表



jdk1.7中数组中存储的叫entry

jdk1.8中数组中存储的叫node



当发生冲突的时候会将新插入的数据连接到当前的位置上，这个时候就形成了链表

插入链表的时候，在java8之前使用的是头插法，原因是作者认为后插入的数组更可能被查找，在java8之后就改为了尾插法。



为什么改为尾插法？

首先当数组的容量是有限的，当数据多次插入后，达到一定的数量数组就会进行扩容，这个时候就是resize；

resize时，有两个因素， capacity和LoadFactor（负载因子）默认为0.75f，当容量为100时，当插入第76个数据的时候，就要进行扩容；



扩容分为两步，首先创建一个新的数组，长度为原来的两倍；之后进行Rehash，把所有的Entry重新hash进数组；

原因是数组长度发生变化以后，hash的规则也将随之改变；



为什么改为尾插？因为当put的时，若有多个线程同时插入数据A，B，C时，可能出现环形链表，就会出现死循坏；

故改为尾插法；

尾插法在插入数据时，因为数据都是往后放的，所以不会出现环形链表；

但是即使使用尾插法，也不代表在多线程中Hashmap就是安全的，因为Hashmap中的put方法中并没有加锁；



Hashmap的初始化长度为16，之所以为16应该是因为统计学的原理，认为人们所需要的容量在这个大小；

而为什么是2的幂则是为了实现平均分布，2的幂次-1后其二进制位数全为1，故在与传入的key的hash值与容量大小-1做运算；



为什么重写equals后需要重写hashcode？

在未重写equals方法时所使用的是object的equals方法，其操作是比较两个对象的内存地址，但因为是两个对象，故equals的答案肯定是false；在使用get方法的时候，通过key的hashcode寻找index，同一index上的不同对象，equals的答案是false，如果我们对equals进行了重写，那就建议一定要对hashcode方法进行重写，保证相同的对象返回相同的hash值，不同对象返回不同的hash值；



**SynchronizedMap**如何实现线程安全的？

其内部维护了一个普通对象Map，还有排斥所mutex，当我们需要调用该方法的时候就需要传入一个Map，若传入mutex参数，则将对象排斥锁赋值未传入的对象，如果没有则将对象排斥锁赋为this，即其内部维护的map，此后再操作map时，就会对对象上锁；



**Hashtable**

hashtable较hashmap来说是线程安全的，适合在多线程的情况下使用，但是效率不太高；

效率低的原因就是在使用hashtable的时候，对数据操作时都会对该对象上锁，所以效率比较低下；



与Hashmap不同的地方在于，Hashmap中允许出现键值未null的情况，但hashtable是不允许出现为null的情况，因为由于null不是对象，无法调用或使用equals和hashcode方法故不能存放空值；

而hashmap中对null值都进行了特殊处理，若为空则赋为0进行hash；

这是因为hashtable使用的是安全失败机制（fail-safe），这会使得我们此次读到的数据不一定是最新的数据；

如果使用null，就会使得hashtable无法判断其对应的key是不存在还是为空；



hashmap和hashtable两者不同点：

hashtable继承了dictionary类，而hashmap继承的是abstractmap类；

初始化容量不同，hashmap是16，hashtable为11，两者的负载因子都是0.75f；

扩容机制不同，hashmap是翻倍，hashtable是翻倍+1；

迭代器不同，hashmap是fail-fast，而hashtable不是；

（fail-fast，迭代器遍历一个对象时，如果遍历过程中集合对象中的数据内容被进行了修改，则会抛出异常，其迭代器在遍历集合时，会使用一个modcount变量，当迭代器遍历下一个元素之前，都会检测modcount，如果与预期modcount不符，则会抛出异常，终止遍历。）



**concurrenthashmap**

线程安全；

并发度高；

其底层时数据+链表组成的；

java8之前，其是由segment数组、hashentry组成，和hashmap一样；

Hashentry和hashmap差不多，不同点是，他使用了volatile修饰了数据value和下一个节点next；

（volatile保证了不同线程对同一个变量的可见性，即如果修改了这个变量，其他线程会立即看到新值；禁止了指令的重排序，如在拥有二次检查的单例模式下，指令的重排可能会导致在单例对象时获得的是一个尚未完全初始化的变量；volatile只能保证对单次读/写的原子性。但不能保证i++这样操作的原子性；）

并发度高的原因，使用了分段锁技术，其中segement继承于reentrantLock，不管是put或者是get都需要做加锁同步处理，当一个线程访问一个segment时，不会影响到其他的segment；并发度与容量大小相同；

put过程中首先会获取锁，若失败说明有其他线程在竞争，则利用scanandlockforput进行获取，进行短暂的自旋之后若还是无法获取锁则会升级为重量级锁保证成功；

get过程中因为有volatile修饰，故一定能够获取到最新的数据，所以不需要加锁；

但是java8之前遍历中还是存在一些问题，我们在查询的时候还得要遍历链表，导致效率低，所以在java8中得到了优化；

抛弃了segment分段锁，采用了CAS+synchronized来保证并发安全；

和Hashmap很像，将Hashentry改成了node，把值与next采用volatile修饰，保证可见性，同时引入了红黑树，在链表大于一定值时会转换（默认为8）；

put中根据key计算出hashcode；

判断是否需要初始化；

定位到node以后，判断是否为空，若为空则直接写入数据，利用CAS尝试写入，失败则自旋保证成功；

当达到一定值时进行扩容；若都不满足则利用synchronized锁写入数据；

当链表长度达到8时转化为红黑树；