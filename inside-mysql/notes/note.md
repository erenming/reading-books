- 数据库：是文件的集合，是依照某种数据模型组织起来并存放于二级存储器中的数据集合
- 数据库实例：是一个程序，表现为一个进程
- mysql构成：
    1. 连接池组件
    2. 管理服务和工具组件
    3. SQL接口组件
    4. 查询分析器组件
    5. 优化器组件
    6. 缓冲(Cache)组件
    7. 插件式储存引擎（关键特性）
    8. 物理文件
- InnoDB体系架构
    ![xx](https://raw.githubusercontent.com/erenming/reading-books/master/inside-mysql/images/WX20190426-155818@2x.png)
    1. 后台线程：
        - Master Thread
        - IO Thread
        - Purge Thread
        - Page Cleaner Thread
    2. 内存：
        - 缓冲池：缓存，提高性能
            ![xx](https://raw.githubusercontent.com/erenming/reading-books/master/inside-mysql/images/WX20190426-161609@2x.png)
        - LRU List, Free List和Flush List
            1. LRU List: 通常，数据库中的缓冲池通过LRU算法来管理
        - 重做日志缓冲(redo log buffer)：持久性的保障之一
        - 额外的内存池：当某些区域内存不够时，向本区域申请
    3. Checkpoint目的
        - 用以缩短数据的恢复时间
        - 缓冲池不够用时，将脏页（内存中已为最新，但是对应磁盘上的数据未同步）刷新到磁盘
        - 重做日志不可用时，刷新脏页
- Master Thread 工作方式:
    1. 主循环(loop)
    2. 后台循环(backgroud loop)
    3. 刷新循环(flush loop)
    4. 暂停循环(suspend loop)
- InnoDB 关键特性
    1. 插入缓冲
        - Insert Buffer
            1. 通常，主键是递增的（聚簇的），因此行的插入顺序是按照主键递增顺序进行插入，无需磁盘的随机读取
            2. 而对于非聚簇索引，为提高插入性能，则需要Insert Buffer技术
                - 首先判断非聚簇索引页是否在缓冲区，若在，直接插入；若不在，则先放入Insert Buffer对象中
                - 之后，以一定频率将多个插入合并为一个操作
            3. 满足条件：
                - 索引为辅助索引(secondery index)
                - 索引不是唯一的
        - Change Buffer: 升级版，对DML(INSERT, UPDATE, DELETE)都进行缓冲
        - 实现：B+树
    2. 两次写(doublewrite): 提高数据页的可靠性
        - 用以`部分写失效`后的数据恢复
        - 即在apply重做日志前，用户需要一个页的副本，当写失效发生时，先通过页的副本来还原该页，再进行重做
    3. 自适应哈希索引(Adpative Hash Index, AHI)：
        - InnoDB会监控对表上各索引页的查询，如果观察到建立哈希索引可以带来速度提升，则建立哈希索引。由此提高性能(O(1))
        - 哈希索引只能用于等值查询
    4. 异步IO: 磁盘操作均为异步IO方式
        - 异步IO能大大提高磁盘IO性能
        - 可以进行IO Merge操作，即将多个IO合并为一个IO
    5. 刷新临接页(Flush Neighbor Page)：提高传统机械硬盘的性能
        - 当刷新一个脏页时，InnoDB检测该页所在区(extent)的所有页，若是脏页，则一起刷新。
- 启动、关闭与恢复: 通过一些参数的调整，在处理数据库故障的情景下使用

