# 字典

- 实现：
    1. 哈希表
    ![xx](https://raw.githubusercontent.com/erenming/reading-books/master/implement-of-redis/images/WX20190723-110912@2x.png)
    2. 哈希表节点
    ![xx](https://raw.githubusercontent.com/erenming/reading-books/master/implement-of-redis/images/WX20190723-111109@2x.png)
        - 冲突解决：拉链法
    3. 字典
    ![xx](https://raw.githubusercontent.com/erenming/reading-books/master/implement-of-redis/images/WX20190723-111410@2x.png)
        - type属性指向一个dictType结构的指针，每个dictType结构保存了一簇用于操作特定类型键值对的函数，Redis会为不同的用途的字典设置不同的类型特定函数
        - privdata属性保存了需要传给那些类型特定函数的可选参数
- 哈希算法：
    1. 使用字典设置的哈希函数，计算key的哈希值。
        - `hash = dict->type->hashFunction(key)`
    2. 根据哈希表的sizemask属性和哈希值，计算出索引值。根据情况不同，ht[x]可以为ht[0]或者ht[1]
        - `index = hash & dict->ht[x].sizemask`
- rehash:
    1. why: 随着操作，为了维持合理的负载因子，需要对哈希表进行相应的扩展或收缩
    2. 步骤
        1. 为ht[1]分配空间
        2. **渐进式**地将ht[0]上的键值对移到ht[1], 并重新哈希
            - 首先将`rehashidx`设置为0，表示rehash工作正式开始
            - 在rehash期间，每次对字典的添加、删除、更新操作，程序顺带将对于的键值对从ht[0]转移到ht[1]上
            - 最终，ht[0]为空时，`rehashidx`设置为1，表示rehash操作完成
        3. 当ht[0]包含的所有键值对都迁移到ht[1]之后，释放ht[0], 将ht[1]设置为ht[0], 并在ht[1]上新建一个空哈希表