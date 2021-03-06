# JVM

### **JVM**：

- 是 Java Virtual Machine 的缩写，模拟实际计算机来实现计算机的功能。组成包括：堆，方法区，栈，本地方法栈，程序计算器等部分。

- 其中方法区和回收堆是共享区，谁都可以使用。
- 栈和程序计算器、本地方法栈区归JVM所有。

**作用：Java源文件经编译成字节码程序，通过JVM将每一条指令翻译成不同平台机器码，通过特定平台运行。**



### 执行过程：

1. ##### 加载.class文件

2. ##### 管理并分配内存

3. ##### 执行垃圾收集

   

### JVM实例

1. 一个独立运行的java程序——进程级别

   一个运行时的java虚拟机（JVM）负责运行一个java程序。

   当启动一个java程序时，一个虚拟机实例诞生；当程序关闭推出，这个虚拟机实例也就随之消亡。

   如果在同一台计算机上同时运行多个java程序，将得到多个java虚拟机实例，每个java程序都运行于它自己的java虚拟机实例中。

   

   2.JVM执行引擎实例则对应了署于运行程序的线程——线程级别。



### JVM的生命周期：

1. JVM实例的诞生

   当启动一个java程序时，一个jvm实例就产生了，任何一个拥有

   ```java
   public static void main(String[] args)
   ```

   的class都可以作为JVM实例运行的起点。

2. JVM实例的运行

   main()作为该程序初始线程的起点，任何其他线程均有该线程启动，JVM内部由两种线程：**守护线程和非守护线程，main（）属于非守护线程**，守护栈同城由JVM自己使用，java程序也可以标明自己创建的线程时守护线程。

3. JVM实例的消亡

   当线程中所有非守护线程都终止时，JVM才退出；若安全管理器运行，程序也可以使用java.lang.Runtime类或者Java.lang.System.exit()来退出。

   



## 实现原理

### 类装载器（ClassLoad ）：

- 主要负责加载 .class 文件
- 是否能执行主要取决于Execution engine（执行字节码，执行本地方法）
- 运行时数据区：方法区，堆，栈，程序计数器，本地方法栈

#### 生命周期：

​	**加载(Loading)、验证(Verification)、准备(Preparation)、解析(Resolution)、初始化(Initialization)、使用(Using)、卸载(Unloading)**

![image-20200818103535898](C:\Users\xiong\AppData\Roaming\Typora\typora-user-images\image-20200818103535898.png)

**加载：**

1. 通过"类全名"来获取定义此次类的二进制字节流
2. 将字节流所代表的静态存储结构转换为方法区的运行时数据结构
3. 在java堆中生成一个代表这个类的java.lang.Class对象，作为方法区这些数据的访问入口

**验证：**这一步主要的目的时确保class文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身安全。

**准备：**该阶段是正式为**类变量分配内存并设置类变量初始值**的阶段，这些内存都将在方法去中进行分配。

**解析：**解析阶段是虚拟机常量池内的符号引用替换为直接引用的过程。

**初始化：**类加载最后阶段，若该类具有超类，则对其进行初始化，执行静态初始化器和静态初始化成员变量。



### 执行引擎：

**解释器：**一条一条地读取，解释并且执行字节码指令。因为是一条条解释和执行，所以会很快的解释字节码，但执行起来较慢。

**即时编译器：**被引入用来弥补解释器的缺点，在解释器解释执行时，在合适的时候将整段字节码编译本地代码，然后引擎就没有必要再解释执行方法了，而可以直接通过本地代码区执行。提高效率。



### 本地方法栈——非线程共享：

主要作用是等级native方法，然后再execution engine执行的时候加载本地方法库。

**Java栈是为执行Java方法服务的**，而**本地方法栈则是为执行本地方法（Native Method）服务的**。

在JVM规范中，并没有对本地方法栈的具体实现方法以及数据结构作强制规定，虚拟机可以自由实现它。在HotSopt虚拟机中直接就把本地方法栈和Java栈合二为一。



### Java栈（栈内存）——非线程共享：

- 又叫栈内存，负责Java程序的运行，它是在线程创建时创建的，所以生命周期也是和线程生命周期一致，同时消亡，线程结束了栈也就释放。
- 特别提醒的是**栈不存在垃圾回收的问**题，因为线程结束栈就是释放了。

**Java栈是Java方法执行的内存模型。**

Java栈中存放的是一个个栈帧，每个栈帧对应一个被调用的方法。

栈帧中包括：**局部变量表**、**操作数表**、**指向当前方法所属的类的运行时常量池的引用**、**方法返回地址和一些额外的附加信息**。

当线程执行一个方法时，就会随之创建一个对应的栈帧，并将简历的栈帧压栈，方法执行完毕后栈帧出栈。

**局部变量表：**用来存储方法中的局部变量。对于基本数据类型的变量，则直接存储它的值，对于引用类型的变量，则存的是指向对象的引用。局部变量表的大小在编译期就可以确定大小了，所以执行期间其大小不会改变。

