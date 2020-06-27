# Java虚拟机

![image-20200530215014248](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200530215014248.png)

## 其他问题

### volatile关键字

轻量级的同步机制：

- 保证可见性
- 不保证原子性
- 禁止指令重排

## JMM（Java内存模型）

JMM是一种抽象概念，并不真实存在，描述一组规则或规范，通过这组规范定义了程序中更变量（包括实例字段、静态字段、和构成数组对象的元素）的访问方式。定义了JVM在计算机内存中的工作方式。

抽象来看：JMM定义了线程和主内存之间的抽象关系，线程之间共享变量存储在主内存，每个线程都有一个私有的工作内存，工作内存中存储了该线程以读/写共享变量的副本

### JVM对Java内存模型的实现

在JVM中，Java内存模型把内存分成了线程栈区和队区。

### Java内存模型带来的问题

#### 可见性（内存模型）

##### JMM关于同步的规定

- 线程解锁前，必须把共享变量的值刷新回主内存
- 线程加锁前，必须读取主内存的最新值到自己的工作内存
- 加锁和解锁是同一把锁

![image-20200528170238607](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200528170238607.png)

速度：硬盘<内存<缓存cahce<cpu

![image-20200528170719487](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200528170719487.png)

内存条->主内存，线程访问主内存的一个对象，将该对象拷贝回自己线程的工作内存（而不是直接改主物理内存），操作完写回主内存。

##### 何为可见性

一个线程将主内存的变量修改后，其他线程都立即可见（通知）。线程间的通信必须通过主内存完成。

使用synchronized支持sync代码块之间的可见性。

使用volatile支持，验证：

```java
class MyData {
	int number = 0;
    public void addTo60() {
        this.number = 60;
    }
}

public class VolatileDemo {
    public static void main(String[] args) {
        MyData myData = new MyData();
        new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + " come in.");
            try {
                TimeUnit.SECONDS.sleep(3);
            } catch() {
                
            }
            myData.addTo60();
        	System.out.println(Thread.currentThread().getName() + " update number to " + myData.number);
        }, "A").start();
        
        // 然后是main线程
        while(myData.number == 0) {
            
        }
        System.out.println(Thread,currentThread().getName() + "mission over");
        // 这句话迟迟不打印，说明没有可见，加上volatile即可保证可见
    }
}
```



#### 原子性（竞争）

不可分割，完整，即某个线程执行某个业务时，不被其他线程干扰，需要整体完整，要么全部成功，要么全部失败。

```java
class MyData {
	volatile int number = 0;
    public void plus() {
        this.number++;
    }
}

public class VolatileDemo {
    public static void main(String[] args) {
		MyData mydata = new MyData();
        for (int i = 0; i <= 20; i++) {
            new Thread(() -> {
                for(int j = 0; j < 1000; j++) {
                    myData.plus();
                }
            }, String.valueOf(i)).start();
        }
        
        // 等待20个线程计算，再用main线程取得结果
        // 1. sleep(5)
        // 2.
        while(Thread.activeCount() > 2) {
            Thread.yield(); // 让出cpu
        }
        
        System.out.println(Thread.currentThread().getName() + "final value: " + myData.number);
    }
}
```

如何解决原子性？

- 原子类
- sync
- Lock

synchronized保证原子性，volatile不保。

#### 有序性（重排序）

volatile提供有序性的支持，禁止指令重排。

#### 重排序

编译器和处理器，为了提高并行度。

![image-20200528212426724](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200528212426724.png) 

处理器在进行重排序的时候需要考虑指令之间的数据依赖性。

- 单线程确保最终执行结果和代码顺序执行结果一样。

- 多线程交替，由于编译器优化重排的存在，两个线程中使用的变量能否保持一致性是无法确定的。

内存屏障（Memory Barrier）又称内存栅栏，一条CPU指令，作用：

- 保证特定操作的执行顺序
- 保证某些变量的内存可见性（利用该特性实现volatile的内存可见性）
- 由于编译器和处理器都能进行指令重排优化，如果在指令间插入一条Memory Barrier则会告诉编译器和CPU，不管什么指令都不能和这条Memory Barrier指令重排，即通过插入内存屏障禁止在内存屏障前后的指令执行重排序优化。
- 内存屏障的另外一个作用时强制刷出各种CPU的缓存数据，因此任何CPU上的线程都能读取到这些数据的最新版本

![image-20200528215703792](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200528215703792.png)

**as-if-serial规则**：不管如何重排序，都必须保证代码在单线程下的运行正确，执行结果不改变。为了遵守as-if-serial语义，**编译器和处理器不会对存在数据依赖关系的操作做重排序，因为这种重排序会改变执行结果**。但是，**如果操作之间不存在数据依赖关系，这些操作可能被编译器和处理器重排序**。

**happens-before规则**：JSR-133使用happens-before的概念来指定两个操作之间的执行顺序。由于这两个操作可以在一个线程之内，也可以是在不同线程之间。因此，JMM可以通过happens-before关系向程序员提供跨线程的内存可见性保证（如果A线程的写操作a与B线程的读操作b之间存在happens-before关系，尽管a操作和b操作在不同的线程中执行，但JMM向程序员保证a操作将对b操作可见）。具体的定义为：

1）如果一个操作happens-before另一个操作，**那么第一个操作的执行结果将对第二个操作可见，而且第一个操作的执行顺序排在第二个操作之前。**

2）两个操作之间存在happens-before关系，并不意味着Java平台的具体实现必须要按照happens-before关系指定的顺序来执行。**如果重排序之后的执行结果，与按happens-before关系来执行的结果一致，那么这种重排序并不非法（也就是说，JMM允许这种重排序）**。

上面的1）是JMM**对程序员的承诺**。

从程序员的角度来说，可以这样理解happens-before关系：如果A happens-before B，那么Java内存模型将向程序员保证——A操作的结果将对B可见，且A的执行顺序排在B之前。注意，这只是Java内存模型向程序员做出的保证！

上面的2）是JMM**对编译器和处理器重排序的约束原则**。


## 运行时数据区域

![image-20200227134247242](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200227134247242.png)

#### 程序计数器

记录正在执行的虚拟机字节码指令的地址（如果正在执行的本地方法则为空）

#### Java虚拟机栈

每个Java方法在执行的同时会创建一个帧用于存储局部变量表、操作数栈、常量池引用等信息。从调用方法直至执行完成的过程，对应着一个栈帧在Java虚拟机中入栈和出栈的过程。

![image-20200227134926331](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200227134926331.png)

可以通过-Xss这个虚拟机参数来指定每个线程的Java虚拟机栈内存大小，在JDK1.4默认为256k，而在JDK1.5+默认为1M：

```java
java -Xss2M HackTheJava
```

