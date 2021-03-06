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

CAS （Compare and Swap）是乐观锁的一种实现方式，是一种轻量级锁，CAS 操作的流程如下，线程在读取数据时不进行加锁，在准备写回数据时，比较原值是否修改，若未被其他线程修改则写回，若已被修改，则重新执行读取流程。

```java
// 这里内部使用CAS实现
AtomicBoolean flag = new AtomicBoolean(true);
flag.comnpreAndSet(true, false);
```

> 更多锁相关的介绍可参考：https://mp.weixin.qq.com/s/hX3RJhJxyy7w0ADMcLlWYg

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
public class DeadLock {
    public static String o1 = "o1";
    public static String o2 = "o2";

    public static void main(String[] args) {
        new Thread(new LockA()).start();
        new Thread(new LockB()).start();
    }
}

class LockA implements Runnable {
    @Override
    public void run() {
        while (true) {
            synchronized (DeadLock.o1) {
                try {
                    System.out.println("LockA lock o1");
                    Thread.sleep(1000L);
                    synchronized (DeadLock.o2) {
                        System.out.println("LockA lock o2");
                        Thread.sleep(60 * 1000);
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

class LockB implements Runnable {
    @Override
    public void run() {
        while (true) {
            synchronized (DeadLock.o2) {
                try {
                    System.out.println("LockB lock o2");
                    Thread.sleep(1000L);
                    synchronized (DeadLock.o1) {
                        System.out.println("LockA lock o1");
                        Thread.sleep(60 * 1000);
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

## 线程池创建参考

```java
// Guava ThreadFactoryBuilder
ThreadFactory namedThreadFactory = new ThreadFactoryBuilder()
	.setNameFormat("demo-pool-%d").build();

//Common Thread Pool
/* corePoolSize：池中的线程数，即使其处于IDLE状态
maximumPoolSize：池中允许的最大线程数
keepAliveTime：当线程数大于核心时，空闲线程在终止之前等待新任务的最长时间
unit：keepAliveTime的时间单位
workQueue：队列，用于在执行task之前保存task
handler：当达到了线程边界和队列容量，无法及时处理时，reject task使用的处理程序
*/
ExecutorService pool = new ThreadPoolExecutor(5, 200,
	0L, TimeUnit.MILLISECONDS,
  new LinkedBlockingQueue<Runnable>(1024), namedThreadFactory, new 
  ThreadPoolExecutor.AbortPolicy());

pool.execute(()-> System.out.println(Thread.currentThread().getName()));
pool.shutdown();//gracefully shutdown
```

可以简单这样理解线程池：

1.如果正在运行少于corePoolSize的线程，尝试使用给定命令启动新线程。调用addWorker时会以原子方式检查runState和workerCount，通过返回false来防止在不应该添加线程时添加线程。（如果未达到maximumPoolSize，那么会扩容？）

2.如果任务可以成功进入队列，那么我们仍然需要仔细检查是否应该添加一个线程（因为自上次检查后现有的线程已经死亡），或者自从进入此方法后池关闭了。 所以我们重新检查状态，如果线程停止了那么将任务回滚进入其他线程，或者如果没有其他线程，则启动新的线程。

3.如果我们不能将任务入队，那么我们尝试添加一个新线程。 如果失败，我们知道我们已关闭或饱和，因此拒绝该任务。

workQueue是一个BlockingQueue，也就是说当前线程数小于corePoolSize，那么就启用新的线程，如果大于corePoolSize（如果有扩容机制，那么就是maximumPoolSize），那么就放到这个workQueue里面（workQueue.offer）

# 参考文献

https://mp.weixin.qq.com/s/WDeewsvWUEBIuabvVVhweA

https://mp.weixin.qq.com/s/cdHfTTvMpH60SwG2bjTMBw

https://mp.weixin.qq.com/s/PrUa0tFyu3UZllP2FRDyVA

