вторник, 14 июля 2009 г. в 10:31:19

Facebook - популярная социальная сеть где можно написать своё приложение. Не люблю толочь воду в ступе, поэтому сразу к делу. Встраивать можно двумя _направлениями_: внешнее приложение в Facebook или Facebook-данные во внешнее приложение (aka [Facebook Connect](http://wiki.developers.facebook.com/index.php/Anatomy_of_a_Facebook_Connect_Site)). Тут я буду говорить о первом, что в принципе более трудоёмко и интересно. Как правило смысл facebook-приложение несёт две функциональности - взаимодействие с друзьями и информативное интегрирование в профиль пользователя.

### Основы  

Встраивать приложение можно в следующие **места**..

- Canvas - собственно страница с приложением. Доступна по ссылке http://apps.facebook.com/НАЗВАНИЕ_ПРОГРАММЫ/
- Profile box - маленький бокс внутри самого профиля пользователя
- Profile tab - новый таб в профиле
- Boxes tab - небольшой блок в табе boxes
- News feed - доступ к потоку обновлений
- Requests box - интерактивные сообщения другим пользователям

Интеграция производится смешанными **возможностями**..

- REST API (http://api.new.facebook.com/restserver.php) который даёт "тяжёлый" доступ для backend-а с возможностями загрузки фото, видео, получении списков друзей, событий, комментариев и тп.
- [FQL](http://wiki.developers.facebook.com/index.php/FQL) - способ запрашивать данные по REST не просто через параметры метода, а уже через SQL-подобный синтаксис
- [FBML](http://wiki.developers.facebook.com/index.php/FBML) - урезанный HTML + свои тэги которые Facebook интерпретирует в окне в своём стиле и дизайне и кэширует при инлайновом показе. Куча заморочек с встроенным валидатором тэгов
- xFBML - FBML-тэги используемые в своём приложении
- FBJS - урезанный JS

### Два пути  

Теперь когда основные термины понятны перейдём к самому приложению которое размещается в Canvas. После создания нового приложения через [developer app](http://www.facebook.com/developers/), скачивания [REST-библиотеки](http://svn.facebook.com/svnroot/platform/clients/packages/facebook-platform.tar.gz) для php, выкладывании приложения на свой сайт и установки в настройках URL для Canvas становится видно что доступно два способа запуска - через **iframe** (+XFBML) либо **чистый FBML** который будет храниться на facebook. Понятное дело первый вариант самый простой. После создания программы и добавления/подтверждения в своём профиле, показ Canvas'а будет сопровождаться обычным iframe + GET-параметрами с префиксом fb_sig_, из которых самый важный это fb_sig_canvas_user. Второй вариант более муторный, но более тесно связан с FB.

![](img/Pasted%20image%2020241019195732.png)

### Права  

Теперь надо подумать о том что приложение делает в принципе. В моём случае это quiz-тест - пользователь отвечает на вопросы и в конце статус ставится на стенку в профиль (profile wall).

Первым делом оказывается что очень полезно иметь подтверждение пользователя для получения данных ([Authorization](http://wiki.developers.facebook.com/index.php/Authorizing_Applications)) которое вызывается методом Facebook::require_login и для пользователя выглядит просто как окошко передачи прав. Покопавшись в документации для публикации данных в Wall (News feed), есть метод [Feed.publishTemplatizedAction](http://wiki.developers.facebook.com/index.php/Feed.publishTemplatizedAction), но оказалось что он устаревший (deprecated). Альтернатива - [Stream.publish](http://wiki.developers.facebook.com/index.php/Stream.publish), и теперь переходим ко второй важной теме - расширенные права ([Extended permissions](http://wiki.developers.facebook.com/index.php/Extended_permissions)).

![](img/Pasted%20image%2020241019195746.png)

Без прав запрос получит просто фатальную ошибку. Для того что-бы программа получала более глубокий доступ над профилем пользователя, последний должен подтвердить это отдельно в диалоговом окне. А вызвать этот диалог что-бы запостить на стенку сообщение или изменить статус пользователя - не так то просто.

```
$facebook->api_client->stream_publish("My new status");
Uncaught exception 'FacebookRestClientException' with message 'The user hasn't authorized the application to perform this action' i
```

Теперь немного тонкостей - документацию в wiki и на форумах надо читать с большим подозрением, потому что часто встречается устаревший код (к примеру названия привилегий/методов stream_publish вместо publish_stream). Методы на проверку привилегий выдают либо фатальную ошибку либо невразумительную отписку на параметры, в том числе и тестовая [API console](http://developers.facebook.com/tools.php?api)

```
if($facebook->api_client->users_hasAppPermission("publish_stream")){
//обновить статус тут
}
```

#### FBML-соблазн  

К этому времени становятся понятными плюсы FBML-режима (принуждение к похожему интерфейсу и поддерживаемые FBML-тэги). У меня таки сработали

- FBML mode + onclick + тэг`<fb:prompt-permission perms="stream_publish">Heelp</fb:prompt-permission>`
- FBML mode + <form promptpermission="status_update"> + onclick
![](img/Pasted%20image%2020241019195805.png)

Iframe + редирект на http://www.facebook.com/authorize.php?api_key=МОЙ_API_KEY&v=1.0&ext_perm=publish_stream&next=http://google.com  
  
Хотя и очень глючно выглядит:

![](img/Pasted%20image%2020241019195819.png)

- FBML mode + redirect выдали "Error while loading page" сообщение

### xFBML

Понятно что права получать через iframe таким образом глючно, хочется одновременно и вместе и порознь с фейсбуком жить. Для этого есть xFBML-тэги которые интерпретируются фейсбуковским яваскриптом. Итого надо в своём приложении надо:

1. Подключить яваскрипт http://static.ak.connect.facebook.com/js/api_lib/v0.4/FeatureLoader.js.php
2. Сделать [xd_receiver.htm](http://wiki.developers.facebook.com/index.php/XFBML)
3. Инициализировать своим апи-ключём
4. Указать путь к корню в настройках Connect-приложения в фейсбуке

Теперь уже права можно спрашивать

```
FB.Connect.showPermissionDialog('publish_stream');
```
![](img/Pasted%20image%2020241019195840.png)
### Затерянное печенько ослика  

Нельзя обойти не упомянув об IE 6/7 и тут. Дело в том что по умолчанию iframe в этих браузерах теряет переменные сессии. Проще говоря - печеньки (cookies) не доходят до сервера, потому что iframe считается "неблагонадёжным" содержанием и он даже показывает глазик в своём статус-баре. Если [подробно](http://stackoverflow.com/questions/389456/cookie-blocked-not-saved-in-iframe-in-internet-explorer) в этом разбираться то для этого есть обоснование в виде [W3C platform for privacy preferences](http://www.w3.org/TR/P3P/#guiding_principles). Для этого проще добавить заголовок

```
header('P3P: CP="IDC DSP COR ADM DEVi TAIi PSA PSD IVAi IVDi CONi HIS OUR IND CNT"');  //for IE6/7
```

В конце концов приложение успешно обновило профиль (куда удалось впихнуть и картинку через аттачмент)

![](img/Pasted%20image%2020241019195852.png)

### Редирект хаоса

Любой тестер свихнулся бы увидя странный мандельбаг с отсутсвием прав на постинг в wall. Вторая меньшая проблема проявлялась в хаотичном выпрыгивании приложения из iframe в большое окно. Как [оказалось](http://stackoverflow.com/questions/595059/facebook-app-iframe-worries-url-problem), фейсбук в разных браузерах странным образом интерпретирует переход по ссылкам и внутренним редиректам. Для этого специально нашёлся метод facebook->require_frame до логина, который привязал в обязательном порядке страницу с фреймом фейсбука. Впрочем внутри между переходами страниц всё-равно засел login.php

По теме:

- Более [тягучее введение](http://blog.cmsdevelopment.com/0000414/) от Димы Шейко
- Некоторые заметки от [cr_it](http://cr-it.livejournal.com/6764.html)