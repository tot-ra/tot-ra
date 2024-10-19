среда, 2 марта 2011 г. в 15:10:30

[E-conomic](http://www.e-conomic.com/) это датская бухгалтерская (accounting) система. Нам то может и дела нет до неё, но всё-таки попробую описать её возможности и интеграцию, поскольку она активно расширяется (уже охватывает 9 стран) и полезна для деловых людей так или иначе связанных с продажами, счетами, юридическими лицами и налогами. Кроме того она уже интегрирована с некоторыми платёжными системами, например с датской [epay](http://epay.dk/).

Прежде всего имеет смысл ознакомится с обучающими видео того, что система умеет. Для программистов же важней всего - схема БД/концептуальная диаграмма.

![](img/Pasted%20image%2020241020020714.png)

Как видно над самыми большими объектами и происходят основные операции — создание счетов, плательщика, продукта, заказа и подписки — invoice, debtor, product, order, subscription соответсвенно. Все мелкие таблички лишь уточняют детали.. как то валюта, налоги или единицы измерения.

Для интеграции со сторонними сервисами доступен API через [SOAP 1.2](https://www.e-conomic.com/secure/api1/EconomicWebService.asmx), который очень скудно документирован и число объектов-методов трудно поддаётся пониманию без знания структуры. Например такой код примерно получается при добавлении клиента

```
$resClient = new SoapClient('https://www.e-conomic.com/secure/api1/EconomicWebService.asmx?wsdl', array('soap_version' => SOAP_1_2));

$resClient->connect(array(  
    'agreementNumber' => ваш_номер_договора,  
    'userName' => логин,  
    'password' => пароль 
));

$arrExDebtor = $resClient->Debtor_FindByNumber(array('number' => 57945)); //ищем какого-то специального пользователя по ID

if (!isset($arrExDebtor->Debtor_FindByNumberResult->Number)){
    $resClient->Debtor_Create(array(
        'number'    => 57945,
        'debtorGroupHandle'  => array('Number' => 3), //добавляем в третью группу
        'name'      => 'Artjom Kurapov',
        'vatZone'    => 'HomeCountry' //зона налогов. Для не входящих в Европейский Союз = Abroad
    ));
}

$resClient->Debtor_SetCurrency(array(
    'debtorHandle' => array('Number'=> 57945),
    'valueHandle'  => array('Code'=>978) //да, это код евро согласно ISO
));
```

Обратите внимание что я всюду опустил try-catch для краткости и множество других обязательных методов по внесению данных. В общем случае каждый запрос надо оборачивать.

---

Теперь попробуем добавить счёт

```
$objInvoice = $resClient->CurrentInvoice_Create(array(
    'debtorHandle'=>array('Number'=>57945),
    'name'=> 'Artjom Kurapov'
));

$resClient->CurrentInvoice_SetCurrency(array(
    'currentInvoiceHandle'=>array('Id'=>$objInvoice->CurrentInvoice_CreateResult->Id),
    'valueHandle'=>array('Code'=>$arrUser['currency']
)));


//теперь добавим строчку.. а цикл нескольких строчек опустил
$objLineHandle = $resClient->CurrentInvoiceLine_Create( array(
    'invoiceHandle'=>array('Id'=>$intInvoiceHandle)
));
$intLineID = $objLineHandle->CurrentInvoiceLine_CreateResult->Id;
$intLineNr = $objLineHandle->CurrentInvoiceLine_CreateResult->Number;

$resClient->CurrentInvoiceLine_SetProduct(array(
    'currentInvoiceLineHandle'=>array('Id'=>$intLineID,'Number'=>$intLineNr),
    'valueHandle'=>array('Number'=> номер_используемого_продукта) //в моём случае статичный
));

$resClient->CurrentInvoiceLine_SetDescription(array(
    'currentInvoiceLineHandle'=>array('Id'=>$intLineID,'Number'=>$intLineNr),
    'value'=> "Описание продукта"
));

$resClient->CurrentInvoiceLine_SetUnitNetPrice(array(
    'currentInvoiceLineHandle'=>array('Id'=>$intLineID,'Number'=>$intLineNr),
    'value'=> 50 //цена единицы товара, нетто
));
$resClient->CurrentInvoiceLine_SetQuantity(array(
    'currentInvoiceLineHandle'=>array('Id'=>$intLineID,'Number'=>$intLineNr),
    'value'=>1 //количество товара
));

$resClient->CurrentInvoiceLine_SetUnit(array(
    'currentInvoiceLineHandle'=>array('Id'=>$intLineID,'Number'=>$intLineNr),
    'valueHandle'=>array('Number' => номер_используемой_единицы_товара) // всякие килограммы, годы, метры..
));
```