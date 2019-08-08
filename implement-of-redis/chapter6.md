# 整数集合

- 实现

    ```c
    typedef struct intset {

        // 编码方式
        uint32_t encoding;

        // 集合包含的元素数量
        uint32_t length;

        // 保存元素的数组
        int8_t contents[];

    } intset;
    ```

    1. 注意`contents`数组的元素类型会随`encoding`而变化，例如`encoding`为`INTSET_ENC_INT_32`时，元素就是一个`int32_t`类型

- 升级
    1. 当新元素比现有元素都要长时，需要升级操作
    2. 好处：
        - 提升整数集合的灵活性
        - 尽可能的节省内存
- 不支持降级