该区域可能会排出如下异常：

- 当进程请求的栈深度超过最大值，会抛出StackOverFlowErro异常；
- 栈进行动态扩展时如果无法申请到足够内存，会抛出OutOfMemoryError异常。

#### 本地方法栈

本地方法栈与Java虚拟机栈类似，它们之间的区别只不过是本地方法栈为本地方法服务。

本地方法一般是用其他语言（C、C++、汇编）编写的，并且被编译为基于本机硬件和操作系统的程序，对待这些方法需要特别处理。

![image-20200227135257943](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200227135257943.png)

#### 堆

所有对象都在这里分配内存，是垃圾收集的主要区域（“GC堆”）。

现代的垃圾收集器基本都是采用分代收集算法，其主要思想是针对不同类型的对象采取不同的垃圾回收算法。可以将堆分成两块：

- 新生代（Young Generation）
- 老年代（Old Generation）

堆不需要连续内存，并且剋动态地增加其内存，增加失败会抛出OutOfMemoryError异常。

可以通过-Xms和-Xmx这两个虚拟机参数来指定一个程序的堆内存大小，-Xms设置初始值，-Xmx设置最大值。

```java
java -Xms1M -Xmx2M HackTheJava
```

#### 方法区

**用于存放已被加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。**

和堆一样不需要连续的内存，并且可以动态扩展，扩展失败一样会抛出OutOfMemoryError异常。

对这块区域进行垃圾回收的主要目标是对常量池的回收和对类的卸载，但是一般比较难实现。

HotSpot虚拟机把它当成永久代来进行垃圾回收。但很难确定永久代的大小，因为它受到很多因素影响，并且每次Full GC之后永久代的大小都会改变，所以经常会抛出OutOfMemoryError异常。为了更容易管理方法区，从JDK1.8开始，移除**永久代，并把方法区移到元空间，它位于本地内存中，而不是虚拟机内存中。**

**方法区是一个JVM规范，永久代与元空间都是其一种实现方式**。在JDK1.8之后，原来永久代的数据被分到了堆和元空间中。

#### 运行时常量池

运行时常量池是方法区的一部分。

Class文件中的常量池（编译器生成的字面量和符号引用）会在类加载后被放入这个区域。

除了在编译器生成的常量，还允许动态生成，例如String类的intern()。

#### 直接内存

在JDK1.4中新引入了NIO类，它可以使用Native函数库直接分配堆外内存，然后通过Java堆里的DirectByteBuffer对象作为这块内存的引用进行操作。这样能在一些场景中显著提高性能，因为避免了在对内存和堆外内存来回拷贝数据。

## 垃圾收集

垃圾收集主要是针对堆和方法区进行。程序计数器、虚拟机栈和本地方法栈这三个区域属于线程私有的，只存在线程的生命周期内，线程结束之后就会消失，因此不需要对这三个区域进行垃圾回收。

### 判断一个对象是否可被回收

#### 引用计数法

为对象添加一个引用计数器，当对象增加一个引用时计数器加1，引用失效时计数器减1。引用计数为0的对象可被回收。

在两个对象出现循环引用的情况下，此时引用计数器永远不为0，导致无法对它们进行回收。正式因为循环引用的存在，因此Java虚拟机不使用引用计数算法。

```java
public class Test {
    public Object instance = null;
    public static void main(String[] args) {
        Test a = new Test();
        Test b = new Test();
        a.instance = b;
        b.instance = a;
        // ...
    }
}
```

在上述代码中，a与b引用的对象实例互相持有了对象的引用，因此当我们把对a对象与b对象的引用去除之后，由于两个对象还存在互相之间的引用，导致两个Test对象无法被回收。

#### 可达性分析算法

以**GC Roots为起始点**进行搜索，可达的对象都是存活的，不可达的对象可被回收。

Java虚拟机使用该算法来判断对象是否可被回收，GC Roots一般包含以下内容：

- 虚拟机栈中**局部变量表**中引用的对象
- 本地方法栈中**JNI中引用的对象**
- 方法区中类**静态属性引用的对象**
- 方法区中的**常量引用的对象**

### 方法区的回收

因为方法区主要存放永久代对象，而永久代对象的回收率比新生代低很多，所有在方法区上进行回收性价比不高。

主要是**对常量池的回收和对类的卸载**。

为了避免内存溢出，在大量使用反射和动态代理的场景都需要虚拟机具备类卸载功能。

类卸载条件很多，需要满足以下三个条件，并且满足了条件也不一定会被卸载：

- 该**类所有的实例都已经回收**，此时堆中不存在该类的任何实例。
- 加载该类的**ClassLoader已经被回收**。
- 该类对应的**Class对象没有在任何地方被引用**，也就无法在任何地方通过反射访问该类方法。

#### finalize()

类似C++的析构函数，用于关闭外部资源。但是try-finally等方式可以做的更好，并且该方法运行代价很高，不确定性大，无法保证各个对象的调用顺序，因此最好不要用。

当一个对象可被回收时，如果需要执行该对象的finalize()方法，那么就有可能在该方法中让对象重新被引用，从而实现自救。自救只能进行一次，如果回收对象之前调用了finalize()方法自救，后面回收时不会再调用该方法。

### 引用类型

无论是通过引用计数算法判断对象的引用数量，还是通过可达性分析算法判断对象是否可达，淡定对象是否可被回收都与引用有关。Java提供了四种强度不同的引用类型。

#### 强引用

被强引用关联的对象永远不会被回收。默认模式，内存泄漏的主要原因之一

使用new一个新对象的方式来创建强引用。

```java
Object obj = new Object();
```

#### 软引用

被软引用关联的对象只有在内存不够的情况下才会被回收，内存足够时不收。

使用SoftReference类来创建软引用。

```java
Object obj = new Object();
SoftReference<Object> sf = new SoftReference<Object>(obj);
obj = null; // 使对象只被软引用关联
```

通常用在对内存敏感的区域，比如高速缓存

![image-20200530231501876](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200530231501876.png)

#### 弱引用

被弱引用关联的对象**一定会被回收**，也就是说它只能存活到下一次垃圾回收发生之前。

使用WeakReference来创建弱引用

```java
Object obj = new Object();
WeakReference<Object> wf = new WeakReference<Object>(obj);
obj = null;
```

**WeakHashMap**

weak key

```java
public static void main(String[] args) {
        WeakHashMap<Integer, String> map = new WeakHashMap<>();

        Integer key = new Integer(1);
        String value = "11";

        map.put(key, value);
        System.out.println(map);
        
        System.gc();
        System.out.println(map); // 此处被回收
    }
```

#### 虚引用

又称为幽灵引用或者幻影引用，一个对象是否有虚引用的存在，不会对其生存时间造成影响，也无法通过虚引用得到一个对象。并不决定回收。

