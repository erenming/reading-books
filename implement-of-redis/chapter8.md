# 对象

- 结构: redis 用对象来包裹特定的数据结构

```c
    typedef struct redisObject {
        // 类型
        unsinged type:4;
        // 编码
        unsigned encoding:4;
        // 指向底层实现数据结构的指针
        void *ptr
        // 引用计数
        int refcount;
        // 对象最后一次被访问时间
        unsigned lru:22;
    }
```

- 类型
    1. 一共5种，`REDIS_STRING, REDIS_LIST, REDIS_HASH, REDIS_SET, REDIS_ZSET`
    2. 对于键值对，键为`REDIS_STRING`, 值为上述5种之一
- 编码
    1. 对象的ptr指针指向的底层数据结构，由对象的encoding属性决定
    2. 优势：使得Redis可以根据不同的场景动态地位一个对象设置不同的编码，从而优化对象的效率
        - 例如元素较少时，使用压缩列表，省内存，劣势可忽略
        - 元素较多时，切换为更为合适双端列表
- 类型检查与多态
    1. 类型检查：通过检查键key的值的类型
    2. 多态：通过对象的编码，来动态选择方法
- 内存回收：使用引用计数机制
