вторник, 3 февраля 2009 г. в 12:38:47

Иногда надо быстро временно закрыть доступ к сайту, а писать красивые вещи, перетаскивать проект в другой каталог - лень. А иногда просто надо сделать закрытую область. Для этого в апаче можно легко с помощью .htaccess файла устроить простую авторизацию. Для этого понадобится

- полный путь к проекту, например /home/data/virt15566/www.mysite.com/
- htpasswd программка, лежащая в апаче

Шаг 1 - заливаем в корень сайта .htaccess файл с таким содержанием

`AuthType Basic   AuthName "Restricted area"   AuthUserFile /home/data/virt15566/www.mysite.com/.htpasswd   Require valid-user`

Шаг 2 - генерируем содержание файла с помощью программки

`c:\>cd C:\Program Files\EasyPHP 3.0\apache\bin   C:\Program Files\EasyPHP 3.0\apache\bin>htpasswd -nmb user pass>.htpasswd`

В итоге получаем .htpasswd файл вида:

`user:$apr1$r2zs21ge$V1CxOLm7r88XNYE0aaJKm.`