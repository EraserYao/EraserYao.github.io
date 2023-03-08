**笔记**

# TCP/IP模型（四层）

1. 应用层：软件实现层面，HTTP

2. 传输层：TCP/UDP。TCP是传输控制协议，相比 UDP 多了很多特性，比如流量控制、超时重传、拥塞控制等，这些都是为了保证数据包能可靠地传输给对方。

UDP 相对来说就很简单，简单到只负责发送数据包，传输效率高。可以在应用层实现TCP的功能。

3. 网络层：IP协议。数据部分+IP头。

IP地址和子网掩码做AND运算得到网络号。

IP 协议的寻址作用是告诉我们去往下一个目的地该朝哪个方向走，路由则是根据「下一个目的地」选择路径。寻址更像在导航，路由更像在操作方向盘。

4. 网络接口层：MAC头部，包含了接收方和发送方的 MAC 地址。网络接口层主要为网络层提供「链路级别」传输的服务，负责在以太网、WiFi 这样的底层网络上发送原始数据包，工作在网卡这个层次，使用 MAC 地址来标识网络上的设备。

Ping是由ICMP协议控制的。

# HTTP解析

1. 解析URL：数据协议+web服务器+目录名/文件名

HTTP的请求报文和响应报文：

![HTTP 的消息格式](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/%E9%94%AE%E5%85%A5%E7%BD%91%E5%9D%80%E8%BF%87%E7%A8%8B/4.jpg)

2. DNS解析

在域名中，**越靠右**的位置表示其层级**越高**。

![域名解析的工作流程](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/%E9%94%AE%E5%85%A5%E7%BD%91%E5%9D%80%E8%BF%87%E7%A8%8B/6.jpg)

3. 协议栈

应用程序（浏览器）通过调用 Socket 库，来委托协议栈工作。协议栈的上半部分别是负责收发数据的 TCP 和 UDP 协议，下面一半是用 IP 协议控制网络包收发操作，在互联网上传数据时，数据会被切分成一块块的网络包，而将网络包发送给对方的操作就是由 IP 负责的。

IP 中的 `ICMP` 协议和 `ARP` 协议。

- `ICMP` 用于告知网络包传送过程中产生的错误以及各种控制信息。
- `ARP` 用于根据 IP 地址查询相应的以太网 MAC 地址。

IP 下面的网卡驱动程序负责控制网卡硬件，而最下面的网卡则负责完成实际的收发操作，也就是对网线中的信号执行发送和接收操作。

4. 交换机

交换机的端口不核对接收方 MAC 地址，而是直接接收所有的包并存放到缓冲区中。因此，**交换机的端口不具有 MAC 地址**。

交换机有mac地址表映射网线端口。找不到对应的mac就广播发送。

广播地址：

- MAC 地址中的 `FF:FF:FF:FF:FF:FF`
- IP 地址中的 `255.255.255.255`

5. 路由

和交换机区别：

- 因为**路由器**是基于 IP 设计的，俗称**三层**网络设备，路由器的各个端口都具有 MAC 地址和 IP 地址；
- 而**交换机**是基于以太网设计的，俗称**二层**网络设备，交换机的端口不具有 MAC 地址。

# MySQL

1. **SQL语句执行**

