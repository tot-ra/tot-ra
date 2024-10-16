понедельник, 28 апреля 2008 г. в 15:16:39

[jQuery](http://docs.jquery.com/) - библиотека о которой в последнее время говорит практически [каждый web-разработчик](http://anton.shevchuk.name/javascript/jquery-for-beginners/), верстальщик и дизайнер. Написанная с учётом CSS, она упрощает доступ к одному или нескольким DOM-элементам. Если вы ещё используете prototype, то можно использовать режим совместимости (правда не факт что у вас будут работать плагины). Стандартно доступ происходит благодаря функции $ или JQuery. Элементу можно добавить (_.addClass_) или отнять (_.removeClass_) CSS-класс. Если это input-элемент, то запись и чтение происходит в аттрибуты элемента (_.attr_) и напрямую к значению (_.val)_ . Внутренние элементы можно задать как через (_.html_). Ajax очень просто вызывается функцией _$.post (url, val, callback)_

Кроме минимализма, ускоренности и CSS-селекторов библиотека мало чем по функциональности отличается от prototype, mootools. Она не расширяет родные JS-объекты, как это делает protype и существует в своём пространстве переменных, поэтому не конфликтует с другими библиотеками

![](img/Pasted%20image%2020241016182436.png)

![](img/Pasted%20image%2020241016182447.png)

![](img/Pasted%20image%2020241016182456.png)

### Примеры

1. Допустим надо сделать массовое удаление из списка (как в почте), для этого генерируется огромный список checkbox'ов, а что-бы их все их массово выделить можно создать глобальный checkbox`<input type="checkbox" name="group[{$item.id}]" class="group_checkbox" onchange="$('.group_checkbox').attr('checked',this.checked);">`
2. Часто встаёт вопрос оповещения пользователей об удачном добавлении, ошибке или просто информации. Для этого можно использовать три CSS-класса, при этом элементы можно прятать по прошествии какого-то времени. Для этого используется конструкция, запускающаяся сразу после создания DOM-дерева (а не всего документа)`$(document).ready(function() { setTimeout(function() {$('.error').fadeOut('slow');}, 20000);});`
3. Работа с select'ами очень проста (даже с multiple=true), например если надо двумя кнопками перекидывать элементы двух списков, то поможет [такой код](http://blog.jeremymartin.name/2008/02/easy-multi-select-transfer-with-jquery.html):`$().ready(function() {   $('#add').click(function() { return !$('#select1 option:selected').remove().appendTo('#select2'); }); $('#remove').click(function() { return !$('#select2 option:selected').remove().appendTo('#select1'); });   $('#save').click(function(){ $('#select1 option').attr('selected',true); $('#select2 option').attr('selected',true);}); //при сохранении формы выделяем все эелементы   });`
4. Выделить в select'е нужный элемент можно без цикла:  
    `$("#mySelect option[value='3']").attr('selected', 'selected');`
5. Проверить число отмеченных checkbox'ов очень просто:`if (!$('.check_one:checked').length)`
6. Каждому элементу можно подключать (bind-ить) обработчики событий, даже несколько за раз…`$('#someinput').bind('keyup change keydown', function() { alert ("one of three events is called!");});`
7. Изменить направление формы в зависимости от select'а можно явно (но лучше выносить все обработчики в отдельный файл)  
    `<form method="post" action="{$data.filter_link}" onsubmit="this.action=this.action+'/'+$('#filter_category option:selected').val();">`

### Классы в javascript

Несмотря на отличные библиотеки я не спешу бросать prototype.js и переходить на jQuery, потому что плохой код можно писать в любом языке и с любой библиотекой. Поэтому немного ликбеза. В Javascript можно делать объекты - очень удобно инициализировать, но они уникальны и их как массив использовать нельзя:

```
var CommentControl={ // объект читающий все комментарии
    strURL:'/comments/read/', //обычный параметр
    Init:function(){ //декларация функции
}}
```

Классы в javascript инициализируются как обычные функции, внутри которых используется ключевое слово this

```
function CommentBox(ID){
    this.strURL='/comments/reply/'+ID; //параметр куда отсылать ответ
    this.ID=ID;
    this.reply=function(){
    alert("AJAX request goes here");
}
this.update=CommentControl.Read(ID); //ссылка на }
```

### Расширения и библиотеки

Расширения ([plugins](http://plugins.jquery.com/)) в основном написаны обычными разработчиками. В качестве более функциональных наработок с интерфейсом создаётся и [jQuery UI](http://ui.jquery.com/), как аналог scriptaculous для prototype, куда входят перетаскиваемые и сортируемые элементы и виджеты для интерфейса - аккордеон, табы, слайдеры и тп. Ниже привожу свой список полезных плагинов + советую уделить внимание презентации об разработке с учётом отключённого javascript. Значительно более детальный список есть на [jquerylist](http://jquerylist.com/)

## Изображения

- [ThickBox](http://jquery.com/demo/thickbox/) - галерея, аналог lightbox
- [FancyBox](http://fancy.klade.lv/) - тоже галерея, мне меньше нравится
- [Galleria](http://devkick.com/lab/galleria/) - галерея
- [Multimedia portfolio](http://www.openstudio.fr/jQuery-Multimedia-Portfolio.html?lang=en) - горизонтальный слайдер с видео, картинками и звуком
- [imgAreaSelect](http://odyniec.net/projects/imgareaselect/) - выделение области для вырезания
- [Анимированный блок](http://www.openstudio.fr/Animated-InnerFade-with-JQuery.html) изображений, подходит при портфолио и панорамах
- [Lightbox](http://www.balupton.com/sandbox/jquery_lightbox/) с более плавной анимацией чем в prototype + занавеска как положено.

##### Таблицы

- [Flexigrid](http://webplicity.net/flexigrid/) - табличные данные
- [InGrid](http://www.reconstrukt.com/ingrid/) - ajax'ная таблица, но не на json, а на открытом html'е. Впрочем компактном и без хаков.

##### Формы

- [ajaxForm](http://www.malsup.com/jquery/form/) - создаём как обычно статичную форму, а скрипт делает из неё ajax'овую
- [Autocomplete](http://www.pengoworks.com/workshop/jquery/autocomplete.htm) - на самом деле подгружает html. Небольшие заминки были с настройкой параметра extraParams
- [DatePicker](http://marcgrabanski.com/code/ui-datepicker/) - показывает генерирует календарик под указынными полями
- [FaceBook like](http://web2ajax.fr/2008/02/03/facebook-like-jquery-and-autosuggest-search-engine/) - автоподсказки
- [jGrow](http://lab.berkerpeksag.com/jGrow) - размер textarea в зависимости от размера текста
- [DamnSmallRTE](http://www.avidansoft.com/dsrte/dsrte.php) - мелкий WYSIWYG
- [Multiselect](http://abeautifulsite.net/notebook_files/62/demo/jqueryMultiSelect.html) как обычный select + выезжающие checkbox-ы
- [NiceForms](http://www.whitespace-creative.com/jquery/jNice/) - обрамляет input-ы div-элементами с закруглёнными углами, стилизует radio и checkbox'ы. Я правда предпочитаю более независимую версию [nice forms](http://www.badboy.ro/articles/2007-01-30/niceforms/)
- [JQuery select](http://www.kaktus.cc/weblog/view/1192629359.html) - конвертирует все select-элементы в UL, которые можно более гибко стилизовать
- [MaskedInput](http://digitalbush.com/projects/masked-input-plugin) - маска заполнения input-форм
- [Закачка файлов](http://www.phpletter.com/DOWNLOAD/) с созданием iframe-элементов (напоминаю что в XHTML) они не валидные
- [Манипуляция checkbox](http://widowmaker.kiev.ua/checkbox/)ами и radio с превращением в iphone-стиль
- [jWYSIWYG](http://projects.bundleweb.com.ar/jWYSIWYG/) - простейший редактор[  
    ](http://malsup.com/jquery/form/#getting-started)

##### Layout

- [RoundedCorners](http://plugins.jquery.com/project/jquery-roundcorners-canvas) - закруглённые углы при помощи генерации элемента canvas
- [CodaSlider](http://www.ndoherty.com/demos/coda-slider/1.1.1/#1) - слайды в дополнение закладкам (tabs)
- [idTabs](http://www.sunsean.com/idTabs/) - закладки для экономии пространства, но надо стилизовать
- [tooltips](http://cssglobe.com/lab/tooltip/01/) - расширенные всплывающие подсказки

Смотрите также:

- [ООП в javascript: наследование](http://javascript.ru/tutorial/object/inheritance)
- [Unobtrusive JavaScript with jQuery](http://www.slideshare.net/simon/unobtrusive-javascript-with-jquery?src=embed "View Unobtrusive JavaScript with jQuery on SlideShare")
- [Перевод статьи Getting Startded with jQuery](http://ouch.kiev.ua/2007-04-17/1299.html)
- [Эффекты jQuery](http://www.linkexchanger.su/2008/61.html)

![](img/Pasted%20image%2020241016182535.png)

<iframe width="934" height="350" src="https://www.youtube.com/embed/8mwKq7_JlS8" title="jQuery" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
