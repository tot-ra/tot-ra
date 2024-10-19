вторник, 7 сентября 2010 г. в 14:35:11

[MongoDB](http://www.mongodb.org/) - очередное веяние моды в веб-разработке, когда хранение данных заранее планируется таким огромным, что необходимо их распределение на несколько серверов с помощью шардирования. Поизучаем же непростой синтаксис агрегирования данных в mongo + php

![](../img/Pasted%20image%2020241020011445.png)

### Уникальное поле  

Итак допустим есть коллекция объектов с повторяющимися полями, а нам хочется вытянуть только уникальные поля. В MySQL это делает DISTINCT(поле), здесь же надо запустить комманду которая вернёт только поле person_id.

```php
$aObjects = $Mongo->command(array(
    'distinct'=>'my_userlog_collection',
    'key'=>'person_id',
    'query'=>array(
        '$or'=>array(
            array('person_id' => new MongoRegex('/'.$sSearch.'.*/i')),
            array('firstname' => new MongoRegex('/.*'.$sSearch.'.*/i')),
            array('lastname' => new MongoRegex('/.*'.$sSearch.'.*/i')),
        )
    ),
));
```

---

Как можно заметить одного лишь поля порой недостаточно - зачастую хочется сделать GROUP BY, т.е. повторяющиеся по заданным полям ряды склеиваются (а в случае различия других полей берётся первый из подходящих). В mongo такую пост-обработку результатов производят запускаемые **пользовательские функции** map, reduce и finalize. Я сначала было подумал что в адаптер Mongo надо передавать имя php-функции и даже начал смотреть синтаксис анонимных функций, но потом оказалось что передаются вовсе не функции а банально **текст с javascript-функцией**  

### Нанести на карту и уменьшить  

Функция map() это своего рода SELECT - надо вызвать функцию emit() у которой первый параметр - ключ, а второй - собственно нужные поля. Конечно можно вместо полей поставить this, по аналогии со звёздочкой в SQL, но мы же знаем что это плохая практика. Но я как плохой, возьму все данные.  

Теперь когда mongos, будет получать от mongod данные, он будет группировать их по первому ключу и передавать в reduce() функцию. Очень много примеров зачем-то делают там циклы и переменные.. Мне достаточно взять самый первый подходящий объект.

В коде выше я для полноты картины добавил регулярный поиск для имитации LIKE %, так что тут не буду повторяться. Поскольку возвращаемый адаптером результат вовсе не список, а какая-то левая статистика, то приходится делать что-то типа под-запроса. Вот тут уже реальные данные, но они в формате array (id, value), что помоему тоже извращение, а поскольку в map я сделал SELECT *, то тут я для наглядности показываю ещё и пост-фильтрацию. В итоге что-бы сгруппировать по person_id надо создать такого монстра..

```php
$aMyCollectionResult = $Mongo->command(array(
    "mapreduce" => 'my_userlog_collection',
    "map"       => new MongoCode("function() { emit(this.person_id,this); }"), //SELECT
    "reduce" => new MongoCode("function(k, vals) { return vals[0]; }"), //GROUP
    "query" => array() //сами поставьте сюда WHERE
));

$aList = $Mongo->oMongo->selectCollection($aMyCollectionResult['result'])->find(
    array(),array("value.person_id",'value.first_name','value.last_name','value.country_code')
); //

print_r(iterator_to_array($aList)); //распечатаем что-бы посмотреть
```

По теме читайте..

- [MongoDB Aggregation III: Map-Reduce Basics](http://kylebanker.com/blog/2009/12/mongodb-map-reduce-basics/)
- [Counting Unique Items with Map-Reduce](http://cookbook.mongodb.org/patterns/unique_items_map_reduce/)
- [Stackoverflow: How to use map/reduce to handle more than 10000 unique keys for grouping in MongoDB?](http://stackoverflow.com/questions/2599665/how-to-use-map-reduce-to-handle-more-than-10000-unique-keys-for-grouping-in-mongo/2608439#2608439)