![查询语句执行流程](https://cdn.xiaolincoding.com/gh/xiaolincoder/mysql/sql%E6%89%A7%E8%A1%8C%E8%BF%87%E7%A8%8B/mysql%E6%9F%A5%E8%AF%A2%E6%B5%81%E7%A8%8B.png)

连接器：MySQL基于TCP，最大连接数由 max_connections 参数控制。

查询缓存：对于更新比较频繁的表，查询缓存的命中率很低的，因为只要一个表有更新操作，那么这个表的查询缓存就会被清空，所以一般不用。

解析语句：词法分析和语法分析。

预处理器：查询表或字段是否存在，将*拓展为表上所有列。

优化器：确定SQL查询的执行方案，选择索引。

执行器：主键索引查询，全表扫描，索引下推。

2. **索引：数据的目录**

(1) 分类：

- 按「数据结构」分类：**B+tree索引、Hash索引、Full-text索引**。
- 按「物理存储」分类：**聚簇索引（主键索引）、二级索引（辅助索引）**。
- 按「字段特性」分类：**主键索引、唯一索引、普通索引、前缀索引**。
- 按「字段个数」分类：**单列索引、联合索引**。

**B+树和B树区别，为什么用B+树**

B+Tree 只在叶子节点存储数据，而 B 树 的非叶子节点也要存储数据，所以 B+Tree 的单个节点的数据量更小，在相同的磁盘 I/O 次数下，就能查询更多的节点。

另外，B+Tree 叶子节点采用的是双链表连接，适合 MySQL 中常见的基于范围的顺序查找，而 B 树无法做到这一点。

二叉树只有两个叶子结点，层次太多，IO操作更频繁。

Hash 表不适合做范围查询，它更适合做等值的查询。

(2) 主键索引（聚簇索引）和二级索引（辅助索引）

- 主键索引的 B+Tree 的叶子节点存放的是实际数据，所有完整的用户记录都存放在主键索引的 B+Tree 的叶子节点里；
- 二级索引的 B+Tree 的叶子节点存放的是主键值，而不是实际数据。

(3) 唯一索引，普通索引和前缀索引

唯一索引可以有多个，可以为空值；普通索引可以重复，用INDEX(column,...)创建。

前缀索引是指对字符类型字段的前几个字符建立的索引，而不是在整个字段上建立的索引，用来减少储存空间。

(4) 联合索引

将多个字段组合成一个索引，其B+树如下所示，叶子结点为双向链表：

![联合索引](https://cdn.xiaolincoding.com/gh/xiaolincoder/mysql/%E7%B4%A2%E5%BC%95/%E8%81%94%E5%90%88%E7%B4%A2%E5%BC%95.drawio.png)

使用联合索引时遵循最最左匹配原则，如果在查询到时候不遵循最左匹配原则，索引就会失效。在索引为(a,b,c)的情况下，是先按 a 排序，在 a 相同的情况再按 b 排序，在 b 相同的情况再按 c 排序。所以，**b 和 c 是全局无序，局部相对有序的**，如果查询时没有a，则会索引失效。

在进行范围查询的时候，先按照a的范围进行索引查询，但是a有序的情况下b是无序的。

(4) 索引优化：前缀索引优化，覆盖索引优化，主键自增，防止索引失效。**索引下推优化**（index condition pushdown)， **可以在联合索引遍历过程中，对联合索引中包含的字段先做判断，直接过滤掉不满足条件的记录，减少回表次数**。

3. 事务

事务是由 MySQL 的引擎来实现的，我们常见的 InnoDB 引擎它是支持事务的。事务具有以下四个特性(ACID)：

- **原子性（Atomicity）**：一个事务中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节，而且事务在执行过程中发生错误，会被回滚到事务开始前的状态，就像这个事务从来没有执行过一样，就好比买一件商品，购买成功时，则给商家付了钱，商品到手；购买失败时，则商品在商家手中，消费者的钱也没花出去。
- **一致性（Consistency）**：是指事务操作前和操作后，数据满足完整性约束，数据库保持一致性状态。比如，用户 A 和用户 B 在银行分别有 800 元和 600 元，总共 1400 元，用户 A 给用户 B 转账 200 元，分为两个步骤，从 A 的账户扣除 200 元和对 B 的账户增加 200 元。一致性就是要求上述步骤操作后，最后的结果是用户 A 还有 600 元，用户 B 有 800 元，总共 1400 元，而不会出现用户 A 扣除了 200 元，但用户 B 未增加的情况（该情况，用户 A 和 B 均为 600 元，总共 1200 元）。
- **隔离性（Isolation）**：数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致，因为多个事务同时使用相同的数据时，不会相互干扰，每个事务都有一个完整的数据空间，对其他并发事务是隔离的。也就是说，消费者购买商品这个事务，是不影响其他消费者购买的。
- **持久性（Durability）**：事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。

事务的隔离级别：

- **读未提交（read uncommitted）**，指一个事务还没提交时，它做的变更就能被其他事务看到；
- **读提交（read committed）**，指一个事务提交之后，它做的变更才能被其他事务看到；
- **可重复读（repeatable read）**，指一个事务执行过程中看到的数据，一直跟这个事务启动时看到的数据是一致的，**MySQL InnoDB 引擎的默认隔离级别**；
- **串行化（serializable）**；会对记录加上读写锁，在多个事务对这条记录进行读写操作时，如果发生了读写冲突的时候，后访问的事务必须等前一个事务执行完成，才能继续执行；

4. 日志

- **undo log（回滚日志）**：是 Innodb 存储引擎层生成的日志，实现了事务中的**原子性**，主要**用于事务回滚和 MVCC**。
- **redo log（重做日志）**：是 Innodb 存储引擎层生成的日志，实现了事务中的**持久性**，主要**用于掉电等故障恢复**；
- **binlog （归档日志）**：是 Server 层生成的日志，主要**用于数据备份和主从复制**；

# Redis基本数据结构，持久化策略

1. Redis 是一种基于内存的数据库，对数据的读写操作都是在内存中完成，因此**读写速度非常快**，常用于**缓存，消息队列、分布式锁等场景**。

Redis 提供了多种数据类型来支持不同的业务场景，比如 String(字符串)、Hash(哈希)、 List (列表)、Set(集合)、Zset(有序集合)、Bitmaps（位图）、HyperLogLog（基数统计）、GEO（地理信息）、Stream（流），并且对数据类型的操作都是**原子性**的，因为执行命令由单线程负责的，不存在并发竞争的问题。

2. Redis用作MySQL的缓存

Redis 具备高性能，将该用户访问的数据缓存在 Redis 中，这样下一次再访问这些数据的时候就可以直接从缓存中获取了，操作 Redis 缓存就是直接操作内存，所以速度相当快。

Redis 具备高并发，直接访问 Redis 能够承受的请求是远远大于直接访问 MySQL 的，所以我们可以考虑把数据库中的部分数据转移到缓存中去，这样用户的一部分请求会直接到缓存这里而不用经过数据库。

3. Redis数据结构

常见的有五种数据类型：**String（字符串），Hash（哈希），List（列表），Set（集合）、Zset（有序集合）**

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/%E5%85%AB%E8%82%A1%E6%96%87/%E4%BA%94%E7%A7%8D%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B.png)

4. Redis线程

**Redis 单线程指的是「接收客户端请求->解析请求 ->进行数据读写等操作->发送数据给客户端」这个过程是由一个线程（主线程）来完成的**。但是**Redis 程序并不是单线程的**，Redis 在启动的时候，是会**启动后台线程**（BIO）

单线程模式

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/%E5%85%AB%E8%82%A1%E6%96%87/redis%E5%8D%95%E7%BA%BF%E7%A8%8B%E6%A8%A1%E5%9E%8B.drawio.png)

5. Redis持久化

当 Redis 重启后，内存中的数据就会丢失，那为了保证内存中的数据不会丢失，Redis 实现了数据持久化的机制，这个机制会把数据存储到磁盘，这样在 Redis 重启就能够从磁盘中恢复原有的数据。

Redis 共有三种数据持久化的方式：

- **AOF 日志**：每执行一条写操作命令，就把该命令以追加的方式写入到一个文件里；
- **RDB 快照**：将某一时刻的内存数据，以二进制的方式写入磁盘；
- **混合持久化方式**：Redis 4.0 新增的方式，集成了 AOF 和 RBD 的优点；

(1) AOF日志实现：

