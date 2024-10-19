четверг, 24 мая 2007 г. в 15:14:19

[Foreign keys](http://dev.mysql.com/doc/refman/5.0/en/innodb-foreign-key-constraints.html) [](http://dev.mysql.com/doc/refman/5.0/en/innodb-foreign-key-constraints.html)это движущая сила реляционных БД. Выборка по нескольким таблицам по прежнему формируется при помощи JOIN-конструкций, но FK создаёт логические связи между таблицами, понятные человеку. Очень полезно в больших проектах, где уследить за всеми связями нелегко, а такие программки как MySQL Workbench хорошо могут эти связи показать.

Создание связей начинается с создания тех таблиц, на которые надо ссылаться. Дальше в ссылающихся таблицах делается поле типа refID. Теперь создаётся уникальный FK который и связывает поля. Ключ может иметь дополнительные свойства, например каскадное удаление или update.

Для удобства создания таких структур можно воспользоваться SQLyog-ом. Типы таблиц естественно только InnoDB. К примеру таблица events ссылающаяся через FK на events_groups выглядит так..

```sql
CREATE TABLE `events` (  
`ID` int(11) NOT NULL auto_increment,  
`groupID` int(11) default NULL,  
UNIQUE KEY `ID` (`ID`),  
KEY `FK_events` (`groupID`),  
CONSTRAINT `FK_events` FOREIGN KEY (`groupID`) REFERENCES `events_groups` (`ID`) ON DELETE CASCADE,  
) ENGINE=InnoDB DEFAULT CHARSET=latin1
```