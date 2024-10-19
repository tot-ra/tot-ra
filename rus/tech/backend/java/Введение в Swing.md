вторник, 19 февраля 2008 г. в 00:01:58

Продолжаю эксперементировать и изучать Java. Сегодня я познакомился с библиотекой javax.swing, которая позволяет с относительной простотой создавать пользовательский интерфейс. На этот раз я использую не любимую Eclipse, а с трудом скачанный JDeveloper с [сайта Oracle](http://www.oracle.com/technology/software/products/jdev/index.html).

![](../img/Pasted%20image%2020241020013500.png)

![](../img/Pasted%20image%2020241020013509.png)

Создаю новую программу - "Application", затем через Wizard (New/Projects/Java Application Project) добавляю проект. Автоматически создаются окошки типа JFrame и по моему выбору - встраивается менюшка (JMenu + JMenuBar + JMenuItem). Элементарное задание - научиться работать с меню и табличкой (JTable).

Добавить элемент меню как оказалось очень просто - как в VisualStudio выбрав в дизайне нужный раздел. А вот связывать событие нажатия пришлось изучая существующий код для AboutBox'а о том как используется ActionListener. Создав второе окно (frame), и связав через addActionListener функцию обработки события, окошко можно показать простым кодом:

`Frame FriendsFrame= new FriendsFrame(); //класс наследует JFrame   FriendsFrame.setVisible(true);`

![](../img/Pasted%20image%2020241020013520.png)
![](../img/Pasted%20image%2020241020013529.png)

С окошками разобрались, добавим теперь за табличку . Как полезно [объясняет RSDN](http://www.rsdn.ru/article/java/QnAJava.xml), чтобы иметь работать с данными таблицы, необходимо иметь модель данных. Можно написать свою реализацию TableModel либо AbstractTableModel. Можно также воспользоваться существующим классом javax.swing.table.DefaultTableModel. Для этого нужно создавать таблицу с явным указанием модели:  
  
```java
DefaultTableModel model = new DefaultTableModel();  
jTable1 = new JTable(model);  
model.addColumn("Name");  
model.addColumn("Posts");  
for (int c=0; c<10; c++){  
model.addRow(new Object[]{"User"+c,Math.round(10*Math.random())});  
}  
  
jTable1.setBounds(new Rectangle(35, 45, 310, 260));
```

  
Читайте также

- Подробно от IBM о [редактировании таблиц](http://www.ibm.com/developerworks/java/library/j-jtable/)
- [Grails](http://grails.org/Grails+Screencasts) аналог фреймворка быстрой разработки "Ruby on Rails" для Java
- [Groovy-builder](http://www.ibm.com/developerworks/ru/library/j-pg04125/) как способ упросить создание интерфейса
- 