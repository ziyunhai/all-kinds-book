[toc]
## Lock 接口
```
public interface Lock {
    void lock();
    void lockInterruptibly() throws InterruptedException;
    boolean tryLock();
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
    void unlock();
    Condition newCondition();
}
```
- lock()、tryLock()、tryLock(long time, TimeUnit unit)和lockInterruptibly()是用来获取锁的。unLock()方法是用来释放锁的

### lock()
- 平常使用得最多的一个方法，就是**用来获取锁。如果锁已被其他线程获取，则进行等待**
### tryLock
- tryLock()方法是有返回值的，它表示用来尝试获取锁，如果获取成功，则返回true，如果获取失败（即锁已被其他线程获取），则返回false，也就说这个方法无论如何都会立即返回。在拿不到锁时不会一直在那等待。
- tryLock(long time,TimeUnitunit)方法和tryLock()方法是类似的，只不过区别在于这个方法在拿不到锁时会等待一定的时间，在时间期限之内如果还拿不到锁，就返回false。如果如果一开始拿到锁或者在等待期间内拿到了锁，则返回true。

### unLock()

```
Lock lock = ...;
lock.lock();
try{
    //处理任务
}catch(Exception ex){
     
}finally{
    lock.unlock();   //释放锁
}
```
> 如果采用Lock，必须主动去释放锁，并且在发生异常时，不会自动释放锁。因此一般来说，使用Lock必须在try{}catch{}块中进行，并且将释放锁的操作放在finally块中进行，以保证锁一定被被释放，防止死锁的发生

### lockInterruptibly

## ReentrantLock

- ReentrantLock，重入锁，是JDK5中添加在并发包下的一个高性能的工具。顾名思义，ReentrantLock支持重入功能，即同一个线程在未释放锁的情况下可以重复获取锁，因此，重入的前提必须是同一个线程


![image-20201126162526648](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/26/16:25:27-image-20201126162526648.png)

相比于synchronized，ReentrantLock需要显式的获取锁和释放锁，相对现在基本都是用JDK7和JDK8的版本，ReentrantLock的效率和synchronized区别基本可以持平了。他们的主要区别有以下几点：

1. 等待可中断，当持有锁的线程长时间不释放锁的时候，等待中的线程可以选择放弃等待，转而处理其他的任务。
2. 公平锁：synchronized和ReentrantLock默认都是非公平锁，但是ReentrantLock可以通过构造函数传参改变。只不过使用公平锁的话会导致性能急剧下降。
3. 绑定多个条件：ReentrantLock可以同时绑定多个Condition条件对象,ReentrantLock基于AQS(**AbstractQueuedSynchronizer 抽象队列同步器**)实现

## ReadWriteLock

- ReadWriteLock也是一个接口
```
public interface ReadWriteLock {
    /**
     * Returns the lock used for reading.
     *
     * @return the lock used for reading
     */
    Lock readLock();

    /**
     * Returns the lock used for writing.
     *
     * @return the lock used for writing
     */
    Lock writeLock();
}
```
- 一个用来获取读锁，一个用来获取写锁。也就是说将文件的读写操作分开，分成2个锁来分配给线程，从而使得多个线程可以同时进行读操作。


## StampedLock
- StampedLock是Java8引入的一种新的所机制,简单的理解,可以认为它是读写锁的一个改进版本,读写锁虽然分离了读和写的功能,使得读与读之间可以完全并发,但是读和写之间依然是冲突的,读锁会完全阻塞写锁,它使用的依然是悲观的锁策略.如果有大量的读线程,他也有可能引起写线程的饥饿
- 而StampedLock则提供了一种乐观的读策略,这种乐观策略的锁非常类似于无锁的操作,使得乐观锁完全不会阻塞写线程
- StampedLock和ReadWriteLock相比，改进之处在于：读的过程中也允许获取写锁后写入！这样一来，我们读的数据就可能不一致，所以，**需要一点额外的代码来判断读的过程中是否有写入**，这种读锁是一种乐观锁。

