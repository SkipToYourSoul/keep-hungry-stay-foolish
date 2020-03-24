## synchronized 关键字

### Q: 说一说自己对于 synchronized 关键字的了解

synchronized关键字解决的是多个线程之间访问资源的同步性，synchronized关键字可以保证被它修饰的方法或者代码块在任意时刻只能有一个线程执行。

另外，在 Java 早期版本中，synchronized属于重量级锁，效率低下，因为监视器锁（monitor）是依赖于底层的操作系统的 Mutex Lock 来实现的，Java 的线程是映射到操作系统的原生线程之上的。如果要挂起或者唤醒一个线程，都需要操作系统帮忙完成，而操作系统实现线程之间的切换时需要从用户态转换到内核态，这个状态之间的转换需要相对比较长的时间，时间成本相对较高，这也是为什么早期的 synchronized 效率低的原因。庆幸的是在 Java 6 之后 Java 官方对从 JVM 层面对synchronized 较大优化，所以现在的 synchronized 锁效率也优化得很不错了。JDK1.6对锁的实现引入了大量的优化，如自旋锁、适应性自旋锁、锁消除、锁粗化、偏向锁、轻量级锁等技术来减少锁操作的开销。

### Q: ReentrantLock和synchronized的区别

synchronized是独占锁，加锁和解锁的过程自动进行，易于操作，但不够灵活。ReentrantLock也是独占锁，加锁和解锁的过程需要手动进行，不易操作，但非常灵活。

synchronized可重入，因为加锁和解锁自动进行，不必担心最后是否释放锁；ReentrantLock也可重入，但加锁和解锁需要手动进行，且次数需一样，否则其他线程无法获得锁。

synchronized不可响应中断，一个线程获取不到锁就一直等着；ReentrantLock可以相应中断。

ReentrantLock能实现公平锁，可使用tryLock实现限时等待。

### Q：锁升级过程

偏向锁：无竞争条件下，线程将自己的指针更新到markword（Object对象前4个字节）中。偏向锁不一定能提高效率，在明确知道多个线程强烈竞争的时候，系统会把资源大量消耗在撤销上。

自旋锁：轻量级锁，每个线程在线程栈中生成LockRecord，用CAS方式尝试把自己的指针更新到markword。每一次锁重入，都会有一个LockRecord。自旋锁占用CPU。

重量级锁：操作系统内核加锁。自旋超过10次或者线程数大于CPU核数/2，升级为重量级锁（JVM 的Adaptive CAS会自适应调整）。

