# 基础知识点

## 线程安全的实现方式

> 多线程场景下取代HashMap的方案？

- 使用Collections.synchronizedMap(Map)创建线程安全的map集合；
- Hashtable
- ConcurrentHashMap

相比之下，ConcurrentHashMap的性能和效率明显高于前两者。

### SynchronizedMap

SynchronizedMap实现线程安全的方式是使用排斥锁mutex，在常规的map方法中加入synchronized操作。

```java
private final Map<K, V> m;
final Object mutex;

public int size() {
	synchronized (mutex) {
		return m.size;
	}
}
```

### HashTable

HashTable是针对数据操作的相应方法上锁。

```java
public synchronized V get(Object key) {
	// do get operation
}
```

### ConcurrentHashMap

ConcurrentHashMap采用了 `CAS + synchronized` 来保证并发安全性。使用Node代替了1.7中的HashEntry，把值和next采用volatile修饰，保证多线程中的数据可见性，引入红黑树。

> volatile的特性是啥？
>
> - 保证了不同线程对这个变量进行操作时的可见性，即一个线程修改了某个变量的值，这新值对其他线程来说是立即可见的。（实现**可见性**）
> - 禁止进行指令重排序。（实现**有序性**）
> - volatile 只能保证对单次读/写的原子性。i++ 这种操作不能保证**原子性**。

#### Put操作

ConcurrentHashMap在进行put操作大致可以分为以下步骤：

1. 根据 key 计算出 hashcode 。
2. 判断是否需要进行初始化。
3. 即为当前 key 定位出的 Node，如果为空表示当前位置可以写入数据，利用 CAS 尝试写入，失败则自旋保证成功。
4. 如果当前位置的 `hashcode == MOVED == -1`,则需要进行扩容。
5. 如果都不满足，则利用 synchronized 锁写入数据。
6. 如果数量大于 `TREEIFY_THRESHOLD` 则要转换为红黑树。

> CAS 是乐观锁的一种实现方式，是一种轻量级锁，CAS 操作的流程如下，线程在读取数据时不进行加锁，在准备写回数据时，比较原值是否修改，若未被其他线程修改则写回，若已被修改，则重新执行读取流程。
>
> 这是一种乐观策略，认为并发操作并不总会发生。

*<u>CAS性能很高，但是我知道synchronized性能可不咋地，为啥jdk1.8升级之后反而多了synchronized？</u>*

synchronized之前一直都是重量级的锁，但是后来java官方是对他进行过升级的，他现在采用的是锁升级的方式去做的。

针对 synchronized 获取锁的方式，JVM 使用了锁升级的优化方式，就是先使用**偏向锁**优先同一线程然后再次获取锁，如果失败，就升级为 **CAS 轻量级锁**，如果失败就会短暂**自旋**，防止线程被系统挂起。最后如果以上都失败就升级为**重量级锁**。

所以是一步步升级上去的，最初也是通过很多轻量级的方式锁定的。

#### Get操作

get操作无需加锁。

- 根据计算出来的 hashcode 寻址，如果就在桶上那么直接返回值。
- 如果是红黑树那就按照树的方式获取值。
- 就不满足那就按照链表的方式遍历获取值。

## HashTable和HashMap的区别

- **实现方式不同**：Hashtable 继承了 Dictionary类，而 HashMap 继承的是 AbstractMap 类。

  Dictionary 是 JDK 1.0 添加的，貌似没人用过这个，我也没用过。

- **初始化容量不同**：HashMap 的初始容量为：16，Hashtable 初始容量为：11，两者的负载因子默认都是：0.75。

- **扩容机制不同**：当现有容量大于总容量 * 负载因子时，HashMap 扩容规则为当前容量翻倍，Hashtable 扩容规则为当前容量翻倍 + 1。

- **迭代器不同**：HashMap 中的 Iterator 迭代器是 fail-fast 的，而 Hashtable 的 Enumerator 不是 fail-fast 的。

  所以，当其他线程改变了HashMap 的结构，如：增加、删除元素，将会抛出ConcurrentModificationException 异常，而 Hashtable 则不会。

> **快速失败（fail—fast）**是java集合中的一种机制， 在用迭代器遍历一个集合对象时，如果遍历过程中对集合对象的内容进行了修改（增加、删除、修改），则会抛出Concurrent Modification Exception。

> **安全失败（fail—save）**，为了避免触发fail-fast机制，导致异常，我们可以使用Java中提供的一些采用了fail-safe机制的集合类。fail-save的集合容器在遍历时不是直接在集合内容上访问的，而是先复制原有集合内容，在拷贝的集合上进行遍历。
>
> java.util.concurrent包下的容器都是fail-safe的，可以在多线程下并发使用，并发修改。同时也可以在foreach中进行add/remove 。

# 知识延伸：同步容器和并发容器

在Java中，同步容器主要包括2类：

- 1、Vector、Stack、HashTable
- 2、Collections类中提供的静态工厂方法创建的类

Vector这样的同步容器的所有公有方法全都是synchronized的，也就是说，我们可以在多线程场景中放心的使用单独这些方法，因为这些方法本身的确是线程安全的。

但是，请注意上面这句话中，有一个比较关键的词：单独。因为，虽然同步容器的所有方法都加了锁，但是对这些容器的复合操作无法保证其线程安全性。需要客户端通过主动加锁来保证。

```java
// 方法中包含复合操作，这样在多线程的环境下，同样是线程不安全的
public Object deleteLast(Vector v){
    int lastIndex  = v.size()-1;
    v.remove(lastIndex);
}
// 这时候需要手动的为该方法加锁，并发度低
public Object deleteLast(Vector v){
  synchronized(v) {
    int lastIndex  = v.size()-1;
    v.remove(lastIndex);
  } 
}
```

这也是为什么在实际开发中，我们选用java.util.concurent包下的并发容器类。并发容器有更好的并发能力。而且其中的ConcurrentHashMap定义了线程安全的复合操作。比如putIfAbsent()、replace()，这2个操作都是原子操作，可以保证线程安全。

# 常见问题

- 谈谈你理解的 Hashtable，讲讲其中的 get put 过程。ConcurrentHashMap同问。
- 1.8 做了什么优化？
- 线程安全怎么做的？
- 不安全会导致哪些问题？
- 如何解决？有没有线程安全的并发容器？
- ConcurrentHashMap 是如何实现的？
- ConcurrentHashMap并发度为啥好这么多？
- 1.7、1.8 实现有何不同？为什么这么做？
- CAS是啥？
- ABA是啥？场景有哪些，怎么解决？
- synchronized底层原理是啥？
- synchronized锁升级策略
- 快速失败（fail—fast）是啥，应用场景有哪些？安全失败（fail—safe）同问。

# 参考文档

https://mp.weixin.qq.com/s/AixdbEiXf3KfE724kg2YIw

https://mp.weixin.qq.com/s/0cMrE87iUxLBw_qTBMYMgA