четверг, 8 января 2009 г. в 18:46:24

Предлагаю решить интересную SQL-задачу. Думаю у среднего девелопера она займёт пол часа или больше (я же сразу спросил у SQL-гуру).

### Условия

Возникла интересная задача при переходе с MyIsam на InnoDB. Как известно, полнотекстовый поиск у последнего движка отсутсвует, поэтому было два решения - делать обычное зеркало на MyIsam где был бы ктому же и очищенный от html текст (кстати так сделано в движке этого блога), либо более глубокая индексация с разбивкой на слова - я упоминал такую идею когда писал про [морфологический поиск](https://kurapov.ee/technology/morphological_search/).

Второй вариант с одной стороны давал кучу головной боли с индексацией, очисткой, держанием многотысячной таблицы, логике поиска - задачи которые должна решать СУБД или расширенный индексатор типа sphinx. С другой стороны - выглядит инновативно, теоретически можно уже организовывать синонимы для тех или иных слов на уровне индекса или искать с учётом расстояния между словами. Вобщем гугл 2. На вскидку такой способ оказался в 6 раз быстрей InnoDB.

Сказано - сделано. Получилось 3 основных таблицы + собственно таблицы которые индексируются.

- words - id, word (varchar, уникальный)
- fields - id,  field (varchar, уникальный; значения например "title", "content"), object_id (int; указывает на id статьи),
- word_fields - word_id, field_id, pos (int, положение в тексте)

---

### Задача

Надо реализовать AND-поиск по этим таблицам с одним запросом где на входе будут два (или больше) слов, а на выходе - список уникальных object_id. Сложность в том, что связующая таблица word_fields имеет не уникальные пары word_id и field_id, поскольку в тексте статьи слова могут повторяться.

Для одного слова запрос выглядит так

```sql
SELECT t2.object_id  
FROM (SELECT id FROM words t0 WHERE t0.word LIKE 'Тест%') as word_ids  
INNER JOIN word_fields t1 ON (t1.word_id=word_ids.id)  
INNER JOIN fields t2 ON (t2.id=t1.field_id AND t2.field='content')  
GROUP BY object_id  
LIMIT 10
```

### Решение

Для OR:

```sql
SELECT f.object_id FROM search_fields f  
INNER JOIN search_word_fields sf ON sf.field_id = f.field_id  
INNER JOIN search_words w ON w.id = sf.word_id AND (w.word LIKE 'Test1%' OR w.word LIKE 'Test2%')
```

Для AND:

```sql
SELECT f.object_id FROM search_fields f  
INNER JOIN search_word_fields sf1 ON sf1.field_id = f.field_id  
INNER JOIN search_words w1 ON w1.id = sf1.word_id AND w1.word LIKE 'cat%'  
INNER JOIN search_word_fields sf2 ON sf2.field_id = f.field_id  
INNER JOIN search_words w2 ON w2.id = sf2.word_id AND w2.word LIKE 'dog%'
```