# 内存管理

## 虚拟机的内存区域有哪些？

程序计数器，堆区，虚拟机桟，本地方法栈，方法区（jdk8改为元数据区）

## 虚拟机的各个区域存储什么？

堆区存储对象，分区回收。

栈区存储基本类型和引用，方法的无限递归产生StackOverflowError错误。每个方法执行即是创建一个栈帧，包括局部变量表，操作数栈，动态链接，方法出入口等信息。其中局部变量表包括编译可知的基本类型，对象引用和返回地址。

方法区存储字节码、常量、静态变量，JIT机器码，常量池。其中常量池逻辑上属于方法区，物理上在java7以后转移到了堆区。

堆外内存包括虚拟机的方法区（元数据区）和直接内存（+XX:MaxDirectMemorySize=xx），nio包的DirectByteBuffer.allocate方法可以申请直接内存，此时不要关闭显式回收。

## 虚拟机栈帧结构是什么？

局部变量表，操作数栈，动态链接，方法返回地址，以及附加信息。

## 这个局部变量为什么没有被回收？

```java
//vm args: -Xms1m -Xmx1m -Xmn100k -Xss10k
public static void main(String[] args){
    {
        byte[] b = new byte[100*1024];
    }
    //int a=0;
    System.gc();
}
```

因为线程栈的本地变量表是以slot变量槽为基本单位，通过索引位置定位，可以重复使用。如果没有进一步的操作，就不会覆盖原先的slot，也就不会回收。如果添加一步任意操作，覆盖slot，原先的引用消失，就会回收。

## 方法区和永久区的关系怎么理解？

方法区是虚拟机规范。永久区是具体的虚拟机实现。HotSpot虚拟机实现了永久区。由各线程共享。jdk1.8取消了永久区，改为元空间，使用堆外的本地内存，宿主机内存大，不容易溢出。

## 方法区存储的内容以及回收过程是什么？

方法区存储了类的元数据信息、静态变量、操作字节码生成的类等。jdk1.6及之前，还包括运行时常量池。

方法区在虚拟机启动时创建，是堆的逻辑组成部分。HotSpot虚拟机的方法区设计为老年代，在major gc和full gc时回收。

回收时永久代中元数据的位置会发生移动，消耗虚拟机性能。并且各种类型数据的回收都要特殊处理永久代的元数据。将元数据从永久代分离，方便管理元空间、gc和元数据。

## 运行时常量池位于那个内存区域？

jdk1.6及之前，运行时常量池是方法区的一部分。jdk1.7将运行时常量池移到堆中。

## new关键字创建对象的过程是什么？

1. 类的加载。

1. 新生代内存的分配算法包括整理的指针碰撞算法，和不整理的空闲列表算法。更新对象指针的线程安全通过cas或者TLAB（需要参数支持）实现。准备完成。

1. 虚拟机对对象设置类的元数据信息、对象的哈希码、gc分代年龄等必要信息，所有的字段为0.解析完成。

1. 执行<init>方法，从父类开始初始化类。

## 对象和对象头包括哪些内容？

HotSpot虚拟机中，对象的内存包括对象头，实例数据和8字节对齐填充。32位和未开启指针压缩的64位虚拟机创建的普通对象的对象头分为2部分：

1. MarkWord部分的位数与jvm相同，包括哈希码、gc分代年龄、锁状态标记、线程持有的锁、偏向线程ID、偏向时间戳等。

1. 类型指针指向类的元数据。

1. 数组对象还包括数组大小的第3部分，这样，64位虚拟机创建的普通/数组对象最小为16/24字节，32位则是8/16字节。

## 这个HashMap对象的内存占用是怎样的？

HashMap<Long,Long>的kv2个长整形数据(8)占空间16B，加上对象头(8)和类指针(8)，达到48B。包装成Entry对象后，加上对象头(16)、next(8)和hash字段(4+4)，达到80B. HashMap作为存储容器，要加上引用(8)达到88B。空间效率为16/88=18%.

## 虚拟机如何获取对象的类信息？

对象对类的访问包括句柄和直接指针两种方法，HotSpot虚拟机使用直接指针法，通过方法栈本地变量表的引用找到实例对象，再由对象的元数据找到类对象，取得类信息。

