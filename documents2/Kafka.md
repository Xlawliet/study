# kafka

Kafka** 是一个分布式的基于发布/订阅模式的消息队列，主要应用于大数据实时处理领域。



RockteMQ的社区活跃度不高，但是是阿里出品。



### 如何保证幂等性

结合业务思考：

如果获取到的数据需要写库，先根据主键查一下，如果数据已存在，就不插入，update

如果获取到的数据写Redis，直接set就好了，redis天然有幂等性

其他的场景，可以在生产发送每条数据的时候，在里面加入一个全局id，如果没消费过，消费之后就在redis里写一下id。

结合具体业务来看。



### 如何处理消息丢失，保证消息的可靠性传输

如果某个broker宕机，就重新选举partition的leader，如果此时其他的follower刚好还有数据没有同步，结果leader就挂了，就会造成数据的丢失。

一般需要设置4个参数：

- 给topic设置relication.factor参数：这个值必须大于1，要求每个partition必须有至少2个副本
- 给kafka服务端设置 min.insync.replicas 参数：这个值必须大于1，要求一个leader至少感知到至少一个follower还跟自己联系没掉队，才能确保当leader挂了的时候还有一个follower
- 在producer端设置acks=all：要求每条数据，必须是写入所有replica之后，才能认为是写成功了。
- 在producer端设置retries=MAX（很大很大的值，无限次重试的意思）：这个是要求一旦写入失败，就无限重试。



### 如何保证消息的顺序性

当用单线程来消费数据的时候，不会出现消息顺序乱掉的情况，但如果使用的是多线程来处理，就可能使得消息的顺序乱掉。

解决方法，将具有相同key的数据存到同一个内存queue中，多线程时依次从中获取数据，就能保证顺序性。



### 如何解决消息队列的延时以及过期失效问题？消息队列满了以后该怎么处理？有几百万消息持续积压几小时，说说怎么解决？

主要是消费端出了问题，不消费了，或者消费的速度极其慢。

比如每次消费之后要写mysql，但是mysql挂掉了，消费端hang那儿不动了。

解决方法：

- 先修复consumer的问题，确保其恢复消费速度，然后将现有的consumer都停掉。
- 新建一个topic，partition是原来的10倍，临时建立好原先10倍的queue数量。
- 写一个临时的分发数据的consumer程序，这个程序部署上去消费积压的数据，消费之后不做耗时的处理，直接均匀轮询写入临时建立好的10倍数量的queue。
- 接着临时征用10倍的机器来部署consumer，每一批consumer消费一个临时queue的数据。这种做法相当于是临时将queue资源和consumer资源扩大10倍，以正常的10倍速度来消费数据。
- 等快速消费完积压数据之后，得恢复原先部署等架构，重新用原先的consumer机器来消费信息。



### 消息队列的特点

**解耦**

允许独立的扩展或修改两边的处理过程，只要确保它们遵守同样的接口约束。

**可恢复性**

系统的一部分组件失效时，不会影响到整个系统。消息队列降低了进程间的耦合度，所以即使一个处理消息的进程挂掉，加入队列中的消息仍然可以在系统恢复后被处理。

**缓冲**

有助于控制和优化数据流经过系统的速度，解决生产消息和消费消息的处理速度不一致的情况。

**灵活性&峰值处理能力**

在访问量剧增的情况下，应用仍然需要继续发挥作用，但是只要的突发流量并不常见，使用消息队列能够使得关键组件顶住突发的访问压力，而不会因为突发的超负荷的请求而完全崩溃。可以动态的增减机器。

**异步通信**

允许用户将消息放入队列中，但并不立即处理。在需要的时候才去处理。



#### 两种模式

##### （1）点对点模式（一对一，消费者主动拉取数据，消息收到后消息清除）

消息生产者生产消息发送到queue中，然后消息消费者从queue中取出并消费消息。消息被消费后，就被queue清除。所以消费者不可能消费到已经被消费过的信息。queue支持存在多个消费者，但是对一个消息而言，只会有一个消费者可以消费。

##### （2）发布/订阅模式（一对多，消费者消费消息之后不会立即清除消息）

- 消息生产者（发布）将消息发布到topic（队列）中，同时有多个消息消费者（订阅）消费该消息。和点对点方式不同，发布到topic的消息可以被所有订阅者消费。缺点：生产者生产消息的速率与消费者消费的速率不同，造成资源浪费或者因为消费者处理能力不足导致系统崩溃。
- 消费者主动到队列中拉取，不再由消息队列主动推送。缺点：当队列中长时间没有消息时，消费者仍然需要定时向队列中去询问。



## Kafka架构

<img src="D:\笔记\个人笔记\Kafka.assets\image-20201221191657591.png" alt="image-20201221191657591" style="zoom:67%;" />



### kafka安装部署

```shell
官网下
解压
改配置文件
只需要改一个能否删除，将注释删除，使之为true即可
先启动zookeeper


#更改为true,否则删除都是逻辑删除
delete.topic.enable=true   

kafka启动命令
bin/kafka-server-start.sh -daemon config/server.properties   ##-daemon命令是使得kafka以守护进程的形式存在
bin/kafka-server-stop.sh config/server.properties
```



