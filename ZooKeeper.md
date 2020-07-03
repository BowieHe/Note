ZooKeeper是一个小型的分布式文件系统，和FastDFS不同，zookeeper只适合储存一些小型的数据或者配置信息

# zookeeper的文件系统

zookeeper底层是一个树形结构，进行数据的存储

```
/
--/NameService
----/Server1
----/Server2
--Configuration
--GroupMembers
----/Member1
----/Member2
--/Apps
----/App1
----/App2
----/App3
------/SubApp1
------/SubApp2
```

zookeeper文件系统和Linux和Windows不同，Linux和Windows中有文件和文件夹概念，文件夹本身不存放数据，文件本身用来存放数据

Zookeeper中的节点，没有文件夹和文件之分，所有的节点都可以进行数据存储，同时也拥有字节点，每个节点成为znode

**znode分类**：

1. ephemeral-临时节点：

   临时节点由某个客户端创建，如果该客户端断开了和zookeeper服务器的连接，该临时节点就会被删除；**临时节点不能有字节点**

2. persistent-持久性节点：

   持久化的节点会永久存在于文件系统中，除非客户端显示的删除该节点

3. ephemeral_sequential-临时顺序节点：

   和临时节点拥有相同的特点，唯一的区别在于该节点会自动维护一个编号

4. persistent_swquential-持久性顺序节点：

   和持久性节点由相同的特点，唯一区别在于该节点会自动维护一个编号

**文件系统操作命令**：

- ls：

  查看路径字节点情况，zk中只能写绝对路径

- Create [-s] [-e] path data

  创建一个节点，在path的路径位置，数据为data（不能为空）；-s为顺序节点，-e为临时节点

- get path

  查看指定路径对应节点数据，每个节点分为数据部分和描述信息

- set path data

  修改指定节点的数据

- delete path

  删除置顶节点数据，如果下面有字节点需要先删除字节点

# Zookeeper的通知机制

客户端可以选择对某个znode 进行监听。当这个znode发生变化的时候（本身添加，删除，修改和字节点的变化），会主动监听这个znode客户端。zookeeper的通知机制有一次性触发原则，znode发生变化后，一旦通知了客户端，则断开客户端的监听。如果继续监听节点的变化，必须重新发起监听。

- exisits-监听节点创建，内容修改，节点的删除
- getData-监听节点的内容修改和节点删除
- getChildren- 监听字节点的添加，删除（字节点的内容变化和字节点的字节点变化不能监听）

# Zookeeper的运用场景

## 配置文件统一管理

在分布式集群的工程中，通常由很多服务部署在不同的服务上，每个服务都有自己的配置信息，如果需要修改某个配置，则可能需要对多态服务器进行配置的修改，是非常不方便的。那么就可以使用zookeeper帮助我们进行统一的配置文件管理。

```
/
--/Configuration
---(watch)--client
---(watch)--client
---(watch)--client
---(watch)--client
```

在zookeeper上创建一个持久化节点，将所有的配置信息都放入这个节点，然后每台服务器都去监听这个节点的变化（watch）。如果有新的配置信息，开发者只需要上传到zookeeper这个节点（更新节点的配置数据）。每个服务器就能收到zookeeper节点的更新通知，然后从节点中读取新的配置，应用到系统中，完成配置更新。

## 集群管理

在某些集群中，可能需要知道其他的集群服务器状态，比如有新的机器加入集群，或者有老的机器退出集群，可以通过zookeeper进行统一的集群管理

```
/
--/GroupMemvers
----/client1  <- regist <- client1
----/client2  <- regist <- client2
----/client3  <- regist <- client3
----/client4  <- regist <- client4
```

## 分布式锁

1. 保持独占

   所有的客户端同时创建一个lock临时节点，谁创建成功谁获得锁。释放锁的时候删除这个节点

2. 控制顺序

   所有客户端在同一节点下面创建临时的顺序节点。只需要让编号最小的机器获得锁

# zookeeper的集群

**集群工作原理**

