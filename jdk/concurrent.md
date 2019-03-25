# 线程

## 线程的状态有哪些？

新建，运行，阻塞，等待，计时等待，结束。

## 线程等待和阻塞的区别是什么？

waiting等待状态是线程在synchronized代码块之外主动进入。

blocked阻塞状态是线程在synchronized代码块之内被动进入。

## sleep和wait方法的相同点与区别是什么？

相同点是：都暂停了线程的执行。

区别是：Thread类的sleep()方法不会释放锁，线程进入计时等待状态。Object类的wait()方法会释放锁，并进入对象的等待队列，线程进入等待状态。notify()方法唤醒后进入运行状态。

## 线程的使用方法有哪些？

Runnable,Callable,Thread,Excutor.

execute方法不返回值。submit方法返回值。shutdown()中断无任务线程。shuodownNow()中断所有线程。isInterupted()方法返回中断信息。isShutdown()都返回true。线程池完全关闭后，isTerminated()返回true.

# 线程池

## 线程池的参数有哪些？

参数包括：核心线程数，最大线程数，空闲时间和单位，阻塞队列，线程池工厂类，拒绝策略。

普通线程池通过ThreadPoolExecutor创建线程，按照线程数增加顺序，依次受影响的参数是：核心线程数、阻塞队列、空闲时间、最大线程数、拒绝策略。

阻塞队列包括ArrayBlockingQueue,LinkedBlockingQueue,SynchronousQueue.

拒绝策略包括AbortPolicy, DiscardOldestPolicy, DiscardPolicy, CallerRunsPolicy。

## JDK实现的线程池有哪几种类型？

jdk实现的线程池包括：FixedThreadPool(核心线程数与), SingleThreadPool, CachedThreadPool以及ShedualedThreadPool,SingleShedualedThreadPool.

FixedThreadPool的核心线程数与最大线程数相等，空闲时间为0, 阻塞队列为LinkedBlockingQueue, 容量为最大整数，那么，参数中的最大线程数、存活时间和拒绝策略都失效。如果核心线程数为1,就是SingleThreadPool。

CachedThreadPool的核心线程数为0,最大线程数为最大整数，空闲时间60s, 阻塞队列为SynchronousQueue。如果提交任务的速度超过消费任务的速度，线程池就会创建新的线程，空闲时间后被回收前，有OOM的风险。

定时线程池ScheduledThreadPoolExecutor主要用于任务的延时执行和定期执行。与普通线程池不同的是：使用无界阻塞队列DelayQueue，那么，空闲时间、最大线程数和拒绝策略都失效。从延时队列获取到期的周期任务。执行完后修改时间并再次入队。scheduleAtFixedRate()和scheduleWithFixedDelay()会向DelayQueue添加ScheduledFutureTask(包含3个long类型的成员变量：time, sequenceNumber, period), DelayQueue封装的优先队列对任务排序(时间早和序列号小的依次优先)。

## 如何确定线程池的核心线程数？

CPU密集型的重负载任务，核心线程数推荐为核数+1. IO密集型的轻负载任务，核心线程数推荐为核数×2. 有优先级需求的任务，可以配置优先阻塞队列。任务执行时间差距过大，可以使用多个线程池，甚至多台机器。执行时间长，适当增加核心线程数。尽可能使用有界队列，防止内存用尽，还可以通过配置饱和抛弃策略返回异常。

## Executor框架包括哪些接口？

任务接口Runnable和Callable, 执行接口Executor和子接口ExecutorService(关键实现类有ThreadPoolExecutor和ScheduledThreadPoolExecutor), 异步计算结果接口Future(实现类为FutureTask, CompletableFuture)

## CountDownLatch的用法?

CountDownLatch是倒计数闩，用于对线程计数。

## 并发工具类有哪些？

CountDownLatch, CyclicBarrier, Semaphore可以控制并发流程，Exchanger在线程间交换数据。

## CyclicBarrier和CountDownLatch的原理和用法有什么不同？

CyclicBarrier循环栅栏正向计数，饱和后清零，可以对线程循环计数。

CountDownLatch倒计数闩反向计数，到零后失效，只能对线程计数一次。

## Semaphore的用法？

获取锁，约束线程数，控制并发。

# ThreadLocal

## ThreadLocal 用途是什么，原理是什么，用的时候要注意什么？

用于对象的线程并发读写，为线程提供本地副本。用的时候要注意键是线程对象的情况下，不能用线程池。

## AQS的原理和工作过程是什么？

AQS队列同步器锁是用于构建锁和其他同步组件的基础抽象类，内部使用int类型的state变量表示同步状态，通过内置的双向链表实现的FIFO同步队列来完成获取线程的排队工作。对开发者屏蔽了同步状态管理、线程排队、等待与唤醒等底层操作。

