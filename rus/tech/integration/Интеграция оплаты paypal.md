вторник, 4 августа 2009 г. в 01:00:02

Деловые люди сталкивающиеся с интернетом хотят **заработать денег**, поэтому paypal позволяющий делать оплату _кредитными картами_ по всему миру - ценнейшая услуга для интеграции на свой сайт. Примерные цены за услугу: 2-4% от суммы +  0,3$ за транзакцию зависит от типа оплаты.  

У Paypal есть несколько возможностей оплаты товара, но к счастью они едины тем что это делается HTML-формой в которой просто разные входы обозначают разные процессы оплаты. Вот некоторые процессы которые можно сделать..  

-  Покупка в "один клик"
    - С передачей данных об оплате (**[Payment Data Transfer](https://cms.paypal.com/us/cgi-bin/?cmd=_render-content&content_ID=developer/howto_html_paymentdatatransfer)**) - в обычном варианте процесс похож на pangalink. В качестве расширения можно включить авторедирект пользователя после оплаты.
    - Buy Now Hosted Button - упрощённая форма где все данные оплаты уже вбиты в paypal-админке. Хороший вариант для оплачиваемого хобби - например если вы продаёте например свой компакт-диск который один единственный для всего сайта.
    - C межсерверным оповещением (Instant Payment Notification)  
        
- Корзина
- Подписка  
    

Ниже я покажу процесс IPN. Для тестирования оплаты есть [Sandbox](https://developer.paypal.com/us/cgi-bin/devscr?cmd=home/main)-режим. Для более детального описания этого и других процессов почитайте старенькую но актуальную статью в [phpclub](http://www.phpclub.ru/detail/article/paypal#ipn)

![](img/Pasted%20image%2020241020021243.png)

### Процесс

В моём случае пользователь покупает "кредиты" (виртуальную валюту сайта) которую может потратить на разные внутренние услуги. Соответсвенно он может выбрать сколько денег он хочет потратить обычным radio-полем  

1. При открытии этой страницы в БД сразу регистрируется новый заказ со статусом "adding". Старые заказы с таким статусом удаляются.  
    
2. Пользователю показывается форма
3. При её подтверждении пользователь переносится на сайт paypal где оплачивает услугу со всеми проверками  
    
4. При удачной оплате Paypal в _фоновом_ режиме говорит серверу по notify_url о состоянии оплаты и именно в это время меняется статус заказа  
    
5. Пользователь перенаправляется на return URL где ему показывается состояние заказа

---

#### Форма

На нашем сайте надо установить HTML-[форму с полями](https://cms.paypal.com/us/cgi-bin/?cmd=_render-content&content_ID=developer/e_howto_html_Appx_websitestandard_htmlvariables), для этого в контроллере я передаю в шаблон такие данные (упрощённые что-бы вам было легче понять)

```php
$this->assign('paypal',array(
            'gate'               => 'https://www.paypal.com/cgi-bin/webscr',
//'https://www.sandbox.paypal.com/cgi-bin/webscr' для режима тестирования

            'receiver_email'     => 'artkur_1249333363_biz@gmail.com',
            'description'        => 'Quarti.ru credits',
            'price'              => 10, //в долларах
            'order_id'           => $iOrder, //номер заказа
            'notify_url'         => 'http://quarti.ru/paypal_direct_responce/',
            'link_return'        => 'http://quarti.ru/payment_completed/'
            'link_cancel'        => 'http://quarti.ru/payment_failed/',
        ));
```

Дальше в шаблоне рисуем собственно форму

```html
<form method="post" action="{$paypal.gate}" id="paypal_form">
    <input type="hidden" name="cmd"             value="_xclick" />
    <input type="hidden" name="business"        value="{$paypal.receiver_email}" />
    <input type="hidden" name="item_name"       value="{$paypal.description}" />
    <input type="hidden" name="item_number"     value="{$paypal.order_id}" />
    <input type="hidden" name="amount"          value="{$paypal.price}" />
    <input type="hidden" name="return"          value="{$paypal.link_return}" />
    <input type="hidden" name="cancel_return"   value="{$paypal.link_cancel}" />
    {if $paypal.notify_url}
    <input type="hidden" name="notify_url"  value="{$paypal.notify_url}" />
    {/if}
    <input type="hidden" name="no_shipping"     value="1" />
    <input type="hidden" name="rm"              value="2" />
</form>
```

### Проверка

Когда пользователь оплачивает покупку, то paypal в фоновом режиме передаёт статус оплаты. Это необходимо что-бы иметь синхронизированную транзакцию даже если пользователь не вернётся на наш сайт после оплаты по каким-то причинам. Проверка простейшая - собираются все данные что "paypal" запостил и переспрашиваются у него через CURL. А дальше уже обновляются статусы в БД, пишется лог со временем, передача ошибок и тп.

```php
private function verifyPaypalNotification($sGatewayURL,$sMerchantEmail,$sError){
    $postdata="";
    foreach ($_POST as $key=>$value) {
        $postdata.=$key."=".urlencode($value)."&";
    }
    
    $postdata .= "cmd=_notify-validate";
    $curl = curl_init($sGatewayURL);
    curl_setopt ($curl, CURLOPT_HEADER, 0);
    curl_setopt ($curl, CURLOPT_POST, 1);
    curl_setopt ($curl, CURLOPT_POSTFIELDS, $postdata);
    curl_setopt ($curl, CURLOPT_SSL_VERIFYPEER, 0);
    curl_setopt ($curl, CURLOPT_RETURNTRANSFER, 1);
    curl_setopt ($curl, CURLOPT_SSL_VERIFYHOST, 1);
    $response = curl_exec ($curl);
    curl_close ($curl);
    
    if ($response != "VERIFIED") {
        $sError="TRANSACTION_NOT_VERIFIED";
        return false;
    }
    
    
    if (strtolower($_POST['receiver_email']) != $sMerchantEmail){
        $sError="INVALID_RECEIVER";
        return false;
    }
    
    if ($_POST["txn_type"] != "web_accept"){
        $sError="INVALID_TRANSACTION_TYPE";
        return false;
    }
    return true;
}
```

  
См также
- [Начало работы с PayPal на PHP](http://kichrum.org.ua/paypal-on-php-01-03-2014.html)