# 线程

## 并行和并发有什么区别？

## 线程与进程的区别

一个进程是一个独立(self contained)的运行环境，它可以被看作一个程序或者一个应用。而线程是在进程中执行的一个任务。Java运行环境是一个包含了不同的类和程序的单一进程。线程可以被称为轻量级进程。线程需要较少的资源来创建和驻留在进程中，并且可以共享进程中的资源。

## 多线程编程的好处是什么？

发挥多核CPU的优势。减少上下文切换，充分使用CPU。

防止阻塞。多线程并发执行可以防止单线程等待资源的阻塞，提高效率。

任务分解。多个线程共享堆内存(heap memory)，因此创建多个线程去执行一些任务会比创建多个进程更好。举个例子，Servlets比CGI更好，是因为Servlets支持多线程而CGI不支持。

## java的线程调度算法是什么？

抢占式。一个线程用完CPU之后，操作系统会根据线程优先级、线程饥饿情况等数据算出一个总的优先级并分配下一个时间片给某个线程执行。

## 创建线程有哪几种方式？

##  runnable 和 callable 有什么区别？

## Thread.sleep(0)的作用是什么

由于Java采用抢占式的线程调度算法，因此可能会出现某条线程常常获取到CPU控制权的情况。为了让某些优先级比较低的线程也能获取到CPU控制权，可以使用`Thread.sleep(0)`手动触发一次操作系统分配时间片的操作，这也是平衡CPU控制权的一种操作。 

## 用户线程和守护线程有什么区别？

当我们在Java程序中创建一个线程，它就被称为用户线程。一个守护线程是在后台执行并且不会阻止JVM终止的线程。当没有用户线程在运行的时候，JVM关闭程序并且退出。一个守护线程创建的子线程依然是守护线程。

## 



## 在多线程中，什么是上下文切换？

上下文切换是指：CPU的控制权因为线程切换而发生转移，寄存器清空原线程缓存的数据。

## 线程的状态有哪些？

新建，运行，阻塞，等待，计时等待，结束。

## 线程等待和阻塞的区别是什么？

waiting等待状态是线程在synchronized代码块之外主动进入。

blocked阻塞状态是线程在synchronized代码块之内被动进入。

## 线程同步与阻塞的关系是什么？

线程同步与否 跟 阻塞非阻塞没关系，同步是个过程，阻塞是线程的一种状态。多个线程操作共享变量时可能会出现竞争。这时需要同步来防止两个以上的线程同时进入临界区内，在这个过程中后进入临界区的线程将阻塞，等待先进入的线程走出临界区。

## 怎么唤醒一个阻塞的线程

如果线程是因为调用了wait()、sleep()或者join()方法而导致的阻塞，可以中断线程，并且通过抛出InterruptedException来唤醒它；如果线程遇到了IO阻塞，无能为力，因为IO是操作系统实现的，Java代码并没有办法直接接触到操作系统。

## 同步和异步有什么区别？

同步和异步最大的区别就在于。一个需要等待，一个不需要等待。同步可以避免出现死锁，读脏数据的发生，一般共享某一资源的时候用，如果每个人都有修改权限，同时修改一个文件，有可能使一个人读取另一个人已经删除的内容，就会出错，同步就会按顺序来修改。

## sleep和wait方法的相同点与区别是什么？

相同点是：都暂停了线程的执行。

区别是：

1. Thread类的sleep()方法不会释放锁，线程进入计时等待状态。
1. Object类的wait()方法会释放锁，并进入对象的等待队列，线程进入等待状态。notify()方法唤醒后进入运行状态。

## notify()和 notifyAll()有什么区别？

notify()唤醒持有该监视器锁的线程，notifyAll()唤醒所有线程。

## 线程的使用方法有哪些？

Runnable,Callable,Thread,Excutor.

execute方法不返回值。submit方法返回值。shutdown()中断无任务线程。shuodownNow()中断所有线程。isInterupted()方法返回中断信息。isShutdown()都返回true。线程池完全关闭后，isTerminated()返回true.

## 什么是多线程环境下的伪共享（falsesharing）？ 

伪共享是多线程系统（每个处理器有自己的局部缓存）中一个众所周知的性能问题。缓存系统中是以缓存行（cache line）为单位存储的。缓存行是2的整数幂个连续字节，一般为32-256个字节。最常见的缓存行大小是64个字节。当多线程修改互相独立的变量时，如果这些变量共享同一个缓存行，就会无意中影响彼此的性能，这就是伪共享。