独占式获取同步状态成功返回true. 线程获取同步状态失败后，加上等待状态等信息封装成Node节点，加入同步队列。自旋检查前驱节点为头节点时，尝试获取同步状态。获取成功后把自己设置为头节点，返回。如果前驱节点的同步状态为唤醒，就阻塞当前线程(LockSupport.park(this))。否则依次去掉大于0的非正常等待状态前驱节点，CAS设置正常等待状态前驱节点的等待状态为唤醒。同步状态释放后，后继节点（同步队列首节点）被唤醒。

共享式获取同步状态返回int类型，大于等于0表示能获取同步状态。 独占式超时获取同步状态成功返回true, 可以响应中断。

## Lock接口的锁与syncronized关键字的区别是什么？

Lock接口的实现类有：ReentrantLock, ReentrantReadWriteLock，同步线程要显式获取和释放锁，比synchronized关键字多的功能包括锁的尝试非阻塞获取、可中断获取、超时获取等获取/释放的操作。读写锁的同步状态通过int变量维护，高/低16位表示读/写锁的获取线程数，写锁降级是在释放写锁前先获取读锁。

Condition是同步器的内部类，由单向链表实现等待队列。await()等待方法使当前线程释放锁，同步队列的首节点被构造成等待节点加入等待队列，线程处于等待状态。signal()唤醒方法尝试获取同步状态，获取锁后把等待队列的首节点移动到同步队列，然后唤醒该节点的线程。

syncronized关键字只有1个同步队列和1个等待队列（Lock实现类可以有多个等待队列）。

## ConcurrentLinkedQueue的原理是什么？

ConcurrentLinkedQueue使用非阻塞方法实现线程安全。入队时经过不超过HOPS次循环定位到实际尾节点，否则重新进入循环，减少与cas更新线程的竞争。然后将入队节点cas设置为实际尾节点，超过HOPS次循环后，将入队节点cas更新为tail尾节点，这样减少更新次数。出队时首先获取头节点的元素，非空则经过HOPS次循环后，cas设置头节点的元素引用为空，并返回元素值。否则说明已经出队，重新获取next节点，直到next节点为空，将最后的节点设置为头节点。

## 原子类的类型有哪些？

原子类更新目标有：基本类型原子类、数组原子类、对象原子类(可以有时间戳)、属性原子类。

## LongAddr是怎样实现高并发的？

分段锁，副本。

# volatile

## volatile关键字的语义是什么？

volatile表示变量不稳定，需要直接读写主内存，具有线程可见性和单变量的原子性。StoreLoad内存屏障把工作内存的数据刷入主内存，其他线程直接从主内存读取，写后读禁止指令重排，保证了线程可见性。

修饰的long/double变量按原子类型来读写，用于状态标记。多线程环境下volatile方式的双重校验等。

volatile编译成机器后增加了一个带有lock前缀的操作，相当于内存屏障，对cpu总线加锁后把新值写入内存，并引起其他核心的缓存失效。这样可以保证此变量对所有线程的可见性，写入内存前的保证对后续代码有依赖的操作执行完成，就可以达到防止指令重排的效果。long和double的读写不是原子的，但由于商用虚拟机几乎都要实现为原子操作，那么就不用加volatile关键字了。

## 实现线程的原子性、可见性和有序性的关键字有哪些？

原子性可以通过synchronized实现。

可见性可以通过synchronized, volatile和final实现。

有序性可以通过synchronized和volatile实现。

## 能创建volatile数组吗？

能创建，volatile保证不改变数组引用，但数组元素可能改变。

## volatile 能使得一个非原子操作变成原子操作吗？

java读取 long/double 类型的变量不是原子的，需要分成两步，如果一个线程正在修改该 long 变量的值，
另一个线程可能只能看到该值的一半（前 32 位）。但是对一个 volatile 型的 long 或 double 变量的读写是原子。

volatile 变量的自增操作不具备原子性，synchronized关键字和原子变量可以保证自增操作的原子性。

## happens-before先行发生原则是什么？

原则包括：程序次序规则，加锁解锁规则，volatile规则，线程启动规则，线程终止规则，线程一场规则，对象终结规则，传递性。

## 锁优化策略有哪些？

锁优化包括：(自适应)自旋锁、锁消除、锁粗化、轻量级锁、偏向锁。

无竞争的情况下偏向锁去掉所有同步。

轻量级锁在栈帧中创建锁记录空间，用于存储锁对象的Mark Word副本。然后将对象的Mark Word被CAS更新为指向锁记录的指针，最后修改Mark Word副本的最后2位为00,则处于轻量级锁定的状态，用栈区操作代替同步原语。

## 了解FutureTask吗？

FutureTask继承AQS, 实现了Future和Runnable接口。3种状态包括未启动、已启动和已完成（正常结束或者取消，立即返回结果）。


