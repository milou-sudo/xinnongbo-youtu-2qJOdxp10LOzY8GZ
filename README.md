
在做商城系统中最常见的就是规格，简写pcs。具体来说就是一个商品有多个属性，每个属性有多个规格，这样就形成一些排列组合，做商品库存的时候就要对这些组合进行设定库存和价格。


比如，一台电脑内存有16G,32G和64G的，硬盘有500G和1T的，显卡有集成显卡和独立显卡的，这样的商品在售卖的时候选定不同规格价格不一样，仓库备货的库存当然也不一样。


那么对商城后台而言就要单独设定这些产品的价格，就需要把所有的组合排列出来设定价格。


现在我们已知应该产生这样的排列组合然后设定价格和库存(**这里面的价格和库存只是为了程序举个例子 并不是实际市场上卖这个价格**) 




| 内存 | 硬盘 | 显卡 | 价格 | 库存 |
| --- | --- | --- | --- | --- |
| 16G | 500G | 集成 | 5000 | 100 |
| 16G | 500G | 独立 | 6000 | 200 |
| 16G | 1T | 集成 | 7000 | 220 |
| 16G | 1T | 独立 | 6500 | 120 |
| 32G | 500G | 集成 | 7000 | 110 |
| 32G | 500G | 独立 | 7500 | 200 |
| 32G | 1T | 集成 | 7500 | 300 |
| 32G | 1T | 独立 | 8000 | 200 |
| 64G | 500G | 集成 | 7500 | 134 |
| 64G | 500G | 独立 | 8000 | 347 |
| 64G | 1T | 集成 | 8500 | 258 |
| 64G | 1T | 独立 | 9000 | 35 |


这里一共是12种组合，那如何用程序生成这样的组合呢？


我们已知从数据库读取能够拿到的属性变量如下：




```
<span style="color: rgba(0, 0, 0, 1)"php
$spec_list = [
    'memory' => ['16G', '32G', '64G'],
    'storage' => ['500G', '1T'],
    'graphics' => ['integrated', 'discrete'],
];
```


我们想要的肯定是一个数组，里面有12个元素，每个元素中就是这些规格的组合。


类似于这样的结果




```
[
  ['16G', '500G', '集成'],
  ['16G', '500G', '独立'],
  ['16G', '1T', '集成'],
  ['16G', '1T', '独立'],
  ['32G', '500G', '集成'],
  ['32G', '500G', '独立'],
  ['32G', '1T', '集成'],
  ['32G', '1T', '独立'],
  ['64G', '500G', '集成'],
  ['64G', '500G', '独立'],
  ['64G', '1T', '集成'],
  ['64G', '1T', '独立'],
]
```


那么怎么实现呢？你要是写三层循环，进行嵌套实现，那也没问题，就是这个属性的个数你得明确的知道有多少种，这样毕竟不是长久之计，而且属性有可能增加或者减少，那将来后台增加了属性，程序该怎么办呢？


还是得自动的计算出来所有组合，不需要手动去检测有多少种属性去foreach遍历。


这里我偷懒的让vscode里面的插件 fitten code 帮我写了一个




```
<span style="color: rgba(0, 0, 0, 1)"php
function generateCombinations($attributes) {
    if (count($attributes) === 0) {
        return [[]];
    }
    $firstAttribute = array_shift($attributes);
    $combinationsWithoutFirst = generateCombinations($attributes);
    $combinations = [];
    foreach ($firstAttribute as $value) {
        foreach ($combinationsWithoutFirst as $combination) {
            $combinations[] = array_merge([$value], $combination);
        }
    }
    return $combinations;
}
```


调用一下看看 




```
<span style="color: rgba(0, 0, 0, 1)"php
$spec_list = [
    'memory' => ['16G', '32G', '64G'],
    'storage' => ['500G', '1T'],
    'graphics' => ['集成', '独立'],
];
$combinations = generateCombinations($spec_list);
var_dump($combinations);
```


![](https://img2024.cnblogs.com/blog/704203/202411/704203-20241120163326119-712161032.png)


 从执行结果来看，是没问题的。


果然是AI改变未来啊，这生成速度比我手动敲的快多了。但是我们也能发现，它用了递归，按道理来说，这种普通的规格属性也不会太多，排列组合几十个已经很多了，再多的话，客户端那边显示就不好看了，而且客户操作起来就不方便，那递归的性能也不会太差。


但是咱们是追求极致的人啊，我再写个循环实现的，避免程序效率太低，看看怎么实现呢？


首先得弄个大循环，在里面一直进行循环，然后还得判断这种组合是否有过，层数如何解决呢?


我想可以先实现二层，再实现多层，逐步增加组合列表，话不多说，看代码




```
<span style="color: rgba(0, 0, 0, 1)"php
function get_combine_list($spec_list) {
    if (count($spec_list) <= 1) {
        return $spec_list;
    }
    $result = [];
    $first_item = array_shift($spec_list);
    foreach ($first_item as $first_value) {
        $result[] = [$first_value];
    }
    while ($spec_list) {
        $tmp_result = [];
        $second = array_shift($spec_list);
        foreach ($result as $result_item) {
            foreach ($second as $second_value) {
                $tmp_result[] = array_merge($result_item, [$second_value]);
            }
        }
        $result = $tmp_result;
    }
    return $result;
}
```


好了，同样的代码调用一下




```
<span style="color: rgba(0, 0, 0, 1)"php
$spec_list = [
    'memory' => ['16G', '32G', '64G'],
    'storage' => ['500G', '1T'],
    'graphics' => ['集成', '独立'],
];
$combinations = get_combine_list($spec_list);
```


发现结果是一样的，大功告成！


上面递归的思路是递归发现自己是不是最后一个，不是的话就再递归，最终找到最后一个，形成组合，然后再与倒数第二个组合，再与倒数第三个组合。


这里我的思路是，先拿出第一个自然形成结果，再拿出第二个进行组合，然后拿出第三个跟现有结果进行组合，直到拿出最后一个规格进行组合最终形成结果赋值。


方法没有优劣，只有是不是适合以及你自己能不能接受，能不能维护。


 本博客参考[豆荚加速器](https://baitenghuo.com) 。转载请注明出处！
