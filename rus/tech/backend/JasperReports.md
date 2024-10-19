четверг, 17 мая 2012 г. в 09:47:03

[JasperReports Server](http://jasperforge.org/projects/jasperserver) это web-приложение на java (spring, hibernate, axis), бегающее на tomcat сервере (по умолчанию - http://localhost:8080/jasperserver/ ) с postgres базой данных. Основная цель - бизнес аналитика, тоесть получение агрегированных данных для отчётности по продажам, пользователям, их поведению.

![](img/Pasted%20image%2020241019193543.png)

![](img/Pasted%20image%2020241019193548.png)
![](img/Pasted%20image%2020241019193553.png)

Как альтернатива вы конечно можете всё написать сами, но готовое решение легче в обучении для далёких от программирования людей. Аналогичные продукты этой многомиллиардной индустрии - [CrystalReports](http://www.crystalreports.com/), [Pentaho](http://www.pentaho.com/), [Windward](http://www.windward.net/), [SpagoBI](http://www.spagoworld.org/xwiki/bin/view/SpagoBI/), [SAS](http://www.sas.com/software/sas9/), [BiRT](http://www.birt-exchange.com/be/home/). Они часто ещё называются **складами данных**, поскольку у них нет тех требований в скорости ответа как у традиционных баз данных, в то же время они могут хватить петабайты данных за десятки лет, который в определённый момент надо проанализировать по новым правилам.  

В качестве источников данных могут выступать:

- любая база данных через JDBC адаптер
- no-sql/масштабируемые хранилища - [MongoDB](http://jasperforge.org/plugins/mwiki/index.php/Bigdatareportingfornosqlandhadoop/MongoDB), Redis, Neo4j, hadoop-hive, couchdb, cassandra..
- веб-услуги через XML, JSON
- файлы (xlsx, csv)  
    
- всевозможные продукты - SugarCRM, SAP, Jboss

[![Схема в виде звёздочки, взято с википедии](https://s3-eu-west-1.amazonaws.com/kurapov/image/f7c114977b4c/original/Star-schema-example.png)](https://s3-eu-west-1.amazonaws.com/kurapov/image/f7c114977b4c/original/Star-schema-example.png "Схема в виде звёздочки, взято с википедии")

Как я уже сказал, используется postgres в качестве хранилища, а для гибкого анализа вместо обычных нормализованных схем с JOIN-ами, используется фактологическая схема в виде звезды, которая и хранит всю информацию. Понятно что по одной таблице такие запросы будут быстрей, да и данные особо то не меняются. Аналитики называют такой подход OLAP

### iReport

Составление отчёта происходит на основе правил в XML файле (видимо так принято во всём java-мире). Файл это можно написать конечно и вручную, но лучше использовать поставляемый IDE - **iReport**, где уже есть заготовленные шаблоны.

[![Выбор источников в iReport](https://s3-eu-west-1.amazonaws.com/kurapov/image/ee66e67012d6/original/Screen+Shot+2012-05-17+at+17.21.11.png)](https://s3-eu-west-1.amazonaws.com/kurapov/image/ee66e67012d6/original/Screen+Shot+2012-05-17+at+17.21.11.png "Выбор источников в iReport")[![Так выглядит выбор шаблона отчёта в iReport](https://s3-eu-west-1.amazonaws.com/kurapov/image/c14add591cfb/original/Screen+Shot+2012-05-17+at+17.23.37.png)](https://s3-eu-west-1.amazonaws.com/kurapov/image/c14add591cfb/original/Screen+Shot+2012-05-17+at+17.23.37.png "Так выглядит выбор шаблона отчёта в iReport")  

Доклады можно делать периодически и высылать по почте. Вот так выглядит настройка дизайна отчёта в редакторе:

[![iReport составление отчёта с SQL](https://s3-eu-west-1.amazonaws.com/kurapov/image/a67b59b2ad95/original/Screen+Shot+2012-05-17+at+17.44.33.png)](https://s3-eu-west-1.amazonaws.com/kurapov/image/a67b59b2ad95/original/Screen+Shot+2012-05-17+at+17.44.33.png "iReport составление отчёта с SQL")  

См. также:

- [Цикл статей от Вадима Войтюка](http://voituk.kiev.ua/intro-jasper-reports/) об интеграции с java-кодом и о создании xml-шаблона  
    
- [Экспорт доклада](http://redev.blogspot.com/2011/01/jasper-reports-server-php.html) в PDF с помощью php через SOAP
- [Using jasperreports with php](http://websites-development.com/blog/using-jasperreports-php)