## 常用的虚拟机参数有哪些？

-Xms, -Xmx, -server, -XX:GCUtil, -XX:+HeapDumpOnOutOfMemeryError 

-Xbootclasspath=... 可以指定启动类路径

-Xint/-Xcomp强制虚拟机运行于解释/编译模式

## finalize方法的作用是什么？

Object类里的方法，在对象回收时被调用，执行时间不可预测，出现的异常会被忽略。

只会被调用一次，不会立即回收。调用后对象的强引用变成弱引用，添加到回收队列。如果gc前改回强引用，就不会被回收了，进而容易出现问题。

当对象的销毁需要释放系统资源时，可以重写这个方法作为第二层安全保障。某些用到finalize的地方，必须显式调用super.finalize()回收链。 

## Java栈什么时候会发生内存溢出，Java堆呢？

递归深度过大，栈溢出。对象数量过大，堆溢出，比如集合类持有对象。解决方案之一是使用弱引用和软引用。

## 写一段内存溢出的代码

```java
//堆溢出，vm args: -Xms10m -Xmx10m -XX:+HeapDumpOnOutOfMemoryError
List<Object> list = new LinkedList<>();
while(true){
    list.add(new Object());
}

//栈溢出，vm args: -Xss10k
public int f(){
	f(n++)+=1;
    return f()-1;
}

//方法区常量池溢出，jdk1.6- vm args: -XX:PermSize=1M -XX:MaxPermSize=1M
List<String> list = new LinkedList<>();
while(true){
    list.add(new String());
}

//方法区溢出，jdk1.7-
while(true){
    //cglib,javaassit,asm等操作字节码生成新的类
}

//直接内存溢出 vm args: -XX:MaxDirectMemorySize=1M
List<ByteBuffer> list = new LinkedList<>();
while(true){
    list.add(ByteBuffer.allocateDirect(100000));
}

```



## 尾递归优化了什么？

递归是方法自调用，栈帧过深就可能栈溢出。尾递归优化可以解决这个问题。

尾递归是把自调用放在最后的一种写法：`return f(n)`或者`f(n)`。

具有尾递归优化功能的编译器就会在自调用的时候直接使用已有的栈帧，不再创建。这样就加快了方法调用速度。

## 调用System.gc()会发生什么?

通知GC开始工作，但是GC真正开始的时间不确定。

## 程序结束的方式有哪些？

1. System.exit()。

1. 正常执行完毕。

1. 程序运行错误，报错。

1. 系统出现问题，虚拟机停止运行。

# 垃圾收集

## 判断对象存活的算法是什么？

1. 引用计数法：

   对象维护一个计数器，对被引用次数计数，变成零，就可以回收了。

   缺点是：不能解决循环引用问题。

1. 可达性分析算法

   从根节点按照引用链关系，进行（深度优先）搜索，判断可达性。然后回收不可达对象。

   根节点包括：虚拟机栈和本地方法栈引用的对象，静态属性和常量引用的对象。

## Java对象的生命周期

创建阶段 、 应用阶段 、不可见阶段 、不可达阶段 、收集阶段 、终结阶段、 对象空间重新分配阶段等。

## 四种引用的回收时间是什么？

强引用对象走回收流程，软引用在内存溢出前回收，弱引用在下次就回收（ThreadLocal使用这种引用），虚引用没有生存时间。

## finalize方法的执行过程？

对象回收时，没有覆盖或者已调用过finalize()方法的对象没有必要执行该方法，必要执行的对象会被放到F-Queue回收队列里，等待低优先级的Finalizer线程执行该方法。等待队列里的对象在回收之前可以人为改变引用类型而被抢救。

## 方法区类对象的回收需要满足什么条件？

1. 该类所有的实例都已经被回收
1. 加载该类的ClassLoader加载器已经被回收。
1. Class对象没有被引用，无法反射访问该类对象的方法。
1. 虚拟机没有提供-Xnoclassgc参数。

## 垃圾收集算法以及对应的老年代收集器有哪些？

标记-清除（Mark-Sweep）：CMS收集器。效率不高，而且有内存碎片。

