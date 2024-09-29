## 内存池杂项

### 分支预测

函数：**long __builtin_expect (long exp, long c)**

- ***exp**：exp* 为一个整型表达式, 例如: (ptr != NULL)

- **c**：*c* 必须是一个编译期常量, 不能使用变量

- **返回值**：返回值等于 第一个参数 *exp*

C代码源码：

```c
#define __plugin_unlikely(cond) __builtin_expect (!!(cond), 0)
#define __plugin_likely(cond) __builtin_expect (!!(cond), 1)
```

原理：

1. CPU在运行程序时，每次会取多个指令并行处理，如果遇到跳转指令，会重新取指令
2. 在if else里，如果需要运行else的代码，需要jmp指令跳转，在大量次数执行的代码里会降低执行效率
3. __builtin_expect 函数可以优化分支预测

使用：

```c
int x = 0;
if (__builtin_expect(x == 0, 1)) {
    printf("X is zero.\n");
} else {
    printf("X is not zero.\n");
}
```

使用__builtin_expect汇编：

```
movl    $0, -4(%rbp)
cmpl    $0, -4(%rbp)
jne     .L2
movl    $.LC0, %edi
call    printf
jmp     .L3
.L2:
movl    $.LC1, %edi
call    printf
.L3:
```

不使用__builtin_expect汇编：

```
movl    $0, -4(%rbp)
cmpl    $0, -4(%rbp)
je      .L2
movl    $.LC1, %edi
call    printf
jmp     .L3
.L2:
movl    $.LC0, %edi
call    printf
.L3:
```

可以看到代码编排有区别

- 使用__builtin_expect后，预测cmpl为1，使用jne，不跳转
- 不使用__builtin_expect，使用je，跳转.L2



### 内存分配对齐

宏：request2size

功能：根据内存对齐计算对齐后，需要多少大小内存

前置宏定义解析

```C++
#define INTERNAL_SIZE_T size_t// 32位4字节，64位8字节

#define SIZE_SZ (sizeof (INTERNAL_SIZE_T))// 32位4字节，64位8字节

#define MALLOC_ALIGNMENT (2 * SIZE_SZ < __alignof__ (long double) \
			  ? __alignof__ (long double) : 2 * SIZE_SZ)// 选取最大对齐大小，long double大小看平台，32平台10字节，64平台16字节

#define MALLOC_ALIGN_MASK (MALLOC_ALIGNMENT - 1)// 根据最大对齐大小取掩码

#define MIN_CHUNK_SIZE (offsetof(struct malloc_chunk, fd_nextsize))// 分块最小大小

#define offsetof(s,m) ((size_t)&(((s*)0)->m))// 得到结构体指针fd_nextsize在结构体malloc_chunk里的偏移量

#define MINSIZE  \
  (unsigned long)(((MIN_CHUNK_SIZE+MALLOC_ALIGN_MASK) & ~MALLOC_ALIGN_MASK))// 最小对齐大小
```

定义

```C++
#define request2size(req)                                         \
  (((req) + SIZE_SZ + MALLOC_ALIGN_MASK < MINSIZE)  ?             \
   MINSIZE :                                                      \
   ((req) + SIZE_SZ + MALLOC_ALIGN_MASK) & ~MALLOC_ALIGN_MASK) 
```

解析：

- `req + SIZE_SZ` 是用户请求的大小加上元数据的大小
- 加上 `MALLOC_ALIGN_MASK`，保证向上取到下一个对齐边界，即从MALLOC_ALIGN_MASK的倍数变成倍数+1
- 位与操作 `& ~MALLOC_ALIGN_MASK` 用于清除最低的几位，得到对齐后的真正大小



### smallbin计算