### 详解
- 和ReadWriteLock相比，写入的加锁是完全一样的，不同的是读取。
- 注意到首先我们通过tryOptimisticRead()获取一个乐观读锁，并返回版本号。接着进行读取，读取完成后，我们通过validate()去验证版本号，如果在读取过程中没有写入，版本号不变，验证成功，我们就可以放心地继续后续操作。
- 如果在读取过程中有写入，版本号会发生变化，验证将失败。在失败的时候，我们再通过获取悲观读锁再次读取。由于写入的概率不高，程序在绝大部分情况下可以通过乐观读锁获取数据，极少数情况下使用悲观读锁获取数据。

## condition
## 锁优化
- 高效并发是从JDK1.5到JDK1.6的一个重要改进，HotSpot虚拟机开发团队在这个版本中花费了很大的精力去对Java中的锁进行优化，如适应性自旋、锁消除、锁粗化、轻量级锁和偏向锁等。这些技术都是为了在线程之间更高效的共享数据，以及解决竞争问题。
- 其实对于使用他的开发者来说是屏蔽掉了的，也就是说，作为一个Java开发，你只需要知道你想在加锁的时候使用synchronized就可以了，具体的锁的优化是虚拟机根据竞争情况自行决定的。
- 自Java 6/Java 7开始，Java虚拟机对内部锁的实现进行了一些优化。这些优化主要包括锁消除（Lock Elision）、锁粗化（Lock Coarsening）、偏向锁（BiasedLocking）以及适应性自旋锁（Adaptive Locking）。这些优化仅在Java虚拟机server模式下起作用（即运行Java程序时我们可能需要在命令行中指定Java虚拟机参数“-server”以开启这些优化）
### 锁分类
#### 可中断锁
- 可以相应中断的锁。
- 在Java中，synchronized就不是可中断锁，而Lock是可中断锁。
- 如果某一线程A正在执行锁中的代码，另一线程B正在等待获取该锁，可能由于等待时间过长，线程B不想等待了，想先处理其他事情，我们可以让它中断自己或者在别的线程中中断它，这种就是可中断锁

#### 可重入锁
- 如果锁具备可重入性，则称作为可重入锁。像synchronized和ReentrantLock都是可重入锁，可重入性在我看来实际上表明了锁的分配机制：**基于线程的分配，而不是基于方法调用的分配**
- 同一个线程在未释放锁的情况下可以重复获取锁，因此，重入的前提必须是同一个线程
> 举个简单的例子，当一个线程执行到某个synchronized方法时，比如说method1，而在method1中会调用另外一个synchronized方法method2，此时线程不必重新去申请锁，而是可以直接执行方法method2

```
class MyClass {
    public synchronized void method1() {
        method2();
    }
     
    public synchronized void method2() {
         
    }
}
```
- 述代码中的两个方法method1和method2都用synchronized修饰了，假如某一时刻，线程A执行到了method1，此时线程A获取了这个对象的锁，而由于method2也是synchronized方法，假如synchronized不具备可重入性，此时线程A需要重新申请锁。但是这就会造成一个问题，因为线程A已经持有了该对象的锁，而又在申请获取该对象的锁，这样就会线程A一直等待永远不会获取到的锁。
- 由于synchronized和Lock都具备可重入性，所以不会发生上述现象

#### 公平锁
- 公平锁即**尽量以请求锁的顺序来获取锁**。比如同是有多个线程在等待一个锁，当这个锁被释放时，等待时间最久的线程（最先请求的线程）会获得该所，这种就是公平锁(可通过队列实现)
- 非公平锁即无法保证锁的获取是按照请求锁的顺序进行的。这样就可能导致某个或者一些线程永远获取不到锁。
- 在Java中，synchronized就是非公平锁，它无法保证等待的线程获取锁的顺序。
- 而对于ReentrantLock和ReentrantReadWriteLock，它默认情况下是非公平锁，但是可以设置为公平锁。

