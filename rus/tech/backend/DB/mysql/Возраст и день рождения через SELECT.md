понедельник, 1 декабря 2008 г. в 14:27:41

Иногда хочется быстро вычислить сколько дней до дня рождения пользователя. Вы все видели это в Одноклассниках и ЖЖ. Это можно делать в запросе например таким образом для mysql 5

```sql
SELECT
TIMESTAMPDIFF(YEAR,users.birth_date,NOW()) years_old,
TIMESTAMPDIFF(DAY, NOW(), DATE_ADD(users.birth_date, INTERVAL 1+TIMESTAMPDIFF(YEAR,users.birth_date,NOW()) YEAR)) next_birth_days
FROM users
```

Нашёл в своём стареньком коде, может кому полезно будет..

Коллега Юра скинул версию для старенькой mysql 4, может это даже лучше:

```sql
SELECT
IF(
    DATE_ADD(birth_date, INTERVAL (DATE_FORMAT(NOW(),'%Y') - DATE_FORMAT(birth_date,'%Y')) YEAR) > NOW(),
    (DATE_FORMAT(NOW(),'%Y') - DATE_FORMAT(birth_date,'%Y')) -1,
    (DATE_FORMAT(NOW(),'%Y') - DATE_FORMAT(birth_date,'%Y'))
   ) as years_old,
DATEDIFF(
    DATE_ADD (birth_date, INTERVAL IF(
    DATE_ADD(birth_date, INTERVAL (DATE_FORMAT(NOW(),'%Y') - DATE_FORMAT(birth_date,'%Y')) YEAR) > NOW(),
    (DATE_FORMAT(NOW(),'%Y') - DATE_FORMAT(birth_date,'%Y')) ,
    (DATE_FORMAT(NOW(),'%Y') - DATE_FORMAT(birth_date,'%Y')) + 1
   ) YEAR ),NOW()) as next_birth_days
FROM users
```