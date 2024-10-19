вторник, 12 декабря 2006 г. в 11:11:12

Кодировки как глобальная проблема невольно зацепили и проблему со шрифтами. На данный момент очень сложно сделать на странице текст своим шрифтом так, что-бы шрифт был виден таким же и на других компьютерах.

Согласно [CSS2](http://www.w3.org/TR/REC-CSS2/fonts.html), шрифты можно подгружать в качестве ресурса, впрочем большинство браузеров это игнорирует.

`@font-face { font-family: Broken15; font-style: normal; font-weight: normal; src: url('/fonts/BROKEN0.eot');   @font-face { font-family: "Kimberley";   src: url(http://www.princexml.com/fonts/larabie/kimberle.ttf) format("truetype");}   h1 { font-family: "Kimberley", sans-serif }`

### Браузерная поддержка  

##### Решение Microsoft & IE

Использовать шрифты уже предустановленные в C:WidnowsFonts . Обычные посетители остаются обычными, система у всех только от Microsoft, больше никого не волнует.  
Для advanced-вебмастеров специально выпускается [WEFT3](http://www.microsoft.com/typography/web/embedding/weft3/tutorial.htm) конвертор True Type шрифтов (.ttf) в Embedded Open Type (.eot). Впрочем сама программа 1997 года, поэтому прийдётся потрудиться закачивать через ftp созданный eot файл, который к тому же привязан к домену сайта.

##### Решение от Mozilla ещё в [buglist](https://bugzilla.mozilla.org/show_bug.cgi?id=70132)-е.

Впрочем некогда в [далёком 1998](http://www.w3.org/Architecture/1998/06/Workshop/paper14/), делалась поддержка Portable Font Resource (pfr) от Bitstream который можно было бы использовать так же как и .eot, вот только создать pfr можно лишь купив [TrueDoc](http://www.bitstream.com/font_rendering/products/truedoc/index.html), увы найти истину - есть поддержка или нет я не смог.

### Замена на Flash  

В народе это называется SIFR ([Scalable Inman Flash Replacement](http://www.mikeindustries.com/sifr/)). Принцип немного закручен - Javascript ищет элементы по тэгам и выставляет поверх flash-слой. Есть упрощённые версии для jquery, [sifr lite](http://www.allcrunchy.com/Web_Stuff/sIFR_lite/) и тп.

В минусы

- пропадают свойства элемента типа href, title;
- Также невозможно отобразить картинку попавшую в такой элемент,
- Нельзя вызвать javascript event на таком элементе
- Проблемы со всплывающими adblock опциями в Firefox+plugins

Также как и в предыдуших решениях надо каким-то образом [шрифт впихнуть](http://digitalretrograde.com/Projects/sifrFontEmbedder/) во flash и при необходимости защитить.

#### Замена на VML/SVG

Есть несколько библиотек - [cufon](http://wiki.github.com/sorccu/cufon/usage) (поддерживаются .otf-шрифты тоже), [typeface](http://typeface.neocracy.org/examples.html) (только .ttf)

### Серверные решения

Почему бы не сделать картинку с нужным текстом шрифтом? Берём php с GD-библиотечкой и кидаем динамично в этот скрипт нужную нам строчку, он сам будет читать нужный файл шрифтов и рисовать что нам нужно. Но зачем изобретать велосипед

- [PHP+CSS dynamic](http://www.artypapers.com/csshelppile/pcdtr/) text replacing справится и сам
- [FLIR](http://facelift.mawhorter.net/)

В минусы

- неизвестный фон заголовков
- wrapping-проблемы
- проблемы с отключенными в браузере картинками
- простая замена картинками означает отсутсвие аналогов h1 тэго, снижение SEO

PS. Интересные шрифты можно найти на [fuelfonts](http://fuelfonts.com/archive/), вот только кириллица редкая.

Читайте также:

- [font face in IE](http://jontangerine.com/log/2008/10/font-face-in-ie-making-web-fonts-work)