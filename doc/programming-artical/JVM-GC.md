# 概述

Java 相比 C/C++ 最显著的特点便是引入了GC机制，它解决了 C/C++ 最令人头疼的内存管理问题，让程序员专注于程序本身，不用关心内存回收这些恼人的问题。GC 真正让程序员的生产力得到了释放，但是程序员很难感知到它的存在，这就好比，我们吃完饭后在桌上放下餐盘即走，服务员会替你收拾好这些餐盘，你不会关心服务员什么时候来收，怎么收。

# GC理论

垃圾回收收集方法，GC回收器，JVM内存区域等。

## JVM内存区域

![JVM](/img/jvm.png)

虚拟机栈：描述的是方法执行时的内存模型。线程私有，生命周期与线程相同。主要保存执行方法时的局部变量表、操作数栈、动态连接和方法返回地址等信息。方法执行时入栈，方法执行完出栈，出栈就相当于清空了数据，入栈出栈的时机很明确，所以这块区域**不需要进行 GC**。

本地方法栈：与虚拟机栈功能非常类似，主要区别在于虚拟机栈为虚拟机执行 Java 方法时服务，而本地方法栈为虚拟机执行本地方法时服务的。这块区域也**不需要进行 GC**。

> 本地方法(Native Method)：简单地讲，一个Native Method就是一个java调用非java代码的接口。如一个java方法的实现由非java语言实现，比如C。

程序计数器：程序计数器的主要作用是记录线程运行时的状态，方便线程被唤醒时能从上一次被挂起时的状态继续执行，需要注意的是，程序计数器是**唯一一个**在 Java 虚拟机规范中没有规定任何 OOM 情况的区域，所以这块区域也**不需要进行 GC**。

本地内存：即堆外内存，包含元空间和直接内存。注意到上图中 Java 8 和 Java 8 之前的 JVM 内存区域的区别了吗，在 Java 8 之前有个**永久代**的概念，主要存储类的信息，常量，静态变量，即时编译器编译后代码等，这部分由于是在堆中实现的，受 GC 的管理，不过由于永久代有 <u>-XX:MaxPermSize</u> 的上限，所以如果动态生成类（将类信息放入永久代）或大量地执行 **String.intern**（将字段串放入永久代中的常量区），很容易造成 OOM。所以在 Java 8 中就把方法区的实现移到了本地内存中的元空间中，这样方法区就不受 JVM 的控制了，也就不会进行 GC，也因此提升了性能，也就不存在由于永久代限制大小而导致的 OOM 异常了（假设总内存1G，JVM 被分配内存 100M， 理论上元空间可以分配 2G-100M = 1.9G，空间大小足够），也方便在元空间中统一管理。综上所述，在 Java 8 以后这一区域也**不需要进行 GC**。

> JDK使用DirectByteBuffer对象来表示堆外内存，同样通过GC管理堆外内存的回收。具体参考：https://www.jianshu.com/p/35cf0f348275

堆：前面几块数据区域都不进行 GC，那只剩下堆了，这里是 GC 发生的区域！对象实例和数组都是在堆上分配的，GC 也主要对这两类数据进行回收。

## 垃圾识别

简述JVM判断堆中的对象实例或者数据是不是需要回收。

### 引用计数法

判断对象的引用次数，若有引用，则引用次数+1，若没有引用（引用次数为0），则此对象可以回收。

```java
// 新建对象，ref=1
String test = new String("ref=1");
// ref = 0，可回收
test = null;
```

引用计数法可能存在循环引用的问题。

```java
// new a and b, ref_a = 1, ref_b = 1;
Test a = new Test("a");
Test b = new Test("b");

// ref_a = 2, ref_b = 2;
a.instance = b;
b.instance = a;

// ref_a = 1, ref_b = 1, 对象置为null，却还是有引用次数
a = null;
b = null;
```

### 可达性算法

现代虚拟机基本都是采用这种算法来判断对象是否存活，可达性算法的原理是以一系列叫做  **GC Root** 的对象为起点出发，引出它们指向的下一个节点，再以下个节点为起点，引出此节点指向的下一个结点。这样通过 GC Root 串成的一条线就叫引用链。直到引用链遍历完毕,如果相关对象不在任意一个以 **GC Root** 为起点的引用链中，则这些对象会被判断为「垃圾」,会被 GC 回收。

