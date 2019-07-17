# 链表

- 链表节点
    ![xx](https://raw.githubusercontent.com/erenming/reading-books/master/implement-of-redis/images/WX20190717-160830@2x.png)
- 链表
    ![xx](https://raw.githubusercontent.com/erenming/reading-books/master/implement-of-redis/images/WX20190717-160942@2x.png)
    1. 双端：prev, next指针，获取前后节点为O(1)
    2. 无环
    3. 带表头、表尾指针
    4. 带链表长度计数器
    5. 多态：通过`dup`、`free`、`match`为节点值类型特定函数，所以链表可以存各种类型的值