```c++
#define NSMALLBINS 64

#define MALLOC_ALIGNMENT (2 * SIZE_SZ < alignof (long double) ? alignof (long double) : 2 * SIZE_SZ) 

#define SMALLBIN_WIDTH MALLOC_ALIGNMENT 

#define SMALLBIN_CORRECTION (MALLOC_ALIGNMENT > CHUNK_HDR_SZ) #define MIN_LARGE_SIZE ((NSMALLBINS - SMALLBIN_CORRECTION) * SMALLBIN_WIDTH) 

#define in_smallbin_range(sz) ((unsigned long) (sz) < (unsigned long) MIN_LARGE_SIZE)
```



### malloc_chunk结构体

```C
struct malloc_chunk {
  
INTERNAL_SIZE_T   prev_size; /* Size of previous chunk (if free). */
INTERNAL_SIZE_T   size;    /* Size in bytes, including overhead. */
  
struct malloc_chunk* fd;     /* double links -- used only if free. */
struct malloc_chunk* bk;
  
  /* Only used for large blocks: pointer to next larger size. */
  struct malloc_chunk* fd_nextsize; /* double links -- used only if free. */
  struct malloc_chunk* bk_nextsize;
};
```

- `prev_size`记录物理上前一个`chunk`的大小(包括`chunk`头)
- `size`，该 `chunk` 的大小，大小必须是 `2 * SIZE_SZ` 的整数倍，代码里有大小强制对齐的
- `fd` 指向下一个（非物理相邻）空闲的 `chunk`
- `bk` 指向上一个（非物理相邻）空闲的 `chunk`
- `fd_nextsize` 指向前一个与当前 chunk 大小不同的第一个空闲块，不包含 `bin` 的头指针，只有 chunk 空闲的时候才使用
- `bk_nextsize` 指向后一个与当前 chunk 大小不同的第一个空闲块，不包含 `bin` 的头指针，只有 chunk 空闲的时候才使用





## ptmalloc

Bins为一个**单向或者双向链表**，存放着**空闲的chunk**（freed chunk）。glibc为了让malloc可以更快找到合适大小的chunk，因此在free掉一个chunk时，**会把该chunk根据大小加入合适的bin**中。

内存池分块由一个**malloc_chunk*类型的Bins数组**存储；

chunk最小为（size_t * 4），size_t = unsigned long int。Chunk由两部分组成，头部header（pre_size+size）和数据部分user data。如果该chunk被free，就会将chunk加入到名为bin的malloc_chunk*类型linked list。

按使用状态通常可分为**allocated chunk、free chunk、 top chunk 和last remainder chunk四种**。按大小可分为**fast、small、large和tcache chunk**

### fast bin

单链表，存储在**类型为malloc_state的fastbinsY链表**里，无论多少位平台都为10个，但是使用多少个跟global_max_fast变量的值有关，默认7个。

默认情况下，对于 SIZE_SZ 为 4B 的平台， 小于 64B 的 chunk 分配请求，对于 SIZE_SZ 为 8B 的平台，小于 128B 的 chunk 分配请求，首先会查找fast bins中是否有所需大小的chunk存在（精确匹配），如果存在，就直接返回。

Fast bin的chunk 大小（含 chunk 头部）：0x10-0x40（64 位 0x20-0x80）B，相邻 bin 存放的大小相差 0x8（0x10）B。

总结以下特点：

- fastbinsY[]，fast bin存放在此数组中；
- 单向链表；
- LIFO（last in first out，后入先出，当下次malloc大小与这次free大小相同时，会从相同的bin取出，也就是会取到相同位置的chunk）；
- 管理 16、24、32、40、48、56、64 Bytes 的 free chunks（默认7个），最多加上 72、80 Bytes；
- 其中的chunk的in_use位（下一个物理相邻的chunk的P位）总为1。也就是说，释放到fastbin的chunk不会被清除in_use标志位。



### small bin

双向链表，62个small bins，存储在bins[2]至bins[63]，同一个 small bin 链表中的 chunk 的大小相同。两个相邻索引的 small bin 链表中的 chunk 大小相差的字节数为 2 个机器字长，即 32 位相差 8 字节（4字节`fd` + 4字节`bk`），64 位相差 16 字节（8字节`fd` + 8字节`bk`）。

