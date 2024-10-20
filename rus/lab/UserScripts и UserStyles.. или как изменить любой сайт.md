вторник, 11 ноября 2014 г. в 17:23:01

Часто сталкиваюсь с желанием поменять что-то на сайтах, добавив свой функционал, сделав их удобней или автоматизировав многие вещи. Во многих случаях это становится **хорошим лайфхаком**. К счастью для браузеров есть расширения, добавляющие возможность запуска user-скриптов. Для firefox это [greasemonkey](https://addons.mozilla.org/ru/firefox/addon/greasemonkey/), а для chrome - [tampermonkey](https://chrome.google.com/webstore/detail/tampermonkey/dhdgffkkebhmkfjojejmpbldmpobfkfo?hl=ru).

Движки "обезьянок" позволяют подключить внешние библиотеки (например jquery) и напрямую манипулировать страницей. Это даёт широкий простор по изменению layoutа и взаимной интеграции сайтов. Например

- [Preview картинок](https://greasyfork.org/en/scripts/404-mouseover-popup-image-viewer) на разных сайтах
- [Добавление download-ссылок](https://monkeyguts.com/code.php?id=396) для ютуба
- [Фавиконки у результатов](https://monkeyguts.com/code.php?id=91) гуглопоиска
- [Создание ссылок](https://greasyfork.org/en/scripts/2709-linkify-plus/code) из текста

К сожалению, хотя это и мощная штука, использовать чужие скрипты довольно опасно, т.к. легко можно утянуть реальные пользовательские данные.  

### UserStyles - только макияж

Если вы боитесь использовать UserScripts, то можно ограничиться только перезаписью CSS с [UserStyles](https://userstyles.org/). Например попробовать [Темный google](https://userstyles.org/styles/61377/black-google-by-panos) или [тёмную википедию](https://userstyles.org/styles/47161/dark-wikipedia-rounded).Это добавляет расширение Styles. Либо поискать специализированное расширение для браузера, например [wikiwand](http://www.wikiwand.com/) для википедии.

Twitter  

Начнём с простого.. Замечали что в VK, когда вы скроллите вниз, то лента увеличивается по ширине? А в твиттере почему-то до этого не дошли, или отказались? Попробуем расширить ленту так же..

```js
// ==UserScript==
// @name         Twitter timeline fullwidth
// @namespace    http://kurapov.name/
// @version      0.1
// @description  Simple adaptive resize of timeline in vk.com style
// @author       Artjom Kurapov
// @match        https://twitter.com/
// @require http://code.jquery.com/jquery-latest.js
// @grant        none
// ==/UserScript==


$(document).scroll(function() {
    if($(document).scrollTop()*1>1000){
        $('#timeline').css('float','none');
        $('#timeline').css('width','100%');
    }
    else{
        $('#timeline').css('float','right');
        $('#timeline').css('width','590px');
    }
})
```

### Amazon.co.uk - конвертирование валюты

Англицкий амазон любит фунты, а хочется всё видеть в евро (а россиянам - видимо в рублях). Аналогичные проблемы на американском амазоне. Есть встроенная конвертация, но она показывается только при оплате.. Поэтому, пишем свой скрипт..

```js
// ==UserScript==
// @name         Amazon currency convertor
// @namespace    http://kurapov.name/
// @version      0.1
// @description  Converts pounds into euros
// @author       Artjom Kurapov <artkurapov@gmail.com>
// @match        https://www.amazon.co.uk/*
// @match        http://www.amazon.co.uk/*
// @require http://code.jquery.com/jquery-latest.js
// @grant        none
// ==/UserScript==

$(document).ready(function(){    
    window.setInterval(function(){
        var reformatPrice = function(i,v){
            if($(v).text().indexOf('£')>=0)
            $(this).text((1.24772749*$(v).text().replace('£','').replace(',','')).toFixed(2)+'€');
        }
        
        $('.a-color-price').each(reformatPrice);
        $('.s9Price').each(reformatPrice);
        $('.price').each(reformatPrice);
    },1000);
});
```

Auto24  

Ещё полезная штука - добавлять относительный показатель километража к цене

```js
// ==UserScript==
// @name         Auto24 km per euro value
// @namespace    http://kurapov.name/
// @version      0.1
// @description  enter something useful
// @author       Artjom Kurapov
// @match        http://www.auto24.ee/kasutatud/nimekiri.php*
// @require http://code.jquery.com/jquery-latest.js
// @grant        none
// ==/UserScript==

$(document).ready(function reset(){
    
    $('#usedVehiclesSearchResult thead tr').append('<th>km/eur</th><th>km/year</th>');
    $('table tr.result-row').each(function(i, tr){
        var price = 1*$('td.price', tr).html().split(' ')[0].replace('&nbsp;','');
        var year = 2015-1*$('td.year', tr).text();
        var km = $('td.make_and_model .extra', tr).html().split('&nbsp;km')[0].replace('&nbsp;','');
        $(tr).append('<td>'+Math.round(km*price/10000)/100+'</td>');
        $(tr).append('<td style="color:'+(km/year>28000?'red':'black')+';">'+Math.round(km/year)+'</td>');
    });
    
    $('.tpl-bannerbar').remove();
    $('.abovesearch-offers').remove();
});
```

### Mamba

Этот скриптик уменьшает пустое пространство в списке и добавляет возможность спрятать тех кто не нравится

```js
// ==UserScript==
// @name         Mamba UI
// @namespace    http://kurapov.ee/
// @version      0.1
// @description  Mamba.ru interface optimizer
// @author       You
// @match        http://www.mamba.ru/search.phtml*
// @grant        none
// ==/UserScript==

$(document).ready(function(){
    $('.MainBlockLeft').remove();
    $('.ui-menu-cont').remove();
    $('.MainBlockRight').css('width','100%');
    $('.SearchBigPhotos li').css('width','300px');
    $('.SearchBigPhotos li').css('margin','0');
    
    $('.ui-menu-slide-wrapper').css('margin','0');
    $('.b-photoline').css('margin','0');
    var blocked = localStorage.getItem('blocked');
    if(blocked==null) blocked = '[]';
    blocked = JSON.parse(blocked);
    
    $('.SearchResult li').each(function(){
        var id = $('a.u-name',this).attr('href').split('/')[3].split('?')[0];
        console.log(id,blocked);
        if(blocked.indexOf(id)>=0){
           $(this).hide();
            console.log('hiding',id);
        }
        
        $(this).append("<button class='hideforever' data-id='"+id+"'>СПРЯТАТЬ</button>");
    });
    
    $('.hideforever').on('click',function(){
        blocked.push($(this).data('id'));
        console.log(blocked);
        localStorage.setItem('blocked', JSON.stringify(blocked));
        $(this).parents('li.U-Normal').hide();
    });
});
```

#### Чужие библиотеки

[https://monkeyguts.com/](https://monkeyguts.com/)  
[https://greasyfork.org/en](https://greasyfork.org/en)  
[https://openuserjs.org/](https://openuserjs.org/)