标记-整理（Mark-Compact）：Serial Old收集器。G1收集器。

复制（Copying）：新生代两个相同大小的Surviver区。G1收集器。

分代收集：新生代和老年代分别设置收集器。

## 垃圾收集器的种类以及搭配关系是什么？

新生代的Serial, ParNew收集器可以和老年代的Serial Old, CMS搭配。

新生代的Parallel Scavenge和老年代的Serial Old, Parallel Old搭配。

G1收集器同时包括新生代和老年代。

## 分代与回收过程是怎样的？

分为新生代、老年代和永久代。

对象的创建：一个对象主要在年轻代的Eden区创建。如果启用了本地线程分配缓冲，将按照线程优先在TLAB上分配。如果对象的大小超过了`PretenureSizeThreshold`空间阈值，或者minor gc后仍然超过了Eden区的大小，就会直接分配到老年代。

回收检查：Eden区没有足够空间分配时，会触发新生代对象的回收。虚拟机先检查老年代最大可用的连续空间是否大于新生代所有对象的总大小或者历次晋升到老年代对象的平均大小。如果大于，就尝试minor gc。否则就full gc。

新生代回收：Eden区对象的年龄都是零。minor gc把新生代存活对象的年龄加一，记录在对象头上。并且转移到survivor区。如果survivor区空间不够，就会通过分配担保机制直接转移到老年代。survivor区的回收使用复制算法，对象年龄超过了 `MaxTenuringThreshold` 最大年龄阈值，就会转移到老年代。如果survivor区中相同年龄所有对象大小的总和超过survivor空间的一半，将和更大年龄的对象一起直接进入老年代。

老年代回收：如果是CMS收集器，老年代对象占用老年代的比重超过了 `CMSInitiatingOccupancyFraction` 回收触发阈值，就会发起major gc。如果预留的内存无法满足回收，就会临时启用ParNew收集器进行full gc，后果是 stop the world 停顿时间长。

永久代回收：full gc会回收。

## 垃圾回收时为什么会停顿？

虚拟机使用可达性分析算法回收，枚举根节点需要确保引用关系不变，因此在安全点和安全区发生停顿。

安全点和安全区是引用关系不变的代码位置和片段。在确认系统完成了根节点枚举之后，就可以离开安全点和安全区。

## 虚拟机如何快速枚举根节点对象？

类加载完成后，HotSpot就计算出对象内不同偏移量的数据类型，以及JIT编译后的引用位置，保存在OopMap的数据结构中。

## 如何安全点的停顿是怎么实现的？

复用的指令序列包括方法调用，循环跳转，异常跳转等，具有“长时间运行”的特征，这些指令会产生安全点，让虚拟机停顿。

线程的停顿是通过主动式中断实现的。中断标志位于安全点和新对象需要分配内存的位置，线程轮询，发现标志为真就中断挂起。

## Parellel Old收集器的特点与使用方法是什么？

追求高吞吐量的Parellel Old收集器唯一适配的多线程新生代收集器是Parallel Scavenge收集器，适合于CPU密集型任务。通过`-XX:MaxGCPauseMillis`设置垃圾收集停顿时间，通过`-XX:GCTimeRatio=99`来设置运行时间对回收时间的比值，即吞吐量的倒数减1后的倒数，是1-99的整数。吞吐量是CPU工作时间占比。

## CMS收集器的收集过程？

初始标记，并发标记，重新标记，并发清除，重置线程。

初始标记根对象。并发标记对堆中的对象进行可达性分析。并发标记过程产生的新对象将被重新标记。最后并发清理，并且重置线程。其中，初始标记和重新标记需要"stop the world"停机，来保护现场。

## CMS收集器的使用要注意什么？

追求低停顿的CMS收集器通过-XX:UseConcMarkSweepGC开启，默认新生代收集器是ParNew收集器，适合于io密集型任务。

默认的回收线程数量是(核数+3)/4，适合核心数较多的CPU。核心数小于4的时候，并发回收的负载重。

