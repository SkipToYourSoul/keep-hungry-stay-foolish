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

1. 对象的年龄到达阈值（ -XX:MaxTenuringThreshold = 15，CMS为6）时，从新生代晋升到老年代
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

> It differs from 'Parallel Scavenge' in that it has enhancements that make it usable with CMS

ParNew 收集器是 Serial 收集器的多线程版本，除了使用多线程，其他像收集算法,STW,对象分配规则，回收策略与 Serial 收集器完成一样。

ParNew 主要工作在 Server 模式，我们知道服务端如果接收的请求多了，响应时间就很重要了，多线程可以让垃圾回收得更快，也就是减少了 STW 时间，能提升响应时间。另一个与性能无关的原因是因为除了 Serial  收集器，**只有它能与 CMS 收集器配合工作**。

在多 CPU 的情况下，由于 ParNew 的多线程回收特性，毫无疑问垃圾收集会更快，也能有效地减少 STW 的时间，提升应用的响应速度。

#### Parallel Scavenge

Parallel Scavenge 收集器也是一个使用**复制算法**，**多线程**，工作于新生代的垃圾收集器。

> A stw, copying collector which use multiple GC threads

CMS 等垃圾收集器关注的是尽可能缩短垃圾收集时用户线程的停顿时间，而 Parallel Scavenge 目标是达到一个可控制的吞吐量（吞吐量 = 运行用户代码时间 / （运行用户代码时间+垃圾收集时间）），也就是说 CMS 等垃圾收集器更适合用到与用户交互的程序，因为停顿时间越短，用户体验越好，而 Parallel Scavenge 收集器关注的是吞吐量，所以更适合做后台运算等不需要太多用户交互的任务。

Parallel Scavenge 收集器提供了两个参数来精确控制吞吐量，分别是控制最大垃圾收集时间的 -XX:MaxGCPauseMillis 参数及直接设置吞吐量大小的 -XX:GCTimeRatio（默认99%）

除了以上两个参数，还可以用 Parallel Scavenge 收集器提供的第三个参数 -XX:UseAdaptiveSizePolicy，开启这个参数后，就不需要手工指定新生代大小,Eden 与 Survivor 比例（SurvivorRatio）等细节，只需要设置好基本的堆大小（-Xmx 设置最大堆）,以及最大垃圾收集时间与吞吐量大小，虚拟机就会根据当前系统运行情况收集监控信息，动态调整这些参数以尽可能地达到我们设定的最大垃圾收集时间或吞吐量大小这两个指标。自适应策略也是 Parallel Scavenge  与 ParNew 的重要区别！

### 老年代回收器

#### Serial Old

Serial 收集器是工作于新生代的单线程收集器，与之相对地，Serial Old 是工作于老年代的单线程收集器，此收集器的主要意义在于给 Client 模式下的虚拟机使用。

#### Parallel Old

> A stw, compacting collector which use multiple GC threads

Parallel Old 是相对于 Parallel Scavenge 收集器的老年代版本，使用多线程和标记整理法，这两者的组合由于都是多线程收集器，真正实现了「吞吐量优先」的目标。

#### CMS

> Concurrent mark sweep, a mostly concurrent, low-pause collector

CMS 收集器是以实现最短 STW 时间为目标的收集器。我们之前说老年代主要用标记整理法，而 CMS 虽然工作于老年代，但采用的是标记清除法，主要有以下四个步骤

1. 初始标记
2. 并发标记
3. 重新标记
4. 并发清除

![jvm-gc](/img/jvm-cms.jpg)

从图中可以的看到初始标记和重新标记两个阶段会发生 STW，造成用户线程挂起，不过初始标记仅标记 GC Roots 能关联的对象，速度很快，并发标记是进行 GC Roots  Tracing 的过程，重新标记是为了修正并发标记期间因用户线程继续运行而导致标记产生变动的那一部分对象的标记记录，这一阶段停顿时间一般比初始标记阶段稍长，但**远比并发标记时间短**。

