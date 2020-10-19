# 排序

## 冒泡排序
```php
function bubbleSort($arr) {
    $len = count($arr);

    $res = [];
    for ($i = 0; $i < $len; $i++) {
        $max = $arr[$i];
        for ($j = $i + 1; $j < $len; $j++) {
            if ($max < $arr[$j]) {
                $max = $arr[$j];
                $arr[$j] = $arr[$i];
                $arr[$i] = $max;
            }
        }

    }

    return $arr;
}
```

## 快速排序
```php
function quickSort($arr) {
    $len = count($arr);
    $middle = $arr[0];
    $left = [];
    $right = [];

    for ($i = 1; $i < $len; $i++) {
        if ($middle < $arr[$i]) {
            $left[] = $arr[$i];
        } else {
            $right[] = $arr[$i];
        }
    }

    if (count($left) > 1) {
        $left = quickSort($left);
    }

    if (count($right) > 1) {
        $right = quickSort($right);
    }

    return array_merge($left, [$middle], $right);
}
```