## Runnable接口和Callable接口的区别

Runnable接口中的run()方法的返回值是void，它做的事情只是纯粹地去执行run()方法中的代码而已。

Callable接口中的call()方法是有返回值的，是一个泛型，和Future、FutureTask配合可以用来获取异步执行的结果。

## 什么地方用到了多线程？

## 什么是线程安全？

如果你的代码在多线程下执行和在单线程下执行永远都能获得一样的结果，那么你的代码就是线程安全的。

## 线程安全的类有哪些？

不可变类，比如String, Integer, Long等。

线程安全的容器类，比如ConcurrentHashMap, ConcurrentSkipListMap,  ConcurrentHashSet, CopyOnWriteArrayList, CopyOnWriteHashSet, SynchronousQueue。

线程相对安全的容器类，比如Vector, Collections.synchronized()方法修饰的类，遍历时改变，会按照fail-fast机制抛出ConcurrentModificationException。

## 如何在线程之间共享数据？

继承Thread类，添加共享的成员对象，通过wait/notify/notifyAll、await/signal/signalAll进行唤起和等待。

使用阻塞队列。

## java内存模型是什么

分为主内存和工作内存。定义了原子操作，volatile变量使用规则，happens-before规则。

## 写一个生产者消费者模式。



# 锁和锁优化

## 在 java 程序中怎么保证多线程的运行安全？

## synchronized的底层实现原理是什么？

JVM对于同步方法和同步代码块的处理方式不同。

同步方法通过`ACC_SYNCHRONIZED`关键字隐式的对方法进行加锁。当线程要执行的方法被标注上`ACC_SYNCHRONIZED`时，需要先获得监视器锁才能执行该方法。

同步代码块通过`monitorenter`和`monitorexit`执行来进行加锁。当线程执行到`monitorenter`的时候要先获得所锁，才能执行后面的方法。当线程执行到`monitorexit`的时候则要释放锁。

每个对象自身维护这一个被加锁次数的计数器，当计数器数字为0时表示可以被任意线程获得锁。当计数器不为0时，只有获得锁的线程才能再次获得锁。即可重入锁。

## 同步方法和同步块，哪个是更好的选择

一般是同步块，因为同步的范围小，同步块之外的代码异步执行，效率高于同步方法。

同步块要放在循环体外面，让虚拟机实现锁粗化的优化。消除重复加锁解锁，避免用户态和内核态的切换，可以提高代码执行的效率。

## synchronized和ReentrantLock的区别

synchronized是关键字，ReentrantLock是类，功能多。

synchronized通过字节码指令加锁，ReentrantLock通过Unsafe。park()方法加锁。

ReentrantLock可以对获取锁的等待时间进行设置，这样就避免了死锁。

ReentrantLock可以获取各种锁的信息。

ReentrantLock结合Conditional可以实现多路通知。

ReentrantLock可以抛出异常。

ReentrantLock必须手动释放锁。







## 线程出现异常会怎样？

线程停止运行，自动释放监视器锁。

## 如何判断线程持有对象监视器锁？

当前线程调用`Thread.holdsLock()`方法。

## java对象头信息是什么？

对象头中包含锁状态标志、线程持有的锁等标志。



这个问题会接着追问：java对象头信息，偏向锁，轻量锁，重量级锁及其他们相互间转化。

## 锁升级的过程是什么？

从偏向锁，到轻量级锁，再到重量级锁。在并发的时候，会发生锁升级。

## 偏向锁和轻量级锁是怎么实现的？

## 公平锁和非公平锁的区别？

公平锁按先后顺序加锁，非公平锁是抢占式加锁。

## 什么是乐观锁和悲观锁

（1）乐观锁：就像它的名字一样，对于并发间操作产生的线程安全问题持乐观状态，乐观锁认为竞争不总是会发生，因此它不需要持有锁，将比较-替换这两个动作作为一个原子操作尝试去修改内存中的变量，如果失败则表示发生冲突，那么就应该有相应的重试逻辑。

（2）悲观锁：还是像它的名字一样，对于并发间操作产生的线程安全问题持悲观状态，悲观锁认为竞争总是会发生，因此每次对某资源进行操作时，都会持有一个独占的锁，就像synchronized，不管三七二十一，直接上了锁就操作资源了。

## ReadWriteLock读写之间互斥吗？

读写互斥。

## ReadWriteLock互斥锁和同步锁的是怎么实现的？