**操作数栈：**栈最典型的一个应用就是表达式求值。

**指向运行时常量池的引用：**因为在方法执行的过程中有可能需要用到类中的常量，所以必须要有一个引用指向运行时常量。

**方法返回地址：**当一个方法执行完毕之后，要返回之前调用它的地方，因此在栈帧中必须保存一个方法返回地址。



### 程序计数器——非线程共享：

程序计数器也成为PC寄存器。

- 是指方法去中的方法字节码由引擎读取下一条指令，它是一个非常小的内存空间。
- 每个线程都有一个程序计数器，是线程私有的，相当于一个指针。

由于程序计数器中存储的数据所占空间的大小不会随程序的执行而发生改变，因此，对于程序计数器是不会发生内存溢出现象的。

在JVM规范中规定，如果线程执行的是非native（本地）方法，则程序计数器中保存的是当前需要执行的指令的地址；如果线程执行的是native方法，则程序计数器中的值是undefined。

### 方法区——线程共享：

- 是指线程共享的，谁都可以共享使用的。
- 通常用来保存装载类的元结构信息。

方法区存储了每个类的信息，类中静态变量，定义为final类型的常量，类中的方法信息以及编译器编译后的代码等。

方法区中的非常重要的部分——**运行时常量池**

**运行时常量池：**用于存放静态编译产生的自变量和符号引用。运行时产生的常量也会存在这个常量池中，比如String和intern方法。它是每一个类或接口的常量池的运行时表示形式，在类和接口被加载到JVM后，对应的运行时常量池就被创建出来。



### 堆——线程共享：

- 是Java虚拟机用来存储对象实例的
- 比如我们在开发过程中使用的new对象，只要通过new创建的对象的内存的对象都在堆分配。
- 需要注意的是堆中的对象内存需要等待垃圾器（GC）进行回收，就是JVM共享区。

JAVA中的堆是用来**存储对象实例**以及**数组**。堆是被所有线程共享的，因此在其中进行对象内存的分配均需要进行加索，导致了new对象的开销是比较大的。

**JVM中只有一个堆**。

堆是JAVA垃圾收集器管理的主要区域，Java的垃圾回收机制会自动进行处理。

堆空间分为**老年代和年轻代**。

老年代：用于存放生命周期长久的实例对象。

年轻代：用于存放刚创建的对象。

### 本地接口（Native interface）：

- 融合不同的编程语言为java所用
- 底层是C、C++写的

### 垃圾回收器（Grabage Collection）:

- 作用域是在方法去和堆中
- 是Java的核心技术
- 主要通过确定对象引用来判断是否回收该对象

## 基本原理

将内存中不再使用的对象进行回收，GC中用于回收的方法成为收集器，由于GC需要消耗一些资源和时间，Java在对对象的生命周期特征进行分析后，按照新生代，旧生代的方式来对对象进行收集，以尽可能的缩短GC对应用造成的暂停。

本地栈，java栈和计数器随线程结束而结束，内存自然跟随回收。

而Java堆和方法区需要考垃圾收集器来处理。



1. ##### 引用计数法

   **堆中每个对象实例都有一个引用计数。当一个对象被创建时，就将该对象实例分配给一个变量，该变量计数设置为1。**当任何其他变量被赋值为这个对象的引用时，计数加1（a=b，则b引用的对象实例的计数器+1），当引用超过了生命周期或被设置为一个新值时，引用计数器减1.任何引用计数器为0的对象实例可以被当作垃圾收集。当一个对象实例被垃圾收集时，它引用的任何对象实例的引用计数器减1。

   **优点：很快的执行**，不会被长时间的打断

   **缺点：无法检测出循环引用。**

2. **可达性分析算法**

   程序把所有的引用关系看作一张图，从一个节点GC ROOTS开始，寻找对应的引用节点，找到之后继续寻找这个节点的引用节点，寻找完毕之后，剩余未被找到的节点则会被认为时没有被引用到的节点，将其判定为回收对象。

   作为GC ROOTS的对象包括下面几种

   - 虚拟机栈中引用的对象（栈帧中的本地变量表）；
   - 方法区中类静态属性引用的对象；
   - 方法区中常量引用的对象；
   - 本地方法栈中JNI（Java Native Interface）引用的对象；  

   不同的对象引用类型会采取不同的方法回收：

3. 标记-清除算法

   分为两个阶段：标记阶段和清除阶段。标记阶段标记出需要被回收的对象，清除阶段就是回收被标记的对象所占用的空间。

   **主要缺点：**效率不高，标记和清除过程的效率都不高。

   ​					空间问题，标记清除之后会产生大量不连续的内存碎片，空间碎片太多可能回导致：当程序在以后的运行过程中需要分配较大对象时无法找到足够的连续内存而不得不提前发出另一次垃圾收集动作。

   