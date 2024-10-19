четверг, 5 июля 2012 г. в 10:35:51

**Профилирование** это анализ потребления ресурсов при работе программы. Этимология слова видимо связана с тем что профиль это некая граница, отсюда - поиск границ компонентов ПО в ресурсном пространстве. В моём случае программа это исполняемые php-скрипты, а ресурсы это время, память и нагрузка на процессор. Время исполнения не всегда связано с _нагрузкой процессора_. Процесс может ждать IO-ответа от более медленных источников, а может и просто спать.

Я писал про [микрооптимизацию в php](http://kurapov.name/rus/technology/web/php/php_speed/). Как правило отмечали критики, в общем случае она не нужна. Профилирование это взгляд с высоты птичьего полёта на подключаемый код. Он делается только на рабочей машинке, а не на production. В некоторых LAMP-сборках инструменты для этого уже установлены.

### XDebug

[XDebug](http://xdebug.org/docs/profiler) - это php модуль, я уже давно с ним работаю как с дебаггером.

```
zend_extension = xdebug.so
xdebug.profiler_enable = 1
xdebug.profiler_aggregate = On
xdebug.profiler_output_dir = /tmp
```

Он позволяет генерировать доклад о профилировании в форме файла. Из-за глубокой иерархичности, доклад обычно весит несколько мегабайт. Настройки модуля хранятся в php.ini, там же можно и указать путь к результату профилирования. Назовём его cachegrind.out - он совместим с более общим [форматом valgrind](http://valgrind.org/)

Можно сделать так что-бы он генерировался постоянно, можно - с помощью куки-триггеров. Файл можно делать на каждый отдельный системный процесс, на отдельный URL запрос, по времени исполнения или [их комбинаций](http://xdebug.org/docs/all_settings#trace_output_name). Дальше его стоит связать с одним из анализаторов:

1. PHPStorm (Tools - Analyze stacktrace)
2. [webgrind](http://code.google.com/p/webgrind/)  
    
3. [WinCacheGrind](http://sourceforge.net/projects/wincachegrind/)  
    
4. [KCacheGrind](http://kcachegrind.sourceforge.net/cgi-bin/show.cgi) 

### XHProf

![Screen Shot 2012-07-06 at 11.31.19.png](https://s3-eu-west-1.amazonaws.com/kurapov/image/thumb/2131.png "Screen Shot 2012-07-06 at 11.31.19.png")

[XHProf](https://github.com/facebook/xhprof/) - пакет [PECL](http://pecl.php.net/package/xhprof), написан в Facebook, поставляется тоже как модуль (xhprof.so или xhprof.dll). Позволяет учитывать значительно больше параметров, в т.ч. нагрузку на CPU и оперативную память. Позволяет сравнивать повторные запуски и агрегировать их результаты. Наиболее полезная фича - суммарный анализ по времени в виде весового графа. Граф генерируется с помощью **graphviz** и утилитки **dot**. У меня были небольшие проблемы под маком, пришлось [поставить вручную](http://www.ryandesign.com/graphviz/).

`pecl install xhprof-0.9.2`

Редактируем php.ini:

```
 extension=xhprof.so
 xhprof.output_dir=/var/tmp/xhprof 
```

Поскольку просмотр результатов xhprof целиком web-based, то надо залинковать где-то в корне htdocs путь к xhprof_html папке. Для подключения, необходимо изменять свои скрипты. Т.е. где-то в начале своего index.php ставится include, инициализация и что-бы не бояться за какой-то exit, хук на register_shutdown_function. Вместо того что-бы инклудить каждый раз, можно воспользоваться php-директивой **auto_prepend_file** и включать это файл всегда

```php
//результат будет виден с ?xhprof=1
if (extension_loaded('xhprof') && $_GET['xhprof']) {
    include_once 'xhprof_lib/utils/xhprof_lib.php';
    include_once 'xhprof_lib/utils/xhprof_runs.php';
    xhprof_enable(XHPROF_FLAGS_CPU + XHPROF_FLAGS_MEMORY);
    
    function end_xhprof(){
        $profiler_namespace = 'myapp'; // namespace вашего приложения
        $xhprof_data = xhprof_disable();
        $xhprof_runs = new XHProfRuns_Default();
        $run_id = $xhprof_runs->save_run($xhprof_data, $profiler_namespace);
        
        //Замените относительный путь к залинкованной папке
        $profiler_url = sprintf('http://' . $_SERVER['SERVER_NAME'] . '/xhprof/index.php?run=%s&source=%s', $run_id, $profiler_namespace);

         echo 'XHProfiler output';
     }
    
     register_shutdown_function('end_xhprof');
 }
```

Можно конечно не делать echo ссылки, а прятать это [в javascript-консольку для firebug'а](http://habrahabr.ru/post/145895/), более подробно советую почитать [статью Игоря Бровченко](http://tigor.com.ua/blog/2009/12/13/profiling-php-with-xhprof/), он даже сравнивал несколько php-фреймворков.. или [Лорензо Албертона](http://techportal.inviqa.com/2009/12/01/profiling-with-xhprof/)  

### Слежка

Профайлеры конечно хороши когда есть начало, конец и результат в виде доклада. Но когда php висит как демон очень долго и хочется понять где же это он тормозит.. В утилитках типа **phpcs** для вывода деталей работы, часто существует флаг -v (verbose mode). Но до того как я это узнал, мой процесс уже час как висел в jenkins'е и я незнал, стоит ли мне убивать его, или он что-то полезное таки делает. К счастью, в общем случае, можно следить за IO-действиями любого процесса без его согласия с помощью **strace**:

```
strace -p IDпроцесса strace php index.php #запустит трейс вместе за запуском php в CLI режиме 
```

## Дебаг зависимостей

PHPStorm умеет искать использование методов и классов, но это статический анализ, а вот для того что-бы динамически построить дерево зависимостей, есть отличный pecl модуль inclued (на момент установки у меня - 0.1.3)

```
pecl install inclued
```

Теперь так же редактируем php.ini:

```
extension=inclued.so
inclued.enabled=1
inclued.dumpdir=/tmp/inclued
```

Это нам даёт при каждом запросе файл с сериализованным выводом функции [inclued_get_data()](http://ee.php.net/manual/ru/function.inclued-get-data.php), которая возвращает все подключённые файлы.

Если следовать мануалу, то по идее так можно сгенерировать png-файлик..

```
php /usr/lib/php/pear/gengraph.php -i inclued.06567.1
dot -Tpng -o inclued.png inclued.out.dot
```

..но у меня это просто белый квадратик. Я попробовал другой gengraph.php из [репозитория](https://github.com/eexit/Inclued) [Йориса Бертелота](http://www.eexit.net/projects/inclued.html#setup)

```
git clone https://github.com/eexit/Inclued.git /Users/artjom/PhpstormProjects/inclued/ 
sudo php /Users/artjom/PhpstormProjects/inclued/Inclued/gengraph.php -i inclued.07576.1
dot -Tpng -o inclued.png inclued.out.dot
open inclued.png
```


Увы он оказался малость битым, работал только режим зависимости классов, а не файлов, к тому же только в горизонтальном виде. Пришлось немного попатчить, в частности у dot есть режим rankdir = "LR"; с помощью которого сгенерилась красота на 6 мб.. ужас что так много инклудится, но красиво что так это обнаруживается. 

Это кстати результат загрузки админ-странички у opensource CRM'ки [zurmo-stable-0.6.90](http://zurmo.org/download), в основном там не столько Yii сложен, сколько незамороженный [Readbean](http://redbeanphp.com/)..

Как оказалось чуть позже, граф зависимостей файлов (режим php gengraph.php -T includes) не работал [из-за конфликта с XDebug](http://stackoverflow.com/questions/4430864/using-inclued-php-pecl-extension).

### Gephi

Если вам как и мне не нравится что XHProf и Inclued полагаются на graphviz и dot для визуализации, то можно воспользоваться более интересным инструментом - Gephi. Тут надо смотреть как вы хотите это анализировать - можно брать наполовину собираемый .dot файл и импортировать его, а поскольку Gephi не умеет кушать json, то как вариант - парсить исходные данные и генерировать что-то типа CSV

Inclued уже сам генерирует inclued.out.dot в указанной папке, поэтому можно использовать его сразу. На выходе имеем..

Для того что-бы нарисовать картинку XHProf, надо чуть изменить файл callgraph_utils.php, в частности функцию xhprof_get_content_by_run. Она передаёт сгенерированный dot-скрипт напрямую в dot утилиту для генерации картинки. Поэтому просто сохраним его на диск для Gephi.. 

```php
$dotfile = '/tmp/'.time().'.dot';
touch($dotfile);
$fp = fopen($dotfile,'w+');
fwrite($fp,$script);
fclose($fp);
```

С этим несколько проблем. Первая - это полуфабрикат - данные для dot уже агрегированные, есть только label, куда засунули и время, название метода, число вызовов, проценты и тп. Вторая - файл генерируется только когда открыть генерацию графа в png-формате в браузере