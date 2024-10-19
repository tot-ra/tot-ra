вторник, 19 декабря 2006 г. в 11:30:45

[Oracle](http://en.wikipedia.org/wiki/Oracle_database) - СУБД для "промышленности", часто устанавливаемая в больших корпорациях. Отличается надёжностью и техническими изысками. С [документацией](http://www.oracle.com/pls/db102/homepage) увы дело туго. Для соединения через php, можно использовать [adoDB](http://adodb.sourceforge.net/) библиотеку.

Для начала полезные запросы:

`select * from product_component_version   select * from ALL_TABLES   select * from USER_TABLES   select * from TABLE_PRIVILEGES`  
Это даст версию Oracle стоит ( например Oracle9i Enterprise Edition 9.2.0.6.0 ), все таблицы, вместе с [системными таблицами](http://www.techonthenet.com/oracle/sys_tables/index.php) и просто пользовательские таблицы, а так же доступ к ним.

Хочется редактировать таблицы/ряды визуально, не вбивая в каждом случае SQL?

- [Oracle Apex](http://en.wikipedia.org/wiki/Oracle_Application_Express) , ранее известный как HTML DB схож с phpMyAdmin и позволяет редактировать объекты (таблицы, тригеры и тп.)
- [Oracle editor](http://oracleeditor.sourceforge.net/) достаточно прост и практичен, если доступа к Apex нет.[  
    ](http://oracleeditor.sourceforge.net/)
- [PhpOracleAdmin](http://developer.berlios.de/projects/phporacleadmin/) как проект умер, впрочем и установка невозможна из-за ошибок  
    `Variable missed mode`
- [Ajaxora](http://ajaxora.sourceforge.net/) - opensource приложение написанное на соревновании Winter of Code.

Критичные отличия Oracle от MySQL, из-за которых возможно прийдётся глобально изменять вашу платформу:

1. Все имена колонок пишутся заглавными буквами
2. Возникают трудности с UPDATE и INSERT данных более 4 тыс символов.
3. Возникают труднсти с autoincrement свойствами. Необходимы триггеры.