четверг, 9 августа 2007 г. в 16:27:59

Все знают про RSS и то как это читать и даже [парсить](http://magpierss.sourceforge.net/) , но как переделать из html-кода статью в валидный RSS для веб-разработчика может быть проблематично.

К типичным проблемам можно отнести присутсвие символов <, >, &. Кроме того сложности с присутсвием тэгов object внутри description приводят к тому что сделать видео объект в rss нельзя.

Пробуем [FeedCreator](http://www.bitfolge.de/rsscreator-en.html). Громадина, поддерживает всевозможные ATOM, RSS 0.9-RSS 2.0, OPML, MBOX. Надо вручную менять на UTF8 кодировку, объект хочет сразу создать xml файл. Хорошо, это в принципе разумно, кэширование в один час для блога не критично, для новостных сайтов надо уменьшать до пары минут.

```php
$rss = new UniversalFeedCreator();  
$rss->useCached();  
$rss->title = "Artjom Kurapov";  
$rss->description = "Personal Blog";  
$rss->link = "http://kurapov.name/";
```

Валидатор всё равно ругается на flash (следовательно object не поддерживается) . Кроме того не нравятся относительные пути. Конешно можно изменить [WYSIWYG](https://kurapov.ee/article/822/) что-бы он сразу генерировал абсолютные пути, но в случае если надо будет менять домен прийдётся много с базой работать. Поэтому мы их генерируем вместе с RSS.

```php
$recEntry->description=preg_replace("//i",'',$recEntry->description);  
$recEntry->description=str_replace("href='/","href='http://kurapov.name/",$recEntry->description);  
$recEntry->description=str_replace('href="/','href="http://kurapov.name/',$recEntry->description);  
$recEntry->date = date('r',$item->unix_added);  
$rss->addItem($recEntry);  
echo $rss->saveFeed("RSS2.0", "feed.xml");
```

И в результате  
[![[Valid RSS]](http://feedvalidator.org/images/valid-rss.png "Validate my RSS feed")](http://feedvalidator.org/check.cgi?url=http%3A//kurapov.name/rss)