CMS收集器在并发清理阶段产生的浮动垃圾只能在下一次gc时被清理，占用的内存资源会影响并发收集线程的。因此，需要为收集器的正常工作预留出足够的内存。老年代对象回收的触发阈值由参数`-XX:CMSInitiatingOccupancyFraction`设置，在jdk1.5中是68%，增大可以提高性能。在jdk1.6中是92%，容易因为预留内存不足而出现"Concurrent Mode Failure"失败。此时临时启用Serial Old收集器来full gc, 停顿的时间就很长了。

标记-清除算法会产生内存碎片，默认开启的`-XX:+UseCMSCompactAtFullCollection`参数会在每次full gc时整理碎片，停顿时间变长，将`+XX:CMSFullGCsBeforeCompaction`参数值设置为大于0的整数，可以在多次full gc后压缩一次。

## G1收集器的特点有哪些？

G1收集器把内存分为相同大小的Region区，每个区都有对应的Remember Set, 用于记录Region区间对象引用、其他收集器新老对象的引用。通过找到引用对象的位置而避免全堆扫描。

并行和并发充分利用了多核CPU的性能。通过区分新对象和旧对象的回收方式提高了分代收集的效果。通过整体的标记-整理算法和局部的复制算法实现空间整合。通过建立可预测的停顿模型实现可预测的停顿。

## G1收集器的收集过程？

初始标记根对象，并且修改TAMS值来为下一阶段在新的Region区创建新对象做准备。并发标记对堆中的对象进行可达性分析。最终标记、筛选回收。

## 哪些参数可以提高虚拟机的回收性能？

多核服务器可以通过-XX:ParallelGCThreads=xx来限制回收线程的数量，提高性能。大对象通过-XX:PretenureSizeThreshold=2000直接分配到老年代，避免在新生代回收耗时。

## 写一段垃圾回收代码。

```java
//minor gc, vm args: -Xms1m -Xmx1m -Xmn100k -verbose:gc +XX:PrintGCDetails
Byte[] a;
for(int i=0;i<20;i++){
    a = new Byte[10*1024];
}

//大对象直接进入老年代, vm args: -Xms1m -Xmx1m -Xmn100k -verbose:gc +XX:PrintGCDetails +XX:PretenureSizeThreshold=1024
Byte[] b = new Byte[100*1024];

//长期存活的对象进入老年代, vm args: -Xms1m -Xmx1m -Xmn100k -verbose:gc +XX:PrintGCDetails +XX:MaxTenuringThreshold=1
LinkedList<Byte[]> c = new LinkedList<>();
for(int i=0;i<20;i++){
    c.add(new Byte[10*1024]);
}
```



# 调试优化

## jdk内置的方法有哪些？

jps -v 显示虚拟机进程号和包括参数的进程名称。

jstat -gcutil [pid] 查看堆内存使用率和回收时间等统计信息。

jstat -class [pid] 查看类加载时间等统计信息。

jmap -dump:format=b,file=xx [pid] 可以得到堆转储文件，其他方法包括：使用-XX:+HeapDumpOnOutOfMemoryError让虚拟机在OOM时生成，使用-XX:+HeapDumpOnCtrlBreak允许按下Ctrl+Break键来生成，Linux系统下通过kill -3 [pid]来生成。

jmap -histo [pid] 显示类和实例数量等对象统计信息。

jhat [dump_file] 把堆转储文件从服务器拷贝到有浏览器界面的机器上分析，默认7000端口可以查看网页分析结果。更强大的分析工具是eclipse的MAT插件。分析结果有堆区的对象数量和内存占比等。

jstack -l [pid] 显示有死锁信息的线程堆栈，也可以输出到文件。

netstat -nat | grep [pid] -c 查询多少机器连接到这个端口上。

ps -eLf |grep java -c 可以查看java线程数

## 大内存的机器如何设置虚拟机？

在一台64位大内存机器上建立32位jvm亲和式逻辑集群可以提高硬件使用效率，防止指针膨胀和回收停顿。均衡软件可以使用根据sessionId自定义均衡算法的nginx。

## 内存溢出的原因有哪些？

内存溢出后，先检查堆区统计信息，如果对象占比正常，就可能是直接内存过大，通过打印线程堆栈定位到OOM，加限制参数解决。另外原因可能包括线程栈、socket缓存、jni内存、虚拟机和gc使用的内存溢出。

## CPU使用率100%的原因有哪些？

