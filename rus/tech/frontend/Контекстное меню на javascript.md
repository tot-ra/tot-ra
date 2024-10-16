суббота, 3 февраля 2007 г. в 23:02:48

[Контекстное меню](http://en.wikipedia.org/wiki/Context_menu) всплывает при нажатии правой кнопкой мыши по некоторому объекту, отсюда и его название. К сожалению многие (windows) разработчики пытаются впихнуть в него все возможные действия над объектом, из-за чего скорость работы и удобство может значительно упасть.

Для web-разработки нет определённых стандартов в создании такого меню для своего сайта, но есть несколько таких javascript'ов:

- [Окошки](http://www.ajaxline.com//node/338) на prototype, без привязки к элементу на который нажали
- похожий, более [старый вариант](http://www.webmonkey.com/webmonkey/05/10/index4a_page6.html?tw=programming)
- [RightContext](http://www.harelmalka.com/rightcontext/) очень тяжёлый вариант на данный момент (версия 0.2.3)
- [jQuery context menu](http://abeautifulsite.net/notebook_files/80/demo/jqueryContextMenu.html)
- [jQuery contextMenu](http://medialize.github.com/jQuery-contextMenu/index.html)

### Вызвать событие

Firefox, IE и Safari поддерживают простой атрибут у html-элементов:

`<body oncontextmenu='alert(10);'>`

Простое казалось бы решение, но на дворе уже какой год и функциональность от представления хочется отделить. И было бы неплохо привязывать это по классу, но при этом учитывать ID каждого элемента.

  
Не забудьте сделать ещё event-ы убирающие меню при обычном щелчке по пустому месту, при минимизации страницы или потери фокуса.

```
function ContextMenu(e,ID){ 
    e = e ? e : window.event;  
    var mouse_position = { 'x' : e.clientX, 'y' : e.clientY };  
    if( typeof( window.pageYOffset ) == 'number' ) {
        scroll_position={'x':window.pageXOffset, 'y': window.pageYOffset};  
    } else if( document.documentElement && ( document.documentElement.scrollLeft || document.documentElement.scrollTop ) ) {  
        scroll_position={'x':document.documentElement.scrollLeft, 'y': document.documentElement.scrollTop}; 
    } else if( document.body && ( document.body.scrollLeft || document.body.scrollTop ) ) {  
        scroll_position={'x':document.body.scrollLeft, 'y': document.body.scrollTop}; 
    }

    $('context_menu').innerHTML=$('context_menu_prototype').innerHTML.replace(/\[ID\]/g,ID);  
    Menu.Select(ID);
    $('context_menu').style.left = mouse_position.x + scroll_position.x + 'px';  
    $('context_menu').style.top = mouse_position.y + scroll_position.y + 'px';  
    $('context_menu').show();
    return false; 
}
```

![](img/Pasted%20image%2020241016183346.png)

![](img/Pasted%20image%2020241016183353.png)

Интересные статьи по этой теме:

- [Небольшое но мощное](http://habrahabr.ru/blogs/javascript/43111/);Хабрахабр
- [Обработка нажатия](http://webew.ru/articles/180.webew) правой кнопки мыши; Александр Бурцев, webew