整个过程中耗时最长的是并发标记和标记清理，不过这两个阶段用户线程都可工作，所以不影响应用的正常使用，所以总体上看，可以认为 CMS 收集器的内存回收过程是与用户线程一起并发执行的。

主要缺点有：

1. CPU资源敏感。垃圾回收线程与用户线程并行。CMS 默认启动的回收线程数是（CPU数量+3）/ 4，如果 CPU 数量只有一两个，那吞吐量就直接下降 50%,显然是不可接受的。
2. CMS 无法处理浮动垃圾（Floating Garbage）。由于在并发清理阶段用户线程还在运行，所以清理的同时新的垃圾也在不断出现，这部分垃圾只能在下一次 GC 时再清理掉（即浮云垃圾），同时在垃圾收集阶段用户线程也要继续运行，就需要预留足够多的空间要确保用户线程正常执行，这就意味着 CMS 收集器不能像其他收集器一样等老年代满了再使用。JDK 1.5 默认当老年代使用了68%空间后就会被激活，当然这个比例可以通过 -XX:CMSInitiatingOccupancyFraction 来设置，但是如果设置地太高很容易导致在 CMS 运行期间预留的内存无法满足程序要求，会导致 **Concurrent Mode Failure** 失败，这时会启用 Serial Old 收集器来重新进行老年代的收集，而我们知道 Serial Old 收集器是单线程收集器，这样就会导致 STW 更长了。
3. 标记清除法产生内存碎片。当然我们可以开启 -XX:+UseCMSCompactAtFullCollection（默认是开启的），用于在 CMS 收集器顶不住要进行 FullGC 时开启内存碎片的合并整理过程，内存整理会导致 STW，停顿时间会变长，还可以用另一个参数 -XX:CMSFullGCsBeforeCompation 用来设置执行多少次不压缩的 Full GC 后跟着带来一次带压缩的。

#### G1 - Garbage First

G1 收集器是面向服务端的垃圾收集器，被称为驾驭一切的垃圾回收器，主要有以下几个特点

- 像 CMS 收集器一样，能与应用程序线程并发执行。
- 整理空闲空间更快。
- 需要 GC 停顿时间更好预测。
- 不会像 CMS 那样牺牲大量的吞吐性能。
- 不需要更大的 Java Heap

与 CMS 相比，它在以下两个方面表现更出色

1. 运作期间不会产生内存碎片，G1 从整体上看采用的是标记-整理法，局部（两个 Region）上看是基于复制算法实现的，两个算法都不会产生内存碎片，收集后提供规整的可用内存，这样有利于程序的长时间运行。
2. 在 STW 上建立了**可预测**的停顿时间模型，用户可以指定期望停顿时间，G1 会将停顿时间控制在用户设定的停顿时间以内。

为什么G1能建立可预测的停顿模型呢？G1 各代的存储地址不是连续的，每一代都使用了 n 个不连续的大小相同的 Region，每个Region占有一块连续的虚拟内存地址。

![jvm-g1](/img/jvm-g1.jpg)

除了和传统的新老生代，幸存区的空间区别，Region还多了一个H，它代表Humongous，这表示这些Region存储的是巨大对象（humongous object，H-obj），即大小大于等于region一半的对象，这样超大对象就直接分配到了老年代，防止了反复拷贝移动。那么 G1 分配成这样有啥好处呢？

传统的收集器如果发生 Full GC 是对整个堆进行全区域的垃圾收集，而分配成各个 Region 的话，方便 G1 跟踪各个 Region 里垃圾堆积的价值大小（回收所获得的空间大小及回收所需经验值），这样根据价值大小维护一个优先列表，根据允许的收集时间，优先收集回收价值最大的 Region,也就避免了整个老年代的回收，也就减少了 STW 造成的停顿时间。同时由于只收集部分 Region,可就做到了 STW 时间的可控。

