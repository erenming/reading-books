# 事件

- Redis是一个事件驱动程序
- 文件事件
    ![xx](https://raw.githubusercontent.com/erenming/reading-books/master/implement-of-redis/images/WX20190515-144534@2x.png)
    1. 尽管多个文件事件可能会并发地出现，但I/O多路复用程序总是会将所有产生事件的套接字，放入一个队列中。然后一个个地向文件事件分派器发送
    2. 事件的类型：如果一个套接字又可读又可写的话，那么服务器将先读套接字、后写套接字
- 时间事件
    1. 定时事件
    2. 周期性事件
    3. 组成属性：
        - id: 服务器为时间事件创建的全局唯一ID
        - when: 毫秒级的时间戳
        - timeProc: 时间时间处理器