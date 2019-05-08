# 锁

- 锁机制用于管理对共享资源的并发访问，提供数据的完整性和一致性
- InnoDB存储引擎中的锁
    1. 锁的类型
        - 共享锁（S Lock），允许事务读一行数据
        - 排它锁（X Lock），允许事务删除或更新一行数据
        ![xx](https://raw.githubusercontent.com/erenming/reading-books/master/inside-mysql/images/WX20190506-223543@2x.png)
        - 意向锁(Intention Lock): 可将锁定的对象分为多个层次，这意味着事务希望在更细粒度上进行加锁
    2. 一致性非锁定读(consistent nonblocking read): 通过行多版本控制的方式来读取当前执行时间数据库中行的数据
        - 如果读取的行正在执行DELETE和UPDATE操作，这时读取操作不会因此去等待行上锁的释放，相反去读取行的一个快照
        - 一个行记录可能有不止一个快照数据，一般称为行多版本技术。带来的并发控制称为多版本并发控制(Multi Version Concurrency Control, MVCC)
        - 下述事务隔离级别使用非锁定的一致性读
            1. `READ COMMITTED`: 对于快照数据，总是读取被锁定行的最新一份快照数据
            2. `REPEATABLE READ`: 对于快照数据，总是读取事务开始时的行数据版本
    3. 自增长与锁
        - 每个含有自增长值的表都有一个自增长计数器
        - 插入操作会依据这个计数器加1赋予自增长列，该方式称为`AUTO-INC Locking`
    4. 外键与锁
        - 对于一个外键列，如果没有显式地对这个列加索引，InnoDB存储引擎自动对其加一个索引
- 锁的算法
    1. Record Lock: 单个行记录上的锁
        - 总是会去锁住索引记录，若无索引则隐式使用主键来锁定
    2. Gap Lock: 间隙锁，锁定一个范围，但不包括记录本身
    3. Next-Key Lock: Gap Lock + Record Lock, 锁定一个范围，并且锁定记录本身
        - 用于解决`Phantom Problem`，左开右闭
    4. 解决`Phantom Problem`（幻象问题）