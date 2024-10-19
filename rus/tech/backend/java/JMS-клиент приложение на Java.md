суббота, 13 декабря 2008 г. в 00:08:53

Недавно я писал про [соединение Flex и JMS](https://kurapov.ee/technology/web/flex_blazeds_JMS/), об расширенных возможностях этого соединения по сравнению с ajax. Недавно я узнал что кроме Jboss есть ещё и [WebORB](http://www.themidnightcoders.com/products.html) и [Bea Weblogic](http://www.oracle.com/bea/index.html) в качестве серверов приложений, которые поддерживают и Java и .NET, первый даже php и ruby. А JMS в свою очередь - часть семейства приложений Message Oriented Middleware.

Вобщих чертах технология JMS как и Java уже достаточно стара и активно используется в распределённых приложениях, где архитектура состоит из нескольких сторон - например системе авторизации через мобильный телефон или при оплате кредитной карточкой.

Но сегодня пишем простое клиентское приложение по работе с службой сообщений, в которую в прошлый раз поступило сообщение из Flex. Для этого нам понадобится Eclipse IDE, настроенный в прошлый раз Jboss сервер и голова на плечах.

![](../img/Pasted%20image%2020241020013151.png)

### Проект  

Первым делом - создаём новый java-проект в Eclipse, куда ставим код который я привожу ниже в приложении. Проект и ответсвенный класс я назвал jms_client. К этому времени запуск приведёт к ошибкам типа java.lang.NoClassDefFoundError , а всё дело в том что к проекту надо импортировать библиотеку javax.jms.*

Странным образом я не смог найти чёткого описания где её достать. Даже установив Development kit и Runtime enviroment 1.6.0.07 и Enterprise Edition Software Development Kit 5, я лишь значительно позже додумался присоединить к проекту внешние JAR-библиотеки из jboss/client/

![](../img/Pasted%20image%2020241020013202.png)

### Queue и Topic. Приём и рассылка

Сообщения отсылаются в JMS одинаково, но как только сообщение попадает в виртуальный канал, то доставка может происходить по двум сценариям.

- В **queue** сообщение можно получить только один раз. Если у канала зарегистрировано несколько получателей, то он выбирается случайным образом. Это свойство позволяет равно распределять нагрузку. Если слушателей нет, то сообщение будет ожидать получателя или срок истечения
- В **topic** сообщение получают все слушатели. Если их нет то сообщение теряется. Отличный вариант для чатов.

Тут как и в истории с Flex надо помнить о том что передаваемые сообщения должны быть связаны с конкретным типом - javax.jms.Queue или javax.jms.Topic. Надо сказать что в зависимости от этого в приложении префикс меняется у всех участвующих объектов.

![](../img/Pasted%20image%2020241020013213.png)

### Код  

Для создания ожидающего приложения можно пойти двумя путями - делать бесконечный while или же создать процесс (поток) - т.е. объект который будет сам себя создавать и слушать - именно это делается ниже в _jms_client.java_

Для коннекта к JMS надо настроить JNDI - интерфейс для обращения к серверным java-объектам по имени. Для этого можно использовать либо файл настроек jndi.properties, либо прописать сразу в коде как это сделал я:

`Properties properties = new Properties();   properties.setProperty(Context.INITIAL_CONTEXT_FACTORY, "org.jnp.interfaces.NamingContextFactory");   properties.setProperty(Context.URL_PKG_PREFIXES, "org.jboss.naming:org.jnp.interfaces");   properties.setProperty(Context.PROVIDER_URL,"jnp://localhost:1099");   InitialContext ctx = new InitialContext(properties);`

Дальше по контексту создаётся соединение по паттерну фабрики, привязывается канал в зависимости от его типа (QueueConnectionFactory или TopicConnectionFactory), открывается сессия и только теперь привязывается обработчик через метод setMessageListener к объекту типа MessageListener.

### Обмен сообщениями

Вся мощь в обмене сообщениями конечно же не ограничивается текстовыми данными. Например из Flex можно отправить полноценный объект

`var message: AsyncMessage = new AsyncMessage();   message.body = {a:3,b:7};   producer.send(message);`

А вот в java-приложение этот объект попадёт как ASObject. Самое странное что этот объект в SDK и jboss я найти не смог. Я нашёл билиотеку [com.carbonfive.flash](http://carbonfive.sourceforge.net/astranslator/api/com/carbonfive/flash/package-summary.html#documentation), но прикрутить я не смог. Поэтому пришлось делать приведение типов.

`public void onMessage(Message msg){      Object tm = (Object) msg;      System.out.println("Got message:" + tm.toString());   }`

Читайте по теме:

- [JMS и WebSphere](http://www.rsdn.ru/article/java/WAS.xml)
- [Код полноценного JMS клиента на Jboss](http://www.jboss.org/file-access/default/members/jbossas/freezone/docs/Server_Configuration_Guide/4/html/JMS_Examples-A_Pub_Sub_Example.html)
- [Jboss getting started](http://www.redhat.com/docs/en-US/JBoss_Enterprise_Application_Platform/4.3.0.cp02/html-single/Getting_Started/index.html)
- [Сервис на Jboss 4.2](http://www.engine4business.ru/jboss-servlet-article.html)
- [The components of JMS](http://www.informit.com/articles/article.aspx?p=26270&seqNum=8)
- [Простая отправка сообщения с JMS](http://arhipov.blogspot.com/2009/10/my-complains-to-jms.html)