Redis 在执行完一条写操作命令后，就会把该命令以追加的方式写入到一个文件里，然后 Redis 重启时，会读取该文件记录的命令，然后逐一执行命令的方式来进行数据恢复。在主线程中执行

Reids 是先执行写操作命令后，才将该命令记录到 AOF 日志里的，这么做其实有两个好处：

**避免额外的检查开销**，**不会阻塞当前写操作命令的执行**。

但是也有风险：

**数据可能会丢失**，**可能阻塞其他操作**。

(2) RBF快照：

AOF 日志记录的是操作命令，不是实际的数据，恢复的时候和能会很慢。RDB 快照就是记录某一个瞬间的内存数据，记录的是实际数据。

Redis 提供了两个命令来生成 RDB 文件，分别是 save 和 bgsave，他们的区别就在于是否在「主线程」里执行：

- 执行了 save 命令，就会在主线程生成 RDB 文件，由于和执行操作命令在同一个线程，所以如果写入 RDB 文件的时间太长，**会阻塞主线程**；
- 执行了 bgsave 命令，会创建一个子进程来生成 RDB 文件，这样可以**避免主线程的阻塞**；

6. Redis集群

(1) 主从复制

主从复制是 Redis 高可用服务的最基础的保证，实现方案就是将从前的一台 Redis 服务器，同步数据到多台从 Redis 服务器上，即一主多从的模式，且主从服务器之间采用的是「读写分离」的方式

