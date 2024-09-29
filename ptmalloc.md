ptmalloc

特点

- 



nginx内存池

特点

- pool的大小，如果是小内存是通过第一个pool的max确定，大内存根据实际确定，通过next指针连起来
- 大小内存定义以传入size与分页大小-1的最小值为界限
- 使用unsigned long计算对齐偏移位置指针
- 分配内存时，large链表就遍历头3个
- 单块内存释放时，仅释放large链表，small链表通过内存池析构释放

缺点

- small内存只增加不释放
- large内存释放会调用free，没有复用