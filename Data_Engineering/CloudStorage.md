# This is the notebook of data engineering about cloud storage  

数据管理的工具（Tools for data management）  
1 文件和格式  
2 数据库，分为SQL和NoSQL的解决方法，以及NoDB  
3 非结构化数据的表示  
4 分布式文件系统（HDFS及GlusterFS和Ceph）  
5 以云为基础的存储（暂时性的，卷，对象存储）  

## 文件和格式  
非专用格式  
1 通用格式：CSV，PDF，TIFF，XML等  
2 专用于科学数据的格式：   
• ROOT (used at CERN-欧洲核子研究实验室)   
• NetCDF (Network Common Data Form)   
• HDF5 (Hierarchical Data Format 分层数据格式)   
• RCFile (Record Columnar File 记录的柱状文件)     
• ORC (Optimised Row Columnar 优化的行柱状)    

## 数据库 Databases     
数据库用于结构化的数据：  
• 关系型（广泛使用的）• 面向对象型  
优点：  
•稳定的系统可用  
•减少数据冗余 Reduced data redundancy  
•增强安全性  
•提供ACID（原子性，一致性，隔离性，耐久性）属性

### ACID
A – Atomicity – 原子性  
一个事务（transaction）中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有被执行过一样。  
C – Consistency – 一致性  
在事务开始之前和事务结束以后，数据库的完整性没有被破坏。这表示写入的资料必须完全符合所有的预设规则，这包含资料的精确度、串联性以及后续数据库可以自发性地完成预定的工作。  
I – Isolation – 隔离性  
数据库允许多个并发事务同时对其数据进行读写和修改的能力，隔离性可以防止多个事务并发执行时由于交叉执行而导致数据的不一致。事务隔离分为不同级别，包括读未提交（Read uncommitted）、读提交（read committed）、可重复读（repeatable read）和串行化（Serializable）。  
D – Durability – 持久性  
事务处理结束后，对数据的修改就是永久的，即便系统故障也不会丢失。    

## NoSQL databases (NoSQL=Not Only SQL)  
NoSQL DEFINITION:
Next Generation Databases mostly addressing some of the points: being non-relational, distributed, open-source and horizontally scalable. 非关系型，分布式，开源，水平扩展性。

### CAP定理（CAP theorem）

在计算机科学中, CAP定理（CAP theorem）, 又被称作 布鲁尔定理（Brewer's theorem）, 它指出对于一个分布式计算系统来说，不可能同时满足以下三点:
一致性(Consistency) (所有节点在同一时间具有相同的数据)
可用性(Availability) (保证每个请求不管成功或者失败都有响应)
分隔容忍(Partition tolerance) (系统中任意信息的丢失或失败不会影响系统的继续运作)

CAP理论的核心是：一个分布式系统不可能同时很好的满足一致性，可用性和分区容错性这三个需求，最多只能同时较好的满足两个。