![gc-roots](/img/jvm-gcroots.jpg)

如图示，如果用可达性算法即可解决上述循环引用的问题，因为从**GC Root** 出发没有到达 a,b,所以 a，b 可回收。

这里可回收并不是代表一定被回收。对象的finalize 方法给了对象一次垂死挣扎的机会，当对象不可达（可回收）时，当发生GC时，会先判断对象是否执行了 finalize 方法，如果未执行，则会先执行 finalize 方法，我们可以在此方法里将当前对象与 GC Roots 关联，这样执行 finalize 方法之后，GC 会再次判断对象是否可达，如果不可达，则会被回收，如果可达，则不回收！

**注意：** finalize 方法只会被执行一次，如果第一次执行 finalize 方法此对象变成了可达确实不会回收，但如果对象再次被 GC，则会忽略 finalize 方法，对象会被回收！这一点切记！

那么这些 **GC Roots** 到底是什么东西呢，哪些对象可以作为 GC Root 呢，有以下几类

- 虚拟机栈（栈帧中的本地变量表）中引用的对象
- 方法区中类静态属性引用的对象
- 方法区中常量引用的对象
- 本地方法栈中 JNI（即一般说的 Native 方法）引用的对象

```java
public class Test {
  public static Test s;
  //方法区中常量引用的对象 s2 指向的对象并不会因为 a 指向的对象被回收而回收
  public static final Test s2 = new Test();
  public static  void main(String[] args) {
    // 虚拟机栈中的引用对象
    Test a = new Test();
    // a 是栈帧中的本地变量，当 a = null 时，由于此时 a 充当了 GC Root 的作用，a 与原来指向的实例 new Test() 断开了连接，所以对象会被回收
    a = null;
    // 方法区中类静态属性引用的对象
    Test b = new Test();
    b.s = new Test();
    // b = null 时，由于 a 原来指向的对象与GC Root(变量b) 断开了连接，所以b原来指向的对象会被回收，而由于我们给 s 赋值了变量的引用，s在此时是类静态属性引用，充当了GC Root的作用，它指向的对象依然存活
    b = null;
  }
}
```

## 垃圾回收方法

现代虚拟机一般都使用可达性算法来识别哪些数据是垃圾，识别垃圾后，针对垃圾回收的方法，有以下几种。

### 标记清除算法

步骤很简单

1. 先根据可达性算法**标记**出相应的可回收对象（图中黄色部分）
2. 对可回收的对象进行回收

优点：操作简单，不需要做移动数据的操作

缺点：会产生比较多的内存碎片

![gc-method1](/img/jvm-gc-method1.jpg)

### 复制算法

把堆等分成两块区域, A 和 B，区域 A 负责分配对象，区域 B 不分配。步骤如下：

1. 对区域 A 使用以上所说的标记法把存活的对象标记出来
2. 把区域 A 中存活的对象都复制到区域 B（存活对象都依次**紧邻排列**）
3. 把 A 区对象全部清理掉释放出空间

优点：解决了内存碎片的问题

缺点：堆内存利用率低，不停的复制移动，效率低

### 标记整理法

前面两步和标记清除法一样，不同的是它在标记清除法的基础上添加了一个整理的过程 ，即将所有的存活对象都往一端移动，紧邻排列，再清理掉另一端的所有区域。

优点：解决了内存碎片的问题

缺点：每进一次垃圾清除都要频繁地移动存活的对象，效率十分低下

## 分代收集算法

分代收集算法整合了以上算法，综合了这些算法的优点，最大程度避免了它们的缺点，所以是现代虚拟机采用的首选算法，与其说它是算法，倒不是说它是一种策略，因为它是把上述几种算法整合在了一起（IBM 专业研究表明，一般来说，98% 的对象都是朝生夕死的，经过一次 Minor GC 后就会被回收）。

以Java8的堆内存为例。根据“对象存活周期的不同”，将堆分为新生代和老年代，默认比例为1：2，新生代又分为 Eden 区， from Survivor 区（简称S0），to Survivor 区(简称 S1),三者的比例为 8: 1 : 1。根据新生代和老年代的特点来选择不同的垃圾回收算法。新生代发送的GC为YoungGC（MinorGC），老年代发生的GC称为OldGC（FullGC）。