G1 收集器的工作步骤如下

1. 初始标记
2. 并发标记
3. 最终标记
4. 筛选回收

可以看到整体过程与 CMS 收集器非常类似，筛选阶段会根据各个 Region 的回收价值和成本进行排序，根据用户期望的 GC 停顿时间来制定回收计划。

## 总结

在生产环境中我们要根据**不同的场景**来选择垃圾收集器组合，如果是运行在桌面环境处于 Client 模式的，则用 Serial + Serial Old 收集器绰绰有余，如果需要响应时间快，用户体验好的，则用 ParNew + CMS 的搭配模式，即使是号称是「驾驭一切」的 G1，也需要根据吞吐量等要求适当调整相应的 JVM 参数，没有最牛的技术，只有最合适的使用场景，切记！

> 历代JVM默认垃圾回收器
>
> - jdk1.7 默认垃圾收集器Parallel Scavenge（新生代）+Parallel Old（老年代）
>
> - jdk1.8 默认垃圾收集器Parallel Scavenge（新生代）+Parallel Old（老年代）
> - jdk1.9 默认垃圾收集器G1
> - jdk10 默认垃圾收集器G1
> - jdk11 默认垃圾收集器G1

# GC实践

GC日志、OOM场景排查和解决方案、内存调试工具。

## JVM参数简介

Java程序启动时可以设置相应的JVM参数。

```java
// 脸谱线上服务启动参数设置样例, 线上服务单容器配置为4core 8g
// 设置初始堆和最大堆大小为6g，新生代老年代比例为1:2，使用并发GC并且并发线程数为4
// 将4分之3的内存分配给堆，其余为堆外内存，实测中内存使用在45 - 70间波动
// 并发回收下，加大老年代空间占比，一般来说，新生代占比也不宜过大，复制算法的消耗是很大的
// 并发线程数 = core，充分利用cpu资源，回收时不需要让线程切换cpu时间导致浪费
-Xms6g -Xmx6g -XX:NewRatio=2 -XX:+UseParallelGC -XX:ParallelGCThreads=4 
// 当JVM发生OOM时，自动生成DUMP文件，方便排查内存泄露等问题
-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=${MARATHON_INJECT_VOLUME_LOG} 
// GC日志相关设置
-XX:+PrintGC -XX:+PrintGCDateStamps -XX:+PrintGCDetails -XX:GCLogFileSize=50M -XX:NumberOfGCLogFiles=10 -XX:+UseGCLogFileRotation -Xloggc:${MARATHON_INJECT_VOLUME_LOG}/gc.log

// jvm启动时gc日志如下
CommandLine flags: -XX:CICompilerCount=15 -XX:GCLogFileSize=52428800 -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/data/lianpu/online-server/logs -XX:InitialHeapSize=6442450944 -XX:MaxHeapSize=6442450944 -XX:MaxNewSize=2147483648 -XX:MinHeapDeltaBytes=524288 -XX:NewRatio=2 -XX:NewSize=2147483648 -XX:NumberOfGCLogFiles=10 -XX:OldSize=4294967296 -XX:ParallelGCThreads=4 -XX:+PrintGC -XX:+PrintGCDateStamps -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseGCLogFileRotation -XX:+UseParallelGC 
  
// YoungGc
2020-02-11T15:39:55.243+0800: 2179926.813: [GC (Allocation Failure) [PSYoungGen: 2087840K->4704K(2089984K)] 2919176K->838184K(6284288K), 0.0102432 secs] [Times: user=0.04 sys=0.00, real=0.01 secs] 
// FullGc
2020-03-03T06:44:24.388+0800: 3962195.958: [Full GC (Ergonomics) [PSYoungGen: 5248K->0K(2089472K)] [ParOldGen: 4190173K->51670K(4194304K)] 4195421K->51670K(6283776K), [Metaspace: 52366K->52366K(1099776K)], 0.1783458 secs] [Times: user=0.15 sys=0.14, real=0.18 secs]
```

