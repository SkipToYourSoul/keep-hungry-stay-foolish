# HashMap知识点汇总

## 数据结构

Java7：数组+链表

Java8：数据+链表（转红黑树）

```java
// node 数据结构
static class Node<K,V> implements Map.Entry<K,V> {
	final int hash;
  final K key;
  V value;
  Node<K,V> next;
  ... ...
}
```

### 链表插入方式

Java8之后由头插法更新至尾插法，Why？

HashMap扩容因素有两个：

- Capacity：当前HashMap长度
- LoadFactory：负载因子，默认0.75

即长度超过当前长度的75%，触发扩容。

扩容分为两步：

- 扩容：创建一个新的Entry空数组，长度是原数组的2倍。
- ReHash：遍历原Entry数组，把所有的Entry重新Hash到新数组。

```java
// Hash公式，因此需要rehash
index = HashCode(Key)&(length - 1)
```

在rehash的过程中，若使用头插入的方法，可能会造成循环链表。如两个Node，Node A为Node B的父节点，resize后，Node B可能会变成Node A的父节点，造成循环引用。

### 链表转红黑树

若桶中链表元素超过8时，会自动转化成红黑树；若桶中元素小于等于6时，树结构还原成链表形式。

红黑树的平均查找长度是log(n)，长度为8，查找长度为log(8)=3，链表的平均查找长度为n/2，当长度为8时，平均查找长度为8/2=4，这才有转换成树的必要；链表长度如果是小于等于6，6/2=3，虽然速度也很快的，但是转化为树结构和生成树的时间并不会太短。

选择6和8的原因是：中间有个差值7可以防止链表和树之间频繁的转换。假设一下，如果设计成链表个数超过8则链表转换成树结构，链表个数小于8则树结构转换成链表，如果一个HashMap不停的插入、删除元素，链表个数在8左右徘徊，就会频繁的发生树转链表、链表转树，效率会很低。

### 初始化长度

初始化大小为16，Why？

```java
/**
	* The default initial capacity - MUST be a power of two.
*/
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
```

16是一个约定，换句话说，用2的次幂都ok。主要是为了位运算的方便，**位与运算比算数计算的效率高了很多**，之所以选择16，是为了服务将Key映射到index的算法。

index计算时，hash结果与length-1进行&运算，此时length-1转换成2进制为1111，即index值等于对key的hash值，能保证hash的均匀分布。

因此在初始化hashmap时，无论构造函数中传入的初始化值为多少，hashmap在初始化时都会自动转成比该值大的最接近的一个2的次幂。

至于细节，可参考：https://mp.weixin.qq.com/s/ktre8-C-cP_2HZxVW5fomQ

### 数据查询

根据key去get数据时，首先对key进行hash计算出index，找到index后，通过equal方法判断key等，从而获取结果value值。

HashMap的equal函数和hashcode函数都是重写过的。

```java
public final int hashCode() {
	return Objects.hashCode(key) ^ Objects.hashCode(value);
}

public final boolean equals(Object o) {
	if (o == this)
  	return true;
  if (o instanceof Map.Entry) {
  	Map.Entry<?,?> e = (Map.Entry<?,?>)o;
    if (Objects.equals(key, e.getKey()) && Objects.equals(value, e.getValue()))
    	return true;
  }
  return false;
}
```

## HashMap常见面试题

- HashMap的底层数据结构？
- HashMap的存取原理？
- Java7和Java8的区别？
- 为啥会线程不安全？
- 有什么线程安全的类代替么?
- 默认初始化大小是多少？为啥是这么多？为啥大小都是2的幂？
- HashMap的扩容方式？负载因子是多少？为什是这么多？
- HashMap的主要参数都有哪些？
- HashMap是怎么处理hash碰撞的？
- hash的计算规则？

## 参考链接

https://github.com/AobingJava/JavaFamily/blob/master/docs/basics/HashMap.md

https://mp.weixin.qq.com/s/ktre8-C-cP_2HZxVW5fomQ