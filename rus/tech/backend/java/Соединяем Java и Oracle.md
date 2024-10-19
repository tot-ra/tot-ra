понедельник, 3 марта 2008 г. в 10:57:55

Я недавно писал о том как соединялся с [Postgre из servlet](https://kurapov.ee/article/postgre_jdbc/)'ов, на этот раз я использую среду разработки JDeveloper, библиотеку интерфейсов [Swing](https://kurapov.ee/article/jtable_swing/) и конечно базу данных Oracle. Бесплатная версия express edition, по аналогии с microsoft'ными XE очень ограниченная и загружающая активно память - на лаптопы ставить не советую. К счастью у меня есть удалённый сервер в университете, который можно эксплуатировать.

После создания Swing-проекта и добавления Oracle драйверов-библиотек, пишем "conn" в качестве параметра запускаемого класса, нажимае Ctrl+Enter и получаем примерно такой код:

```java
public static Connection getConnection() throws SQLException { //Удобней так указывать исключение (exception) на случай ошибки  
    String thinConn = "jdbc:oracle:thin:@localhost:1521:ORCL"; //thin - тип драйвера, ORCL - имя сессии  
    DriverManager.registerDriver(new OracleDriver());  
    Connection conn = DriverManager.getConnection(thinConn,"Scott","Tiger");  
    conn.setAutoCommit(false); //по умолчанию запросы БД выполняются только после ручного подтверждения - Commit'а  
   return conn;  
}
```