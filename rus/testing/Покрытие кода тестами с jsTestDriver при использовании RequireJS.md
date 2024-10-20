понедельник, 18 марта 2013 г. в 09:46:44

Когда начинаешь писать динамический фронтэнд с backbone, постепенно выясняется что неплохо бы применять практику организации файлов как в backend. Файл разбивается на папки по типам - коллекции сюда, модели сюда, view туда. По мере разрастания этих файлов, хочется лучше видеть зависимости. 

Тут в дело включается [requirejs](http://requirejs.org/) - он навязывает определение зависимостей и обёртку кода через свою **define()** функцию. Можно конечно попробовать подключать файлы без этого, но я столкнулся с проблемами в iPad. Через какое-то время замечаешь удобство от наглядности зависимостей одного модуля от другого, подобно ключевому слову **import** в css или java.

Но с таким объявлением возникает ещё две проблемы - тормоза от числа файлов (которые по идее должны лечится объединением с помощью node + r.js или серверным ускорением с SPDY) и сложностью с юнит-тестированием.

С юнит тестами я конечно использую [jsTestDriver](https://code.google.com/p/js-test-driver/) потому что с ним [просто работать в PHPStorm](http://blog.jetbrains.com/webide/tag/jstestdriver/). Никаких Jasmine/QUnit пока не было нужно. Проблема в том что он хочет сам декларативно управлять загрузкой нужных файлов, конфликтуя с requirejs

К примеру так выглядит у меня файлик jstestdriver_config.jstd:

```
load:
- ../js/lib/underscore-1.4.4.js
- ../js/lib/backbone.js
- ../js/lib/require.js
- unit/config.js
- unit/require-js-testcase.js

test:
- unit/models/*.js 
```

   

Тесты же теперь выглядят так..

```
RequireJSTestCase("Story", [
    'models/story'
], {

    setUp: function(){
        this.object = new Devclub.Models.Story();
    },

    'testName': function () {
        assertEquals("Vasja Pupkin", this.object.formatName("Vasja", "Pupkin"));
    }
});
```

   

Оставшиеся проблемы - config.js для requirejs хочет довольно экзотический абсолютный путь к файлам (заметьте префикс /test/). А я хочу что-бы тесты лежали отдельно от кода.

```
require.config({
    baseUrl: '/test//Users/artjom/PhpstormProjects/Devclub/js/'
}); 
```

   

Но ещё большая печалька, которую я немогу обойти - **не генерируется покрытие** подгруженного кода. Это совсем плохо, до степени "а не избавится ли от этого requirejs вовсе". Я конечно [добавил issue](http://youtrack.jetbrains.com/issue/WI-17366) в PHPStorm для поддержки, но сомневаюсь что они за это возьмутся без ваших голосов.

upd. Второй способ асинхронно подгружать зависимости - использовать плагин для jstestdriver - [AMDLoaderPlugin](https://github.com/Grandrath/jstestdriver-requirejs/blob/master/test/lib/AMDLoaderPlugin.js). Хотя он не вынуждает использовать AsyncTestCase, но всё-равно покрытие не даёт

### В итоге

Я всё-таки перешёл к загрязнению глобального namespace и отключению requirejs вовсе. Это накладывает ограничения на возможности юнит тестов - они должны быть с минимальными зависимостями, поэтому определение тестируемых объектов я делаю теперь так..

```
define(['backbone', 'devclub'], function (Backbone, Devclub) {
    Devclub.Models.Story = Backbone.Model.extend({});
    return Devclub.Models.Story;
});
```

   

До начала тестирования я объявляю Devclub в глобальном пространстве, делаю load всех моделей и выполняю каждое определение благодаря перезаписанной define функции, которая выглядит так..

```
var Devclub;

function define(dependencies, callback){
    callback(Backbone, Devclub);
}
```