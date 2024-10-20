понедельник, 28 июля 2014 г. в 11:47:18

[Protractor](http://angular.github.io/protractor/) — движок для запуска системных (end-to-end, браузерных) тестов. Внутри он использует [selenium](http://www.seleniumhq.org/) с драйверами для браузера (chromedriver), а сами тесты пишутся с синтаксисом [jasmine](http://jasmine.github.io/). Про него и карму я [уже писал](http://kurapov.name/rus/lab/quality_control/karma_jasmine_testing_angularjs/), впрочем mocha и cucumber тоже поддерживаются.

Из особенностей - protractor имеет интеграцию с angular (находит модели и [repeat](https://docs.angularjs.org/api/ng/directive/ngRepeat)-директивы) - отсюда и название слов (_angle_ - угол, _protractor_ - транспортир, т.е. угловая линейка), но может тестировать и любые другие сайты. Из недостатков - 

1. Мне всё-таки пришлось добавить angular.js и ng-app к body на не-ангулярную страницу логина что-бы работал protractor
2. В отличие от selenium, тесты надо писать вручную, никакого browser-плагина нет для записи кликов как то было в [selenium ide](http://docs.seleniumhq.org/projects/ide/)

![](../lab/img/Pasted%20image%2020241020024321.png)

Ставим, запускаем selenium и в другом процессе запускаем исполнение тестов по заданной конфигурации:

```
npm install protractor -g
webdriver-manager start
clear && protractor conf.js --verbose
```

   

Конфигурация в стиле node-модуля, просто экспортирует данные о том где запущен selenium, какие spec-файлы надо запустить, какие переменные сделать глобальными.

```
exports.config = {
    seleniumAddress: 'http://localhost:4444/wd/hub',
    specs: [
        'login.js',
        'profile.js',
        'categories.js',
        'products.js'
    ],
    baseUrl: 'http://kurapov.name',
    onPrepare: function() {
        global.$p = protractor.getInstance();
        //global.$ = require('./myjquery.js').$;
        browser.driver.manage().window().setSize(1200, 800);
    },
    
    capabilities: [
        {
            'browserName': 'chrome',
            'chrome.cli.args': ['--ignore-ssl-errors=true', '--web-security=false']
        },

        {
            'browserName': 'phantomjs',
            'phantomjs.cli.args': ['--debug=false', '--ignore-ssl-errors=true', '--web-security=false', '--webdriver-loglevel=INFO']
        }
    ]
    
    
    jasmineNodeOpts: {
        showColors: true
    }
```

 

### Недостатки работы с DOM

В моём случае я использую свою wrapper-функцию $() для обращения к DOM. Проблема в том, что protractor общается с selenium драйверами через свои интерфейсы, т.е. вы не можете напрямую как-то обращаться к DOM браузера или с удобством вызывать javascript. 

Локаторы

- **by.css**  
    
- **by.model** 
- **by.repeater** 
- by.id
- by.js
- by.className 
- by.linkText
- by.name
- by.partialLinkText
- by.tagName
- by.xpath
- by.addLocator
- by.binding
- by.buttonText
- by.cssContainingText
- by.exactBinding
- by.options
- by.partialButtonText

Найти элементы можно с помощью локаторов, из которых самый популярный — by.css('#myelement'). Дальше надо обернуть локатор выборкой элемента:

element(by.css('#myelement')).click();

Проблема в том, что элементов может быть много, а может и вообще не быть. Поэтому ваш тест должен знать что ищет 

element.all(by.css('.row')).first()

Это плохо, потому что если вы обращаетесь к **элементу которого ещё нет**, то protractor выкинет ошибку и stacktrace. Это усложняет написание повторяемых тестов.

Поэтому для себя я написал обёртку вида $('#myelement .row',3).click() которая будет выбирать третий элемент.

Вторая проблема с локаторами — асинхронность. Т.е. это конечно хорошо что гибко, но тесты получаются сложными. Методы действия над элементами как .click(), .getText() или .count() возвращают **promise-обьект** и получается что для правильной последовательности теста, надо городить лестницы из click-then

Та же проблема была и с селениумом кстати. Это неопределённость от синхронизации приложения с DOM, из-за которой приходится писать различные хаки с таймаутом

### Практически полезные фишки

В файле тестов полезно перед каждым тестов вставлять ожидание..

```
describe('profile', function () {
    beforeEach(function () {
        $p.sleep(400); //в миллисекундах
        $p.ignoreSynchronization = true; //для страниц без angular синхронизации

        console.log(jasmine.getEnv().currentSpec.description + ' started:');
    });
    
    ..
});
```

   

При работе со списками, в зависимости от того используете вы angular или нет, имеет смысл обращаться либо by.repeater, либо по css, например:

#countries select option:nth-child(2) 

Тестирование субмита форм с помощью энтера.. да и вообще эмуляция нажатий клавиш тоже возможна..

element( by.model('name') ).sendKeys("James Bond", protractor.Key.ENTER); 

Protractor умеет работать с алертами и confirm-диалогами..

$p.switchTo().alert().accept();

Он и файлы может загружать, но для этого надо заинклудить path из nodejs

```
var path = require('path');
$("#new_avatar input[type='file']").val(path.resolve(__dirname,'./testfiles/artjom_kurapov.jpg'));
```

   

В приведённом мною выше конфиге вы увидите, что я пытаюсь несколько браузеров использовать. Увы, [phantomjs](http://phantomjs.org/) у меня пока не удалось успешно запустить. Он поднимается, открывает страницу, ищет элемент и потом падает  

> "errorMessage":"Element is not currently interactable and may not be manipulated" 

### Дебаг

В коде теста, браузер можно остановить. Для меня это была основной способ разобраться что не так

browser.pause();

Кроме того protractor можно запустить в дебаг режиме, тогда клавишами c и d можно проматывать комманды теста

protractor debug conf.js

Есть ещё вариант открыть страницу и попробовать интерактивный синтаксис обращения к DOM. Об этом подробней в видео советую посмотреть - это научит вас лексикону

node ./bin/elementexplorer.js http://kurapov.name

### Организация тестов

Тесты можно сгруппировать в suite, для этого в конфиге можно задать:

  suites: {  

    users: 'spec/*.user.js',  

    admins: 'spec/*.admin.js'  

  } 

И запускать только конкретную группу:

protractor conf.js --suite admins

Когда приложение слишком большое, то имеет смысл создавать т.н. **page objects** - отдельные обьекты к которым будут привязываться элементы по css, а в самих тестах уже использовать только свойства (property) этих обьектов.

  

Написание тестов достаточно противоречиво - с одной стороны они должны быть изолированы что-бы можно было запустить каждый тест самим по себе, с другой - короткими, без повторения нудных операций авторизации и подхода к нужному месту. Для этого я тесты обёртываю в такую конструкцию..  
  

myApp.usePageAs('artjom@kurapov.name', 'admin/dash', function () {  
//test here  
});

Код myApp позволяет мне логиниться если я ещё не залогинен, используя стандартные аккаунты для тестов, открывать страницу и запускать функцию теста. В myApp я помещаю и методы проверки каких-то оповещений (messages/notifications), на случай если DOM и стили оных поменяются.

  

См. также.. 

- [Egghead - Getting started with Protractor](https://egghead.io/lessons/angularjs-getting-started-with-protractor)   
    
- [Тестируем AngularJS используя Protractor](http://stepansuvorov.com/blog/2014/02/angularjs-protractor/)