为一个对象设置虚引用的唯一目的是能在这个对象被回收时受到一个系统通知。/监控

使用虚引用的get只会返回null。

Java允许使用finalize方法在收集器将对象回收时做一些额外工作。

使用PhantomRerence来创建虚引用。

```java
Object obj = new Object();
PhantomReference<Object> pf = new PhantomReference<Object>(obj);
obj = null;
```

**引用队列**

![image-20200530232615023](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200530232615023.png)

gc之后，weak引用的对象进入引用队列。虚引用同：

![image-20200530232841956](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200530232841956.png)

#### 总结

![image-20200530233036161](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200530233036161.png)



### 垃圾回收算法

4种。尚硅谷说是引用计数、复制、标记清楚、标记整理。（博客，分代收集）

垃圾回收器和垃圾收集算法的关系？

GC算法是内存回收的方法论，垃圾收集器是算法的落地实现。

目前没有完美的、万能的垃圾收起，针对具体应用使用不同的垃圾收集器。

1. 什么样的对象需要回收？

   - 根搜索算法，内存中每个对象看作一个节点，定义一些对象为根节点“GC Roots”。JVM会起一个线程从所有的GC Roots开始往下遍历，当遍历完之后如果发现一些对象不可达，那么就认为这些对象已经不可用，需要被回收

   - 简单来说，某个对象到GC Roots没有引用链，那么这个对象不可用，需要回收

2. 疑问

   1. 为什么它们可以作为GC roots？因为这些东西被认为是在被使用，JVM设计者命令JVM将它们作为GC Roots
   2. GC Roots是什么？GC Roots是一个统称，是所有可以用作“根集可达性算法”中的根
   3. 它们存在在哪里？没有所谓的存放位置，它们都是字节码加载运行过程中加入JVM中的一些普通对象，只不过被认为是GC Roots
   4. 是引用还是对象？引用就是对象，因为对于Java语言来说单独的引用没有意义
   5. 是放在堆里还是方法栈？都有，只要他被认为是被使用的。但堆中开辟的对象都是在其他位置有一个引用的
   6. 最后虚拟机会回收GC Roots吗？不会但不保证绝对，就HotSpot版的虚拟机是不会的

3. 可作为GC Roots的对象？

   - Class——由系统类加载器加载的对象，这些类不能够被回收，它们可以以静态字段的方式保存持有其他对象。注，通过用户自定义的类加载器加载的类，除非相应的java.lang.Class实例以其他的某种方式成为roots，否则它们并不是roots

   - Thread——活着的线程

   - Stack Local——Java方法的local变量或参数

   - JNI Local——JNI方法的local变量或参数

   - JNI Global——全局JNI引用

   - Monitor Used——用于同步的监控对象

   - Held by JVM——用于JVM特殊目的的由GC保留的对象，与JVM实现相关。可能已知的一些类型：系统类加载器、一些JVM知道的重要的异常类、一些用于处理异常的预分配对象以及一些自定义的类加载器等

   - ///////////////// 书里//////////////////

   - 虚拟机栈（栈帧中的本地变量表）中引用的对象

     - a是栈帧中的本地变量，当a=null时，由于此时a充当了GC Roots的作用，a与原来指向的实例断连，所以对象不再可用，成为垃圾

       ```java
       public class Test {
           public static  void main(String[] args) {
           Test a = new Test();
           a = null;
           }
       }
       ```

       

   - 方法区中类静态属性引用的对象

     - 当栈帧中的本地变量a=null时，由于a原来指向的对象与GC Roots（a）断连，所以a原来指向的对象会被回收，由于我们给s赋值了变量的引用，s在此时时类静态属性引用，可以作为GC Roots，它指向的对象依然可用。

       ```java
       public  class Test {
           public  static Test s;
           public static  void main(String[] args) {
           Test a = new Test();
           a.s = new Test();
           a = null;
           }
       }
       ```

   - 方法区中常量引用的对象

     - 常量s指向的对象不会被回收

       ```
       public  class Test {
       public  static  final Test s = new Test();
               public static void main(String[] args) {
               Test a = new Test();
               a = null;
               }
       }
       ```

   - 本地方法栈中JNI（一般说的Native方法）引用的对象

     - 本地方法即Java调用非Java代码的接口，该方法并非Java实现，Java通过JNI调用本地方法，而本地方法时以库文件的形式存放的（win下面是dll文件，unix下是so文件）。通过调用本地的哭文件的内部方法，使Java可以实现和本地机器的紧密联系，调用系统级的各接口方法。

     - 当调用Java方法时，虚拟机会创建一个栈帧并压入虚拟机栈，而当它调用本地方法时，虚拟机会保持虚拟机栈不变，虚拟机只是简单地动态连接并直接调用指定的本地方法。

     - 而Java调用本地方法时，JC（例，本地方法栈中JNI的对象引用）会被本地方法栈压入栈中，因此只会在此本地方法执行完后才会释放。

       ```java
       JNIEXPORT void JNICALL Java_com_pecuyu_jnirefdemo_MainActivity_newStringNative(JNIEnv *env, jobject instance，jstring jmsg) {
       ...
          // 缓存String的class
          jclass jc = (*env)->FindClass(env, STRING_PATH);
       
       ```

       

```
1.System Class ----------Class loaded by bootstrap/system class loader. For example, everything from the rt.jar like java.util.* . 
2.JNI Local ----------Local variable in native code, such as user defined JNI code or JVM internal code. 
3.JNI Global ----------Global variable in native code, such as user defined JNI code or JVM internal code. 
4.Thread Block ----------Object referred to from a currently active thread block. Thread ----------A started, but not stopped, thread. 
5.Busy Monitor ----------Everything that has called wait() or notify() or that is synchronized. For example, by calling synchronized(Object) or by entering a synchronized method. Static method means class, non-static method means object. 
6.Java Local ----------Local variable. For example, input parameters or locally created objects of methods that are still in the stack of a thread. 
7.Native Stack ----------In or out parameters in native code, such as user defined JNI code or JVM internal code. This is often the case as many methods have native parts and the objects handled as method parameters become GC roots. For example, parameters used for file/network I/O methods or reflection. 7.Finalizable ----------An object which is in a queue awaiting its finalizer to be run. 
8.Unfinalized ----------An object which has a finalize method, but has not been finalized and is not yet on the finalizer queue. 
9.Unreachable ----------An object which is unreachable from any other root, but has been marked as a root by MAT to retain objects which otherwise would not be included in the analysis.
10.Java Stack Frame ----------A Java stack frame, holding local variables. Only generated when the dump is parsed with the preference set to treat Java stack frames as objects. 
11.Unknown ----------An object of unknown root type. Some dumps, such as IBM Portable Heap Dump files, do not have root information. For these dumps the MAT parser marks objects which are have no inbound references or are unreachable from any other root as roots of this type. This ensures that MAT retains all the objects in the dump.
```

