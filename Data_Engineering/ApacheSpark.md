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

# mesos模式
