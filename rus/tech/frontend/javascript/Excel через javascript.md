вторник, 10 апреля 2007 г. в 12:25:10

**Microsoft Excel** - незаменимый spreadsheet-редактор, попробуем сделать нечто похожее уже в web-середе.

Выбираем [Simple spreadsheet](http://www.simple-groupware.de/demo/simple_spreadsheet/spreadsheet.php?lang=en), который уже умеет достаточно много, немного его исправим и создадим возможность сохранять все данные в БД. GPL лицензия также радует.

Прежде всего - изменить размеры редактируемого окна, которое запускается из spreadsheet.php. Для этого изменяем styles.css и при необходимости - spreadsheet.js, где делаются собственно таблицы.

Дальше - сделаем сохранение данных. Для этого надо весь код поместить в форму, добавить submit, к которому приписать onclick='save();'

Simple spreadsheet хранит все данные в своём "javascript" формате который фактически просто переменные инициализации, поэтому сохранять данные мы будем не только их, но и данные в csv формате. Для этого есть функция saveCSV, которую достаточно немного изменить и добавить спрятанный textarea c id='csv'.

`getObj("csv").value = out;`

Данные сохраняются в таблицу БД. Что-бы их обратно показать в таблице для изменений, достаточно в spreadsheet.php сохранённые данные передать в $init_data

Для чтения CSV надо выдавать заголовок типа

`header("Pragma: public");   header('Content-Type: text/csv');   //header('Content-Type: application/vnd.ms-excel');   header('Cache-Control: must-revalidate, post-check=0, pre-check=0');   header("Content-Description: downloaded from iunu.net as an example");   header('Content-Disposition: attachment; filename="'.MD5(time()).'.csv"');`

Код достаточно понятен, например что-бы при нажатии Tab фокус переходил на соседнюю клетку, я добавил в функцию keypress практически в самый конец

`if (keyCode==9) {   saveCell();   ret=false;   editCell(currRow,currCol+1,keyCode);   }`

Если csv не подходящий вариант, и хочется создания xls, можно воспользоваться портированным с perl, [writeexcel](http://www.bettina-attack.de/jonny/view.php/projects/php_writeexcel/demo/) 'ем, а для импорта xls есть [spreadsheet_excel_reader](http://pear.php.net/pepr/pepr-proposal-show.php?id=304) однако проблемы с utf8 всё ещё не имеются..