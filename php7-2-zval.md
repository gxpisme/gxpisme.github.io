---

title: PHP7.2的变量 zval 结构

date: 2018-04-17 19:42:01

tags:

    - PHP

categories: PHP

---

<!-- more -->

>环境 PHP7.2

>上一节介绍了PHP5.6版本，zval的结构。这节介绍下PHP7.2版本 zval结构。

### _zval_struct 结构

```

typedef struct _zval_struct zval;

struct _zval_struct {

    zend_value        value;            /* value */

    union {

        struct {

            ZEND_ENDIAN_LOHI_4(

                zend_uchar    type,         /* active type */

                zend_uchar    type_flags,   

                zend_uchar    const_flags,

                zend_uchar    reserved)     /* call info for EX(This) */

        } v; 

        uint32_t type_info;

    } u1;

    union {

        uint32_t     next;                 /* hash collision chain */

        uint32_t     cache_slot;           /* literal cache slot */

        uint32_t     lineno;               /* line number (for ast nodes) */

        uint32_t     num_args;             /* arguments number for EX(This) */

        uint32_t     fe_pos;               /* foreach position */

        uint32_t     fe_iter_idx;          /* foreach iterator index */

        uint32_t     access_flags;         /* class constant access flags */

        uint32_t     property_guard;       /* single property guard */

        uint32_t     extra;                /* not further specified */

    } u2;

};

```

- zval 结构内嵌一个union类型的zend_value 下面会有所介绍

- u1 结构

    + 变量的类型就通过`u1.v.type`区分

- u2 结构

    + 假如zval只有:value、u1两个值，整个zval的大小也会对齐到16byte，既然不管有没有u2大小都是16byte，把多余的4byte拿出来用于一些特殊用途还是很划算的，比如next在哈希表解决哈希冲突时会用到，还有fe_pos在foreach会用到......

#### `u1.v.type`值，文件`Zend/zend_types.h`

```

/* regular data types */

#define IS_UNDEF                    0

#define IS_NULL                     1

#define IS_FALSE                    2

#define IS_TRUE                     3

#define IS_LONG                     4

#define IS_DOUBLE                   5

#define IS_STRING                   6

#define IS_ARRAY                    7

#define IS_OBJECT                   8

#define IS_RESOURCE                 9

#define IS_REFERENCE                10

/* constant expressions */

#define IS_CONSTANT                 11

#define IS_CONSTANT_AST             12

/* fake types */

#define _IS_BOOL                    13

#define IS_CALLABLE                 14

#define IS_ITERABLE                 19

#define IS_VOID                     18

/* internal types */

#define IS_INDIRECT                 15

#define IS_PTR                      17

#define _IS_ERROR                   20

```

### zend_value 结构

```

typedef union _zend_value {

    zend_long         lval;             /* long value  int整型*/

    double            dval;             /* double value float型*/

    zend_refcounted  *counted;

    zend_string      *str;              // string 字符串

    zend_array       *arr;              // array  数组

    zend_object      *obj;              // object 对象

    zend_resource    *res;              // resource 资源类型

    zend_reference   *ref;              // 引用类型 通过 &$var_name定义

    zend_ast_ref     *ast;              // 下面几个都是内核使用的value

    zval             *zv; 

    void             *ptr;

    zend_class_entry *ce; 

    zend_function    *func;

    struct {

        uint32_t w1;

        uint32_t w2;

    } ww;

} zend_value;

```

- ** IS_NULL ** `_zval_struct`结构中`u1.v.type`标记为`NULL`类型，`_zvalue_value`不存储。

- ** IS_TRUE ** `_zval_struct`结构中`u1.v.type`标记为布尔true类型，`_zvalue_value`不存储。

- ** IS_FALSE ** `_zval_struct`结构中`u1.v.type`标记为布尔false类型，`_zvalue_value`不存储。

- ** IS_LONG ** `_zval_struct`结构中`u1.v.type`标记为`INT`类型，`_zvalue_value`中`lval`存储，

- ** IS_DOUBLE ** `_zval_struct`结构中`u1.v.type`标记为`DOUBLE`类型，`_zvalue_value`中`dval`存储，

- ** IS_STRING ** `_zval_struct`结构中`u1.v.type`标记为`STRING`类型，`_zvalue_value`中`zend_string`存储，

- ** IS_ARRAY ** `_zval_struct`结构中`u1.v.type`标记为`ARRAY`类型，`_zvalue_value`中`zend_array`存储，

- ** IS_OBJECT ** `_zval_struct`结构中`u1.v.type`标记为`OBJECT`类型，`_zvalue_value`中`zend_object`存储，

- ** IS_RESOURCE ** `_zval_struct`结构中`u1.v.type`标记为`RESOURCE`类型，`_zvalue_value`中`zend_resource`存储，

#### `zend_string` 字符串

```

struct _zend_string {

    zend_refcounted_h gc;               // 变量引用信息，比如当前value的引用数，所有用到引用计数的变量类型都会有这个结构

    zend_ulong        h;                /* hash value 哈希值，数组中计算索引时会用到 */

    size_t            len;              // 字符串长度，通过这个值保持二进制安全

    char              val[1];           // 字符串内容，变长struct，分配时按len长度申请内存

};

```

#### `zend_array` 数组

```

// array是PHP中非常强大的一个数据结构，它的底层实现就是普通有序HashTable

struct _zend_array {

    zend_refcounted_h gc;   // 引用计数，与字符串相同

    union {

        struct {

            ZEND_ENDIAN_LOHI_4(

                zend_uchar    flags,

                zend_uchar    nApplyCount,

                zend_uchar    nIteratorsCount,

                zend_uchar    consistency)

        } v;

        uint32_t flags;

    } u;

    uint32_t          nTableMask;   // 计算bucket索引时的掩码

    Bucket           *arData;       // bucket 数组

    uint32_t          nNumUsed;     // 已用bucket数

    uint32_t          nNumOfElements; // 已有元素数，nNumOfElements <= nNumUsed，因为删除的并不是直接从arData中移除

    uint32_t          nTableSize;     // 数组的大小 为2^n

    uint32_t          nInternalPointer; // 数值索引

    zend_long         nNextFreeElement;

    dtor_func_t       pDestructor;

};

```

#### `zend_object` 对象

```

struct _zend_object {

    zend_refcounted_h gc;

    uint32_t          handle; // TODO: may be removed ???

    zend_class_entry *ce;     // 对象对应的class类

    const zend_object_handlers *handlers;

    HashTable        *properties;   // 对象属性hash表

    zval              properties_table[1];

};

```

#### `zend_resource` 资源

```

// 资源

// tcp链接、文件句柄

struct _zend_resource {

    zend_refcounted_h gc;

    int               handle; // TODO: may be removed ???

    int               type;

    void             *ptr;

};

```

参考资料

https://github.com/pangudashu/php7-internal/blob/master/2/zval.md

http://www.phpinternalsbook.com/php7/internal_types/zvals/basic_structure.html