![img](https://img-blog.csdnimg.cn/img_convert/2b7231b6aabb9a9a2e2390ab3a280b2d.png)

(2) 哨兵模式

当 Redis 的主从服务器出现故障宕机时，需要手动进行恢复。哨兵模式做到了可以监控主从服务器，并且提供**主从节点故障转移的功能。**

![img](https://img-blog.csdnimg.cn/26f88373d8454682b9e0c1d4fd1611b4.png)

7. 缓存

当**大量缓存数据在同一时间过期（失效）或者 Redis 故障宕机**时，如果此时有大量的用户请求，都无法在 Redis 中处理，于是全部请求都直接访问数据库，从而导致数据库的压力骤增，严重的会造成数据库宕机，从而形成一系列连锁反应，造成整个系统崩溃，这就是**缓存雪崩**的问题。

如果缓存中的**某个热点数据过期**了，此时大量的请求访问了该热点数据，就无法从缓存中读取，直接访问数据库，数据库很容易就被高并发的请求冲垮，这就是**缓存击穿**的问题。

当用户访问的数据，**既不在缓存中，也不在数据库中**，导致请求在访问缓存时，发现缓存缺失，再去访问数据库时，发现数据库中也没有要访问的数据，没办法构建缓存数据，来服务后续的请求。那么当有大量这样的请求到来时，数据库的压力骤增，这就是**缓存穿透**的问题。

# 三次招手四次挥手

TCP的四元组可以确定唯一一个连接

![TCP 四元组](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jZG4uanNkZWxpdnIubmV0L2doL3hpYW9saW5jb2Rlci9JbWFnZUhvc3QyLyVFOCVBRSVBMSVFNyVBRSU5NyVFNiU5QyVCQSVFNyVCRCU5MSVFNyVCQiU5Qy9UQ1AtJUU0JUI4JTg5JUU2JUFDJUExJUU2JThGJUExJUU2JTg5JThCJUU1JTkyJThDJUU1JTlCJTlCJUU2JUFDJUExJUU2JThDJUE1JUU2JTg5JThCLzEwLmpwZw?x-oss-process=image/format,png)

(1) 三次握手

![TCP 三次握手](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost4/%E7%BD%91%E7%BB%9C/TCP%E4%B8%89%E6%AC%A1%E6%8F%A1%E6%89%8B.drawio.png)

- 一开始，客户端和服务端都处于 `CLOSE` 状态。先是服务端主动监听某个端口，处于 `LISTEN` 状态
- 客户端会随机初始化序号（`client_isn`），将此序号置于 TCP 首部的「序号」字段中，同时把 `SYN` 标志位置为 `1`，表示 `SYN` 报文。接着把第一个 SYN 报文发送给服务端，表示向服务端发起连接，该报文不包含应用层数据，之后客户端处于 `SYN-SENT` 状态
- 服务端收到客户端的 `SYN` 报文后，首先服务端也随机初始化自己的序号（`server_isn`），将此序号填入 TCP 首部的「序号」字段中，其次把 TCP 首部的「确认应答号」字段填入 `client_isn + 1`, 接着把 `SYN` 和 `ACK` 标志位置为 `1`。最后把该报文发给客户端，该报文也不包含应用层数据，之后服务端处于 `SYN-RCVD` 状态
- 客户端收到服务端报文后，还要向服务端回应最后一个应答报文，首先该应答报文 TCP 首部 `ACK` 标志位置为 `1` ，其次「确认应答号」字段填入 `server_isn + 1` ，最后把报文发送给服务端，这次报文可以携带客户到服务端的数据，之后客户端处于 `ESTABLISHED` 状态。
- 服务端收到客户端的应答报文后，也进入 `ESTABLISHED` 状态

**第三次握手是可以携带数据的，前两次握手是不可以携带数据的**。

TCP 的连接状态查看，在 Linux 可以通过 `netstat -napt` 命令查看。

三次握手的**首要原因是为了防止旧的重复连接初始化造成混乱**

第一次握手丢失会触发超时重传；第二次握手丢失客户端会认为第一次握手没有成功，也会触发超时重传。如果第三次握手丢失，如果服务端那一方迟迟收不到这个确认报文，就会触发超时重传机制，重传 SYN-ACK 报文，直到收到第三次握手，或者达到最大重传次数。

(2) 四次挥手

![客户端主动关闭连接 —— TCP 四次挥手](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jZG4uanNkZWxpdnIubmV0L2doL3hpYW9saW5jb2Rlci9JbWFnZUhvc3QyLyVFOCVBRSVBMSVFNyVBRSU5NyVFNiU5QyVCQSVFNyVCRCU5MSVFNyVCQiU5Qy9UQ1AtJUU0JUI4JTg5JUU2JUFDJUExJUU2JThGJUExJUU2JTg5JThCJUU1JTkyJThDJUU1JTlCJTlCJUU2JUFDJUExJUU2JThDJUE1JUU2JTg5JThCLzMwLmpwZw?x-oss-process=image/format,png)

- 客户端打算关闭连接，此时会发送一个 TCP 首部 `FIN` 标志位被置为 `1` 的报文，也即 `FIN` 报文，之后客户端进入 `FIN_WAIT_1` 状态。
- 服务端收到该报文后，就向客户端发送 `ACK` 应答报文，接着服务端进入 `CLOSE_WAIT` 状态。
- 客户端收到服务端的 `ACK` 应答报文后，之后进入 `FIN_WAIT_2` 状态。
- 等待服务端处理完数据后，也向客户端发送 `FIN` 报文，之后服务端进入 `LAST_ACK` 状态。
- 客户端收到服务端的 `FIN` 报文后，回一个 `ACK` 应答报文，之后进入 `TIME_WAIT` 状态
- 服务端收到了 `ACK` 应答报文后，就进入了 `CLOSE` 状态，至此服务端已经完成连接的关闭。
- 客户端在经过 `2MSL` 一段时间后，自动进入 `CLOSE` 状态，至此客户端也完成连接的关闭。

解决TIME_WAIT状态过多的首要方法就是**避免服务器频繁主动断开连接**

解决 time_wait 状态大量存在，导致新连接创建失败的问题，一般解决办法：

1.客户端：HTTP 请求的头部，connection 设置为 keep-alive，保持存活一段时间：现在的浏览器，一般都这么进行了

2.服务器端：

允许 time_wait 状态的 socket 被重用

缩减 time_wait 时间，设置为**1 MSL（即，2 mins）**

但是**等待2MSL时间主要目的是怕最后一个ACK包对方没收到，那么对方在超时后将重发第三次握手的FIN包，主动关闭端接到重发的FIN包后可以再发一个ACK应答包**

# TCP基础

(1)  超时重传

重传机制的其中一个方式，就是在发送数据时，设定一个定时器，当超过指定的时间后，没有收到对方的 `ACK` 确认应答报文，就会重发该数据，也就是我们常说的**超时重传**。

(2) 流量控制

如果一直无脑的发数据给对方，但对方处理不过来，那么就会导致触发重发机制，从而导致网络流量的无端的浪费。

为了解决这种现象发生，**TCP 提供一种机制可以让「发送方」根据「接收方」的实际接收能力控制发送的数据量，这就是所谓的流量控制。**

(3) 拥塞控制

**在网络出现拥堵时，如果继续发送大量数据包，可能会导致数据包时延、丢失等，这时 TCP 就会重传数据，但是一重传就会导致网络的负担更重，于是会导致更大的延迟以及更多的丢包，这个情况就会进入恶性循环被不断地放大....**

所以，TCP 不能忽略网络上发生的事，它被设计成一个无私的协议，当网络发送拥塞时，TCP 会自我牺牲，降低发送的数据量。

于是，就有了**拥塞控制**，控制的目的就是**避免「发送方」的数据填满整个网络**。

# 线程池

1. 线程和进程的区别

进程是对运行时程序的封装，是**系统进行资源调度和分配的的基本单位，实现了操作系统的并发**；

线程是进程的子任务，**是CPU调度和分派的基本单位**，**用于保证程序的实时性，实现进程内部的并发；线程是操作系统可识别的最小执行和调度单位**。每个线程都独自占用一个**虚拟处理器**：独自的**寄存器组**，**指令计数器和处理器状态**。每个线程完成不同的任务，但是**共享同一地址空间**（也就是同样的**动态内存，映射文件，目标代码等等**），**打开的文件队列和其他内核资源**。

线程在进程下行进（单纯的车厢无法运行）

一个进程可以包含多个线程（一辆火车可以有多个车厢）

不同进程间数据很难共享（一辆火车上的乘客很难换到另外一辆火车，比如站点换乘）

同一进程下不同线程间数据很易共享（A车厢换到B车厢很容易）

进程要比线程消耗更多的计算机资源（采用多列火车相比多个车厢更耗资源）

进程间不会相互影响，一个线程挂掉将导致整个进程挂掉（一列火车不会影响到另外一列火车，但是如果一列火车上中间的一节车厢着火了，将影响到所有车厢）

进程可以拓展到多机，进程最多适合多核（不同火车可以开在多个轨道上，同一火车的车厢不能在行进的不同的轨道上）

进程使用的内存地址可以上锁，即一个线程使用某些共享内存时，其他线程必须等它结束，才能使用这一块内存。（比如火车上的洗手间）－"互斥锁"

进程使用的内存地址可以限定使用量（比如火车上的餐厅，最多只允许多少人进入，如果满了需要在门口等，等有人出来了才能进去）－“信号量”

2. 线程池

**线程池（Thread Pool）**是一种基于池化思想管理线程的工具，经常出现在多线程服务器中，如MySQL。

线程过多会带来额外的开销，其中包括创建销毁线程的开销、调度线程的开销等等，同时也降低了计算机的整体性能。线程池维护多个线程，等待监督管理者分配可并发执行的任务。这种做法，一方面避免了处理任务时创建销毁线程开销的代价，另一方面避免了线程数量膨胀导致的过分调度问题，保证了对内核的充分利用。

使用线程池可以带来一系列好处：

- **降低资源消耗**：通过池化技术重复利用已创建的线程，降低线程创建和销毁造成的损耗。
- **提高响应速度**：任务到达时，无需等待线程创建即可立即执行。
- **提高线程的可管理性**：线程是稀缺资源，如果无限制创建，不仅会消耗系统资源，还会因为线程的不合理分布导致资源调度失衡，降低系统的稳定性。使用线程池可以进行统一的分配、调优和监控。
- **提供更多更强大的功能**：线程池具备可拓展性，允许开发人员向其中增加更多的功能。比如延时定时线程池ScheduledThreadPoolExecutor，就允许任务延期执行或定期执行。

3. 线程池解决的问题

线程池解决的核心问题就是资源管理问题。

在并发环境下，系统不能够确定在任意时刻中，有多少任务需要执行，有多少资源需要投入。

4. 核心设计与实现

Java中的线程池核心实现类是ThreadPoolExecutor，ThreadPoolExecutor实现的顶层接口是Executor，顶层接口Executor提供了一种思想：将任务提交和任务执行进行解耦。用户无需关注如何创建线程，如何调度线程来执行任务，用户只需提供Runnable对象，将任务的运行逻辑提交到执行器(Executor)中，由Executor框架完成线程的调配和任务的执行部分。

ThreadPoolExecutor运行机制如：

![图2 ThreadPoolExecutor运行流程](https://p0.meituan.net/travelcube/77441586f6b312a54264e3fcf5eebe2663494.png)



# String/List/Hashmap（红黑树/链表）

**字符串**

![img](https://kstar-1253855093.cos.ap-nanjing.myqcloud.com/baguwenpdf/_StringTable_posper.png)

**Java容器**

![img](https://kstar-1253855093.cos.ap-nanjing.myqcloud.com/baguwenpdf/_Java%E5%AE%B9%E5%99%A8%E6%80%9D%E7%BB%B4%E5%AF%BC%E5%9B%BE_posper.png)

**List**

![img](https://kstar-1253855093.cos.ap-nanjing.myqcloud.com/baguwenpdf/_Java_List_%E6%BA%AF%E9%A3%8E.png)

1. HashMap

(1) HashMap 底层是一个数组

(2) 数组中每个元素是一个单向链表(即，采用拉链法解决哈希冲突)

单链表的节点每个节点是 Node<K, V> 类型

(3) 同一个单链表中所有 Node 的 hash值不一定一样，但是他们对应的数组下标一定一样

数组下标利用哈希函数/哈希算法根据 hash值计算得到的

(4) HashMap 是数组和单链表的结合体

数组查询效率高，但是增删元素效率较低

单链表在随机增删元素方面效率较高，但是查询效率较低

HashMap 将二者结合起来，充分它们各自的优点

(5) HashMap 特点

无序、不可重复

无序:因为不一定挂在那个单链表上了

(6) 为什么不可重复？

通过重写 equals 方法保证的。

JDK 1.8 之后，对 HashMap 底层数据结构(单链表)进行了改进

如果单链表元素超过8个，则将单链表转变为红黑树;
如果红黑树节点数量小于6时，会将红黑树重新变为单链表。

2. HashMap，HashTable，ConcurrentHashMap线程安全：

HashMap 是**线程不安全的**。在多线程条件下，容易导致死循环。JDK 1.8 HashMap 采用数组 + 链表 + 红黑二叉树的数据结构，优化了 1.7 中数组扩容的方案，解决了 Entry 链死循环和数据丢失问题。但是多线程背景下，put 方法存在数据覆盖的问题。

HashTable是线程安全的。

在ConcurrentHashMap中，无论是读操作还是写操作都能保证很高的性能：在进行读操作时(几乎)不需要加锁，而在写操作时通过锁分段技术只对所操作的段加锁而不影响客户端对其它段的访问。特别地，在理想状态下，ConcurrentHashMap 可以支持 16 个线程执行并发写操作（如果并发级别设为16），及任意数量线程的读操作。
ConcurrentHashMap的高效并发机制是通过以下三方面来保证的：

(1) 通过锁分段技术保证并发环境下的写操作；

(2) 通过 HashEntry的不变性、Volatile变量的内存可见性和加锁重读机制保证高效、安全的读操作；

(3) 通过不加锁和加锁两种方案控制跨段操作的的安全性。

3. String、StringBuilder、StringBuffer:

String 底层数组用 final 修饰，不可变。

StringBuilder 底层数组没有用 final 修饰，可变;线程不安全，效率高(一般用的多)

StringBuffer 底层数组没有用 final 修饰，可变;线程安全，效率低(一般用的少)

# HTTP状态码

| 100  | Continue                        | 继续。客户端应继续其请求                                     |
| ---- | ------------------------------- | ------------------------------------------------------------ |
| 101  | Switching Protocols             | 切换协议。服务器根据客户端的请求切换协议。只能切换到更高级的协议，例如，切换到HTTP的新版本协议 |
|      |                                 |                                                              |
| 200  | OK                              | 请求成功。一般用于GET与POST请求                              |
| 201  | Created                         | 已创建。成功请求并创建了新的资源                             |
| 202  | Accepted                        | 已接受。已经接受请求，但未处理完成                           |
| 203  | Non-Authoritative Information   | 非授权信息。请求成功。但返回的meta信息不在原始的服务器，而是一个副本 |
| 204  | No Content                      | 无内容。服务器成功处理，但未返回内容。在未更新网页的情况下，可确保浏览器继续显示当前文档 |
| 205  | Reset Content                   | 重置内容。服务器处理成功，用户终端（例如：浏览器）应重置文档视图。可通过此返回码清除浏览器的表单域 |
| 206  | Partial Content                 | 部分内容。服务器成功处理了部分GET请求                        |
|      |                                 |                                                              |
| 300  | Multiple Choices                | 多种选择。请求的资源可包括多个位置，相应可返回一个资源特征与地址的列表用于用户终端（例如：浏览器）选择 |
| 301  | Moved Permanently               | 永久移动。请求的资源已被永久的移动到新URI，返回信息会包括新的URI，浏览器会自动定向到新URI。今后任何新的请求都应使用新的URI代替 |
| 302  | Found                           | 临时移动。与301类似。但资源只是临时被移动。客户端应继续使用原有URI |
| 303  | See Other                       | 查看其它地址。与301类似。使用GET和POST请求查看               |
| 304  | Not Modified                    | 未修改。所请求的资源未修改，服务器返回此状态码时，不会返回任何资源。客户端通常会缓存访问过的资源，通过提供一个头信息指出客户端希望只返回在指定日期之后修改的资源 |
| 305  | Use Proxy                       | 使用代理。所请求的资源必须通过代理访问                       |
| 306  | Unused                          | 已经被废弃的HTTP状态码                                       |
| 307  | Temporary Redirect              | 临时重定向。与302类似。使用GET请求重定向                     |
|      |                                 |                                                              |
| 400  | Bad Request                     | 客户端请求的语法错误，服务器无法理解                         |
| 401  | Unauthorized                    | 请求要求用户的身份认证                                       |
| 402  | Payment Required                | 保留，将来使用                                               |
| 403  | Forbidden                       | 服务器理解请求客户端的请求，但是拒绝执行此请求               |
| 404  | Not Found                       | 服务器无法根据客户端的请求找到资源（网页）。通过此代码，网站设计人员可设置"您所请求的资源无法找到"的个性页面 |
| 405  | Method Not Allowed              | 客户端请求中的方法被禁止                                     |
| 406  | Not Acceptable                  | 服务器无法根据客户端请求的内容特性完成请求                   |
| 407  | Proxy Authentication Required   | 请求要求代理的身份认证，与401类似，但请求者应当使用代理进行授权 |
| 408  | Request Time-out                | 服务器等待客户端发送的请求时间过长，超时                     |
| 409  | Conflict                        | 服务器完成客户端的 PUT 请求时可能返回此代码，服务器处理请求时发生了冲突 |
| 410  | Gone                            | 客户端请求的资源已经不存在。410不同于404，如果资源以前有现在被永久删除了可使用410代码，网站设计人员可通过301代码指定资源的新位置 |
| 411  | Length Required                 | 服务器无法处理客户端发送的不带Content-Length的请求信息       |
| 412  | Precondition Failed             | 客户端请求信息的先决条件错误                                 |
| 413  | Request Entity Too Large        | 由于请求的实体过大，服务器无法处理，因此拒绝请求。为防止客户端的连续请求，服务器可能会关闭连接。如果只是服务器暂时无法处理，则会包含一个Retry-After的响应信息 |
| 414  | Request-URI Too Large           | 请求的URI过长（URI通常为网址），服务器无法处理               |
| 415  | Unsupported Media Type          | 服务器无法处理请求附带的媒体格式                             |
| 416  | Requested range not satisfiable | 客户端请求的范围无效                                         |
| 417  | Expectation Failed              | 服务器无法满足Expect的请求头信息                             |
|      |                                 |                                                              |
| 500  | Internal Server Error           | 服务器内部错误，无法完成请求                                 |
| 501  | Not Implemented                 | 服务器不支持请求的功能，无法完成请求                         |
| 502  | Bad Gateway                     | 作为网关或者代理工作的服务器尝试执行请求时，从远程服务器接收到了一个无效的响应 |
| 503  | Service Unavailable             | 由于超载或系统维护，服务器暂时的无法处理客户端的请求。延时的长度可包含在服务器的Retry-After头信息中 |
| 504  | Gateway Time-out                | 充当网关或代理的服务器，未及时从远端服务器获取请求           |
| 505  | HTTP Version not supported      | 服务器不支持请求的HTTP协议的版本，无法完成处理               |

# Spring AOP，事务

![img](https://kstar-1253855093.cos.ap-nanjing.myqcloud.com/baguwenpdf/_Spring_Horse.png)

1. AOP：即面向切面编程，它将业务逻辑的各个部分进行隔离，使开发人员在编写业务逻辑时可以专心于核心业务，从而提高了开发效率。

2. MyBatis

   (1) MyBatis是一个优秀的持久层框架，它对jdbc的操作数据库的过程进行封装，使开发者只需要关注 SQL 本身，而不需要花费精力去处理例如注册驱动、创建connection、创建statement、手动设置参数、结果集检索等jdbc繁杂的过程代码。

   (2) Mybatis通过xml或注解的方式将要执行的各种statement（statement、preparedStatemnt、CallableStatement）配置起来，并通过java对象和statement中的sql进行映射生成最终执行的sql语句，最后由mybatis框架执行sql并将结果映射成java对象并返回。

- mybatis配置
  SqlMapConfig.xml，此文件作为mybatis的全局配置文件，配置了mybatis的运行环境等信息。
  mapper.xml文件即sql映射文件，文件中配置了操作数据库的sql语句。此文件需要在SqlMapConfig.xml中加载。
- 通过mybatis环境等配置信息构造SqlSessionFactory即会话工厂
- 由会话工厂创建sqlSession即会话，操作数据库需要通过sqlSession进行。
- mybatis底层自定义了Executor执行器接口操作数据库，Executor接口有两个实现，一个是基本执行器、一个是缓存执行器。
- Mapped Statement也是mybatis一个底层封装对象，它包装了mybatis配置信息及sql映射信息等。mapper.xml文件中一个sql对应一个Mapped Statement对象，sql的id即是Mapped statement的id。
- Mapped Statement对sql执行输入参数进行定义，包括HashMap、基本类型、pojo，Executor通过Mapped Statement在执行sql前将输入的java对象映射至sql中，输入参数映射就是jdbc编程中对preparedStatement设置参数。
- Mapped Statement对sql执行输出结果进行定义，包括HashMap、基本类型、pojo，Executor通过Mapped Statement在执行sql后将输出结果映射至java对象中，输出结果映射过程相当于jdbc编程中对结果的解析处理过程。

# JVM垃圾回收

Java内存

![img](https://kstar-1253855093.cos.ap-nanjing.myqcloud.com/baguwenpdf/_Jvm%E5%86%85%E5%AD%98_%E5%AE%89%E4%B9%8B%E8%8B%A5%E7%B4%A0.png)

![img](https://kstar-1253855093.cos.ap-nanjing.myqcloud.com/baguwenpdf/_%E6%80%9D%E7%BB%B4%E5%AF%BC%E5%9B%BE_JVM_ym.png)

1. 哪些内存需要回收

   JVM的内存结构包括五大区域：程序计数器、虚拟机栈、本地方法栈、堆区、方法区。其中程序计数器、虚拟机栈、本地方法栈3个区域随线程而生、随线程而灭，因此这几个区域的内存分配和回收都具备确定性。而Java堆区和方法区的内存是动态的，需要进行垃圾回收。

2. 如何判断一个对象是垃圾

   (1) 引用计数法：堆中每个对象实例都有一个引用计数。当一个对象被创建时，就将该对象实例分配给一个变量，该变量计数设置为1。当任何其它变量被赋值为这个对象的引用时，计数加1，但当一个对象实例的某个引用超过了生命周期或者被设置为一个新值时，对象实例的引用计数器减1，高效但是无法处理循环引用，现在基本已经抛弃。

   (2) 可达性分析算法：根搜索算法的中心思想，就是从某一些指定的根对象（GC Roots）出发，一步步遍历找到和这个根对象具有引用关系的对象，然后再从这些对象开始继续寻找，从而形成一个个的引用链（其实就和图论的思想一致），然后不在这些引用链上面的对象便被标识为引用不可达对象。

   在Java语言中，可作为GC Roots的对象包括下面几种：

   - 虚拟机栈中引用的对象（栈帧中的本地变量表）；
   - 方法区中类静态属性引用的对象；方法区中常量引用的对象；
   - 本地方法栈中JNI（Native方法）引用的对象。

3. 方法区回收判断：

   方法区主要回收的内容有：**废弃常量**和**无用的类**。
   对于废弃常量也可通过引用的可达性来判断，但是对于无用的类则需要同时满足下面3个条件：

   - 该类所有的实例都已经被回收，也就是Java堆中不存在该类的任何实例；
   - 加载该类的ClassLoader已经被回收；
   - 该类对应的java.lang.Class对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。

4. 常用垃圾回收算法：

   (1) 标记-清除法：标记的过程其实就是上面的 **可达性算法(根搜索)** 所标记的不可达对象，当所有的待回收的“垃圾对象”标记完成之后，便进行第二个步骤：**统一清除**。优点是性能比较高，缺点是容易产生不连续的内存块。

   (2) 标记-整理法：该算法并不会直接清除掉可回收对象 ，而是让所有的对象都向一端移动，然后将端边界以外的内存全部清理掉。

   (3) 复制算法：复制算法将内存区域均分为了两块（记为S0和S1），而每次在创建对象的时候，只使用其中的一块区域（例如S0），当S0使用完之后，便将S0上面存活的对象全部复制到S1上面去，然后将S0全部清理掉。缺点是费内存。

5. 分代收集：

   它的核心思想是根据对象存活的生命周期将内存划分为若干个不同的区域。一般情况下将堆区划分为**老年代（Tenured Generation）** 和 **新生代（Young Generation）**，在堆区之外还有一个代就是**永久代（Permanet Generation）(JDK1.8后移除)**。老年代的特点是每次垃圾收集时只有少量对象需要被回收，而新生代的特点是每次垃圾回收时都有大量的对象需要被回收，那么就可以根据不同代的特点采取最适合的收集算法。

6. 垃圾收集器：

   - 新生代可配置的回收器：Serial、ParNew、Parallel Scavenge
   - 老年代配置的回收器：CMS、Serial Old、Parallel Old

# B树/B+树

B+ 树就是对 B 树做了一个升级，MySQL 中索引的数据结构就是采用了 B+ 树，B+ 树结构如下图：

![图片](https://img-blog.csdnimg.cn/img_convert/b6678c667053a356f46fc5691d2f5878.png)

B+ 树与 B 树差异的点，主要是以下这几点：

- 叶子节点（最底部的节点）才会存放实际数据（索引+记录），非叶子节点只会存放索引；
- 所有索引都会在叶子节点出现，叶子节点之间构成一个有序链表；
- 非叶子节点的索引也会同时存在在子节点中，并且是在子节点中所有索引的最大（或最小）。
- 非叶子节点中有多少个子节点，就有多少个索引；

# Java多线程

![img](https://kstar-1253855093.cos.ap-nanjing.myqcloud.com/baguwenpdf/_ThreadLocal%E6%80%9D%E7%BB%B4%E5%AF%BC%E5%9B%BE_%E6%BA%AF%E9%A3%8E.png)

**僵尸进程**：一个**进程**使用fork创建子**进程**，如果子**进程**退出，而父**进程**并没有调用wait 或waitpid 获取子**进程**的状态信息，那么子**进程**的**进程**描述符仍然保存在系统中，这种**进程**称之为僵死**进程**。 

**孤儿进程**：一个父**进程**退出，而它的一个或多个子**进程**还在运行，那么这些子**进程**将成为**孤儿进程**。

**Java多线程的创建**

**通过Executors类提供的方法。**

1. newCachedThreadPool

   创建一个可缓存的线程池，若线程数超过处理所需，缓存一段时间后会回收，若线程数不够，则新建线程。

2. newFixedThreadPool

   创建一个固定大小的线程池，可控制并发的线程数，超出的线程会在队列中等待。

3. newScheduledThreadPool

   创建一个周期性的线程池，支持定时及周期性执行任务。

4. newSingleThreadExecutor

   创建一个单线程的线程池，可保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。

**通过ThreadPoolExecutor类自定义。**

```Java
public ThreadPoolExecutor(int corePoolSize,
	int maximumPoolSize,
	long keepAliveTime,
	TimeUnit unit,
	BlockingQueue<Runnable> workQueue,
	ThreadFactory threadFactory,
	RejectedExecutionHandler handler) {
	// 省略...
}
```

# 死锁

一般来说，要出现死锁问题需要满足以下条件：

1. 互斥条件：一个资源每次只能被一个线程使用。
2. 请求与保持条件：一个进程因请求资源而阻塞时，对已获得的资源保持不放。
3. 不剥夺条件：进程已获得的资源，在未使用完之前，不能强行剥夺。
4. 循环等待条件：若干进程之间形成一种头尾相接的循环等待资源关系。

只要破坏死锁 4 个必要条件之一中的任何一个，死锁问题就能被解决。

# Java19新特性

1. **记录模式 **（预览版）

使用 ***记录模式*** 增强 Java 编程语言以解构记录值，可以嵌套记录模式和类型模式，实现强大的、声明性的和可组合的数据导航和处理形式。

这是一个预览语言功能。

2. **Linux/RISC-V 移植**

将 JDK 移植到 Linux/RISC-V，目前仅支持 RISC-V 的 RV64GV 配置（包含向量指令的通用 64 位 ISA）。将来可能会考虑支持其他 RISC-V 配置，例如通用 32 位配置 (RV32G)。

3. **外部函数和内存 API （预览版）**

引入一个 API，Java 程序可以通过该 API 与 Java 运行时之外的代码和数据进行互操作。通过该 API 可有效地调用外部函数（ JVM 之外的代码）和安全地访问外部内存（不受 JVM 管理的内存），使得 Java 程序能够调用本机库并处理本机数据，而不会出现 JNI 的脆弱性和危险。

这是个预览版 API 。

4. **虚拟线程（预览版）**

将虚拟线程引入 Java 平台。虚拟线程是轻量级线程，可显著地减少编写、维护和观察高吞吐量并发应用程序的工作量。

虚拟线程 `java.lang.Thread` 是在底层操作系统线程（OS 线程）上运行 Java 代码，但在代码的整个生命周期内不捕获 OS 线程的实例。这意味着许多虚拟线程可以在同一个 OS 线程上运行 Java 代码，从而有效地共享它。

虚拟线程是由 JDK 而不是操作系统提供的线程的轻量级实现，也是**用户模式线程**的一种形式。用户模式线程在 Java 的早期版本中被称为 [“绿色线程”](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fen.wikipedia.org%2Fwiki%2FGreen_threads)，当时操作系统线程的概念还不够成熟和普及， Java 的所有绿色线程都共享一个 OS 线程（M:1 调度），随着线程概念的发展，绿色线程最终被现在的平台线程超越，实现为 OS 线程的包装器（1:1 调度），而最新引入的虚拟线程采用 M:N 调度，其中大量 (M) 虚拟线程被调度为在较少数量 (N) 的 OS 线程上运行。

5. **Vector API （第四次孵化）**

引入一个 API 来表达在运行时能够可靠编译的向量计算，在支持的 CPU 架构上优化向量指令，从而实现优于标量计算的性能。

6. **Switch 模式匹配（第三预览版）**

用 `switch` 表达式和语句的模式匹配，以及对模式语言的扩展来增强 Java 编程语言。将模式匹配扩展到 `switch` 中，允许针对一些模式测试表达式，这样就可以简明而安全地表达复杂的面向数据的查询。

该特性最早在 Java 17 中作为预览版出现， Java 19 为第三次预览。

7. **结构化并发（孵化阶段）**

引入用于结构化并发的 API 来简化多线程编程，结构化并发将不同线程中运行的多个任务视为单个工作单元，从而简化错误处理、提高可靠性并增强可观察性。

这是一个孵化阶段的 API。

**Java 19创建platform thread**

`Runnable gcdRunnable = new GCDRunnable();`

`Thread thread = Thread .ofPlatform() .unstarted(gcdRunnable);`

**Structured Concurrency**：

为了让程序更易于读，理解，更快的写和更安全。

避免线程泄漏和孤儿线程，线程的生命周期都被限制在一个封闭的范围中。

模仿structured programming， 有明确的入口和出口点的执行流代码块；严格嵌套操作的生存期，以反映代码中的语法嵌套。

StructuredTaskScope是Structured Concurrency的基本接口，其定义了几个子类：ShutdownOnFailure和ShutdownOnSuccess。

# Docker

Docker的优势总结如下：

- 更快的启动时间。Docker容器启动是几秒钟的事情，因为容器只是一个操作系统进程而已。带有完整操作系统的虚拟机则需要几分钟来加载。
- 更快部署。不需要建立一个新的环境。使用Docker,Web开发团队只需要下载Docker镜像并在不同的服务器上运行。
- 容器更易管理与扩展。因为销毁与运行容器比销毁与运行虚拟机更快。
- 计算资源的更好利用，因为在一个服务器上你可以运行的容器比虚拟机要多。
- 支持多种操作系统，Windows,Mac,Debian等等。

# Hash算法

# HTTP和HTTPS

# 

# Zookeeper

Design
• Provide a reusable solution.
• Provide a basic kernel or a coordination core upon with applications can build more sophisticated
coordination mode.
• Move away from the server-oriented model. Instead provide some APIs.
• Move away from blocking primitives so as not to slow down an entire service just because some
components are slow.
• Support for FIFO client request ordering and linearizable writes

Participants and Service Model
• Client: use of the Zookeeper service.
• Server: ensemble of replicated services for load balancing and fault tolerance.
• Session: client connects to the server to closing,
• znode: in memory data-object for client. Like traditional file system but not store much file.
	Ephemeral znode: Remain alive while in session and cleaned while close.
	Regular znode: created by user and root node maintained by Zookeeper service
