пятница, 10 декабря 2010 г. в 11:06:14

Google как и Твиттер, предоставляет разработчикам возможность использовать **OAuth 1.0** для авторизации пользователя [стороннему приложению](https://www.google.com/accounts/IssuedAuthSubTokens) предоставлению конкретных данных по API, причём поскольку Google не централизованный Facebook и у него много полноценных сервисов типа Youtube и Picasa, то для каждого из них выборка данных своя. В общем эта авторизация (ключевые слова - OpenID, AuthSub, Federated Login) и доступ к данным (ключевые слова - JSON, XML, REST, Atom) называется [**Google Data Protocol**](http://code.google.com/intl/ru/apis/gdata/).

![](img/Pasted%20image%2020241020020908.png)

### Используем Zend Framework  

OAuth библиотек много - есть поставляемая для твиттера библиотека, есть [поставляемая для гугла](http://code.google.com/p/oauth-php/).. но я воспользуюсь Zend - платформой.  
  
1. [Регистрируем домен](https://www.google.com/accounts/ManageDomains) = приложение, запоминаем Consumer Key + Secret. На локальной машине не получится - надо полноценно работающий домен  

2. Ставим из Zend Framework два модуля - Crypt и Oauth.

3. Создаём настройки где указываем пути к услугам авторизации

```
$aGoogleConfig = array(
    'callbackUrl'       =>  'http://kurapov.name',
    'siteUrl'           => 'https://www.google.com/accounts/',
    'authorizeUrl'      => 'https://www.google.com/accounts/OAuthAuthorizeToken',
    'requestTokenUrl'   => 'https://www.google.com/accounts/OAuthGetRequestToken',
    'accessTokenUrl'    => 'https://www.google.com/accounts/OAuthGetAccessToken',
          'consumerKey'       => 'kurapov.name',
    'consumerSecret'    => 'netetonenastojashijsekreteokfpwoekrf'
 );

$consumer = new Zend_Oauth_Consumer($aGoogleConfig);
$token = null;
```

---

4. Теперь проверяем есть ли у нас ключ доступа (access token) - на самом деле этот этап в самом конце происходит, но в коде он стоит именно тут. Ключ доступа мы получаем либо обменом на ключ запроса (request token), либо из сессии/БД куда его сохранили после первого получения. Метод getAccessToken возвращает полноценный объект, поэтому мы сериализуем его

```
if($_SESSION['GOOGLE_ACCESS_TOKEN']){
    $token = unserialize($_SESSION['GOOGLE_ACCESS_TOKEN']);
}
else
    if(isset($_GET['oauth_token'])){
    $token = $consumer->getAccessToken( $_GET, unserialize($_SESSION['GOOGLE_REQUEST_TOKEN']) );
    $_SESSION['GOOGLE_ACCESS_TOKEN'] = serialize($token));
}
```

5. Основная часть - если ключа доступа нет - запрашиваем ключ запроса с указанными двумя услугами (scope). Первый даст доступ к контактам а второй - к почте. Если же клиент уже подтвердил запрос и ключ доступа мы имеем (или получили из сессии/БД), то делаем два http-запроса.

```
if(!$token){
    $token = $consumer->getRequestToken(array( 'scope' => 'http://www-opensocial.googleusercontent.com/api/people/ https://www.googleapis.com/auth/userinfo#email'));

    $_SESSION['GOOGLE_REQUEST_TOKEN'] = serialize($token));
    $consumer->redirect();
}
else{
    $client = $token->getHttpClient($aGoogleConfig);
    $client->setUri('https://www-opensocial.googleusercontent.com/api/people/@me/@self');
    $client->setMethod(Zend_Http_Client::GET);
    $response = $client->request();

    $data = Zend_Json::decode($response->getBody());

    $client = $token->getHttpClient($aGoogleConfig);
    $client->setUri('https://www.googleapis.com/userinfo/email');
    $client->setMethod(Zend_Http_Client::GET);
    $response = $client->request();

    $emailData=explode('&',$response->getBody());
    if($emailData){
        foreach($emailDataas $sRow){
            $aRow=explode('=',$sRow);
            $data[$aRow[0]]=$aRow[1];
        }
    }

    $arrProfile=array(
        'identifier'       => $data['entry']['profileUrl'],
        'access_token'     => serialize($token),
        'verifiedEmail'    => $data['email'],
        'name'             => $data['entry']['name'],
        'photo'            => $data['entry']['thumbnailUrl'],
        'url'              => $data['entry']['profileUrl']
        );
}
```

В помощь существует [OAuth playground](http://googlecodesamples.com/oauth_playground/). Если вам нужны другие данные от гугла, то по протоколу достаточно определить другой scope, вот некоторые из них..

|   |   |
|---|---|
Некоторые возможные значения scope-параметра
|Analytics|https://www.google.com/analytics/feeds/|
|Google Buzz|https://www.googleapis.com/auth/buzz|
|Calendar|https://www.google.com/calendar/feeds/|
|Contacts|https://www.google.com/m8/feeds/|
|Documents|https://docs.google.com/feeds/|
|GMail|https://mail.google.com/mail/feed/atom|
|orkut|https://orkut.gmodules.com/social/rest|
|Picasa Web|https://picasaweb.google.com/data/|

  
Единственная огромная проблема всего этого решения — повторный запрос на привилегии когда пользователь уже авторизовал доступ. В твиттере это решалось использование authenticate URL, в фейсбуке в принципе нет такой проблемы.. с гуглом же пока есть [гибридное решение](http://googlecodesamples.com/hybrid/) с OpenID где я сейчас и [копаюсь](http://code.google.com/p/gdata-samples/source/browse/#svn/trunk/hybrid%3Fstate%3Dclosed).


![](img/Pasted%20image%2020241020020924.png)