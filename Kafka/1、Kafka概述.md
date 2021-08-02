 

# 1、概述：

**学习kafka前，先了解下消息队列的两种形式：**

**（1）点对点模式**（一对一，消费者主动拉取数据，消息收到后消息清除）

  消息生产者生产消息发送到Queue中，然后消息消费者从Queue中取出并且消费消息。消息被消费以后，queue中不再有存储，所以消息消费者不可能消费到已经被消费的消息。Queue支持存在多个消费者，但是对一个消息而言，只会有一个消费者可以消费。该系统的典型应用就是订单处理系统，其中每个订单将有一个订单处理器处理，但多个订单处理器可以同时工作。

![img](https://img-blog.csdnimg.cn/20210527165109215.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

**（2）发布/订阅模式**（一对多，消费者消费数据之后不会清除消息）

消息生产者（发布）将消息发布到topic中，同时有多个消息消费者（订阅）消费该消息。和点对点方式不同，发布到topic的消息会被所有订阅者消费。

![img](https://img-blog.csdnimg.cn/20210527165145969.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xteWp5MTk5Ng==,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

# **2、Kafka基础架构：**

kafka支持消息持久化，消费端为拉模型来拉取数据，消费状态和订阅关系有客户端负责维护，消息消费完 后，不会立即删除，会保留历史消息。因此支持多订阅时，消息只会存储一份就可以了。

![img](https://img-blog.csdnimg.cn/20210527171002370.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xteWp5MTk5Ng==,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

一个典型的kafka集群中包含若干个Producer，若干个Broker，若干个Consumer，以及一个zookeeper集群； kafka通过zookeeper管理集群配置，选举leader，以及在用户组（Consumer Group）发生变化时进行Rebalance（负载均衡）；Producer使用push模式将消息发布到Broker；Consumer使用pull模式从Broker中订阅并消费消息。

![img](https://img-blog.csdnimg.cn/2021052717114627.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xteWp5MTk5Ng==,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

# 3、kafka术语：

***\*Broker\****：kafka集群中包含一个或者多个服务实例，这种服务实例被称为Broker,Broker就是一台kafka服务器

***\*Topic\****：每条发布到kafka集群的消息都有一个类别，这个类别就叫做Topic，也就是主题，比如：订单主题，日志主题，每个不同主题存放不同主题的数据

***\*Partition\****：Partition是一个物理上的概念，每个Topic包含一个或者多个Partition，就是分区，一个topic的数据分区存放，如果一个topic有300GB数据三个分区，则每个分区就存放100GB数据

***\*Producer\****：负责发布消息到kafka的Broker中生产者

***\*Consumer\****：消息消费者,向kafka的broker中读取消息的客户端

***\*Consumer\**** ***\*Group\****：每一个Consumer属于一个特定的Consumer Group（可以为每个Consumer指定 groupName）

## 1、Kafka术语拓展说明：

### topic

1.kafka 将消息以topic为单位进行归类，比如：订单主题，日志主题，每个不同主题存放不同主题的数据。

2.topic 特指kafka处理的消息源的不同分类。

3.topic是一种分类或者发布的一些列记录的名义上的名字，kafka主题始终是支持多用户订阅的，也就是说一个主题可以有零个，一个或者多个消费者订阅写入的数据。

4.kafka集群中可以有多个主题(topic)。

5.生产者和消费者消费数据一般以主题为单位消费数据，如果分更细粒度的话就是分区级别。



### Partitions (分区数)

分区数就是控制一个topic切分成多少个log,可以显示去指定，如果不显示指定则就会默认使用配置文件中配置num.partitions的配置数量，配置文件在安装目录中config目录中的server.properties文件中。

一个broker服务下是可以创建多个分区，每一个分区都会有一个编号，编号是从0开始，分区中的数据是有序的，就是和生产数据进来的数据的数据顺序一致

问题：如何保证一个主题(topic)下的数据是有序的(也就是生产数据是什么样的顺序，消费的时候数据也是什么样的数据)？

答：则可以设置一个主题(topic)下只有一个分区(partition)，这样就保证数据是有序的，因为同一个分区中的数据就是有序的。

- topic的Partition数量在创建topic时配置
- Partition数量决定了每个Consumer group中并发消费者的最大数量

```
为什么说Partition数量决定了每个Consumer group中并发消费者的最大数量？

比如一个topic中只有3个partition，而消费用户组有4个消费者，则只有三个消费者可以消费数据，也就是只有三个并发消费数据，有一个消费者是空闲的。

因为同一个topic中的同一个partition同一时间只能被一个消费者消费，比如说订单topic中的partition1已经被消费，这时其他消费者就不能再消费订单topic中的partition1了。

所以说Partition数量决定了每个Consumer group中并发消费者的最大数量。

问题：如果生产者生产数据比较快，消费者消费数据比较慢，应该怎么改进？

答：就是利用Partition数量决定了每个Consumer group中并发消费者的最大数量这个原理，可以创建多个partition去提高消费者消费数据的并发数量从而提高消费者消费数据的速度。
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)



### Partition Replication(副本数)

![img](https://img-blog.csdnimg.cn/20210528230246766.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xteWp5MTk5Ng==,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

副本数（replication-factor）:就是分区（partition）做备份的数量，控制保存在几个不同的broker上，一般情况副本数等于broker的个数，也就是等一Kafka服务器个数

副本因子是包含本身，同一个副本因子不能放在同一个Broker中。

问题：只有一个broker服务器，是否可以创建多个副本因子？
 答：不可以！创建主题时，副本因子应该小于等于可用的broker数，副本就是保证数据可靠性，备份作用，所以不能备份在同一台服务器上，那样做没有什么意义，

如果都在一台服务器上，只要这台服务器挂掉就会数据全部丢失，假设现在有三台Kafka服务器（broker），副本数最多就是3个。就算其中一台服务器挂掉，

但是还有其他两台服务器备份数据，这也是副本数保证了Kafka数据的可靠性的。

如果某一个分区有三个副本因子，就算其中一个挂掉，那么只会剩下的两个中去选择一个leader，但不会再去其他的broker中，另启动一个副本（因为在另一台启动的话，存在数据传递，只要在机器之间有数据传递，就会长时间占用网络IO，kafka是一个高吞吐量的消息系统，这个情况不允许发生）所以不会在零个broker中启动。

如果所有的副本都挂了，生产者如果生产数据到指定分区的话，将写入不成功。



![img](https://img-blog.csdnimg.cn/20210528232214982.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xteWp5MTk5Ng==,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

副本因子操作以分区（partition）为单位的。每个分区（partition）都有各自的主副本和从副本；主副本叫做leader，从副本叫做 follower

（在有多个副本的情况下，kafka会为同一个分区下的分区，设定角色关系：一个leader和N个 follower），

消费者和生产者都是从leader读写数据，不与follower交互，所以从副本需要向主副本去拉去数据，处于同步状态的副本叫做***\*in-sync-replicas\****(ISR);follower通过拉的方式从leader同步数据。

lsr表示：当前可用的副本



### Partition offset

![img](https://img-blog.csdnimg.cn/20210528235042345.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xteWp5MTk5Ng==,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

任何发布到此partition的消息都会被直接追加到log文件的尾部，每条消息在文件中的位置称为oﬀset（偏移量），oﬀset是一个long类型数字，它唯一标识了一条消息，消费者通过（oﬀset，partition，topic）跟踪记录。如上图中partition0每进去一条数据就会打上数字作为标记，如：0号数据，1号数据......

```
Partition offset作用就是帮助消费者在消费数据的时候根据标记去消费数据，消费过的数据不会重复去消费，没有消费过的数据也不会丢失没有消费。
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## 2、kafka分区和消费组之间的关系

消费组： 由一个或者多个消费者组成，同一个组中的消费者对于同一条消息只消费一次。某一个主题下的分区数，对于消费组来说，应该小于等于该主题下的分区数。如下所示：

```
如：某一个主题有4个分区，那么消费组中的消费者应该小于4，而且最好与分区数成整数倍 1 2 4
同一个分区下的数据，在同一时刻，不能同一个消费组的不同消费者去消费
比如说：partition_1中数据aaa被消费组1中消费者1已经在消费，则消费组1中其他消费者不可以消费。
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

总结：分区数越多，同一时间内就可以有越多的消费者来进行消费，消费数据的速度就会越快，从而提高了消费的性能。
