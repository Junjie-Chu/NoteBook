# Hadoop相关题目，知识点等

## Hadoop的守护进程
在Hadoop中，主要的后台守护进程包括：  
NameNode元数据服务器    
 --主节点，存储文件的元数据（文件名，文件目录结构，文件属性——生成时间，副本数，文件权限），以及每个文件的块列表和块所在的DataNode等  
SecondaryNameNode辅助元数据服务器      
 --用来监控HDFS状态的辅助后台程序，每隔一段时间获取HDFS元数据快照    
DataNodes块存储      
 --在本地文件系统存储文件块数据，以及块数据校验    
JobTracker任务调度    
 --负责接收用户提交的作业，负责启动、跟踪任务执行    
TaskTrackers任务执行    
 --负责执行由JobTracker分配的任务，管理各个任务在每个节点的执行情况    

## Hadoop的三种模式
-单机模式    
-伪分布式模式 Pseudo /soodow/ distribution     
-完全分布式模式    

单机模式框架  
-默认模式。分布式  
-不对配置文件进行修改。  
-使用本地文件系统，而不是分布式文件系统。  
-Hadoop不会启动NameNode、DataNode、JobTracker、TaskTracker等守护进程，Map()和Reduce()任务做为同一个进程的两个部分来执行的。    
-用于对MapReduce程序的逻辑进行调试，确保程序的正确。    

伪分布式模式  
-在一台主机模拟多主机。  
-Hadoop启动NameNode、DataNode、JobTracker、TaskTracker这些守护进程都在同一台机器上运行，是相互独立的Java进程。    
-在这种模式下，Hadoop使用的是分布式文件系统，各个作业也是由JobTraker服务，来管理的独立进程。在单机模式之上增长了代码调试功能，容许检查内存使用状况，HDFS输入输出，以及其余的守护进程交互。    
相似于彻底分布式模式，所以，这种模式经常使用来开发测试Hadoop程序的执行是否正确。（彻底分布式的一种特例）    
-修改3个配置文件：core-site.xml（Hadoop集群的特性，做用于所有进程及客户端）、hdfs-site.xml（配置HDFS集群的工做属性）、mapred-site.xml（配置MapReduce集群的属性）    
-格式化文件系统    

彻底分布式模式  
-Hadoop的守护进程运行在由多台主机搭建的集群上，是真正的生产环境。    
-在全部的主机上安装JDK和Hadoop，组成相互连通的网络。    
-在主机间设置SSH免密码登陆，把各从节点生成的公钥添加到主节点的信任列表。    
-修改3个配置文件：core-site.xml、hdfs-site.xml、mapred-site.xml，指定NameNode和JobTraker的位置和端口，设置文件的副本等参数    
-格式化文件系统    
