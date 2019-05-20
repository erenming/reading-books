# 复制

- 旧版复制功能实现
    1. 同步：当客户端向服务器发送`SLAVEOF`命令时，将从服务器的数据库状态更新至主服务器当前所处的数据库状态
    ![xx](https://raw.githubusercontent.com/erenming/reading-books/master/implement-of-redis/images/WX20190515-170415@2x.png)
    2. 命令传播：主服务器将自己执行的写命令，发送给从服务器，从而实现同步
    3. 缺陷
        - 断线后重复制：这种情况下，会对断线前已经一致的键值对重复复制，从而导致低效
- 新版复制功能实现
    1. 完整重同步：同旧版`SYNC`
    2. 部分重同步
        - 断线重连时，若条件允许，主服务器可将从服务器连接断开期间执行的写命令发送给从服务器，从服务器只需要接收并执行这些命令就可以实现同步
- 部分重同步的实现
    1. 复制偏移量
        - 主服务器每次想服务器传播N个字节的数据时，就将自己的复制偏移量的值加N
        - 从服务器每次收到主服务器传播来的N个字节数据时，就将自己的复制偏移量加上N
    2. 复制积压缓冲区
        - 由服务器维护的一个长度固定、先进先出的队列
        ![xx](https://raw.githubusercontent.com/erenming/reading-books/master/implement-of-redis/images/WX20190516-104353@2x.png)
    3. 服务器ID
        - 每个Redis服务器，无论主从，都有自己的运行ID
- 复制的实现
    1. 设置主服务器的地址和端口
    2. 建立套接字连接
    3. 发送PING命令
    4. 身份验证
    5. 发送端口信息
    6. 同步
    7. 命令传播
- 心跳检测
    1. 命令传播阶段，从服务器默认会以没秒一次的频率，向主服务器发送`REPLCONF ACK <replication_offset>`命令
    2. 用以检测主服务器的网路连接状态
        - 主服务器通过发送和接收`REPLCONF ACK`命令来检查两者之间的网络连接是否正常。
        - 若主服务器超过1秒未收到从服务器的`ACK`, 则网络连接故障了
    3. 辅助实现min-slives配置选项
    4. 检测命令丢失
        - 通过`REPLCONF ACK`发现偏移量不一样，主服务器将积压缓冲区的相应偏移段的数据重传到从服务器