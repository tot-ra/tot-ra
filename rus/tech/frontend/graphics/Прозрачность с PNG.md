понедельник, 8 января 2007 г. в 15:04:20

PNG - специально созданный графический формат для WEB, однако его использование затруднено багами в IE. PNG позволяет использовать альфа-канал, попросту - многоуровневую прозрачность. Используем IE 5.5 фильтр [AlphaImageLoader](http://msdn.microsoft.com/library/default.asp?url=/workshop/author/filter/reference/filters/AlphaImageLoader.asp) (обратите внимание на передаваемые параметры и sizingMethod)  

Также обратите внимание на то что в image src стоит прозрачный GIF. Для элементарных структур этот медо и сгодился бы, но что делать с прописанными в css background-ах, динамичными картинками в JS?

CSS background ещё можно исправить, заменив например

`background-image: url('image.png');`

на

`filter:progid:DXImageTransform.Microsoft.AlphaImageLoader(enabled=true, sizingMethod=scale, src='image.png');`

Из готовых пакетов

- [PNG behaviour](http://webfx.eae.net/dhtml/pngbehavior/pngbehavior.html) , странным образом использующий htc файл внутри css.
- [Justin Koivisto](http://koivi.com/ie-png-transparency/) пробует заменить все тэги с png на лету
- [Bob Osola](http://homepage.ntlworld.com/bobosola/pnginfo.htm) грамотно заменяет png на фильтр javascript-ом
- [DaltonLP](http://www.daltonlp.com/view/217) популяризирует png в качестве background, но без background-repeat

Для простой прозрачности фона можно использовать

`filter:alpha(opacity=25);-moz-opacity:.25;opacity:.25;`

Как альтернативу CSS, можно использовать [PNGPong](http://blog.psyrendust.com/pngpong/), который через javascript создаёт flash-объект и загружает в него png, который в свою очередь отображается с прозрачностью.