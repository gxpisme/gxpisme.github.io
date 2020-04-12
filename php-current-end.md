---

title: PHP current、end函数遇到的坑

date: 2018-04-16 18:28:15

tags:

    - PHP

categories: PHP

---

<!-- more -->

>环境 PHP7.2

## 背景

一个主任务中有许多子任务，每个子任务都有开始时间、结束时间。我想计算主任务耗费的总时长。代码如下

```

$arrData = [

    [

        'start' => 1,

        'end' => 10,

    ],

    [

        'start' => 12,

        'end' => 20,

    ],

];

echo end($arrData)['end'] - current($arrData)['start'];

```

结果竟然是8，不能理解。

然后逐个打印这两个值。

```

$arrData = [

    [

        'start' => 1,

        'end' => 10,

    ],

    [

        'start' => 12,

        'end' => 20,

    ],

];

echo current($arrData)['start'], PHP_EOL; // 输出为1

echo end($arrData)['end'] , PHP_EOL;      // 输出为20

```

第一个输出为1，第二个输出为20。没问题啊，等等，如果先打印end，再打印current呢。

```

$arrData = [

    [

        'start' => 1,

        'end' => 10,

    ],

    [

        'start' => 12,

        'end' => 20,

    ],

];

echo end($arrData)['end'] , PHP_EOL;      // 输出为20

echo current($arrData)['start'], PHP_EOL; // 输出为12

```

第一个输出20，没问题，第二个输出12，输出12，输出12 .....

## 翻开源码`current`函数

```

ext/standard/array.c

PHP_FUNCTION(current)

{

    HashTable *array;

    zval *entry;

    ZEND_PARSE_PARAMETERS_START(1, 1)

        Z_PARAM_ARRAY_OR_OBJECT_HT(array)

    ZEND_PARSE_PARAMETERS_END();

    // 重点zend_hash_get_current_data()函数， 获取当前指针指向的值

    if ((entry = zend_hash_get_current_data(array)) == NULL) {

        RETURN_FALSE;

    }

    if (Z_TYPE_P(entry) == IS_INDIRECT) {

        entry = Z_INDIRECT_P(entry);

    }

    ZVAL_DEREF(entry);

    ZVAL_COPY(return_value, entry);

}

Zend/zend_hash.h

// 传址，当前的指针

#define zend_hash_get_current_data(ht) \

    zend_hash_get_current_data_ex(ht, &(ht)->nInternalPointer)

Zend/zend_hash.c

// current函数传过来的为ht, &(ht)->nInternalPointer

// 获取ht的偏移量为pos的值

ZEND_API zval* ZEND_FASTCALL zend_hash_get_current_data_ex(HashTable *ht, HashPosition *pos)

{

    uint32_t idx = *pos;

    Bucket *p;

    IS_CONSISTENT(ht);

    if (idx != HT_INVALID_IDX) {

        p = ht->arData + idx;

        return &p->val;

    } else {

        return NULL;

    }

}

```

## 翻开源码`end`函数

```

PHP_FUNCTION(end)

{

    HashTable *array;

    zval *entry;

    ZEND_PARSE_PARAMETERS_START(1, 1)

        Z_PARAM_ARRAY_OR_OBJECT_HT_EX(array, 0, 1)

    ZEND_PARSE_PARAMETERS_END();

    // 重点将指针移到最后一个位置

    zend_hash_internal_pointer_end(array);

    if (USED_RET()) {

        // 重点zend_hash_get_current_data()函数， 获取当前指针指向的值

        if ((entry = zend_hash_get_current_data(array)) == NULL) {

            RETURN_FALSE;

        }

        if (Z_TYPE_P(entry) == IS_INDIRECT) {

            entry = Z_INDIRECT_P(entry);

        }

        ZVAL_DEREF(entry);

        ZVAL_COPY(return_value, entry);

    }

}

// 重点看下这个zend_hash_internal_pointer_end函数

Zend/zend_hash.h

// 指针传址

#define zend_hash_internal_pointer_end(ht) \

    zend_hash_internal_pointer_end_ex(ht, &(ht)->nInternalPointer)

Zend/zend_hash.c

ZEND_API void ZEND_FASTCALL zend_hash_internal_pointer_end_ex(HashTable *ht, HashPosition *pos)

{

    uint32_t idx;

    IS_CONSISTENT(ht);

    HT_ASSERT(ht, &ht->nInternalPointer != pos || GC_REFCOUNT(ht) == 1);

    idx = ht->nNumUsed;

    // 在这里循环 将当前的指针，指向最后一个

    while (idx > 0) {

        idx--;

        if (Z_TYPE(ht->arData[idx].val) != IS_UNDEF) {

            *pos = idx;

            return;

        }

    }

    *pos = HT_INVALID_IDX;

}

```

## 总结

- 先执行current，再执行end，没有问题。

- 先执行end，此时当前指针到了最后。再执行current，得到的值是最后的一个值。

源码注解：https://github.com/gxpisme/read_php/commit/f53beab43abf3549d00252b68e20bfb6ac9cf091

推荐看下PHP5.6的hash table结构。

http://blog.xpisme.com/posts/PHP/2018/04/09/php-hashtable/

推荐看下PHP7.2的hash table结构。

http://blog.xpisme.com/posts/PHP/2018/04/19/php7-hashtable/

