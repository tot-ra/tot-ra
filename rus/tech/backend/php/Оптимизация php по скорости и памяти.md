четверг, 13 сентября 2007 г. в 20:07:32

До оптимизации быстродействия php-кода я настоятельно рекомендую сначала оптимизировать http-запросы, уменьшить размер js и css, продумать кэширование темплейтов, оптимизировать запросы БД и только тогда заниматься проверкой кода на производительность. Однако стиль программирования надо выбирать до выполнения самой работы и смотреть на ниже перечисленные пункты с точки зрения безопасности - лучше знать о подводных камнях заранее, чем латать дыры потом.

|   |   |   |
|---|---|---|
|Плохо|Хорошо|Разница|
|$a="text $b";|$a='text '.$b;|50%|
|eregi("(ма[a-zа-я]{1,20})",$text);|preg_match("/(ма[a-zа-я]{1,20})/im",$text);|76%|
|$test[a][b]=1;|$text['a']['b']=2;|361%|
|foreach($test as $n)|$it=0; while($it<100000)|254%|
|while (list($k, $v) = each($test))|foreach($test as $k=>$v)|22%|
|substr_compare("abcd","ab",0,2)|strpos("abcd","ab")===0|22%|

Читайте так же:

- [php.spb.ru](http://php.spb.ru/php/speed.html)
- [reinholdweber.com](http://reinholdweber.com/?p=3)
- [Евгений Степанищев - о substr](http://bolknote.ru/2008/07/21/~1790)

Поиск прожорливых мест в памяти можно грубо делать так:

```php
function array_size($arr) {  
    ob_start();  
    print_r($arr);  
    $mem = ob_get_contents();  
    ob_end_clean();  
    $mem = preg_replace("/\n +/", "", $mem);  
    $mem = strlen($mem);  
    return $mem;  
}  
  
foreach($GLOBALS as $key=>$value){  
    $memEstimate = array_size($value);  
    echo($key.':'.$memEstimate);  
}
