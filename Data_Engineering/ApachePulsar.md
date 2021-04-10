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

## Pulsar里subscription类型的理解
Pulsar订阅有四种类型:  
1.独占 Exclusive  
2.灾备 failover  
3.共享 shared  
4.键共享 key-shared  

example:  
1.现在有1个topic,多个consumer以独占模式订阅,且订阅名字相同(subscription_name),报错(error!)  
2.现在有1个topic,多个consumer以独占模式订阅,且订阅名字不同(subscription_name),所有consumer都会接收到topic中所有的message  
3.现在有1个topic,多个consumer以shared模式订阅,订阅名字相同(subscription_name),每个consumer只会接收到topic中的部分message(分配给它的message)    
4.现在有1个topic,多个consumer以灾备模式订阅,且订阅名字相同(subscription_name),运行时,第一个consumer先接收topic中的所有数据,如果不坏,其他consumer不接收message.如果第一个consumer坏了,第二个consumer接替它的位置.    



  

