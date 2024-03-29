[TOC]

#  Redis数据结构

##  1、简单动态字符串

```c++
struct sdshdr {
	int len;
	int free;
	char buf[];
}
```

​	***len***记录了buf数组中已经使用的字节的数量，等于sds所保存字符串的长度；***free***记录了buf数组中未使用字节的数量；***buf***数组用于保存字符串。

​	sds与c语言的字符串相比，在安全性、效率和功能方面更有优势：

- **获取字符串长度为常数复杂度**

  c语言的字符串数据结构不会记录字符串长度，要获取字符串长度需要遍历一遍buf数组。

- **杜绝缓冲区溢出**

  c语言的strcat函数在源字符串空间不足时出现缓冲区溢出，而sds会在原字符串空间不足时自动扩容。

- **减少修改字符串时带来的内存分配次数**

  - [ ] *空间预分配*

    空间预分配用于优化字符串的增长——当对sds修改后空间不足时：若sds的len属性小于1MB，则程序会分配和len属性同样大小的空闲空间；若sds的len属性大于1MB，则程序会分配1MB的空闲空间。

  - [ ] *惰性释放*

    惰性释放用于优化字符串的缩短——当对sds进行缩短操作后，程序并不会立即使用内存重分配来回收缩短后多出来的字节，而是用free属性记录下来，以供将来使用。

- **二进制安全**

  c字符串以空字符串结尾，不能用于保存视频、音频、图片、压缩文件等二进制文件；而sds使用len而不是空字符串作为结尾，数据在写入时是什么样的，他被读取时就是什么样的。

- **兼容了部分c字符串函数**

  sds遵循了c字符串以空字符串结尾的规则，使得sds可以重用一部分c字符串函数库。

  

##  2、链表

```c++
struct listNode {
	listNode *prev;
	listNode *next;
	void *value;
}
```

```c++
struct list {
	listNode *head;
	listNode tail;
	unsigned long len;
	...
}
```

​	list结构为链表提供了表头指针***head***，表尾指针***tail***，链表长度计数器***len***，而***dup***、***free***和***match***的成员函数则是用于实现多态链表所需的类型特定函数。

​	redis的链表结构特点如下：

- **双向**
- **无环**
- **带表头和表尾指针**
- **带链表长度计数器**
- **多态，可用于保存何种不同类型的值**



##  3、字典

```c++
struct dictht {
	dictEntry **table;
	unsigned long size;
	unsigned long sizemask;
	unsigned long used;
}
```

​	***table***是一个dictEntry类型的数组；***size***是hash表的大小（即table的长度）；***sizemask***是哈希表大小掩码，用于计算索引值；***used***是hash表中已有键值对数量。

```c++
struct dictEntry {
	void *key;
	union {
		void *val;
		uint_64_t u64;
		int64_t s64;
	} v;
	dictEntry *next; 
}
```

​	***key***是键值对的键；***v***是key对应的键值对的值，其中的v可以是一个指针、或者是一个unit_64_t整数，还可以是一个int64_t整数；***next***是指向下一个键值对节点的指针，采用链地址法解决hash冲突。

```c++
strcut dict {
	dictType *type;
	void *privdata;
	dictht ht[2];
	int rehashidx;		
}
```

​	***type***是指向dictType结构的指针，用于为不同类型的字典设置不用的类型特定函数；***privatedata***则保存了需要传给类型特定函数的可选参数；***ht***是一个包含两个项的数组，数组中的每个元素都是一个dictht哈希表，一般情况下，dict只会使用ht[0]，只有在rehash时会使用ht[1]；***rehashidx***记录了当前rehash的进度，当前没有进行rehash时值为-1。

​	redis使用MurmurHash2算法计算键的hash值，这种算法的好处是即使输入的键是有规律的，算法也能给出一个很好的随机分布性。

​	为了让hash表的负载因子维持在一个合理的范围内，当hash表的键值对数量过大或过小时，程序会对hash表的大小进行相应的扩大或缩小操作。

​	***loadfactor=h[0].used / size***

