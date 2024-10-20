пятница, 9 марта 2012 г. в 14:57:17

Я фанатею тестированием, в последнее время - интеграционным. И всегда хочется покрыть различные области приложения - сначала простые функции юнит тестами.. потом контроллеры моками, потом API интеграционными, наконец UI системными.. но почта для меня всегда оставалась недоступным горизонтом - а ведь хочется автоматизировать и то что почта приходит правильная.. и что вообще отсылается.

Под никсы есть два основных почтовых сервера - sendmail и [postfix](http://www.postfix.org/postconf.5.html). Я работаю с последним. 

### Настройка postfix

Прежде всего надо что-бы работал сервер и php вообще с ним был связан. Это вообще история эпичная если у вас это не работает. Особенно всевозможные relayhostы. Но допустим у вас всё настроено и для нормальных адресов почта высылается. Нам надо перенаправить всю почту высылаемую для одного адреса - в файл. Для этого (на маке) изменяем /etc/aliases и добавляем:

`integration_mail_test: "| cat > /tmp/maildump.log"`

Это значит что для такого-то пользователя всё содержимое будет pipeline'иться в этот временный лог-файл. В принципе ничто не мешает так же всё содержание писем пихать не в текстовый файл, а [в php скрипт](http://jeroensmeets.net/setup-postfix-to-forward-incoming-email-to-php/). После этого запускаем

`sudo newaliases`

Теперь, поскольку это будет работать только с локальным сервером (integration_mail_test@localhost), а у нас проекты часто будут иметь валидацию доменов, то надо бы добавить в файл основных настроек /etc/postfix/main.cf алиас для доменов..

`mydestination = $myhostname, localhost, fakedomain.com` 

Рестартанём сервак..

`sudo postfix reload`

Теперь по идее должен работать и integration_mail_test@fakedomain.com.

### Проверка почты с php

Что-бы php смог прочитать файл с письмом, надо похимичить с правами доступа. Поскольку это не production, то я просто поставил полный доступ всем

`chmod 777 /tmp/maildump.log`

Теперь читать файла очень просто..

`file_get_contents('/tmp/maildump.log');`

Но сырой исходник со всеми заголовками не всегда приятно проверять, поэтому что-бы разрезать письмо на логические части я использую [произвольный парсер plancake](https://github.com/plancake/official-library-php-email-parser/blob/master/PlancakeEmailParser.php), а поскольку он ещё и не до конца конвертирует тело письма (надеюсь пофиксят), то ещё оборачиваю в декодер и ищу нужное мне место.. и естественно всё это внутри phpunit-теста

`mail('integration_mail_test@fakedomain.com','subj','somebody text with /activate_email/ link');sleep(4); // ждём обновления файла, всё-таки может занять время покуда почтовый сервер сообразит$parser = new PlancakeEmailParser(file_get_contents('/tmp/maildump.log'));$parser->getHTMLBody();$this->assertEquals( strpos(quoted_printable_decode($parser->getHTMLBody()), '/activate_email/') !== false, true);`

### Полезные лайфхаки

Проверить работает ли у вас почта можно из терминала, просто послав

`mail artkurapov@gmail.com>Subject: пишете заголовокПишете текст. В конце ставите точку на новой строке и оно отсылается.`

Или подсоединившись по телнету:

`telnet localhost 25 HELO имясервера MAIL From: <отправитель> RCPT To: получатель`

Читайте также..

- [PHPUnit email integration testing](http://www.thedeveloperday.com/phpunit-email-integration-testing-using-sendmai/)
- [MailCatcher](http://mailcatcher.me/)