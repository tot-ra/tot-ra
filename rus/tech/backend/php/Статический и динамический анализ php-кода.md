понедельник, 15 августа 2011 г. в 10:04:50

Автоматический анализ кода (_static code analysis_) очень полезен на больших проектах и он часто встраивается в серверы [непрерывной интеграции](http://www.martinfowler.com/articles/continuousIntegration.html). Некоторые IDE уже поставляются с простыми аналитическими инструментами, но первые всё-таки предпочтительней, потому что туда смотрит вся комманда. Всё что этот софт делает это периодически смотрит в систему версионирования (SVN) и строит график качества (и например запускает юнит-тесты). По сути это аналог комплекса упражнений для человека, поддерживающих [хорошее здоровье](http://ithappens.ru/story/7693) и бъющих тревогу если возникает рак спагетти-кода.

Самые известные CI-серверы:

- [Hudson](http://hudson-ci.org/) и его бесплатный двойник [Jenkins](http://jenkins-ci.org/) (на java, много модулей) → есть [шаблон](http://jenkins-php.org/)
- [CruiseControl](http://cruisecontrol.sourceforge.net/)
- Atlassian [Bamboo](http://www.atlassian.com/software/bamboo/)
- Jetbrains [TeamCity](http://www.jetbrains.com/teamcity/)
- [BuildBot](http://trac.buildbot.net/)  
    
- [Xinc  
    ](http://code.google.com/p/xinc/)
- [Arbit](http://arbitracker.org/arbit/about.html) (включает в себя project management и bugtracker)

### Статический анализ кода и его метрики

В целом, статический анализ кода сводится к оценке метрик кода с поиском потенциальных проблем. Начиная с банальных необъявленных или не использующихся переменных и заканчивая поиском дубликатов. Метрики позволяют оценить цену и сложность проекта для планирования дальнейших работ, позволяют оценить влияние от новых методологий и инструментов, уменьшают время поиска проблемных мест как по пакетам так и по разработчикам.

Метрик [очень много](http://www.aivosto.com/project/help/pm-list.html) и каждый аналитический инструмент по разному их формулирует и сокращает, поэтому единой картинки добиться сложно, хотя и есть закономерность в шагании по стопам java

#### Размерные метрики

- NOP - число пакетов
- NOC - число классов → Число классов в пакете (Java =17, C++=19)
- NOM - число методов → Число методов в классе (Java = 7, C++ = 9)
- LOC - число строк → Ошибок/KLOC, Документации/KLOC
- IOp - число параметров входа/выхода
- IOg - число переменных вызываемых методом (классовых и глобальных, не локальных)
- IOvar = IOp + IOg
- CS - число атрибутов и методов. Может указывать на слишком большую ответственность одного класса
- NOO - число переписанных родительских методов в дочерних классах. Указывает на высокую или низкую абстракцию
- NOA - число новых методов в зависимости от глубины наследования

#### Метрики наследования

- ANDC - среднее число наследующих классов (Java = 0.41, C++ = 0.28)
- AHH - средняя высота иерархии (Java = 0.21, C++ = 0.13)
- MIF = число унаследованных но не перезаписанных методов / общее число методов, указывает на степень абстракции или специализации класса (значение от 0 до 1)
- PF - фактор наследования = число наследуемых методов / (число новых методов * число дочерних классов ), показывает насколько используются наследуемые методы (значение от 0 до 1)

#### Метрики сложности

- CF - фактор связанности двух классов (т.е. один класс вызывает или использует методы/свойства другого класса) = число вызовов / максимальное число вызовов (значение от 0 до 1)
- CALLS - число вызовов методов
- FANOUT- число (вызываемых ) классов → (fin+fout)2 * len
- Структурная сложность = CALLS 2
- Сложность данных SC = IOvar / (CALLS +1)
- Цикломатическая сложность CC = число решений / число строк (Java = 0.2, C++ = 0.25). Разные научные определения ([Halstead](http://en.wikipedia.org/wiki/Halstead_complexity_measures), McCabe, McClure)
- Системная сложность: SYSC = SC + SD

Некоторые программы ещё измеряют степень безопасности через число точек прямого входа, т.е. использования параметров GET, POST, SESSION, FILE. 

  

#### Инструменты

Для php таких аналитических программ относительно мало — есть консольные узкоспециализированные программки:

- [Depend](http://pdepend.org/) - использует некоторые приведённые выше метрики для анализа сложности `pdepend --overview-pyramid=out.svg my_project_namepdepend --jdepend-chart=out2.svg my_project_name   pdepend --jdepend-xml=out.xml my_project_namedependencies.php out.xml -o out3.svg`  
    
- [Mess detector](http://phpmd.org/) - ищет неиспользуемый код и сложные выражения `phpmd my_project_name text phpmd.xml`  
    
- [Code sniffer](http://pear.php.net/package/PHP_CodeSniffer/) - проверяет названия методов и переменных согласно правилам из XML-файла настроек `phpcs --standard=CodeREview --report-source my_project_name`  
    
- [Dead code detector](https://github.com/sebastianbergmann/phpdcd) ищет невызываемый код
- [Copy-paste detector](https://github.com/sebastianbergmann/phpcpd)
- [phploc](http://github.com/sebastianbergmann/phploc) - оценивает размер  
    `phploc my_project_name`  
    
- [PHPLint](http://www.icosaedro.it/phplint/) - проверяет синтаксис и генерирует документацию
- [Analzer for Type Mismatches](https://github.com/colder/phantm) - ищет возможные ошибки с несовместимостью типов

Ещё есть чуть более **общие** [PMD](http://pmd.sourceforge.net/) (на java) и [phpsat](http://www.program-transformation.org/PHP/PhpSat), **специализированные** [RIPS](http://www.php-security.org/2010/05/24/mops-submission-09-rips-a-static-source-code-analyser-for-vulnerabilities-in-php-scripts/index.html), [RATS](https://www.fortify.com/ssa-elements/threat-intelligence/rats.html) (как вариант Fortify360 + Jenkins), [Yasca](http://yasca.org/), [Pixy](http://pixybox.seclab.tuwien.ac.at/pixy/) и **агрегаторы** всего что только можно - PHPUnit,  [PHPLint](http://www.icosaedro.it/phplint/), [Sonar](http://www.sonarsource.org/)

### Динамический анализ кода

В отличие от статического анализа, здесь система должна работать. Причём нас на этом этапе заботят - скорость работы всей системы (и узкие места) и логическая безошибочность. Соответсвенно решается это  

- Покрытие кода тестами (Statement, Branch, Path coverage)
- Граф скорости загрузки и использования памяти в зависимости от методов

Инструменты

- Selenium
- XDebug  
    
- Zend Server
- [KCachegrind](http://kcachegrind.sourceforge.net/html/Home.html), Webgrind, [cachegrindvisualizer](http://code.google.com/p/cachegrindvisualizer/)