##### 标记-清除

![image-20200227145607078](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200227145607078.png)

在标记阶段，程序会检查每个对象是否为活动对象，如果是活动对象，则程序会在对象头部打上标记。

在清除节点，会进行对象回收并取消标志位，另外，还会判断回收后的分块与前一个空闲分块是否连续，若连续，会合并这两个分块。回收对象就是把对象作为分块，连接到被称为“空闲链表”的单向链表，之后进行分配时只需要遍历这个空闲链表，就可以找到分块。

在分配时，程序会搜索空闲链表寻找空间大于等于新对象大小size的块block。如果它找到的块等于size，会直接返回这个分块；如果找到的块大于size，会将块分割成大小为size与（block-size）的两部分，返回大小为size的分块，并把大小为(block-size)的块返回给空闲链表。

不足：

- 标记和清除过程效率都不高；
- 会产生大量不连续的内存碎片，导致无法给大对象分配内存。

##### 标记-整理

![image-20200227150154743](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200227150154743.png)

让所有存活的对象都向一端移动，然后清理掉端边界以外的内存。

优点：不会产生内存碎片

缺点：需要移动大量对象，处理效率比较低 

##### 复制

![image-20200227150244785](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200227150244785.png)

将内存划分为大小相等的块，每次只使用其中一块，当这一块内存用完了就将还存活的对象复制到另一块上面，然后把使用过的内存空间进行一次清理。

主要不足是只使用了内存的一半。

现在的商业虚拟机都采用这种收集算法回收**新生代**，但是并不是划分为大小相等的两块，而是一块较大的Eden空间和两块较小的Survivor空间，每次使用Eden和其中一块Survivor。在回收时，将Eden和Survivor中还存活着的对象全部复制到另一块Survivor上，最后清理Eden和使用过的那一块Survivor。

HotSpot虚拟机的Eden和Survivor大小比例默认为8：1，保证了内存的利用率达到90%。如果每次回收有多余10%的对象存活，那么一块Survivor就不够用了，此时需要依赖于老年代进行空间分配担保，也就是借用老年代的空间存储放不下的对象。

##### 分代收集

现在的商业虚拟机采用分代收集算法，它根据对象存活周期将内存划分为几块，不同块采用适当的收集算法。

一般将堆氛围新生代和老年代：

- 新生代：复制算法
- 老年代：标记-清除算法或者标记-整理算法

### 垃圾收集器

垃圾回收的方式：四种

- 串行：Serial，STW单线程设计，会暂停其他线程
- 并行：Parallel，多个垃圾手机线程并行工作，也会暂停用户线程，用于科学计算，可以容忍STW
- 并发：CMS，用户线程和垃圾线程同时执行，不一定是并行，可能交替执行，适用于响应时间有要求的，暂停时间较短，互联网公司
- G1：Java9的默认，将堆内存分割成不同的区域，然后并发的对各区域收集

![image-20200531001441103](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200531001441103.png)

![image-20200531001448565](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200531001448565.png)

![image-20200531001004188](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200531001004188.png)

![image-20200227151231586](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200227151231586.png)

![image-20200531002804012](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200531002804012.png)

以上是HotSpot虚拟机中的7个垃圾收集器，连线表示收集器可以配合使用。

- 单线程与多线程：单线程指的是垃圾收集器只使用一个线程，而多线程使用多个线程。
- 串行于并行：串行指的是垃圾收集器与用户程序交替执行，这意味着在执行垃圾收集的时候需要停顿用户程序（Stop the world）；并行指的是垃圾收集器和用户程序同时执行。除了CMS和、ParNew、G1之外，其他垃圾收集器都是以串行的方式执行？？？。

##### 如何查看默认的垃圾收集器？如何配置？理解？

Java -XX:PrintCommandLineFlags -version

-XX:UseSerialGC

-XX:UseParallelGC。java8默认并行收集器

##### 一些说明

一些参数：

- DefNew：Default New Generation
- Tenured：Old
- ParNew：Parallel New Generation
- PSYoungGen：Parallel Scavenge
- ParOldGen：Parallel Old Generation

Client与Server：

- Client基本不用，32位Windows默认Client的JVM
- 32位其他操作系统，2G内存同游有两个CPu以上使用Server
- 64位only Server

##### 新生代

- 串行GC：Serial
- 并行GC：ParNew，只新生代用并行
- 并行回收GC：Parallel/Parallel Scavenge， 配在新生代，老年代也会被激活使用并行

##### Serial收集器

![image-20200227151849177](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200227151849177.png)

Serial翻译为串行，也就是说它以**串行**的方式执行。

它是单线程的收集器，只会使用**一个线程**进行垃圾收集工作，需要暂停其他所有工作线程直到收集完。

它的优点是**简单高效**，在**单个CPU环境**下，由于没有线程交互的开销，因此有最高的单线程**收集效**率。

它是Client场景下默认的新生代收集器，因为在该场景下内存一般来说不会很大。它收集一两百兆垃圾的停顿时间可以控制在100多ms内，只要不是太频繁，这点停顿时间是可以接受的。

JVM参数：-XX：+UseSerialGC，开启后新生代用Serial，老年代用Serial Old



##### ParNew收集器

![image-20200227152145195](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200227152145195.png)

它是Serial收集器的多线程版本。它是**Server场景下默认的新生代收集器**，除了性能原因外，主要是因为除了Serial收集器，只有它能与**CMS收集器配合**使用。ParNew和Serial Old不再推荐使用。

参数：-XX：UseParNewGC，开启后，新生代ParNew，老年代Serial Old

备注：-XX：ParallelGCThreads，限制线程数量，默认与CPU树木相同

##### Parallel Scavenge收集器

新生代垃圾回收器，吞吐量优先收集器。**串行收集器在新生代和老年代的并行化**。

与ParNew的不同，它关注**可控制的吞吐量**，有**自适应调节策略**，虚拟机根据当前系统的运行情况收集性能监控信息，动态调整参数以提供最合适的停顿时间或最大的吞吐量。

参数：-XX:UseParallelGC和-XX:UseParallelOldGC，二者互相激活

备注：-XX:ParallelGCThreads=n，表示启动多少的GC线程，cpu>8，N=5或8，cpu小于8，等于Cpu

下面废话

与ParNew一样是多线程收集器其他收集器目标是尽可能缩短垃圾收集时用户线程的停顿时间，而它的目标是达到一个可控制的吞吐量，因此它被称为“吞吐量优先“收集器。这里的吞吐量指CPU用于运行用户程序的时间占总时间的比值。

