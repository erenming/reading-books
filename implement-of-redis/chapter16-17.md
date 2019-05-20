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
    3. 槽指派
        - 集群的整个数据库被分为16384个槽(slot)，数据库中的每个键都属于这16384中的其中一个，集群中的每个节点可以处理0到最多16384个槽
        - 使用`CLUSTER ADDSLOTS <slot> [slot ...]`将相应的槽分配给节点
        - `clusterNode.slots`属性是一个二进制位数组(bit array)
        ![xx](https://raw.githubusercontent.com/erenming/reading-books/master/implement-of-redis/images/WX20190520-231502@2x.png)
            1. `clusterNode.slots`在索引i上的二进制位的值为1，代表该节点处理槽i
            2. `clusterNode.slots`在索引i上的二进制位的值为0，代表该节点不处理槽i
        - 通过槽相关消息，各个节点更新`clusterState.nodes`字典中的其他节点的`clusterNode.slots`属性
        - `clusterState.slots`数组记录了集群中所有槽的指派信息，而`clusterNode.slots`记录了该节点的槽指派信息