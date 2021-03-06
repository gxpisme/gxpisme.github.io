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
![](/image/algorithm-quicksort.gif)
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

## 归并排序
![](/image/algorithm-mergesort.gif)

```php

function mergeSortMain($arr) {
    $len = count($arr);
    if ($len <= 1) {
        return $arr;
    }


    $middle = floor($len / 2);

    $left = array_slice($arr, 0, $middle);
    $right = array_slice($arr, $middle);
    $left = mergeSortMain($left);
    $right = mergeSortMain($right);

    return mergeSort($left, $right);
}


function mergeSort($one, $two) {
    $oneLength = count($one);
    $twoLength = count($two);
    $oneI = 0;
    $twoI = 0;

    $tmp = [];
    while ($oneI < $oneLength && $twoI < $twoLength) {
        $tmp[] = $one[$oneI] > $two[$twoI] ? $one[$oneI++] : $two[$twoI++];
    }

    while ($oneI < $oneLength) {
        $tmp[] = $one[$oneI++];
    }

    while ($twoI < $twoLength) {
        $tmp[] = $two[$twoI++];
    }

    return $tmp;
}
```

## 选择排序

![](/image/algorithm-selectionsort.gif)
```php

function selectionSort($arr) {
    $len = count($arr);
    if ($arr <= 1) {
        return $arr;
    }

    for ($i = 0; $i < $len; $i++) {
        $k = $i;
        for ($j = $i + 1; $j < $len; $j++) {
            if ($arr[$k] < $arr[$j]) {
                $k = $j;
            }
        }
        if ($k != $i) {
            $t = $arr[$k];
            $arr[$k] = $arr[$i];
            $arr[$i] = $t;
        }
    }

    return $arr;
}
```

## 插入排序

![](/image/algothrim-insertionsort.gif)

```php

function insertionSort($arr) {
    $len = count($arr);
    if ($arr <= 1) {
        return $arr;
    }

    for ($i = 1; $i < $len; $i++) {
        $t = $arr[$i];
        $j = $i - 1;

        while ($arr[$j] < $t) {
            $arr[$j+1] = $arr[$j];
            $arr[$j] = $t;
            if ($j-- <= 0) {
                break;
            }
        }
    }

    return $arr;
}
```
