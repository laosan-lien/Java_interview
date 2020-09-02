# MySQL

## mysql的分布式事务支持

MySQL分布式事务介绍

**InnoDB**存储引擎提供了对XA事务的支持，并通过XA事务来支持分布式事务的实现。分布式事务指的是允许多个独立的事务资源参与到一个全局的事务中。事务资源通常是关系型数据库系统，但也可以是其他类型的资源。全局事务要求在其中的所有参与的事务要么都提交，要么都回滚，这对于事务原有的ACID要求又有了提高。**另外，在使用分布式事务时，InnoDB存储引擎的事务隔离级别必须设置为SERIALIZABLE。**

XA事务语允许不同数据库之间的分布式事务，如一台服务器是MySQL数据库的，另一台是Oracle数据库的，又可能还有一台服务器是SQL Server数据库的，只要参与在全局事务中的每个节点都支持XA事务。分布式事务可能在分布式架构或银行系统的转账中比较常见，入用户David需要从上海转10000元到北京的用户Mariah的银行卡中。

\# Bank@Shanghai

update account set money = money - 10000 where user='David';

\# Bank@Beijing

update account set money = money + 10000 where user='Mariah';

在这种情况下，一定需要使用分布式事务来保证数据的安全。如果发生的操作不能全部提交或回滚，那么任何一个节点出现问题都会导致严重的结果。要么是David的账号被扣了款，但Mariah没收到，又或者是David的账户没有扣款，Mariah却收到钱了。

XA事务由一个或多个资源管理器（resource managers）、一个事务管理器（transaction manager）以及一个应用程序（application program）组成。

**资源管理器**：提供访问事务资源的方法，通常一个数据库就是一个资源管理器。

**事务管理器**：协调参与全局事务中的各个事务，需要和参与全局事务的所有资源管理器进行通信。

**应用程序**：定义事务的边界，指定全局事务中的操作。

在MySQL数据库的分布式事务中，资源管理器就是MySQL数据库，事务管理器为连接MySQL服务器的客户端。下图显示了一个分布式事务的模型。



