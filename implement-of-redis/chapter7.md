# 压缩列表

- 一种为节省内存而开发的顺序型数据结构
- 其作为列表键和哈希键的底层实现之一
- 实现特点

    1. 节点的`previous_entry_length`: 保存上一个节点的长度，通过指针运算，可快速地查找上一个节点
    2. 节点的`encoding`属性，通过对`位`的编码，实现区分字节数组和整数