## 锁优化策略有哪些？

锁优化包括：(自适应)自旋锁、锁消除、锁粗化、轻量级锁、偏向锁。

无竞争的情况下偏向锁去掉所有同步。

轻量级锁在栈帧中创建锁记录空间，用于存储锁对象的Mark Word副本。然后将对象的Mark Word被CAS更新为指向锁记录的指针，最后修改Mark Word副本的最后2位为00,则处于轻量级锁定的状态，用栈区操作代替同步原语。

## 什么是死锁？

## 什么情况下会产生死锁？

1. 互斥条件（进程独占资源）

2. 请求与保持（进程因请求资源而阻塞时，对已获得的资源保持不放） 

3. 不剥夺条件（进程已获得的资源，在末使用完之前，不能强行剥夺） 

4. 循环等待（若干进程之间形成一种头尾相接的循环等待资源关系）

## 死锁如何排查，以及解决方案？

死锁是指两个以上的线程永远阻塞的情况，这种情况产生至少需要两个以上的线程和两个以上的资源。

分析死锁，我们需要查看Java应用程序的线程转储。我们需要找出那些状态为BLOCKED的线程和他们等待的资源。每个资源都有一个唯一的id，用这个id我们可以找出哪些线程已经拥有了它的对象锁。

嵌套锁按顺序加锁，反向释放。只在需要的地方使用锁和避免无限期等待是避免死锁的通常办法。

## 写一个死锁。

```
static class BadLock implements Runnable {
    Integer a,b;
    public BadLock(Integer a, Integer b){
        this.a=a;
        this.b=b;
    }
    
    @Override
    public void run(){
        synchronized(a){
            synchronized(b){
                System.out.println("ok");
            }
        }
    }
}
//线程多一点才容易看出来
public static void main(String[] args){
    for(int i=0;i<100;i++){
        new Thread(new BadLock(1,2)).start();
        new Thread(new BadLock(2,1)).start();
    }
}
```



# 线程池

## 为什么要用线程池？

线程复用，防止生成过多的线程对象，导致fullgc或者oom。

还可以控制并发数量。

## 线程池的参数有哪些？

参数包括：核心线程数，最大线程数，空闲时间和单位，阻塞队列，线程池工厂类，拒绝策略。

普通线程池通过ThreadPoolExecutor创建线程，按照线程数增加顺序，依次受影响的参数是：核心线程数、阻塞队列、空闲时间、最大线程数、拒绝策略。

塞队列包括ArrayBlockingQueue,LinkedBlockingQueue,SynchronousQueue.

拒绝策略包括AbortPolicy, DiscardOldestPolicy, DiscardPolicy, CallerRunsPolicy。

## 线程池处理提交任务的流程是什么？

1. 判断核心线程数没有达到上限，就创建新的核心线程。
1. 满了之后，将任务放到工作队列。
1. 工作队列满了之后，如果最大线程数没满，就

## 创建线程池有哪几种方式？

## 线程池都有哪些状态？

## 线程池中 submit()和 execute()方法有什么区别？

## JDK实现的线程池有哪几种类型？

jdk实现的线程池包括：FixedThreadPool(核心线程数与), SingleThreadPool, CachedThreadPool以及ShedualedThreadPool,SingleShedualedThreadPool.

FixedThreadPool的核心线程数与最大线程数相等，空闲时间为0, 阻塞队列为LinkedBlockingQueue, 容量为最大整数，那么，参数中的最大线程数、存活时间和拒绝策略都失效。如果核心线程数为1,就是SingleThreadPool。

CachedThreadPool的核心线程数为0,最大线程数为最大整数，空闲时间60s, 阻塞队列为SynchronousQueue。如果提交任务的速度超过消费任务的速度，线程池就会创建新的线程，空闲时间后被回收前，有OOM的风险。

定时线程池ScheduledThreadPoolExecutor主要用于任务的延时执行和定期执行。与普通线程池不同的是：使用无界阻塞队列DelayQueue，那么，空闲时间、最大线程数和拒绝策略都失效。从延时队列获取到期的周期任务。执行完后修改时间并再次入队。scheduleAtFixedRate()和scheduleWithFixedDelay()会向DelayQueue添加ScheduledFutureTask(包含3个long类型的成员变量：time, sequenceNumber, period), DelayQueue封装的优先队列对任务排序(时间早和序列号小的依次优先)。

## 如何确定线程池的核心线程数？

