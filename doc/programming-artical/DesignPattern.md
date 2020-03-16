# Java设计模式总结

创建型模式：单例、工厂、建造者、原型

> **创建型模式(Creational Pattern)：**对类的实例化过程进行了抽象，能够将软件模块中对象的创建和对象的使用分离。为了使软件的结构更加清晰，外界对于这些对象只需要知道它们共同的接口，而不清楚其具体的实现细节，使整个系统的设计更加符合单一职责原则。

结构型模式：适配器、桥接、组合、装饰、外观、享元、代理

> **结构型模式(Structural Pattern)：** 描述如何将类或者对象结合在一起形成更大的结构，就像搭积木，可以通过简单积木的组合形成复杂的、功能更为强大的结构

行为模式：职责链、命令、解释器、迭代器、中介者、备忘录、观察者、状态、策略

> 行为型模式(Behavioral Pattern)是对在不同的对象之间划分责任和算法的抽象化。
>
> 通过行为型模式，可以更加清晰地划分类与对象的职责，并研究系统在运行时实例对象之间的交互。在系统运行时，对象并不是孤立的，它们可以通过相互通信与协作完成某些复杂功能，一个对象在运行时也将影响到其他对象的运行。

# 创造性模式

## 单例

**饿汉方式**。指全局的单例实例在类装载时构建

```java
// 饿汉方式（线程安全）
public class Singleton {
  private static Singleton instance = new Singleton();
  private Singletion() {}
  public static Singleton getInstance() {
    return instance;
  }
}

// 使用枚举实现
public enum Singleton {
  INSTANCE;
  public void doSomeThing() {}
}

public class callSingletion {
  public static void main(String[] args) {
    Singleton singleton = Singleton.INSTANCE;
    singleton.doSomeThing();
  }
}
```

> 《Java与模式》中，作者这样写道，使用枚举来实现单实例控制会更加简洁，而且无偿地提供了序列化机制，并由JVM从根本上提供保障，绝对防止多次实例化，是更简洁、高效、安全的实现单例的方式。

**懒汉方式**。指全局的单例实例在第一次被使用时构建。

```java
// 使用双重检查加锁实现，保证线程安全
public class Singleton {
  private volatile static Singleton instance;
  private Singleton() {}
  public static Singleton getInstance() {
    if (instance == null) {
      synchronized(Singleton.class) {
        if (instance == null) {
          instance = new Singleton();
        }
      }
    }
    return instance;
  }
}

// 使用内部类实现, 显式调用 getInstance 方法时，才会显式装载 SingletonHolder 类
public class Singleton {
  private static class SingletonHolder {
    private static final Singleton instance = new Singleton();
  }
  private Singletion() {}
  public static Singleton getInstance() {
    return SingletonHolder.instance;
  }
}
```



# 参考文献

[https://github.com/Snailclimb/JavaGuide/blob/master/docs/system-design/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F.md](https://github.com/Snailclimb/JavaGuide/blob/master/docs/system-design/设计模式.md)