- [ ] **rehash**

  - 为字典的ht[1]分配空间：若执行的是扩大操作，则ht[1]的大小为第一个大于等于ht[0].used*2的2的n次方幂；若执行的是收缩操作，则ht[1]的大小为第一个大于等于ht[0].used的2的n次方幂。
  - 将保存在ht[0]上的键值对rehash到ht[1]上：重新计算键的hash值和索引值，把键值对放在ht[1]对应的位置上。
  - 当ht[0]上所有的键值对全部迁移到ht[1]上之后，释放ht[0]，将ht[1]设置为ht[0]，并在ht[1]上创建一个新的空表，以供下次rehash。

- [ ] **渐进式 rehash**

  hash表的rehash并不是一次性、集中性地完成，而是分多次、渐进式地完成的。

  - 为ht[1]分配空间，让字典同时拥有ht[0]和ht[1]
  - 在字典中维护rehashidx，并将值设为0，表示rehash工作开始
  - 在rehash期间，每次对字典的执行增删改查时，除了执行指定的操作外，还顺便将ht[0]hash表在rehashidx索引上的所有值迁移到ht[1]，当rehash完成后将rehashidx值加一
  - 随着操作的不断执行，最后ht[0]上所有的键值对都会被rehash到ht[1]上，这时将rehashidx设置为-1，表示rehash操作完成
  - 在渐进式rehash期间，字典的删改查操作会在两个hash表上进行，新增的键只会被保存到ht[1]上，保证了ht[0]的只减不增，最终会变成一个空表

- [ ] **hash表的扩展与收缩**

  当以下条件中的任意一个满足时，程序会自动执行hash表的扩展操作：

  - 服务器目前没有执行BGSAVE或BGREWRITEAOF命令，且hash表的负载因子大于等于1
  - 服务器当前正在执行BGSAVE或BGREWRITEAOF命令，且hash表的负载因子大于等于5

  当hash表的负载因子小于0.1时，程序会自动执行hash表的收缩操作。

  

##  4、跳跃表

```c++
struct zskiplistNode {
	zskpilistLevel {
		zskiplistNode *forward;
		unsigned int span; 
	} level [];
	zskipListNode *backward;
	double score;
	robj *obj;
}
```



- ***level***数组包含多个元素，每个元素都包含一个指向其他节点的指针用于加快访问其他节点的速度。每次创建一个跳跃表节点时，程序会根据幂次定律随机生成一个1-32之间的值作为level数组的大小，这个大小就是层的“高度”。level数组的每个元素都有一个指向表尾方向元素的指针***forward***，用于从表头向表尾方向访问节点；层的跨度***span***属性用于记录两个节点之间的距离，跨度越大，表示两个节点相距越远，指向null的所有前进指针的跨度都是0。遍历操作只需要使用forward操作即可完成，与span无关，span实际上是用于计算排位的——在访问某个节点的过程中，将沿途经过的所有层的跨度加起来，就是该节点在跳跃表中的排位。

- ***backward***是节点的后退指针，用于从表尾向表头方向访问节点，每个节点只有一个后退指针，所以每次只能回退到上一个节点。

- ***score***代表节点分值，是一个double类型的浮点数，跳跃表中的所有节点都按照分值从小到大排序。

- ***obj***代表节点的成员对象，是一个指向字符串对象的指针。

  在同一个跳跃表中，各节点保存的成员对象必须是唯一的，但是各节点保存的分值可以是相同的，当分值相同时，节点按照成员对象的自然顺序排序。

```c++
struct zskiplist {
	zskiplistNode *head;
	zskiplistNode *tail;
	unsigned long length;
	int level;
}
```

​	***head***指向跳跃表的表头；***tail***指向跳跃表的表尾；***length***记录跳跃表的长度，即跳跃表目前包含节点的数量（不含头结点）；***level***记录当前跳跃表中节点最大的层数（不含头结点）。



##  5、整数集合

```c++
struct intset {
	uint32_t encoding;
	uint32_t length;
	int8_t contents[];
}
```

- ***encoding***是整数的编码方式

  若encoding的值为INTSET_ENC_INT16，则contents就是一个int16_t类型的数组；若encoding的值为INTSET_ENC_INT32，则contents就是一个int32_t类型的数组；若encoding的值为INTSET_ENC_INT64，则contents就是一个int64_t类型的数组。

