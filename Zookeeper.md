 # Zookeeper

## Zookeeper快速领导者选举原理

## 人类选举的基本原理

正常情况下，选举是一定要投票的。

我们应该都经历过投票，在投票时我们可能会将票投给和我们关系比较好的人，如果你和几个候选人都比较熟，这种情况下你会将选票投给你认为能力比较强的人，如果你和几个候选人都不熟，并且你自己也是候选人的话，这时你应该会认为你是这些候选人里面最厉害的那个人，大家都应该选你，这时你就会去和别人交流以获得别人的投票，但是很有可能在交流的过程中，你发现了比你更厉害的人，这时你如果脸皮不是那么厚的话，你应该会改变你的决定，去投你觉得更厉害的人，最终你将得到在你心中认为最厉害的人，且将票投给他，选票将会放在投票中，最后从投票箱中进行统计，获得票数最多的人当选。

在这样一个选举过程中我们提炼出四个基本概念：

1. 个人能力：投我认为能力最强的人，这是投票的基本规则
2. 改票：能力最强的人是逐渐和其他人沟通之后的结果，类似改票，先投给A，但是后来发现B更厉害，则改为投B
3. 投票箱：所有人公用一个投票箱
4. 领导者：获得投票数最多的人为领导者

## Zookeeper选举的基本原理

Zookeeper集群模式下才需要选举。



Zookeeper的选举和人类的选举逻辑类似，Zookeeper需要实现上面人类选举的四个基本概念；

1. 个人能力：Zookeeper是一个数据库，集群中节点的数据越新就代表此节点能力越强，而在Zookeeper中可以通事务id(zxid)来表示数据的新旧，一个节点最新的zxid越大则该节点的数据越新。所以Zookeeper选举时会根据zxid的大小来作为投票的基本规则。
2. 改票：Zookeeper集群中的某一个节点在开始进行选举时，首先认为自己的数据是最新的，会先投自己一票，并且把这张选票发送给其他服务器，这张选票里包含了两个重要信息：**zxid**和**sid**，sid表示这张选票投的服务器id，zxid表示这张选票投的服务器上最大的事务id，同时也会接收到其他服务器的选票，接收到其他服务器的选票后，可以根据选票信息中的zxid来与自己当前所投的服务器上的最大zxid来进行比较，如果其他服务器的选票中的zxid较大，则表示自己当前所投的机器数据没有接收到的选票所投的服务器上的数据新，所以本节点需要改票，改成投给和刚刚接收到的选票一样。
3. 投票箱：Zookeeper集群中会有很多节点，和人类选举不一样，Zookeeper集群并不会单独去维护一个投票箱应用，而是在每个节点内存里利用一个数组来作为投票箱。每个节点里都有一个投票箱，节点会将自己的选票以及从其他服务器接收到的选票放在这个投票箱中。因为集群节点是相互交互的，并且选票的PK规则是一致的，所以每个节点里的这个投票箱所存储的选票都会是一样的，这样也可以达到公用一个投票箱的目的。
4. 领导者：Zookeeper集群中的每个节点，开始进行领导选举后，会不断的接收其他节点的选票，然后进行选票PK，将自己的选票修改为投给数据最新的节点，这样就保证了，每个节点自己的选票代表的都是自己暂时所认为的数据最新的节点，再因为其他服务器的选票都会存储在投票箱内，所以可以根据投票箱里去统计是否有超过一半的选票和自己选择的是同一个节点，都认为这个节点的数据最新，一旦整个集群里超过一半的节点都认为某一个节点上的数据最新，则该节点就是领导者。



通过对四个概念的在Zookeeper中的解析，也同时介绍了一下Zookeeper领导者选举的基本原理，只是说选举过程中还有更多的细节需要我们了解，下面我结合源码来给大家详细的分析一下Zookeeper的快速领导者选举原理。



## 领导者选举入口

ZooKeeperServer表示单机模式中的一个zkServer。

