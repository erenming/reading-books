# 简单动态字符串

- Redis构建了一种名为简单动态字符串(SDS)的抽象类型，作为Redis的默认字符串表示
- SDS定义：
    ![xx](https://raw.githubusercontent.com/erenming/reading-books/master/implement-of-redis/images/WX20190717-110720@2x.png)
    1. SDS遵循C字符串以空字符结尾的惯例，空字符不计入len，是为了方便直接调用C相关函数
- SDS与C字符串的区别
    1. 获取字符串长度的时间复杂度为O(1)
        - 因为len字段会记录
        - 而C字符串则每次都需要遍历字符串
    2. 可杜绝缓冲区溢出
        - SDS的相关API会根据需要对SDS的空间进行扩容或缩容
        - 而C字符串需要程序员手动申请足够的空间，否则就会缓冲区溢出
    3. 减少字符串修改带来的内存重分配次数
        - 扩容策略
            1. len < 1M, 则buf扩容为原len的两倍
            2. len >= 1M, 修改后的所有字符的基础长度上，再加1M
        - 缩容策略
            1. 字符串缩短后，根据实际字符长度，动态改变free和len。
            2. 特定时，使用SDS的API真正释放空间
    4. 二进制安全
        - buf里面存的是字节(二进制数据)而非实际字符
        