- ***length***属性记录了整数集合的大小，即contents数组的长度

- ***contents***数组中的数据项对应了整数集合中的元素

  数组中的各个项按照从小到大的顺序排列，且数组中不含任何重复项。

- [ ] **升级**

  ​	每当我们将一个比集合中所有元素类型都长的元素加入到集合中，整数集合需要先升级才能将元素添加到集合中。引发升级的新元素长度总是比数组中的元素长，所以这个新元素要么大于数组中的所有元素，要么小于数组中的所有元素；因此，该新元素要么位于底层数组的最开头，要么位于底层数组的最末尾。

  ​	升级步骤如下：

  1. 根据新元素类型，扩展整数集合底层数组的空间，并为新元素分配内存空间
  2. 将数组中的元素转换为新元素相同的类型，而后将转换后的元素放置到对应的位置上，在这一过程中保持底层数组的有序性不改变
  3. 将新元素放置到数组对应的位置上

  **注意：**整数集合不支持降级，一旦对数组进行了升级操作，编码就会一直保持升级后的状态。

- [ ] **升级的好处**

- 提升灵活性
- 节约内存



##  6、压缩列表

​	压缩列表是redis为了节约内存设计的，由一系列特殊编码的连续内存块组成的顺序型数据结构。一个压缩列表可以包含任意多个节点，每个节点可以保存一个字节数组或一个整数值。

```c++
struct ziplist {
	uint32_t zlbytes;
	uint32_t zltail;
    uint16_t zllen;
    zlentry *entry;
    uint8_t zlend;
}
```

- ***zlbytes***记录整个压缩列表占用的内存字节数，在对压缩列表进行内存重分配或计算zlend所在位置时使用。
- ***zltail***记录压缩列表的表尾节点距离压缩列表的起始地址有多少字节，通过该偏移量，程序无需遍历整个压缩列表即可确定表尾节点地址。
- ***zllen***记录了压缩列表的节点数量，当值小于UINT_16_MAX(65535)时，该属性即为压缩列表长度；当该值大于65535时，列表实际数量需要遍历列表才能计算出来。
- ***entry***是一个指向列表表头的指针，节点长度由几点保存的内容决定。
- ***zlend***是一个特殊值0xFF（255），用于标记压缩列表的尾端。

```c++
struct zlentry {
	int previous_entry_length;
    int encoding;
    void *content;
}
```

- ***previous_entry_length***属性以字节为单位，记录了压缩列表前一个节点的长度。它可以是一字节或五字节：当前一个节点的长度小于254个字节，则为一字节；当前一个节点的长度大于等于254个字节，则为5字节，其中第一个字节为0xFE(254)，之后的四个字节用于保存前一字节的长度。使用这个属性加上当前字节的起始地址，可以很方便的计算出前一个几点的起始地址。

- ***encoding***属性记录了节点的content属性所保存数据的类型及长度。它可以是一字节、两字节或五字节：当为一字节、两字节或五字节且值的最高位为00、01或10时，表示字节数组编码，即content数组是一个字节数组，数组长度由编码除去最高两位之后的其他位记录；当为一字节且值的最高位为11时，表示content属性保存着整数值，整数值的类型和大小由除去最高两位之后的其他位记录。

- ***content***属性保存了节点的值，只可以是一个字节数组或一个整数值，值的类型和长度由encoding属性决定。

- [ ] **连锁更新**

  ​	当压缩列表中恰好存在多个连续的、长度介于250字节—253字节之间的节点，新增一个大于等于254个字节的节点或删除一个小于254个字节长的节点，就有可能会引发连锁更新，导致压缩列表从表头开始每个节点都需要内存重分配（扩大内存空间）以保存节点数据。但即使出现连锁更新，只要实际更新的节点数量不多，就不会造成性能影响。

  ​	因为连锁更新在最坏情况下需要对压缩列表进行N次空间重新配操作，而每次空间重分配的最坏复杂度为O(N)，所以连锁更新的最坏复杂度为O(N*N).



## 参考文献

redis设计与实现		