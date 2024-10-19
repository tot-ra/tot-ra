четверг, 12 февраля 2015 г. в 18:11:05

Современные приложения всё больше начинают напоминать полноценное desktop решение, где в итоге запускается build-процесс с помощью grunt и мы получаем один кешируемый js файл. Теперь встаёт вопрос, как бы его загрузить так что-бы показать % подгрузки?

Есть два отдельных скрипта — [$script](https://github.com/ded/script.js), который инжектит новые script-элементы с обратной связью и [progress.js](http://usablica.github.io/progress.js/), который симпатично показывает степень загрузки. Проблема в том что оба они уже сами по себе тяжёлые, в итоговый загружаемый билд-файл их нельзя вставить, а внедрять в index.html слишком бы его раздуло.

К тому же $script не поддерживает [степень загрузки](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/Using_XMLHttpRequest#Monitoring_progress), а progress.js не использует % от ширины страницы. Поэтому я написал более простенький вариант.

```
$script = function (path, callback, sizeInChars) {
    var head = document.getElementsByTagName('head')[0];
    var body = document.getElementsByTagName('body')[0];
    var div = document.createElement('div');
    div.setAttribute('style', 'background-color:#3498db; width:0px;height:3px;transition: width 0.5s linear;');
    body.insertBefore(div, body.firstChild);
    var src = path + (path.indexOf('?') === -1 ? '?' : '&');
    var oReq = new XMLHttpRequest();
    oReq.addEventListener("progress", function (e) {
        div.style.width = (100 * ( e.lengthComputable ? (e.loaded / e.total) : (e.loaded / sizeInChars))) + '%';
    }, false);
    oReq.onreadystatechange = function (e) {
        if (oReq.readyState != 4) return;
        var el = document.createElement('script');
        el.text = oReq.responseText;
        head.insertBefore(el, head.lastChild);
        callback();
        window.setTimeout(function () {
            div.remove();
        }, 500);
    };
    oReq.open('GET', src, true);
    oReq.send(null);
}
```

  

Итого — js файлы подгружаются как текст через ajax, внедряются (без eval), вызывается callback, при этом показывается div отражающий степень загрузки. Единственная проблема — вы должны **знать размер** загружаемого файла, что в случае nginx + gzip не приходит с заголовками. Поэтому приходится использовать например так..

```
$script('/js/app.js', function () {
    angular.bootstrap(document, ['myApp']);
}, 4879435);
```