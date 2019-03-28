# 分布式

## 分布式系统的的应用

分布式系统一致性，分布式事务，分布式存储，分布式计算

## 进程间通信有哪几种方式？

1）管道（Pipe），2）命名管道（named pipe），3）信号（Signal），4）消息（Message）队列，5）共享内存，6）内存映射（mapped memory），7）信号量（semaphore），8）套接口（Socket）

## CAP定理说一下，为什么三者只能选二，为什么分区容忍性必须保证。 

一致性，可用性和分区容忍性不可以同时实现。CP系统包括zk,AP系统包括。

分布式系统的CAP不能同时实现，传统的OLTP业务要求关系型数据库满足CA即可，大型互联网平台的一个功能往往需要调用多个服务和数据库，业务上满足AP就必须把C降级为最终一致性。

## 分布式系统常见的问题有哪些？

通信异常(丢包或延时)，网络分区(局部集群)，三态(成败和超时)和节点故障(宕机或假死)。

## 什么是base理论？

BASE理论是对CA权衡的结果，三要素包括：基本可用(响应延时和功能降级)，弱状态(副本数据同步的延时)和最终一致性(包括因果一致性、读己之所写、会话一致性、单调读/写一致性)。



### 网络分区问题
主节点复活，会抢夺资源的使用权，系统功能失调。解决方案包括：(高可用的)第三方检测服务器仲裁，周期性ping主从机，如果有异常，则暂停业务操作，多次重试后重启或者退出。租期机制，原先的从服务器开始行使主服务器的功能后，给原先的主服务器发送新的租期，让原租期失效。

## 什么是幂等性，以及解决方案？

幂等性是重试的业务不会产生不良后果，方案是唯一ID。

## 分布式ID生成方案

- 数据库主键生成(mysql替换插入唯一键，获取last_insert_id)，

    ```
    生成表
    CREATE TABLE `Tickets64` (
      `id` bigint(20) unsigned NOT NULL auto_increment,
      `stub` char(1) NOT NULL default '',
      PRIMARY KEY  (`id`),
      UNIQUE KEY `stub` (`stub`)
    ) ENGINE=MyISAM
    业务生成
    REPLACE INTO Tickets64 (stub) VALUES ('a');
    SELECT LAST_INSERT_ID();

    多机配置为：
    TicketServer1:
    auto-increment-increment = 2
    auto-increment-offset = 1

    TicketServer2:
    auto-increment-increment = 2
    auto-increment-offset = 2
    ```
- uuid及其组合时间戳的变种

-  snowflake算法生成ID(时间毫秒数+机器号+流水号)

  时间--用前面41 bit来表示时间，精确到毫秒，可以表示69年的数据

  机器ID--用10 bit来表示，也就是说可以部署1024台机器

  序列号--用12 bit来表示，意味着每台机器，每毫秒最多可以生成4096个ID

- redis方法

  参考snowflake算法，id分为三段：时间，分片号，流水号。其中，流水号通过incr和incrby来生成。

  TIME命令返回时间四元组，处理为毫秒数。

- zookeeper临时顺序节点生成id。

  定义一个节点，创建临时顺序子节点，根据返回的节点名称，取序号部分组合得到id.

## 分布式ID生成速度如何提高？

放到缓存或者消息队列里

数据分布方式包括：哈希取模，一致性哈希