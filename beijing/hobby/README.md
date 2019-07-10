
## 多线程

### JMM
Java内存模型（Java Memory Model），

[内存可见性参考地址](https://www.cnblogs.com/SacredOdysseyHD/p/8438410.html)

共享数据的访问权限必须定义为private

JVM模型两条规定：

	1、线程对共享变量的所有操作必须在自己的内存中进行，不能直接从主内存中读写
	2、不同线程之间无法直接访问其它线程工作内存中的变量，线程间变量值的传递需要通过主内存来完成



原子性：即一个操作或多个操作要么全执行并且在执行的过程中不会被任何因素打断，要么都不执行。
（与事务一样，他们是一个团队，同生共死。）

Java只保证了基本数据类型的变量和赋值操作才是原子性的（注：在32位的JDK环境下，对64位数据的读取不是原子性操作*，如long、double）。

要实现共享变量的可见性，必须保证两点：

	(1) 线程修改后的共享变量值能够及时从工作内存刷新到主内存中；
	(2) 其他线程能够及时的把共享变量的最新值从主内存更新到自己的工作内存中。

在Java语言层面支持的可见性实现原理方式有synchronize和volatile。

导致共享变量在线程间不可见的原因：

	1、线程的交叉执行；
	2、重排序结合线程的交叉执行；
	3、共享变量更新后的值没有在工作内存与主内存及时更新。

### synchronized
能够实现代码的原子性（同步）和 内存的可见性。【2个特点，原子性+可见性】

JVM对其的两条规定：

	1、线程解锁前，必须把共享变量的最新值刷新到主内存中；
	2、线程枷锁前，将清空工作内存中的共享变量的值，从而使用共享变量时需要从主内存中重新读取新的值。（枷锁与解锁需要是同一把锁）。
	3、线程解锁前对共享线程变量的修改在下次枷锁时对其他线程可见。

线程执行互斥代码的过程：
	
	1、获得互斥锁；
	2、清空工作内存；
	3、从主内存拷贝变量的最新的值到工作内存；
	4、执行代码；
	5、将更改后的共享变量的值刷新到主内存；
	6、释放互斥锁。
### volatile
保证变量的可见性，不能保证变量的符合操作原子性。【1个特点，可见性】

保证volatile修改的变量操作的原子性方法：
	
	1、使用synchronized关键字
	2、使用ReentrantLock锁（java.util.concurrent.locks包下）
	3、使用AtomicInteger（java.util.concurrent.antomic包下）

> synchronized(this){ this.number++; }	

> public synchronized int increase(){...}	//这样造成程序性能更加低效。

AtomicInteger是一个提供原子操作的Integer类，通过线程安全的方式操作加减，因此十分适合高并发情况下的使用。


### Memory Barrier(内存屏障)
public native void loadFence();//内存屏障，禁止 load 操作重排序，即屏障前的load操作不能被重排序到屏障后，屏障后的 load 操作不能被重排序到屏障前

public native void storeFence();//内存屏障，禁止 store 操作重排序，即屏障前的 store 操作不能被重排序到屏障后，屏障后的 store 操作不能被重排序到屏障前

public native void fullFence();//内存屏障，禁止 load、store 操作重排序

[内存屏障理解参考链接](https://blog.csdn.net/caoshangpa/article/details/78853919)

程序在运行时内存实际的访问顺序和程序代码编写的访问顺序不一定一致，这就是内存乱序访问。内存乱序访问行为出现的理由是为了提升程序运行时的性能。内存乱序访问主要发生在两个阶段：

	1、编译时，编译器优化导致内存乱序访问（指令重排）
	2、运行时，多 CPU 间交互引起内存乱序访问

Memory Barrier 能够让 CPU 或编译器在内存访问上有序。一个 Memory Barrier 之前的内存访问操作必定先于其之后的完成。Memory Barrier 包括两类：

	1、编译器Memory Barrier
	2、CPU Memory Barrier
	
编写诸如无锁数据结构，那么 Memory Barrier 还是很有用的

## 内存分配
### 非堆内存
官方说法："Java虚拟机具有一个堆(Heap)，堆是运行时数据区域，所有的实例和数组的内存均是从此处分配。堆是在
Java虚拟机启动时创建的。在JVM中堆之外的内存称之为非堆内存（Non-heap memory）。"

Java主要管理两种类型的内存：堆和非堆

PermGen space：全称是Permanent Generation space，即永久代。就是说是永久保存的区域,用于存放
Class和Meta信息，Class在被Load的时候被放入该区域，GC(Garbage Collection)应该不会对PermGen space
进行清理，所以如果你的APP会LOAD很多CLASS的话，就很可能出现PermGen space错误。

Heap space：存放Instance。

Java Heap分为3个区，Young即新生代，Old即老生代和Permanent。

Young保存刚实例化的对象。当该区被填满时，GC会将对象移到Old区。Permanent区则负责保存反射对象。

堆内存分配
	
	1、JVM初始分配的堆内存由-Xms指定，默认是物理内存的1/64；
	2、JVM最大分配的堆内存由-Xmx指定，默认是物理内存的1/4。
	3、默认空余堆内存小于40%时，JVM就会增大堆直到-Xmx的最大限制；
	4、空余堆内存大于70%时，JVM会减少堆直到-Xms的最小限制。
	5、因此服务器一般设置-Xms、-Xmx 相等以避免在每次GC 后调整堆的大小。
	6、如果-Xmx 不指定或者指定偏小，应用可能会导致java.lang.OutOfMemory错误，此错误来自JVM，不是Throwable的，无法用try…catch捕捉。

非堆内存分配

	1、JVM使用-XX:PermSize设置非堆内存初始值，默认是物理内存的1/64；
	2、由XX:MaxPermSize设置最大非堆内存的大小，默认是物理内存的1/4。还有一说：MaxPermSize缺省值和-server -client选项相关，-server选项下默认MaxPermSize为64m，-client选项下默认MaxPermSize为32m。这个我没有实验。
	3、-XX:MaxPermSize设置过小会导致java.lang.OutOfMemoryError: PermGen space 就是内存益出。
	4、为什么会内存益出：
		a、这一部分内存用于存放Class和Meta的信息，Class在被 Load的时候被放入PermGen space区域，它和存放Instance的Heap区域不同。
		b、GC(Garbage Collection)不会在主程序运行期对PermGen space进行清理，所以如果你的APP会LOAD很多CLASS 的话,就很可能出现PermGen space错误。
	5、这种错误常见在web服务器对JSP进行pre compile的时候。
	
>jmap -heap pid  # 打印pid进程的堆内存分配情况。
	
[非堆内存理解参考链接](https://www.cnblogs.com/lfs2640666960/p/8516916.html)

[JVM——内存结构](https://blog.csdn.net/qq_37141773/article/details/82154440)

JVM内存结构主要有三大块：堆内存、方法区和栈。
	
	1、【堆内存】是JVM中最大的一块由年轻代和老年代组成，而年轻代内存又被分成三部分，Eden空间、From Survivor空间、To Survivor空间,默认情况下年轻代按照8:1:1的比例来分配；
	2、【方法区】存储【类】信息、【常量】、【静态变量】等数据，是线程共享的区域，为与Java堆区分，方法区还有一个别名【Non-Heap(非堆)】； 
	3、【栈】又分为【java虚拟机栈】和【本地方法栈】主要用于方法的执行。

JVM内存按线程数据区划分：
	
	1、线程共享数据区（【堆内存】、【方法区】）：存放数据的地方
	2、线程隔离（私有）的数据区（【程序计数器】、【虚拟机栈】、【本地方法栈】）：（执行的地方）

[JVM中的新生代和老年代（Eden空间、两个Survior空间）](https://blog.csdn.net/jisuanjiguoba/article/details/80156781)

[Java8内存划分参考链接](https://blog.csdn.net/yjp198713/article/details/78759933/)

	1、在 JDK 1.7 和 1.8 中将字符串常量池由永久代转移到堆中；
	2、存放类相关信息的地方也不在heap（堆）中。在元空间里；
	3、在jdk1.8中没有永久代的概念；
	4、metaspace其实由两大部分组成：Klass Metaspace + NoKlass Metaspace 
	5、metaspace主要相关参数
	6、Metaspace的内存分配与管理
	7、为什么要将永久代替换成Metaspace？

### Unsafe类【非常值得研究的一个类】
 sun.misc.Unsafe提供了可以随意查看及修改JVM中运行时的数据结构，尽管这些功能在JAVA开发本身是不适用的，
 Unsafe是一个用于研究学习HotSpot虚拟机非常棒的工具，因为它不需要调用C++代码，或者需要创建即时分析的工具。

[Unsafe 相关整理参考链接](https://www.jianshu.com/p/2e5b92d0962e)