业务代码的死循环。

长时间的full gc。

复杂的操作，比如正则解析，低效的SQL。

HashMap线程不安全的操作导致的bug。

外部调用，如Runtime.getRuntime().exec()方法可以通过fork新进程来调用外部shell脚本，非常耗资源。

## 系统变慢的原因有哪些？

频繁的虚拟机回收。过长的系统调用链。数据库线程池满。

## 如何定位到问题线程？

jps或者top得到进程号，top -pH [pid] 可以查看该进程的线程列表，printf "%x\n" [tid]将线程号格式化为16进制。jstack [pid] 打印线程堆栈到文件，通过grep搜索这个文件来定位到问题线程位置。

还可以通过Thread类的getAllStackTraces()方法来获取所有线程的StackTraceElement对象，返回给前台管理页面。

## 如何得到堆转储文件？

jmap -dump:format=b,file=xx [pid] 可以得到堆转储文件，其他方法包括：使用-XX:+HeapDumpOnOutOfMemoryError让虚拟机在OOM时生成，使用-XX:+HeapDumpOnCtrlBreak允许按下Ctrl+Break键来生成，Linux系统下通过kill -3 [pid]来生成。

## 全回收的次数过多导致系统变慢，如何确认并解决？



定位到问题线程后，发现"VM Thread"和"nid=0x.."的信息，分别表示垃圾回收线程和操作系统的线程id。此时可以初步确认是垃圾回收导致系统变慢。

jstat -gcutil [tid] [time] [num] 可以查看GC情况，FGC数量大而且在增加，就可以进一步确认是垃圾回收所致。

生成堆转储文件，用eclipse的MAT插件分析，可以确定是哪个对象比较耗内存。

如果内存占用不多，那么就要考虑另一种情况，代码或者第三方依赖包中是否有显式的System.gc()调用。通过查看堆转储文件可以判断System.gc()引起GC，可以添加`-XX:+DisableExplicitGC`参数让虚拟机来禁用显式GC。

## 虚拟机编译后的优化策略有哪些？

编译后的优化包括：公共子表达式消除，数组边界检查消除，方法内联（消除调用成本），逃逸分析（包括栈上分配、同步消除、标量替换）。

# 类加载器

## 数组类是通过什么类加载器创建的？

数组类不通过类加载器创建，而是由虚拟机直接创建的。数组类的元素，可能通过类加载器创建。

## 类加载的过程是什么？

加载阶段，读取的静态字节流被转化为方法区的运行时数据结构，生产类对象。

验证阶段，验证文件格式、元数据、字节码和符号引用。

准备阶段，对类变量分配内存并设置初始值（final修饰的静态属性此时抢先赋值）。

解析阶段，把常量池内的符号引用替换为直接引用。

初始化阶段，执行类构造器<clinit>()方法，优先顺序为父类-子类，构造器-静态语句赋值。类的接口和接口的父接口的<clinit>()方法都不需要先执行，只有父接口中定义的变量使用时，父接口才会初始化。

## 一个对象的生命周期是什么？

加载，连接，初始化，使用，卸载回收。其中，连接包括验证，准备，解析。

## 类加载器有哪些？
启动类加载器（bootstrap class loader）：只加载 JVM 自身需要的类，或者Xbootstrap参数指定的类。
扩展类加载器（extensions class loader）：加载 JAVA_HOME/lib/ext 目录下或者由系统变量 -Djava.ext.dir 指定位路径中的类库。
应用程序类加载器（application class loader）：加载系统类路径 java -classpath 或 -Djava.class.path 下的类库。
自定义类加载器（java.lang.classloder）：继承 java.lang.ClassLoader 的自定义类加载器。

注意：-Djava.ext.dirs 会覆盖 Java 本身的 ext 设置，造成 JDK 内建功能无法使用。可以像下面这样指定参数：-Djava.ext.dirs=./plugin:$JAVA_HOME/jre/lib/ext。

它们的关系如下：
启动类加载器，C++实现，没有父类。
扩展类加载器（ExtClassLoader），Java 实现，父类加载器为 null。
应用程序类加载器（AppClassLoader），Java 实现，父类加载器为 ExtClassLoader 。
自定义类加载器，父类加载器为AppClassLoader。

