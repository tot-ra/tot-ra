пятница, 23 ноября 2007 г. в 12:17:00

![](https://glassfish-theme.dev.java.net/logo.gif)Я писал про [WSDL](https://kurapov.ee/article/1494) некоторое время назад, правда я не совсем понимал как работают web-сервисы. Теперь же я покажу на простом примере зачем они в действительности нужны. В нашем случае service (_услуга_) это низкоуровневый способ передачи данных между клиентом и поставщиком через SOAP-протокол, при котором для удобства клиента, все услуги изначально описываются WSDL формату и представляют себя набор функций и параметров.

Вы можете просмотреть [видеокаст о создании сервисов](http://download.java.net/javaee5/screencasts/metro-nb6/), где в качестве поставщика используется java-приложение работающее на сервере glassfish, которое создаётся при помощи [Netbeans 6 IDE](http://download.netbeans.org/netbeans/6.0/final/). В видеокасте так же показан пример клиентского приложения, но я думаю что это не достаточно эффектный пример, ведь [MS Visual Studio 2005](http://www.microsoft.com/express/2005/) имеет удобную поддержку web-сервисов.

### Создание сервиса  

Создайте новый проект как new web application, работающего на [glassfish](https://glassfish.dev.java.net/) сервере (File -- New Project). Правый щелчок на проект -- New -- Web Service. Введя любое название package можно добавить функции (operations) и входные параметры. Переключившись в вид исходного кода (source) можно уже заниматься логикой. Сервисы так же можно создать и на сервере [jboss](http://labs.jboss.com/), а [tomcat](http://tomcat.apache.org/) вроде не поволяет.

### Использование

Прежде всего - копируем ссылку описания сервиса из настроек (правый щелчок на webservice --Properties), которая выглядит примерно так:

`http://localhost:8080/WebApplication1/HelloService?wsdl   `

Дальше открываем Visual Studio,создаём новый проект хотя-бы на Visual Basic и добавляем сервис через Data -- Add new data source, куда помещаем скопированную выше ссылку. Теперь в проекте можно легко использовать данные из web-сервиса в качестве обычной переменной:

`Dim serv As New localhost.HelloService   MsgBox(serv.sayHello("Artjom")); //сервис должен ответить "Privet Artjom"   `

Однако, если ваша функция в web-сервисе возвращает тип матрицы (массив из массива строк), то Visual studio не поймёт этого типа,  посчитав обычным массивом строк.

Читайте также

- [Как создать web-сервис на сервере jboss](http://www.engine4business.ru/jboss-servlet-article.html)
- [Web-сервис на tomcat](http://www.ericsson.com/mobilityworld/sub/open/technologies/open_development_tips/docs/jwsdp)  
    
- [Spring framework](http://static.springframework.org/spring-ws/site/reference/html/tutorial.html)