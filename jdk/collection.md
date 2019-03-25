# 集合

## 集合类有哪些？继承关系是怎样的？

Map接口的实现类有：EnumMap(数组), TreeMap（红黑树）, HashMap(哈希), LinkedHashMap(哈希+链表), IdentityHashMap(哈希), WeakHashMap(哈希)

Collection的子接口有List, Queue, Set

List接口的实现类有：ArrayList(数组), LinkedList(链表)

Queue接口的实现类有：ArrayDeque(数组), LinkedList(链表), PriorityQueue(二叉堆)

Set接口的实现类有：TreeSet(红黑树), HashSet(哈希), LinkedHashSet(哈希+链表)

其中，Dictionary, HashTable(用ConcurrentHashMap替换), Vector(用CopyOnWriteList替换), Stack都是线程安全的遗留类。

![一张图看清](/source/images/collections.png)

继承关系如上图所示。

## fail-fast 和 fail-safe的区别是什么？

单/多线程环境下，fail-fast机制是：正在遍历的集合被修改，快速失败迭代器抛出ConcurrentModificationException。单线程环境的remove()方法会让expectModcount和modcount 相等，所以不会抛出这个异常。这个异常不能保证并发修改的条件，仅用于检测bug。比如HashMap,HashSet,ArrayList,Vector。

fail-safe机制是：任何对集合结构的修改都会在一个复制的集合上进行修改，因此不会抛出ConcurrentModificationException。问题是需要复制集合，无法保证读取最新数据。比如CopyOnWriteArrayList。

## fail-fast机制是如何检测的？

为了保证避免迭代器遍历直接访问的内部数据被修改，迭代器内部维护了一个标记 "mode" ，当集合结构改变（添加删除或者修改），标记"mode"会被修改，而迭代器每次的hasNext()和next()方法都会检查该"mode"是否被改变，当检测到被修改时，抛出ConcurrentModificationException。

## Arrays和System的sort方法的算法实现是怎么样的？

插入排序+快速排序。

## 当一个集合被作为参数传递给一个函数时，如何才可以确保函数不能修改它？

在作为参数传递之前，我们可以使用Collections.unmodifiableCollection(Collection c)方法创建一个只读集合，这将确保改变集合的任何操作都会抛出UnsupportedOperationException。

## 如何从给定集合那里创建一个synchronized的集合？

可以使用Collections.synchronizedCollection(Collection c)根据指定集合来获取一个synchronized（线程安全的）集合。

## 集合框架里实现的通用算法有哪些？

Java集合框架提供常用的算法实现，比如排序和搜索。Collections类包含这些方法实现。大部分算法是操作List的，但一部分对所有类型的集合都是可用的。部分算法有排序、搜索、混编、最大最小值。

## 我们如何对一组对象进行排序？

如果我们需要对一个对象数组进行排序，我们可以使用Arrays.sort()方法。如果我们需要排序一个对象列表，我们可以使用Collection.sort()方法。两个类都有用于自然排序（使用Comparable）或基于标准的排序（使用Comparator）的重载方法sort()。Collections内部使用数组排序方法，所有它们两者都有相同的性能，只是Collections需要花时间将列表转换为数组。

## 如何在遍历时移除Collection的元素？

使用迭代器的遍历和remove()方法。不要在foreach遍历的时候使用集合类的remove()方法，因为foreach会自动生成一个iterator来遍历，但同时该list正在被Iterator.remove() 修改，多线程情况下会抛出ConcurrentModificationException 异常。

## 与Java集合框架相关的有哪些最好的实践？

（1）根据需要选择正确的集合类型。比如，如果指定了大小，我们会选用Array而非ArrayList。如果我们想根据插入顺序遍历一个Map，我们需要使用TreeMap。如果我们不想重复，我们应该使用Set。

（2）一些集合类允许指定初始容量，所以如果我们能够估计到存储元素的数量，我们可以使用它，就避免了重新哈希或大小调整。

（3）基于接口编程，而非基于实现编程，它允许我们后来轻易地改变实现。

（4）总是使用类型安全的泛型，避免在运行时出现ClassCastException。

（5）使用JDK提供的不可变类作为Map的key，可以避免自己实现hashCode()和equals()。

（6）尽可能使用Collections工具类，或者获取只读、同步或空的集合，而非编写自己的实现。它将会提供代码重用性，它有着更好的稳定性和可维护性。

# 并发集合

## 并发集合类有哪些？

Map接口的并发实现类有：ConcurrentHashMap, ConcurrentSkipListMap

List接口的并发实现类有：CopyOnWriteList

Queue接口的并发实现类是阻塞队列：ArrayBlockingQueue(数组实现的有界队列), LinkedBlockingQueue(有界队列，但默认最大长度为最大整数), PriorityBlockingQueue(堆实现的支持优先级排序的无界队列), DelayQueue(延时获取元素的无界阻塞队列), SynchronousQueue(没有容量，现场交接), LinkedTransferQueue(链表组成的无界阻塞队列，有供求则优先), LinkedBlockingDeque(实现原理为Condition).

Set接口的并发实现类有：CopyOnWriteSet, ConcurrentSkipListSet

# Map

