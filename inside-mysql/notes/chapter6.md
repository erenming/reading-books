# 锁

- 锁机制用于管理对共享资源的并发访问，提供数据的完整性和一致性
- InnoDB存储引擎中的锁
    1. 锁的类型
        - 共享锁（S Lock），允许事务读一行数据
        - 排它锁（X Lock），允许事务删除或更新一行数据
        ![xx](https://raw.githubusercontent.com/erenming/reading-books/master/inside-mysql/images/WX20190506-223543@2x.png)
        - 意向锁(Intention Lock): 可将锁定的对象分为多个层次，这意味着事务希望在更细粒度上进行加锁
    2. 一致性非锁定读(consistent nonblocking read): 通过行多版本控制的方式来读取当前执行时间数据库中行的数据