CPU密集型的重负载任务，核心线程数推荐为核数+1. IO密集型的轻负载任务，核心线程数推荐为核数×2. 有优先级需求的任务，可以配置优先阻塞队列。任务执行时间差距过大，可以使用多个线程池，甚至多台机器。执行时间长，适当增加核心线程数。尽可能使用有界队列，防止内存用尽，还可以通过配置饱和抛弃策略返回异常。

## 高并发、任务执行时间短的业务怎样使用线程池？并发不高、任务执行时间长的业务怎样使用线程池？并发高、业务执行时间长的业务怎样使用线程池？

1. 高并发、任务执行时间短。

   参考CacheThreadPool，核心线程数设置为CPU核数*2，最大线程数n倍核心数。**阻塞队列**为ArrayBlockingQueue。等待时间为零。

1. 并发不高、任务执行时间长。

   IO密集型任务，**核心线程数**设置为CPU核数*2，最大线程数为n倍核心数。**阻塞队列**为LinkedBlockingQueue。

   计算密集型任务，**核心线程数**设置为CPU核数+1，最大线程数为1.5倍核心数，阻塞队列为SynchronousQueue。

1. 并发高、业务执行时间长。

   参考FixedThreadPool，设置核心线程数和最大线程数设置为2倍核心数，**阻塞队列**为零容量的SynchronousQueue，执行AbortPolicy的**拒绝策略**。

## Executor框架包括哪些接口？

任务接口Runnable和Callable, 执行接口Executor和子接口ExecutorService(关键实现类有ThreadPoolExecutor和ScheduledThreadPoolExecutor), 异步计算结果接口Future(实现类为FutureTask, CompletableFuture)

# 并发包



## 并发工具类有哪些？

CountDownLatch, CyclicBarrier, Semaphore可以控制并发流程，Exchanger在线程间交换数据。

## CountDownLatch的用法?

CountDownLatch是倒计数闩，用于对线程计数。

## CyclicBarrier和CountDownLatch有什么区别？

（1）CyclicBarrier的某个线程运行到某个点上之后，该线程即停止运行，直到所有的线程都到达了这个点，所有线程才重新运行；CountDownLatch则不是，某线程运行到某个点上之后，数值减1，该线程继续运行。

（2）CyclicBarrier只能唤起一个任务，CountDownLatch可以唤起多个任务

（3）CyclicBarrier可重用，CountDownLatch不可重用，计数值为0该CountDownLatch就不可再用了 。

## Semaphore的用法？

Semaphore就是一个信号量，它的作用是限制某段代码块的并发数。Semaphore有一个构造函数，可以传入一个int型整数n，表示某段代码最多只有n个线程可以访问，如果超出了n，那么请等待，等到某个线程执行完毕这段代码块，下一个线程再进入。由此可以看出如果Semaphore构造函数中传入的int型整数n=1，相当于变成了一个synchronized了。

## Semaphore拿到执行权的线程之间是否互斥？

Semaphore可以有多把锁，可以允许线程同时拥有执行权，如果并发访问对象，就会产生线程安全问题。

## 写一个你认为最好的单例模式。

可以保证线程安全的有：枚举，静态内部类，双重校验锁。校验锁的判断参数最好加上volatile关键字。

# ThreadLocal

## ThreadLocal 用途是什么，原理是什么，用的时候要注意什么？

ThreadLocal用于并发读写。在每个Thread里面维护了一个以开地址法实现的ThreadLocal.ThreadLocalMap，为线程提供本地副本，线程间数据隔离不共享，解决了线程安全问题。

线程池的线程，用完后要手动取消ThreadLocal。

## AQS的原理是什么？

AQS队列同步器锁是用于构建锁和其他同步组件的基础抽象类，内部使用int类型的state变量表示同步状态，通过内置的双向链表实现的FIFO同步队列来完成获取线程的排队工作。对开发者屏蔽了同步状态管理、线程排队、等待与唤醒等底层操作。重写tryLock和tryRelease方法可以实现并发功能。

独占式获取同步状态成功返回true. 线程获取同步状态失败后，加上等待状态等信息封装成Node节点，加入同步队列。自旋检查前驱节点为头节点时，尝试获取同步状态。获取成功后把自己设置为头节点，返回。如果前驱节点的同步状态为唤醒，就阻塞当前线程(LockSupport.park(this))。否则依次去掉大于0的非正常等待状态前驱节点，CAS设置正常等待状态前驱节点的等待状态为唤醒。同步状态释放后，后继节点（同步队列首节点）被唤醒。