停顿时间越短越适合需要与用户交互的程序，良好的响应速度能提升用户体验。而高吞吐量可以高效率地利用CPU时间，尽快完成程序的运算任务，适合在后台运算而不需要太多交互的任务。

缩短停顿时间是以牺牲吞吐量和新生代空间来换取的：新生代空间变小，垃圾回收变得频繁，导致吞吐量下降。

可以通过一个开关参数打开GC自适应调节策略（GC Ergonomics），就不需要手工指定新生代的大小（-Xmn）、Eden和Survivor区的比例、晋升老年代对象年龄等细节参数了。虚拟机会根据当前系统的运行情况收集性能监控信息，动态调整这些参数以提供最合适的停顿时间或者最大的吞吐量。

##### 老年代

- 串行GC：
- 并行GC：Parallel Old
- 并发标记清除GC/CMS：

##### Serial Old收集器

![image-20200227152833469](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200227152833469.png)

是Serial收集器的老年代版本，也是给Client场景下的虚拟机使用。如果用在Server场景使用，它有两大用途：

- 在JDK1.5以及之前版本（Parallel Old诞生以前）中与Parallel Scavenge收集器搭配使用
- 作为CMS收集器的后备预案，在并发收集发生ConcurrentModeFailure时使用

##### Parallel Old收集器

![image-20200227153030407](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200227153030407.png)

是Parallel Scavenge收集器的老年代版本。1。8默认

参数：-XX:UseParallelOldGC，互相激活

在注重吞吐量以及CPU资源敏感的场合，都可以优先考虑Parallel Scavenge加Parallel Old收集器。

##### CMS收集器

![image-20200227153127100](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200227153127100.png)

CMS（Concurrent Mark Sweep），Mark Sweep指标记-清除算法。以获取最短回收挺短时间位目标，适合互联网网站或者BS系统的服务器上，重视服务器的响应速度，希望停顿时间最短，CMS适合堆内存大、CPU核多的，也是G1出现之前大型应用的首选收集器。

并发指与用户线程一起执行。

参数：-XX:+UseConcMarkSweepGC，开启后自动开启ParNew，异常情况CMS会转为Serial Old做后备。

分以下四个流程：

- 初始标记：仅仅是标记一下GC Roots能直接关联到的对象，速度很快，需要停顿。
- 并发标记：进程GC Roots Tracing过程，它在整个回收过程中耗时最长，不需要停顿，继续标记，标记全部对象
- 重新标记：为了修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，需要停顿。修正变动，二次确认
- 并发清除：不需要停顿，用用户线程并发

优点：

- 在整个过程中耗时最长的并发标记和并发清除过程中，收集器线程都可以与用户线程一起工作，不需要停顿。

缺点：

- 吞吐量低：低停顿时间是以牺牲吞吐量为代价的，导致CPU利用率不够高
- 无法处理浮动垃圾，可能出现Concurrent Mode Failure。浮动垃圾是指并发清除节点由于用户线程继续运行而产生的垃圾，这部分垃圾只能到下一次GC时才能进行回收。由于浮动垃圾的存在，因此需要预留出一部分内存，意味着CMS收集不能像其他收集器那样等待老年代快满的时候再回收。如果预留的内存不够存放浮动垃圾，就会出现Concurrent Mode Failure，这是虚拟机将临时启动Serial Old来提到CMS。
- 标记-清除算法导致空间碎片，往往出现老年代空间剩余，但无法找到足够大的连续空间来分配当前读喜庆，不得不提前触发一次Full GC。

##### G1收集器

-XX:+UseG1GC

G1（Garbage-First）它是一款面向服务端应用的垃圾收集器，在多CPU和大内存的场景下有很好的性能。HotSpot开发团队赋予它的使命是未来可以替换掉CMS收集器。G1是一个有整理内存过程的垃圾收集器，不回产生很多内存碎片，G1的STW更可控，G1在停顿时间上添加了预测机制，用户可以制定期望停顿时间。

在实现高吞吐量的同时，尽可能满足垃圾收集暂停时间的要求，另外：

- 类似于CMS，与用户线程并发执行
- 整理空闲空间更快
- 需要更多的时间来预测GC停顿时间
- 不希望牺牲大量的吞吐性能
- 不需要更大的Java Heap

堆被分为新生代和老年代，其他收集器进行收集的范围都是整个新生代或者老年代，而G1可以直接对新生代和老年代一起回收。

主要改变：Eden、S、Tenured等内存区域不再连续，变成了一个个大小一样的region，每个region从1M到32M不等，一个region有可能属于Eden、S、Tenured。

![image-20200531011438039](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200531011438039.png)

##### G1的底层原理

![image-20200531011711694](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200531011711694.png)



![image-20200227154350621](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200227154350621.png)

G1把堆划分成多个大小相等的独立区域（Region），新生代和老年代不再物理隔离。

![image-20200227154436167](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200227154436167.png)

![image-20200531011759454](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200531011759454.png)

![image-20200531011924317](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200531011924317.png)

通过引入Region概念，从而将原来的一整块内存空间划分成多个的小空间，使得每个小空间可以进行单独进行垃圾回收。这种划分方法带来了很大的灵活性，使得可预测的停顿时间模型成为可能。通过记录每个Region垃圾回收时间以及回收所获得的空间（这两个值是通过过去回收的经验获得），并维护一个优先列表，每次根据允许的收集时间，优先回收价值最大的Region。

每个Region都有一个Remembered Set，用来记录该Region对象的引用对象所在的Region。通过使用Remembered Set，在做可达性分析的时候就可以避免全堆扫描。

![image-20200227155151263](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200227155151263.png)

如果不计算维护Remembered Set的操作，G1收集器的运作大致可以划分为以下几个步骤：

- 初始标记
- 并发标记
- 最终标记：为了修正在并发标记期间因用户程序继续运作而导致标记产生变动的那一部分标记记录，虚拟机将这段时间对象变化记录在线程的Remembered Set Logs里面，最终标记阶段需要把Remembered Set Logs的数据合并到Remembered Set中。这个阶段需要停顿线程，但是可并行执行。
- 筛选回收：首先对各个Region中的回收价值和成本进行排序，根据用户所期望的GC停顿时间来制定回收计划。此阶段其实也可以做到与用户程序一起并发执行，但是因为只回收一部分Region，时间是用户可控制的，而且停顿用户线程将大幅度提高收集效率。

具备如下特点：

- 空间整合：整体来看是基于“标记-整理”算法实现的收集器，从局部（两个Region之间）上来看是基于“复制”算法实现的，这意味着运行期间不会产生内存碎片；
- 可预测的停顿：能让使用者明确地指定在一个长度为M毫秒的时间片段内，消耗在GC上的时间不得超过N毫秒。

![image-20200531012017397](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200531012017397.png)