![jvm-heap](/img/jvm-heap.jpg)

### 工作原理

#### 对象在新生代的分配与回收

大部分对象在短时间内会被回收，因此对象一般分配在Eden区（有例外）。Eden区对象分配回收分如下几个步骤。

1. 新生成的对象分配在Eden区
2. Eden区将满时，触发MinorGC，存活的对象移动至S0区，对象年龄+1
3. 下一次MinorGC触发时，Eden区和S0区的存活对象移动至S1区，其余回收，对象年龄+1
4. 重复上述两个步骤，存活对象不停的在S0区和S1区切换，直到达到条件晋升到老年代

因此新生代用的垃圾回收算法为**复制算法**。因为Eden区分配的对象大部分MinorGC后都消亡了，因此最大限度地降低了复制算法造成的对象频繁拷贝带来的开销。

#### 对象晋升老年代

对象满足以下三个条件的任意一个则晋升至老年代。

1. 对象的年龄到达阈值（ -XX:MaxTenuringThreshold = 15）时，从新生代晋升到老年代
2. 大对象（-XX:PretenureSizeThreshold，此参数只对Serial及ParNew两款收集器有效）
3. S0区相同年龄对象大小之和大于其分配内存空间的一半以上时

> 空间分配担保
>
> 在发生 MinorGC 之前，虚拟机会先检查老年代最大可用的连续空间是否大于新生代所有对象的总空间，如果大于，那么Minor GC 可以确保是安全的,如果不大于，那么虚拟机会查看 HandlePromotionFailure 设置值是否允许担保失败。如果允许，那么会继续检查老年代最大可用连续空间是否大于历次晋升到老年代对象的平均大小，如果大于则进行 Minor GC，否则可能进行一次 Full GC。

#### Stop the world

如果老年代满了，会触发 Full GC, Full GC 会同时回收新生代和老年代（即对整个堆进行GC），它会导致 Stop The World（简称 STW）。所谓的 STW, 即在 GC（minor GC 或 Full GC）期间，只有垃圾回收器线程在工作，其他工作线程则被挂起。

> 为啥在垃圾收集期间其他工作线程会被挂起？想象一下，你一边在收垃圾，另外一群人一边丢垃圾，垃圾能收拾干净吗。

因此，减少FullGC的频率是服务调优的主要目标。由于 Full GC（或Minor GC） 会影响性能，所以我们要在一个合适的时间点发起 GC，这个时间点被称为 Safe Point，这个时间点的选定既不能太少以让 GC 时间太长导致程序过长时间卡顿，也不能过于频繁以至于过分增大运行时的负荷。一般当线程在这个时间点上状态是可以确定的，如确定 GC Root 的信息等，可以使 JVM 开始安全地 GC。Safe Point 主要指的是以下特定位置：

- 循环的末尾

- 方法返回前

- 调用方法的 call 之后

- 抛出异常的位置 

另外需要注意的是由于新生代的特点（大部分对象经过 Minor GC后会消亡）， Minor GC 用的是复制算法，而在老生代由于对象比较多，占用的空间较大，使用复制算法会有较大开销（复制算法在对象存活率较高时要进行多次复制操作，同时浪费一半空间）所以根据老生代特点，在老年代进行的 GC 一般采用的是标记整理法来进行回收。

## 垃圾回收器种类

Java 虚拟机规范并没有规定垃圾收集器应该如何实现，因此一般来说不同厂商，不同版本的虚拟机提供的垃圾收集器实现可能会有差别，一般会给出参数来让用户根据应用的特点来组合各个年代使用的收集器，主要有以下垃圾收集器。

![jvm-gc](/img/jvm-gc.jpg)

- 在新生代工作的垃圾回收器：Serial, ParNew, ParallelScavenge
- 在老年代工作的垃圾回收器：CMS，Serial Old, Parallel Old
- 同时在新老生代工作的垃圾回收器：G1

图片中的垃圾收集器如果存在连线，则代表它们之间可以配合使用，接下来我们来看看各个垃圾收集器的具体功能。

### 新生代回收器

#### Serial

工作在新生代的，单线程的垃圾收集器。

