четверг, 3 марта 2011 г. в 17:47:43

[XDebug](http://www.xdebug.org/) это отличный php-модуль для правильного дебага приложения, который в «старших» языках (читай - не _интерпретируемых_) уже сразу был встроен в компилятор. Необходимость в полноценном дебаге очевидная в сложных приложениях, где воспроизведение ошибки занимает относительно много времени, а объём данных не позволяет копаться в мегабайтах от _print_r()_, хотя этот модуль [позволяет и такие отчёты](http://habrahabr.ru/blogs/php/31452/)

![](img/Pasted%20image%2020241019200948.png)

Посколько xdebug это модуль, то не на всяком shared-хостинге он имеется и поэтому подразумевается что разработчик подымет у себя php+xdebug сам. После этого в php.ini включается модуль и его настройки по умолчанию. Заметьте что remote_host по IP ограничивает число работающих с дебагерром.

```
extension=C:\Program Files\php\ext\php\xdebug-2.1.0-5.3-vc9.dll
xdebug.profiler_enable = 1 
xdebug.remote_host=127.0.0.1
xdebug.remote_port=9000
xdebug.remote_handler=dbgp
xdebug.idekey=
```

Для правильного дебага надо соединить IDE и модуль Xdebug, что-бы последний остановил работу php и передал данные всех переменных в PHPStorm:

1. Включить прослушивание 9000 порта в PHPStorm и режим дебага в IDE через Shift+F9 или из меню Run/Debug.
![](img/Pasted%20image%2020241019201001.png)

2. Прописать в куки режим дебага что-бы X-debug на удалённом сервере понял когда ему стоит работать. Jetbrains сделали [генератор букмарков](http://www.jetbrains.com/phpstorm/marklets/) для простой работой с печеньками. Второй вариант - специальные плагины для браузеров. Параметр Ide key вводится такой же как и в настройках IDE и в php.ini

Теперь можно поставить breakpoint на любой строке. Проблемы начинаются когда данные таки начинают бегать -  если удалённый сервер на линуксе, а у вас винда то естественно пути которые на линуксе не совсем соответсвуют той иерархии кода которую видет PHPStorm. Благо эта проблема решается [маппингом папок](http://blogs.jetbrains.com/webide/2011/02/zero-configuration-debugging-with-xdebug-and-phpstorm-2-0/#more-1364) на нужное место. Вторая проблема - закрытый код. XDebug как ни в чём не бывало выдаёт все пути что запускалось.. а IDE ведь не может этого проверить.

![](img/Pasted%20image%2020241019201016.png)

Наконец главная проблема - как выкидывать trace автоматически при возникновении ошибки, без установки breakpoint? Я частично решил для себя эту проблему используя свой регистратор ошибок вместе с xdebug_break(), проблема в том что фатальные ошибки не до конца показывают stack trace.

```php
function ErrorHandler($errno, $errstr, $errfile, $errline) {
    if (!in_array($errno,array(E_NOTICE,2048))) {
        xdebug_break();
        restore_error_handler();
        trigger_error($errno.$errstr." in ".$errfile." on line ".$errline."; showed by error handler ");
    }
}

function shutDownFunction() {
    if(!is_null($e = error_get_last())){
        xdebug_break();
    }
}

function exceptionHandler($exception) {
    xdebug_break(); 
    restore_exception_handler();
}

set_error_handler('ErrorHandler');
register_shutdown_function('shutdownFunction');
set_exception_handler('exceptionHandler');  
```