![image-20200531012114874](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200531012114874.png)



![image-20200531012329964](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200531012329964.png)

![image-20200531012400019](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200531012400019.png)

比CMS的优势：

- 内存碎片减少
- 精确控制停顿

##### 垃圾收集器的选择

![image-20200531010544537](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200531010544537.png)

![image-20200531010605495](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200531010605495.png)

## 内存分配与回收策略

#### Minor GC和Full GC

- Minor GC：回收新生代，因为新生代对象存活时间很短，因此Minor GC会频繁执行，执行的速度一般也比较快。
- Full GC：回收老年代和新生代，老年代对象其存活时间长，因此Full GC很少执行，执行速度会比Minor GC慢很多。

#### 内存分配策略

##### 对象优先在Eden分配

大多数情况下，对象在新生代Eden上分配，当Eden空间不够时，发起Minor GC。

##### 大对象直接进入老年代

大对象是指需要连续内存空间的对象，最典型的大对象是那种很长的字符串以及数组。

经常出现大对象会提前出发垃圾收集以获得足够的连续空间分配给大对象。

-XX:PretenureSizeThreshold，大于此值的对象直接在老年代分配，避免在Eden和Survivor之间的大量内存复制。

##### 长期存活的对象进入老年代

为对象定义年龄计数器，对象在Eden出生并经过Minor GC仍然存活，将移动到Survivor中，年龄就增加1岁，增加到一定年龄则移动到老年代中。

-XX:MaxTenuringThreshold用来定义年龄的阈值。

##### 动态对象年龄判定

虚拟机并不是永远要求对象的年龄必须达到MaxTenuringThreshold才能晋升老年代，如果在Survivor中相同年龄所有对象大小的总和大于Survivor空间的一般，则年龄大于等于该年龄的对象可以直接进入老年代，无需等到MaxTenuringThreshold中要求的年龄。

##### 空间分配担保

在发生Minor GC之前，虚拟机首先检查老年代最大可用的连续空间是否大于新生代所有对象总空间，如果条件成立的话，那么Minor GC可以确认是安全的。

如果不成立的话虚拟机会查看HandlePromotionFailure的值是否允许担保失败，如果允许那么就会继续检查老年代最大可用的连续空间是否大于历次晋升到老年代对象的平均大小，如果大于，将尝试着进行一次Minor GC；如果小于，或者HandlePromotionFailure的值不允许冒险，那额就要进行一次Full GC。

#### Full GC的触发条件

对于Minor GC，其触发条件非常简单，当Eden空间满时，就将触发一次Minor GC。而Full GC则相对复杂，有以下条件：

1. 调用System.gc()

   只是建议虚拟机执行Full GC，但是虚拟机不一定真正去执行，不建议使用，而是让虚拟机来管理内存。

2. 老年代空间不足

   老年代空间不足的常见场景为前文所讲的大对象直接进入老年代、长期存活的对象进入老年代等。

   为了避免以上原因引起的Full GC，应当尽量不要创建过大的对象以及数组。除此之外，可以通过-Xmn虚拟机参数调大新生代的大小，让对象尽量在新生代被回收掉，不进入老年代。还可以通过-XX:MaxTenuringThreshold调大对象进入老年代的年龄，让对象在新生代多存活一段时间。

3. 空间分配担保失败

   使用复制算法的Minor GC需要老年代的内存空间作担保，如果担保失败会执行一次Full GC。

4. JDK1.7及以前的永久代空间不足

   在JDK1.7及以前，HotSpot虚拟机中的方法区是用永久代实现的，永久代中存放的一些Class的信息、常量、静态变量等数据。

   当系统中要加载的类、反射的类和调用的方法较多时，永久代可能会被占满，在未配置为采用CMS GC的情况下也会执行Full GC。如果经过Full GC仍然回收不了，那么虚拟机会抛出java.lang.OutOfMemoryError。

   为避免以上原因引起的Full GC，可采用的方法为增大永久代空间或转为使用CMS GC。

5. Concurrent Mode Failure

   执行CMS GC的过程中同时有对象要放入老年代，而此时老年代空间不足（可能是GC过程中浮动垃圾过多导致暂时性的空间不足），便会报Concurrent Mode Failure错误，并触发Full GC。

## 类加载机制

类是在运行期间第一次使用时动态加载的，而不是一次性加载所有类。因为如果一次性加载，那么会占用很多内存。

#### 类的生命周期

![image-20200228142745180](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200228142745180.png)

包括以下7个阶段：

- **加载（Loading）**
- **验证（Verification）**
- **准备（Preparation）**
- **解析（Resolution）**
- **初始化（Initialization）**
- 使用（Using）
- 卸载（Unloading）

#### 类加载过程

包含了加载、验证、准备、解析和初始化这5个阶段。

##### 加载

加载时类加载的一个阶段，注意不要混淆。

加载过程完成以下三件事：

- 通过类的完全限定名称获取定义该类的二进制字节流。
- 通过该字节流表示的静态存储结构转换为方法区的运行时存储结构。
- 在内存中生成一个代表该类的Class对象，作为方法区中该类各种数据的访问入口。

其中二进制字节流可以从以下方式中获取：

- 从ZIP包读取，成为JAR、EAR、WAR格式的基础。
- 从网络中获取，最典型的应用时Applet。
- 运行时计算生成，例如动态代理计数，在java.lang.reflect.Proxy使用ProxyGenerator.generateProxyClass的代理类的二进制字节流。
- 由其他文件生成，例如由JSP文件生成对应的Class类。

##### 验证

确保Class文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全。

##### 准备

类变量是被static修饰的变量，准备阶段为类变量分配内存并设置初始值，使用的是方法区的内存。

实例变量不会在这阶段分配内存，它会在对象实例化时随着对象一起被分配在堆中。应该注意到，实例化不是类加载的一个过程，类加载发生在所有实例化操作之前，并且类加载只进行一次，实例化可以进行多次。

初始值一般为0值，例如下面的变量value被初始化为0，而不是123.

```java
public static int value = 123;
```

如果类变量是常量，那么它将初始化为表达式所定义的值而不是0。例如下面的常量value被初始化为123而不是0。

```java
public static final int value = 123;
```

##### 解析

将常量池的符号引用替换为直接引用的过程。

其中解析过程在某些情况下可以在初始化阶段之后再开始，这是为了之后java的动态绑定。

##### 初始化

初始化阶段真正开始执行类中定义的Java程序代码。初始化阶段是虚拟机执行类构造器<clinit>()方法的过程。在准备阶段，类变量已经赋过一次系统要求的初始值，而在初始化阶段，根据程序员通过程序制定的主管计划区初始化类变量和其他资源。