zookeeper集群可能有N台机器，这些机器中一定会存在一个leader，其他的机器就是follower。对于客户端来说，可以随意连接任何一个集群中服务器。如果某个客户端需要对zk进行更改的操作，这些操作命令最终需要提交给leader。leader将命令分发给所有的集群服务器。当一半以上的集群服务器执行该命令成功，则leader就会通知所有节点进行事务提交，达到数据同步更新的目的。

**zookeeper集群的过半数存活原则**

在zookeeper集群中，只有当存活的机器数量超过总集群一半的时候，整个集群才能正常工作。基于过半数存活原则，zookeeper的集群数量一定是奇数

**原因**

因为整个集群中，有可能因为“脑裂“，导致整个集群分为2个甚至多个集群，如果没有“过半数存活的机制“，那么整个zookeeper集群提供的数据将无法再保证数据一致性。所以为了保证整体数据的强一致性，zookeeper规定了过半数存活这个原则。

**zookeeper集群角色**

leader, follower, observer（功能和follower一样，区别在于observer不回参与投票环节）

# leader选举

服务器状态

- looking：寻找leader状态。当服务器处于该状态时，它会认为当前集群中没有 leader，因此需要进入leader选举状态。
- leading： 领导者状态。表明当前服务器角色是leader。
- following： 跟随者状态。表明当前服务器角色是follower。
- observing：观察者状态。表明当前服务器角色是observer

### 服务器启动时期的leader选举

在集群初始化阶段，当有一台服务器server1启动时，其单独无法进行和完成 leader选举，当第二台服务器server2启动时，此时两台机器可以相互通信，每台机器都试图找到leader，于是进入leader选举过程。选举过程如下:

> 1. 每个server发出一个投票。由于是初始情况，server1和server2都会将自己作为 leader服务器来进行投票，每次投票会包含所推举的服务器的myid和zxid，使用(myid, zxid)来表示，此时server1的投票为(1, 0)，server2的投票为(2, 0)，然后各自将这个投票发给集群中其他机器。
> 2. 集群中的每台服务器接收来自集群中各个服务器的投票。
> 3. 处理投票。针对每一个投票，服务器都需要将别人的投票和自己的投票进行pk，pk规则如下优先检查zxid。zxid比较大的服务器优先作为leader。如果zxid相同，那么就比较myid。myid较大的服务器作为leader服务器。对于Server1而言，它的投票是(1, 0)，接收Server2的投票为(2, 0)，首先会比较两者的zxid，均为0，再比较myid，此时server2的myid最大，于是更新自己的投票为(2, 0)，然后重新投票，对于server2而言，其无须更新自己的投票，只是再次向集群中所有机器发出上一次投票信息即可。
> 4. 统计投票。每次投票后，服务器都会统计投票信息，判断是否已经有过半机器接受到相同的投票信息，对于server1、server2而言，都统计出集群中已经有两台机器接受了(2, 0)的投票信息，此时便认为已经选出了leader
> 5. 改变服务器状态。一旦确定了leader，每个服务器就会更新自己的状态，如果是follower，那么就变更为following，如果是leader，就变更为leading

### 服务器运行时期的Leader选举

- 在zookeeper运行期间，leader与非leader服务器各司其职，即便当有非leader 服务器宕机或新加入，此时也不会影响leader，但是一旦leader服务器挂了，那么整个集群将暂停对外服务，进入新一轮leader选举，其过程和启动时期的Leader选举过程基本一致。
- 假设正在运行的有server1、server2、server3三台服务器，当前leader是 server2，若某一时刻leader挂了，此时便开始Leader选举。选举过程如下:

> 1. 变更状态。leader挂后，余下的服务器都会将自己的服务器状态变更为looking，然后开始进入leader选举过程。
> 2. 每个server会发出一个投票。在运行期间，每个服务器上的zxid可能不同，此时假定server1的zxid为122，server3的zxid为122，在第一轮投票中，server1和server3都会投自己，产生投票(1, 122)，(3, 122)，然后各自将投票发送给集群中所有机器。
> 3. 接收来自各个服务器的投票。与启动时过程相同
> 4. 处理投票。与启动时过程相同，此时，server3将会成为leader。
> 5. 统计投票。与启动时过程相同。
> 6. 改变服务器的状态。与启动时过程相同。