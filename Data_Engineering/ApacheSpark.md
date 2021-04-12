# Spark基本架构
![image](https://user-images.githubusercontent.com/65893273/114412838-10193880-9be0-11eb-9ce8-b76793508314.png)  
- 构建Spark Application的运行环境，启动SparkContext
- SparkContext向资源管理器（可以是Standalone，Mesos，Yarn）申请运行Executor资源，并启动StandaloneExecutorbackend，
- Executor向SparkContext申请Task
- SparkContext构建成DAG图，将DAG图分解成Stage(数据的本地性体现在这里)、将Taskset发送给Task Scheduler，最后由Task Scheduler将Task发送给Executor运行
- Task在Executor上运行，运行完释放所有资源
![image](https://user-images.githubusercontent.com/65893273/114422973-51621600-9be9-11eb-8860-c54ae1ed7be3.png)  

# Spark应用涉及的基本概念：
1. mater:主要是控制、管理和监督整个spark集群
2. client：客户端，将用应用程序提交，记录着要业务运行逻辑和master通讯。
3. sparkContext：spark应用程序的入口，负责调度各个运算资源，协调各个work node上的Executor。主要是一些记录信息，记录谁运行的，运行的情况如何等。这也是为什么编程的时候必须要创建一个sparkContext的原因了。
4. Driver Program：每个应用的主要管理者，每个应用的老大. driver负责具体事务运行并跟踪，运行Application的main()函数并创建sparkContext。
5. RDD：spark的核心数据结构，可以通过一系列算子进行操作，当Rdd遇到Action算子时，将之前的所有的算子形成一个有向无环图(DAG)。再在spark中转化成为job，提交到集群执行。一个app可以包含多个job
6. worker Node:集群的工作节点，可以运行Application代码的节点，接收mater的命令并且领取运行任务，同时汇报执行的进度和结果给master，节点上运行一个或者多个Executor进程。
7. exector：为application运行在workerNode上的一个进程，该进程负责运行Task，并且负责将数据存在内存或者磁盘上。每个application都会申请各自的Executor来处理任务。

# 依赖,DAG有向无环图,stage划分
- 窄依赖：  
父RDD的一个分区只去到了子RDD的一个分区,没有shuffle.  
![image](https://user-images.githubusercontent.com/65893273/114427621-d8b18880-9bed-11eb-889e-d1bb332222fc.png)  
- 宽依赖  
父RDD的一个分区去到了子RDD的至少两个分区,出现shuffle.
- DAG  
指明的数据的流向,即RDD之间的依赖关系.   
- Stage切割  
Stage切割规则：从后往前，遇到宽依赖就切割stage 

流程:    
1. Spark任务会根据RDD之间的依赖关系，形成一个DAG有向无环图，
2. DAG会提交给DAGScheduler，DAGScheduler会把DAG划分成互相依赖的多个stage，划分stage的依据就是RDD之间的宽窄依赖,遇到宽依赖就划分stage，窄依赖继续.   
3. 每个stage包含一个或多个task任务，然后将这些task以taskSet的形式提交给TaskScheduler运行。stage是由一组并行的task组成。  

![image](https://user-images.githubusercontent.com/65893273/114429415-f384fc80-9bef-11eb-82da-31e1027dac75.png)  
DAG的最后一个阶段会为每个stage的最后一个rdd的每个partition生成一个ResultTask，即每个Stage里面的Task的数量是由该Stage中最后一个RDD的Partition的数量所决定的.    
而其余所有阶段(rdd从每个stage流出时)都会生成ShuffleMapTask.      
之所以称之为ShuffleMapTask是因为它需要将自己的计算结果通过shuffle到下一个stage中；    
也就是说上图中的stage1和stage2相当于mapreduce中的Mapper,而ResultTask所代表的stage3就相当于mapreduce中的reducer。    

# RDD
1. 什么是RDD？  
RDD（Resilient Distributed Dataset）叫做弹性分布式数据集，是Spark中最基本的数据抽象，它代表一个不可变、可分区、里面的元素可并行计算的集合。RDD具有数据流模型的特点：自动容错、位置感知性调度和可伸缩性。RDD允许用户在执行多个查询时显式地将工作集缓存在内存中，后续的查询能够重用工作集，这极大地提升了查询速度。

2. RDD的属性  
-（1）一组分片（Partition），即数据集的基本组成单位。对于RDD来说，每个分片都会被一个计算任务处理，并决定并行计算的粒度。用户可以在创建RDD时指定RDD的分片个数，如果没有指定，那么就会采用默认值。默认值就是程序所分配到的CPU Core的数目。
-（2）一个计算每个分区的函数。Spark中RDD的计算是以分片为单位的，每个RDD都会实现compute函数以达到这个目的。compute函数会对迭代器进行复合，不需要保存每次计算的结果。
-（3）RDD之间的依赖关系。RDD的每次转换都会生成一个新的RDD，所以RDD之间就会形成类似于流水线一样的前后依赖关系。在部分分区数据丢失时，Spark可以通过这个依赖关系重新计算丢失的分区数据，而不是对RDD的所有分区进行重新计算。
-（4）一个Partitioner，即RDD的分片函数。当前Spark中实现了两种类型的分片函数，一个是基于哈希的HashPartitioner，另外一个是基于范围的RangePartitioner。只有对于于key-value的RDD，才会有Partitioner，非key-value的RDD的Parititioner的值是None。Partitioner函数不但决定了RDD本身的分片数量，也决定了parent RDD Shuffle输出时的分片数量。
-（5）一个列表，存储存取每个Partition的优先位置（preferred location）。对于一个HDFS文件来说，这个列表保存的就是每个Partition所在的块的位置。按照“移动数据不如移动计算”的理念，Spark在进行任务调度的时候，会尽可能地将计算任务分配到其所要处理数据块的存储位置。

3. Spark支持两个类型（算子）操作：Transformation和Action
- 3.1　Transformation  
主要做的是就是将一个已有的RDD生成另外一个RDD。Transformation具有lazy特性(延迟加载)。Transformation算子的代码不会真正被执行。只有当我们的程序里面遇到一个action算子的时候，代码才会真正的被执行。这种设计让Spark更加有效率地运行。e.g. map/flatmap/filter等     
- 3.2 Action  
触发代码的运行，我们一段spark代码里面至少需要有一个action操作。e.g. collect/reduce/show等  

map vs flatmap
1. map(func)  
将原数据的每个元素传给函数func进行格式化，返回一个新的分布式数据集。(原文：Return a new distributed dataset formed by passing each element of the source through a function func.)  
2. flatMap(func)  
跟map(func)类似，但是每个输入项和成为0个或多个输出项(所以func函数应该返回的是一个序列化的数据而不是单个数据项)。(原文：Similar to map, but each input item can be mapped to 0 or more output items (so func should return a Seq rather than a single item).)  

理解:  
有二箱鸡蛋，每箱5个，现在要把鸡蛋加工成煎蛋，然后分给学生。  
map做的事情：把二箱鸡蛋分别加工成煎蛋，还是放成原来的[两箱]，分给2组学生；  
flatMap做的事情：把二箱鸡蛋分别加工成煎蛋，然后放到一起[10个煎蛋]，分给10个学生； 
flatMap比map多一个flatten的操作.   

# Spark应用(Application)执行过程中各个组件的概念：
由小到大:  
1. Task(任务)：RDD中的一个分区对应一个task，task是单个分区上最小的处理流程单元。
2. TaskSet(任务集)：一组关联的，但相互之间没有Shuffle依赖关系的Task集合。
3. Stage(调度阶段)：一个taskSet对应的调度阶段，每个job会根据RDD的宽依赖关系被切分很多Stage，每个stage都包含 一个TaskSet。
4. job(作业)：由Action算子触发生成的由一个或者多个stage组成的计算作业。
5. application：用户编写的spark应用程序，由一个或者多个job组成，提交到spark之后，spark为application分派资源，将程序转换并执行。
- DAGScheduler：根据job构建基于stage的DAG，并提交stage给TaskScheduler。
- TaskScheduler:将Taskset提交给Worker Node集群运行并返回结果。

![image](https://user-images.githubusercontent.com/65893273/114414833-c3366180-9be1-11eb-9649-227914cb3154.png)

# Spark standalone模式
![image](https://user-images.githubusercontent.com/65893273/114410749-3047f800-9bde-11eb-9662-b4201c447436.png)  
- Standalone模式使用Spark自带的资源调度框架
- 采用Master/Slaves的典型架构，选用ZooKeeper来实现Master的HA(high availability).只有一个Master是active,其他都是standby(备用)
- 该模式主要的节点有Client节点、Master节点和Worker节点。  
其中Driver既可以运行在Master节点上中，也可以运行在本地Client端。  
当用spark-shell交互式工具提交Spark的Job时，Driver在Master节点上运行；  
当使用spark-submit工具提交Job或者在Eclips、IDEA等开发平台上使用”new SparkConf.setManager(“spark://master:7077”)”方式运行Spark任务时，Driver是运行在本地Client端上的  

# Yarn模式
Yarn学到再补充.  
## Yarn-client
![image](https://user-images.githubusercontent.com/65893273/114423593-e82ed280-9be9-11eb-8183-7268c5534507.png)  
注意 spark context(driver)的位置!client模式在yarn集群外,cluster模式在集群内.  

## Yarn-cluster
![image](https://user-images.githubusercontent.com/65893273/114423634-f381fe00-9be9-11eb-9084-1dda2417d79b.png)  
Cluster模式下，driver运行在ApplicationMaster中，负责向Yarn（ResourceManager）申请资源，并监督Application的运行情况，当Client（这里的Client指的是Master节点）提交作业后，就会关掉Client，作业会继续在yarn上运行，这也是Cluster模式不适合交互类型作业的原因。
而Client模式，AM仅向Yarn（RM）申请executor资源，之后Client会和请求的Container通信来进行任务的调度，即Client不能被关闭。  
cluster模式时要注意各个节点的权限.在client下高权限master节点调试没问题,但是在cluster模式下低权限节点可能无法访问数据.  
# mesos模式
