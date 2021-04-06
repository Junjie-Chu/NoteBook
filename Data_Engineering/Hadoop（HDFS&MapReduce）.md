# Hadoop（HDFS&MapReduce）

## Hadoop 的组成  
(1) Hadoop HDFS：一个高可靠、高吞吐量的分布式文件系统；  
(2) Hadoop MapReduce：一个分布式的离线并行计算框架；  
(3) Hadoop YARN：作业调度与集群资源管理的框架；  
(4) Hadoop Common：支持其他模块的工具模块。  

## HDFS原理  
HDFS 是 Hadoop 分布式文件系统，并且运行在普通硬件上。这就意味着 HDFS 不需要优秀的硬件资源、高昂的硬件成本，只需要简单的物理机组成分布式集群，HDFS 使用横向拓展（增加机器）来提高存储容量，而非纵向扩展（提高单个机器的配置）。HDFS 并非唯一的分布式文件系统，还有 GFS、TFS 等，但 HDFS 是使用最多的开源分布式文件存储系统，具有高度容错（highly fault-tolerant）及低成本的特点。  
### HDFS架构及构成
![image](https://user-images.githubusercontent.com/65893273/113686703-1f891500-96fa-11eb-95c4-903399154923.png)   
HDFS是master/slave结构。  
HDFS Client  
和 HDFS 打交道也是通过一个 client library. 无论读取一个文件或者写一个文件，我们都是把数据交给 HDFS client，它负责和 Name nodes 以及 Data nodes 联系并传输数据。  
（1）文件切分。文件上传HDFS的时候，Client将文件切分成一个一个的Block，然后进行存储；    
（2）与NameNode交互，获取文件的位置信息；  
（3）与DataNode交互，读取或者写入数据；  
（4）Client提供一些命令来管理HDFS，比如启动或者关闭HDFS；  
（5）Client可以通过一些命令来访问HDFS；

Name Nodes  
HDFS 把文件存在多个机器上，并且不把“在哪些机器上存的”，“如何存的”这些内部的信息暴露给使用者，而是只显示给用户一个像普通 linux 文件结构的文件系统。这些内部信息存在Name nodes中。在 HDFS 里， Name node 保存了整个文件系统信息，包括文件和文件夹的结构。类似于Linux， HDFS 也是把文件和文件夹表示为 inode, 每个 inode 有自己的所有者，权限，创建和修改时间等等。HDFS 可以存很大的文件，所以每个文件都被分成一些 data block，存在不同机器上, name node 就负责记录一个文件有哪些 data block，以及这些 data block 分别存放在哪些机器上。Name nodes 还负责管理文件系统常用操作，比如创建一个文件，重命名一个文件，创建一个文件夹，重命名一个文件夹等。当我们通过 HDFS client 向 HDFS 读取或者写文件时，所有的读写请求都是先发给 Name nodes, 它负责创建或者查询一个文件，然后再让 HDFS client 和 Data nodes 联系具体的数据传输。  
（1）管理HDFS的名称空间；  
（2）管理数据块（Block）映射信息；  
（3）配置副本策略；机架感应等。  
（4）处理客户端读写请求。  

Data Nodes  
存储 data block 的机器叫做 Data nodes. 在读写过程中，Data nodes 负责直接把用户读取的文件 block 传给 client，也负责直接接收用户写的文件。当我们读取一个文件时：HDFS client 联系 Name nodes，获取文件的 data blocks 组成、以及每个 data block 所在的机器以及具体存放位置；HDFS client 联系 Data nodes, 进行具体的读写操作。  
（1）存储实际的数据块；  
（2）执行数据块的读/写操作。  

Secondary NameNode：热备份指挂掉之后可以接替工作的。冷备一般是备份数据等，帮助恢复，不能直接马上替代。并非NameNode的热备。当NameNode挂掉的时候，它并不能马上替换NameNode并提供服务。    
（1）辅助NameNode，分担其工作量；  
（2）定期合并Fsimage和Edits，并推送给NameNode；  
（3）在紧急情况下，可辅助恢复NameNode。  

Blockreport  
NameNode 会与 DataNode 之间会通过心跳机制进行通信，每个 DataNode 会定期向 NameNode 发送心跳以及 Blockreport ，Blockreport 上包含了该 DataNode 上的 block 列表，这种心跳机制也是 NameNode 检测 DataNode 是否存活的依据。默认发送心跳的时间是 3 秒，默认判断 DataNode 是否存活的时间是 10 分钟，也就是 10 分钟接收不到该 DataNode 的心跳，则认为它已经宕机，不会再与该 DataNode 发送读写操作。重写副本的触发条件：DataNode 节点不可用、某一个备份文件处于故障状态、DataNode 磁盘出现故障、备份数量发生改变。如果Name Node宕机了，hdfs系统就出问题了，需要修复。  

注意：在读写一个文件时，当我们从 Name nodes 得知应该向哪些 Data nodes 读写之后，我们就直接和 Data node 打交道，不再通过 Name nodes.  

### HDFS冗余备份
![image](https://user-images.githubusercontent.com/65893273/113687252-b655d180-96fa-11eb-8f92-85eb2f150cf6.png)  
以文件 part-0 为例，备份数是 2，block id 是 1 和 3，在 Datanodes 中可以找到，id 为 1 的 block 有两个，id 为 3  的 block 有两个，分别存储在不同 Datanode。两个不同 block id 组合起来就是一个完整的名称为 part-0 的文件。同时，由于备份数是2，在图中的data nodes中我们可以看见，1有两份，3也有两份。  
如果块大小是128mb，文件大小是200mb，则需要分成两个block，第一个block大小是128mb，第二个block块大小是128mb，但是实际占用大小是72mb。这与普通文件系统不同，在普通文件系统中，如果一个文件是5kb，分块是4kb，那这个文件需要2个块，实际占用的空间为8kb。  

### HDFS写操作
![image](https://user-images.githubusercontent.com/65893273/113688230-af7b8e80-96fb-11eb-8de6-47b2d5a11a7c.png)  
1、客户端向NameNode发出写文件请求。  
2、NameNode检查是否已存在文件、检查权限。若通过检查，直接先将操作写入EditLog，并返回输出流对象。响应是否可以上传。      
（注：WAL，write ahead log，先写Log，再写内存，因为EditLog记录的是最新的HDFS客户端执行所有的写操作。如果后续真实写操作失败了，
由于在真实写操作之前，操作就被写入EditLog中了，故EditLog中仍会有记录）  
3、client端按128MB的块切分文件。向Namenode发送请求上传第一个block。  
4、NameNode返回datanode的名单，表示该块存储在哪些datanode中，发送给client。  
5、client将NameNode返回的DataNode列表和Data数据一同发送给最近的第一个DataNode节点，然后该datanode节点会与剩下的节点建立传输通道，通道连通后返回确认信息给客户端；表示通道已连通，可以传输数据。  
6、 客户端收到确认信息后，通过网络向就近的datanode节点写第一个block块的数据；就近的datanode收到数据后，首先会缓存起来；然后将缓存里数据保存一份到本地，一份发送到传输通道；让剩下的datanode做备份。  
client将NameNode返回的分配的可写的DataNode列表和Data数据一同发送给最近的第一个DataNode节点，此后client端和NameNode分配的多个DataNode构成pipeline管道，client端向输出流对象中写数据。client每向第一个DataNode写入一个packet，这个packet便会直接在pipeline里传给第二个、第三个…DataNode。  
（注：并不是写好一个块或一整个文件后才向后分发）  
每个DataNode写完一个块后，会返回确认信息。    
7、第一个block块写入完毕，若客户端还有剩余的block未上传；则客户端会从（3）开始，继续执行上述步骤；直到整个文件上传完毕。写完数据，关闭输输出流。    
8、发送完成信号给NameNode。    
注：发送完成信号的时机取决于集群是强一致性还是最终一致性，强一致性则需要所有DataNode写完后才向NameNode汇报。最终一致性则其中任意一个DataNode写完后就能单独向NameNode汇报，HDFS一般情况下都是强调强一致性）    
图对应版本的流程：  
1）客户端通过Distributed FileSystem模块向NameNode请求上传文件，NameNode检查目标文件是否已存在，父目录是否存在。  
2）NameNode返回是否可以上传。  
3）客户端请求第一个 block上传到哪几个datanode服务器上。  
4）NameNode返回3个datanode节点，分别为dn1、dn2、dn3。  
5）客户端通过FSDataOutputStream模块请求dn1上传数据，dn1收到请求会继续调用dn2，然后dn2调用dn3，将这个通信管道建立完成。  
6）dn1、dn2、dn3逐级应答客户端。  
7）客户端开始往dn1上传第一个block（先从磁盘读取数据放到一个本地内存缓存），以packet为单位，dn1收到一个packet就会传给dn2，dn2传给dn3；dn1每传一个packet会放入一个应答队列等待应答。  
8）当一个block传输完成之后，客户端再次请求NameNode上传第二个block的服务器。（重复执行3-7步）。  

### HDFS读操作
![image](https://user-images.githubusercontent.com/65893273/113690241-c622e500-96fd-11eb-9e08-b0a452a073b8.png)  
1）客户端通过Distributed FileSystem向NameNode请求下载文件。NameNode通过查询元数据，找到文件块所在的DataNode地址，发送给客户端。  
2）客户端根据接收到的datanode列表，挑选一台DataNode（就近原则，然后随机）服务器，请求读取数据。  
3）DataNode开始传输数据给客户端（从磁盘里面读取数据输入流，以Packet为单位来做校验）。  
···假设第一块的数据读完了，就会关闭指向第一块的datanode连接。接着读取下一块。这些操作对client来说是透明的，client的角度看来仅仅是读一个持续不断的流。  
···假设第一批block都读完了， DFSInputStream就会去namenode拿下一批block的locations。然后继续读。假设全部的块都读完，这时就会关闭掉全部的流。  
···假设在读数据的时候， DFSInputStream和datanode的通讯发生异常。就会尝试正在读的block的排序第二近的datanode,而且会记录哪个datanode错误发生，剩余的blocks读的时候就会直接跳过该datanode。 DFSInputStream也会检查block数据校验和，假设发现一个坏的block,就会先报告到namenode节点，然后DFSInputStream在其它的datanode上读该block的镜像。  
4）客户端以Packet为单位接收，先在本地缓存，然后写入目标文件。  

注意，在读和写的过程中，client直接连接datanode来检索数据，而namenode来负责为每个block提供最优的datanode。 namenode仅仅处理block location的请求，这些信息都载入在namenode的内存中。hdfs通过datanode集群能够承受大量client的并发访问。  

### HDFS里secondary namenode的机制
![image](https://user-images.githubusercontent.com/65893273/113692201-ddfb6880-96ff-11eb-85b0-ab1d4acd4954.png)    
1）第一阶段：NameNode启动  
（1）第一次启动NameNode格式化后，创建fsimage和edits文件。如果不是第一次启动，直接加载编辑日志和镜像文件到内存。  
（2）客户端对元数据进行增删改的请求。  
（3）NameNode记录操作日志，更新滚动日志。  
（4）NameNode在内存中对数据进行增删改查。  
2）第二阶段：Secondary NameNode工作  
（1）Secondary NameNode询问NameNode是否需要checkpoint。直接带回NameNode是否检查结果。  
（2）Secondary NameNode请求执行checkpoint。  
（3）NameNode滚动正在写的edits日志。  
（4）将滚动前的编辑日志和镜像文件拷贝到Secondary NameNode。  
（5）Secondary NameNode加载编辑日志和镜像文件到内存，并合并。  
（6）生成新的镜像文件fsimage.chkpoint。  
（7）拷贝fsimage.chkpoint到NameNode。  
（8）NameNode将fsimage.chkpoint重新命名成fsimage 

NameNode与SecondaryNameNode 的区别与联系？     
1）区别  
（1）NameNode负责管理整个文件系统的元数据，以及每一个路径（文件）所对应的数据块信息。  
（2）SecondaryNameNode主要用于定期合并命名空间镜像和命名空间镜像的编辑日志。  
2）联系：  
（1）SecondaryNameNode中保存了一份和namenode一致的镜像文件（fsimage）和编辑日志（edits）。  
（2）在主namenode发生故障时（假设没有及时备份数据），可以从SecondaryNameNode恢复数据。 

## MapReduce原理（映射化简）
### MapReduce流程
![image](https://user-images.githubusercontent.com/65893273/113695516-83fca200-9703-11eb-942c-b046517194ee.png)  
Key stages of MapReduce  
1.Split input  
2.Map  
3.Shuffle  
4.Reduce  
我们专注于Map和Reduce，洗牌（shuffle）等由架构完成。MapReduce就是分治，并行处理。

Reduce  
某个键的所有键值对都会被分发到同一个reduce操作中。确切的说，这个键和这个键所对应的所有值都会被传递给同一个Reducer。reduce过程的目的是将值的集合转换成一个值（例如求和或者求平均），或者转换成另一个集合。这个Reducer最终会产生一个键值对。需要说明的是，如果job不需要reduce过程的话，那么reduce过程也是可以不用的。    
单个reduce任务的输入通常来自于所有mapper的输出。排过序的map输出需通过网络传输发送到运行reduce任务的节点，数据在reduce端合并并由用户定义的reduce函数处理。  
reduce任务的数量并非由输入数据的大小决定，而是独立指定的。  
真实的应用中，几乎所有作业都会把reducer的个数设置成较大的数字，否则由于所有中间数据都会放到一个reduce任务中，作业的处理效率就会及其低下。  
增加reducer的数量能缩短reduce进程；但是reducer数量过多又会导致小文件过多而不够优化。一条经验法则是：目标reducer保持每个运行在5分钟左右，且产生至少一个HDFS块的输出比较合适。  


### MapReduce架构
![image](https://user-images.githubusercontent.com/65893273/113707803-0345a200-9713-11eb-8c8a-7e9fd3f491ef.png)  
和HDFS一样，MapReduce也是采用Master/Slave的架构，MapReduce包含四个组成部分，分别为Client、JobTracker、TaskTracker和Task。  
1）Client 客户端    
　　每一个 Job 都会在用户端通过 Client 类将应用程序以及配置参数 Configuration 打包成 JAR 文件存储在 HDFS，并把路径提交到 JobTracker 的 master 服务，然后由 master 创建每一个 Task（即 MapTask 和 ReduceTask） 将它们分发到各个 TaskTracker 服务中去执行。    
2）JobTracker  
   有单点故障。JobTracke负责资源监控和作业调度。JobTracker 监控所有TaskTracker 与job的健康状况，一旦发现失败，就将相应的任务转移到其他节点；同时，JobTracker 会跟踪任务的执行进度、资源使用量等信息，并将这些信息告诉任务调度器，而调度器会在资源出现空闲时，选择合适的任务使用这些资源。在Hadoop中，任务调度器是一个可插拔的模块，用户可以根据自己的需要设计相应的调度器。  
3）TaskTracker  
　　TaskTracker 会周期性地通过Heartbeat 将本节点上资源的使用情况和任务的运行进度汇报给JobTracker，同时接收JobTracker 发送过来的命令并执行相应的操作（如启动新任务、杀死任务等）。TaskTracker 使用"slot"等量划分本节点上的资源量。"slot"代表计算资源（CPU、内存等）。一个Task 获取到一个slot 后才有机会运行，而Hadoop 调度器的作用就是将各个TaskTracker 上的空闲slot分配给Task 使用。slot分为Map slot 和Reduce slot 两种，分别供Map Task 和Reduce Task 使用。TaskTracker 通过slot 数目（可配置参数）限定Task 的并发度。TaskTracker 周期性的向 JobTracker 汇报心跳，如果一定的时间内没有汇报这个心跳，JobTracker 就认为该TaskTracker 挂掉了，它就会把上面所有任务调度到其它TaskTracker（节点）上运行。这样即使某个节点挂了，也不会影响整个集群的运行。  
4）Task  
　　Task 分为Map Task 和Reduce Task 两种，均由TaskTracker 启动。HDFS 以固定大小的block 为基本单位存储数据，而对于MapReduce 而言，其处理单位是split。split 是一个逻辑概念，它只包含一些元数据信息，比如数据起始位置、数据长度、数据所在节点等。它的划分方法完全由用户自己决定。但需要注意的是，split 的多少决定了Map Task 的数目，因为每个split 只会交给一个Map Task 处理。MapTask和ReduceTask 也可能运行挂掉。比如内存超出了或者磁盘挂掉了，这个任务也就挂掉了。 这个时候 TaskTracker 就会把每个MapTask和ReduceTask的运行状态回报给 JobTracker，JobTracker 一旦发现某个Task挂掉了，它就会通过调度器把该Task调度到其它节点上。这样的话，即使任务挂掉了，也不会影响应用程序的运行。    

### MapReduce例子
两个例子：  
1. word count  
![image](https://user-images.githubusercontent.com/65893273/113696197-44828580-9704-11eb-86ec-313602dae080.png)  
内部流程图：  
![image](https://user-images.githubusercontent.com/65893273/113708073-60415800-9713-11eb-99c2-b569123850f3.png)  
input：  
输入一个大文件。  
split：  
将大文件切分为若干小文件（若干行）。此处为0/1/2三行。
map：
将每一行切分为单个单词。每个单词和1形成一个键值对。如（Deer，1）。  
map阶段也可以进行初步化简，比如car，1/car，1/river，1，可以在shuffle之前化简为car，2/river，1  
shuffle 或者 sort：  
现在大部分由架构自己完成。该部分输出为形如【（car，1）(car，1) (car,1)】，【(deer，1) (deer，1)】等  
Reduce：  
对上一步的结果进行化简。  
【（car，1）(car，1) (car,1)】，【(deer，1) (deer，1)】化简为【 (car，3)】，【(deer，2)】  
Finalize:  
合并为一个文件。  

过程中shuffle等是乱序的，执行过程是高度并行的。  

2.倒排索引（单词出现在哪些文档里？）
![image](https://user-images.githubusercontent.com/65893273/113709607-486ad380-9715-11eb-8582-17ea88f44890.png)    
1.首先输入很多的文档，这里面每一行的不是文档编号了，我们要存它的文档编号。  
2.拆解的时候加入新的概念，Worker，这里有三个Worker，每个Worker负责1行，也可以负责多行，每个Worker就需要进行Map拆解，把每个单词都拆成单词和出现的文档ID。（food，2）（music，2）      
3.接着Shuffle有两个新Worker，他们把单词一拆，数据一部分给Worker4，一部分给Worker5，拿到以后会排个序，这也是最基本的Shuffle过程。比如，把food的放在一起。（food，0）（food，2）      
4.接着就是Reduce，把相同的东西合并到一起去，比如food出现在两个文档里，就是0和2。（food，（0，2））.  
5.最终就是把这些结果放到一起去就可以让我们查找了。  












