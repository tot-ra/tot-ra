четверг, 2 января 2014 г. в 18:15:23

Решил тут поиграть с двумя технологиями которые прям bleeding edge нынче в IT - [phonegap](http://phonegap.com/) и [angularjs](http://angularjs.org/). Первый позволяет абстрагироваться от родных языков для каждой платформы.. а это между прочим

- Android - Java
- iOS - Objective-C
- Blackberry - Java
- Palm OS - C, C++, Pascal
- Symbian - C++
- Windows C#

Всё пишется на html/js а исполняется уже в Webkit/IE9 компонентах внутри пакета с приложением. В результате я простое приложение которое отображает список вещей, вытащенных через ajax/json пишу не две недели как то было [на нативном андроиде/java](http://kurapov.name/rus/technology/android_introduction/), а за день, включая ios тоже. Короче сплошь плюсы

Идея моего приложения довольно проста. Для [танцевального конкурса](http://dancebaltcup.eu/) надо написать голосовалку выступлений для зрителей. По результатам голосования можно выдать потом приз зрительских симпатий. Оценивать по громкости хлопков - не дело. А телефон - хорошее, почти уникальное средство для идентификации.

### Ставим Phonegap

`sudo npm install -g phonegap`

Для разработки нам понадобятся:

- Android SDK (можно вместе с [android studio](http://developer.android.com/sdk/installing/studio.html))  
    Надо чтобы android был в PATH либо, в моём случае - линк  
    sudo ln -s /Library/android-sdk-macosx/tools/android /usr/bin/android Надо чтобы API нужной версии был установлен (в моём случае 19). С этим  
    
- Последний Xcode (и Maverics)
- Как бонус - chrome [Ripple emulator](http://emulate.phonegap.com/)

### Создаём hello-world приложение

 

```
phonegap create dancebaltcup
cd dancebaltcup
phonegap local build android
phonegap install --emulator=emulator-5554 android

phonegap local build ios
sudo npm install -g ios-sim
phonegap install ios
```


![](img/Pasted%20image%2020241020175252.png)

### Добавляем браузер и AngularJS

Открываем браузер и настраиваем внешний вид

![](img/Pasted%20image%2020241020175310.png)

Чистим стили от sample-приложения и старый index.js  
Копируем файлы в локальную папку и добавляем в index.html

- https://ajax.googleapis.com/ajax/libs/angularjs/1.2.6/angular.js
- http://ajax.googleapis.com/ajax/libs/angularjs/1.2.6/angular-touch.js
- http://ajax.googleapis.com/ajax/libs/angularjs/1.2.6/angular-route.js   
    

Добавляем приложение ng-app="myApp" в app.js. Приложение будет получать список выступлений (performances) по ajax, но пока что захардкодим..

Добавляем приложение ng-app="myApp" в app.js. Приложение будет получать список выступлений (performances) по ajax, но пока что захардкодим..  

```
var myApp = angular.module('myApp', [
    'ngTouch',
    'ngRoute'
]);

myApp.factory('Performances', function () {
    return [
        {title: 'Dolphins', club: 'Mystika'},
        {title: 'Bees', club: 'LDF'},
        {title: 'Mushrooms', club: 'Dance Act'}
    ];
});
```

  

Теперь надо контроллер ng-controller="MainCtrl" определить в controllers.js. Он то и будет список этих выступлений использовать

```
function PerformancesCtrl($scope, Performances){
    $scope.performances = Performances;
}
```

  

Теперь сам html. Тут и app и вложенный контроллер, и список выступлений. Он создаётся в цикле, к которому прикручена фильтрация.

```
<div class="app" ng-app="myApp">
    <h2>Performances</h2>

    <div ng-controller="PerformancesCtrl">
        <input type="text" ng-model="search.$" style="width:100%; padding:10px;">
        <div style="padding:10px;">
        <table style="width:100%;">
            <tr ng-repeat="performance in performances | filter:search">
                <td>{{performance.title}}</td>
                <td>{{performance.club}}</td>
            </tr>
        </table>
        </div>
    </div>
</div>
```

Выглядит пока-что скудно, но зато работает! Главное начать.

![](img/Pasted%20image%2020241020175331.png)

![](img/Pasted%20image%2020241020175337.png)

### Внешний вид и страницы

Поскольку phonegap предоставляет кросс-платформенное решение, он не занимается стилизацией под ios или андроид, также он не предоставляет вам никаких инструментов по управлением внешним видом или правилами как должна быть построена навигация (guidelines).

Если вы забыли как выглядят мобильные приложения, то неплохой набор примеров есть на [jqmgallery](http://www.jqmgallery.com/). По сути это обычная вёрстка сайтов, просто надо учитывать особенности навигации.

Теперь начинаешь задумываться о том какие css-фреймворки использовать что-бы приложение было и красиво и дёшево. 

Целые комбайны включают:

- [Ionic framework](http://ionicframework.com/docs/components/)  
    
- [Kendo UI](http://www.kendoui.com/mobile.aspx)  - платный для коммерческого использования
- [Onsen UI](http://s.onsen.io/)  
    
- [CHUI](http://chocolatechip-ui.com/)   
    

Кроме твиттерного бутстрапа есть ещё..  
[Almost Flat UI](http://websymphony.net/almost-flat-ui/), [uikit](http://www.getuikit.com/), [bootflat](http://www.flathemes.com/index.html), [yauikv2](http://codepen.io/hugo/pen/eoaDw), [Brick](http://mozilla.github.io/brick/), [Ink v2](http://ink.sapo.pt/), [Flatby UI](http://codepen.io/dennisschipper/details/EFwah), [MetroStyle Web UI](http://joseprosello.com/78961/911022/works/metrostyle-web-ui), [Pure](http://purecss.io/), [Futurico](http://scaffy.railsware.com/futurico/)

![](img/Pasted%20image%2020241020175353.png)

Некоторые мобильные приложения состоят из трёх слайдов. Например фейсбук. Одно видимое - по центру, остальные спрятаны по-бокам. Для их показа надо использовать графическое ускорение, так что стащим [стили у Кристофа](http://coenraets.org/blog/2013/03/hardware-accelerated-page-transitions-for-mobile-web-apps-phonegap-apps/). Раз уж мы взялись за стили, переделаем css на [less](http://lesscss.org/) или если вам больше нравится - на sass.

```
.horizontalTransform(@x) {
    -webkit-transform: translate3d(@x, 0, 0);
    transform: translate3d(@x, 0, 0);
}

.page {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;

    .horizontalTransform(0);

    &.left {
        .horizontalTransform(-100%);
    }

    &.center {
        .horizontalTransform(0%);
    }

    &.right {
        .horizontalTransform(100%);
    }

    &.transition {
        -webkit-transition-duration: .25s;
        transition-duration: .25s;
    }
}
```

   

Отлично. Добавим в html слайды справа и слева. Теперь как заставить их двигаться? Добавим [pageslider.js](https://raw.github.com/ccoenraets/PageSlider/master/pageslider.js), который именно по этим классам и будет разбираться что куда поворачивать, добавляя transition класс. Добавим стилей для фона и начнём работать с ajax как того хочет angular.

### Обмен данными с $http.JSONP

Избавимся от захардкоденных данных. Поставим на сервере php код который будет отвечать..

```
"Access-Control-Allow-Origin: *");
header("Content-type: application/json");

echo $_GET['callback'].'([{title: "Dolphins", club: "LDF"},{title: "Bees", club: "LDF"},{title: "Mushrooms", club: "Dance Act"}]);';
```

   

Заметьте что данные обёрнуты в функцию, чьё название приходит вместе с запросом. Это и специфические заголовки необходимы пока что что-бы нормально тестировать JSONP-запросы на случай если вы работаете не с локальной машиной. Теперь в клиенте заменим контроллер, что-бы получать эти данные. JSON_CALLBACK на самом деле будет заменяться каждый раз на что-то своё

```
function PerformancesCtrl($scope, $http) {
    $http.jsonp("http://dancebaltcup.eu/front/performances?callback=JSON_CALLBACK")
    .success(function (data) {
        $scope.performances = data;
    }).error(function(data,status,headers,config){
        console.error(status);
    });
}
```

   

### Client-side данные

Что-бы не хардкодить данные в приложении но и не спрашивать их каждый раз по сети, будем использовать хранение на клиенте. Обычно максимальный объём данных в районе 5-50 мб. 

Тут есть несколько слоёв и решений - использовать напрямую доступные в webkit:
![](img/Pasted%20image%2020241020175409.png)

- localStorage, sessionStorage
- [Web SQL](http://www.html5rocks.com/en/tutorials/webdatabase/todo/) (заморожена w3c в 2010 г.)  
    
- IndexedDB - индексированная БД!
- [Application cache](http://www.html5rocks.com/ru/tutorials/appcache/beginner/) - кеширование ресурсов (картинок, html файлов)
- [fileAPI](http://www.w3.org/TR/FileAPI/) - доступ напрямую к файловой системе  
    

Либо абстрагироваться вместе с js-БД типа:

- [TaffyDB](http://www.taffydb.com/)  
    
- [PouchDB](http://pouchdb.com/)  
    
- [NeDB](https://github.com/louischatriot/nedb)

Попробуем использовать [модуль localStorage для AngularJS](http://gregpike.net/demos/angular-local-storage/angular-local-storage.js). Просто копируем себе файл, добавляем в app использование модуля LocalStorageModule  и используем localStorageService

```
function PerformancesCtrl($scope, $http, localStorageService) {
    $scope.performances = localStorageService.get('performances');
    if($scope.performances===null){
        $http.jsonp("http://dancebaltcup.eu/front/performances?callback=JSON_CALLBACK")
        .success(function (data) {
            $scope.performances = data;
            localStorageService.set('performances',data);
        });
    }
}
```

   

В следующих статьях я постараюсь раскрыть более продвинутые штучки, по мере того как буду их изучать. Stay tuned

