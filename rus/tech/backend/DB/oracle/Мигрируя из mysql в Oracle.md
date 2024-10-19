пятница, 16 февраля 2007 г. в 17:21:46

### No Limits

В Oracle нет конструкции LIMIT 0,10, ограничивающей число результатов. Вместо этого следует использовать в WHERE области такое поле как ROWNUM. А что-бы эмулировать не только число результатов, но и offset, попробуйте запрос с внутренними подзапросами..

`SELECT * FROM (SELECT rownum as linenum, tempSubReq.* FROM (SELECT * FROM MY_TEST_TABLE WHERE MY_NAME= 'ARTJOM KURAPOV') tempSubReq WHERE rownum <= 11) WHERE linenum >= 2   `

### Access denied

В Oracle существует системная таблица пользователей, которым можно присвоить и отнять привилегии на работу с БД, поэтому не удивляйтесь если работая с чужим сервером у вас вдруг не работает ALTER TABLE..`revoke all on cms_content from juribig`А в общем синтаксис выглядит как

{grant/revoke} {select/update/delete/insert/references/alter/index} on {имятаблицы/имяфункции} from {имяпользователя/public}

### Date format?  

По сравнению с Mysql, Oracle имеет свои [методы](http://www.lamp2lapo.com/2006/11/28/mysql-to-oracle-date-and-time-helper-functions/) работы с датами..

Так вместо функции NOW() или CURTIME(), Oracle имеет SYSDATE

`SELECT DAYOFMONTH(MAX(post_date)) AS lastday,MONTH(MAX(post_date)) AS lastmonth ...   `

в свою очередь превращаются в

`SELECT TO_CHAR(MAX(post_date), 'DD') AS lastday, TO_CHAR(MAX(post_date), 'MM') AS lastmonth`

### Autoincrement  

Oracle отличается наличием своего языка PL/SQL, и наличием такого понятия как sequence, что и заменяет необходимость в отдельном параметре стобца как автоувеличение. Впрочем некоторые умники пытались избежать [их изучения](http://borkweb.com/story/sequence-lesstrigger-less-oracle-auto-increment), но это не совсем грамотное решение.