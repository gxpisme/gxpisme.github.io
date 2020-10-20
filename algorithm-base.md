# 基础的算法

## 二分查找法
```php
function binarySearch($arr, $key) {
    $len = count($arr);
    $min = 0;
    $max = $len - 1;

    while ($min < $max) {
        $middle = floor(($max + $min) / 2);
        if ($key == $arr[$middle]) {
            return $middle;
        }
        // 要搜索的值 小于 中间值 则最小范围不变 最大范围调整到中间
        if ($key < $arr[$middle]) {
            $max = $middle;
        }

        // 要搜索的值 大于 中间值 则最大范围不变 最小范围调整到中间
        if ($key > $arr[$middle]) {
            $min = $middle;
        }
    }

    return -1;
}
```

## 单链表
### 单链表翻转
```
    // 通过迭代来翻转单链表
    public static Node reverseNodeByIteration(Node head) {
        Node pre = null;
        Node next = null;

        while (head != null) {
            next = head.getNext();

            head.setNext(pre);
            pre = head;

            head = next;
        }

        return pre;
    }

    // 通过递归翻转单链表
    public static Node reverseNodeByRecursive(Node head) {
        if (head == null || head.getNext() == null) {
            return head;
        }
        Node newHead = reverseNodeByRecursive(head.getNext());

        Node temp = head.getNext();
        temp.setNext(head);

        head.setNext(null);
        return newHead;
    }
```

参考文档：[](https://www.cnblogs.com/keeya/p/9218352.html)

### 单链表是否有环

## 树
### 广度优先遍历
### 深度优先遍历