chunk size小于0x200（64位下0x400）字节的chunk叫做small chunk。

Chunk大小同样是从16字节（32字节）开始每次+8字节（16字节），最大16 + 8 * 61 = 504字节（32 + 16 * 61 = 1008字节）。

chunk size小于0x200（64位下0x400）字节（512或1024）的chunk叫做small chunk，而small bins存放的就是这些small chunk。Chunk大小同样是从16字节开始每次+8字节。

释放非 fast chunk 时，按以下步骤执行：

1. 若前一个相邻chunk空闲，则合并，触发对前一个相邻 chunk的unlink操作
2. 若下一个相邻chunk是top chunk，则合并并结束；否则继续执行 3
3. 若下一个相邻 chunk 空闲，则合并，触发对下一个相邻chunk的unlink 操作；否则，设置下一个相邻 chunk 的 PREV_INUSE 为 0
4. 将现在的chunk插入unsorted bin。
5. 若size超过了FASTBIN_CONSOLIDATION_THRESHOLD，则尽可能地合并 fastbin中的chunk，放入unsorted bin。若top chunk大小超过了 mp_.trim_threshold，则归还部分内存给 OS。

总结有以下特点。

- 双向循环链表
- Chunk size < 0x400 byte（64位）
- FIFO（先入先出）
- 根据大小再分成62个大小不同的bin
  - 0x20,0x30…0x60,0x70…



### large bin

双向链表，63个large bins，存储在bins[64]至bins[126]，large bins 中的每一个 bin 都包含一定范围内的 chunk，其中的 chunk 按 fd 指针的顺序从大到小排列。相同大小的 chunk 同样按照最近使用顺序排列。

Large bins存放的是大于等于0x200（64位下0x400）字节的chunk。

bins[64]至bins[126]存放的字节大小递增：

- 前32个bins：从128 * SIZE_SZ 字节开始每次 + 16 * SIZE_SZ 字节
- 接下来的16个bins：每次+ 32 * SIZE_SZ 字节
- 接下来的8个bins：每次+ 64 * SIZE_SZ 字节
- 接下来的4个bins：每次 + 128 * SIZE_SZ 字节
- 接下来的2个bins：每次 + 256 * SIZE_SZ 字节
- 最后的1个bin：只有一个chunk，大小和large bins剩余的大小相同

同一个bin中的chunks不是相同大小的，按大小降序排列。

总结：

- 双向循环链表（每个节点降序排序）
- Chunk size >= 0x400
- Freed chunk多两个指针fd_nextsize、bk_nextsize指向前一块和后一块large chunk
- 根据大小再分成63个bin但大小不再是固定大小增加
  - 前32个bin为0x400+0x40*i
  - 32~48bin为0x1380+0x200*i
  - …以此类推
- 不再是每个bin中的chunk大小都固定，每个bin中存着该范围内不同大小的bin并在过程中进行排序用来加快寻找的速度，大的chunk会放在前面，小的chunk会放在后面
- FIFO（先入先出）



### unsorted bin

双向链表，1个unsorted bin，存储在bins[1]，这里面的 chunk 没有进行排序，存储的 chunk 比较杂。

大小超过fast bins阈值（global_max_fast）的chunk被释放时会加入到unsorted bin，这使得ptmalloc2可以复用最近释放的chunk，从而提升效率。

分配时，如果在 unsorted bin 中没有合适的 chunk，就会把 unsorted bin 中的所有 chunk分别加入到所属的 bin 中，然后再在 bin 中分配合适的 chunk。

当程序申请大于global_max_fast内存时，glibc会遍历unsorted bin，每次取最后的一个unsorted bin。

1. 如果 unsorted chunk 满足以下四个条件，它就会被切割为一块满足申请大小的 chunk 和另一块剩下的 chunk，前者返回给程序，后者重新回到 unsorted bin。
   - 申请大小属于 small bin 范围
   - unosrted bin 中只有该 chunk
   - 这个 chunk 同样也是 last remainder chunk（N-K 字节的 chunk）
   - 切割之后的大小依然可以作为一个 chunk
2. 否则，从 unsorted bin 中删除 unsorted chunk。
   - 若 unsorted chunk 恰好和申请大小相同，则直接返回这个 chunk
   - 若 unsorted chunk 属于 small bin 范围，插入到相应 small bin
   - 若 unsorted chunk 属于 large bin 范围，则跳转到 3
3. 此时 unsorted chunk 属于 large bin 范围
   - 若对应 large bin 为空，直接插入 unsorted chunk，其 fd_nextsize 与 bk_nextsize 指向自身。
   - 否则，跳转到 4
4. 到这一步，我们需按大小降序插入对应 large bin
   - 若对应 large bin 最后一个 chunk 大于 unsorted chunk，则插入到最后
   - 否则，从对应 large bin 第一个 chunk 开始，沿 fd_nextsize（即变小）方向遍历，直到找到一个 chunk 命名为c，其大小小于等于 unsorted chunk 的大小
   - 若c大小等于unsorted chunk大小，则插入到c后面
   - 否则，插入到c前面

直到找到满足要求的unsorted chunk，或无法找到，去top chunk切割为止。总结以下特点。

- 双向循环链表
- 当free的chunk大小大于等于144字节时，为了效率，glibc并不会马上将chunk放到相对应的bin中，而会先放到unsorted bin
- 而下次malloc时将会先找找看unsorted bin中是否有合适的chunk，找不到才会去对应的bin中寻找，此时会顺便把unsorted bin的chunk放到对应的bin中，但small bin除外，为了效率，反而先从small bin找



### Tcache

`glibc`从2.26版本开始引入的一个内存分配优化机制，类似于`fastbin`一样的东西，每条链上**最多可以有7个chunk**，`free`的时候当`tcache`满了才放入`fastbin`或`unsorted bin`，`malloc`的时候优先去`tcache`找。

`tcache`管理的内存块大小范围通常是小于或等于 512 字节（1024 字节）

`tcache`中存在多个`bin`（类似于`fast bin`和`small bin`），每个`bin`用于存储特定大小范围的内存块。`tcache` bin的大小是以`8`字节递增的，从`8`字节（即第一个`bin`）开始到最大允许大小（例如1024字节）。

例如，在64位系统中：

- `tcache` bin 1：存储大小为`8`字节的内存块。
- `tcache` bin 2：存储大小为`16`字节的内存块。
- `tcache` bin 3：存储大小为`24`字节的内存块。
- 依此类推，直到`128`个`bin`，用于存储最大`1024`字节大小的内存块。

基本工作方式：

- malloc 时，会先 malloc 一块内存用来存放 tcache_perthread_struct 。
- free 内存，且 size 小于 small bin size 时
  - 先放到对应的 tcache 中，直到 tcache 被填满（默认是 7 个）
  - tcache 被填满之后，再次 free 的内存和之前一样被放到 fastbin 或者 unsorted bin 中
  - tcache 中的 chunk 不会合并（不取消 inuse bit）
- malloc 内存，且 size 在 tcache 范围内
  - 先从 tcache 取 chunk，直到 tcache 为空
  - tcache 为空后，从 bin 中找
  - tcache 为空时，如果 fastbin/smallbin/unsorted bin 中有 size 符合的 chunk，会先把 fastbin/smallbin/unsorted bin 中的 chunk 放到 tcache 中，直到填满。之后再从 tcache 中取；因此 chunk 在 bin 中和 tcache 中的顺序会反过来。

### 块释放策略

- 当释放一个内存块时，如果其大小小于或等于`tcache`的最大块大小，它将首先被放入`tcache`中。如果`tcache`已满（默认每个`bin`最多缓存`7`个块），块将被放入`fast bin`、`small bin`或`unsorted bin`中，具体取决于块的大小和其他因素。



### malloc分配

#### 简单分配流程

主线程的arnea称为“main_arena”。子线程的arnea称为“thread_arena”；主线程无论一开始malloc多少空间，只要size<128KB，kernel都会给132KB的heap segment(rw)。这部分称为main arena。 main_arena 并不在申请的 heap 中，而是一个全局变量，在 libc.so 的数据段。

1. 在第一次malloc的时候，glibc就会将堆切成两块chunk，第一块chunk就是分配出去的chunk，剩下的空间视为top chunk
2. 之后要是分配空间不足时将会由top chunk分配出去，它的size为表示top chunk还剩多少空间

假设 Top chunk 当前大小为 N 字节，用户申请了 K 字节的内存，那么 Top chunk 将被切割为：

- 一个 K 字节的 chunk，分配给用户
- 一个 N-K 字节的 chunk，称为 Last Remainder chunk

后者成为新的 Top chunk。如果连 Top chunk 都不够用了，那么：

- 在 main_arena 中，用 brk() 扩张 Top chunk
- 在 non_main_arena 中，用 mmap() 分配新的堆



#### 复杂分配流程

1. 判断这个`chunk_size`是否`<=global_max_fast`，是则寻找能匹配的`fast bin`，另外如果该`size`的`fast bin`为空，继续往下
2. 判断这个`chunk_size`是否在`small_bin`范围，是则寻找能匹配的`small_bin`，另外如果该`size`的`small_bin`为空，继续往下
3. 如果不在`small_bin`范围，会在`else`合并`fastbin`里空闲的分块到`top chunk`或`unsorted bin`
4. `unsorted bin`则会找第一个能满足的`chunk`并返回或者切割之后返回，`unsorted bin` 中每遍历一个不满足要求的`unsorted bin`就会把该`unsorted bin`加到合适的small bin或者`large bin`当中。如果切割之后剩余的部分<`MINSIZE`，那么则不会切割整个返回。
5. 如果还是找不到，那么就会切割`top_chunk`。如果`top_chunk`都不能满足请求的大小，则会`free` `top_chunk`并再一次向操作系统申请新的`top_chunk`，这次申请同样还是申请一个`0x21000B`的`top_chunk`，通常情况下旧的`top_chunk`和新申请的`top_chunk`物理相邻，那么如果`free` 旧的`top_chunk`进入了一个非`fast bin`的链当中，就会被新的`top_chunk`合并。
6. 如果一次申请的内存超过`0x200000B`，那么就不会在heap段上分配内存，将会使用`mmap`在`libc`的`data`段分配内存。通常利用就是每次分配给分配地址，分配`size`没限制那就`malloc`一个很大的内存就可以直接泄露`libc`的地址。 



### malloc释放

1. 根据传入地址减去偏移`CHUNK_HDR_SZ`，得到内存块结构体地址
2. 根据内存块的`IS_MMAPPED`比特位判断是否`mmap`分配，是则使用munmap释放掉，否继续往下
3. 得到内存块是在哪个分配区分配的，返回该分配区
4. 进入`_int_free`函数
5. 判断是否在`fast_bin`范围，是则上锁插入到`fast_bin`里对应大小的链表尾部(链表一直存储尾部)，然后返回
6. 判断上一个内存块是否空闲，是则与当前内存块合并
7. 如果下一个内存块是`top chunk`，将这个合并后的内存块合并到`top chunk`里，如果不是判断是否空闲，是合并，然后放入`unsorted bin`
8. 如果最终内存块的大小大于`FASTBIN_CONSOLIDATION_THRESHOLD`，执行`malloc_consolidate`函数合并`fast_bin`里的内存块到`unsorted_bin`
9. 如果`top chunk`大小大于收缩阈值`trim_threshold`，收缩`top chunk`



## tcmalloc