![img](https:////upload-images.jianshu.io/upload_images/9565093-675e8cccfe8eb332..jpg?imageMogr2/auto-orient/strip|imageView2/2/w/528/format/webp)

分布式事务通常采用2PC协议，全称Two Phase Commitment Protocol。该协议主要为了解决在分布式数据库场景下，所有节点间数据一致性的问题。在分布式事务环境下，事务的提交会变得相对比较复杂，因为多个节点的存在，可能存在部分节点提交失败的情况，即事务的ACID特性需要在各个数据库实例中保证。总而言之，在分布式提交时，只要发生一个节点提交失败，则所有的节点都不能提交，只有当所有节点都能提交时，整个分布式事务才允许被提交。

分布式事务通过2PC协议将提交分成两个阶段，在第一阶段，所有参与全局事务的节点都开始准备（PREPARE），告诉事务管理器它们准备好提交了。在第二阶段，事务管理器告诉资源管理器执行ROLLBACK还是COMMIT。如果任何一个节点显示不能提交，则所有的节点都被告知需要回滚。可见与本地事务不同的是，分布式事务需要多一次的PREPARE操作，待收到所有节点的同意信息后，再进行COMMIT或是ROLLBACK操作。

MySQL分布式事务操作

**1. MySQL XA事务的语法**

主要有：

\# 在mysql实例中开启一个XA事务,指定一个全局唯一标识；

mysql> XA START 'any_unique_id';

\# XA事务的操作结束；

mysql> XA END 'any_unique_id ';

\# 告知mysql准备提交这个xa事务；

mysql> XA PREPARE 'any_unique_id';

\# 告知mysql提交这个xa事务；

mysql> XA COMMIT 'any_unique_id';

\# 告知mysql回滚这个xa事务；

mysql> XA ROLLBACK 'any_unique_id';

\# 查看本机mysql目前有哪些xa事务处于prepare状态；

mysql> XA RECOVER;

**2. XA事务恢复**

如果执行分布式事务的mysql crash了，MySQL按照如下逻辑进行恢复：

a. 如果这个xa事务commit了，那么什么也不用做。

b. 如果这个xa事务还没有prepare，那么直接回滚它。

c. 如果这个xa事务prepare了，还没commit， 那么把它恢复到prepare的状态，由用户去决定commit或rollback。

当mysql crash后重新启动之后，执行“XA RECOVER；”查看当前处于prepare状态的xa事务，然后commit或rollback它们。

**MySQL分布式事务限制**

**a. XA事务和本地事务以及锁表操作是互斥的**

开启了xa事务就无法使用本地事务和锁表操作

mysql> xa start 't1xa';

Query OK, 0 rows affected (0.04 sec)

mysql> begin;

ERROR 1399 (XAE07): XAER_RMFAIL: The command cannot be executed when global transaction is in theACTIVE state

mysql> lock table t read;

ERROR 1399 (XAE07): XAER_RMFAIL: The command cannot be executed when global transaction is in theACTIVE state

开启了本地事务就无法使用xa事务

mysql> begin;

Query OK, 0 rows affected (0.00 sec)

mysql> xa start 'rrrr';

ERROR 1400 (XAE09): XAER_OUTSIDE: Some work is done outside global transaction

**b. xa start之后必须xa end，否则不能执行xa commit和xa rollback**

所以如果在执行xa事务过程中有语句出错了，你也需要先xa end一下，然后才能xa rollback。

mysql> xa start 'tt';

Query OK, 0 rows affected (0.00 sec)

mysql> xa rollback 'tt';

ERROR 1399 (XAE07): XAER_RMFAIL: The command cannot be executed when global transaction is in the ACTIVE state

mysql> xa end 'tt';

Query OK, 0 rows affected (0.00 sec)

mysql> xa rollback 'tt';

Query OK, 0 rows affected (0.00 sec)

**MySQL 5.7对分布式事务的支持**

一直以来，MySQL数据库是支持分布式事务的，但是只能说是有限的支持，具体表现在：

已经prepare的事务，在客户端退出或者服务宕机的时候，2PC的事务会被回滚。

在服务器故障重启提交后，相应的Binlog被丢失。

上述问题存在于MySQL数据库长达数十年的时间，直到MySQL-5.7.7版本，官方才修复了该问题。下面将会详细介绍下该问题的具体表现和官方修复方法，这里分别采用官方MySQL-5.6.27版本(未修复)和MySQL-5.7.9版本(已修复)进行验证。

先来看下存在的问题，我们先创建一个表如下：

CREATE TABLE t(

id INT AUTO_INCREMENT PRIMARY KEY,

a INT

)ENGINE=InnoDB;

对于上述表，通过如下操作进行数据插入：

mysql> XA START 'mysql56';

mysql> INSERT INTO t VALUES(1,1);

mysql> XA END 'mysql56';

mysql> XA PREPARE 'mysql56';

通过上面的操作，用户创建了一个分布式事务，并且prepare没有返回错误，说明该分布式事务可以被提交。通过命令XA RECOVER查看显示如下结果：

mysql> XA RECOVER;

+----------+--------------+--------------+---------+

| formatID | gtrid_length | bqual_length | data |

+----------+--------------+--------------+---------+

| 1  | 7   | 0   | mysql56 |

+----------+--------------+--------------+---------+

若这时候用户退出客户端后重连，通过命令xa recover会发现刚才创建的2PC事务不见了。***即prepare成功的事务丢失了，不符合2PC协议规范！！！***

产生上述问题的主要原因在于：MySQL 5.6版本在客户端退出的时候，自动把已经prepare的事务回滚了，那么MySQL为什么要这样做？这主要取决于MySQL的内部实现，MySQL 5.7以前的版本，对于prepare的事务，***MySQL****是不会记录binlog的（**官方说是减少fsync，起到了优化的作用）。只有当分布式事务提交的时候才会把前面的操作写入binlog信息，所以对于binlog来说，***分布式事务与普通的事务没有区别，而prepare以前的操作信息都保存在连接的IO_CACHE中，如果这个时候客户端退出了，以前的binlog信息都会被丢失，再次重连后允许提交的话，会造成Binlog丢失，从而造成主从数据的不一致，所以官方在客户端退出的时候直接把已经prepare的事务都回滚了！

官方的做法，貌似干得很漂亮，牺牲了一点标准化的东西，至少保证了主从数据的一致性。但其实不然，若用户已经prepare后在客户端退出之前，MySQL发生了宕机，这个时候又会怎样？

MySQL在某个分布式事务prepare成功后宕机，宕机前操作该事务的连接并没有断开，这个时候已经prepare的事务并不会被回滚，所以在MySQL重新启动后，引擎层通过recover机制能恢复该事务。当然该事务的Binlog已经在宕机过程中被丢失，这个时候，如果去提交，则会造成主从数据的不一致，**即提交没有记录Binlog，从上丢失该条数据。**所以对于这种情况，官方一般建议直接回滚已经prepare的事务。

以上是MySQL 5.7以前版本MySQL在分布式事务上的各种问题，那么MySQL 5.7版本官方做了哪些改进？这个可以从官方的WL#6860描述上得到一些信息，我们还是本着没有实践就没有发言权的态度，从具体的操作上来分析下MySQL 5.7的改进方法。还是以上面同样的表结构进行同样的操作如下：

mysql> XA START 'mysql57';

mysql> INSERT INTO t VALUES(1,1);

mysql> XA END 'mysql57';

mysql> XA PREPARE 'mysql57'

这个时候，我们通过mysqlbinlog来查看下Master上的Binlog，结果如下：



![img](https:////upload-images.jianshu.io/upload_images/9565093-867c5db86302502d..png?imageMogr2/auto-orient/strip|imageView2/2/w/1000/format/webp)

同时也对比下Slave上的Relay log，如下：



![img](https:////upload-images.jianshu.io/upload_images/9565093-dd5935fefd7c4e55..png?imageMogr2/auto-orient/strip|imageView2/2/w/1000/format/webp)

通过上面的操作，明显发现在prepare以后，从XA START到XA PREPARE之间的操作都被记录到了Master的Binlog中，然后通过复制关系传到了Slave上。也就是说MySQL 5.7开始，**MySQL对于分布式事务，在prepare的时候就完成了写Binlog的操作，通过新增一种叫XA_prepare_log_event的event类型来实现，这是与以前版本的主要区别(以前版本prepare时不写Binlog)。**

当然仅靠这一点是不够的，因为我们知道Slave通过SQL thread来回放Relay log信息，由于prepare的事务能阻塞整个session，而回放的SQL thread只有一个(不考虑并行回放)，那么SQL thread会不会因为被分布式事务的prepare阶段所阻塞，从而造成整个SQL thread回放出现问题？这也正是官方要解决的第二个问题：怎么样能使SQL thread在回放到分布式事务的prepare阶段时，不阻塞后面event的回放？其实这个实现也很简单(在xa.cc::applier_reset_xa_trans)，只要在SQL thread回放到prepare的时候，进行类似于客户端断开连接的处理即可(把相关cache与SQL thread的连接句柄脱离)。最后在Slave服务器上，用户通过命令XA RECOVER可以查到如下信息：

mysql> XA RECOVER;

+----------+--------------+--------------+---------+

| formatID | gtrid_length | bqual_length | data |

+----------+--------------+--------------+---------+

| 1  | 7   | 0   | mysql57 |

+----------+--------------+--------------+---------+

至于上面的事务什么时候提交，一般等到Master上进行XA COMMIT  ‘mysql57’后，slave上也同时会被提交。

**总结**

综上所述，MySQL 5.7对于分布式事务的支持变得完美了，一个长达数十年的bug又被修复了，因而又多了一个升级到MySQL 5.7版本的理由。