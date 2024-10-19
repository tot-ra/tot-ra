пятница, 8 ноября 2013 г. в 10:23:09

Небольшая заметка для себя о том как управиться с копированием и установкой на локалку существующего production-magento сайта.

- Что-бы видеть ошибки  
    `cp /errors/local.xml.sample /errors/local.xml`
- Что-бы поменять настройки БД  
    `nano /app/etc/local.xml`
- Что-бы [сменить пути](http://www.magentocommerce.com/wiki/recover/restore_base_url_settings) идём в БД табличку **core_config_data** и находим по path LIKE '%base_url' ссылки. Меняем на новые.
- Не помогло? Пути по прежнему кидают на старый сайт? Это скорей всего из-за того что Magento кеширует настройки в xml файлах. Если вы удалите кеш-файлы в var-папке..  
    rm -rf var/*  
      
    Этого может быть мало. В моём случае хардкорный дебаг Varien_Simplexml_Configкласса указал что файлы подтягиваются из временных папок.. в моём случае на маке в /var/tmp. Тоесть надо ещё  
    rm -rf /var/tmp/magento/*
- Ещё одна фишка - [не ставьте](http://www.magentocommerce.com/boards/viewthread/43148/) localhost в качестве base_url