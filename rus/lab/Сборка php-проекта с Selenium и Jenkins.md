понедельник, 15 августа 2011 г. в 20:47:50

В этой статье я теперь попытаюсь описать как собирать php-проект вместе и как проводить deployment так, что-бы разработчики видели прогресс и состояние здоровья проекта с помощью инструментов [статического анализа](http://kurapov.name/rus/lab/php_code_analysis/) кода, что-бы были видны результаты запуска unit- и selenium- тестов с результатом покрытия, что-бы проверялся принятый стиль кода. В качестве груши для битья я возьму Drupal 7.8 для анализа.

### Установка Jenkins на Ubuntu

![](img/Pasted%20image%2020241020022417.png)


Jenkins удобно поставляется как java war-файл и может спокойно и на Windows бегать, но я хочу переносимую ОС с хорошим бэкапом, поэтому поставим сервер на виртуальную машинку с Ubuntu 11.   

`sudo add-apt-repository ppa:hudson-ubuntu/testing sudo apt-get update sudo apt-get install jenkins`

Теперь на http://localhost:8080 и в /var/lib/jenkins у нас стоит сервак. Поставим PEAR и немного библиотек для phpunit:

`apt-get install php-pear pear config-set auto_discover 1 pear install pear.phpqatools.org/phpqatools PHPDocumentor`

### Установка расширений

Плагины можно поставить и из web-интерфейса.. но из кода вроде как быстрей..

`cd /tmp wget http://localhost:8080/jnlpJars/jenkins-cli.jar java -jar jenkins-cli.jar -s http://localhost:8080 install-plugin xunit java -jar jenkins-cli.jar -s http://localhost:8080 install-plugin clover java -jar jenkins-cli.jar -s http://localhost:8080 install-plugin git java -jar jenkins-cli.jar -s http://localhost:8080 install-plugin checkstyle java -jar jenkins-cli.jar -s http://localhost:8080 install-plugin cloverphp java -jar jenkins-cli.jar -s http://localhost:8080 install-plugin dry java -jar jenkins-cli.jar -s http://localhost:8080 install-plugin htmlpublisher java -jar jenkins-cli.jar -s http://localhost:8080 install-plugin jdepend java -jar jenkins-cli.jar -s http://localhost:8080 install-plugin plot java -jar jenkins-cli.jar -s http://localhost:8080 install-plugin pmd java -jar jenkins-cli.jar -s http://localhost:8080 install-plugin violations java -jar jenkins-cli.jar -s http://localhost:8080 safe-restart`

### Если проект лежит в github или beanstalk

Jenkins хорошо работает и с SVN, но я использую git и в частности публичный репозиторий github. Поставим git и скачаем config.xml файл-заготовку (как новый неактивный проек всех настроек для быстрого создания нового проекта...

`apt-get install git git config --system user.email "artkurapov@gmail.com" git config --system user.name "Artjom Kurapov"`

Для безопасного доступа к моему приватному хранилищу на github, надо сделать новые ssh-ключи, желательно без пароля, от имени отдельного пользователя jenkins. Результат надо будет сохранить /var/lib/jenkins/.ssh/id_rsa (и это [будет спрошено](http://help.github.com/linux-set-up-git/) в процессе)

![](img/Pasted%20image%2020241020022442.png)

`sudo -u jenkins ssh-keygen -t rsa "artkurapov@gmail.com" sudo cat /var/lib/jenkins/.ssh/id_rsa.pub`

Последней строчкой мы открываем файл сгенерированного публичного ключа - его надо скопировать в [панель управления](https://github.com/account/ssh) github или beanstalk.

### Настройка проекта по шаблону

[Phptemplate](http://jenkins-php.org/) это специальный шаблон для php-проектов. Для его установки`cd /var/lib/jenkins/jobs sudo -u jenkins git clone git://github.com/sebastianbergmann/php-jenkins-template.git php-template java -jar /tmp/jenkins-cli.jar -s http://localhost:8080 reload-configuration`

Перезагрузить Jenkins можно либо коммандой **service jenkins restart**, либо по HTTP открыть страницу /restart

### Сборка

Теперь создадим новый проект на основе выше описанного шаблона. В качестве источника кода ставим SSH-путь к проекту на github. Если теперь запустить "build now" то в принципе код скачается, но анализа кода не произойдёт и поэтому выдать красивые графики плагины не смогут.

![](img/Pasted%20image%2020241020022456.png)

Для того что-бы происходил реальный анализ кода, в репозиторий надо добавить build.xml файл где будет описан процесс сборки. Таким образом jenkins запустит [Apache Ant](http://ant.apache.org/) коммандой **ant build**, который в свою очередь найдёт строчку в этом файле.. 

`<target name="build" depends="clean,phpunit,parallelTasks,phpcb" />`

Дальше ant последовательно запустит эти подзадачи. Хитрые задачи можно группировать в последовательно (sequential тэг) или параллельно с фиксированным числом потоков (parallel тэг). К примеру если phpunit-задача описана так..

`<target name="phpunit"> <exec executable="phpunit" dir="${basedir}/" failonerror="true"> <arg line="--log-junit ${basedir}/build/logs/junit.xml --coverage-clover ${basedir}/build/coverage/clover.xml --coverage-html ${basedir}/build/coverage --bootstrap ${basedir}/tests/bootstrap.php ${basedir}/tests/unit/" /> </exec> </target>`

То соответственно результаты покрытия методов пойдут в clover.xml, которые будут подхвачены дальше Jenkins-плагинами для html-отчёта. То же самое с остальными блоками в этом файле - просто описываются какие программы проводят анализ кода на разные метрики, какие пути стоит игнорировать и куда сохранять результаты.

## Selenium и VNC

Мне в своё время никто не объяснял, но в виртуальной машине вполне можно запустить браузер (в данном случае на шестой дисплей)

`DISPLAY=:6 firefox`

Другое дело что визуально это будет видно только если подключиться соответсвующей клиентской программой (TigerVNC, Chicken VNC) к специально поставленному и запущенному VNC серверу:

`x11vnc -display :6`

Если напрямую соединиться нельзя, то клиент может запустить ssh-туннель и уже в клиентской программе пробовать соединиться с локальным сервером

`ssh -l ArtjomKurapov -L 5906:127.0.0.1:5906 example.com`

Теперь, для Selenium желательно создать отдельный профиль, что-бы небыло проблем с popup'ами и диалоговыми окнами, которые надо отключить в настройках. Режим управления профилями открывается коммандой

`firefox -profilemanager`

Теперь основная, сложнейшая комманда, которая указывает что используется 6 дисплей, запускается selenium сервер и указывает путь к профилю (у вас путь будет другой). 

`export DISPLAY=:6 & java -jar selenium-server-standalone-2.20.0.jar -port 4445 -firefoxProfileTemplate /root/.mozilla/firefox/n2gorg4.Selenium default user/`

Каждый раз когда Selenium будет запускаться, он будет поднимать браузер на заданном дисплее и прогонять через него тесты, при этом разработчик может смотреть процесс через vnc. Остановить selenium можно удалённо http-запросом http://localhost:4444/selenium-server/driver/?cmd=shutDownSeleniumServer

Задачи разные и всё конечно зависит от того что вы хотите в результатах увидеть..

- Измерить размер проетка (phploc)
- Расчитать метрики (pdepend)
- Найти говнокод (phpmd)
- Найти нарушения стандартов стиля кода (phpcs)
- Найти дублирующийся код (phpcpd)
- Сгенерить документацию (phpdoc)
- Агрегировать логи в просмотрщик кода (phpcb)

### Проблемы

Я столкнулся с уймой проблем..

| Ошибка                                                                                                                                                                                                                                                                           | Решение                                                                                                                                                       |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `FATAL: command execution failed.Maybe you need to configure the job to choose one of your Ant installations? java.io.IOException: Cannot run program "ant" (in directory "/var/lib/jenkins/jobs/wordpress/workspace"): java.io.IOException: error=2, No such file or directory` | Разрешилось тем, что в настройках Jenkins надо добавить версию ant (по видимому у меня в системе он не установлен был) и дальше связать проект с этой версией |
| `/var/lib/jenkins/jobs/wordpress/workspace/build.xml:19: Execute failed: java.io.IOException: Cannot run program "phpunit": java.io.IOException: error=2, No such file or directory`                                                                                             | Разрешилось обновлением Pear установкой PHPUnit:  <br>`pear update sudo pear install phpunit/PHPUnit`                                                         |
| `[exec] The Xdebug extension is not loaded. [exec] PHP Fatal error: Allowed memory size of 268435456 bytes exhausted (tried to allocate 4194304 bytes) in ..`                                                                                                                    | при анализе кода надо дофига оперативки. Если возникнет такая ошибка, то падает весь билд.                                                                    |