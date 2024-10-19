понедельник, 13 сентября 2010 г. в 15:58:14

Facebook [Graph API](http://developers.facebook.com/docs/api) - новая версия программного интерфейса фейсбука где данные пользователя передаются в JSON формате, а список связанных объектов и [привилегий](http://developers.facebook.com/docs/authentication/permissions) значительно понятней (друзья, фото, видео, события, группы, посещения, события и тп. ). Например так выглядит информация обо мне (я чуток урезал).

```json
{
   "id": "712392972",
   "name": "Artjom Kurapov",
   "first_name": "Artjom",
   "last_name": "Kurapov",
   "link": "http://www.facebook.com/artkurapov",
   "about": "http://kurapov.name",  
   "verified": true,
}
```

Впрочем проблем с [прошлого раза](http://kurapov.name/rus/technology/web/facebook_app) не убавилось и [каша](http://developers.facebook.com/docs/authentication/) как со стандартами, так и с документацией остаётся - **API меняется так часто**, что документация в wiki [не поспевает](http://sambro.is-super-awesome.com/2010/05/20/the-facebook-platform/), интернет полон старых решений и [разобраться](http://forum.developers.facebook.net/viewtopic.php?pid=147959) в элементарных вещах не так то просто.

---

### Связка  

Если в твиттере OAuth основана на подписях, то фейсбук решил упростить себе работу и для безопасности канала просто используется SSL соединение. Допустим моё приложение хочет в фоновом режиме синхронизировать события из моего сайта с моими событиями в фейсбуке. Это значит что надо [зарегистрировать](http://developers.facebook.com/setup/) приложение, авторизовать по OAuth доступ приложению к своим данным и потом в фоновом режиме делать синхронизацию. Теперь посмотрим в деталях как выглядят последние два пункта..  

1. Ставим URL приложения. Копируем appId и secret к себе в код - это наш **ключ приложения** (consumer key)
2. Проверяем в приложении есть ли у нас сохранённый где-то access key и если нет то **редиректим** пользователя на https://graph.facebook.com/oauth/authorize с параметрами 
    - client_id - приложение
    - redirect_uri - куда вернуться
    - scope - прав, мне к примеру понадобятся постоянный доступ и создание событий (offline_access, create_event)  
        
3. Пользователь получает диалог, подтверждает и возвращается к нам с code параметром - ключём запроса, который мы [обменяем](http://stackoverflow.com/questions/2697258/facebook-graph-api-oauth-token) на ключ доступа (access key) по https://graph.facebook.com/oauth/access_token, передав следующие данные...  
    - client_id
    - client_secret
    - code
    - redirect_uri
    - Если вы передадите ещё и type=client_cred то ключ доступа будет урезанным (вида 134790075639751|e9PC-PhvPnOvlVbgbLmITd24hnQ) и вы будете действовать не столько от имени пользователя, сколько от имени приложения. Поэтому лучше этот параметр не указывать и получать полный ключ (вида  134790075639751|ad22e11d67b06933774e26da-712393972|D8PoAPDdvv8onIAf_CasljjK7Pk )  
        

В общем основная часть примерно так выглядит..

```php
if(!$token){
    if($_REQUEST['code']){
        $token = file_get_contents('https://graph.facebook.com/oauth/access_token?client_id='.CONSUMER_KEY.'&client_secret='.CONSUMER_SECRET.'&code='.$_REQUEST['code'].'&redirect_uri='.CONSUMER_URL);
    }
    else header('Location: https://graph.facebook.com/oauth/authorize?client_id='.CONSUMER_KEY.'&scope=offline_access,create_event,publish_stream,user_events&redirect_uri='.CONSUMER_URL);
}
```

Заметьте, что если вы пытаетесь получить события пользователя, но не спросили прав то данные будут пустые. Получив ключ доступа, можно спокойно спрашивать данные о пользователе, обновлять и синхронизировать события как и задумывалось.

По теме..  

- [Хабрахабр - Авторизация приложений и схема подписи данных на базе OAuth 2.0](http://habrahabr.ru/blogs/facebook/102964/)
- [Facebook access tokens from canvas apps](http://sambro.is-super-awesome.com/2010/05/28/facebook-access-tokens-from-canvas-apps/)
- Stackoverflow - [Do Facebook Oauth 2.0 Access Tokens Expire?](http://stackoverflow.com/questions/2687770/do-facebook-oauth-2-0-access-tokens-expire)
- Stackoverflow - [Facebook access_token invalid](http://stackoverflow.com/questions/2705756/facebook-access-token-invalid)