kk.sh群起脚本

```shell
#!/bin/bash

case $1 in
"start"){
		
		for i in vmware001 vmware002 vmware003
		do
			echo "************$i************"
			ssh $i "/usr/local/kafka/bin/kafka-server-start.sh -daemon /usr/local/kafka/config/server.properties"
		done
};;

"stop"){
		
		for i in vmware001 vmware002 vmware003
		do
			echo "************$i************"
			ssh $i "/usr/local/kafka/bin/kafka-server-start.sh"
		done
};;

esac
```

```shell
将其写到/bin目录下
vi kk.sh # 后将上述脚本复制进去
chmod 777 kk.sh
```



### 命令行操作

1)查看当前服务器中所有topic

```shell
./kafka-topics.sh --zookeeper localhost:2181 --list
bin/kafka-topics.sh --list --zookeeper vmware100:2181
bin/kafka-topics.sh --list --zookeeper localhost:2181
```

2)创建topic

```shell
./kafka-topics.sh --zookeeper localhost:2181 --create --replication-facotr 3 --partitions 1 --topic first
bin/kafka-topics.sh --create --zookeeper localhost:2181 --topic first --parititions 2 --replication-factor 1

bin/kafka-topics.sh --create -zookeeper localhost:2181 -replication-factor 1 -partitions 2 -topic first
```

选项说明：

--topic 定义topic名 

--replication-factor 定义副本数

--partitions 定义分区数



3)删除topic

```shell
./kafka-topics.sh -zookeeper localhost:2181 --delete -topic first
```

需要将server.properties中设置delete.topic.enable=true 否则只是标记删除

4)发送消息

```shell
./kafka-console-producer.sh --broker-list localhost:9092 --delete --topic first
bin/kafka-console-producer.sh -topic first -broker-list localhost:9092
>hello world
>atguigu atguigu
```

5)消费消息

```shell
./kafka-console-consumer.sh --broker-list localhost:2181 --delete --topic first

bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic first

#从头消费
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --from-beginning --topic first
```

--from-beginning:会把主题中以往所有的数据都读取出来

6)查看某个topic的详情

```shell
./kafka-topics.sh --describe --topic first --zookeeper localhost:2181 
./kafka-topics.sh --describe -topic first --zookeeper localhost:2181             
```

7)修改分区数

```shell
./kafka-topics.sh --zookeeper localhost:2181 --alter --topic first --partitions 6
```



### 分区策略

#### 分区原因

1. 方便在集群中拓展，每个partition可以通过调整以适应它所在的机器，而每一个topic又可以有多个partition组成。
2. 可以提高并发，因为可以以Partition为单位读写了。



#### 分区原则

1. 指明Partition的情况下，直接将指明的值作为parition值
2. 没有指明parition的情况下，但有key，将key的hash值与topic的partition数进行取余后得到partition值。
3. 既没有partition也没有key的情况下，第一次调用时随机生成一个整数（后面每次调用在这个值上进行自增），将这个值与topic可用的partition总数取余后得到partition值。



### 数据可靠性保证

为保证producer发送的数据，能可靠的发送到指定的topic，topic的每个partition收到producer发送的数据后，都需要向producer发送ack，如果producer收到ack，才会进行下一轮数据的发送，否则重新发送数据。

#### acks参数配置：

0：producer不等待broker的ack，这一操作提供了一个最低的延迟，broker一接受到还没有写入磁盘就已经返回，当broker故障时有可能丢失数据；

1：producer等待broker的ack，partition的leader落盘成功后返回ack，如果在follower同步成功之前leader故障，那么将会丢失数据；

-1（all）：producer等待broker的ack，partition的leader和follower全部落盘成功后才返回ack。但是如果在follower同步完成后，broker发送ack之前，leader发生故障，那么会造成数据重复；



### Kafka消费者

#### 消费方式

consumer采用pull模式从broker中读取消息。

==push模式==很难适应消费速率不同的消费者，因为消息发送速率是由broker决定的。

==pull模式==不足之处是，如果kafka没有数据，消费者可能会陷入循环中，一直返回空数据。针对这一问题，kafka在消费者消费数据时会传入一个时长参数timeout，如果当前没有数据可供消费，consumer会等待一段时间之后再返回，这段时长即为timeout。

#### 分区分配策略

一个consumer group中有多个consume，一个topic有多个parition，所以必然会涉及到partition的分配问题，即确定那个partition由哪个consumer来消费

两种策略：一时RoundRobin，一是Range





### 消息发送流程

Kafka的Producer发送消息采用的是异步发送的方式。在消息发送的过程中，涉及到了两个线程——main线程和Sender线程，以及一个线程共享变量——RecordAccumulator。main线程将消息发送给RecordAccumulator，Sender线程不断从RecordAccumulator中拉去消息发送到broker中。