JVM 参数主要分以下三类

1、 标准参数（-），所有的 JVM 实现都必须实现这些参数的功能，而且向后兼容；例如 **-verbose:gc**（输出每次GC的相关情况)

2、 非标准参数（-X），默认 JVM 实现这些参数的功能，但是并不保证所有 JVM 实现都满足，且不保证向后兼容，栈，堆大小的设置都是通过这个参数来配置的。例如

| 参数示例 | 参数含义                        |
| -------- | ------------------------------- |
| -Xms512m | JVM启动时设置的初始堆大小为512M |
| -Xms512m | 可分配的最大堆大小为512M        |
| -Xmn200m | 年轻代大小为200M                |
| -Xss128k | 设置每个线程的栈大小为128K      |

3、非Stable参数（-XX），此类参数各个 jvm 实现会有所不同，将来可能会随时取消，需要慎重使用。-XX:-option 代表关闭 option 参数，-XX:+option 代表要开启 option 参数。例如要启用串行 GC，对应的 JVM 参数即为 -XX:+UseSerialGC。非 Stable 参数主要有三大类

- 行为参数（Behavioral Options）：用于改变 JVM 的一些基础行为，如启用串行/并行 GC

| 参数示例                | 表示意义                                                  |
| :---------------------- | :-------------------------------------------------------- |
| -XX:-DisableExplicitGC  | 禁止调用System.gc()；但jvm的gc仍然有效                    |
| -XX:-UseConcMarkSweepGC | 对老生代采用并发标记交换算法进行GC                        |
| -XX:-UseParallelGC      | 启用并行GC                                                |
| -XX:-UseParallelOldGC   | 对Full GC启用并行，当-XX:-UseParallelGC启用时该项自动启用 |
| -XX:-UseSerialGC        | 启用串行GC                                                |

- 性能调优（Performance Tuning）：用于 jvm 的性能调优，如设置新老生代内存容量比例

| 参数示例                      | 表示意义                              |
| :---------------------------- | :------------------------------------ |
| -XX:MaxHeapFreeRatio=70       | GC后java堆中空闲量占的最大比例        |
| -XX:NewRatio=2                | 新生代内存容量与老生代内存容量的比例  |
| -XX:NewSize=2.125m            | 新生代对象生成时占用内存的默认值      |
| -XX:ReservedCodeCacheSize=32m | 保留代码占用的内存容量                |
| -XX:ThreadStackSize=512       | 设置线程栈大小，若为0则使用系统默认值 |

- 调试参数（Debugging Options）：一般用于打开跟踪、打印、输出等 JVM 参数，用于显示 JVM 更加详细的信息

| 参数示例                          | 表示意义                            |
| :-------------------------------- | :---------------------------------- |
| -XX:HeapDumpPath=./java_pid.hprof | 指定导出堆信息时的路径或文件名      |
| -XX:-HeapDumpOnOutOfMemoryError   | 当首次遭遇OOM时导出此时堆中相关信息 |
| -XX:-PrintGC                      | 每次GC时打印相关信息                |
| -XX:-PrintGC Details              | 每次GC时打印详细信息                |

## 发生OOM的主要几种场景及相应解决方案

### JVM规范中描述在栈上发生的异常

**在单线程下**，当栈桢太大或虚拟机容量太小导致内存无法分配时，都会发生 StackOverflowError 异常。例如

- **单个线程**请求栈深度大于虚拟机所允许的最大深度（如常用的递归调用层级过深等）
- 单个线程定义了大量的本地变量，导致方法帧中本地变量表长度过大等也会导致 StackOverflowError 异常

虚拟机在扩展栈时无法申请到足够的内存空间，会抛出 OOM （OutOfMemory）异常。