## HashMap与Hashtable的区别是什么？

HashMap 用于存储查找键值对。使用哈希算法，数据结构是数组+链表，jdk1.8加上了红黑树。

与Hashtable相比，线程不安全，速度快，但并发扩容时可能形成环状链表，导致get操作的时候，cpu空转到100%使用率。可以接受null键和值，默认容量16。Hashtable通过对方法加syncronized关键字实现线程安全，并发效率低，默认容量11,新元素加到链表头部。

## HashMap底层原理和扩容方案是什么？

HashMap插入数据，使用哈希算法，调用hashCode()方法计算键的哈希值，得到数组下标位置。如果没有碰撞，放入桶中。否则以链表的方式链接到后面。链表长度超过8,就转换为红黑树。长度小于6之后，转换回链表。如果节点已经存在，就替换旧值。如果键值对的数量达到容量(默认16)的负载因子0.75倍，就需要2倍扩容。并且把原来的对象rehash到新的数组中，新的位置只可能在2个地方：原下标位置，或者加上原数组长度+原下标。

HashMap查找数据，先用键的哈希值定位到数组位置，然后调用equals()方法依次查找对应的节点。找到了就返回对应的值。

HashMap的默认初始容量是16, 键值对数/数组位点的比例低于扩容因子0.75，哈希碰撞的键值对将按照链表或者红黑树结构加入到同一数组元素位点，高于0.75后的哈希碰撞引发2倍扩容。

## HashMap如何解决碰撞冲突的？

通过设计足够分散的哈希函数可以减少冲突，String, Integer这样的不可变对象已经重写了equals()和hashCode()方法，作为键，可以在寄存器缓存不变的hashcode.

发生冲突时，通过拉链法生成链表来解决冲突。链表长度可能过大，但容量也会超过负载上限，进而扩容。

其他解决碰撞冲突的方法还有开放定址法，具体包括线性探查法、二次探查法、双重散列法。

## 如何决定选用HashMap还是TreeMap？

需要快速插入和查找，用HashMap。需要键的排序，用TreeMap。

## LinkedHashMap和PriorityQueue的区别是什么？

LinkedHashMap可以按照数组和有序的双向链表的方式访问，PriorityQueue

为了保证同一个对象，保证在equals相同的情况下hashcode值必定相同，如果重写了equals而未重写hashcode方法，因为equal都是根据对象的特征进行重写的（可能就会出现两个没有关系的对象equals相同的），但hashcode确实不相同的。

## ConcurrentHashMap的原理是什么？

ConcurrentHashMap的原理，在jdk1.7的版本中维护了一个默认16的分段锁数组，每个桶位置是一个类似HashMap的数组段，数组长度都是2倍扩容策略。

插入数据，先计算键的哈希值，定位到段的位置，然后段内加锁，定位到链表位置，遍历链表插入数据。如果超过阈值，就扩容并且rehash.

查找数据，先计算键的哈希值，分段锁哈希计算定位到段的位置，volatile读取段数组，段数组内再次哈希计算定位到链表位置，volatile读取链表，顺序遍历得到对应的值。

1.8的版本利用CAS插入首节点，然后用syncronized关键字来保证链表或红黑树的插入并发安全，底层结构是数组+链表+红黑树。扩容时，cas修改数组长度，cas并发移动。

查找数据，同HashMap.

1.8的版本，递归创建ConcurrentHashMap对象时，可能出现死循环，这个bug在1.9版本被修复。

# List

## 实现List接口的类的继承关系图是怎样的？

![List继承关系图](/source/images/list.png)

如上图所示。

## ArrayList与Vector的区别是什么？

## 多线程场景下如何使用List？

ArrayList不是线程安全的，可以在加锁的方法内使用。

Collections 的 synchronizedList() 方法对ArrayList的方法加锁，实现了线程安全。

使用线程安全的CopyOnWriteList，原理是通过写入复制的数组实现了锁分离策略。

## 为什么ArrayList的elementData加上transient修饰？

ArrayList 实现了 Serializable 接口，transient关键字可以防止elementData数组被序列化，并且重写了writeObject()方法。

## 遍历一个List的方式以及实现原理是什么？

for循环遍历，基于集合外部的计数器读取。适合支持 RandomAccess 接口的数组结构的集合类。

迭代器遍历，使用了统一遍历集合的接口的 Iterator模式。

foreach循环遍历，内部使用了迭代器的实现。代码简洁，但是不能在遍历过程中操作数据集合，如删除、替换。

## Java中List遍历的最佳实践是什么？

使用迭代器的遍历和remove()方法。不要在foreach遍历的时候使用集合类的remove()方法，因为foreach会自动生成一个iterator来遍历，但同时该list正在被Iterator.remove() 修改，多线程情况下会抛出ConcurrentModificationException 异常。

## ArrayList如何实现扩容？

jdk7的ArrayList的无参构造方法先指向空数组，加入元素后得到的最小容量（实值）至少重设为默认初始数组长度10，如果大于原数组长度（位点），就1.5倍扩容，新长度容量至少重设为最小容量。如果超过最大数组长度，就根据新长度容量是否大于最大数组长度（常数），把新长度容量分别增加到最大整数或者最大数组长度。







