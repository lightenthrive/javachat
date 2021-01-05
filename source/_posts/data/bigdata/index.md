# GFS

## GFS是什么？

Google早期研发的分布式文件系统。

## GFS的设计目标是什么？

主要有四个目标：

(1) **高可用**（availability）：是指7\*24提供服务，任何一台机器挂了或者磁盘坏了，服务不终止，文件不丢失。

(2) **高可靠**（reliability）：是指争取的输入，得到正确的输出，读取a文件，不会得到b文件。

(3) **高性能**（performance）：吞吐量每秒响应几十万请求。

(4) **可扩展**（scalability）：加机器，就能提升性能，就能存更多文件。

## GFS对外提供什么接口？

文件创建，删除，打开，关闭，读，写，快照。

其中，快照其实是快速文件目录树的拷贝，并不是所有文件的快照。其他的接口和单机文件系统差不多。

## GFS的系统架构如何？

系统里只有文件客户端，主服务器，存储服务器三个角色。

![img](https://mmbiz.qpic.cn/mmbiz_png/YrezxckhYOy0liasGc4c0Qslaicic5QgcXJpJGT8iaSKgpYqrO4GDyBBqlpRVay0VEgxonHKSoNwNNXXlvdP8jrVBA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

如上图：

(1) **客户端**（GFS client），是以库的形式提供的，提供的就是对外要用的接口。

(2) **主服务器**（GFS master），是单点，存储文件信息，目录信息，文件服务器信息，那个文件存在哪些文件服务器上等元数据。

(3) **存储服务器**（GFS chunk-server），是集群，存储文件。



## 为什么要设计单点master？

单点master意味着有一个节点可以避免分布式锁，可以拥有全局视野，能够统一调度与监控，系统整体复杂度降低很多。比如，锁可以降级成本地锁，分布式调度可以降级为单点调度。

 更具体的：

(1) master拥有所有文件目录结构，要操作某个文件，必须获得相应的锁。并发写操作的应用场景决定锁冲突其实不大。

(2) master拥有全局视野，能够避免死锁。

(3) master知道chunk-server的信息，能够很容易的做chunk-server监控，负载均衡。

(4) master知道所有文件的副本分布信息，能够很容易的做文件大小的负载均衡。

负载均衡分为请求量的均衡，文件存储的容量均衡。

## GFS的高可用是怎么保证的？

高可用又分为服务高可用，文件存储高可用，均通过“冗余+自动故障转移”的思路是实现。

(1) **master高可用**：冗余了一台影子master，平时不工作，master挂了工作，以保证master的高可用。

*画外音：master资源利用率只有50%。*

(2) **chunk-server高可用**：本身是集群，冗余服务。

*画外音：当有chunk-server挂掉，master能检测到，并且知道哪些文件存储在chunk-server上，就可以启动新的实例，并复制相关文件。*

(3) **文件存储高可用**：每一份文件会存三份，冗余文件。

##  GFS的高性能是怎么保证的？

多个**chunk-server**可以通过线性扩展提升处理能力和存储空间，GFS的潜在瓶颈是单点master，所以GFS要想达到**超高性能，主要架构优化思路**在于，“提升master性能，减少与master交互”。

(1) 只存储元数据，不存储文件数据，不让**磁盘容量**成为master瓶颈。

(2) 元数据会存储在磁盘和内存里，不让**磁盘IO**成为master瓶颈。

(3) 元数据大小内存完全能装得下，不让**内存容量**成为master瓶颈。

(4) 所有数据流，数据缓存，都不走master，不让**带宽**成为master瓶颈。

(5) 元数据可以缓存在客户端，每次从客户端本地缓存访问元数据，只有元数据不准确的时候，才会访问master，不让**CPU**成为成为master瓶颈。

 当然，chunk-server虽然有多个，也会通过一些手段提升chunk-server的性能，例如：

(1) 文件块使用64M，避免太多碎片降低性能。

(2) 使用追加写，而不是随机写，提升性能。

(3) 使用TCP长连接，提升性能。

## GFS如何保证系统可靠性？

保证元数据与文件数据的可靠性，GFS使用了很多非常经典的手段。

(1) 元数据的变更，会先写日志，以确保不会丢失。

*画外音：日志也会冗余，具备高可用。*

(2) master会轮询探测chunk-server的存活性，保证有chunk-server失效时，chunk-server的状态是准确的。

*画外音：文件会存多份，短时间内chunk-server挂掉是不影响的。*

(3) 元数据的修改是原子的，由master控制，master必须保证元数据修改的顺序性。

(4) 文件的正确性，通过checksum保证。

(5) 监控，快速发现问题。 

## 读操作的核心流程？

文件读取是最高频的操作。

(1) client读本地缓存，看文件在哪些chunk-server上。

(2) 如果client本地缓存miss，询问master文件所在位置，并更新本地缓存。

(3) 从一个chunk-server里读文件，如果读取到，就返回。

## 写操作的核心流程？

为了保证数据高可用，数据必须在多个chunk-server上写入多个副本，首先要解决的问题是，**如何保证多个chunk-server上的数据是一致的呢？**

想想一个MySQL集群的多个MySQL实例，是如何保证多个实例的数据一致性的。bingo！确定一个主实例，串行化所有写操作，然后在其他实例重放相同的操作序列，以保证多个实例数据的一致性。

 GFS也采用了类似的策略，一个文件冗余3份，存在3个chunk-server上，如下图步骤1-7：

![img](https://mmbiz.qpic.cn/mmbiz_png/YrezxckhYOy0liasGc4c0Qslaicic5QgcXJenicht9Hqg7mryjj0bZOKiayeL6QPHTYejVkhK4gJp96MmXt7PKWJgXA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

(1) client访问master，要发起文件写操作。

*画外音：假设client本地缓存未生效。*

(2) master返回数据存储在ABC三个实例上，并且告之其中一个实例是主chunk-server。

(3) client将数据流传递给所有chunk-server。

(4) client将控制流产地给主chunk-server。

(5) 主chunk-server进行本地操作串行化，并将序列化后的命令发送给其他chunk-server。

(6) 其他chunk-server按照相同的控制流对数据进行操作，并将结果告诉主chunk-server。

(7) 主chunk-server收到其他所有chunk-server的成果执行结果后，将结果返回client。

*画外音：MySQL的主库是写瓶颈，GFS不会出现这样的问题，每个文件的主chunk-server是不同的，所以每个实例的写请求也是均衡的。*

 这里需要说明的是，GFS对于写操作，执行的是最**保守的策略**，必须所有chunk写成功，才会返回client写成功（写吞吐会降低）。这样的好处是，读操作只要一个chunk读取成功，就能返回读成功（读吞吐会提升）。

*画外音：这也符合R+W>N的定理，N=3份副本，W=3写3个副本才算成功，R=1读1个副本就算成功。R+W>N定理未来再详述。*

 之所以这么设计，和文件操作“读多写少”的特性有关的，Google抓取的网页，更新较少，读取较多，这也是一个设计折衷的典型。

*画外音：任何脱离业务的架构设计都是耍流氓。* 

## 你所知道的“数据流与控制流分离”的设计准则有哪些？

(1) 控制流数据量小，client直接与主chunk-server交互。

(2) 数据流数据量大，client选择“最近的路径”发送数据。

*画外音：所谓“最近”，可以通过IP的相似度计算得到。* 

## GFS的架构，体现了哪些经典的设计实践？

- 简化系统角色，单点master降低系统复杂度
- 不管是文件还是服务，均通过“冗余+故障自动转移”保证**高可用**
- 由于存在单点master，GFS将“降低与单点master的交互”作为**性能优化核心**
- 通过写日志，原子修改，checksum，快速监控快速恢复等方式保证**可靠性与完整性**
- 通过串行化保证多个副本数据的**一致性**
- 控制流与数据流分离，提高性能

# MapReduce

## 什么是MapReduce？

它不是一个产品，而是一种解决问题的分治法思路，它有多个工程实现，Google在论文中也给出了它自己的工程架构实现。

## MapReduce这个编程模型解决什么问题？

能够用分治法解决的问题，例如：

- 网页抓取
- 日志处理
- 索引倒排
- 查询请求汇总
- …

*画外音：能够发现，现实中有许多基于分治的应用需求。*

## 为什么是Google，发明了这个模型？

Google网页抓取，分析，倒排的多个应用场景，当时的技术体系，解决不了Google大数据量高并发量的需求，Google被迫进行技术创新，思考出了这个模型。

*画外音：谁痛谁想办法。* 

## 为什么MapReduce对“能够用分治法解决的问题”特别有效？

分治法，是将一个大规模的问题，分解成多个小规模的问题(分)，多个小规模问题解决，再统筹小问题的解(合)，就能够解决大规模的问题。

*画外音：分治法详见《分治法与减治法》。*

## Google MapReduce为什么能够成功？

Google为了方便用户使用系统，提供给了用户很少的接口，去解决复杂的问题。
(1) Map函数接口：处理一个基于key/value(后简称kv)的成对(pair)数据集合，同时也输出基于kv的数据集合；
(2) Reduce函数接口：用来合并Map输出的kv数据集合；

*画外音：MapReduce系统架构，能在大规模普通PC集群上实现并行处理，和GFS等典型的互联网架构类似。*

用户仅仅关注少量接口，不用关心并行、容错、数据分布、负载均衡等细节，又能够解决很多实际的问题，还有这等好事！

## 能不能举一个例子，说明下MapReduce的Map函数与Reduce函数是如何解决实际问题的？ 

举例：假设要统计大量文档中单词出现的个数。

 **Map**

输入KV：pair(文档名称，文档内容)

输出KV：pair(单词，1)

*画外音：一个单词出现一次，就输出一个1。*

 **Reduce**

输入KV：pair(单词，1)

输入KV：pair(单词，总计数)

 以下是一段伪代码，

```python
Map(list<pair($doc_name, $doc_content)>){
	foreach(pair in list)
		foreach($word in $doc_content)
			echo pair($word, 1); // 输出list<k,v>
}
```

*画外音：如果有多个Map进程，输入可以是一个pair，不是一个list。*

```python
Reduce(list<pair($word, $count)>){// 大量(单词,1)
    map<string,int> result;
    foreach(pair in list)
        result[$word] += $count;
     foreach($key in result)
        echo pair($key, result[$key]); // 输出list<k,v>
}
```

*画外音：即使有多个Reduce进程，输入也是list<pair>，因为它的输入是Map的输出。* 

最早在单机的体系下计算，输入数据量巨大的时候，处理很慢。如何能够在短时间内完成处理，很容易想到的思路是，将这些计算分布在成百上千的主机上，但此时，会遇到各种复杂的问题，例如：

- 并行计算
- 数据分发
- 错误处理
- 集群通讯

思路比结论更重要。

MapReduce的**核心思路**是：

- 并行
- 先分再合

下图简述了MR计算“词频统计”的过程。

![img](https://mmbiz.qpic.cn/mmbiz_png/YrezxckhYOzT8aaHBQicEmzickibhbdCupUQf63DEhpuI6eqn9icStmAzlnMctPgq3cIM8KxYZWJick872NvMzpYBoQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**从左到右四个部分，分别是：**

- 输入文件
- 分：M个并行的map计算实例
- 合：R个并行的reduce计算实例
- 输出结果 

**先看最后一步，reduce输出最终结果。**

![img](https://mmbiz.qpic.cn/mmbiz_png/YrezxckhYOzT8aaHBQicEmzickibhbdCupUVzLySUZMDDK3NcTeHfo3PZqKPk9be74LYBCGONR7ok3ApN0B4KQoPg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

可以看到，R个reduce实例并发进行处理，直接输出最后的计数结果。

实例1输出：(a, 256)(able, 128)(emacs, 1)

实例2输出：(f*ck, 32768) (coding, 65535)

实例3输出：(vim,65535)(x, 16)(zero, 258)

*画外音：这就是总结果，可以看到vim比emacs受欢迎很多。*

 需要理解的是，由于这是业务计算的最终结果，一个单词的计数不会出现在两个实例里。即：如果(a, 256)出现在了实例1的输出里，就一定不会出现在其他实例的输出里。

*画外音：否则的话，还需要合并，就不是最终结果了。* 

**再看中间步骤，map到reduce的过程。**

![img](https://mmbiz.qpic.cn/mmbiz_png/YrezxckhYOzT8aaHBQicEmzickibhbdCupUJKZib0OesD4DrAC5gCPGsbicESjZluTyWRD6G8LuYvhjicziblREajh7Pw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

可以看到，M个map实例的输出，会作为R个reduce实例的输入。 

### 潜在问题一：每个map都有可能输出(a, 1)，而最终结果(a, 256)必须由一个reduce输出，那如何保证每个map输出的同一个key，落到同一个reduce上去呢？

这就是“分区函数”的作用。 

#### 什么是分区函数？

分区函数，是使用MapReduce的用户需所实现的，决定map输出的每一个key应当落到哪个reduce上的函数。

*画外音：如果用户没有实现，会使用默认分区函数。* 

以词频统计的应用为例，分区函数可能是：

(1) 以[a-g]开头的key落到第一个reduce实例；

(2) 以[h-n]开头的key落到第二个reduce实例；

(3) 以[o-z]开头的key落到第三个reduce实例；

*画外音：有点像数据库水平切分的“范围法”。* 

#### 分区函数实现要点是什么？

为了保证每一个reduce实例都能够差不多时间结束工作任务，分区函数的**实现要点**是：尽量负载均衡。

***画外音：即数据均匀分摊。*** 

上述词频统计的分区函数，就不是负载均衡的，有些reduce实例处理的单词多，有些reduce处理的单词少，这样就可能出现，所有reduce实例都处理结束，最后等待一个长尾reduce的情况。 

对于词频统计，负载更为均衡的分区函数为：

hash(key) % 3

*画外音：有点像数据库水平切分的“哈希法”。* 

### **潜在问题二**：每个map都有可能输出多个(a, 1)，这样无形中增大了网络带宽资源，以及reduce的计算资源，**有没有办法进行优化呢？**

这就是“合并函数”的作用。 

#### 什么是合并函数？

有时，map产生的中间key的重复数据比重很大，可以提供给用户一个自定义函数，在一个map实例完成工作后，本地就做一次合并，这样网络传输与reduce计算资源都能节省很多。 

合并函数在每个map任务结束前都会执行一次，一般来说，合并函数与reduce函数是一样的，区别是：

- **合并函数**执行map实例本地数据合并
- **reduce函数**执行最终的合并，会收集多个map实例的数据 

对于词频统计应用，合并函数可以将：

一个map实例的多个(a, 1)合并成一个(a, $count)输出。 

**最后看第一个个步骤，输入文件到map的过程。**

![img](https://mmbiz.qpic.cn/mmbiz_png/YrezxckhYOzT8aaHBQicEmzickibhbdCupUFYszfSaI7iaIuqY0kGUNGLm8ibAVnwW5E73o96nhf7q870W4mVujiaZTg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

### **潜在问题三**：如何确定文件到map的输入呢？

随意即可，只要负载均衡，均匀切分输入文件大小就行，不用管分到哪个map实例。

*画外音：无论分到那个map都能正确处理。* 

**结论**

Google MapReduce实施了一系列的优化。

- **分区函数**：保证不同map输出的相同key，落到同一个reduce里
- **合并函数**：在map结束时，对相同key的多个输出做本地合并，节省总体资源
- **输入文件到map如何切分**：随意，切分均匀就行 

希望大家对MapReduce的优化思路有一个了解，**思路比结论更重要**。

### 画一下MapReduce的总体架构图。

![img](https://mmbiz.qpic.cn/mmbiz_png/YrezxckhYOzxHvcSSJ15NK4FUUMAeOGciaC1GrCS7C0H8y8GOaHRRmRVYtTriaMBK0jOwogXQrhGeRicic44k003bg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

先看下，有个直观的印象。

 

### 用户使用GoogleMR系统，必须输入的是什么？

- 输入数据，必选

*画外音：否则系统处理啥。*

- map函数，必选
- reduce函数，必选

*画外音：分治法，分与合的业务逻辑。*

- 分区函数，必选

*画外音：保证同一个key，在合并阶段，必须落到同一个reduce上，系统提供默认hash(key)法。*

- 合并函数，可选

*画外音：看用户是否需要在map结束阶段进行优化。* 

### 用户提供各个输入后，GoogleMR的执行流程是什么？

*画外音：*

*不妨假设，用户设置了M个map节点，R个reduce节点；**例如：M=500，R=200* 

(1) 在集群中创建大量可执行实例副本(fork)；

(2) 这些副本中有一个master，其他均为worker，任务的分配由master完成， M个map实例和R个reduce实例由worker完成；

(3) 将输入数据分成M份，然后被分配到map任务的worker，从其中一份读取输入数据，执行用户的map函数处理，并在**本地内存**生成临时数据；

(4) 本地内存临时数据，通过分区函数，被分成R份，周期性的写到**本地磁盘**，由master调度，传给被分配到reduce任务的worker；

(5) 负责reduce任务的worker，从远程读取多个map输出的数据，执行用户的reduce函数处理，处理结果写入输出文件；

*画外音：可能对key要进行外部排序。*

(6) 所有map和reduce的worker都结束工作后，master唤醒用户程序，MapReduce调用返回，结果被输出到了R个文件中。 

### GoogleMR系统里的master和worker是啥？

(1) **master**：单点master会存储一些元数据，监控所有map与reduce的状态，记录哪个数据要给哪个map，哪个数据要给哪个reduce，掌控全局视野，做中控；

*画外音：是不是和GFS的master非常像？*

(2) **worker**：多个worker进行业务逻辑处理，具体一个worker是用来执行map还是reduce，是由master调度的；

画外音：是不是和工作线程池非常像？这里的worker是分布在多台机器上的而已。 

### master的高可用是如何保证的？

一个简单的方法是，将元数据固化到磁盘上，用一个shadow-master来做高可用。

*画外音：GFS不就是这么干的么？* 

然而现实情况是：没有将元数据固化到磁盘上，元数据被存放在master的内存里用以提高工作效率，当master挂掉后，通知用户“任务执行失败”，让其选择重新执行。

*画外音：*

*(1) 单点master，掌控全局视野，能让系统的复杂性降低非常多；*

*(2) master挂掉的概率很小；*

*(3) 不做高可用，能让系统的复杂性降低非常多；* 

### worker的高可用是如何保证的？

master会周期性的ping每个worker，如果超时未返回，master会把对应的worker置为无效，把这个worker的工作任务重新执行：

- 如果重新执行的是reduce任务，不需要有额外的通知
- 如果重新执行的是map任务，需要通知执行reduce的worker节点，输入数据换了一个worker 

### 随时都可能有map或者reduce挂掉，任务完成前重新被执行，会不会影响MR的最终结果？

在用户输入不变的情况下，MR的输出一定是不变的，这就要求MR系统必须具备幂等性：

- 对相同的输入，不管哪个负责map的worker执行的结果，一定是不变的，产出的R个本地输出文件内容也一定是不变的
- 对于M个map，每个map输出的R个本地文件，只要这些输入不变，对应接收这些数据的reduce的worker执行结果，一定是不变的，输出文件内容也也一定是不变的

###  长尾效应怎么解决？

一个MR执行时间的最大短板，往往是“长尾worker”。

 导致“长尾worker”的原因有很多：

(1) 用户的分区函数设计得不合理，导致某些reduce负载不均，要处理大量的数据；

*画外音：*

*最坏的情况，所有数据最终都落到一个reduce上，分布式并行处理，转变为了单机串行处理；*

*所以，分区函数的负载均衡性，是用户需要考虑的。*

 (2) 因为系统的原因，worker所在的机器磁盘坏了，CPU有问题，也可能导致任务执行很慢；

 GoogleMR有一个“备用worker”的机制，当某些worker的执行时间超出预期时，会启动另一个worker执行相同的任务，以尝试解决长尾效应。

总结

Google MapReduce架构，提现了很多经典架构实践：

- 单点master简化系统复杂度
- 单点master不高可用，简化系统复杂度
- master对worker的监控以及重启，保证worker高可用
- 幂等性，保证结果的正确性
- 多个worker执行同一个任务优化长尾问题

# BigTable

## 什么是BigTable？

BigTable是一个**稀疏的**、**分布式的**、**持久化的**、**多维度排序（结构化）的**、**大数据量**存储系统，它能够解决符合map数据模型业务的存储问题。用于“大数据量、高吞吐量、快速响应”等不同应用场景。

## 有BigTable之前，Google面临什么问题？

Google并不是一群人坐在办公室开会，想出来的系统，Google面临着很实际的业务问题。

**典型场景一：网页存储**

 Google每天要抓取很多网页：

- 新出现的网页，**新URL**
- 旧网页，**旧URL** 

**对一个已抓取的网页，旧URL为啥要反复抓取？**

因为，网页会更新，例如新浪首页：

sina.com.cn/index.html

URL虽然没有变，但依然会抓取。

*画外音：我去，相当于，被抓取的URL集合，只会无限增大，趋近无穷，这里面的技术难题，不知道大家感不感兴趣？* 

这里，对于**存储系统的需求**，是要存储：不同URL，不同时间Time，的内容Content。

*画外音：URL+”Content”+Time => Binary。* 

网页的实际内容Binary，是Spider抓取出来的。 

**典型场景二：Google Analytics** 

Google Analytics要给站长展示其网站的流量PV，独立用户数UV，典型访问路径等，以帮助站长了解站点情况，优化站点。 

这里，对于**存储系统的需求**，是要存储，不同URL，不同时间Time，的PV和UV。

*画外音：*

*URL+”PV”+Time => $count*

*URL+”UV”+Time => $count* 

PV和UV的值，是MapReduce离线任务计算出来的。 

不管是“网页存储”还是“站点统计”存储，它们都有几个共同的特点：

- 数据量极大，TB，PB级别
- 和时间维度相关
- 同一个主键，属性与值有映射

*画外音：*

*主键是URL，属性是“Content”，值是网页Binary；*

*主键是URL，属性是“PV”和“UV”，值是计数count。* 

这是Google曾经遇到的难题，面对这些难题，典型的解决方案又有哪些呢？

*画外音：不是一上来就搞新方案，最先肯定是想用现有的技术要如何解决。* 

## 解决方案

### 最容易想到的主键，属性，值的存储系统是什么？

没错，就是关系型数据库：

![img](https://mmbiz.qpic.cn/mmbiz_jpg/YrezxckhYOwembVFqRNYDKpmaCncTCU2ia8PXC6WIKSpvexs4prgoicic6h1wqI64CnO69VTahibbAlWgsI4GZUcGw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

如上图所示，用户表

User(uid PK, name, gender, age, sex)

就是一个典型的主键，属性，值的存储模型：

- **主键**，不同用户的uid
- **属性**，schema的列名
- **值**，不同主键的各个列名，对应的值

 使用excel来举例是很直观的，这是一个二维table。

*画外音：黄色的主键是一个维度，橙色的属性是一个维度。* 

### 用二维table能不能解决Google网页存储的问题呢？

![img](https://mmbiz.qpic.cn/mmbiz_jpg/YrezxckhYOwembVFqRNYDKpmaCncTCU2XIzdiaVVibYBVdAC1ABicRRMxHdr5BGqaMpfpEN63WafSHp16lLepxHuQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

如上图所示，如果没有时间维度Time，似乎是可以的：

- **主键**，使用URL
- **属性**，schema的列名，例如content，author等
- **值**，不同URL的内容与作者等值 

但是，一旦加入时间维度Time，二维table似乎就不灵了。

*画外音：*

*增加一个time属性是没有用的；*

*增加一个time属性，只能记录同一个URL，某一个time的content，不能记录多个time的多个content；*

*增加一个time属性，联合主键，URL就不是KEY了；*

###  能不能用二维table存储三维数据呢？

似乎可以通过trick的手段，在key上做文章，用key+time来拼接新key来实现。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/YrezxckhYOwembVFqRNYDKpmaCncTCU29N3r5eyJ1hX1FFh9p7WtL2vL1QicrEUV7AbH9CeMib710oyAkiaVWBSMg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

如上图所示，仍然是二维table，通过URL+Time来瓶装key，也能够实现，存储同一个URL，在不同Time，的不同content、author。

 但是，这种trick方案存在的问题是：

- 没法实现URL查询

*画外音：**key上无法进行%like%查询。*

- 大量空洞，浪费存储空间

这并不是一个好的方案。

 况且，当数据量达到TB、PB级别时，传统单机关系型数据库，根本无法满足Google的业务需求。

###  BigTable解决什么问题？

传统二维small table，无法解决Google面临的存储问题，于是Google搞了一个big table来解决。 

Google对这些业务模型进行分析，在二维table的基础上扩充，抽象了一个新的“三维table”：

- **主键**，使用URL
- **属性**，schema的列名，例如content，author等
- **时间**，timestamp
- **值**，不同URL的内容与作者等值

 ![img](https://mmbiz.qpic.cn/mmbiz_jpg/YrezxckhYOwembVFqRNYDKpmaCncTCU2uIfia6PqXibxC5FjQIwl079QTEpUicSXvm37JiafTaurDdKaEmogSe77tw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

如上图所示：

- 第一维：key（屎黄色）
- 第二维：属性（橙色）
- 第三维：time（蓝色）

同一个key，不同属性，不同时间，会存储一个value。

 

不像以行为单位进行存储的传统关系型数据库，这个三维的大表格BigTable是一个稀疏列存储系统。

*画外音：能够压缩空间。*

 

它的**数据模型**的本质是一个map：

key + column + time => value

的一个超级大map。

*画外音：*

*很多业务符合这一个模型；*

*Google的东西能解决业务问题，所以用的人多，这一点很重要。*

 

