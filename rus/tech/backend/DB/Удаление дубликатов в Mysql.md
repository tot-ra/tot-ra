пятница, 9 ноября 2007 г. в 15:33:54

Очень часто всплывает тема поиска дубликатов в таблицах, особенно когда надо установить UNIQUE constraint. Всегда можно обойтись таким запросом..

```sql
SELECT login, COUNT(login) AS cnt
FROM sys_users GROUP BY login
HAVING ( COUNT(login) > 1 )
```

И теперь..

```sql
DELETE t1 FROM sys_users t1, sys_users t2 
WHERE t1.login=t2.login AND t1.ID > t2.ID
```

Ещё один красивый и лаконичный способ..

```sql
ALTER IGNORE TABLE sys_users ADD UNIQUE INDEX(login);
```

Читайте также:

- [Синтаксис DELETE](http://dev.mysql.com/doc/refman/5.0/en/delete.html) в mysql