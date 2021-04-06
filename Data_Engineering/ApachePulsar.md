# Apache Pulsar的概述，安装和应用

## Pulsar 概述
Pulsar 是一个用于服务器到服务器的消息系统，具有多租户、高性能等优势。 Pulsar 最初由 Yahoo 开发，目前由 Apache 软件基金会管理。  

Pulsar 的关键特性如下：    
&emsp;Pulsar 的单个实例原生支持多个集群，可跨机房在集群间无缝地完成消息复制。  
&emsp;极低的发布延迟和端到端延迟。  
&emsp;可无缝扩展到超过一百万个 topic  
&emsp;简单的客户端 API，支持 Java、Go、Python 和 C++。  
&emsp;支持多种 topic 订阅模式（独占订阅、共享订阅、故障转移订阅）。  
&emsp;通过 Apache BookKeeper 提供的持久化消息存储机制保证消息传递。    
&emsp;&emsp;--由轻量级的 serverless 计算框架 Pulsar Functions 实现流原生的数据处理。  
&emsp;&emsp;--基于 Pulsar Functions 的 serverless connector 框架 Pulsar IO 使得数据更易移入、移出 Apache Pulsar。  
&emsp;&emsp;--分层式存储可在数据陈旧时，将数据从热存储卸载到冷/长期存储（如S3、GCS）中。  

## Setting up Apache pulsar
### A 本地安装Apache Pulsar
1.进入root模式，更新包  
$ sudo -s  
$ apt update; apt -y upgrade;    
2.从官网下载pulsar  
$ wget https://archive.apache.org/dist/pulsar/pulsar-2.7.0/apache-pulsar-2.7.0-bin.tar.gz  
3.解压压缩包，进入文件目录  
$ tar xvfz apache-pulsar-2.7.0-bin.tar.gz    
$ cd apache-pulsar-2.7.0  

### B 在一个单独的机器里开始pulsar（standalone模式）
1. To see the log of pulsar on the terminal and also view logs from other components of the pulsar on a single machine. We will run our commands in the terminal multiplexer.  
在终端的复用器里查看日志，指令如下。  
$ tmux new -s pulsar    
2. Start a local cluster using the following pulsar command by navigating to bin directory, and specifying standalone mode. 导航进入bin目录，启动一个本地集群，模式为standalone  
$ bin/pulsar standalone
3. 如果pulsar成功安装，则没有error。官方指导上只有三条info，在我实际操作中是不一致的。是否正常运行，通过检查consumer是否有接收message来确定。      





  