<clinit>()是由编译器自动收集类中所有类变量的赋值动作和静态语句块中的语句合并产生的，编译器收集的顺序由语句在源文件中出现的顺序决定。特别注意的是，静态语句块只能访问到定义在它之前的类变量，定义在它之后的类变量只能赋值，不能访问。例如：

```java
public class Test {
    static {
        i = 0;  //可以
        System.out.println(i) // 编译器提示“非法向前引用”
    }
    static int i = 1;
}
```

由于父类的<clinit>()方法先执行，也就意味着父类中定义的静态语句块的执行要优先于子类。例如以下代码：

```java
static class Parent {
    public static int A = 1;
    static {
        A = 2;
    }
}

static class Sub extends Parent {
    public static int B = A;
}

public static void main(String[] args) {
    System.out.println(sub.B); // 2
}
```

接口中不可以使用静态语句块，但仍然有类变量初始化的赋值操作，因此接口与类一样都会生成<clinit>()方法。但接口与类不同的是，执行接口的<clinit>()方法不需要先执行父接口的<clinit>()方法。只有当父接口中定义的变量使用时，父接口才会初始化。另外，接口的实现类在初始化时也一样不会执行接口的<clinit>()方法。

虚拟机会保证一个类的<clinit>()方法在多线程环境下被正确的加锁和同步，如果多个线程同时初始化一个类，只会有一个线程执行这个类的<clinit>()方法，其他线程都会阻塞等待，知道活动现场执行<clinit>()方法完毕。如果在一个类的<clinit>()方法中有耗时的操作，就可能造成多个线程阻塞，在实际过程中此种阻塞很隐蔽。

#### 类初始化时机

##### 主动引用

虚拟机规范中并没有强制约束何时进行加载，但是规范严格规定了又且只有下列五种情况必须对类进行初始化（加载、验证、准备都会随之发生）：

- 遇到new、getstaic、putstatic、invokestatic这四条字节码指令时，如果类没有进行过初始化，则必须先触发其初始化。最常见的生成这4条指令的场景是：使用new关键字实例化对象的时候；读取或设置一个类的静态字段（被final修饰、已在编译期把结果放入常量池的静态字段除外）的时候；以及调用一个类的静态方法的时候。
- 使用java.lang.reflect包的方法对类进行反射调用的时候，如果没有进行初始化，则需要先触发其初始化。
- 当初始化一个类的时候，如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化。
- 当虚拟机启动时，用户需要指定一个要执行的主类（包含main()方法的那个类），虚拟机会先初始化这个主类；
- 当使用JDK1.7的动态语言支持时，如果一个java.lang.invoke.MethodHandle实例最后的解析结果为REF_getStatic，REF_putStatic，REF_invokeStatic的方法句柄，并且这个方法句柄所对应的类没有进行过初始化，则需要先触发其初始化。

##### 被动引用

以上5种场景中的行为称为对一个类进行主动引用。除此之外，所有引用类的方式都不会触发初始化，称为被动引用。被动引用的常见例子包括：

- 通过子类引用父类的静态字段，不会导致子类初始化。

  ```java
  System.out.println(SubClass.value); // value 字段在SuperClass中
  ```

- 通过数组定义来引用类，不会触发此类的初始化。该过程对数组类进行初始化，数组是一个由虚拟机自动生成的、直接继承自Object子类，其中包含了数组的属性和方法。

  ```java
  SuperClass[] sca = new SuperClass[10];
  ```

- 常量在编译阶段会存入调用类的常量池中，本质上并没有直接引用到定义常量的类，因此不会触发定义常量的类的初始化。

  ```java
  System.out.println(ConstClass.HELLOWORLD);
  ```

#### 类与类加载器

两个类相等，需要类本身相等，并且使用同一个类加载器进行加载。这是因为每一个类加载器都拥有一个独立的类名称空间。

这里的相等，包括类的Class对象的equals()方法、isAssignableFrom()方法、isInstance()方法的返回结果为true，也包括使用instanceof关键字做对象所属关系判定结果为true。

#### 类加载器分类

从Java虚拟机的角度来讲，只存在以下两种不同的类加载器：

- 启动类加载器（Bootstrap ClassLoader），使用C++实现，是虚拟机自身的一部分；
- 所有其他类的加载器，使用Java实现，独立于虚拟机，继承自抽象类java.lang.ClassLoader

从Java开发人员的角度看，类加载器可以划分为：

- 启动类加载器（Bootstrap ClassLoader）此类加载器负责将存放在<JRE_HOME>\lib目录中的，或者被-Xbootclasspath参数制定的路径中的，并且是虚拟机识别的（仅按照文件名识别，如rt.jar，名字不符合的类库即使放在lib目录中也不会被加载）类库加载到虚拟机内存中。启动类加载器无法被Java程序直接引用，用户在编写自定义类加载器时，如果需要把加载请求委派给启动类加载器，直接使用null代替即可。
- 扩展类加载器（Extension ClassLoader）这个类加载器是由ExtClassLoader（sun.misc.Launcher$ExtClassLoader）实现的。它负责将<JAVA_HOME>/lib/ext或者被java.ext.dir系统变量所制定路径中的所有类库加载到内存中，开发者可以直接使用扩展类加载器；
- 应用程序类加载器（Application ClassLoader）这个类加载器是由AppClassLoader（sun.misc.Launcher$AppClassLoader）实现的。由于这个类加载器是ClassLoader中的getSystemClassLoader()方法的返回值，因此一般称为系统类加载器。它负责加载用户类路径（ClassPath）上所指定的类库，开发者可以直接使用这个类加载器，如果应用程序中没有自定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器。

#### 双亲委派模型

应用程序是由三种类加载器互相配合从而实现类加载，除此之外还可以加入自己定义的类加载器。

下图展示了类加载器之间的层次关系，称为双亲委派模型（Parents Delegation Model）。该模型要求除了顶层的启动类加载器外，其他的类加载器都要有自己的父类加载器。这里的父子关系一般通过组合关系（Composition）来实现，而不是继承关系。

![image-20200228225631231](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200228225631231.png)

##### 工作过程

一个类加载器首先将类加载器请求转发到父类加载器，只有当父类加载器无法完成时才尝试自己加载。

##### 好处

使得Java类随着它的类加载器一起具有一种带有优先级的层次关系，从而使得基础类得到统一。

例如java.lang.Object存放在rt.jar中，如果编写另一个java.lang.Object并放到ClassPath中，程序可以编译通过。由于双亲委派模型的存在，所以在rt.jar中的Object使用的是启动类加载器，而ClassPath中的Object使用的是应用程序类加载器。rt.jar中的Object优先级更高，那么程序中所有的Object都是这个Object。

##### 实现

