четверг, 12 июля 2007 г. в 21:55:33

XML remote procedure call на самом деле очень простая процедура, при помощи которой я теперь могу писать в своём блоге и копировать статью в livejournal.

С виду, самым простым решением вероятно выглядело бы создание такого процесса, где передача данных на сервер LJ происходил бы браузером. Достаточно создать отдельный iframe, в него поместить форму, в которую копировать содержание из другой формы и в итоге публиковать в LJ. Но - во первых это уродливо, во вторых не факт что так можно исхитриться.

Гораздо проще и удобнее все данные передать через xml-rpc. Для этого - устанавливаем готовую [библиотеку](http://phpxmlrpc.sourceforge.net/) и используем функцию..

```php
  function post2livejournal($subject,$event,$time=0) {
    require_once('lib/xmlrpc.inc');
    $lj_userid='my_livejournal_username';
    $lj_passwd='my_secret_password';
    
    if (!$time)$time=time();
    $year=date('Y',$time);
    $month=date('m',$time);
    $day=date('d',$time);
    $hour=date('H',$time);
    $minute=date('i',$time);

    $client=new xmlrpc_client("/interface/xmlrpc", "www.livejournal.com", 80);

    $params = new xmlrpcval( array(
    'username' => new xmlrpcval($lj_userid,'string'),
    'password' => new xmlrpcval($lj_passwd,'string'),
    'ver' => new xmlrpcval('1','string'),
    'lineendings' => new xmlrpcval('pc','string'),
    'event' => new xmlrpcval($event,'string'),
    'subject' => new xmlrpcval($subject,'string'),
    'year' => new xmlrpcval($year,'int'),
    'mon' => new xmlrpcval($month,'int'),
    'day' => new xmlrpcval($day,'int'),
    'hour' => new xmlrpcval($hour,'int'),
    'min' => new xmlrpcval($minute,'int')),'struct'
    );

    $msg = new xmlrpcmsg('LJ.XMLRPC.postevent');
    $msg->addparam($params);
    $client->setDebug(0);
    $result = $client->send($msg);
  } 
```

А что-бы не появилось ошибок типа **Application failed during request deserialization** проверьте кодировку в библиотеке - наверняка пытается utf отослать как iso (см xmlrpc.inc:222) 

`$GLOBALS['xmlrpc_internalencoding']='UTF-8';//ISO-8859-1`