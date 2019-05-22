# RDB 持久化

- 数据库状态：Redis服务器中的非空数据库以及它们的键值对统称
- 通过保存数据库中的键值对来记录数据库状态
- RDB文件的创建与载入
    1. 使用SAVE(同步阻塞)或BGSAVE(多进程异步)
    2. Redis在启动时检测到RDB文件存在，它就会自动载入RDB文件
    ![xx](https://raw.githubusercontent.com/erenming/reading-books/master/implement-of-redis/images/WX20190514-164801@2x.png)
    3. 载入时，服务器一直处于阻塞
- 自动间隔保存
    1. 设置保存条件`save`：对应于`saveparams`
    2. `dirty`计数器：记录了距离上一次成功执行SAVE或BGSAVE命令之后，服务器对数据库状态进行了多少次修改(DML)
    3. `lastsave`时间戳：记录了服务器上一次成功执行SAVE或BGSAVE命令的时间
    ![xx](https://raw.githubusercontent.com/erenming/reading-books/master/implement-of-redis/images/WX20190515-092135@2x.png)
    4. 程序遍历`saveparams`中的所有条件，只要其中一个满足，那么服务器执行`BGSAVE`命令

# AOF 持久化

- 通过保存Redis服务器所执行的写命令来记录数据库状态
- AOF持久化实现
    1. 命令追加
        - AOF开启下，服务器在执行完一个写命令后，会以协议格式将被执行的写命令追加到服务器状态的`aof_buf`缓冲区的末尾
    2. AOF文件的写入与同步
        - Redis服务器进程可以说就是一个事件循环
        ![xx](https://raw.githubusercontent.com/erenming/reading-books/master/implement-of-redis/images/WX20190515-112637@2x.png)
    3. AOF文件的载入与数据还原
        ![xx](https://raw.githubusercontent.com/erenming/reading-books/master/implement-of-redis/images/WX20190515-134424@2x.png)
    4. AOF重写
        - 通过重写，Redis服务器可以创建一个新的AOF文件来替代现有的AOF文件，新旧两者保存的数据库状态相同
        - 新的AOF文件整合了就AOF中的冗余的命令
        - AOF重写是有歧义的名字，因为实际上它是通过读取数据库中的键值对来实现的，程序无需对现有的AOF文件进行任何操作