以下是抽象类java.lang.ClassLoader的代码片段，其中的loadClass()方法运行过程如下：先检查类是否已经加载过，如果没有则让父类加载器去加载。当父类加载器加载失败时抛出ClassNotFountException，此时尝试自己去加载。

```java
public abstract class ClassLoader {
    private final ClassLoader parent; // 组合形成父子关系
    public Class<?> loadClass(String name) throws ClassNotFoundException {
        return loadClass(name, false);
    }
    
    protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
        synchronized (getClassLoadingLock(name)) {
            // check if the class has already been loaded
            Class<?> c = findLoadedClass(name);
            if(c == null) {
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    
                }
                
                if (c == null) {
                    c = findClass(name);
                }
            }
            
            if(resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
    
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        throw new ClassNotFoundException(name);
    }
}
```

#### 自定义类加载器实现

以下代码中的FileSystemClassLoader是自定义类加载器，继承自java.lang.ClassLoader，用于加载文件系统上的类。它首先根据类的全名在文件系统上查找类的字节码文件(.class文件)，然后读取该文件内容，最后通过defineClass()方法来把这些字节代码转换成java.lang.Class类的实例。

java.lang.ClassLoader的loadClass()实现了双亲委派模型的逻辑，自定义类加载器一般不去重写它，但是需要重写findClass()方法。

```java
public class FileSystemClassLoader extens ClassLoader {
    private String rootDir;
    
    public FileSystemClassLoader(String rootDir) {
        this.rootDir = rootDir;
    }
    
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        byte[] classData = getClassData(name);
        if (classData == null) {
            throw new ClassNotFoundException();
        } else {
            return defineClass(name, classData, 0, classData.length);
        }
    }
    
    private byte[] getClassData(String className) {
        String path = classNameToPath(className);
        try {
            InputStream ins = new FileInputStream(path);
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            int bufferSize = 4096;
            byte[] bufer = new byte[bufferSize];
            int bytesNumRead;
            while ((bytesNumRead = ins.read(bufer)) != -1) {
                baos.write(buffer, 0, bytesNumRead);
            }
            return baos.toByteArray();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }
    
    private String classNameToPath(String className) {
        return rootDir + File.separatorCar + className.replace('.', File.separatorChar) + ".class";
    }
}
```

## JVM调优

### JVM参数类型

#### 标配参数

- java -version
- java -help
- java -showversion

#### X参数

- -Xint：解释执行
- -Xcomp：第一次使用就编译成本地代码
- -Xmixed：混合模式

#### XX参数（重点）

##### Boolean类型

-XX:+或者-某个属性，+表示开启，-表示关闭。

例如：-XX:+PrintGCDetails

##### KV设值类型

-XX:key=value

例如：-XX:MetaspaceSize=1024m\-XX:MaxTenuringThreshold=15

##### jinfo命令

查看java运行信息，先用jps -l找到编号，然后jinfo -flag PrintGCDetails id 查看是否开启PrintGCDetails。

Info -flags id：比较全面的参数

##### 经典参数

-Xms：等价-XX:InitialHeapSize。  1/64内存

-Xmx：等价-XX:MaxHeapSize。 1/4内存

##### 重要命令：查看家底参数

java -XX:+PrintFlagsInitial 查看所有初始化参数。

java -XX:+PrintFlagsFinal 最终参数，修改更新后的4

- 说明：:=为人为修改的。=为默认的

java -XX:PrintCommandLineFlags ...

### 常用参数

注：程序级Runtiome可以获得一些参数信息

- -Xms：等价-XX:InitialHeapSize。  1/64内存默认
- -Xmx：等价-XX:MaxHeapSize。 1/4内存默认
- -Xss：设置单个线程栈的线程，一般默认为512k-1024k，等价于-XX:ThreadStackSize，如果查看时，为0表示系统默认值
- -Xmn：设置新生代大小，默认1/3，老年2/3
- -XX:MetaspaceSize：设置元空间大小，元空间不再虚拟机中，而使用本地内存 ，默认20M左右，潜在的OOM
- -XX:+PrintGCDetails：
  - 输出GC详细日志
  - GC：![image-20200530224821107](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200530224821107.png)
  - FullGC：![image-20200530225052995](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200530225052995.png)
- -XX:SurvivorRatio：8:1:1默认，eden:s1:s2，默认-XX:Survivor=8，如果设4 即4:1:!
- -XX:NewRatio：新生代和老年代，默认2新占1老占2，如果配4，新1老4，与-Xmn共存，以-Xmn为准
- -XX:MaxTenuringThreahold：最大年龄，默认15，最大15，对象头age字段4bit

### OOM-OutOfMemoryError

- Java.lang.StackOverflowError：栈溢出，递归调用

- Java.lang.OOM: Java heap space：堆溢出，大数组

- java.lang.OOM: GC overhead limit exceeded，GC回收时间过长（GC效果不明显），代码问题

  ```java
  int i = 0;
  List<String> list = new ArrayList<>();
  while (true) {
   try/catch   list.add(String.valueOf(++i).intern())
  }  直接内存调小
  ```
- java.lang.OOM: Direct buffer memory，例如NIO可以使用ByteBuffer可以使用allocteDirect，在直接内存分配，不会被回收，本地直接内存可能用光

- java.lang.OOM:unable to create new native thread，高并发，线程太多了
- java.lang.OOM:Metaspace（也就是方法区）

### 生产环境服务器变慢，如何（Linux命令）

#### 整机：top

看cpu和mem，load average

uptime 直接看load average

#### CPU：vmstat

vmstat -n 2 3 2秒采样一次，采3次![image-20200531013456034](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200531013456034.png)

#### 内存：free

![image-20200531013820911](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200531013820911.png)

![image-20200531013837917](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200531013837917.png)

#### 硬盘：df

一般硬盘管够。

df

df -h

#### 磁盘IO：iostat

mysql常见，instate -xdk 2 3

#### 网络IO：ifstat

需要下载该命令。

![image-20200531014211301](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200531014211301.png)

#### CPU高，定位与分析思路

1. 先用top找出CPU占用过高的
2. ps -ef或jps进一步定位， 得到进程编号
3. 定位到具体**线程**或者代码哪一行

![image-20200531014518230](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200531014518230.png)

4. ![image-20200531014626934](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200531014626934.png)

## github

![image-20200531015623558](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200531015623558.png)

### in关键词

![image-20200531015922156](/Users/zhaojiangdong/Library/Application Support/typora-user-images/image-20200531015922156.png)

### stars或fork查找

springboot starts:>=5000

springboot forks:>500

Spingboot forks:100..200 stars:80..100

### awesome加强搜索

awsome 关键字

### 高亮某一行代码

url+#L数字，指出关键行号

url+#L2-L23，高亮多行

### 项目内搜索T

列出文件

快捷键t，按下英文字母t

### 区域搜索用户

Location:

Language: