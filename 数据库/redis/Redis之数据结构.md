# Redis之数据结构



## 字符串

Redis字符串类型数据结构如下，简称为SDS

```c
struct sdshdr {
  int len;
  int free;
  char buf[];
}
```

free记录了buf数组中未使用字节的数量；

len记录了buf数组中已使用字节的数量，等于SDS所保存字符串的长度；

buf：字节数组，用于保存字符串；



SDS遵循C语言字符串规范，以空字符作为字符串结尾符号，该空字符不计算中SDS中的len属性；SDS在Redis可以作为一个key，还被用作缓冲区buffer【AOF模块中的AOF缓冲区，客户端状态的输入缓冲区】



为什么定义SDS，不直接使用C语言中的字符串？

- 获取字符串长度，C字符串需要O(N)获取字符串长度，而SDS只需要O(1)；
- 杜绝缓冲区溢出，C字符串当分配的内存不足时，进行更新操作，将会出现空间被修改，SDS空间分配策略杜绝了发生的可能；
- 减少修改字符串带来的内存重分配次数
  - 空间预分配，优化SDS的字符串增长操作；
    - 当需要对SDS进行空间扩展的时候，程序不仅会为SDS分配修改所需的空间，还会为SDS分配额外的未使用空间；
    - 如果SDS的len长度小于1MB，程序分配和len属性同样大小的未使用空间，这时的SDS的len属性和free属性相等；
    - 如果SDS的长度大于等于1MB，那么程序会分配1MB的未使用空间；
  - 惰性空间释放，优化SDS的字符串缩短操作；
    - 当需要缩短SDS保存的字符串，程序并不立即使用内存重分配来回收缩短后多出来的字节，而是使用free属性记录这些多出来的字节数，以备使用；
- 二进制安全
  - C语言字符串必须符合某种编码，并且初了字符串末尾，字符串中不能包含空字符，被程序认为是字符串结尾，限制了C字符串只能保存文本数据，而不能保存图像、视频、压缩文件等二进制数据；
  - 所有SDS API都会以处理二进制的方式来处理SDS存放在buf数组，程序不会对其中的数据做任何限制、过滤和假设；





## 链表

链表在Redis中应用，例如实现Redis中列表键、发布与订阅、慢查询、监视器等；



```c
typedef struct listNode {
  struct listNode *prev;
  struct listNode *next;
  void *value;
}listNode;
```

adlist.h/list和Java中的LinkedList差不多；

Redis的链表的特性：

- 双端：链表节点带有prev和next指针，获取某个节点的前置节点和后置节点的负责度都是O(1)；
- 无环：表头节点的prev指针和表尾节点的next指针都指向NULL，对链表的访问以NULL为终点；
- 带链表长度计数器：adlist.h/list结构len属性对链表节点进行计数，程序获取链表中节点数量的复杂度为O(1)；
- 多态：链表节点使用void*指针来保存节点值，并且通过list结构的dup、free、match三个成员函数为节点值设置类型特定函数，所以链表可以保存各种不同类型的值；



## 字典

Redis的字典使用哈希表作为底层实现，一个哈希表里面包括多个哈希表节点，每个哈希表节点就保存了字典中的一个键值对；

### 哈希表

Redis字典使用的哈希表由dict.h/dictht结构定义；

```c++
typedef struct dictht {
  dictEntry **table;
  unsigned long size;
  unsigned long sizemask;
  unsignedlong used;
} dictht;
```

table是一个数组，数组中的每个元素都是一个指向dict.h/dictEntry结构的指针，每个dictEntry结构保存着一个键值对；

size记录了哈希表的大小，也即是table数组的大小；

used记录哈希表目前已有节点的数量；

sizemask的值总是等于size-1，该值和哈希值一起决定一个key应该放到table数组的那个索引；

### 哈希表节点

哈希表节点使用dictEntry结构表示，每个dictEntry结构都保存着一个键值对；

```c++
typedef struct dictEntry {
  void *key;
  union {
    void *val;
    uint64_tu64;
    int64_ts64;
  } v;
  struct dictEntry *next;
} dictEntry;
```

key属性保存着键值对中的键，而v属性保存着键值对中的值，其中值可以是一个指针、uint64_t整数、int64_t整数；

next是指向另一个哈希表节点的指针，这个指针可以将多个哈希表相同的键值对连接在一起，来解决键冲突的问题；

### 字典

Redis中的字典由dict.h/dict结构表示；

```c++
typedef struct dict {
  dictType *type;
  void *privdata;
  dictht ht[2];
  in trehashidx;
}
```

type是一个指向dictType结构的指针，每个dictType结构保存了用于操作特定键值对的函数；

privdata保存了需要传给那些类型特定函数的可选参数；

ht是一个包含两项的dictht哈希表，字典只使用ht[0]，ht[1]哈希表只会在对ht[0]哈希表进行rehash时使用；

trehashidx记录了rehash目前的进度，如果目前没有在进行rehash，值为-1；