## Tomcat类加载器是怎么样的？

## 双亲委派模型的加载过程是什么？

按照应用类加载器、扩展类加载器、启动类加载器的顺序，从子类加载器到父类加载器递归调用loadClass()方法，依次检查类是否已经被加载过。父类加载器加载失败后抛出ClassNotFoundException异常，子类加载器调用findClass()方法加载这个类，依次类推。检查过程和加载过程是进栈出栈过程，顺序相反。

## 如何自定义类加载器？

自己实现类加载器可以通过继承ClassLoader类，为保持保持双亲委派模型，重写findClass()方法。

## 双亲委派模型的三次破坏过程？

1. 重写loadClass()方法。
2. 线程上下文类加载器可以通过Thread类的setContextClassLoader()方法设置类加载器，来加载独立厂商实现的服务提供者接口(SPI)的类代码。。
3. 代码热替换和模块热部署，使用自定义的类加载器加载类。OSGi的bundle加载。

## Java中的对象一定在堆上分配吗？

开启了xx之后就可以在栈上分配。

## 类文件的结构是怎样的？

文件开头约定的魔数为：0xCAFEBABE(4)，接着是次版本号(2)，主版本号(2)，常量池，访问标志，类/父类/接口索引集合，字段表集合，方法表集合，属性表集合。

## 类什么时候初始化？

1. 初始化，也就是new时候会初始化类。

1. 访问类或者接口中的静态变量或者对其赋值。

1. 调用类的静态方法。

1. 反射（Class.forName("com.geminno");）。

1. 初始化它的子类，父类也会初始化。

1. 虚拟机启动时被标明是启动类的类（java Test），直接用java.exe运行某个类。

## 模块化怎么实现？

## Java9是怎么实现模块化的？

# 编译优化

## 编译与反编译

## Java代码的编译与反编译

## Java的反编译工具

javap 、jad 、CRF

## 即时编译器

词法分析，语法分析（LL算法，递归下降算法，LR算法），语义分析，运行时环境，中间代码，代码生成，代码优化

# 线上问题分析

## dump获取

线程Dump、内存Dump、gc情况

## dump分析

分析死锁、分析内存泄露

## dump分析及获取工具

jstack、jstat、jmap、jhat、Arthas

## 自己编写各种outofmemory，stackoverflow程序

HeapOutOfMemory、 Young OutOfMemory、MethodArea  OutOfMemory、ConstantPool OutOfMemory、DirectMemory OutOfMemory、Stack  OutOfMemory Stack OverFlow

## Arthas

jvm相关、class/classloader相关、monitor/watch/trace相关、

options、管道、后台异步任务

文档：<https://alibaba.github.io/arthas/advanced-use.html>

####常见问题解决思路

内存溢出、线程死锁、类加载冲突

## 使用工具尝试解决以下问题，并写下总结

当一个Java程序响应很慢时如何查找问题、

当一个Java程序频繁FullGC时如何解决问题、

如何查看垃圾回收日志、

当一个Java应用发生OutOfMemory时该如何解决、

如何判断是否出现死锁、

如何判断是否存在内存泄露

使用Arthas快速排查Spring Boot应用404/401问题

使用Arthas排查线上应用日志打满问题

利用Arthas排查Spring Boot应用NoSuchMethodError

说一下 jvm 的主要组成部分？及其作用？

\195. 说一下 jvm 运行时数据区？

\196. 说一下堆栈的区别？

\197. 队列和栈是什么？有什么区别？

\198. 什么是双亲委派模型？

\199. 说一下类加载的执行过程？

\200. 怎么判断对象是否可以被回收？

\201. java 中都有哪些引用类型？

\202. 说一下 jvm 有哪些垃圾回收算法？

\203. 说一下 jvm 有哪些垃圾回收器？

\204. 详细介绍一下 CMS 垃圾回收器？

\205. 新生代垃圾回收器和老生代垃圾回收器都有哪些？有什么区别？

\206. 简述分代垃圾回收器是怎么工作的？

\207. 说一下 jvm 调优的工具？

\208. 常用的 jvm 调优的参数都有哪些？

