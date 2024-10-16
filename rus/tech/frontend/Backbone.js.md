понедельник, 17 октября 2011 г. в 10:35:44

![](img/Pasted%20image%2020241016183217.png)

[Backbone](http://documentcloud.github.com/backbone/) это javascript-библиотека для тяжёлых frontend javascript приложений, как например gmail или twitter. В таких приложениях вся логика интерфейса ложится на браузер, что впрочем даёт очень значительное преимущество в скорости интерфейса. А как организовать код, для таких достаточно больших проектов? Который не просто набор глобальных переменных и jquery—DOM связей. Для меня перенос бэкэнда вперёд был довольно непривычен.

Backbone близок по духу еще одной библиотеке для богатого фронта - [knockout](http://knockoutjs.com/), но как можно судить по названиям, они отличаются по замыслу. Нокаут занимается активной Позвоночни же отвечает только за достаточную многослойность архитектуры, которую надо программисту унаследовать.

### Основные классы

В качестве ядра используются наследуемые "классы"

- **Router** и History - принимают url и говорят какой view надо запустить
- **View** - привязывается к dom-элементам и в зависимости от их вложённости отвечает за их данные. Именно View вызываются из Route что-бы открыть прошлое состояние. И именно View реагирует на пользователя через подписку на события ( поле events)
- **Collection** - массив моделей (не обязательно одинаковых), привязывается к View и может оповещать его об изменениях
- **Model** - основные классы сущностей, имеют url для получения и изменения данных по RESTful http с backend

![](img/Pasted%20image%2020241016183231.png)


View, будучи конечным результатом и наиболее нагруженным классом, напрямую зависит от [js-шаблонизатора](http://documentcloud.github.com/underscore/#template) из underscore библиотеки. Шаблонизатор ничем практически не отличается - только тем что все шаблоны подгружаются сразу в script-тэгах и переменные выделяются <%=foobar%> тэгами с мини-логикой. Начало выглядит примерно так..

```
MyApp.EditableSpanView = Backbone.View.extend({
    template: _.template($('#span_wrapper').html()), 
    
    initialize: function(){
        this.render();
    },
    
    render:function(){
        $(this.el).html(this.template(this.model.toJSON()));
    }
});
```

Это объявление View с 1-1 зависимостью с DOM элементом #span_wrapper откуда мы возьмём html конструкцию и модель this.model с данными, которая ещё не задана. Документация backbone не блещет деталями, о том присвоить < model или collection если он не общий. Как-то так:

```
realSpanView = new MyApp.EditableSpanView({model: ItemModel, id:'output' });
```

Это реализация View. Тут выполнится унаследованные конструктор initialize и рисование в render. Результат будет в элементе с id output — если он уже на странице имеется то в нем, если нет то появится.  

### События

Backbone имеет своё представление о том как должны распространятся события. Я когда-то писал о [сложности этого в DOM](http://kurapov.name/rus/technology/ui/js/onclick_event_order/). Тут же всё проще. События распространяются двумя путями

1. От DOM-элемента через View.events в соответствующий написанный вами View-метод  
    
2. От модели или коллекции через подписанные на неё объекты выше (см. диаграмму выше), в т.ч. View

Насчёт первого пункта можно добавить что имеет смысл делать вложенные View-объекты (как группы DOM-деревьев). Например MyBodyView для глобально позиционируемых слоёв, MyListView с коллекцией и MyItemView для конкретного элемента.

По второму пункту - есть магический метод this.bind, который связывает два типа объектов, обычно в initialize. Например MyListView.initialize имеет смысл связать view с коллекцией, что-бы изменение коллекции вызывало изменение view. 

  

```
this.collection.bind('add', this.add, this); 
this.collection.bind('remove', this.remove, this);
```

Кажется таким очевидным и простым, но этого нет нигде по умолчанию, вы сами должны определить что должно кого оповещать. И хуже всего то, что порой не понимаешь механизма работы платформы. Модели не должны ничего знать о своём отображении, управление идёт наверху - view должен привязать себя к уничтожению и всякому изменению модели

### Столкнулся с проблемами

Получение данных на бэкэнде из модели приходят вовсе не в POST, а экзотически

```
$params = (array)json_decode(file_get_contents('php://input'));
```

Например вам необходимо постоянно поддерживать свежесть коллекции. Коллекция загружается по url параметру из fetch и дальше вызывается reset где обычно коллекция наполняется опять моделями. 

Так вот для того что-бы сделать преобработку данных без перезаписи собственно reset метода, я вешаю bind на reset, вот только аргумент результатов - не json-список и не список переконвертированных моделей у которых распечатать свойства можно только через .get(), а непонятный формат в котором впрочем есть models параметр по которому можной пройтись.

И такого непривычного поведения достаточно что-бы выбить из колеи. То же ключевое слово this - при bind теряется и на самом деле используется связанный объект (коллекция), для этого есть третий параметр или же bindAll из underscore.

#### Вывод

Библиотека интересная, ограничивающая программистов привыкших к расхлябанному jquery-подходу подвешивания событий на DOM и принуждающая frontend писать едва ли не так же качественно как и backend. Поэтому использовать её стоит в ограниченных условиях, когда надо написать больше JS-кода чем обычно. Альтернативы Backbone — [Spine.js](http://spinejs.com/), [Sproutcore](http://www.sproutcore.com/), [Cappuccino](http://cappuccino.org/), [EJS](http://embeddedjs.com/getting_started.html). Из "почитать по теме" советую подкаст [sitepoint 145](http://www.sitepoint.com/podcast-145-backbone-js-fundamentals-with-addy-osmani/), [bbtutorials](http://backbonetutorials.com/), [хабр](http://habrahabr.ru/blogs/javascript/118782/), а для особо одарённых — про [тестирование с Jasmine](http://tinnedfruit.com/2011/03/03/testing-backbone-apps-with-jasmine-sinon.html).

<iframe width="467" height="260" src="https://www.youtube.com/embed/258gBoR734U" title="Backbone.js walkthrough of Models and Views (Part 1/2)" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

<iframe width="467" height="260" src="https://www.youtube.com/embed/XGGIc800WFM" title="Backbone.js walkthrough of Models and Views (Part 2/2)" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

<iframe width="467" height="260" src="https://www.youtube.com/embed/lPiM4T1lR58" title="Using Backbone.js with Rails: Patterns from the Wild by Sarah Mei" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

<iframe title="vimeo-player" src="https://player.vimeo.com/video/28096809?h=40c7590e64" width="640" height="360" frameborder="0"    allowfullscreen></iframe>
