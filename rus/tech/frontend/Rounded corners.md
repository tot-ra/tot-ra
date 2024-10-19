пятница, 23 февраля 2007 г. в 19:51:40

Закруглённые углы - элемент дизайна, подражения природе. В природе не существует углов - всё разрушается, выветривается, стачивается и в результате - закругляется и как правило - тоже не идеально.  
  
Не известно какой гений додумался закруглять уголки блоков в дизайне страничек, но пальму первенства можно отдать [Apple](http://apple.com/) и MacOS.

Возможные решения

- решение в-лоб - делаем картинки, выпиливаем в photoshop набор уголков и настраиваем в css свойства нашего блока (div или table), см. [screencast](http://www.sampsonvideos.com/videos/CSS/CSS_RoundedCorners/FLV/)
- [CurvyCorners](http://www.curvycorners.net/usage.php) решение наиболее подходящее. Всё далает javascript, достаточно указать id элемента и радиусы закругления. Можно устроить и разные настройки для разных элементов. Поддерживается прозрачность и сглаживание.
- [NiftyCorners](http://www.html.it/articoli/nifty/index.html) чуть посложнее - кроме javascript функций и их инициализаций надо указывать и css стили. Поддерживает прозрачность, но настроить немного сложнее. Генерирует лишние inline-тэги b для создания углов без картинок.
- [Roundozer](http://pixel-apes.com/roundozer/?page=roundozer) так же при помощи JS закругляет уголки, правда немного тормознут, зато отечественного производства!
- [ShaderBorder](http://www.ruzee.com/blog/shadedborder) кроме уголков ещё предлагает делать тень
- Планируется что в CSS3 будет поддержка закруглённых углов, а пока gecko-engine браузеры могут использовать параметр  
    `-moz-border-radius: 4px; /* Firefox */   -khtml-border-radius: 4px;   -webkit-border-radius: 4px; /* Safari, Chrome */   border-radius:4px; /* CSS3*/   behavior: url(border-radius.htc); /*IE*/`

Читайте также:

- [Dimox пишет](http://dimox.name/smooth_rounded_corners_no_images/) по этой же теме  
    
- [Сводная таблица](http://www.smileycat.com/miaow/archives/000044.php) всех продуктов по этой теме
- [Поддержка закруглённых углов](http://shapeshed.com/journal/curved_boxes_in_css/) в некоторых браузерах, а в будующем и в CSS3