# Java多线程学习总结

汇总了Java开发中常见的多线程问题。

## 什么是线程安全

“线程安全”也不是指线程的安全，而是指内存的安全。

目前主流操作系统都是多任务的，即多个进程同时运行。为了保证安全，每个进程只能访问分配给自己的内存空间，而不能访问别的进程的，这是由操作系统保障的。在每个进程的内存空间中都会有一块特殊的公共区域，通常称为堆（内存）。进程内的所有线程都可以访问到该区域，这就是造成问题的潜在原因。

所以线程安全指的是，在堆内存中的数据由于可以被任何线程访问到，在没有限制的情况下存在被意外修改的风险。即堆内存空间在没有保护机制的情况下，对多线程来说是不安全的地方，因为你放进去的数据，可能被别的线程“破坏”。

## 保障内存安全的方法

### 变量操作

#### 局部变量

操作系统会为每个线程分配属于它自己的内存空间，通常称为栈内存，其它线程无权访问。这也是由操作系统保障的。如果一些数据只有某个线程会使用，其它线程不能操作也不需要操作，这些数据就可以放入线程的栈内存中。较为常见的就是局部变量。局部变量会在每个线程的栈内存中都分配一份。

由于线程的栈内存只能自己访问，所以栈内存中的变量只属于自己，其它线程根本就不知道。

#### ThreadLoal

当变量不再是局部变量，而是类变量时，这时候就需要通过其他方法保证变量的安全。对ThreadLocal的变量，每个线程在运行时都会拷贝一份存储到自己的本地。

```java
class Solution {
	ThreadLocal<String> cat = new ThreadLocal<>();
	getCat() {
		return cat.get();
	}
}
```

线程类（Thread）有一个成员变量，类似于Map类型的，专门用于存储ThreadLocal类型的数据。从逻辑从属关系来讲，这些ThreadLocal数据是属于Thread类的成员变量级别的。从所在“位置”的角度来讲，这些ThreadLocal数据是分配在公共区域的堆内存中的。

ThreadLocal就是，把一个数据复制N份，每个线程认领一份，各玩各的，互不影响。

#### 常量或只读变量

```java
final String cat = "miao";
```

常量或只读变量，它们对于多线程是安全的，想改也改不了。

### 锁操作

#### 互斥锁

如果公共区域（堆内存）的数据，要被多个线程操作时，为了确保数据的安全（或一致）性，需要在数据旁边放一把锁，要想操作数据，先获取锁再说吧。

加锁的方式有很多，锁的等级也不一样，之后详谈。

#### 乐观锁

乐观锁持乐观态度，就是假设我的数据不会被意外修改，如果修改了，就放弃，从头再来。悲观锁持悲观态度，就是假设我的数据一定会被意外修改，那干脆直接加锁得了。

CAS 是乐观锁的一种实现方式，是一种轻量级锁，CAS 操作的流程如下，线程在读取数据时不进行加锁，在准备写回数据时，比较原值是否修改，若未被其他线程修改则写回，若已被修改，则重新执行读取流程。

## 生产者消费者问题

使用ReentrantLock加锁，重点在await和signalAll方法的应用。

```java
// producer and consumer
public class ProducerAndConsumer {
    static class Producer implements Runnable {
        MyQueue queue;

        public Producer(MyQueue queue) {
            this.queue = queue;
        }

        @Override
        public void run() {
            for (int i = 0; i < 100; i++) {
                try {
                    Thread.sleep(100);
                    queue.putEle(i);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    static class Consumer implements Runnable {
        MyQueue queue;

        public Consumer(MyQueue queue) {
            this.queue = queue;
        }

        @Override
        public void run() {
            for (int i = 0; i < 100; i++) {
                try {
                    Thread.sleep(100);
                    queue.takeEle();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    static class MyQueue {
        Lock lock = new ReentrantLock();
        Condition prodCondition = lock.newCondition();
        Condition consumerCondition = lock.newCondition();

        final int CAPACITY = 10;
        Queue<Object> container = new LinkedList<>();
        int count = 0;

        public void putEle(Object ele) throws InterruptedException {
            try {
                lock.lock();
                while (count == CAPACITY) {
                    System.out.println("Producer waiting...");
                    prodCondition.await();
                }
                System.out.println("Produce a ele, num = " + ele);
                container.add(ele);
                count ++;
                consumerCondition.signalAll();
            } finally {
                lock.unlock();
            }
        }

        public Object takeEle() throws InterruptedException {
            try {
                lock.lock();
                while (count == 0) {
                    System.out.println("Consumer waiting...");
                    consumerCondition.await();
                }
                Object ele = container.poll();
                System.out.println("Consume a ele, num = " + ele);
                count --;
                prodCondition.signalAll();
                return ele;
            } finally {
                lock.unlock();
            }
        }
    }

    public static void main(String[] args) {
        MyQueue queue = new MyQueue();

        ExecutorService excutor = Executors.newFixedThreadPool(10);
        excutor.submit(new Producer(queue));
        excutor.submit(new Producer(queue));
        excutor.submit(new Consumer(queue));
    }
}
```

## 死锁问题

```java
// 死锁case
```



# 常见面试题

## synchronized 关键字

### 说一说自己对于 synchronized 关键字的了解

synchronized关键字解决的是多个线程之间访问资源的同步性，synchronized关键字可以保证被它修饰的方法或者代码块在任意时刻只能有一个线程执行。

另外，在 Java 早期版本中，synchronized属于重量级锁，效率低下，因为监视器锁（monitor）是依赖于底层的操作系统的 Mutex Lock 来实现的，Java 的线程是映射到操作系统的原生线程之上的。如果要挂起或者唤醒一个线程，都需要操作系统帮忙完成，而操作系统实现线程之间的切换时需要从用户态转换到内核态，这个状态之间的转换需要相对比较长的时间，时间成本相对较高，这也是为什么早期的 synchronized 效率低的原因。庆幸的是在 Java 6 之后 Java 官方对从 JVM 层面对synchronized 较大优化，所以现在的 synchronized 锁效率也优化得很不错了。JDK1.6对锁的实现引入了大量的优化，如自旋锁、适应性自旋锁、锁消除、锁粗化、偏向锁、轻量级锁等技术来减少锁操作的开销。

### ReentrantLock和synchronized的区别

synchronized是独占锁，加锁和解锁的过程自动进行，易于操作，但不够灵活。ReentrantLock也是独占锁，加锁和解锁的过程需要手动进行，不易操作，但非常灵活。

synchronized可重入，因为加锁和解锁自动进行，不必担心最后是否释放锁；ReentrantLock也可重入，但加锁和解锁需要手动进行，且次数需一样，否则其他线程无法获得锁。

synchronized不可响应中断，一个线程获取不到锁就一直等着；ReentrantLock可以相应中断。

ReentrantLock能实现公平锁，可使用tryLock实现限时等待。

# 参考文献

https://mp.weixin.qq.com/s/WDeewsvWUEBIuabvVVhweA

https://mp.weixin.qq.com/s/cdHfTTvMpH60SwG2bjTMBw

https://mp.weixin.qq.com/s/PrUa0tFyu3UZllP2FRDyVA

