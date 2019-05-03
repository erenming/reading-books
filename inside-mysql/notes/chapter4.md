# 章节4-表

- 索引组织表：根据主键顺序组织存放的一种存储方式
    1. 首先判断表中是否有非空的唯一索引，若有，则该列即为主键
    2. 如果不符合上述条件，InnoDB存储引擎自动创建一个6字节大小的指针
- InnoDB逻辑存储结构：
    ![xx](https://raw.githubusercontent.com/erenming/reading-books/master/inside-mysql/images/WX20190429-231302@2x.png)
    1. 所有数据逻辑地存于表空间(tablespace), 其又由段(segment)、区（extent）、页(page)
    2. 表空间：存储所有数据
        - 若启动`innodb_file_per_table`，则每张表内的数据单独一个表空间
        - 但每个表自己的表空间只存数据、索引和插入缓冲
        - 其他存于原来的共享表空间里
    3. 段：常见数据段、索引段、回滚段等
    4. 区：连续页组成的空间，每个区大小为1M
    5. 页：或块，磁盘管理的最小单位
    6. 行：InnoDB储存引擎是面向行的(row-oriented)
- 约束：保证数据的完整性
- 视图：一个命名的虚表，数据没有实际的物理存储
    
