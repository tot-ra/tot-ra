вторник, 10 января 2017 г. в 11:04:37

Недавно начал использовать триггеры в БД. Полезная штука.

Первый случай использования - валидация данных. Позволяет на уровне БД сохранять **транзитивную целостность**, т.е. более глубокую проверку, чем просто существующий foreign key. 

Вот простейший пример. Есть три таблички - site, category, article. При добавлении нового ряда в article, мы хотим проверить что categoryID использует тот же siteID что и article. Забудем сейчас про нормализацию, допустим siteID нужен и там и там. Триггером валидация решается так..

```sql
DROP TRIGGER IF EXISTS trigger_article_insert;
CREATE TRIGGER trigger_advert_insert
BEFORE INSERT ON article
FOR EACH ROW
	BEGIN
		IF NEW.siteID <>
		   (SELECT category.siteID
			FROM category
			WHERE category.ID=NEW.categoryID)
		THEN
			SIGNAL SQLSTATE '46000'
			SET MESSAGE_TEXT = 'Cannot insert row: site ID is different for article and category END IF; END;
```

Аналогично мы можем решать и вопрос обновления данных. 

В обычных случаях мы могли делать сразу JOIN или подзапрос, либо уже создать VIEW, но при высоких нагрузках лучше иметь триггер..

```sql
DROP TRIGGER IF EXISTS trigger_article_insert2;
CREATE TRIGGER trigger_article_insert2
AFTER INSERT ON advert
FOR EACH ROW UPDATE category c SET
	size = (SELECT count(a.id) FROM article a WHERE a.categoryID=NEW.categoryID AND a.status='published')
WHERE c.id=NEW.campaignID;
```