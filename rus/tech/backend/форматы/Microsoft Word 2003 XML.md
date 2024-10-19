среда, 10 января 2007 г. в 13:07:31

Продукт Microsoft, Word 2007 проработан основательно - изменён не только дизайн но и введена поддержка XML экспорта. Увы структура его неизвестна, а ресурс, указанный в качестве документации, schemas.microsoft.com пуст. 

Сама по себе схема также известна как WordProcessingML и её простейший вид:

```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>  
<?mso-application progid="Word.Document"?>  
<w:wordDocument xmlns:w="http://schemas.microsoft.com/office/word/2003/wordml">  
<w:body>  
<w:p>  
<w:r>  
<w:t>WordML -- XML in Microsoft Word 2003</w:t>  
</w:r>  
</w:p>  
</w:body>  
</w:wordDocument>
```

Что это предоставляет? Теперь возможно экспортировать информацию в xml, а затем при помощи парсера распознавать нужные блоки и использовать в своём приложении, т.е. фактически возможен Data Mining. Другое дело что порою легче сделать copy-paste, но я не рассматриваю этот вариант пока что.

Список документации: 

- [MSDN](http://msdn.microsoft.com/library/default.asp?url=/msdnmag/issues/03/11/xmlfiles/toc.asp) информации
- [XML in office](http://msdn2.microsoft.com/en-us/office/aa905545.aspx)  
- [WordprocessignML overview](http://rep.oio.dk/Microsoft.com/officeschemas/wordprocessingml_article.htm) от датчан

Некоторые готовые продукты..

-  [WordMLToFO](http://www.antennahouse.com/product/wordmltofo.htm) XSLT конвертор платный