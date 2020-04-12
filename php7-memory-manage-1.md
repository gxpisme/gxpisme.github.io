---
title: PHP7 内存管理 （一）
date: 2018-05-24 18:01:03
tags:
    - PHP
categories: PHP
---
<!-- more -->
>用 C 语言编程时，开发者要手工地进行内存管理。
>在用PHP编程时，不用考虑内存的情况。是因为C做了内存管理。

### 三种粒度的内存块
- huge(chunk)  每个chunk的大小为2MB
- large(page)  每个page的大小为4KB
- small(slot)  slot的大小不固定

### huge(chunk)
>大内存(大于2MB) 分配本质上是：分配n个chunk。
>通过`zend_mm_huge_list`管理，大内存之间构成一个单向链表。

```
// 大内存的数据结构  大内存之间构成一个单向链表
struct _zend_mm_huge_list {
    void              *ptr;
    size_t             size;
    zend_mm_huge_list *next; //指针指向下一个
};
```

图解：
![](http://cdn.xpisme.com/201805241849_848.png)

上图中`*ptr`指向的`chunk`具体是什么？

`chunk`是向系统申请、释放内存的最小粒度，chunk之间构建成双向链表。
每个`chunk`的大小为2MB，被分为512个`page`，每个`page`4KB (2MB/512=4KB)。

`chunk`的数据结构
```
struct _zend_mm_chunk {
    zend_mm_heap      *heap;
    zend_mm_chunk     *next;       // 指向下一个chunk
    zend_mm_chunk     *prev;       // 指向上一个chunk
    uint32_t           free_pages; // 当前chunk的剩余可用page数
    uint32_t           free_tail;  
    uint32_t           num; 
    char               reserve[64 - (sizeof(void*) * 3 + sizeof(uint32_t) * 3)]; 
    zend_mm_heap       heap_slot;  
    zend_mm_page_map   free_map;   
    zend_mm_page_info  map[ZEND_MM_PAGES];
};
```


### large(page)
>每个page 4KB

依旧用chunk结构表示
```
struct _zend_mm_chunk {
    zend_mm_heap      *heap;
    zend_mm_chunk     *next;
    zend_mm_chunk     *prev;
    uint32_t           free_pages; // 当前chunk的剩余可用page数
    uint32_t           free_tail;  
    uint32_t           num; 
    char               reserve[64 - (sizeof(void*) * 3 + sizeof(uint32_t) * 3)]; 
    zend_mm_heap       heap_slot;  
    zend_mm_page_map   free_map;   // 标识各page是否已使用的bitmap，总大小512bit。对应page总数，每个page占一个bit位。
    zend_mm_page_info  map[ZEND_MM_PAGES];
};
```

### small(slot)
>slot内存是把若干个page按固定大小分割好的内存块，内存池定义了30种大小的slot内存。
>最小的slot大小为8byte，最大的为3072byte。
>0~7递增8byte， 8byte 16byte 24byte 32byte 40byte 48byte 56byte 64byte
>8~11递增16byte 80byte 96byte 112byte 128byte
>12~15递增32byte 160byte 192byte 224byte 256byte
>16~19递增64byte 320byte 384byte 448byte 512byte
>20~23递增128byte 640byte 768byte 896byte 1024byte
>24~27递增256byte 1280byte 1536byte 1792byte 2048byte
>28~29递增512byte 2560byte 3072byte

slot 0~15各占1个page
![](http://cdn.xpisme.com/201805291058_752.png)


```
Zend/zend_alloc_sizes.h
/* 
    num, size, count, pages 
    四个值的含义依次是：slot编号、slot大小、slot数量、占用page数
*/
#define ZEND_MM_BINS_INFO(_, x, y) \
    _( 0,    8,  512, 1, x, y) \
    _( 1,   16,  256, 1, x, y) \
    _( 2,   24,  170, 1, x, y) \
    _( 3,   32,  128, 1, x, y) \
    _( 4,   40,  102, 1, x, y) \
    _( 5,   48,   85, 1, x, y) \
    _( 6,   56,   73, 1, x, y) \
    _( 7,   64,   64, 1, x, y) \
    _( 8,   80,   51, 1, x, y) \
    _( 9,   96,   42, 1, x, y) \
    _(10,  112,   36, 1, x, y) \
    _(11,  128,   32, 1, x, y) \
    _(12,  160,   25, 1, x, y) \
    _(13,  192,   21, 1, x, y) \
    _(14,  224,   18, 1, x, y) \
    _(15,  256,   16, 1, x, y) \
    _(16,  320,   64, 5, x, y) \   //slot16占 5个page 4096byte * 5 / 320byte = 64个
    _(17,  384,   32, 3, x, y) \   //slot17占 3个page 4096byte * 3 / 384byte = 32个
    _(18,  448,    9, 1, x, y) \   //slot18占 这里不明白为什么只占用1个page
    _(19,  512,    8, 1, x, y) \   //slot19占 1个page 4096byte / 512byte = 8个
    _(20,  640,   32, 5, x, y) \   //slot20占 5个page 4096byte * 5 / 640byte = 32个
    _(21,  768,   16, 3, x, y) \   //slot21占 3个page 4096byte * 3 / 768byte = 16个
    _(22,  896,    9, 2, x, y) \   //slot22占 这里不明白为什么占用2个page
    _(23, 1024,    8, 2, x, y) \   //slot23占 2个page 4096byte *2 / 1024byte = 8个
    _(24, 1280,   16, 5, x, y) \   //slot24占 5个page 4096byte * 5 / 1280byte = 16个
    _(25, 1536,    8, 3, x, y) \   //slot25占 3个page 4096byte * 3 / 1536byte = 8个
    _(26, 1792,   16, 7, x, y) \   //slot26占 7个page 4096byte * 7 / 1792byte = 16个
    _(27, 2048,    8, 4, x, y) \   //slot27占 8个page 4096byte *4 / 2048byte = 8个
    _(28, 2560,    8, 5, x, y) \   //slot28占 5个page 4096byte * 5 / 2560byte = 8个
    _(29, 3072,    4, 3, x, y) //slot29占 3个page 4096byte * 3 / 3072byte = 4个
```


### zend_mm_heap
```
struct _zend_mm_heap {
    ...
    // 小内存分配的可用位置链表，ZEND_MM_BINS等于30，即此数组表示的是各种大小内存对应的链表头部
    zend_mm_free_slot *free_slot[ZEND_MM_BINS]; /* free lists for small sizes */

    // 大内存链表
    zend_mm_huge_list *huge_list;               /* list of huge allocated blocks */

    // 指向chunk链表的头部
    zend_mm_chunk     *main_chunk;
    ...
}
```


图解：
![](http://cdn.xpisme.com/201805291123_247.png)
