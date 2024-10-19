понедельник, 8 января 2007 г. в 17:07:45

### Введение

[OpenID](http://openid.net/) это логика разделения аутентификации ([authentication](http://en.wikipedia.org/wiki/AAA_protocol)), которая в свою очередь отвечает за то, что-бы пользователь был в действительности тем за кого он себя выдаёт. В качестве методов authentication можно выделить пароль, сертификат, биометрические данные.

После распознавания с кем идёт дело, вступает в игру процесс правообладания (authorization), который даёт всем знать какие права у пользователя имеются.

### Возможности Open ID

Оставляя комментарий на малоизвестном вам сайте, достаточно ввести адресс своего блога, где вы аутентифицированы и вы, как по волшебству, получите положительный результат. В чём состоит техническая задача OpenID?

---

Спросить у указанного вами блога (openid сервера) залогинены ли вы там.. Из огромных плюсов следовательно можно выделить:

1. Единая система аутидентификации - не надо на каждом сайте логинится, достаточно быть залогиненым на одном
2. Возможный удар по спаму. Конечно можно создать отдельный домен и поставить там openid-сервер, но это может стоить денег, а блокируется легко.
3. Реальное подтверждение кто комментировал (т.е. надёжная связь с сайтом комментатора)

С другой стороны есть и недостатки

1. Нет полноценного обмена метаданными, нет стандарта  
    
2. Не достоверные данные - всё завязано на сайте дающем подтверждение (по сравнению со скажем Эстонской ID-картой гражданина и сертификатами)  
    
3. Трудно интегрируется с сайтами
4. Не надёжен - если сломать сайт провайдера (блог), то можно от его имени везде авторизироваться. Следовательно openid нельзя использовать в банковских системах.

### Готовые решения

Серверные решения [можно протестировать](http://openidenabled.com/resources/openid-test/diagnose-server/) сразу же на несколько действия на оффициальном сайте. Если вы не хотите или не можете интегрировать OpenID со своей системой, то можете делегировать это другим сайтам - достаточно вписать в HEAD сайта нечто подобное:  
`<link rel="openid.server" href="http://www.livejournal.com/openid/server.bml" />   <link rel="openid.delegate" href="http://tot_ra.livejournal.com" />`

##### phpMyID

OpenID сервеh [phpMyID](http://siege.org/projects/phpMyID/) легко ставистя в блог. Небольшие проблемы могут возникнуть при установке из-за встроенной системы аутидентификации, построенной на [HTTP-Digest](http://static.userland.com/userLandDiscussArchive/msg012483.html), которая по сути позволяет не передавать сам пароль по сети в открытом виде, а генерировать хэш на стороне пользователя в браузере и лишь потом сверяться с ним. Эта функциональность излишня если у вас она у вас уже на сайте есть, поэтому смело редактируйте функцию _authorize_mode_. Если же всё-таки хочется её оставить но на хостинге php работает в CGI-режиме, то возможно auth-параметры не будут передаваться.. что-бы это обойти - добавьте в .htaccess следующий код:

```
SetEnvIf Authorization "(.*)" PHP_AUTH_DIGEST=$1  
RewriteCond %{HTTP:Authorization} !^$  
RewriteCond %{QUERY_STRING} openid.mode=authorize  
RewriteCond %{QUERY_STRING} !auth=  
RewriteCond %{REQUEST_METHOD} =GET  
  
RewriteRule (.*) ?%{QUERY_STRING}&auth=%{HTTP:Authorization} [L]
```  
Также было мною испробовано:

- Jonathan Daugherty [библиотека](http://www.openidenabled.com/openid/libraries/php/) связанная с PEAR
- [Simple OpenID](http://webscripts.softpedia.com/scriptDownload/Simple-OpenID-PHP-Class-Download-12398.html). Надо попробовать.  
    
- OpenID 1.1 сервер от [Taral](http://taral.livejournal.com/147710.html) без DH-SHA1 поддержки и испытывает трудности при safe mode, line **105**`$r = fopen("/dev/urandom", "rb");`
- Videntity.org [предлагает](http://videntity.org/) свой портированный с python на php вариант, и хотя написан он отлично, с классами, примерами и наследованием, портирован он недостаточно хорошо - познакомьтесь с  
    
    **Warning**: shell_exec() has been disabled for security reasons  
    in **oid_util.php** on line **362:**
    
    ``$r = trim(`python -c "print pow($base, $exponent, $modulus)"`);``

См. также

- [openID в Эстонии](https://openid.ee/about/english)  
    
- [Plug-in для Wordpress](http://verselogic.net/projects/wordpress/wordpress-openid-plugin/)
- [Установка в Drupal](http://www.solargate.ru/ustanovka-openid-servera-v-drupal)