среда, 20 апреля 2011 г. в 13:57:18

[Greasemonkey](https://addons.mozilla.org/ru/firefox/addon/greasemonkey/) - отличный плагин для Firefox, позволяющий выполнять произвольный javascript на стороне пользователя. Тем самым это универсальный плагин заточенный под конкретный сайт, хоть и не дающий браузерного UI как другие плагины, но дающий возможность менять DOM существующих сайтов.

![](img/Pasted%20image%2020241019193024.png)

Мне вот надоело что в GMail постоянно показывается блок текста со всякой статистикой и ссылками - поставил Greasemonkey, нашёл XPath адрес этого блока с firebug, добавил его устранение, обернул в таймаут для загрузки страницы и вуаля.. чистота.

Это элементарный пример (проще только по ID элемент найти) скриптов, огромное количество которых можно найти на [userscripts.org](http://userscripts.org/)

```js
// ==UserScript==
// @author	   Artjom Kurapov <artkurapov@gmail.com>
// @name           gmail_modifier
// @namespace      gmail_modifier
// @include        https://mail.google.com/mail/?shva=1#inbox
// ==/UserScript==

var aWindow = (typeof unsafeWindow != 'undefined')? unsafeWindow: window;

aWindow.setTimeout(function(){

var xpathResult = window.frames[3].document.evaluate('/html/body/div/div[2]/div/div[2]/div/div[2]/div[2]', window.frames[3].document, null, XPathResult.ORDERED_NODE_SNAPSHOT_TYPE, null);

for (var i=0; i<xpathResult.snapshotLength; i++) {
    var nodeToHide = xpathResult.snapshotItem(i);
    nodeToHide.style.display='none';
}

},6000);
```