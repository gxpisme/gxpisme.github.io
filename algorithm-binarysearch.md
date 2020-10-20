# 二分查找法

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