因此，根据 CAP 原理将 NoSQL 数据库分成了满足 CA 原则、满足 CP 原则和满足 AP 原则三大类：  
CA - 单点集群，满足一致性，可用性的系统，通常在可扩展性上不太强大。  
若系统选CA，则为了保证不出现分区，通常是单点集群，或者花费更多的性能用于维护读写一致性、事务与关联操作，传统的RDBMS就是如此。  
CP - 满足一致性，分区容忍性的系统，通常性能不是特别高。  
在分区的时候选择CP，则分布式系统停止服务，直到完成节点数据一致性为止，此时系统满足CP原则，如MongDB、HBase、Redis、Neo4j  
AP - 满足可用性，分区容忍性的系统，通常可能对一致性要求低一些。  
在分区的时候选择AP，则分布式系统继续提供服务，但是不保证数据一致，两次请求结果可能不同，此时系统满足AP原则，如Amazon Dynamo数据库  
![image](https://user-images.githubusercontent.com/65893273/113516114-248b7e80-95ab-11eb-80e5-6a1e6621f074.png)  

### BASE

BASE：Basically Available, Soft-state, Eventually Consistent。由eBay的架构师Dan Pritchett源于对大规模分布式系统的实践总结，在ACM上发表文章提出BASE理论，BASE理论是对CAP理论的延伸，核心思想是即使无法做到强一致性（Strong Consistency，CAP的一致性就是强一致性），但应用可以采用适合的方式达到最终一致性（Eventual Consitency）。  

BASE模型（反ACID模型），完全不同于ACID模型，牺牲高一致性，以获得可用性或可靠性：  
Basically Available 基本可用：  
基本可用是指分布式系统在出现故障的时候，允许损失部分可用性，即保证核心可用，支持分区失败(e.g. sharding碎片，划分数据库)。  
电商大促时，为了应对访问量激增，部分用户可能会被引导到降级页面，服务层也可能只提供降级服务。这就是损失部分可用性的体现。  
Soft state 软状态：   
状态可以有一段时间不同步，异步。是指允许系统存在中间状态，而该中间状态不会影响系统整体可用性。分布式存储中一般一份数据至少会有三个副本，允许不同节点间副本同步的延时就是软状态的体现。mysql replication的异步复制也是一种体现  
Eventually consistent 最终一致：  
最终数据是一致的就可以了，而不是时时一致。是指系统中的所有数据副本经过一定时间后，最终能够达到一致的状态。弱一致性和强一致性相反，最终一致性是弱一致性的一种特殊情况  

BASE思想的主要实现有    
1.按功能划分数据库    
2.sharding碎片，即分多个节点存储    

总结：  
BASE思想主要强调基本的可用性，如果需要高可用性，也就是纯粹的高性能，那么就要以一致性或容错性为牺牲，BASE思想的方案在性能上还是有潜力可挖的。  
ACID是传统数据库常用的设计理念，追求强一致性模型。BASE支持的是大型分布式系统，提出通过牺牲强一致性获得高可用性。  
ACID和BASE代表了两种截然相反的设计哲学，在分布式系统设计的场景中，系统组件对一致性要求是不同的，因此ACID和BASE又会结合使用。       

ACID vs BASE

| ACID                 |BASE                            | 
| :------              |:------|
| 原子性(Atomicity)	  |基本可用(Basically Available）    |   
| 一致性(Consistency)  |软状态或柔性事务(Soft state）      |   
| 隔离性(Isolation)	  |最终一致性 (Eventual consistency) |   
| 持久性 (Durable)     |                                 |  

### ACID、CAP、BASE的联系  
ACID：是RDBMS（关系型数据库）中遵循的事务处理基本原则，但是也是影响其性能的原因，NoSQL是分布式数据库一般不保证ACID原则。  
CAP：CAP理论是针对分布式系统而言的。CAP理论的核心是：一个分布式系统不可能同时很好的满足一致性，可用性和分区容错性这三个需求，最多只能同时较好的满足两个。  
BASE: 与ACID是RDBMS强一致性的四个要求对应，BASE是NoSQL通常对可用性及一致性的弱要求原则，它们的意思分别是，BASE：Basically Available（基本可用）, Soft-state（软状态/柔性事务。 "Soft state" 可以理解为"无连接"的）, Eventually Consistent（最终一致性）。    

联系：    
理解CAP理论只是指多个数据副本之间读写一致性的问题，那么它对RDBMS与NoSQL数据库来讲是完全一样的，它只是运行在分布式环境中的数据管理设施在设计读写一致性问题时需要遵循的一个原则而已，却并不是NoSQL数据库具有优秀的水平可扩展性的真正原因。而如果将CAP理论中的一致性C理解为读写一致性、事务与关联操作的综合，则可以认为关系型数据库选择了C与A，而NoSQL数据库则全都是选择了C与P。这才是用CAP理论来支持NoSQL数据库设计正确认识。

### 不同类型：  
键值（key-value store）:
例子：BerkleyDB, DyanmoDB, Redis    
应用场景：内容缓存，主要用于处理大量数据的高访问负载，也用于一些日志系统等等。  
数据模型：Key 指向 Value 的键值对，Value格式不限，通常用hash table来实现  
特点：可以通过key快速查询到其value。一般来说，存储不管value的格式，照单全收。（Redis包含了其他功能）    

列存储数据库（Column store）：  
例子：HBase, BigTable, Cassandra    
应用场景：分布式的文件系统    
数据模型：以列簇式存储，将同一列数据存在一起    
特点：顾名思义，是按列存储数据的。最大的特点是方便存储结构化和半结构化数据，方便做数据压缩，对针对某一列或者某几列的查询有非常大的IO优势。    

文档型数据库（Document store）：  
例子：CouchDB， MongoDb    
应用场景：Web应用（与Key-Value类似，Value是结构化的，不同的是数据库能够了解Value的内容）    
数据模型：Key-Value对应的键值对，Value为结构化数据        
特点：文档存储一般用类似json的格式存储，存储的内容是文档型的。这样也就有机会对某些字段建立索引，实现关系数据库的某些功能。    

图形(Graph)数据库（Graph store）：    
例子：Neo4J， InfoGrid， Infinite Graph ，polyglot   
应用场景：社交网络，推荐系统等。专注于构建关系图谱    
数据模型：图结构    
特点：图形关系的最佳存储。使用传统关系数据库来解决的话性能低下，而且设计使用不方便。可以利用图结构相关算法。比如最短路径寻址，N度关系查找等。   

对象存储：    
例子：db4o，Versant  
特点：通过类似面向对象语言的语法操作数据库，通过对象的方式存取数据。

#### 哈希表
散列表（Hash table，也叫哈希表），是根据关键码值(Key value)而直接进行访问的数据结构。也就是说，它通过把关键码值映射到表中一个位置来访问记录，以加快查找的速度。这个映射函数叫做散列函数，存放记录的数组叫做散列表。  
给定表M，存在函数f(key)，对任意给定的关键字值key，代入函数后若能得到包含该关键字的记录在表中的地址，则称表M为哈希(Hash）表，函数f(key)为哈希(Hash) 函数。  

• 若关键字为k，则其值存放在f(k)的存储位置上。由此，不需比较便可直接取得所查记录。称这个对应关系f为散列函数，按这个思想建立的表为散列表。  
• 对不同的关键字可能得到同一散列地址，即k1≠k2，而f(k1)=f(k2)，这种现象称为冲突（英语：Collision），又叫哈希碰撞。具有相同函数值的关键字对该散列函数来说称做同义词。  
• 若对于关键字集合中的任一个关键字，经散列函数映象到地址集合中任何一个地址的概率是相等的，则称此类散列函数为均匀散列函数（Uniform Hash function），这就是使关键字经过散列函数得到一个“随机的地址”，从而减少冲突  

哈希表是一种表结构，我们可以直接根据给定的key值计算出目标位置。在工程中这一表结构实现通常采用数组。  
与普通的列表不同的地方在于，普通列表仅能通过下标来获取目标位置的值，而哈希表可以根据给定的key计算得到目标位置的值。  
在列表查找中，使用最广泛的二分查找算法，复杂度为O(log2n)，但其始终只能用于有序列表。普通无序列表只能采用遍历查找，复杂度为O(n)。  
而拥有较为理想的哈希函数实现的哈希表，对其任意元素的查找速度始终为常数级，即O(1)。  

实例：  
在典型的哈希表实现中，将数组总长度设为模数，将key值直接对其取模，所得的值为数组下标。（取模即为f哈希函数）  
![image](https://user-images.githubusercontent.com/65893273/113517280-83ec8d00-95b1-11eb-9ab0-296e1c90f11a.png)
如图所示的三组数据，key为32，23，64，数组总长度为8。  
取模后，分别被映射到下标为0和7的位置中，显而易见的，第1组数据和第3组数据发生了哈希碰撞。    

解决哈希碰撞的方法：    
常用的解决方案有散列法和拉链法。  
散列法又分为开放寻址法和再散列法等。  
拉链法，即：在每个冲突处构建链表，将所有冲突值链入链表，如同拉链一般一个元素扣一个元素，故名拉链法。  
![image](https://user-images.githubusercontent.com/65893273/113517385-315fa080-95b2-11eb-91b7-171abc512764.png)  
如图所示，哈希表地址为0-15.在地址0处，存放一个链表（496->896）。  
需要注意的是，如果遭到恶意哈希碰撞攻击，拉链法会导致哈希表退化为链表，即所有元素都被存储在同一个节点的链表中，此时哈希表的查找速度=链表遍历查找速度=O(n)。

## 分布式文件系统（Distributed FileSystems）
• NFS (Network FileSystem)  
NFS（Network File System，网络文件系统）是当前主流异构平台共享文件系统之一。主要应用在UNIX环境下。最早是由Sun Microsystems开发，现在能够支持在不同类型的系统之间通过网络进行文件共享，广泛应用在FreeBSD、SCO、Solaris等异构操作系统平台，允许一个系统在网络上与他人共享目录和文件。通过使用NFS，用户和程序可以像访问本地文件一样访问远端系统上的文件，使得每个计算机的节点能够像使用本地资源一样方便地使用网上资源。换言之，NFS可用于不同类型计算机、操作系统、网络架构和传输协议运行环境中的网络文件远程访问和共享。  
NFS的工作原理是使用客户端/服务器架构，由一个客户端程序和服务器程序组成。服务器程序向其他计算机提供对文件系统的访问，其过程称为输出。NFS客户端程序对共享文件系统进行访问时，把它们从NFS服务器中“输送”出来。文件通常以块为单位进行传输。其大小是8KB（虽然它可能会将操作分成更小尺寸的分片）。NFS传输协议用于服务器和客户机之间文件访问和共享的通信，从而使客户机远程地访问保存在存储设备上的数据。  
• Ceph Storage  
Ceph是一种为优秀的性能、可靠性和可扩展性而设计的统一的、分布式文件系统。  
• GlusterFS (Gnu Cluster FileSystem )  
GlusterFS是Scale-Out存储解决方案Gluster的核心，它是一个开源的分布式文件系统，具有强大的横向扩展能力，通过扩展能够支持数PB存储容量和处理数千客户端。GlusterFS借助TCP/IP或InfiniBand RDMA网络将物理分布的存储资源聚集在一起，使用单一全局命名空间来管理数据  
• HDFS (Hadoop Distributed FileSystem)  
Apache公司开发，课程主要使用的工具  
• MooseFS  
MooseFS是一种分布式文件系统。基于linux内核，提供整套分布式文件服务。    




