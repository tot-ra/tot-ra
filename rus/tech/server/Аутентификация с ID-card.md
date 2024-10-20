четверг, 25 января 2007 г. в 14:14:18

ID-карта это обычная пластиковая карточка с чипом, где хранятся личные сертификаты гражданина или постоянного жителя Эстонии.

Оффициальный сайт — id.ee. Есть два разработчика приложения - SK + [ideelabor](http://ideelabor.ee/) и RIA + [Smartlink](ftp://ftp.smartlink.ee/pub/id/windows/msi/).

### Установка  

1. Устройство для чтение карточки можно купить в SEB-банке за 95 EEK
2. Устанавливаем [утилиту](http://www.sk.ee/pages.php/0202070203)
3. Устанавливаем [драйверы](https://www.id.ee/installer) из IE, а для Firefox скачиваем и запускаем [idCardmozutilsmozilla-util.exe](http://installer.id.ee/media/ID-kaart-firefox.exe) для установки в Firefox
4. Если в Firefox по прежнему не получается работать, то надо
    - экспортировать из _ID-card tool_ (Certificates / Save to file / Autentication certificate) сертификат авторизации в файл
    - импортировать его в Firefox (Tools / Options / Advanced / View certificates / Import)

### ID-card login  

Создать аутентификацию и авторизацию на своём сайте достаточно просто, но надо иметь **https**-поддержку, а это значит выделенный IP.

1. Создаём .htaccess файл в папке для корня https, обычно это чтотот типа secure/htdocs/ со следующим содержимым:  
      
      
    `SSLVerifyClient Optional SSLVerifyDepth 3`
2. Ставим в эту же папку index.php.Теперь при переходе к этой папке в скрипте будет доступна переменная $GLOBALS["SSL_CLIENT_S_DN"].
3. Проверяем кто выдал сертификат $GLOBALS["SSL_CLIENT_I_DN"] и ищем в БД соответсвующий личный код.

https://github.com/riho/Drupal-ID-Card   

http://code.google.com/p/esteid/source/browse/misc/configure-apache.sh