#### 读写锁
- 读写锁将对一个资源（比如文件）的访问分成了2个锁，一个读锁和一个写锁。正因为有了读写锁，才使得多个线程之间的读操作不会发生冲突。

#### 自旋锁
##### 背景
- 在程序中，Java虚拟机的开发工程师们在分析过大量数据后发现：共享数据的锁定状态一般只会持续很短的一段时间，为了这段时间去挂起和恢复线程其实并不值得
##### 定义
- 如果物理机上有多个处理器，可以让多个线程同时执行的话。我们就可以让后面来的线程"稍微等一下"，但是并不放弃处理器的执行时间，看看持有锁的线程会不会很快释放锁。
- 这个"稍微等一下"的过程就是自旋。
##### 对比阻塞锁
- 自旋锁和阻塞锁最大的区别就是，到底要不要放弃处理器的执行时间。对于阻塞锁和自旋锁来说，都是要等待获得共享资源。但是阻塞锁是放弃了CPU时间，进入了等待区，等待被唤醒。而自旋锁是一直“自旋”在那里，时刻的检查共享资源是否可以被访问。
- 如果持有锁的线程很快就释放了锁，那么自旋的效率就非常好。但是如果持有锁的线程占用锁时间较长，等待锁的线程自旋一定次数后还是拿不到锁而被阻塞，那么自旋就白白浪费了CPU的资源。
所以自旋的次数直接决定了自旋锁的性能。**JDK自旋的默认次数为10次**，可以通过参数-XX:PreBlockSpin来调整。

##### 意义
- 由于自旋锁只是将当前线程不停地执行循环体，不进行线程状态的改变，所以响应速度更快。但当线程数不停增加时，性能下降明显，因为每个线程都需要执行，占用CPU时间。如果线程竞争不激烈，并且保持锁的时间短。适合使用自旋锁。

> 就像去银行办业务的例子，当你来到银行，发现柜台前面都有人的时候，你需要取一个号，然后再去等待区等待，一直等待被叫号。这个过程是比较浪费时间的，那么有没有什么办法改进呢？
有一种比较好的设计，那就是银行提供自动取款机，当你去银行取款的时候，你不需要取号，不需要去休息区等待叫号，你只需要找到一台取款机，排在其他人后面等待取款就行了

#### 适应自旋锁
- 所谓自适应就**意味着自旋的次数不再是固定的**，它是由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定。线程如果自旋成功了，那么下次自旋的次数会更加多，因为虚拟机认为既然上次成功了，那么此次自旋也很有可能会再次成功，那么它就会允许自旋等待持续的次数更多。
- 如果对于某个锁，很少有自旋能够成功的，那么在以后要或者这个锁的时候自旋的次数会减少甚至省略掉自旋过程，以免浪费处理器资源。
- 有了自适应自旋锁，随着程序运行和性能监控信息的不断完善，虚拟机对程序锁的状况预测会越来越准确，虚拟机会变得越来越聪明。

#### 偏向锁
- 大多数情况下锁不仅不存在多线程竞争，而且总是由同一个线程多次获取，所以引入偏向锁让线程获得锁的代价更低。
- 偏向锁认为环境中不存在竞争情况，锁只被一个线程持有，**一旦有不同的线程获取或竞争锁对象，偏向锁就升级为轻量级锁**。
- 偏向锁在无多线程竞争的情况下可以减少不必须要的轻量级锁执行路径。

#### 轻量级锁
- 在大多数情况下同步块并不会出现竞争情况，大部分情况是不同线程交替持有锁，所以引入轻量级锁可以减少重量级锁对线程的阻塞带来的开销。
- 轻量级锁认为环境中线程几乎没有对锁对象的竞争，**即使有竞争也只需要稍微等待（自旋）下就可以获取锁，但是自旋次数有限制，如果超过该次数，则会升级为重量级锁**。

