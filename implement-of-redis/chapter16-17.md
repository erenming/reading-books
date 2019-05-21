# Sential

- Sential是Redis高可用性的解决方案
    1. 由一个或多个Sential实例组成的Sential系统可以监视多个主服务器以及其下从服务器
    2. 当被监视的主服务器进入下线状态时，自动将该主服务器下的某个从服务器升级为主服务器
- 选举领头Sential
    1. 先到先得原则：最先想目标Sentinel发送设置要求的源Sential将成为目标Sential的局部领头，而之后收到的所有设置要求都会被拒绝
    2. 如果某个Sential被**半数**以上的Sential设置成了局部领头，那么这个Sential成为领头Sential

# 集群

- Redis集群是Redis提供的分布式数据库方案，集群通过分片(sharding)来进行数据共享，并提供复制和故障转移功能
- 节点
    1. 一个Redis集群通常由多个节点(node)组成，一开始相互独立并都处于一个只包含自己的集群中
    2. `CLUSTER MEET`命令实现
        ![xx](https://raw.githubusercontent.com/erenming/reading-books/master/implement-of-redis/images/WX20190520-165813@2x.png)
- 槽指派
    1. 集群的整个数据库被分为16384个槽(slot)，数据库中的每个键都属于这16384中的其中一个，集群中的每个节点可以处理0到最多16384个槽
    2. 使用`CLUSTER ADDSLOTS <slot> [slot ...]`将相应的槽分配给节点
    3. `clusterNode.slots`属性是一个二进制位数组(bit array)
    ![xx](https://raw.githubusercontent.com/erenming/reading-books/master/implement-of-redis/images/WX20190520-231502@2x.png)
        - `clusterNode.slots`在索引i上的二进制位的值为1，代表该节点处理槽i
        - `clusterNode.slots`在索引i上的二进制位的值为0，代表该节点不处理槽i
    4. 通过槽相关消息，各个节点更新`clusterState.nodes`字典中的其他节点的`clusterNode.slots`属性
    5. `clusterState.slots`数组记录了集群中所有槽的指派信息，而`clusterNode.slots`记录了该节点的槽指派信息
- 在集群中执行命令
    1. 在对数据库的16384个槽都进行指派之后，集群就会进入上线状态，此时客户端可向集群中的节点发送命令
    2. 接收命令及处理流程
        - 若键所在的槽正好指派给了当前节点，那么节点直接执行这个命令
        - 若键所在的槽并没有指派给当前节点，那么节点会返回MOVED错误，指引客户端至正确的节点并再次发送之前的命令
        ![xx](https://raw.githubusercontent.com/erenming/reading-books/master/implement-of-redis/images/WX20190521-113013@2x.png)
    3. 节点只能使用0号数据库，并用`clusterState.slots_to_keys`跳跃表来保存槽和键之间的关系
- 重新分片
    ![xx](https://raw.githubusercontent.com/erenming/reading-books/master/implement-of-redis/images/WX20190521-141408@2x.png)
- ASK错误
    1. 属于被迁移槽的的一部分键值对保存在源节点里面，而另一部分键值对则保存在目标节点里面
    ![xx](https://raw.githubusercontent.com/erenming/reading-books/master/implement-of-redis/images/WX20190521-145828@2x.png)
- 复制与故障转移
    1. 设置从节点
        - 集群中的所有节点都会在代表主节点的clusterNode结构的slaves属性和numslaves属性中记录正在复制这个主节点的从节点名单
    2. 故障检测
        - 集群中的每个节点都会定期地向急群中的其他节点发送PING消息，以此检测对方是否在线。若不在线，则标记为PFAIL(疑似下线)
        - 当急群中，超过一半负责处理槽的主节点都将某个主节点x报告为疑似下线，那么这个主节点x将被标记为FAIL(已下线)
    3. 故障转移
        ![xx](https://raw.githubusercontent.com/erenming/reading-books/master/implement-of-redis/images/WX20190521-163152@2x.png)
    4. 选举新的主节点
        - 基于Raft算法的leader selection
- 消息
    1. 集群中的各个节点通过发送和接受消息来进行通信
    2. 消息类型：
        - MEET: 发送者请求加入接受者所在集群
        - PING
            1. 集群中的每个节点默认每隔1秒就从已知节点列表中随机选择5个节点，然后对这5个节点中最长时间没有发送过PING消息的节点发送PING消息，以此检测被选节点是否在线
        - PONG: 针对PING消息的应答
        ![xx](https://raw.githubusercontent.com/erenming/reading-books/master/implement-of-redis/images/WX20190521-170911@2x.png)
        - FAIL: 主A判断主B处于FAIL状态，A广播一条关于B的FAIL消息
        ![xx](https://raw.githubusercontent.com/erenming/reading-books/master/implement-of-redis/images/WX20190521-170942@2x.png)
        - REPUBLISH