QuoruPeer表示集群模式中的一个zkServer。

QuoruPeer类定义如下：



```java
public class QuorumPeer extends ZooKeeperThread implements QuorumStats.Provider
```



定义表明QuorumPeer是一个ZooKeeperThread，表示是一个线程。

当集群中的某一个台zkServer启动时QuorumPeer类的start方法将被调用。



```java
public synchronized void start() {
        loadDataBase(); // 1
        cnxnFactory.start(); // 2
        startLeaderElection(); // 3 
        super.start();  // 4
    }
```



1. zkServer中有一个内存数据库对象ZKDatabase， zkServer在启动时需要将已被持久化的数据加载进内存中，也就是加载至ZKDatabase。
2. 这一步会开启一个线程来接收客户端请求，但是需要注意，这一步执行完后虽然成功开启了一个线程，并且也可以接收客户端线程，但是因为现在zkServer还没有经过初始化，实际上把请求拒绝掉，知道zkServer初始化完成才能正常的接收请求。
3. 这个方法名很有误导性，这个方法并没有真正的开始领导选举，而是进行一些初始化
4. 继续启动，包括进行领导者选举、zkServer初始化。



## 领导者选举策略

上文QuorumPeer类的startLeaderElection会进行领导者选举初始化。

首先，领导者选举在Zookeeper中有3种实现：