在 **Client 模式**下简单有效，对于限定单个 CPU 的环境来说，Serial 单线程模式无需与其他线程交互，减少了开销，专心做 GC 能将其单线程的优势发挥到极致，另外在用户的桌面应用场景，分配给虚拟机的内存一般不会很大，收集几十甚至一两百兆（仅是新生代的内存，桌面应用基本不会再大了），STW 时间可以控制在一百多毫秒内，只要不是频繁发生，这点停顿是可以接受的，所以对于运行在 Client 模式下的虚拟机，Serial 收集器是新生代的默认收集器。

#### ParNew

ParNew 收集器是 Serial 收集器的多线程版本，除了使用多线程，其他像收集算法,STW,对象分配规则，回收策略与 Serial 收集器完成一样。

ParNew 主要工作在 Server 模式，我们知道服务端如果接收的请求多了，响应时间就很重要了，多线程可以让垃圾回收得更快，也就是减少了 STW 时间，能提升响应时间。另一个与性能无关的原因是因为除了 Serial  收集器，**只有它能与 CMS 收集器配合工作**。

在多 CPU 的情况下，由于 ParNew 的多线程回收特性，毫无疑问垃圾收集会更快，也能有效地减少 STW 的时间，提升应用的响应速度。

#### Parallel Scavenge

Parallel Scavenge 收集器也是一个使用**复制算法**，**多线程**，工作于新生代的垃圾收集器。

CMS 等垃圾收集器关注的是尽可能缩短垃圾收集时用户线程的停顿时间，而 Parallel Scavenge 目标是达到一个可控制的吞吐量（吞吐量 = 运行用户代码时间 / （运行用户代码时间+垃圾收集时间）），也就是说 CMS 等垃圾收集器更适合用到与用户交互的程序，因为停顿时间越短，用户体验越好，而 Parallel Scavenge 收集器关注的是吞吐量，所以更适合做后台运算等不需要太多用户交互的任务。

### 老年代回收器

#### Serial Old

Serial 收集器是工作于新生代的单线程收集器，与之相对地，Serial Old 是工作于老年代的单线程收集器，此收集器的主要意义在于给 Client 模式下的虚拟机使用。

#### Parallel Old

Parallel Old 是相对于 Parallel Scavenge 收集器的老年代版本，使用多线程和标记整理法，这两者的组合由于都是多线程收集器，真正实现了「吞吐量优先」的目标。

#### CMS

CMS 收集器是以实现最短 STW 时间为目标的收集器。我们之前说老年代主要用标记整理法，而 CMS 虽然工作于老年代，但采用的是标记清除法，主要有以下四个步骤

1. 初始标记
2. 并发标记
3. 重新标记
4. 并发清除

![jvm-gc](/img/jvm-cms.jpg)

从图中可以的看到初始标记和重新标记两个阶段会发生 STW，造成用户线程挂起，不过初始标记仅标记 GC Roots 能关联的对象，速度很快，并发标记是进行 GC Roots  Tracing 的过程，重新标记是为了修正并发标记期间因用户线程继续运行而导致标记产生变动的那一部分对象的标记记录，这一阶段停顿时间一般比初始标记阶段稍长，但**远比并发标记时间短**。

整个过程中耗时最长的是并发标记和标记清理，不过这两个阶段用户线程都可工作，所以不影响应用的正常使用，所以总体上看，可以认为 CMS 收集器的内存回收过程是与用户线程一起并发执行的。

主要缺点有：

CPU资源敏感。垃圾回收线程与用户线程并行。



## 总结

在生产环境中我们要根据**不同的场景**来选择垃圾收集器组合，如果是运行在桌面环境处于 Client 模式的，则用 Serial + Serial Old 收集器绰绰有余，如果需要响应时间快，用户体验好的，则用 ParNew + CMS 的搭配模式，即使是号称是「驾驭一切」的 G1，也需要根据吞吐量等要求适当调整相应的 JVM 参数，没有最牛的技术，只有最合适的使用场景，切记！

> JVM默认垃圾回收器？？？

# GC实践

GC日志、OOM场景排查、内存调试工具。

# 参考文献

https://mp.weixin.qq.com/s/_AKQs-xXDHlk84HbwKUzOw

https://mp.weixin.qq.com/s/if81us1RNSOILzL8pwj72g