共享式获取同步状态返回int类型，大于等于0表示能获取同步状态。 独占式超时获取同步状态成功返回true, 可以响应中断。

## CAS的原理是什么？

比较并替换CAS(Compare and Swap)，假设有三个操作数：内存值V、旧的预期值A、要修改的值B，当且仅当预期值A和内存值V相同时，才会将内存值修改为B并返回true，否则什么都不做并返回false，整个比较并替换的操作是一个原子操作。CAS一定要volatile变量配合，这样才能保证每次拿到的变量是主内存中最新的相应值，否则旧的预期值A对某条线程来说，永远是一个不会变的值A，只要某次CAS操作失败，下面永远都不可能成功。

CAS虽然比较高效的解决了原子操作问题，但仍存在三大问题。循环时间长开销很大。只能保证一个共享变量的原子操作。ABA问题。

## Lock接口的锁与syncronized关键字的区别是什么？

Lock接口的实现类有：ReentrantLock, ReentrantReadWriteLock，同步线程要显式获取和释放锁，比synchronized关键字多的功能包括锁的尝试非阻塞获取、可中断获取、超时获取等获取/释放的操作。读写锁的同步状态通过int变量维护，高/低16位表示读/写锁的获取线程数，写锁降级是在释放写锁前先获取读锁。

Condition是同步器的内部类，由单向链表实现等待队列。await()等待方法使当前线程释放锁，同步队列的首节点被构造成等待节点加入等待队列，线程处于等待状态。signal()唤醒方法尝试获取同步状态，获取锁后把等待队列的首节点移动到同步队列，然后唤醒该节点的线程。

syncronized关键字只有1个同步队列和1个等待队列（Lock实现类可以有多个等待队列）。

# 原子类

## 原子类的类型有哪些？

原子类更新目标有：基本类型原子类、数组原子类、对象原子类(可以有时间戳)、属性原子类。

## LongAddr是怎样实现高并发的？

分段锁，副本。

# volatile

## volatile关键字的语义是什么？

volatile表示变量不稳定，需要直接读写主内存，具有线程可见性和单变量的原子性。StoreLoad内存屏障把工作内存的数据刷入主内存，其他线程直接从主内存读取，写后读禁止指令重排，保证了线程读取的可见性。加上CAS的写入的原子性，可以保证读写的原子性。

修饰的long/double变量按原子类型来读写，用于状态标记。多线程环境下volatile方式的双重校验等。

volatile是如何保证线程可见性的？

volatile编译成机器后增加了一个带有lock前缀的操作，相当于内存屏障，对cpu总线加锁后把新值写入内存，并引起其他核心的缓存失效。这样可以保证此变量对所有线程的可见性。

写入内存前的保证对后续代码有依赖的操作执行完成，就可以达到防止指令重排的效果。

## 如何实现线程的原子性、可见性和有序性？

原子性可以通过synchronized或者原子类实现。

可见性可以通过synchronized, volatile和final实现。

有序性可以通过synchronized和volatile实现。

## 能创建volatile数组吗？

能创建，volatile保证不改变数组引用，但数组元素可能改变。

## volatile 能使得一个非原子操作变成原子操作吗？

## java读取 long/double 类型的变量不是原子的？

不是原子的，前后32位需要分成两步读写。如果一个线程写入了一半，
另一个线程就可能读取到不可描述的值。

volatile 型的 long 或 double 变量的读写是原子。

但由于商用虚拟机几乎都要实现为原子操作，那么就不用加volatile关键字了。

## happens-before先行发生规则是什么？

程序次序规则

加锁解锁规则

volatile规则

线程启动规则

线程终止规则

线异常规则

对象终结规则

传递性。



## 了解FutureTask吗？

FutureTask继承AQS, 实现了Future和Runnable接口。3种状态包括未启动、已启动和已完成（正常结束或者取消，立即返回结果）。

## synchronized 和 volatile 的区别是什么？

# 设计

## 多线程并发编程的最佳实践，你是怎么用的？

使用本地变量和不可变类。

减小锁的作用域。

使用线程池的Executor框架创建线程。

使用同步，不使用wait/notify。

使用BlockingQueue实现生产者-消费者模式。

使用并发集合，不使用加锁的集合。

使用Semaphore实现有界的访问。

使用同步代码块，不使用同步方法。

避免使用静态变量。