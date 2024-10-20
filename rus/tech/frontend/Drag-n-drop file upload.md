суббота, 26 сентября 2009 г. в 22:21:16

В продолжение статьи о [новинках в html5](https://kurapov.ee/rus/technology/web/html5_intro/), вот нашёл [демо](http://yehudab.com/apps/drag-n-drop/demo.html) где уже работает загрузка файлов обычным перетаскиванием в [браузер Firefox 3.6](http://ftp.mozilla.org/pub/mozilla.org/firefox/nightly/). Обычный drag-n-drop файлов пока в стабильных браузерах [максимально что могут](http://decafbad.com/blog/2009/07/15/html5-drag-and-drop) — создать события аналогичные перетаскиванию dom-элементов в определённые контейнеры.

Если взглянуть в код, то можно заметить что у события появляются такие параметры и методы:

`event.dataTransfer.files event.dataTransfer.mozTypesAt(0) event.dataTransfer.files[0].getAsBinary()`

А для отсылки файла используется обычный аяксный объект у которого появляется метод sendAsBinary(). Вобщем пока это выглядит как хак и видимо что надо больше внимания уделить спецификации html5, что-бы пользователь случайно не загрузил весь C:\Windows, что-бы можно было временно отложить загрузку или отфильтровать что конкретно надо загружать, или до какой глубины надо структуру подгружать и как обрабатывать.

### Jquery dnd-upload

Написал два jquery-плагина - один для поддержки Firefox 3.6 (Namoroka) HTML5 загрузки, другой для поддержки Google gears. Можно использовать оба вместе..

Самый простой вариант для gears:

`$('#drop_area').uploadg({gate:'http://somesite.com/upload_here_path/'});`

Заметьте что ключ получаемого у сервера файла будет "file" (в пхп соответсвенно $_FILES['file']). А вот расширенный пример кода инициализации..

`var dragndropConfig={ beforeLoad:function(){ this.gate=some_dynamic_file_upload_url+'?parentID='+SomeOtherDynamicObject.ID; if(SomeDisableReason){  this.can_proceed=false; } }, onProgress:function(event) { if (event.lengthComputable) { var percentage = Math.round((event.loaded * 100) / event.total); if (percentage &lt; 100) {  $('#some_progress_div').html('Uploading file..'+percentage+'%'); } } }, onComplete:function(event,txt) { $('#some_progress_div').html('done');  } }; if (window.google && google.gears){ $('#drop_area').uploadg(dragndropConfig); } else{ $('#drop_area').upload5(dragndropConfig); }`

Осторожно, Google gears всё ещё [имеют баги](http://groups.google.com/group/gears-users/browse_thread/thread/f6036bcc4a22cba5) связанные с такой загрузкой.