#### 重量级锁
- 监视器锁Monitor

### 优化

#### 锁消除
- 锁消除，是JIT编译器对内部锁的具体实现所做的一种优化。
- 在动态编译同步块的时候，JIT编译器可以借助一种被称为逃逸分析（Escape Analysis）的技术来判断同步块所使用的锁对象是否只能够被一个线程访问而没有被发布到其他线程。
- 如果同步块所使用的锁对象通过这种分析被证实只能够被一个线程访问，那么JIT编译器在编译这个同步块的时候就会取消对这部分代码的同步。
> 锁优化，是JIT编译器的功能，所以，无法使用现有的反编译工具查看具体的优化结果
- 在使用synchronized的时候，如果JIT经过逃逸分析之后发现并无线程安全问题的话，就会做锁消除。

#### 锁粗化
> 在代码中，需要加锁的时候，我们提倡尽量减小锁的粒度，这样可以避免不必要的阻塞。这也是很多人原因是用同步代码块来代替同步方法的原因，因为往往他的粒度会更小一些，这其实是很有道理的。

- 可以说，大部分情况下，减小锁的粒度是很正确的做法，只有一种特殊的情况下，会发生一种叫做锁粗化的优化。

> 就像你去银行办业务，你为了减少每次办理业务的时间，你把要办的五个业务分成五次去办理，这反而适得其反了。因为这平白的增加了很多你重新取号、排队、被唤醒的时间。

- 如果在一段代码中连续的对同一个对象反复加锁解锁，其实是相对耗费资源的，这种情况可以适当放宽加锁的范围，减少性能消耗。
- 当JIT发现一系列连续的操作都对同一个对象反复加锁和解锁，甚至加锁操作出现在循环体中的时候，会将加锁同步的范围扩散（粗化）到整个操作序列的外部。

```
for(int i=0;i<100000;i++){  
   synchronized(this){  
       do();  
}
```
```
synchronized(this){  
   for(int i=0;i<100000;i++){  
       do();  
}
```

#### 锁的膨胀过程
- synchronized锁膨胀过程就是无锁 → 偏向锁 → 轻量级锁 → 重量级锁的一个过程。这个过程是随着多线程对锁的竞争越来越激烈，锁逐渐升级膨胀的过程。
- **偏向锁、轻量级锁、重量级锁三种形式，分别对应了锁只被一个线程持有、不同线程交替持有锁、多线程竞争锁三种情况**

##### 详细过程
###### 无锁
- 一个锁对象刚刚开始创建的时候，没有任何线程来访问它，此时线程状态为无锁状态。Mark word（锁标志位-01 是否偏向-0）

###### 偏向锁
- 当线程A来访问这个对象锁时，它会偏向这个线程A。线程A检查Mark word（锁标志位-01 是否偏向-0）为无锁状态。此时，有线程访问锁了，无锁升级为偏向锁，Mark word（锁标志位-01，是否偏向-1，线程ID-线程A的ID）
- 当线程A执行完同步块时，不会主动释放偏向锁。持有偏向锁的线程执行完同步代码后不会主动释放偏向锁，而是等待其他线程来竞争才会释放锁。Mark word不变（锁标志位-01，是否偏向-1，线程ID-线程A的ID）
- 当线程A再次获取这个对象锁时，检查Mark word（锁标志位-01，是否偏向-1，线程ID-线程A的ID），偏向锁且偏向线程A，可以直接执行同步代码。这样偏向锁保证了总是同一个线程多次获取锁的情况下，每次只需要检查标志位就行，效率很高。

###### 轻量级锁
- 当线程A执行完同步块之后，线程B获取这个对象锁 检查Mark word（锁标志位-01，是否偏向-1，线程ID-线程A的ID），偏向锁且偏向线程A。有不同的线程获取锁对象，偏向锁升级为轻量级锁，并由线程B获取该锁。

###### 重量级锁