操作系统给每个进程分配的内存是有限制的，比如 32 位的 Windows 限制为 2G。虚拟机提供了参数来控制 Java 堆和方法的这两部内存的最大值，剩余的内存为 「2G - Xmx（最大堆容量）= 线程数 * 每个线程分配的虚拟机栈（-Xss)+本地方法栈 」（程序计数器消耗内存很少，可忽略），每个线程都会被分配对应的虚拟机栈大小，所以总可创建的线程数肯定是固定的。不过这也给我们提供了一个新思路，如果是因为建立过多的线程导致的内存溢出，而我们又想多创建线程，可以通过减少最大堆（-Xms）和增大虚拟机栈大小（-Xss）来实现。

### 堆溢出 （**java.lang.OutOfMemoryError:Java heap space**）

OOM的原因主要有两个，分别是：1）大对象分配；2）内存泄露

```java
/**
* VM Args:-Xmx12m
* 大对象分配造成OOM
 */
class OOM {
  	// 2 * 1M * 4byte = 8M, 4M newGen, 8M oldGen
    static final int SIZE=2*1024*1024;
    public static void main(String[] a) {
        int[] i = new int[SIZE];
    }
}
```

程序整个生命周期中，如果GC无法回收对象，就会发生内存泄漏。

对于这种内存泄漏导致的 OOM， 单纯地增大堆大小是无法解决根本问题的，只不过是延缓了 OOM 的发生，最根本的解决方式还是要通过 **heap dump analyzer** 等方式来找出内存泄漏的代码来修复解决。

### java.lang.OutOfMemoryError:GC overhead limit exceeded

Sun 官方对此的定义：超过98%的时间用来做 GC 并且回收了不到 2% 的堆内存时会抛出此异常。

这种情况下会快速的将CPU资源打满，产生问题的原因与堆溢出类似，解决方法如下：

- 检查项目中是否有大量的死循环或有使用大内存的代码，优化代码。
- dump 内存，检查是否存在内存泄露，如果没有，可考虑通过 -Xmx 参数设置加大内存。

### java.lang.OutOfMemoryError:Permgen space

java8前使用永久代存放被虚拟机加载的类，常量，静态变量，JIT 编译后的代码等信息，所以如果错误地频繁地使用 String.intern() 方法或运行期间生成了大量的代理类都有可能导致永久代溢出，解决方案如下

- 是否永久代设置的过小，如果可以，适应调大一点
- 检查代码是否有大量的反射操作
- dump 之后通过 mat 检查是否存在大量由于反射生成的代码类

### java.lang.OutOfMemoryError:Requested array size exceeds VM limit

该错误由 JVM 中的 native code 抛出。JVM 在为数组分配内存之前，会执行基于所在平台的检查：分配的数据结构是否在此平台中是可寻址的，平台一般允许分配的数据大小在 1 到 21 亿之间，如果超过了这个数就会抛出这种异常。

### java.lang.OutOfMemoryError: Out of swap space

Java 应用启动的时候分被分配一定的内存空间(通过 -Xmx 及其他参数来指定)， 如果 JVM 要求的总内存空间大小大于可用的本机内存，则操作系统会将内存中的部分数据交换到硬盘上。

如果此时 swap 分区大小不足或者其他进程耗尽了本机的内存，则会发生 OOM, 可以通过增大 swap 空间大小来解决,但如果在交换空间进行 GC 造成的 「Stop The World」增加大个数量级，所以增大 swap 空间一定要慎重，所以一般是通过增大本机内存或优化程序减少内存占用来解决。

### Out of memory:Kill process or sacrifice child

操作系统内存不足时，通过内核调度任务中的「Out of memory killer」的调度器，优先杀掉占用内存大的应用进程。

