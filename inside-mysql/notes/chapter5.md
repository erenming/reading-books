# 索引与算法

- B+树：一种为磁盘或其他直接存取辅助设备设计的一种平衡二叉树
    1. B+树并不能找到一个给定键值的具体行，只能找到被查找数据行所在的页
    2. 插入操作：
        - B+树的插入必须保证插入后叶子节点中的记录依然排序
        ![xx](https://raw.githubusercontent.com/erenming/reading-books/master/inside-mysql/images/WX20190503-114303@2x.png)
    3. 删除操作：
        - B+树使用填充因子（fill factor）来控制树的删除变化，50%是填充因子可设的最小值
        ![xx](https://raw.githubusercontent.com/erenming/reading-books/master/inside-mysql/images/WX20190503-114621@2x.png)
- B+ 树索引
    1. 聚集索引(clusted index)
        ![xx](https://raw.githubusercontent.com/erenming/reading-books/master/inside-mysql/images/WX20190503-145459@2x.png)
        - 按照每张表的主键构造一棵B+树，同时叶子节点（数据页）中存放的即为整张表的行记录数据
        - 在逻辑上连续
            1. 页通过双向链表链接，页按照主键的顺序排序
            2. 每个页中的记录也是通过双向链表进行维护，物理存储上同样可不按照主键顺序存储
    2. 辅助索引(secondary index)
        - 叶子节点包含键值，且还包含了一个书签(bookmark)，这个书签就是相应行数据的聚集索引键
        - 当通过辅助索引来寻找数据时，InnoDB存储引擎会遍历辅助索引并通过叶级别的指针获取指向主键索引的主键，
        然后通过主键索引来找到一个完整的行记录
    3. B+树索引的分裂
        - 并不总是从页的中间记录开始，这样可能会导致页空间的浪费
    4. B+树索引的管理
- Cardinality值
    1. 一般来说，访问表很少一部分时使用B+树索引才有意义
    2. 低选择性：可能取值的范围很小，例如性别、类型等，对此添加B+树索引毫无意义
    3. 高选择性：取值范围很广，几乎没有重复，对此添加B+树索引最适合
- B+树索引的使用
    1. 联合索引：对表上的多个列进行索引
        - 优点：已经对第二个键值进行了排序处理。例如对于索引`(a, b)`，查询语句`SELECT * FROM table where a = 1 order by b`可以避免对`b`列的实际排序
    2. 覆盖索引(convering index)：从辅助索引中就可以得到查询的记录，而不需要查询聚集索引中的记录
        - 优点1：辅助索引不包含整行记录的所有信息，故其大小远小于聚集索引，从而减少大量IO操作
- InnoDB存储引擎中的哈希算法采用的是`拉链法`处理冲突
- 全文检索(Full-Text Search)：将存储于数据库中的整本书或整篇文章中的任意内容信息查找出来的技术
    1. 倒排索引(inverted index)：也是一种索引结构，在辅助表(auxiliary table)中存储了单词和单词自身在一个或多个位置之间的映射
        - 全文索引通过其实现
        - 映射利用关联素组实现，两种表现形式
            1. incerted file index, `{单词: 单词所在的文档ID}`
               ![xx](https://raw.githubusercontent.com/erenming/reading-books/master/inside-mysql/images/WX20190504-222938@2x.png)
            2. full inverted index, `{单词：(单词所在文档的ID, 在文档中的具体位置)}`
                ![xx](https://raw.githubusercontent.com/erenming/reading-books/master/inside-mysql/images/WX20190504-223016@2x.png)
    2. InnoDB全文索引：使用了`full inverted index`
        - 辅助表(auxiliary table)是持久化的
        - FTS Index Cache（全文检索索引缓存）：用以提高性能，是红黑树结构
    3. 优势：比like快（即便加了索引，也要对索引进行扫描），功能更强大