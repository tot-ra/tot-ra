вторник, 10 июня 2008 г. в 20:12:06

Cache — временные данные или устройство по их хранению, созданные для ускорения чтения/записи. Все программисты это знают. Ускорение загрузки web-сайтов тема обширная, начинающаяся с сервера и заканчивающаяся клиентом. К сожалению я не нашёл более-менее подходящих решений по объединению и кэшированию js-кода, поэтому к своему блогу я написал свою схему, о которой вкратце и расскажу..

Существует сжатие «[packer](http://dean.edwards.name/packer/)», которое убирает все символы форматирования и переименовывает имена функций и переменных в js и предоставляет т.н. minified-версию скрипта. Все с этим прекрасно знакомы на примере больших библиотек jQuery, TinyMCE, prototype. Кроме того что код становится совершенно не читаемым, это может вызвать неработоспособность кода, когда имена переменных динамические.

---

Моя идея простая — разделять js/css по файлам разработчикам надо для поддержания модульной структуры. Обычно я в контроллере создаю список файлов которые надо присоединить к данному документу, вместо того что-бы прописывать это вручную в темплейте. Но теперь надо сделать так, что-бы до показа темплейта вызывалась функция кэширования, которая проходилась бы по списку, проверяла из них локальные файлы на время изменения, объединяла в один файл и создавала или перезаписывала gz-файл с именем, сформированным из md5-хэша имён входящих файлов.

Всё просто и в сумме заняло часа 4 на раздумье. Привожу метод cache_js из класса Controller.

```javascript
function cache_js(){  
	$arrNewJS=array();  
	$strHash='';  
	$strGzipContent='';  
	$intLastModified=0;  
	
	foreach ((array)$this->scripts as $file){  
		if (substr($file,0,5)=='http:') continue;  
		if ($file[0]=='/') $strFilename=sys_root.$file;  
		else $strFilename=sys_root.'app/front/view/'.$file;  
		$strHash.=$file;  
		$strGzipContent.=file_get_contents($strFilename);  
		$intLastModified=$intLastModified<filemtime($strFilename) ? filemtime($strFilename) : $intLastModified;  
	}  
	
	$strGzipHash=md5($strHash);  
	$strGzipFile=sys_root.'app/front/view/js/bin/'.$strGzipHash.'.gz';  
	  
	if (file_exists($strGzipFile) && $intLastModified>filemtime($strGzipFile) || !file_exists($strGzipFile)){  
	if (!file_exists($strGzipFile)) touch($strGzipFile);  
	$gz = gzopen($strGzipFile,'w9');  
	gzputs ($gz, $strGzipContent);  
	gzclose($gz);  
}
```

Про css я писать не стану, идея аналогичная. Единственный вопрос в автоматизации кэширования и в порядке входящих файлов. Впрочем в результате сжатия 5 запросов превратилось в 1, а суммарный размер уменьшился в 3 раза.

В случае с css оказалось, что файл должен иметь расширение не .gz, а всё-таки .css, но заархивированный варинат тоже подходит

Читайте так-же:

- [Умное кеширование и версионность](http://javascript.ru/optimize/cache-versioning) в js/css
- [Практический JS/CSS](http://webo.in/articles/habrahabr/07-gzip-all/): архивируем всё
- [Сжатие css и js](http://vectora.ru/articles-and-tutorials/49-web-technologies/117-css-js-compression-without-performance-penalties) без потери производительности
- [Best practices](http://developer.yahoo.com/performance/rules.html) for speeding up your web-site