![image.png](https://cdn.nlark.com/yuque/0/2019/png/365147/1560665899887-31324eb7-bcec-46a1-8678-4e044af03f9a.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_6bKB54-t5a2m6Zmi5ZGo55Gc%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10%2Fresize%2Cw_1500)

其中LeaderElection、AuthFastLeaderElection已经被标为过期，不建议使用，所以现在用的都是**快速领导者选举****FastLeaderElection，**我们着重来介绍FastLeaderElection。





## 快速领导者选举



快速领导者选举实现架构如下图：

![image.png](https://cdn.nlark.com/yuque/0/2019/png/365147/1560666915378-6f10ffe0-3792-4bc1-afad-12323a3fbd7c.png)



### 传输层初始化

从架构图我们可以发现，快速领导者选举实现架构分为两层：**应用层**和**传输层**。所以初始化核心就是初始化传输层。



初始化步骤：

1. 初始化QuorumCnxManager
2. 初始化QuorumCnxManager.Listener
3. 运行QuorumCnxManager.Listener
4. 运行QuorumCnxManager
5. 返回FastLeaderElection对象



#### QuorumCnxManager介绍

QuorumCnxManager就是传输层实现，QuorumCnxManager中几个重要的属性：

- ConcurrentHashMap<Long, ArrayBlockingQueue<ByteBuffer>> **queueSendMap**
- ConcurrentHashMap<Long, SendWorker> **senderWorkerMap**
- ArrayBlockingQueue<Message> **recvQueue**
- QuorumCnxManager.Listener



传输层的每个zkServer需要发送选票信息给其他服务器，这些选票信息来至应用层，在传输层中将会按服务器id分组保存在**queueSendMap**中。



传输层的每个zkServer需要发送选票信息给其他服务器，SendWorker就是封装了Socket的发送器，而**senderWorkerMap**就是用来记录其他服务器id以及对应的SendWorker的。



传输层的每个zkServer将接收其他服务器发送的选票信息，这些选票会保存在**recvQueue**中，以提供给应用层使用。



QuorumCnxManager.Listener负责开启socket监听。



细化后的架构图如下：

![image.png](https://cdn.nlark.com/yuque/0/2019/png/365147/1560669001378-f9186f9f-c211-482d-aca0-e64f3f550207.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_6bKB54-t5a2m6Zmi5ZGo55Gc%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10%2Fresize%2Cw_1500)



#### 服务器之间连接问题

在集群启动时，一台服务器需要去连另外一台服务器，从而建立Socket用来进行选票传输。那么如果现在A服务器去连B服务器，同时B服务器也去连A服务器，那么就会导致建立了两条Socket，我们知道Socket是双向的，Socket的双方是可以相互发送和接收数据的，那么现在A、B两台服务器建立两条Socket是没有意义的，所以ZooKeeper在实现时做了限制，只允许**服务器ID较大者去连服务器ID较小者，小ID服务器去连大ID服务器会被拒绝**，伪代码如下**：**



```
if (对方服务器id < 本服务器id) {
    closeSocket(sock); // 关闭这条socket
    connectOne(sid);   // 由本服务器去连对方服务器
} else {
    // 继续建立连接
}
```



#### SendWorker、RecvWorker介绍



上文介绍到了SendWorker，它是zkServer用来向其他服务器发送选票信息的。

类结构如下：

```
class SendWorker extends ZooKeeperThread {
    Long sid;
    Socket sock;
    RecvWorker recvWorker;
    volatile boolean running = true;
    DataOutputStream dout;
}
```



它封装了socket并且是一个线程，实际上SendWorker的底层实现是：SendWorker线程会不停的从**queueSendMap**中获取选票信息然后发送到Socket上。

基于同样的思路，我们还需要一个线程从Socket上获取数据然后添加到**recvQueue**中，这就是**RecvWorker**的功能。



所以架构可以演化为下图，通过这个架构，选举应用层直接从recvQueue中获取选票，或者选票添加到queueSendMap中既可以完成选票发送：

![image.png](https://cdn.nlark.com/yuque/0/2019/png/365147/1560671013696-ec9a62d5-dcc4-469b-8a43-280215576cb2.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_6bKB54-t5a2m6Zmi5ZGo55Gc%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)



### 应用层初始化

#### FastLeaderElection类介绍

FastLeaderElection类是快速领导者选举实现的核心类，这个类有三个重要的属性：

- LinkedBlockingQueue<**ToSend**> **sendqueue**;
- LinkedBlockingQueue<**Notification**> **recvqueue**;
- Messenger **messenger**;

- - Messenger.WorkerSender
  - Messenger.WorkerReceiver



服务器在进行领导者选举时，在发送选票时也会同时接受其他服务器的选票，FastLeaderElection类也提供了和传输层类似的实现，将待发送的选票放在**sendqueue**中，由**Messenger.****WorkerSender**发送到传输层**queueSendMap**中。

同样，由**Messenger.****WorkerReceiver**负责从传输层获取数据并放入**recvqueue**中。



这样在应用层了，只需要将待发送的选票信息添加到**sendqueue**中即可完成选票信息发送，或者从**recvqueue**中获取元素即可得到选票信息。



在构造FastLeaderElection对象时，会对**sendqueue、****recvqueue**队列进行初始化，并且运行**Messenger.****WorkerSender**与**Messenger.****WorkerReceiver**线程。



此时架构图如下：



![image.png](https://cdn.nlark.com/yuque/0/2019/png/365147/1560673545253-d663a7b0-3459-4123-8b20-285e300290e9.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_6bKB54-t5a2m6Zmi5ZGo55Gc%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)



到这里，QuorumPeer类的startLeaderElection方法已经执行完成，完成了传输层和应用层的初始化。



### 快速领导者选举实现



QuorumPeer类的start方法前三步分析完，接下来我们来看看第四步：



```
super.start();
```



QuorumPeer类是一个ZooKeeperThread线程，上述代码实际就是运行一个线程，相当于运行QuorumPeer类中的run方法，这个方法也是集群模式下Zkserver启动最核心的方法。



总结一下QuorumPeer类的start方法：

1. 加载持久化数据到内存
2. 初始化领导者选举策略
3. 初始化快速领导者选举传输层
4. 初始化快速领导者选举应用层
5. 开启主线程



主线程开启之后，QuorumPeer类的start方法即执行完成，这时回到上层代码可以看到主线程会被join住：

```
quorumPeer.start(); // 开启线程
quorumPeer.join(); // join线程
```





接下来我们着重来分析一下主线程内的逻辑。



#### 主线程

在主线程里，会有一个主循环(Main loop)，主循环伪代码如下：



```
while (服务是否正在运行) {
    switch (当前服务器状态) {
        case LOOKING:
            // 领导者选举
            setCurrentVote(makeLEStrategy().lookForLeader());
            break;
        case OBSERVING:
            try {
                // 初始化为观察者
            } catch (Exception e) {
                LOG.warn("Unexpected exception",e );                        
            } finally {
                observer.shutdown();
                setPeerState(ServerState.LOOKING);
            }
            break;
        case FOLLOWING:
            try {
                // 初始化为跟随者
            } catch (Exception e) {
                LOG.warn("Unexpected exception",e);
            } finally {
                follower.shutdown();
                setPeerState(ServerState.LOOKING);
            }
            break;
        case LEADING:
            try {
                // 初始化为领导者
            } catch (Exception e) {
                LOG.warn("Unexpected exception",e);
            } finally {
                leader.shutdown("Forcing shutdown");
                setPeerState(ServerState.LOOKING);
            }
        break;
    }
}
```



这个伪代码实际上**非常非常重要****，大家细心的多看几遍。**

**
**

根据伪代码可以看到，当服务器状态为LOOKING时会进行领导者选举，所以我们着重来看领导者选举。



#### lookForLeader

当服务器状态为LOOKING时会调用FastLeaderElection类的lookForLeader方法，这就是领导者选举的应用层。



##### 1.初始化一个投票箱

```
HashMap<Long, Vote> recvset = new HashMap<Long, Vote>();
```



##### 2.更新选票，将票投给自己

```
updateProposal(getInitId(), getInitLastLoggedZxid(), getPeerEpoch());
```



##### 3.发送选票

```
sendNotifications();
```



##### 4.不断获取其他服务器的投票信息，直到选出Leader

```
while ((self.getPeerState() == ServerState.LOOKING) && (!stop)){
    // 从recvqueue中获取接收到的投票信息
    Notification n = recvqueue.poll(notTimeout, TimeUnit.MILLISECONDS);
    
    if (获得的投票为空) {
        // 连接其他服务器
    } else {
        // 处理投票
    }
}
```



##### 5.连接其他服务器

因为在这一步之前，都只进行了服务器的初始化，并没有真正的去与其他服务器建立连接，所以在这里建立连接。



##### 6.处理投票

判断接收到的投票所对应的服务器的状态，也就是投此票的服务器的状态：



```
switch (n.state) {
    case LOOKING:
        // PK选票、过半机制验证等
        break;
    case OBSERVING:
        // 观察者节点不应该发起投票，直接忽略
        break;
    case FOLLOWING:
    case LEADING:
        // 如果接收到跟随者或领导者节点的选票，则可以认为当前集群已经存在Leader了，直接return，退出lookForLeader方法。
}
```



##### 7. PK选票

```
if (接收到的投票的选举周期 > 本服务器当前的选举周期) {
    // 修改本服务器的选举周期为接收到的投票的选举周期
    // 清空本服务器的投票箱（表示选举周期落后，重新开始投票）
    // 比较接收到的选票所选择的服务器与本服务器的数据谁更新，本服务器将选票投给数据较新者
    // 发送选票
} else if(接收到的投票的选举周期 < 本服务器当前的选举周期){
    // 接收到的投票的选举周期落后了，本服务器直接忽略此投票
} else if(选举周期一致) {
    // 比较接收到的选票所选择的服务器与本服务器当前所选择的服务器的数据谁更新，本服务器将选票投给数据较新者
    // 发送选票
}
```



##### 8.过半机制验证

本服务器的选票经过不停的PK会将票投给数据更新的服务器，PK完后，将接收到的选票以及本服务器自己所投的选票放入投票箱中，然后从投票箱中统计出与本服务器当前所投服务器一致的选票数量，判断该选票数量是否超过集群中所有跟随者的一半（选票数量 > 跟随者数量/2），如果满足这个过半机制就选出了一个**准Leader。**

##### 9.最终确认

选出**准Leader**之后，再去获取其他服务器的选票，如果获取到的选票所代表的服务器的数据比准Leader更新，则准Leader卸职，继续选举。如果没有准Leader更新，则继续获取投票，直到没有获取到选票，则选出了最终的Leader。

Leader确定后，其他服务器的角色也确定好了。



## 领导选举完成后

上文**主线程小节**有一段非常重要的伪代码，这段伪代码达到了一个非常重要的功能，就是：

> ZooKeeper集群在进行领导者选举的过程中不能对外提供服务

根据伪代码我们可以发现，只有当集群中服务器的角色确定了之后，while才会进行下一次循环，当进入下一次循环后，就会根据服务器的角色进入到对应的初始化逻辑，初始化完成之后才能对外提供服务。

# Zookeeper请求处理原理分析

Zookeeper是可以存储数据的，所以我们可以把它理解一个数据库，实际上它的底层原理本身也和数据库是类似的。



## 数据库的原理



我们知道，数据库是用来存储数据的，只是数据可以存储在内存中或磁盘中。而Zookeeper实际是结合了这两种的，Zookeeper中的数据即会存储在**磁盘**中以达到持久化的目的，也会同步到**内存**中以到达快速访问的目的。



事实上，用过Zookeeper的同学应该知道，Zookeeper中有两种类型的节点：**持久化节点**和**临时节点**。

- 持久化节点：会持久化在磁盘中，除非主动删除，将一直存在。
- 临时节点：不会持久化在磁盘中，只会存储在内存中，创建这个临时节点的Session一旦过期，此临时节点也将自动被删除。



## 数据库处理数据的原理

作为一个数据库，肯定是要接收客户端创建、修改、删除、查询节点等请求的。



在Zookeeper中对于请求分为两类：

- 事务性请求
- 非事务性请求



### 事务性请求

Zookeeper通常都是以集群模式运行的，也就是Zookeeper集群中各个节点的数据需要保持一致的。但是和Mysql集群不一样的是：

- Mysql集群中，从服务器是异步从主服务器同步数据的，这中间的间隔时间可以比较长。
- Zookeeper集群中，当某一个集群节点接收到一个写请求操作时，该节点需要将这个写请求操作发送给其他节点，以使其他节点同步执行这个写请求操作，从而达到各个节点上的数据保持一致，也就是数据一致性。我们通常说Zookeeper保证CAP理论中的CP就只这个意思。



> Zookeeper集群底层是怎么保证数据一致性的，其实是用的**两阶段提交+过半机制**来保证的

事务性请求包括：更新操作、新增操作、删除操作，结合上面的分析，因为这些操作是会影响数据的，所以要保证这些操作在整个集群内的事务性，所以这些操作就是事务性请求。



### 非事务性请求

那么非事务性请求就好理解的，像查询操作、exist操作这些不影响数据的操场，就不需要集群来保持事务性，所以这些操场就是非事务性请求。



> Zookeeper在处理事务性请求时，比处理非事务性请求要复杂很多



## 数据在磁盘中的表示



假设我们现在在Zookeeper中有一个数据节点，节点名为`/datanode`，内容为`125`，该节点是持久化节点，所以该节点信息会保存在文件中。



可能大家都会认为是类似下面这样方式保存在磁盘文件中的，方法一：

| 节点名    | 节点内容 |
| --------- | -------- |
| /datanode | 125      |

但是除开这种表示方法，还有另外一种表示方法，**快照+事务日志**，比如方法二：



当前快照：

| 节点名    | 节点内容 |
| --------- | -------- |
| /datanode | 120      |

当前事务日志：

| 事务ID  | 操作   | 节点名    | 节点内容修改前 | 节点内容修改后 |
| ------- | ------ | --------- | -------------- | -------------- |
| 1000010 | update | /datanode | 120            | 121            |
| 1000011 | update | /datanode | 121            | 125            |

乍一看方法二比方法一要更复杂，并且占用的磁盘更多。但是我们上文提到过，Zookeeper集群中的节点在处理事务性请求时，需要将**事务操作**同步给其他节点，所以这里的事务操作是一定要进行持久化的，以便在同步给其他节点时出现异常进行补偿。所以就出现了**事务日志**。实际上事务日志还运行数据进行**回滚**，这个在两阶段提交中也是非常重要的。



那么**快照**又有什么用呢？事务日志一定要有，但是随着时间的推移，日志肯定会越来越多，所以肯定不能持久化历史上所有的日志，所以Zookeeper会定时的进行**快照**，并删除之前的日志。



那么如果按方法二这么存储数据，在对数据进行查询时就不太方便了。上文说到，Zookeeper为了提高数据的查询速度，会在内存中也存储一份数据，那么内存中的这份数据又该怎么存呢？



## 数据在内存中的表示



Zookeeper中的数据在内存中的表示其实和上文的方法一很类似，只是Zookeeper中的数据是具有文件目录特点的，说白了就是Zookeeper中的数据节点的名字一定要以`“/”`开头，这样就导致Zookeeper中的数据类似一颗树：

![image.png](https://cdn.nlark.com/yuque/0/2019/png/365147/1561620207784-24f721dc-43fa-4074-928c-056339f10657.png)



一颗具有父子层级的多叉树，在Zookeeper源码中叫**DataTree**。



## 请求处理逻辑



请看下图：

![image.png](https://cdn.nlark.com/yuque/0/2019/png/365147/1561621802788-05b75f79-a8cb-4c2a-b271-38910c25fd53.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_6bKB54-t5a2m6Zmi5ZGo55Gc%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)



> 请注意，对于上图，Zookeeper真正的底层实现，zk1是Leader，zk2和zk3是Learner，是根据[领导者选举](https://mp.weixin.qq.com/s/z73f6rQXYvh2byfkO8tMoA)选出来的。



非事务性请求直接读取DataTree上的内容，DataTree是在内存中的，所以会非常快。



## 总结

这篇文章介绍了Zookeeper在处理请求时的几个核心概念：

1. 事务性请求
2. 事务日志
3. 快照
4. DataTree
5. 两阶段提交

# Zookeeper如何解决脑裂问题

## 什么是脑裂



脑裂(split-brain)就是“大脑分裂”，也就是本来一个“大脑”被拆分了两个或多个“大脑”，我们都知道，如果一个人有多个大脑，并且相互独立的话，那么会导致人体“手舞足蹈”，“不听使唤”。



脑裂通常会出现在集群环境中，比如ElasticSearch、Zookeeper集群，而这些集群环境有一个统一的特点，就是它们有一个大脑，比如ElasticSearch集群中有Master节点，Zookeeper集群中有Leader节点。



本篇文章着重来给大家讲一下Zookeeper中的脑裂问题，以及是如果解决脑裂问题的。





## Zookeeper集群中的脑裂场景

对于一个集群，想要提高这个集群的可用性，通常会采用多机房部署，比如现在有一个由6台zkServer所组成的一个集群，部署在了两个机房：

![image.png](https://cdn.nlark.com/yuque/0/2019/png/365147/1563867147007-e5000b66-fbe7-4958-89c7-11800de04f7c.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_6bKB54-t5a2m6Zmi5ZGo55Gc%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)



正常情况下，此集群只会有一个Leader，那么如果机房之间的网络断了之后，两个机房内的zkServer还是可以相互通信的，如果**不考虑过半机制**，那么就会出现每个机房内部都将选出一个Leader。

**![image.png](https://cdn.nlark.com/yuque/0/2019/png/365147/1563867309583-b3c9d494-d91e-41f0-bb1f-310354cc14c4.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_6bKB54-t5a2m6Zmi5ZGo55Gc%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)**



这就相当于原本一个集群，被分成了两个集群，出现了两个“大脑”，这就是脑裂。



对于这种情况，我们也可以看出来，原本应该是统一的一个集群对外提供服务的，现在变成了两个集群同时对外提供服务，如果过了一会，断了的网络突然联通了，那么此时就会出现问题了，两个集群刚刚都对外提供服务了，数据该怎么合并，数据冲突怎么解决等等问题。



刚刚在说明脑裂场景时，有一个前提条件就是没有考虑过半机制，所以实际上Zookeeper集群中是不会出现脑裂问题的，而不会出现的原因就跟过半机制有关。



## 过半机制

在领导者选举的过程中，如果某台zkServer获得了超过半数的选票，则此zkServer就可以成为Leader了。

过半机制的源码实现其实非常简单：

```
public class QuorumMaj implements QuorumVerifier {
    private static final Logger LOG = LoggerFactory.getLogger(QuorumMaj.class);
    
    int half;
    
    // n表示集群中zkServer的个数（准确的说是参与者的个数，参与者不包括观察者节点）
    public QuorumMaj(int n){
        this.half = n/2;
    }

    // 验证是否符合过半机制
    public boolean containsQuorum(Set<Long> set){
        // half是在构造方法里赋值的
        // set.size()表示某台zkServer获得的票数
        return (set.size() > half);
    }
    
}
```



大家仔细看一下上面方法中的注释，核心代码就是下面两行：



```
this.half = n/2;
return (set.size() > half);
```



举个简单的例子：

如果现在集群中有5台zkServer，那么half=5/2=2，那么也就是说，领导者选举的过程中至少要有三台zkServer投了同一个zkServer，才会符合过半机制，才能选出来一个Leader。



那么有一个问题我们想一下，**选举的过程中为什么一定要有一个过半机制验证？**

因为这样不需要等待所有zkServer都投了同一个zkServer就可以选举出来一个Leader了，这样比较快，所以叫快速领导者选举算法呗。



那么再来想一个问题，**过半机制中为什么是大于，而不是大于等于呢？**



这就是更脑裂问题有关系了，比如回到上文出现脑裂问题的场景：

![image.png](https://cdn.nlark.com/yuque/0/2019/png/365147/1563868159921-23d50d01-ec38-45e3-bb93-76f4bb27f896.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_6bKB54-t5a2m6Zmi5ZGo55Gc%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)



当机房中间的网络断掉之后，机房1内的三台服务器会进行领导者选举，但是此时过半机制的条件是set.size() > 3，也就是说至少要4台zkServer才能选出来一个Leader，所以对于机房1来说它不能选出一个Leader，同样机房2也不能选出一个Leader，这种情况下整个集群当机房间的网络断掉后，整个集群将没有Leader。



而如果过半机制的条件是set.size() >= 3，那么机房1和机房2都会选出一个Leader，这样就出现了脑裂。所以我们就知道了，为什么过半机制中是**大于**，而不是**大于等于**。就是为了防止脑裂。



如果假设我们现在只有5台机器，也部署在两个机房：

![image.png](https://cdn.nlark.com/yuque/0/2019/png/365147/1563865876119-268f52aa-3fce-4337-ab5a-ed0e19fb388c.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_6bKB54-t5a2m6Zmi5ZGo55Gc%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)



此时过半机制的条件是set.size() > 2，也就是至少要3台服务器才能选出一个Leader，此时机房件的网络断开了，对于机房1来说是没有影响的，Leader依然还是Leader，对于机房2来说是选不出来Leader的，此时整个集群中只有一个Leader。



所以，我们可以总结得出，有了过半机制，对于一个Zookeeper集群，要么没有Leader，要没只有1个Leader，这样就避免了脑裂问题。