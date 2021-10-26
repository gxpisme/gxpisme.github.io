# 敏感词树 DFA(Deterministic Finite Automaton)


## 构建敏感词树
```
$obj = new DFA();
$obj->addKeyWord('王八蛋');
$obj->addKeyWord('王八羔子');
$obj->addKeyWord('香烟');
$obj->addKeyWord('狗儿子');
$obj->getHashMap();


Array
(
    [王] => Array
        (
            [end] => 0
            [八] => Array
                (
                    [end] => 0
                    [蛋] => Array
                        (
                            [end] => 1
                        )

                    [羔] => Array
                        (
                            [end] => 0
                            [子] => Array
                                (
                                    [end] => 1
                                )

                        )

                )

        )

    [香] => Array
        (
            [end] => 0
            [烟] => Array
                (
                    [end] => 1
                )

        )

    [狗] => Array
        (
            [end] => 0
            [儿] => Array
                (
                    [end] => 0
                    [子] => Array
                        (
                            [end] => 1
                        )

                )

        )

)
```

## 判断敏感词是否是敏感词树
```
var_dump($obj->searchKey('王八蛋'));
var_dump($obj->searchKey('王八'));
```

## DFA 类
```
class DFA
{
    private $arrHashMap = [];

    public function getHashMap() {
        print_r($this->arrHashMap);
    }

    /**
     * 构建敏感词树
     */
    public function addKeyWord($strWord) {
        $len = mb_strlen($strWord, 'UTF-8');

        // 传址
        $arrHashMap = &$this->arrHashMap;
        for ($i=0; $i < $len; $i++) {
            $word = mb_substr($strWord, $i, 1, 'UTF-8');
            // 已存在
            if (isset($arrHashMap[$word])) {
                if ($i == ($len - 1)) {
                    $arrHashMap[$word]['end'] = 1;
                }
            } else {
                // 不存在
                if ($i == ($len - 1)) {
                    $arrHashMap[$word] = [];
                    $arrHashMap[$word]['end'] = 1;
                } else {
                    $arrHashMap[$word] = [];
                    $arrHashMap[$word]['end'] = 0;
                }
            }
            // 传址
            $arrHashMap = &$arrHashMap[$word];
        }
    }

    /**
     * 判断该词是否是敏感词
     */
    public function searchKey($strWord) {
        $len = mb_strlen($strWord, 'UTF-8');
        $arrHashMap = $this->arrHashMap;
        for ($i=0; $i < $len; $i++) {
            $word = mb_substr($strWord, $i, 1, 'UTF-8');
            if (!isset($arrHashMap[$word])) {
                // reset hashmap
                $arrHashMap = $this->arrHashMap;
                continue;
            }
            if ($arrHashMap[$word]['end']) {
                return true;
            }
            $arrHashMap = $arrHashMap[$word];
        }
        return false;
    }
}
```
