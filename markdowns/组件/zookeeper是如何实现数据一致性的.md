众所周知，zookeeper是一个开源的分布式协调服务，很多分布式的应用都是基于zookeeper来实现分布式锁，服务管理，服务发现，通知订阅等功能。那么。zookeeper自身是如何在分布式环境下实现数据的一致性的呢。

# 结构
既然zookeeper是在分布式环境下提供服务的，那么它必须要解决的问题就是单点问题，因此zookeeper是一个主备的结构。zookeeper存在leader,follower,observer三种角色,这三种角色在实际服务集群中都是服务节点。

1. leader:处理所有请求，为客户的提供读和写服务
2. follower:只提供读服务，有机会通过选举成为leader
3. observer:只提供读服务 

由以上三种角色的介绍可知，zookeeper中所有请求都是交给leader处理的，因此，如果leader挂了，zookeeper就无法再提供服务，这就是单点问题。所幸在leader挂了之后，follower能够通过选举成为新的leader。 

那么问题来了，follower是如何被选举成为新的leader的？新的leader又是如何保证数据的一致性的？这些问题的答案都在于zookeeper所用的分布式一致性协议ZAB。

# ZAB协议
ZAB协议是为zookeeper专门设计的一种支持奔溃恢复的原子广播协议。虽然它不像Paxos算法那样通用通用，但是它却比Paxos算法易于理解。在我看来ZAB协议主要的作用在于三个方面
1. 选举出leader;
2. 同步节点之间的状态达到数据一致
3. 数据的广播。

## 几个概念的定义
在了解ZAB协议具体过程之前不要先了解几个概念。
### 事务
zookeeper作为一个分布式协调服务，需要leader节点去接受外部请求，转化为内部操作（比如创建，修改，删除节点），需要原子性执行的几个操作就组成了事务，这里用T代表。zookeeper要求有序的处理事务，因此给每个事务一个编号，每进来一个事务编号加1，假设leader节点中进来n个事务，可以表示为T1,T2,T3…Tn。为了防止单点问题导致事务数据丢失，leader节点会把事务数据同步到follower节点中。

### 事务队列
leader和follower都会保存一个事务队列，用L表示，L=T1,T2,T3…Tn，leader的事务队列是接受请求保存下来的，follower的事务队列是leader同步过来的，因此leader的事务队列是最新，最全的。

### 任期
在zookeeper的工作过程中，leader节点崩溃，重新选举出新的leader是很正常的事情，所以zookeeper的运行历史中会产生多个leader，就好比一个国家的历史中会相继出现多为领导人。为了区分各个leader，ZAB协议用一个整数来表示任期，我们假设用E表示任务。zookeeper刚运行时选举出的第一个leader的任期为E=1；第一个leader崩溃后，下面选举出来的leader，任期会加1，E=2;一次类推。加入任期概念的事务应该表示为T(E,n)

## 协议过程
### 选举leader
1. 每个follower广播自己事务队列中最大事务编号maxId
2. 获取集群中其他follower发出来的maxId，选取出最大的maxId所属的follower，33. 投票给改follower，选它为leader。
3. 统计所有投票，获取投票数超过一半的follower被推选为leader

### 同步数据
1. 各个follower向leader发送自己保存的任期E
2. leader,比较所有的任期，选取最大的E，加1后作为当前的任期E=E+1
3. 将任务E广播给所有follower
4. follower将任期改为leader发过来的值，并且返回给leader事务队列L
5. leader从队列集合中选取任期最大的队列，如果有多个队列任期都是最大，则选取事务编号n最大的队列Lmax。将Lmax最为leader队列，并且广播给各个follower。
6. follower接收队列替换自己的事务队列，并且执行提交队列中的事务。 
  至此各个节点的数据达成一致，zookeeper恢复正常服务。

### 广播
1. leader节点接收到请求，将事务加入事务队列，并且将事务广播给各个follower。
2. follower接收事务并加入都事务队列，然后给leader发送准备提交请求。
3. leader 接收到半数以上的准备提交请求后，提交事务同时向follower 发送提交事务请求
4. follower提交事务。
---------------------
作者：炜爷 
来源：CSDN 
原文：https://blog.csdn.net/qqqq0199181/article/details/80865828 
版权声明：本文为博主原创文章，转载请附上博文链接！