пятница, 2 февраля 2007 г. в 13:03:39

Дизайн базы данных дело приятное и также важное при создании новой системы.  
Под дизайном я подразумеваю

1. Разбиение  проекта на независимые логические существа данных, т.е. таблицы. Обычно ни клиент ни управляющий проектом не описывают их.
2. Создание структуры таблицы с корректными типами данных.
3. Создание связей между таблицами через [Foreign Key](http://0804team.kiev.ua/dm/blog/2006/05/07/%d1%8d%d0%ba%d1%81%d0%ba%d1%83%d1%80%d1%81-%d0%b2-sql/) , если возможно
4. Создание функций, триггеров, последовательностей и прочих надстроек если это возможно.

Для хорошего понимания и удобства таблицы составляются визуально и связываются. В качестве визуальных программ-редакторов можно выделить..[  
](http://fabforce.net/dbdesigner4/)

- [Mysql Workbench](http://dev.mysql.com/downloads/gui-tools/5.0.html)
- [FabForce DBdesigner4](http://fabforce.net/dbdesigner4/)
- [MicroOLAP Database Designer](http://www.microolap.com/products/database/mysql-designer/) (платный)
- В MS Access уже встроено
- [DB visualizer](http://www.minq.se/products/dbvis/)
- SQL maestro for MySQL