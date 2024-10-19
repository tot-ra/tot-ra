среда, 8 августа 2012 г. в 07:20:52

[Drupal 7](http://drupal.org/) это конструктор сайтов (CMF). А это - его краткий обзор на основе того **малого опыта** что у меня был (так что это скорей статья для меня самого). Седьмая версия уже устарела, скоро готовится выйти 8 версия, частично перенявшая symfony framework

### Если обобщить

Основное отличие от CMS - то что схему данных пользователь добавляет сам. Фактически вместо одной таблички БД, часто используется несколько таблиц под каждое поле что с одной стороны даёт **гибкие возможности пользователю** на лету создавать динамические наборы объектов, давать привилегии на него или его свойства, делать многоязычность и многоверсионность.

С другой стороны это отходит от классического понятия реляционного хранения в MySQL - данные, независимо от типа фактически хранятся как bigtable, а бездумные настройки и импорт данных легко могут привести к размеру таблиц в миллион рядов, при каких-нибудь 20к изначальных элементах. Понятно что это сложно поддерживать, оптимизировать и мигрировать в долгосрочной перспективе при росте.

PHP-процессы при загрузке страниц довольно далеки от оптимального и могут занимать 50-70 мб, что вынуждает активно оптимизировать сайты с кешированием на Varnish и Memcache, перекидыванием поиска на Solr, включением opcode кэширования, Nginx и тп.

Теоретически Drupal поддерживает другие БД типа Mongo, но практически многие модули заточены под MySQL.

В плане frontend всё неплохо - "drupal frontend developer" может работать без хорошего IDE, потому что php-шаблоны генерируются из админки со своими названиями и префиксами в зависимости от типа блоков. Поэтому и возникают  компании, как то из местных - [okia](http://www.okia.ee/), [mekaia](http://mekaia.ee/), [fenomen](http://www.fenomen.ee/), банально легко интегрировать дизайн, когда схема тоже сделана в UI.

### Терминология

Вот термины которыми оперирует система и стандартный сайт

1. Content/Entity type концептуально это аналог классов - у них тоже есть свойства (Fields + Properties)  
    
2. Entities - собственно instance классов  
    
3. Nodes (+ Article, Users, Comments, Tags) - тоже по сути Entities, экземпляры классов  
    
4. Views, Regions, Panels. Динамические (могут вызывать методы)
5. Menus - древовидная структура меню. Элементы могут быть привязаны к странице и её частям. Доступ по URL могут
6. Modules. Собственно функционал.
7. Themes. Аналогично модулям, только для стилей (CSS)

### Структура админки

- Content - редактирование статей и прочего содержания клиентом
- Structure - порядок размещения блоков, изменение типов данных

### Структура БД и пример запроса

Центральное место - табличка node, на которой и завязано всё содержимое. Пример запроса..

```
db_query("SELECT * FROM node LIMIT 5")->fetchField();
fetchObject(); 
```

![](img/Pasted%20image%2020241019193416.png)


### Качество кода

Код - необъектное говно. Нет классов, интерфейсов, пространства имён. Сплошь и рядом используются глобальные переменные, поэтому юнит тестами покрыть его нельзя. Модули состоят из набора обычных функций со своеобразной практикой наименования, например приватные функции имеют _ префикс.  Впрочем, если сравнивать с Joomla, то архитектура более предсказуемая.

Хуки (специально названные функции) меняют данные на лету, если существует функция c похожим именем, проходя по всем модулям. Отсюда и тормоза и гейзенбаги. Например предлагая hook_foo_bar будет вызван baz_foo_bar в модуле baz.   

Есть свой формат для кода, но он не PSR, использует пробелы вместо табов. PHPStorm можно настроить с правилами [Code_Sniffer скачав из модуля](http://drupal.org/project/eclipse_code_validator).

### Drush

[Drush](http://drush.ws/) это консольная утилита, облегчающая рутину, в частности очистку кэша. 

   

Ставится просто, но нуждается в связи с локальным mysql

```
pear channel-discover pear.drush.org 
sudo pear install drush/drush 
sudo ln -s /tmp/mysql.sock /var/mysql/mysql.sock
```

   

Теперь, находясь в папке проекта можно быстро ставить модули и чистить кэш

```
drush dl module_name 
drush en module_name 
drush cc all
```

   

### Модули

У каждого своя папка с файлами .info и .module. По важности выделяются четыре типа, код лежит в разных местах

1. core, optional core (/modules)  
    
2. contrib modules (sites/all/modules),  
    
3. custom modules (sites/default/modules) + VCS symlink 

#### Важные модули

  

|   |   |   |
|---|---|---|
|- Views<br>- Rules - для ограничений (форм)<br>- Pathauto & token - для ЧПУ<br>- Devel - для дебага<br>- Wysiwyg / inline images<br>- Context<br>- Flags<br>- Entity API<br>- Menu Block<br>- Menu Breadrumb   <br>    <br>- Module filter|- Chart APi<br>- Features<br>- Feeds<br>- Custom formatters<br>- SmartCrop<br>- File Entity<br>- IMCE<br>- Media <br>- CAPTCHA|- OAuth<br>- Entity tokens<br>- Image crop<br>- Job scheduler<br>- Multiblock<br>- Pathauto<br>- Token<br>- Google analytics<br>- Webform<br>- Chaos tools|

### Дистрибутивы

- [Commerce Kickstart](http://drupal.org/project/commerce_kickstart)   
    
- [Open Atrium](http://openatrium.com/)   
    
- [RedHen CRM](http://drupal.org/project/redhen)   
    
- [Open Publish](http://drupal.org/project/openpublish)   
    
- [Videola](http://videola.tv/)   
    
- [Julio](http://drupal.org/project/julio)   
    
- [OpenPublic](http://drupal.org/project/openpublic)   
    
- [COD (Conference Organizing Distribution)](http://drupal.org/project/cod)   
    
- [Pressflow](http://pressflow.org/)   
    

### Почитать

- [Drupal 7 - the essentials](http://drupal.org/node/1576418)
- [API reference](http://api.drupal.org/api/drupal) 
- drupal.org/project/**машинное_имя_модуля**
- [Setting up Jetbrains PhpStorm for use as a Drupal IDE](http://tiger-fish.com/blog/setting-jetbrains-phpstorm-use-drupal-ide)
- [Drupal formatting](http://blog.jetbrains.com/webide/2012/03/new-in-4-0-drupal-coding-style-support/)