看了以上的各种 OOM 产生的情况，可以看出：**GC 和是否发生 OOM 没有必然的因果关系!** GC 主要发生在堆上，而 从以上列出的几种发生 OOM 的场景可以看出，空间不足无法再创建线程，或者存在死循环一直在分配对象导致 GC 无法回收对象或者一次分配大内存数组（超过堆的大小）等都可能导致 OOM。

## OOM问题排查的一些方法

内存泄漏是最常见的造成 OOM 的一种原因，在现实中各种不同的服务场景，可采用不同的方法来排查。

### 分析dump（堆转储快照）文件

运行 Java 时添加 「-XX:+HeapDumpOnOutOfMemoryError」 参数来导出内存溢出时的堆信息，生成 hrof 文件, 添加 「-XX:HeapDumpPath」可以指定 hrof 文件的生成路径,如果不指定则 hrof 文件生成在与字节码文件相同的目录下。

通过分析文件找到对应造成OOM的代码行。

### 使用jvisualvm来分析

jvisualvm 的功能强大，除了可以实时监控堆内存的使用情况，还可以跟踪垃圾回收，运行中 dump 中堆内存使用情况、cpu分析，线程分析等，是查找分析问题的利器，更骚的是它不光能分析本地的 Java 程序，还可以分析线上的 Java 程序运行情况, 本身这款工具也是随 JDK 发布的，是官方力推的一款运行监视，故障处理的神器。

### 常用命令分析

部署在容器中的服务，安装工具是一件比较繁琐的事情，此时，可以使用一些命令排查。

（TODO：+况指导调研的方法）

jps 可以列出正在运行的虚拟机进程，并显示执行虚拟机主类及这些进程的本地虚拟机唯一 ID。

拿到进程的 pid 后，我们就可以用 jmap 来 dump 出堆转储文件了，执行命令如下：

```java
jmap -dump:format=b,file=heapdump.phrof pid
```

但这个命令在生产上一定要慎用！因为JVM 会将整个 heap 的信息 dump 写入到一个文件，heap 比较大的话会导致这个过程比较耗时，并且执行过程中为了保证 dump 的信息是可靠的，会暂停应用！

jstat 是用于监视虚拟机各种运行状态信息的命令行工具，可以显示本地或者远程虚拟机进程中的类加载，内存，垃圾收集，JIT 编译等运行数据，jstat 支持定时查询相应的指标，如下

```java
jstat -gc 2764 250 22
```

## 再谈 JVM 参数设置

我们再谈谈如何设置 JVM 参数。

1、首先 Oracle 官方推荐堆的初始化大小与堆可设置的最大值一般是相等的，即 Xms = Xmx，因为起始堆内存太小（Xms），会导致启动初期频繁 GC，起始堆内存较大（Xmx）有助于减少 GC 次数

2、调试的时候设置一些打印参数，如-XX:+PrintClassHistogram -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintHeapAtGC -Xloggc:log/gc.log，这样可以从gc.log里看出一些端倪出来

3、系统停顿时间过长可能是 GC 的问题也可能是程序的问题，多用 jmap 和 jstack 查看，或者killall -3 Java，然后查看 Java 控制台日志，能看出很多问题

4、 采用并发回收时，年轻代小一点，年老代要大，因为年老大用的是并发回收，即使时间长点也不会影响其他程序继续运行，网站不会停顿

5、仔细了解自己的应用，如果用了缓存，那么年老代应该大一些，缓存的 HashMap 不应该无限制长，建议采用 LRU 算法的 Map 做缓存，LRUMap 的最大长度也要根据实际情况设定，常用Guava的LoadingCache。

要设置好各种 JVM 参数，还可以对 server 进行压测， 预估自己的业务量，设定好一些 JVM 参数进行压测看下这些设置好的 JVM 参数是否能满足要求

# 参考文献

https://mp.weixin.qq.com/s/_AKQs-xXDHlk84HbwKUzOw

https://mp.weixin.qq.com/s/if81us1RNSOILzL8pwj72g

视频教程：https://www.bilibili.com/video/av97929214?p=2