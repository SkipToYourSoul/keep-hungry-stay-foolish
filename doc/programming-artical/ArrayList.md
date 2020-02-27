# 基础知识点

## 数据结构

ArrayList就是数组列表，主要用来装载数据，当我们装载的是基本类型的数据int，long，boolean，short，byte…的时候我们只能存储他们对应的包装类，它的主要底层实现是数组Object[] elementData。

```java
public ArrayList(int initialCapacity) {
  if (initialCapacity > 0) {
    this.elementData = new Object[initialCapacity];
  } else if (initialCapacity == 0) {
    this.elementData = EMPTY_ELEMENTDATA;
  } else {
    throw new IllegalArgumentException("Illegal Capacity: "+ initialCapacity);
  }
}

public ArrayList() {
  this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```

> ArrayList与LinkedList以及Vector的区别？
>
> 1. LinkedList内部用链表实现，插入删除效率高，查找效率低，而ArrayList则相反。
> 2. Vector是线程安全的，方法用synchronize修饰。
> 3. Collections.synchronizedList也是线程安全的，用互斥锁(mutex)保证线程安全。

## 常用方法

关于内部数组长度扩容的问题。若无参数构造函数初始化，默认为空数组，在使用add方法添加元素时，才分配默认DEFAULT_CAPACITY = 10的初始容量。每次扩容时，新的长度为currentLength + currentLength >> 1。

```java
// ArrayList内部数组扩容
private void grow(int minCapacity) {
  // overflow-conscious code
  int oldCapacity = elementData.length;
  int newCapacity = oldCapacity + (oldCapacity >> 1);
  if (newCapacity - minCapacity < 0)
    newCapacity = minCapacity;
  if (newCapacity - MAX_ARRAY_SIZE > 0)
    newCapacity = hugeCapacity(minCapacity);
  // minCapacity is usually close to size, so this is a win:
  elementData = Arrays.copyOf(elementData, newCapacity);
}
```

增删操作都是通过数据copy的方式实现的，所以效率会偏低。

> ArrayList（int initialCapacity）会不会初始化数组大小？

```java
ArrayList<Integer> list = new ArrayList<>(10);
System.out.println(list.size());
list.add(1);
System.out.println(list.size());

// 以上代码输出为：0， 10
```

原因是，构造函数初始化时，只会初始化内部数组，不会更改size的值。此时set方法也会报错。

```java
public E set(int index, E element) {
	rangeCheck(index);
	E oldValue = elementData(index);
	elementData[index] = element;
	return oldValue;
}

private void rangeCheck(int index) {
	if (index >= size)
		throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
```

## 使用场景

日常开发中ArrayList是比较常用的。适合查找>增删的场景。

若增删操作比较多，建议使用LinkedList；

对其他场景，可以了解相关的基础结构例如Queue、Stack等，更复杂的包括ArrayBlockingQueue等。

ArrayList就是动态数组，用MSDN中的说法，就是Array的复杂版本，它提供了动态的增加和减少元素，实现了ICollection和IList接口，灵活的设置数组的大小等好处。

# 参考文献

https://mp.weixin.qq.com/s/WoGclm7SsbURGigI3Mwr3w