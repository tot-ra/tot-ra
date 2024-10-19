среда, 23 мая 2007 г. в 21:05:16

Никогда не задумывались как работает гениальный **copy/paste**? Правильно, благодаря всяким OLE и COM, которые продвигал Microsoft. В чём же гениальность этой функции? Interoperability, о как!

**WSDL** это формальный стандарт для описания сервизов сервисов. Под сервисом естественно подразумевается какой-то сайт, портал или просто скрипт. А зачем его описывать? Затем что-бы использовать в других таких же скриптах, программах и сайтах, а это и есть interoperability.

Этот стандарт выглядит как обычный XML файл, разрабатывается [W3C](http://www.w3.org/TR/2006/CR-wsdl20-primer-20060106/) и всевозможными корпорациями. Файл состоит из 4 основных тэгов:

- types в котором под-тэги elements описывают "переменные"  
    
- interface в котором operations описывают возможные "функции"  
    
- binding в котором детально объясняется каждая operation
- service в целом описывает местонахождение конкретного сервиса и используемые им операции

Более старые версии WSDL 1.1 имели иную структуру с portType тэгами.

В завершение можно сказать что такой стандарт предназначен для ПО, где важно предоставление некоторых программных интерфейсов для открытого доступа, которые могут использовать третьи лица на совершенно другом языке, другими алгоритмами и тп.

См. также SOAP, [типы wsdl](http://www.raleigh.ru/XML/w3schools/wsdl/ports.php) операций

<iframe width="934" height="350" src="https://www.youtube.com/embed/uPgrgX8PyyM" title="Just enough web services to survive" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
